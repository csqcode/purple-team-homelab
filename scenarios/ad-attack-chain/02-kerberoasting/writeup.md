# 01 - Kerberosating

**MITRE ATT&CK:** T1558.003 - Steal or Forge Kerberos Tickets: Kerberoasting
  - T1018 - Remote System Discovery

**Status:** Complete

---

## Objective
Use NBTscan to find the DC and continue the attack chain. Use Impacket to get user SPNs, providing elevated access to the domain.

---

## Environment / Prerequisites

- **Attacker:** Kali
- **Target:** svc_sql
- **Starting access:** jcyber username and password
- **Lab weakening required:** Enable RC4 Encryption for Kerberos

---

## Attack Execution
1. Perform an NBTscan to find the IP address of the DC
   - ```nbtscan 10.10.10.0/24```
   - <img width="169" height="52" alt="Kali_DC" src="https://github.com/user-attachments/assets/cc1809e9-1514-424f-b77e-21fa2a4035b2" />


2. Use Impacket to get a NTLM hash
   - ```impacket-GetUserSPNs lab.local/jcyber -dc-ip 10.10.10.4 -request```
   - <img width="441" height="221" alt="Kali_Kerberoast" src="https://github.com/user-attachments/assets/6ddaf1d8-c4a1-4717-8475-68bf4a086e47" />
3. Use John or Hashcat like in 01-responder

---

## Explanation
- After the previous steps taken in 01-responder, we were able to gain access to the AD. Though this access has no administrator privileges, it lets us request a Kerberos ticket-granting service service ticket for the domain's SPN.
- Since some of these tickets may have RC4 encryption, they can be exploited to gain access to the service account associated with the specific SPN. This allows for privilege escalation, leaving us with administrator access to the domain.

---

## Detection Engineering
**Telementry Source:** Windows Security

```index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17```

**Rationale:** Security ID 4769 tracks the instances of successful and failed ticket requests. Encryption Type 0x17 maps to the RC4 encryption system, which is outdated and exploited in the attack we emulated.

---

## Investigation Walkthrough
Splunk:
  
  <img width="681" height="439" alt="Splunk_Kerberoast" src="https://github.com/user-attachments/assets/d66cf883-91db-4abf-a755-775a8fde4711" />
  
Alert: EID 4769, Encryption Type 0x17 (RC4) - TGS request for svc_sql SPN

Initial Assessment: RC4 encryption is suspicious on a modern system and should be investigated; its presence alone, however, is not indicative of an attack.

Investigation: Queried Account_Name --> Found it was the low-privileged 'jcyber' account present in the LLMNR attack. Looked at the log time to see that the ticket was requested shortly after the LLMNR attack.

Correlation: Cross-referenced with the LLMNR alert on the same low-priv account within a short timeframe - consistent with a credential-theft to privilege-escalation attack chain

Verdict: True Positive - Kerberoasting attempt

Recommended Response: Reset svc_sql password, review SPN necessity, enable AES-only encryption, treat jcyber and svc_sql as compromised accounts

---

## Detection Gaps / Limitations
- The Splunk alert is prone to false positives on older systems that still primarily use RC4 encryption. This will be less common on modern systems.
- Suricata gave a false positive alert for possible GoldenPac use. This is not unexpected as GoldenPac is also made by impacket. While it would mislead an investigation in the real world, it would still show that the user has been compromised and there is an active threat in the AD.

---

## Real-World Mitigation
- Disable RC4 encryption or enable AES encryption
- Use strong passwords that expire periodically
- Limit the privileges of service accounts

---

## Takeaways
- This attack exemplifies the importance of updating systems. If RC4 encryption was disabled, I would not have been able to do this attack at all
- While systems can be forgotten about and remain outdated, it is less likely to happen with proper security measures in place. Suricata was sending me dozens of alerts about how my Kerberos encryption measures were weak. 
- I was shocked by how nobody on the domain had to do anything for this step. Kali alone was able to escalate privileges using the automatic Kerberos ticketing.
- If I were to replicate this again, I would like to configure an alert to detect NBTscan



