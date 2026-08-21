# 03 - Lateral Movement

**MITRE ATT&CK:** T1021.001 - Remote Services: Remote Desktop Protocol
   - T1548.002 - Bypass User Account Control
   - T1048.003 - Exfiltration Over Unencrypted Non-C2 Protocol

**Status:** Complete

---

## Objective

Use the credentials gained from 02-kerberoasting to access an admin account. Use these privileges to exfiltrate data. 

---

## Environment / Prerequisites

- **Attacker:** Kali
- **Target:** sensitive.txt
- **Starting Access:** Administrator credentials
- **Lab Weakening Required:** N/A

---

## Attack Execution
1. Establish an RDP session between Kali and Windows 11
   - ```xfreerdp /u:svc_sql /p:[password] /v:10.10.10.x /drive:kali,/home/kali/Downloads /dynamic-resolution```
   - <img width="359" height="262" alt="Kali_RDP" src="https://github.com/user-attachments/assets/32a5cbf8-1d19-4cb0-a12d-b2414328d2f2" />

2. Once connected, begin looking around for data to exfiltrate
3. Use administrator privileges to access other administrators' files
   - <img width="271" height="197" alt="Kali_Access" src="https://github.com/user-attachments/assets/2ff0d15b-76d0-4b5e-9594-75f12812c4ca" />

5. Copy data and paste it into the shared drive between Kali and Windows 11
6. Verify file is in Kali
   - <img width="165" height="52" alt="Kali_Sensitive" src="https://github.com/user-attachments/assets/edcef731-ea8c-43cf-acd2-0cd311963d41" />

---

## Explanation
- Establishing an RDP session allows full access to the AD with administrator privileges. The specific command we used generates a shared storage drive between Kali and Windows. Any files put in there will be accessible to both instances. This also allows for the transfer of other exploits, like Sharphound and Mimikatz.
- Since this administrator account had no restrains, we could access whatever data we wanted. Peeking through the documents of other administrators only requires us to confirm a pop-up window using our privileges.
- Copying data instead of moving it leaves less of a trace. Once it has been copied into the drive, it will automatically appear in Kali's designated folder to be used and manipulated

---

## Detection Engineering

**Telementry Source:** Suricata, Windows Sysmon

**Rationale:** I chose not to make any alerts for this step due to the risk of false positives. Generating an alert for any RDP session to a specific computer is impractical. Instead of this, I opted to get my conclusions from an active investigation. I looked at the different logs generated after the Kerberoasting session and decided what needed the most attention.

---

## Investigation Walkthrough

Security RDP:

<img width="596" height="439" alt="Splunk_RemoteAccess" src="https://github.com/user-attachments/assets/fc61d7b3-206f-4acc-b987-511618f5b190" />

Alert: EID 4624, Logon Type 10 - Remote Desktop Logon (Successful)

Initial Assessment: A remote login is common and expected in an enterprise environment. Nothing particularly suspicious about the logon.

Investigation: The log shows that svc_sql was logged into. This also occurred after the Kerberoasting attack from before. Since there is no source IP address, we cannot rule out if this was a legitimate or malicious logon.

Correlation: We established before that svc_sql was a compromised account that needs to be monitored. The previous attacker could be trying to gain administrator access.

Verdict: Suspicious - Further investigation needed

Recommended Response: Look at other telemetry sources to see new angles of the logon.

Suricata RDP:

<img width="922" height="753" alt="image" src="https://github.com/user-attachments/assets/b039e94b-abf0-4dec-b46a-6d7b3617599a" />

Alert: Suricata rdp (Remote Desktop Connection Established)

Initial Assessment: The log name and contents initially portray the same story from before, but we have new information about the source IP address

Investigation: Query src_ip --> Found to be the same IP address from the LLMNR attack.

Correlation: After the LLMNR attack, the src_ip gained low-privileged access to the system. It was then able to use this access to create a Kerberos Ticket and get the credentials of svc_sql. We are now seeing the attacker use these credentials to log into svc_sql.

Verdict: True Positive - Privileged Access

Recommended Response: Reset svc_sql password, further monitor actions of svc_sql, jcyber, and the known source IP of the attacker.

Sysmon Exfiltration:

<img width="718" height="353" alt="Splunk_Exfiltration" src="https://github.com/user-attachments/assets/9131c53e-b006-4cbd-86c6-df962ab310e5" />

Alert: ID 1 (Process Creation)

Initial Assessment: Process creation is normal and expected of an active account; with the knowledge of a present attacker, however, this log should be investigated.

Investigation: The User is listed to be svc_sql. Query Computer_Name --> Matches the IP address of the target computer in the LLMNR attack. The Image line implies svc_sql created, moved, or destroyed a note. The CommandLine field shows this action with the note happened in a tsclient drive called 'kali'. Queried the file 'sensitive.txt' --> Found to be a note taken from the j.admin (domain administrator) account present on the same computer.

Correlation: tsclient helps to create a storage drive in a remote desktop session. For a drive named 'kali' to be present after a known compromised login, this is likely a drive meant for the attacker to move data on or off the system. With the known file 'sensitive.txt' found in this drive, the attacker stole sensitive data. This would have been possible with the administrator privileges of svc_sql.

Verdict: True Positive - Lateral movement and data exfiltration

Recommended Response: Disable the j.admin account and force a credential reset. Assume the entire domain is compromised. Identify the data that was actually taken from j.admin. Restrict RDP access and require MFA.

---

## Detection Gaps / Limitations
- Without any active alerts for this step, it can be hard to catch malicious activity (especially if the computer is noisy).
- This step may not be the most realistic for large enterprises where each user has their own device. This may be better representative of smaller businesses, where a tighter budget means administrators and common users have to share the same devices.

---

## Real-World Mitigation
- Use multi-factor authentication for RDP sessions
- Use firewall rules to only allow RDP sessions from specific zones within a network
- Use an IDS/IPS to detect malicious network traffic

---

## Takeaways
- Though I did not have any alerts, I feel like I was still able to get an accurate representation of the situation post-kerberoasting.
- I will need to look more into how I can make specific alerts without generating as many false positives.



