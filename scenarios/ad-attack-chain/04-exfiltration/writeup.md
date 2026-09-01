# 04 - Exfiltration

**MITRE ATT&CK:** T1021.001 - Remote Services: Remote Desktop Protocol

**Status:** Complete

---

## Objective

Use new privileges to access and exfiltrate sensitive data.

---

## Environment / Prerequisites
- **Attacker:** Kali
- **Target:** sensitive.txt
- **Starting Access:** j.admin files
- **Lab Weakening Required:** N/A

---

## Attack Execution
1. Look for files in j.admin's user folder
   - <img width="271" height="197" alt="Kali_Access" src="https://github.com/user-attachments/assets/2ff0d15b-76d0-4b5e-9594-75f12812c4ca" />
3. Copy data and paste it into shared drive between Kali and Windows 11
4. Verify file reached Kali
   - <img width="165" height="52" alt="Kali_Sensitive" src="https://github.com/user-attachments/assets/edcef731-ea8c-43cf-acd2-0cd311963d41" />
---

## Explanation
- We can use our privileges from 03-lateral-movement to access anything belonging to j.admin. If we encounter anything worth taking, we can store it in a dynamic drive that was created when we started the remote session into svc_sql. This will automatically place the data in a linked drive in Kali.

---

## Detection Engineering

**Telemetry Source:** Sysmon

**Rationale:**  I chose not to make any alerts for this step due to the risk of false positives. Copying data on computer is expected.

---

## Investigation Walkthrough

<img width="718" height="353" alt="Splunk_Exfiltration" src="https://github.com/user-attachments/assets/9131c53e-b006-4cbd-86c6-df962ab310e5" />

**Alert:** ID 1 (Process Creation)

**Initial Assessment:** Process creation is normal and expected of an active account; with the knowledge of a present attacker, however, this log should be investigated.

**Investigation:** The User is listed to be svc_sql. Query Computer_Name --> Matches the IP address of the target computer in the LLMNR attack. The Image line implies svc_sql created, moved, or destroyed a note. The CommandLine field shows this action with the note happened in a tsclient drive called 'kali'. Queried the file 'sensitive.txt' --> Found to be a note taken from the j.admin (domain administrator) account present on the same computer.

**Correlation:** tsclient helps to create a storage drive in a remote desktop session. For a drive named 'kali' to be present after a known compromised login, this is likely a drive meant for the attacker to move data on or off the system. With the known file 'sensitive.txt' found in this drive, the attacker stole sensitive data. This would have been possible with the administrator privileges of svc_sql.

**Verdict:** True Positive - Data exfiltration

**Recommended Response:** Disable the j.admin account and force a credential reset. Assume the entire domain is compromised. Identify the data that was actually taken from j.admin. Restrict RDP access and require MFA.

---

## Detection Gaps / Limitations
- Without any alerts, determining exfiltration from normal behavior can be tedious.
- Very hard to determine legitimate traffic from malicious traffic

---

## Real-World Mitigation
- Use multi-factor authentication for RDP sessions
- Use firewall rules to only allow RDP sessions from specific zones within a network
- Use an IDS/IPS to detect malicious network traffic

---

## Takeaways
- If I did not have my knowledge of the lab setup, this process likely would have taken far longer.
- In the future, I'll need to look into how I can properly create an alert for exfiltration
