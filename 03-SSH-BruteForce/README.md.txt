# Attack 03 — SSH Brute Force Attack
**Tool:** Hydra | **Target:** Metasploitable 2 | **Difficulty:** Beginner-Intermediate

---

## What is SSH?
SSH (Secure Shell) is the primary protocol used to remotely manage Linux servers.
It encrypts the connection — but encryption does NOT stop attackers from trying 
thousands of passwords automatically. This is called a Brute Force attack.

Every internet-facing SSH server receives automated brute force attempts within 
MINUTES of going live. This is not theoretical — it happens constantly in the real world.

---

## What is Hydra?
Hydra is a fast and flexible password brute force tool that supports 50+ protocols
including SSH, FTP, Telnet, HTTP, RDP and more. It automates credential testing
at high speed — trying hundreds of passwords per minute automatically.

---

## Lab Environment
| Role | Details |
|------|---------|
| Attacker | Kali Linux 2025.2 |
| Target | Metasploitable 2 — 192.168.6.129 |
| Tool | Hydra v9.5 |
| Protocol | SSH — Port 22 |

---

## Vulnerability Found
Target was running OpenSSH 4.7p1 released in 2008.
This version has absolutely no brute force protection — no rate limiting,
no account lockout, no fail2ban. An attacker can try unlimited passwords 
with zero consequences until they get in.

Combined with a weak default password — this system was compromised in under 30 seconds.

---

## Attack Steps

### Step 1 — Reconnaissance
Confirmed SSH is running and identified the exact version:
```
nmap -sV -p 22 192.168.6.129
Result: 22/tcp open ssh OpenSSH 4.7p1
```

### Step 2 — Manual Connection Attempt
Attempted manual SSH login to confirm connectivity and understand 
what legacy algorithm errors appear with such an old SSH version.

### Step 3 — Wordlist Creation
Created a targeted password list for the lab.
In real attacks, rockyou.txt (14 million passwords) or custom 
breach databases are used.

### Step 4 — Brute Force With Hydra
```
hydra -l msfadmin -P small_pass.txt ssh://192.168.6.129 -t 4 -V
```
Hydra systematically tried each password until it found the match.

### Step 5 — Credentials Cracked
```
[22][ssh] host: 192.168.6.129   login: msfadmin   password: msfadmin
```
Full SSH access gained. Attack complete.

### Step 6 — Auth Log Analysis
After gaining access, pulled /var/log/auth.log from the target.
This shows exactly what a SOC analyst sees during a brute force attack —
multiple Failed password entries followed by one Accepted password entry.

---

## What The Auth Log Shows (SOC Perspective)
```
Failed password for msfadmin from 192.168.6.128
Failed password for msfadmin from 192.168.6.128
Failed password for msfadmin from 192.168.6.128
Accepted password for msfadmin from 192.168.6.128
```
Pattern: Multiple failures + one success from same IP = brute force + compromise.
This is a CRITICAL alert in any SOC environment.

---

## MITRE ATT&CK Mapping
| Field | Detail |
|-------|--------|
| Tactic | Credential Access |
| Technique | T1110.001 — Brute Force: Password Guessing |
| Platform | Linux |

---

## SOC Detection Rules

**Splunk:**
```
index=linux sourcetype=secure "Failed password"
| stats count by src_ip
| where count > 10
```

**Alert Rule:**
More than 10 failed SSH logins from single IP in 60 seconds = brute force in progress.
Successful login after multiple failures = compromised account. Immediate investigation required.

---

## How Real Attackers Stay Hidden
- They use distributed brute force — 1000 different IPs each trying 1 password
- No single IP gets blocked by fail2ban
- They spray slowly — 1 attempt every 30 minutes to avoid detection
- They use leaked credential databases — 60-80% of users reuse passwords

---

## Defence Mechanisms
| Defence | How It Helps |
|---------|-------------|
| Install fail2ban | Auto-bans IPs after N failed logins |
| Disable password auth | Forces SSH key pairs — passwords useless |
| Change SSH port | Reduces automated bot traffic by 90% |
| Enable MFA | Even cracked password is not enough |
| VPN-only SSH | SSH not even reachable without VPN |

---

## Evidence Files
| File | Description |
|------|-------------|
| hydra_results.txt | Full Hydra output showing cracked credentials |
| auth_log_sample.txt | Auth logs from target showing brute force pattern |
| nmap_ssh_scan.txt | Reconnaissance scan output |
| screenshots/ | Visual proof of every attack step |

---

## Key Takeaway
SSH brute force is one of the most common attacks in the world.
The defence is simple — disable password authentication entirely and use SSH keys.
Yet thousands of servers get compromised this way every single day
because administrators never implement basic hardening.

As a SOC analyst — any SSH brute force alert must be treated as
an active incident until proven otherwise.