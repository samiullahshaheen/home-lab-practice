# Attack 09 — ARP Poisoning (Bettercap)

## Objective
ARP has no authentication, so an attacker can trick a victim and the
gateway into sending traffic through them instead of each other. This lab
performs an ARP poisoning MITM attack and checks whether Wazuh detects it.

## Environment
| Role | Host | IP |
|---|---|---|
| Attacker | Kali Linux | 192.168.249.141 |
| Victim | Windows 10 | 192.168.249.140 |
| Gateway | VMware host-only | 192.168.249.2 |

Tool: bettercap
Target for credential capture: public HTTP login test page (plaintext, no TLS)

## Steps
1. Checked IP/ARP config on victim and attacker.
2. Enabled IP forwarding on Kali:
   `sudo sysctl -w net.ipv4.ip_forward=1`
3. Ran bettercap and poisoned both directions:
   ```
   sudo bettercap -iface eth0
   set arp.spoof.targets 192.168.249.140
   set arp.spoof.fullduplex true
   arp.spoof on
   net.probe on
   net.sniff on
   ```
4. Verified on the victim — `arp -a` showed the gateway's MAC replaced by
   the attacker's MAC.
5. Browsed to an insecure HTTP login page on the victim and submitted
   test credentials.
6. Bettercap captured the full plaintext POST request — username and
   password visible in the clear.
7. Cleaned up: stopped spoofing/sniffing, disabled IP forwarding.
8. Checked Wazuh for alerts across the attack window.

## Evidence
| Screenshot | Shows |
|---|---|
| `01_victim_ip_arp_before.png` | Victim baseline IP/ARP |
| `01_attacker_ip_arp_before.png` | Attacker baseline IP/ARP |
| `02_ip_forward_enabled.png` | IP forwarding on |
| `03_bettercap_sniff_victim_traffic.png` | Victim traffic visible to attacker |
| `04_victim_arp_poisoned_before_after.png` | Gateway MAC overwritten |
| `05_victim_browser_http_login_page.png` | Victim on insecure login page |
| `06_http_credentials_captured.png` | Plaintext credentials captured |
| `07_cleanup_ip_forward_disabled.png` | Attack torn down |
| `08_wazuh_no_detection_gap.png` | No Wazuh alerts fired |

## Detection Result
Wazuh generated **no alerts**. Its default ruleset watches host-level
logs, file integrity, and registry changes — it has no built-in ARP table
monitoring, so Layer 2 attacks like this go unseen by default.

## Fix
- osquery polling the ARP table for unexpected MAC changes, forwarded to Wazuh
- Suricata/Zeek with ARP spoof rules, feeding Wazuh
- arpwatch on the segment, log-shipped to Wazuh

**Next:** re-test this attack against Wazuh/Splunk after Attack 10, once
detection improvements are in place.

**Takeaway:** ARP poisoning is easy on an unsegmented LAN and exposes
plaintext HTTP traffic completely. Host-only SIEM tuning misses it —
Layer 2 monitoring has to be added deliberately.



This attack was performed entirely within an isolated VMware host-only lab network with no internet-facing or third-party systems in scope.