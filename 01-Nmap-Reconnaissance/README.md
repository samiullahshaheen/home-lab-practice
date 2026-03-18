# Attack 01 — Network Reconnaissance with Nmap

## Lab Details
| Field | Detail |
|-------|--------|
| Date | 18 March 2026 |
| Category | Reconnaissance |
| Difficulty | Beginner |
| MITRE ATT&CK | T1046 — Network Service Scanning |
| Tool | Nmap 7.95 |

## Lab Setup
| Machine | Role | IP |
|---------|------|----|
| Kali Linux | Attacker | 192.168.6.128 |
| Metasploitable 2 | Victim | 192.168.6.129 |

Network: Host-Only adapter — fully isolated from internet.

## Commands I Ran

Step 1 — Find all live hosts on the network:
nmap -sn 192.168.6.0/24

Result: Found 3 hosts alive including Metasploitable at .129

Step 2 — Scan first 1000 ports:
nmap -sS -p 1-1000 192.168.6.129

Result: Found 12 open ports including FTP, SSH, Telnet, SMB

Step 3 — Full scan all 65535 ports with version and OS:
nmap -sV -O -A -p- 192.168.6.129 -oN scan_output.txt

Result: Found 29 open ports. Full output saved in scan_output.txt

## Critical Findings

| Port | Service | Version | Risk Level | Why Dangerous |
|------|---------|---------|------------|---------------|
| 21 | FTP | vsftpd 2.3.4 | CRITICAL | Anonymous login allowed — no password needed |
| 23 | Telnet | Linux telnetd | HIGH | Sends username and password in plaintext |
| 445 | SMB | Samba 3.0.20 | CRITICAL | Known vulnerable — exploitable remotely |
| 1524 | Bindshell | Metasploitable | CRITICAL | Root shell open — anyone can get full control |
| 3306 | MySQL | 5.0.51a | HIGH | Database exposed directly on network |
| 5900 | VNC | Protocol 3.3 | HIGH | Remote desktop with weak authentication |
| 6667 | IRC | UnrealIRCd | HIGH | Known backdoor in this version |

Total open ports discovered: 29

## What a SOC Analyst Should Detect

When an attacker runs Nmap against your network,
your SIEM or IDS should alert on this:

1. Single IP sending SYN to 100+ ports in under 60 seconds
2. SYN packets with no ACK response in firewall logs
3. Malformed TCP packets from OS detection — unusual flag combinations
4. Sequential port scanning pattern from one source IP

Detection Rule Example for Snort or Suricata:
alert tcp any any -> $HOME_NET any (flags:S;
threshold: type threshold, track by_src,
count 100, seconds 60; msg:"Port Scan Detected";)

## Defence Recommendations
- Deploy Wazuh or Snort IDS on all endpoints
- Rate-limit SYN packets per source IP at firewall
- Enable port scan detection in Suricata
- Disable all unused services — close every port not needed
- Never run Telnet — use SSH only
- Never allow anonymous FTP

## Lessons Learned
- 29 open ports found in under 6 minutes
- Port 1524 exposes a root shell — full system compromise in seconds
- Anonymous FTP means anyone can access files without a password
- Telnet sends credentials in plaintext — one sniff = full access
- Old software versions are immediately visible to any attacker
- Always save Nmap output with -oN flag for documentation habit

## Full Scan Output
See scan_output.txt in this folder

## Next
Attack 02 — FTP Anonymous Login and Data Exfiltration