# TryHackMe — Easy Peasy Writeup

## Overview

**Room:** Easy Peasy
**Difficulty:** Easy
**Focus Areas:** Enumeration, Web Exploitation, Hash Cracking, Steganography, Privilege Escalation

This room focuses on strong enumeration and chaining multiple techniques together to progress.

---

## Step 1: Enumeration

### Nmap Scan

```bash
nmap -sC -sV -p- <TARGET_IP>
```

### Key Findings:

* Port 80 → HTTP (Nginx)
* Port 6498 → SSH
* Port 65524 → HTTP (Apache)

Always scan all ports (`-p-`) to avoid missing services.

---

## Step 2: Web Enumeration

### Directory Brute Force

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/seclists/.../common.txt
```

### Findings:

* /hidden
* /hidden/whatever

Inspecting the source code revealed an encoded string.

### Decoding

* Encoding: Base62
* Tool: CyberChef
* Result: First flag

---

## Step 3: Exploring Port 65524

Navigate to:

```
http://<TARGET_IP>:65524
```

### Findings:

* Default Apache page
* Hidden data in source code
* robots.txt contains a hash

### Action:

* Identify hash type (MD5)
* Crack using online tools or local utilities

Result: Second flag

---

## Step 4: Hidden Directory Discovery

From decoded hints, a new directory is found:

```
/n0th1ng3ls3m4tt3r
```

Contents:

* Hash
* Image file (possible steganography)

---

## Step 5: Hash Cracking

### Identify Hash

```bash
hash-identifier
```

### Crack Hash

```bash
john --wordlist=easypeasy.txt hash.txt
```

Password recovered from wordlist.

---

## Step 6: Steganography

Extract hidden data from image:

```bash
steghide extract -sf image.jpg
```

Use the cracked password to retrieve hidden content.

---

## Step 7: SSH Access

```bash
ssh user@<TARGET_IP> -p 6498
```

Login successful. User flag obtained.

---

## Step 8: Privilege Escalation

### Enumeration

```bash
./linpeas.sh
```

### Finding:

* Misconfigured cron job

Exploit the cron job to execute commands as root.

Root flag obtained.

---

## Key Takeaways

* Enumeration is critical
* Always check source code and hidden files
* Use CyberChef for quick decoding
* Combine multiple techniques (hash cracking + steganography)
* Privilege escalation often involves misconfigurations

---

## Tools Used

* Nmap
* Gobuster
* CyberChef
* John the Ripper
* Hash-Identifier
* Steghide
* LinPEAS

---

## Conclusion

This room reinforces the importance of methodical enumeration and chaining techniques effectively during penetration testing.
