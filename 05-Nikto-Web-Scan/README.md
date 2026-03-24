# Attack 05: Nikto Web Vulnerability Scanner

**Tool:** Nikto | **Target:** Metasploitable 2 | **Difficulty:** Beginner

---

## What is Nikto?

Nikto is an open-source web server scanner that checks for over 6,700 potentially dangerous files, outdated server software, and configuration issues. It is one of the first tools attackers use when they discover a web server.

Think of it as an automated attacker that knocks on every door of your web server to see which ones are unlocked.

---

## What is Web Vulnerability Scanning?

Web vulnerability scanning is the process of automatically checking a web server for known security issues. The scanner sends thousands of requests looking for:

- Outdated software versions with known exploits
- Exposed admin panels like phpMyAdmin
- Test files left behind by developers
- Dangerous configurations
- Backup files containing sensitive data

The scary reality — an attacker can map your entire web attack surface in under 5 minutes without ever logging in.

---

## What is Nikto Used For?

Nikto automates the reconnaissance phase of a web attack. Instead of manually checking for vulnerable files one by one, Nikto does it automatically and generates a complete report of everything that could be exploited.

---

## Lab Environment

| Role | Details |
|------|---------|
| Attacker Machine | Kali Linux |
| Target Machine | Metasploitable 2 |
| Target IP | 192.168.6.129 |
| Tool Used | Nikto v2.5.0 |
| Protocol Attacked | HTTP on Port 80 |

---

## Vulnerability Identified

The target was running Apache 2.2.8 which was released in 2008. This version has 31 known CVEs (publicly documented vulnerabilities). The server also had:

- phpMyAdmin exposed to anyone
- Dangerous HTTP methods (PUT, TRACE) enabled
- Test directories left in production
- CGI scripts accessible for remote code execution

---

## Attack Walkthrough

### Step 1 — Reconnaissance

First confirmed the target is reachable:
```
ping 192.168.6.129
```

Result showed target online and responding.

### Step 2 — Basic Server Scan

Ran initial Nikto scan against the target:
```
nikto -h http://192.168.6.129 -o nikto_results.txt -Format txt
```

Nikto began sending thousands of requests checking for:
- Server version information
- Dangerous files and directories
- Outdated software
- Configuration issues

### Step 3 — Web Application Scan

Ran a comprehensive scan targeting web application vulnerabilities:
```
nikto -h http://192.168.6.129 -o web_application_results.txt -Format txt
```

This scan revealed additional findings specific to CGI scripts and default application files.

### Step 4 — Reviewing Findings

The scan results showed the complete attack surface:

- **Apache 2.2.8** — 31 known CVEs available for exploitation
- **phpMyAdmin exposed** — Database management interface accessible to anyone
- **HTTP PUT method enabled** — Attacker can upload malicious files
- **HTTP TRACE method enabled** — Enables cross-site scripting attacks
- **/test/ directory** — Test files left in production
- **/cgi-bin/test.cgi** — CGI script accessible for remote code execution

---

## What the Nikto Output Looks Like

```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.6.129
+ Target Hostname:    192.168.6.129
+ Target Port:        80
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) DAV/2
+ /: Apache/2.2.8 appears to be outdated (current is at least Apache/2.4.54). 31 CVEs found.
+ /phpMyAdmin/: phpMyAdmin directory found.
+ /test/: Test directory found.
+ /cgi-bin/test.cgi: CGI test script found.
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, TRACE ...
+ /: TRACE method is enabled. Could allow XSS attacks.
+ 7909 requests: 0 error(s) - 127 item(s) reported
```

Each line is a potential entry point for an attacker.

---

## Red Team Notes — Attacker Mindset

As an attacker, Nikto is your reconnaissance weapon. Here is what goes through an attacker's mind during this scan:

| Finding | What Attacker Thinks |
|---------|---------------------|
| Apache 2.2.8 | 31 public exploits available. Let me check Exploit-DB |
| phpMyAdmin exposed | Default credentials? Can I get database access? |
| PUT method enabled | I can upload a web shell directly |
| TRACE method enabled | XSS attacks possible |
| /test/ directory | Maybe developers left credentials here |
| /cgi-bin/test.cgi | CGI often leads to remote code execution |

**Why Nikto is dangerous:** An attacker can map your entire web infrastructure and identify multiple attack vectors in under 5 minutes. Every finding in the Nikto report is a potential breach waiting to happen.

---

## SOC Analyst Notes — Defender Mindset

As a SOC analyst, your job is to detect this scanning activity before it escalates to exploitation. Here is what you should be monitoring:

| Detection Point | What to Look For |
|-----------------|------------------|
| Web server logs | Single IP making hundreds of requests in seconds |
| User-Agent string | Contains "Nikto" or unusual browser identifiers |
| Request patterns | Sequential requests for /admin, /test, /phpmyadmin, /backup |
| HTTP methods | OPTIONS, TRACE, PUT requests from same source |
| Error rates | High volume of 404 and 403 responses |
| Time patterns | Scanning activity during off-hours |

**What a SOC analyst does with this detection:**

1. **Immediate triage** — Is this a legitimate security scan or an attacker?
2. **Source IP investigation** — Check threat intelligence for reputation
3. **Correlation** — Has this IP been seen elsewhere in the environment?
4. **Containment** — Block IP at firewall if confirmed malicious
5. **Hunting** — Search for other systems this IP may have scanned

---

## MITRE ATT&CK

| Field | Detail |
|-------|--------|
| Tactic | Reconnaissance |
| Technique | T1595.002 — Active Scanning: Vulnerability Scanning |

---

## How SOC Analysts Detect This

- More than 100 requests from single IP to web server in under 60 seconds
- User-Agent header containing "Nikto" or unusual values
- Requests for phpMyAdmin, test directories, or backup files
- OPTIONS or TRACE HTTP methods being used
- High volume of 404 errors from same source IP

---


---

## How to Defend Against This

| Defence | What it Does |
|---------|-------------|
| Deploy a WAF | Blocks scanner signatures and rate-limits requests |
| Remove test files | Delete /test, /doc, /phpmyadmin from production |
| Disable dangerous HTTP methods | Turn off PUT, DELETE, TRACE in web config |
| Patch management | Update Apache to latest version regularly |
| Server header hardening | Remove version from Server: header |
| Rate limiting | Block IPs making more than 50 requests per minute |
| IDS/IPS rules | Deploy Snort/Suricata with scanner detection rules |

---

## Evidence Files

| File | What it Contains |
|------|-----------------|
| nikto_results.txt | Full output of basic server scan |
| web_application_results.txt | Full output of web application scan |
| screenshots/ | Visual proof of every step of the attack |

---

## Key Lesson

Nikto reveals what attackers see when they first discover your web server. In under 5 minutes, it can identify outdated software, exposed admin panels, dangerous configurations, and multiple exploitation paths.

As a SOC analyst, when you see a single IP making hundreds of web requests for phpMyAdmin and test directories — you are not looking at routine traffic. You are watching the first step of a potential breach. The difference between a successful attack and a prevented one is detection speed.

The fix is simple — remove test files, patch software, disable dangerous HTTP methods, and deploy a WAF. Yet thousands of servers remain vulnerable because basic hardening is skipped.

---

*This lab was performed in an isolated environment on systems I own. All activities were conducted for educational purposes only.*