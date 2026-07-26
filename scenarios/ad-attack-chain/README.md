## Domain & Host Inventory
- Domain: LAB.LOCAL
- DC: Windows 2022 Server
- Workstation: Windows 11
- Attacker: Kali Linux
- Subnet: 10.10.10.0/24
- Key Accounts and their roles
  - j.admin (Domain Admin)
  - svc_sql (Domain user, SPN, local admin on Windows 11)
  - jcyber (Nonprivileged user on Windows 11)
- Data: sensitive.txt
  - Stored on Windows 11 under j.admin's account
  - Placed there via RDP
  - Attacker's ultimate goal

## Detection Stack Baseline
- Suricata on Proxmox host, logging to eve.json
- Splunk Free (500MB/day limit) with cron scheduling used for alerts
- Sysmon + Universal Forwarder used on Windows 11 and DC. SwiftOnSecuirty baseline config
  - Key IDs collected: 1, 3, 4, 10, 11
- Windows Event Logs: Security, System, Application
- GPO Audit subcategories enabled
  - Key IDs collected: 4768, 4769, 4624, 4625, 4720, 4738, 4732, 4688, 4719, 4103, 4104, 4698, 4697
 
## Deliberate Control Regressions
| Control | Change | GPO Location | Explanation |
| --- | --- | --- | --- |
| SMB Signing | Disabled | Computer Config --> Security Options | Disabled to allow for LLMNR poisoning |
| RunAsPPL | 0 | Registry / Local Security Policy | Disabled to allow for Mimikatz use in future scenarios |
| Credential Guard | Off | Computer Config --> Admin Templates | Disabled to allow for Mimikatz use in future scenarios |
| WDigest | Enabled | Registry | Enabled to allow for Mimikatz use in future scenarios |
| RC4 Kerberos | Permitted | Security Options --> Network Security: Configure Encryption Types | Enabled to allow for Kerberoasting |
| Defender | Disabled | Admin Templates --> Windows Components --> Microsoft Defender Antivirus | Disabled to allow for Sharphound or malware use in future scenarios |

## Backup Process
- Backups of individual target VMs were taken immediately before starting any attacks. They can be restored after future sessions to bring the lab back to a vulnerable, untouched state after a lengthy attack.

## Real-World Application
- In a real environment, none of the aforementioned controls would need to be set in the manner reflected in this lab. These settings are meant to create an insecurity that is intentional and instructive, not a misunderstanding of best practices.
