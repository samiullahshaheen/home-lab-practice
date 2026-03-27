# Attack 06: Directory Enumeration + iptables Rate Limiting Defense

**Tool:** Gobuster | **Defense:** iptables | **Target:** Metasploitable 2 | **Difficulty:** Beginner

---

## Objective

Demonstrate how an attacker uses directory enumeration to map a web server's attack surface, then implement a network-level defense using iptables rate limiting — and prove the defense works by running the same attack again.

---

## Lab Environment

| Role | Details |
|------|---------|
| Attacker | Kali Linux |
| Target | Metasploitable 2 |
| Target IP | 192.168.6.129 |
| Attacker IP | 192.168.6.128 |
| Tool Used | Gobuster v3.6 |
| Defense | iptables rate limiting |
| Protocol | HTTP on Port 80 |

---

## What is Directory Enumeration?

Directory enumeration is the process of automatically brute-forcing web server paths to discover hidden files, admin panels, backup files, and configuration directories that are not linked from the main page.

Attackers use this during the reconnaissance phase to build a complete picture of the web server's attack surface before choosing an exploitation method.

**In this lab, Gobuster found the following in under 2 minutes with no authentication:**

| Path | Status | Risk |
|------|--------|------|
| /phpMyAdmin | 301 | Database admin panel exposed |
| /phpinfo.php | 200 | Full PHP and system info exposed |
| /test | 301 | Test directory left in production |
| /dav | 301 | WebDAV enabled — file upload possible |
| /twiki | 301 | Wiki application exposed |

Every one of these is a potential entry point for an attacker.

---

## MITRE ATT&CK

| Field | Detail |
|-------|--------|
| Tactic | Reconnaissance |
| Technique | T1595.003 — Active Scanning: Wordlist Scanning |

---

## Phase 1 — Attack (BEFORE Defense)

### Step 1 — Confirm Target is Reachable

```bash
ping 192.168.6.129
```

### Step 2 — Run Directory Enumeration

```bash
gobuster dir -u http://192.168.6.129 \
-w /usr/share/wordlists/dirb/common.txt \
-o ~/Desktop/before_gobuster.txt
```

**Result:** Gobuster completed successfully. 13 paths discovered. Scan ran with 10 threads, zero errors, zero timeouts.

See: `before_gobuster.txt` for full output.

---

## Phase 2 — Defense Implementation

### What is iptables?

iptables is a Linux firewall that controls incoming and outgoing network traffic using rules. In this lab, we use it to rate-limit connections to port 80 — blocking any IP that makes more than 15 new connections in 10 seconds.

This is a real technique used in production environments to slow down or stop automated scanners and brute-force attacks.

### Step 3 — SSH into Target and Apply Rules

```bash
ssh msfadmin@192.168.6.129
```

### Step 4 — Apply iptables Rate Limiting

```bash
sudo iptables -A INPUT -p tcp --dport 80 \
-m conntrack --ctstate NEW \
-m recent --set

sudo iptables -A INPUT -p tcp --dport 80 \
-m conntrack --ctstate NEW \
-m recent --update --seconds 10 --hitcount 15 \
-j DROP
```

### Step 5 — Verify Rules Applied

```bash
sudo iptables -L INPUT -n -v
```

**Result:** Two rules confirmed active on port 80. DROP rule ready to block aggressive scanners.

---

## Phase 3 — Attack Again (AFTER Defense)

### Step 6 — Run the Same Attack With Higher Thread Count

```bash
gobuster dir -u http://192.168.6.129 \
-w /usr/share/wordlists/dirb/common.txt \
-o ~/Desktop/after_gobuster.txt \
-t 50
```

**Result:** Scan failed at 31.70% progress. Hundreds of `context deadline exceeded` errors. Server stopped responding to scanner requests.

### Step 7 — Check DROP Counter

```bash
sudo iptables -L INPUT -n -v
```

**Result: 20,114 packets DROPPED by the firewall rule.**

---

## Before vs After Comparison

| Metric | Before Defense | After Defense |
|--------|---------------|---------------|
| Scan completion | 100% (4615/4615) | 31.70% (1913/4615) |
| Errors | 0 | 200+ timeouts |
| Directories found | 13 | 5 (scan stopped) |
| Packets dropped | 0 | 20,114 |
| Scanner behaviour | Ran freely | Blocked and stalled |

---

## SOC Analyst Notes — What to Monitor

| Detection Point | What to Look For |
|-----------------|------------------|
| Web server logs | Single IP making hundreds of requests in seconds |
| User-Agent string | Contains "gobuster", "nikto", or unusual identifiers |
| Request patterns | Sequential requests for /admin, /test, /phpmyadmin |
| Error rates | High volume of 404 responses from same source IP |
| iptables logs | DROP counter increasing rapidly |

**SOC Response Steps:**
1. Alert fires on high request volume from single IP
2. Investigate source IP in threat intelligence platform
3. Confirm scanner User-Agent in web server logs
4. Check if any 200 responses returned sensitive paths
5. Block IP at firewall if confirmed malicious
6. Hunt for same IP across other systems in environment

---

## Limitations of This Defense

iptables rate limiting is a basic first layer. A skilled attacker can bypass it by:
- Reducing thread count to stay under the threshold
- Distributing the scan across multiple IP addresses
- Spoofing source IP addresses

**Production environments should layer additional controls:**
- Web Application Firewall (WAF) with scanner signature detection
- Fail2ban for automated IP blocking
- SIEM alerting on anomalous request patterns
- ModSecurity with OWASP Core Rule Set

---

## Evidence Files

| File | Contents |
|------|---------|
| before_gobuster.txt | Full scan output before defense |
| after_gobuster.txt | Scan output showing errors after defense |
| screenshots/01-before-scan-running.png | Gobuster running — no defense |
| screenshots/02-before-scan-results.png | Full results — 13 directories found |
| screenshots/03-before-scan-file-output.png | Saved output file |
| screenshots/04-defense-iptables-applied.png | iptables rules + DROP counter proof |
| screenshots/05-after-scan-errors.png | Errors beginning at 31% |
| screenshots/06-after-scan-errors-continued.png | Mass timeouts from blocked scanner |

---

## Key Lesson

Directory enumeration takes under 2 minutes and requires no credentials. Without rate limiting, an attacker can silently map your entire web server before you notice anything unusual.

The iptables rule in this lab dropped 20,114 malicious packets and stopped the scanner at 31% completion — before it could discover the most sensitive paths.

Defense does not have to be complex. A single firewall rule, applied correctly, disrupts automated attacks immediately.

---

*This lab was performed in an isolated environment on systems I own. All activities were conducted for educational purposes only.*