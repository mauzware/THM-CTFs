## Flip - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/flip)

Log in as the admin and capture the flag!

If you can...

Whenever you are ready, click on the Start Machine button to fire up the Virtual Machine. Please allow 3-5 minutes for the VM to fully start.

The server is listening on port 1337 via TCP. You can connect to it using Netcat or any other tool you prefer.

**IP: 10.10.20.11**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Python, Cryptography <br>

**Tools Used**: /<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Flippin

- In the source code you will find credentials `access_username=admin&password=sUp3rPaSs1`. Don't even try to use these credentials since it's not that easy.

- After exploring a bit through theory and documentation, the whole point of this challenge is to XOR flip first 2 bytes of the leaked ciphertext in order to get the flag.

  ```
  nc 10.10.20.11 1337
  Welcome! Please login as the admin!
  username: bdmin
  bdmin's password: sUp3rPaSs1
  Leaked ciphertext: b7753a27665448fb15ba92b6f4f8ed989c1ba67426ce0f384b0dd3c1fd76fb9960510f76f4f9c9ed4f4898c6d4b1f538
  enter ciphertext: b4753a27665448fb15ba92b6f4f8ed989c1ba67426ce0f384b0dd3c1fd76fb9960510f76f4f9c9ed4f4898c6d4b1f538
  No way! You got it!
  A nice flag for you: THM{FliP_DaT_B1t_oR_G3t_Fl1pP3d}
  ```
  
- I tried to automate the whole process with `pwntools` but I always got a ciphertext with one character less for some reason so I simplified the whole process.

  ```
  cat xorfirsttwo.py      
  leaked_ciphertext = "b7753a27665448fb15ba92b6f4f8ed989c1ba67426ce0f384b0dd3c1fd76fb9960510f76f4f9c9ed4f4898c6d4b1f538"
  xor = ord('b') ^ ord('a')
  flipped = hex(int(leaked_ciphertext[0:2], 16) ^ xor)[2:]
  ciphertext = flipped.encode("utf-8")
  print(ciphertext)
  
  python3 xorfirsttwo.py 
  b'b4'
  ```

---

## 🏁 Flags

- **Flag**: `THM{FliP_DaT_B1t_oR_G3t_Fl1pP3d}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Making the automation work and it didnt work lol.`

- What did I learn?
  `Some practice.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
