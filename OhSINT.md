## OhSINT - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/ohsint)

<i>What information can you possible get with just one image file?<i/>

<i>**Note: This challenge was updated on 2024-02-01. If you are following any older walkthroughs, expect a small change. Additionally, the file is also available on the AttackBox, under the /Rooms/OhSINT directory.**</i>

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: OSINT <br>

**Tools Used**: Exiftool, wigle.net, ImgOps<br>

[ImgOps](https://imgops.com/)<br>
[EXIF](https://exif.tools/)<br>
[wigle.net](https://www.wigle.net/)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 OSINT

- What is this user's avatar of? -> `cat`

  Put the picture in exiftools/ImgOps and find the author ; OWoodflint author ; google him and find his Twitter account

- What city is this person in? -> `London`

  You have his `BSSID: B4:5D:50:AA:86:41` on his Twitter account, paste it in wigle.net ; ImgOps will also provide United Kingdom as his location.

- What is the SSID of the WAP he connected to? -> `UnileverWiFi`

  Zoom in the area in wigle.net and click somewhere approximately near his location.

- What is his personal email address? -> `OWoodflint@gmail.com`

  You can find his GitHub page when you google it, his email is there.

- What site did you find his email address on? -> `GitHub`

- Where has he gone on holiday? -> `New York`

  He has a WordPress blog on his GitHub page, visit the blog.

- What is the person's password? -> `pennYDr0pper.!`

  Ok this one was kinda annoying since it takes some time. You have to inspect the source code of his blog, I found it in this section:

  ```
  <div class="entry-content"></div>
  <p class="has-text-color" style="color:#ffffff"; data-adtags-visited="true">pennYDr0pper.!</p>
  ```

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding his password.`

- What did I learn?
  `Just some basic OSINT stuff and got some practice with wigle.net and exiftool.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
