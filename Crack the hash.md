## Crak the hash - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/crackthehash)

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Crypto <br>

**Tools Used**: John The Ripper, Hashcat, Hash Identifier, CyberChef, Crackstation<br>
[CyberChef](https://gchq.github.io/CyberChef/)<br>
[Crackstation](https://crackstation.net/)<br>
[Hash Identifier](https://hashes.com/en/tools/hash_identifier)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## ⚙️ Level 1

<i>Can you complete the level 1 tasks by cracking the hashes?</i>

<i>**For all hashes I used rockyou.txt as a wordlist**.</i>

- 48bb6e862e54f2a795ffc4e541caed4d -> MD5 hash -> `easy`
  
  ```
  john --wordlist=/rockyou.txt --format=Raw-MD5 hash1.txt
  ```

- CBFDAC6008F9CAB4083784CBD1874F76618D2A97 -> SHA1 hash -> `password123`
  
  ```
  john --wordlist=/rockyou.txt --format=Raw-SHA1 hash2.txt
  ```
  
- 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032 -> SHA256 hash -> `letmein`
  
  ```
  john --wordlist=/rockyou.txt --format=Raw-SHA256 hash2.txt
  ```

- $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom -> Blowfish hash-> `bleh`

  ```
  hashcat -m 3200 hash4.txt /rockyou.txt
  ```

- 279412f945939ba78ce0758d3fd83daa -> MD4 hash -> `Eternity22` ; I cracked this one in Crackstation just for funzies, just paste it there and thats it.
  
---

## ⚙️ Level 2

<i>This task increases the difficulty. All of the answers will be in the classic rock you password list.<i>

<i>You might have to start using hashcat here and not online tools. It might also be handy to look at some example hashes on [hashcats page](https://hashcat.net/wiki/doku.php?id=example_hashes).<i>

<i>Also keep in mind that some hashes will take some time to crack...</i>

- 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032 -> SHA256 hash -> `letmein`

  ```
  john --wordlist=/rockyou.txt --format=Raw-SHA256 hash6.txt
  ```

- 1DFECA0C002AE40B8619ECF94819CC1B -> NTLM hash -> `n63umy8lkf4i` ; Also cracked this one on Crackstation.
  
- $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02. -> Salt: aReallyHardSalt ; sha512crypt $6$, SHA512 (Unix) -> `waka99`

  ```
  hashcat -m 1800 hash8.txt /rockyou.txt
  ```

- e5d8870e5bdd26602cab8dbe07a942c8669e56d6 -> Salt: tryhackme ; HMAC SHA1 hash -> `481616481616`

  ```
  hashcat -m 110 hash9.txt /rockyou.txt
  ```

---

## 💬 Notes & Reflections

- What was challenging in this room for me? <br>
  `Nothing much, probably waiting for cracking to finish...`
  
- What did I learn?<br>
  `Got some practice with cracking and thats it.`
  
- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!
  
- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
