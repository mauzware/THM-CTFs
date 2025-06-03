## Capture! - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/capture)

SecureSolaCoders has once again developed a web application. They were tired of hackers enumerating and exploiting their previous login form. 
They thought a Web Application Firewall (WAF) was too overkill and unnecessary, so they developed their own rate limiter and modified the code slightly.

Before we start, download the required files by pressing the Download Task Files button.

Please wait approximately 3-5 minutes for the application to start.

You can find the web application at: http://MACHINE_IP

**IP: 10.10.250.238**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Bypassing Login Page <br>

**Tools Used**: Nmap, Gobuster<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- As always, I started with Nmap and Gobuster.

  ```
  nmap -sC -sV 10.10.250.238

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 2d:e3:13:58:63:96:7c:2d:7c:a8:ea:4a:3b:9e:5f:01 (RSA)
  |   256 b5:58:d1:80:ad:27:85:9f:b6:ff:35:ac:c5:b5:67:14 (ECDSA)
  |_  256 9c:5b:cd:ca:86:63:11:a3:94:f8:f4:47:50:32:70:4e (ED25519)
  80/tcp open  http    Werkzeug httpd 2.2.2 (Python 3.8.10)
  |_http-server-header: Werkzeug/2.2.2 Python/3.8.10
  | http-title: Site doesn't have a title (text/html; charset=utf-8).
  |_Requested resource was /login
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
  
  gobuster dir -u http://10.10.250.238 -w /usr/share/wordlists/dirb/common.txt 
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /home                 (Status: 302) [Size: 199] [--> /login]
  /login                (Status: 200) [Size: 1942]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Now brute force time.

---

## 🌐 Getting the flag

- Since Hydra took forever to do nothing, I wrote a Python script to bypass the login. When you download the ZIP task file you will have 2 dictionaries.

  ```
  python3 brute.py                                                                          
  [~] Enter the IP address: 10.10.250.238
  [+] Loaded 878 usernames to attempt.
  [+] Username natalie found. Beginning the password cracking   
  [+] Loaded 1567 passwords to attempt.
  [+] Password found: sk8board369                             
  [+] Username: natalie
  [+] Password: sk8board
  ```
  
- Now just login with newly found credentials and get the flag. `7df2eabce36f02ca8ed7f237f77ea416`

---


---

## 🏁 Flags

- **Flag**: `7df2eabce36f02ca8ed7f237f77ea416`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Writing a script that will bypass captcha.`

- What did I learn?
  `Nothing new.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
