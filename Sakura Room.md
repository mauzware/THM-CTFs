## Sakura Room - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/sakura)

Background
This room is designed to test a wide variety of different OSINT techniques. With a bit of research, most beginner OSINT practitioners should be able to complete these challenges. 
This room will take you through a sample OSINT investigation in which you will be asked to identify a number of identifiers and other pieces of information in order to help catch a cybercriminal. 
Each section will include some pretext to help guide you in the right direction, as well as one or more questions that need to be answered in order to continue on with the investigation. 
Although all of the flags are staged, this room was created using working knowledge from having led and assisted in OSINT investigations both in the public and private sector. 

NOTE: All answers can be obtained via passive OSINT techniques, DO NOT attempt any active techniques such as reaching out to account owners, password resets, etc to solve these challenges.

If you have any other questions, comments, or suggestions, please reach out to us at @OSINTDojo on Twitter.

Instructions
Ready to get started? Type in "Let's Go!" in the answer box below to continue.

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: OSINT <br>

**Tools Used**: Exiftool, Google Reverse Image Search <br>

[Exiftools](https://exif.tools/)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 TIP-OFF

- What username does the attacker go by? `SakuraSnowAngelAiko`

  Put the image in Exiftools

  ```
  Docname	pwnedletter.svg
  Export-filename	/home/SakuraSnowAngelAiko/Desktop/pwnedletter.png
  Export-xdpi	96
  Export-ydpi	96
  ```

---

## 🔍 RECONNAISSANCE

- What is the full email address used by the attacker? `SakuraSnowAngel83@protonmail.com`

- What is the attacker's full real name? `Aiko Abe`

  ```
  You can find both flags on attacker Github and Twitter or LinkedIn
  Use https://whatsmyname.app/ to find attackers Github
  ```

---

## 🔍 UNVEIL

- What cryptocurrency does the attacker own a cryptocurrency wallet for? `Ethereum`

- What is the attacker's cryptocurrency wallet address? `0xa102397dbeeBeFD8cD2F73A89122fCdB53abB6ef`

- What mining pool did the attacker receive payments from on January 23, 2021 UTC? `Ethermine`

- What other cryptocurrency did the attacker exchange with using their cryptocurrency wallet? `Theter`

  ```
  First two flags can be found on attackers Github
  For last two flags use blockhain explorer and input attackers wallet address
  ```

---

## 🔍 TAUNT

- What is the attacker's current Twitter handle? `SakuraLoverAiko`

- What is the BSSID for the attacker's Home WiFi? `84:AF:EC:34:FC:F8`

  ```
  First flag is pretty straightforward, for second flag you'll have to use Tor Browser.
  ```

---

## 🔍 HOMEBOUND

- What airport is closest to the location the attacker shared a photo from prior to getting on their flight? `DCA`

- What airport did the attacker have their last layover in? `HND`

- What lake can be seen in the map shared by the attacker as they were on their final flight home? `Lake Inawashiro`

- What city does the attacker likely consider "home"? `Hirosaki`

  ```
  Use images from attacker's Twitter in order to finish this CTF
  https://wigle.net for the last flag
  ```
  
---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Learning more OSINT lol.`

- What did I learn?
  `More OSINT!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
