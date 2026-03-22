TryHackMe — Anthem Writeup
 --Overview
Platform: TryHackMe

Room: Anthem

Difficulty: Easy

Focus Areas: Enumeration, OSINT, Web Exploitation, Privilege Escalation

The Anthem room is a beginner-friendly Windows machine that emphasizes methodical enumeration and OSINT techniques to gather credentials and gain access. 

Step 1: Enumeration
🔹 Nmap Scan
First, I performed an initial scan to identify open ports and services:

nmap -sC -sV -Pn <target-ip>
//It's really important to use -Pn, if not used it won't reveal anything...

Key Findings:

Port 80 (HTTP) → Web server

Port 3389 (RDP) → Remote Desktop access

These services indicate:

A web application for initial enumeration.
Potential remote access via RDP later.

Step 2: Web Enumeration

---Website Analysis
Visiting the web server revealed a blog-based website.
Use Gobuster or FFUF, whichever you prefer.... your style...
Few pages are highlighted - namely authors and robots.txt

(Shortcut - /authors actually contains the flag3, it was the first flag i found)

Key actions:

Browsed pages manually
Checked source code
Looked for hidden data

🔹 robots.txt Discovery
curl http://<target-ip>/robots.txt
👉 Found:

Hidden directories

A possible password string

This is common since robots.txt often exposes sensitive paths or data. 

🔹 CMS Identification
From enumeration:

Identified Umbraco CMS

👉 Important because:

Login panel exists at /umbraco

Suggests possible credential reuse

🧠 OSINT & Information Gathering
🔹 Administrator Identification
Found a poem on the website

Searched it online (OSINT)

Identified the administrator’s name

👉 Key lesson:

Not all enumeration is technical — OSINT plays a major role.

🔹 Email Pattern Discovery

Found a user email format on the site

Derived admin email using same pattern

Example logic:

firstname initial + lastname @ domain
--- SG@anthem.com
🚩 Flag Discovery
Flags were hidden in:

Page source

Metadata

Author pages

🔍 Method:
Inspect element

Search for THM{} patterns

👉 This reinforces:

Always check source code during web enumeration

💻 Initial Access
Using gathered credentials:

Username → derived from OSINT

Password → found in robots.txt

Connected via RDP:


Privilege Escalation --
Use this command - rdesktop -u SG -p UmbracoIsTheBest! <IP>
(it will get you remote access to the machine)
After gaining access:

🔹 Steps:
Explored filesystem

Enabled hidden files

Located backup/restore file

Added the current user to the group which can access the backup files through Properties->Security->Add

🔹 Key Issue:
Misconfigured file permissions

👉 Modified permissions → accessed sensitive file
👉 Retrieved administrator credentials
Go to Administrator->Desktop->root.txt
(AND BOOM!! the final flag is there)
🏁 Key Takeaways
Enumeration is the most critical phase

OSINT can reveal:

usernames

patterns

Always check:

robots.txt

page source

Misconfigurations (permissions) are common escalation paths

💡 Final Thoughts
This room demonstrates that:

You don’t need advanced exploits — careful enumeration + logic is enough

It’s a strong beginner lab for building:

1. Web enumeration skills

2. OSINT mindset

2. Real-world attack flow understanding
