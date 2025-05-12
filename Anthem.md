## Anthem - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/anthem)

<i>This task involves you, paying attention to details and finding the 'keys to the castle'.

This room is designed for beginners, however, everyone is welcomed to try it out!

Enjoy the Anthem.

In this room, you don't need to brute force any login page. Just your preferred browser and Remote Desktop.

Please give the box up to 5 minutes to boot and configure.</i>

**IP: 10.10.8.217**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Windows, Web <br>

**Tools Used**: Nmap, Gobuster<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Webiste Analysis

- Starting with Nmap and Gobuster.

  ```
  nmap -Pn -sC -sV -T5 -p- 10.10.8.217

  PORT     STATE SERVICE       VERSION
  80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  | http-robots.txt: 4 disallowed entries 
  |_/bin/ /config/ /umbraco/ /umbraco_client/
  |_http-title: Anthem.com - Welcome to our blog
  3389/tcp open  ms-wbt-server Microsoft Terminal Services
  | rdp-ntlm-info: 
  |   Target_Name: WIN-LU09299160F
  |   NetBIOS_Domain_Name: WIN-LU09299160F
  |   NetBIOS_Computer_Name: WIN-LU09299160F
  |   DNS_Domain_Name: WIN-LU09299160F
  |   DNS_Computer_Name: WIN-LU09299160F
  |   Product_Version: 10.0.17763
  |_  System_Time: 2025-05-11T19:12:30+00:00
  |_ssl-date: 2025-05-11T19:12:35+00:00; +3s from scanner time.
  | ssl-cert: Subject: commonName=WIN-LU09299160F
  | Not valid before: 2025-05-10T19:03:56
  |_Not valid after:  2025-11-09T19:03:56
  Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
  ```

  ```
  gobuster dir -u http://10.10.8.217/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /archive              (Status: 301) [Size: 118] [--> /]
  /Archive              (Status: 301) [Size: 118] [--> /]
  /authors              (Status: 200) [Size: 4065]
  /blog                 (Status: 200) [Size: 5389]
  /Blog                 (Status: 200) [Size: 5389]
  /categories           (Status: 200) [Size: 3536]
  /install              (Status: 302) [Size: 126] [--> /umbraco/]
  /robots.txt           (Status: 200) [Size: 192]
  /RSS                  (Status: 200) [Size: 1869]
  /rss                  (Status: 200) [Size: 1869]
  /search               (Status: 200) [Size: 3464]
  /Search               (Status: 200) [Size: 3464]
  /sitemap              (Status: 200) [Size: 1035]
  /SiteMap              (Status: 200) [Size: 1035]
  /tags                 (Status: 200) [Size: 3589]
  /umbraco              (Status: 200) [Size: 4078]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- You'll see bunch of errors with Gobuster, but don't worry about them. In `/robots.txt` you will find a password and CMS for website. Domain name is written at the bottom of main page.

  ```
  UmbracoIsTheBest! 

  # Use for all search robots
  User-agent: *
  
  # Define the directories not to crawl
  Disallow: /bin/
  Disallow: /config/
  Disallow: /umbraco/
  Disallow: /umbraco_client/
  ```
  
- OK, so this one took a while, I spent a lot of time googling different things, but in order to get the answers for last two questions you have to google the poem written by one of the users...

  ```
  Born on a Monday,
  Christened on Tuesday,
  Married on Wednesday,
  Took ill on Thursday,
  Grew worse on Friday,
  Died on Saturday,
  Buried on Sunday.
  That was the end… 
  ```

- Google will give you the author name and for the email just follow the pattern of Jane Doe email.

---

## 🌐 Spot the flags

- So here, I actaully finished this task before the first one lol. All the flags are in the source codes of different pages. I found them naturally during exploration.

  ```
  http://10.10.8.217 source code -> THM{G!T_G00D}

  http://10.10.8.217/archive/we-are-hiring/ source code -> THM{L0L_WH0_US3S_M3T4}
  
  http://10.10.8.217/archive/a-cheers-to-our-it-department/ source code -> THM{AN0TH3R_M3TA}
  
  http://10.10.8.217/authors/jane-doe/ source code -> THM{L0L_WH0_D15}
  ```
  
- Let's finish the job.

---

## ⚙️ Final stage

- I got the right credentials for username on my 5th or 6th try. Just use the administrators credentials and the password that we found. I won't write the credentials here, I'll let you think a bit about them. It is a really common combination for the usernames.
  
- I used `rdesktop` to login but you can use `xfreerdp3` or `Remmina` depending on your preferance. User flag is on the desktop. `THM{N00T_NO0T}`
  
- So for the root flag, I allowed hidden files to be visible and found folder `backup` with `restore.txt` inside. Unfortunately, we can't read the content inside.

- Here, I spent some time trying various different commands in PowerShell in order to get access to the content, but nothing worked. What worked for me is changing the Security Properties of `restore.txt`.
  I honestly don't know why `icacls` didn't work, it even gave me the output `C:\backup\restore.txt Successfully processed 1 files; Failed processing 0 files`.

  ```
  Right Click on the file
  Properties
  Security
  Add
  then add your user and allow Read, Write commands
  i allowed everything just to be safe
  ```

- From this point it's a tutorial, take the Administrator password and login as Administrator. Root flag is waiting for you on Desktop.

---

## 🏁 Flags

- **User Flag**: `THM{N00T_NO0T}`
- **Root Flag**: `THM{Y0U_4R3_1337}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Honestly, I thought that it will be much harder, it took a little bit of googling to escalate but that's it.`

- What did I learn?
  `New way of privelege escalation on Windows.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
