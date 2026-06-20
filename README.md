# DVWA Penetration Testing Lab

A full penetration test conducted against DVWA (Damn Vulnerable Web Application) in an isolated virtual lab environment. This project demonstrates practical offensive security skills including reconnaissance, exploitation of common web vulnerabilities, and professional reporting.

---

## Lab Environment

| Machine | OS | Role | IP |
|---|---|---|---|
| DVWA-Server | Ubuntu Server 22.04 | Target | 192.168.100.129 |
| Kali-Attacker | Kali Linux | Attacker | 192.168.100.x |

**Virtualization:** VMware Workstation  
**Network:** Host-Only (isolated lab)

---

## Tools Used

- **Nmap** — Port scanning and service enumeration
- **Nikto** — Web vulnerability scanning
- **Hydra** — Brute force attacks
- **SQLMap** — Automated SQL injection
- **Burp Suite** — Web application proxy
- **Metasploit** — Exploitation framework

---

## Vulnerabilities Exploited

| # | Vulnerability | Severity | Status |
|---|---|---|---|
| 1 | Brute Force | High | ✅ Completed |
| 2 | Command Injection | Critical | ✅ Completed |
| 3 | SQL Injection | Critical | 🔄 In Progress |
| 4 | File Inclusion (LFI) | High | ⏳ Pending |
| 5 | File Upload | Critical | ⏳ Pending |
| 6 | XSS Reflected | Medium | ⏳ Pending |
| 7 | XSS Stored | Medium | ⏳ Pending |
| 8 | CSRF | Medium | ⏳ Pending |

---

## Project Structure
DVWA-Penetration-Testing-Lab/

├── 01-Setup/

│   └── screenshots/

├── 02-Recon/

│   └── screenshots/

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

---

## Methodology

This project follows the standard penetration testing methodology:

1. **Reconnaissance** — Passive and active information gathering
2. **Scanning** — Port scanning, service enumeration, vulnerability scanning
3. **Exploitation** — Exploiting identified vulnerabilities
4. **Post Exploitation** — Privilege escalation, maintaining access
5. **Reporting** — Documenting findings with remediation recommendations

---

## Disclaimer

This project was conducted in an isolated lab environment for educational purposes only. All attacks were performed against intentionally vulnerable software. Never attempt these techniques against systems you do not own or have explicit permission to test.

---

## Author

**AHMAD ZHAIRAH**  
Cybersecurity Student  
[LinkedIn](https://www.linkedin.com/in/ahmad-zhairah-670089417/) | [GitHub](https://github.com/albert-security-cop/DVWA-Penetration-Testing-Lab)
