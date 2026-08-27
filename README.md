# purple-team-homelab
This homelab aims to emulate security scenarios that enterprises may face using realistic attack chains and building detection across Splunk and Suricata. 

## Hardware
- Dell Latitude 3310 (Isolator)
  - intel i5 8365u, 16gb ram, 128gb ssd
- Dell Precision Tower 3430 SSF (Main)
  - intel i7 8700, 64gb ram, 1tb NVME + 256gb SATA
 
## Tools
| Category | Tools |
|---|---|
| Firewall/Router | pfSense |
| Hypervisor | Proxmox |
| SIEM | Splunk |
| VPN | TailScale |
| IDS/IPS | Suricata |
| Attack Tools | Kali Linux |
| Domain | Windows Server 2022 AD |
| Target | Windows 11 Workstation |

## Design Philosophy
This lab was designed with a primary focus on Splunk analysis. Targets are weakened to provide an easier surface of attack. Rather than focusing on simulating as many attacks as possible, this lab simulates a few detailed attack chains in favor of in-depth analysis. 

## Scenarios
| Scenario | Techniques Covered | Detection Sources | Status | 
| --- | --- | --- | --- |
| [AD Attack Chain](scenarios/ad-attack-chain/) | LLMNR/NBT-NS Poisoning, Kerberoasting, Lateral Movement, Scheduled Task Persistence, data exfiltration | Sysmon, Security Event Logs, Suricata | Completed |
| Multi-VM Correlation | Triage, Attacks across hosts, Cross-source timeline correlation | Sysmon, Security Event Logs, Suricata | Planned |
| Dynamic Malware Analysis | Process tree analysis, Network callback detection, persistence observation | Sysmon | Planned |

## Repo Structure
- [scenarios/](scenarios/) - Contains folders that are categorized by the attacks they simulate
- [spl/](spl/) - Contains folders with Splunk alerts and searches
- [rules/](rules/) - Contains folders with Suricata rules
- [diagrams/](diagrams/) - Contains diagrams of the lab itself and the various attacks

## Key Detection Engineering Decisions
1. Skipping out on more advanced attacks (Mimikatz, Bloodhound) in favor of more realistic movement - I initially planned to have a more extensive AD attack chain, but I realized complexity for the sake of analysis is not always useful. An attacker is less likely to prolong their attack using complex tools if they could just use admin privileges to directly access data. 

## Notable Gaps/Limitations
1. Not fully realistic - Target VMs are very weak and act in a manner that doesn't inherently correlate with reality. This tradeoff allows for easier attacks, and in turn, a more direct analysis of malicious behavior. 
