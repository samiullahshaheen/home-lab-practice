# Attack 03 — SSH Brute Force Attack
**Tool:** Hydra | **Target:** Metasploitable 2 | **Difficulty:** Beginner-Intermediate

---

## What is SSH?
SSH (Secure Shell) is a protocol used to remotely control Linux servers
through a terminal. Think of it as a secure remote desktop but through
a command line.

SSH encrypts the connection — but encryption does NOT prevent an attacker
from trying thousands of passwords automatically until one works.
This is called a Brute Force attack.

---

## What is a Brute Force Attack?
A brute force attack is when an attacker tries every possible password
from a list until the correct one is found. No hacking skills required.
Just a tool, a wordlist, and time.

The scary reality — every server connected to the internet receives
automated brute force attempts within minutes of going online.
This is not theoretical. It happens every single day.

---

## What is Hydra?
Hydra is a tool that automates password brute forcing.
Instead of manually trying passwords one by one, Hydra does it
automatically at high speed against protocols like SSH, FTP, and more.

---

## Lab Environment
| Role | Details |
|------|---------|
| Attacker Machine | Kali Linux 2025.2 |
| Target Machine | Metasploitable 2 |
| Target IP | 192.168.6.129 |
| Tool Used | Hydra v9.5 |
| Protocol Attacked | SSH on Port 22 |

---

## Vulnerability Identified
The target was running OpenSSH 4.7p1 which was released in 2008.
This version has zero brute force protection meaning an attacker
can try unlimited passwords with no cooldown and no lockout.

The target also had a weak default password which made it
even easier to crack.

---

## Attack Walkthrough

### Step 1 — Reconnaissance
First confirmed that SSH is running on the target:
```
nmap -sV -p 22 192.168.6.129
```
Result showed port 22 open with OpenSSH 4.7p1 running.

### Step 2 — Manual Connection
Tried connecting manually to confirm the target responds
and to understand what error messages appear with such
an old SSH version.

### Step 3 — Creating a Wordlist
Created a small password list for the lab.
In real attacks, hackers use wordlists with millions of
real passwords leaked from previous data breaches.

### Step 4 — Running Hydra
```
hydra -l msfadmin -P small_pass.txt ssh://192.168.6.129 -t 4 -V
```
Hydra tried each password one by one automatically.

### Step 5 — Password Cracked
```
[22][ssh] host: 192.168.6.129   login: msfadmin   password: msfadmin
```
Full SSH access gained. The entire attack took under 30 seconds.

### Step 6 — Checking the Logs
After gaining access, pulled the authentication log from the target.
This log shows every failed and successful login attempt made
during the attack. This is exactly what a SOC analyst would
investigate after receiving a brute force alert.

---

## What the Auth Log Looks Like
```
Failed password for msfadmin from 192.168.6.128
Failed password for msfadmin from 192.168.6.128
Failed password for msfadmin from 192.168.6.128
Accepted password for msfadmin from 192.168.6.128
```
Multiple failures followed by one success from the same IP address
is the classic pattern of a successful brute force attack.
Any SOC analyst seeing this pattern must treat it as an active incident.

---

## MITRE ATT&CK
| Field | Detail |
|-------|--------|
| Tactic | Credential Access |
| Technique | T1110.001 — Brute Force: Password Guessing |

---

## How SOC Analysts Detect This
- More than 10 failed login attempts from one IP in 60 seconds
- Successful login immediately after multiple failures
- Login attempts happening at unusual hours like 3 AM
- Login from a foreign country or unknown IP address

---

## How Real Attackers Stay Hidden
Instead of thousands of attempts from one IP which gets blocked,
real attackers spread the attack across hundreds of different IPs
each trying only one or two passwords. This way no single IP
gets flagged and the attack stays under the radar.

---

## How to Defend Against This
| Defence | What it Does |
|---------|-------------|
| Install fail2ban | Automatically blocks IPs after too many failed logins |
| Use SSH keys instead of passwords | Password becomes completely useless to attacker |
| Change SSH port from 22 | Reduces automated attack traffic significantly |
| Disable root login over SSH | Attacker cannot directly target the most powerful account |
| Enable two factor authentication | Even correct password is not enough to get in |

---

## Evidence Files
| File | What it Contains |
|------|-----------------|
| hydra_results.txt | Full Hydra output showing the cracked password |
| auth_log_sample.txt | Login logs from the target machine |
| nmap_ssh_scan.txt | Nmap scan showing port 22 open |
| screenshots/ | Visual proof of every step of the attack |

---

## Key Lesson
SSH brute force is one of the most common attacks in existence.
The fix is simple — disable password login and use SSH keys instead.
Yet thousands of servers get compromised this way every day
because administrators skip basic security hardening.

As a SOC analyst, any brute force alert on SSH must be
investigated immediately and treated as a live incident
until proven otherwise.