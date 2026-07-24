# 03 - Lateral Movement

**MITRE ATT&CK:** T1210 - Exploitation of Remote Services

**Status:** Complete

---

## Objective

Use the credentials gained from 02-kerberoasting to access an admin account. Use these privileges to exfiltrate data. 

---

## Environment / Prerequisites

**Attacker:** Kali
**Target:** sensitive.txt
**Starting Access:** Administrator credentials
**Lab Weakening Required:** N/A

---

## Attack Execution
1. Establish an RDP session between Kali and Windows 11
   - xfreerdp /u:svc_sql /p:[password] /v:10.10.10.x /drive:kali,/home/kali/Downloads /dynamic-resolution
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

## Detection Engineering

**Telementry Source:** Suricata, Windows Sysmon

Security RDP:

<img width="596" height="439" alt="Splunk_RemoteAccess" src="https://github.com/user-attachments/assets/fc61d7b3-206f-4acc-b987-511618f5b190" />

- After the investigation in 02-kerberoasting, we know that svc_sql is a compromised account. As a result this remote connection would be something to investigate. It could be a legitimate user trying to access the account (providing even more vulnerabilities due to the active threat), or an attacker trying to use administrator privileges.
- Since we can't rule out anything from this log alone, we need to look at other sources

Suricata RDP:

<img width="922" height="753" alt="image" src="https://github.com/user-attachments/assets/b039e94b-abf0-4dec-b46a-6d7b3617599a" />

- This log confirms the RDP session to be malicious. The source IP matches the attacker IP from earlier attacks.
- From this point onward, everything that the svc_sql account does needs to be questioned. The attacker has full access to the AD, so every action taken must be investigated.

Sysmon Exfiltration:

<img width="718" height="353" alt="Splunk_Exfiltration" src="https://github.com/user-attachments/assets/9131c53e-b006-4cbd-86c6-df962ab310e5" />

- Sysmon ID 1 is triggered everytime a new process is created. Reading through the log, we can see that a new file named "sensitive.txt" was made in a shared drive named "kali".
- With this in mind, it would be useful to search the system for traces of "sensitive.txt". The file could represent some data Kali had stolen, or it could be a malicious file to be planted on the system.
