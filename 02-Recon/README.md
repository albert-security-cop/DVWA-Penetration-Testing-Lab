# Phase 2 — Reconnaissance

## Objective
Identify open ports, running services, software versions, and web application vulnerabilities on the target machine before any exploitation attempts.

---

## Target Information

| Field | Value |
|---|---|
| IP Address | 192.168.100.129 |
| OS | Ubuntu Server 22.04 |
| Web Server | Apache 2.4.66 |
| Application | DVWA |

---

## 1. Nmap Basic Scan

**Command:**
```bash
nmap 192.168.100.129
```

**Results:**

PORT   STATE SERVICE

22/tcp open  ssh

80/tcp open  http

**Findings:**
- 2 open ports discovered
- Port 22 running SSH
- Port 80 running HTTP

---

## 2. Nmap Service Version Scan

**Command:**
```bash
nmap -sV 192.168.100.129
```

**Results:**

PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu

80/tcp open  http    Apache httpd 2.4.66 (Ubuntu)

**Findings:**
- SSH version identified — OpenSSH 10.2p1
- Apache version identified — 2.4.66
- OS confirmed — Ubuntu Linux

---

## 3. Nmap Aggressive Scan

**Command:**
```bash
nmap -A 192.168.100.129
```

**Findings:**
- HTTP title reveals Apache default page
- Network distance — 1 hop (direct connection)
- OS fingerprinting confirmed Linux kernel

---

## 4. Nmap Full Port Scan

**Command:**
```bash
nmap -p- 192.168.100.129
```

**Findings:**
- Only 2 ports open across all 65535 ports
- Attack surface is minimal — SSH and HTTP only

---

## 5. Nikto Web Vulnerability Scan

**Command:**
```bash
nikto -h http://192.168.100.129
```

**Findings:**

| Vulnerability | Severity | Details |
|---|---|---|
| ETag Inode Leakage | Medium | CVE-2003-1418, server leaks filesystem info via ETags |
| Missing Content-Security-Policy | Medium | No CSP header configured |
| Missing Strict-Transport-Security | Medium | No HSTS header configured |
| Missing Permissions-Policy | Low | No Permissions-Policy header |
| Missing Referrer-Policy | Low | No Referrer-Policy header |
| Missing X-Content-Type-Options | Low | Content type sniffing possible |
| Deprecated X-Frame-Options | Low | Should be replaced with CSP frame-ancestors |
| HTTP Methods Exposed | Info | GET, POST, OPTIONS, HEAD allowed |

---

## Summary

| Scan | Findings |
|---|---|
| Open Ports | 2 (SSH, HTTP) |
| Services Identified | OpenSSH 10.2p1, Apache 2.4.66 |
| Web Vulnerabilities | 8 findings |
| Attack Surface | Port 80 — DVWA web application |

---

## Screenshots

| # | Description |
|---|---|
| 01 | Nmap basic scan results |
| 02 | Nmap service version scan (-sV) |
| 03 | Nmap aggressive scan (-A) |
| 04 | Nmap full port scan (-p-) |
| 05 | Nikto scan results |

---

## Next Phase
[Phase 3 — Exploitation](../03-Exploitation/)
