## Corridor - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/corridor)

<i>You have found yourself in a strange corridor. Can you find your way back to where you came?

In this challenge, you will explore potential IDOR vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a hash, don't they?). 
This could help you uncover website locations you were not expected to access.</i>

**IP: 10.10.174.161**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: IDOR <br>

**Tools Used**: Crackstation<br>

[Crackstation](https://crackstation.net/)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 IDOR Exploitation

- When you visit the web app, click on different rooms and they will point you to different endpoints. Take each of thos endpoints and crack those hashes.

  ```
  Hash					Type	Result
  eccbc87e4b5ce2fe28308fd9f2a7baf3	md5	3
  e4da3b7fbbce2345d7772b0674a318d5	md5	5
  d3d9446802a44259755d38e6d163e820	md5	10
  c9f0f895fb98ab9159f51fd0297e236d	md5	8
  c81e728d9d4c2f636f067f89cc14862c	md5	2
  c51ce410c124a10e0db5e4b97fc2af39	md5	13
  c4ca4238a0b923820dcc509a6f75849b	md5	1
  c20ad4d76fe97759aa27a0c99bff6710	md5	12
  a87ff679a2f3e71d9181a67b7542122c	md5	4
  8f14e45fceea167a5a36dedd4bea2543	md5	7
  6512bd43d9caa6e02c990b0a82652dca	md5	11
  45c48cce2e2d7fbdea1afc51c7c6ad26	md5	9
  1679091c5a880faf6fb5e6087eb1b2dc	md5	6
  ```
  
- Since each hash is a hashed number, take number 0, hash it to MD5 and add it to url and you'll get the flag.

  ```
  0 -> cfcd208495d565ef66e7dff9f98764da -> MD5

  http://10.10.174.161/cfcd208495d565ef66e7dff9f98764da
  flag{2477ef02448ad9156661ac40a6b8862e}
  ```

---

## 🏁 Flags

- **Flag**: `flag{2477ef02448ad9156661ac40a6b8862e}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `Nothing new.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
