# Task-5
# Capstone Project & Incident Response

## Overview
The primary objective of this project is to execute a full web application penetration test — from reconnaissance through exploitation and evidence collection — against DVWA (Damn Vulnerable Web Application) in a controlled lab environment, followed by an incident response simulation to detect, contain, and eradicate the simulated attack, and a post-incident report.

---

## Technical Stack & Tools Used
* **Operating System (Attacker):** Kali Linux
* **Target Application:** DVWA (Damn Vulnerable Web Application), hosted via Docker
* **Network Scanning:** Nmap, Nikto, Gobuster
* **Log Analysis:** Apache access logs

---

## Exploitation Scenarios & Remediation

### 1. Penetration Testing Methodology
* **Approach:** Followed a standard web app pentest workflow — Recon → Scanning → Exploitation → Evidence Collection → Incident Response → Reporting — documenting each phase with command output and screenshots.
* **Recon:** Identified the live service and web server details on the target using Nmap.
  ```bash
  nmap -sV -p 80 192.168.56.102
  ```
* **Finding:** Port 80 running Apache, hosting DVWA (PHP/MySQL), security level set to Low for baseline testing.
* **Directory & Content Discovery:**
  ```bash
  nikto -h http://192.168.56.102
  gobuster dir -u http://192.168.56.102 -w /usr/share/wordlists/dirb/common.txt
  ```

### 2. SQL Injection
* **Exploitation:** Bypassed authentication and extracted database content via the DVWA SQL Injection module.
  ```
  ' OR '1'='1
  ```
  ```bash
  sqlmap -u "http://192.168.56.102/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=<session>; security=low" --dbs
  ```
* **Result:** Enumerated database names, tables, and extracted user credentials from the `users` table.
* **Mitigation:** Use parameterized queries / prepared statements; apply input validation and least-privilege database accounts.

### 3. Cross-Site Scripting (Stored XSS)
* **Exploitation:** Injected a persistent script into the DVWA Guestbook (Stored XSS) module.
  ```html
  <script>alert(document.cookie)</script>
  ```
* **Result:** Payload executed on every page reload, demonstrating session-hijacking potential via cookie theft.
* **Mitigation:** Sanitize and encode user input/output (HTML entity encoding); apply a Content Security Policy (CSP).

### 4. Command Injection
* **Exploitation:** Injected OS commands through the DVWA ping utility.
  ```bash
  127.0.0.1; whoami
  127.0.0.1; cat /etc/passwd
  ```
* **Result:** Achieved arbitrary command execution on the underlying server via unsanitized shell input.
* **Mitigation:** Avoid passing user input to shell commands; use safe APIs/library calls, and strict input whitelisting.

### 5. Unrestricted File Upload
* **Exploitation:** Uploaded a malicious PHP web shell through the DVWA File Upload module and accessed it directly via the browser.
  ```bash
  curl http://192.168.56.102/hackable/uploads/shell.php?cmd=id
  ```
* **Result:** Achieved remote command execution via the uploaded web shell.
* **Mitigation:** Validate file type/extension and MIME type server-side; store uploads outside the web root; disable script execution in the upload directory.

### 6. Incident Response Simulation
* **Detection:** Analyzed Apache access logs to identify indicators of the simulated attack (SQLmap user-agent, repeated login POST requests, `UNION SELECT` query strings).
  ```bash
  grep -i "sqlmap\|UNION SELECT" /var/log/apache2/access.log
  ```
* **Containment:**
  ```bash
  sudo ufw deny from <attacker-ip>
  ```
* **Eradication:**
  ```bash
  rm /var/www/html/dvwa/hackable/uploads/shell.php
  ```
* **Post-Incident Report:** Documented the attack timeline, root cause, actions taken, and lessons learned.
* **Mitigation:** Deploy centralized logging/alerting, enforce `fail2ban` for repeated auth failures, and schedule regular vulnerability rescans.

---
