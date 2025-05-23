## Juicy Details - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/juicydetails)

<i>﻿Introduction

You were hired as a SOC Analyst for one of the biggest Juice Shops in the world and an attacker has made their way into your network. 

Your tasks are:

Figure out what techniques and tools the attacker used
What endpoints were vulnerable
What sensitive data was accessed and stolen from the environment
An IT team has sent you a zip file containing logs from the server. Download the attached file, type in "I am ready!" and get to work! There's no time to lose!</i>

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Log Analysis <br>

**Tools Used**: Any text editor<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Reconnaissance

- What tools did the attacker use? (Order by the occurrence in the log)? `nmap, hydra, sqlmap, curl, feroxbuster`

- What endpoint was vulnerable to a brute-force attack? `/rest/user/login`

  ```::ffff:192.168.10.5 - - [11/Apr/2021:09:16:30 +0000] "POST /rest/user/login HTTP/1.0" 401 26 "-" "Mozilla/5.0 (Hydra)"```

- What endpoint was vulnerable to SQL injection? `/rest/products/search`

- What parameter was used for the SQL injection? `q`

  ```
  ::ffff:192.168.10.5 - - [11/Apr/2021:09:29:16 +0000] "GET /rest/products/search?q=1%27%29%20AND%205895%3DCAST%28%28CHR%281[SNIP!]0AND%20%28%27SWvi%27%3D%27SWvi HTTP/1.1" 500 - "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"
  ```
  
- What endpoint did the attacker try to use to retrieve files? (Include the /) `/ftp`

  ```::ffff:192.168.10.5 - - [11/Apr/2021:09:34:33 +0000] "GET /ftp HTTP/1.1" 200 4852 "-" "feroxbuster/2.2.1"```

---

## 📁 Stolen Data

- What section of the website did the attacker use to scrape user email addresses? `products, reviews`

  ```::ffff:192.168.10.5 - - [11/Apr/2021:09:11:42 +0000] "GET /rest/products/20/reviews HTTP/1.1" 200 405 "http://192.168.10.4/" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"```

- Was their brute-force attack successful? If so, what is the timestamp of the successful login? (Yay/Nay, 11/Apr/2021:09:xx:xx +0000) `11/Apr/2021:09:16:31 +0000`

  ```::ffff:192.168.10.5 - - [11/Apr/2021:09:16:31 +0000] "POST /rest/user/login HTTP/1.0" 200 831 "-" "Mozilla/5.0 (Hydra)"```

- What user information was the attacker able to retrieve from the endpoint vulnerable to SQL injection? `email, password`

  ```::ffff:192.168.10.5 - - [11/Apr/2021:09:31:04 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 [SNIP!]```

- What files did they try to download from the vulnerable endpoint? (endpoint from the previous task, question #5) `coupons_2013.md.bak, www-data.bak`

  ```
  Sun Apr 11 09:35:45 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/www-data.bak", 2602 bytes, 544.81Kbyte/sec
  Sun Apr 11 09:36:08 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/coupons_2013.md.bak", 131 bytes, 3.01Kbyte/sec
  ```

- What service and account name were used to retrieve files from the previous question? (service, username) `ftp, anonymous`

  ```Sun Apr 11 08:29:34 2021 [pid 6846] [ftp] OK LOGIN: Client "::ffff:192.168.10.5", anon password "IEUser@"```

- What service and username were used to gain shell access to the server? (service, username) `ssh, www-data`

  ```Apr 11 09:41:19 thunt sshd[8260]: Accepted password for www-data from 192.168.10.5 port 40112 ssh2```

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `Nothing new, just practicing log analysis I guess.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
