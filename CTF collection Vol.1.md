## CTF collection Vol.1 - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/ctfcollectionvol1)

<i>Just another random CTF room created by me. Well, the main objective of the room is to test your CTF skills. For your information, vol.1 consists of 20 tasks and all the challenges are extremely easy. Stay calm and Capture the flag.</i> :)

<i>Note: All the challenges flag are formatted as THM{flag}, unless stated otherwise</i>

<i>High five!</i>

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Crypto, Steganography <br>

**Tools Used**: Exiftool, Steghide, Stegsolve, Binwalk, Zbartools, Cipher Identifier, CyberChef, Wireshark

[CyberChef](https://gchq.github.io/CyberChef/)<br>

[Exiftool](https://exif.tools/)<br>

[Cipher Identifier](https://www.dcode.fr/cipher-identifier)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Ready, set, go!

- Q1 : Can you decode the following? `VEhNe2p1NTdfZDNjMGQzXzdoM19iNDUzfQ==`
  <br>

  It's a Base64 encoding.

  ```
  echo 'VEhNe2p1NTdfZDNjMGQzXzdoM19iNDUzfQ==' | base64 -d
  THM{ju57_d3c0d3_7h3_b453}
  ```
  
- Q2 : Meta! meta! meta! meta...................................
  <br>

  Download the file and upload it to `exiftool` <br> `Owner Name	THM{3x1f_0r_3x17}`
  
- Q3 : Something is hiding. That's all you need to know.
  <br>

  Download the file, ITS STEGO BOI AND MOMMY !!!! 🦕❤️ Extract the message with `steghide` and read it, there is no passphrase just press enter.

  ```
  sudo apt install steghide
  
  steghide --extract -sf Extinction_1577976250757.jpg
  Enter passphrase: 
  wrote extracted data to "Final_message.txt".
  
  cat Final_message.txt 
  It going to be over soon. Sleep my child.
  
  THM{500n3r_0r_l473r_17_15_0ur_7urn}
  ```

- Q4 : Huh, where is the flag? `THM{wh173_fl46}`

- Q5 : Such technology is quite reliable.
  <br>

  Download the file and use `zbarimg` to extract flag from the QR code.

  ```
  sudo apt install zbartools
  
  zbarimg QR_1577976698747.png 
  QR-Code:THM{qr_m4k3_l1f3_345y}
  scanned 1 barcode symbols from 1 images in 0 seconds
  ```

- Q6 : Reverse it or read it? Both works, it's all up to you.
  <br>

  Download the file upload it to CyberChef  `THM{345y_f1nd_345y_60} Hello there, wish you have a nice day`

- Q7 : Can you decode it? `3agrSy1CewF9v8ukcSkPSYm3oKUoByUpKG4L`
  <br>

  This is Base58, CyberChef will do the job. `THM{17_h45_l3553r_l3773r5}`

- Q8 : Left, right, left, right... Rot 13 is too mainstream. Solve this `MAF{atbe_max_vtxltk}`
  <br>

  Cipher Identifier will tell you that is ROT Cipher and will decode it `THM{hail_the_caesar}`

- Q9 : Make a comment. I'm hungry now... I need the flag
  <br>

  When I saw this I knew that developer tools will do the job quickly. Flag is in the `<div>` of this question. `THM{4lw4y5_ch3ck_7h3_c0m3mn7}`

- Q10 : I accidentally messed up with this PNG file. Can you help me fix it? Thanks, ^^
  <br>

  OK, I did this one with `hexedit` and it's very simple, change first 4 bytes to this `89 50 4E 47`, just be careful when editing.

  ```
  sudo apt install hexedit
  
  hexedit spoil_1577979329740.png

  change first 4 bytes to this -> 89 50 4E 47
  
  save and exit -> CTRL+X Y
  
  open the image and it will show the flag THM{y35_w3_c4n}
  ```

- Q11 : Some hidden flag inside Tryhackme social account.
  <br>

  Hidden message is on Reddit, its Base64 and when decoded it will provide another message, just use the provided dork and you can't miss it.

  ```
  https://www.reddit.com/r/tryhackme/comments/jy8p3p/about_ctf_collection_vol1/
  SlVTVCBPTkUgTU9SRSBTVEVQICEhISEhISEhIApzaXRlOiJyZWRkaXQuY29tIiBpbnRleHQ6IlRITSIgaW50aXRsZToidHJ5aGFja21l

  echo 'SlVTVCBPTkUgTU9SRSBTVEVQICEhISEhISEhIApzaXRlOiJyZWRkaXQuY29tIiBpbnRleHQ6IlRITSIgaW50aXRsZToidHJ5aGFja21l' | base64 -d
  JUST ONE MORE STEP !!!!!!!! 
  site:"reddit.com" intext:"THM" intitle:"tryhackme

  THM{50c14l_4cc0un7_15_p4r7_0f_051n7}
  ```

- Q12 : What is this? `++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>++++++++++++++.------------.+++++.>+++++++++++++++++++++++.<<++++++++++++++++++.>>-------------------.---------.++++++++++++++.++++++++++++.<++++++++++++++++++.+++++++++.<+++.+.>----.>++++.`
  <br>
  
  Cipher Identifier will do the job, I almost died from laughing when I saw name of the cipher lol... `THM{0h_my_h34d}`

- Q13 : Exclusive strings for everyone!

  `S1: 44585d6b2368737c65252166234f20626d`
  `S2: 1010101010101010101010101010101010`

  OK, so for this one you have to do S1 XOR S2. I made a Python script for this one but there are many other ways to do it.

  ```
  nano xor.py
  s1 = bytes.fromhex("44585d6b2368737c65252166234f20626d")
  s2 = bytes.fromhex("1010101010101010101010101010101010")
  
  flag = bytes([a ^ b for a, b in zip(s1, s2)])
  print(flag.decode())
  
  python3 xor.py 
  THM{3xclu51v3_0r}
  ```

- Q14 : Binary walk. Please exfiltrate my file :)
  <br>

  Download the file then use `binwalk` to extract the directory then read the message.

  ```
  binwalk hell_1578018688127.jpg 

  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  0             0x0             JPEG image data, JFIF standard 1.02
  30            0x1E            TIFF image data, big-endian, offset of first image directory: 8
  265845        0x40E75         Zip archive data, at least v2.0 to extract, uncompressed size: 69, name: hello_there.txt
  266099        0x40F73         End of Zip archive, footer length: 22
  
  binwalk -e hell_1578018688127.jpg 
  
  DECIMAL       HEXADECIMAL     DESCRIPTION
  --------------------------------------------------------------------------------
  265845        0x40E75         Zip archive data, at least v2.0 to extract, uncompressed size: 69, name: hello_there.txt
  
  WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
  
  cat hello_there.txt                 
  Thank you for extracting me, you are the best!
  
  THM{y0u_w4lk_m3_0u7}
  ```

- Q15 : There is something lurking in the dark. What does the flag said?
  <br>

  Download the image and use `stegsolve` to read the flag by pressing left arrow on keyboard. Here's the [installation](https://www.aldeid.com/wiki/Stegsolve)

  ```
  java -jar stegsolve.jar
  open image
  press left arrow
  
  THM{7h3r3_15_h0p3_1n_d4rkn355}
  ```

- Q16 : How good is your listening skill? P/S: The flag formatted as `THM{Listened Flag}`, the flag should be in All CAPS.
  <br>

  THIS ONE IS A BANGER HAHAHAHAHAHA LOVE IT! Download the file and use zbarimg to get the link then it's on you.

  ```
  zbarimg QRCTF_1579095601577.png 
  QR-Code:https://soundcloud.com/user-86667759/thm-ctf-vol1
  scanned 1 barcode symbols from 1 images in 0 seconds
  
  THM{SOUNDINGQR}
  ```

- Q17 : Dig up the past. Sometimes we need a 'machine' to dig the past.

  `
  Targetted website: https://www.embeddedhacker.com/
  Targetted time: 2 January 2020
  `

  This one is pretty simple, `archive.org` will do the job for you.

  ```
  archive.org
  https://web.archive.org/web/20200102131252/https://www.embeddedhacker.com/
  THM{ch3ck_th3_h4ckb4ck} 
  ```

- Q18 : Can you solve the following? By the way, I lost the key. Sorry >.< `MYKAHODTQ{RVG_YVGGK_FAL_WXF}` Flag format: TRYHACKME{FLAG IN ALL CAP}
  <br>

  Yeaaah right, it's uncrackable for sure, use CyberChef Viginere Decode with key `thm` `TRYHACKME{YOU_FOUND_THE_KEY}`

- Q19 : Decode the following text. `581695969015253365094191591547859387620042736036246486373595515576333693`
  <br>

  This one was a bit tricky ngl, it goes from decimal to hexadecimal. Use [this tool](https://www.rapidtables.com/convert/number/decimal-to-hex.html) for converting decimal to hex. CyberChef will decode it from hex.

  ```
  original string:
  581695969015253365094191591547859387620042736036246486373595515576333693
  
  decimal to hex:
  54484D7B31375F6A7535375F346E5F307264316E3472795F62343533357D
  
  then put hex string to CyberChef
  THM{17_ju57_4n_0rd1n4ry_b4535}
  ```

- Q20 : I just hacked my neighbor's WiFi and try to capture some packet. He must be up to no good. Help me find it.
  <br>

  As you already know, Wireshark is the GOAT when it comes to packets. Use `http` filter then `Follow HTTP stream` on packet 1825.

  ```
  GET /flag.txt HTTP/1.1
  Host: 192.168.247.140
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate
  Connection: keep-alive
  Upgrade-Insecure-Requests: 1
  If-Modified-Since: Fri, 03 Jan 2020 04:36:45 GMT
  If-None-Match: "e1bb7-15-59b34db67925a"
  Cache-Control: max-age=0
  
  
  HTTP/1.1 200 OK
  Date: Fri, 03 Jan 2020 04:43:14 GMT
  Server: Apache/2.2.22 (Ubuntu)
  Last-Modified: Fri, 03 Jan 2020 04:42:12 GMT
  ETag: "e1bb7-20-59b34eee33e0c"
  Accept-Ranges: bytes
  Vary: Accept-Encoding
  Content-Encoding: gzip
  Content-Length: 52
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/plain
  
  THM{d0_n07_574lk_m3}
  
  Found me!
  ```

- I had so much fun with this challenge — loved it! Thank you, DesKel! 👑 See ya in the next one!

---

## 💬 Notes & Reflections

- What was challenging in this room for me?🦕
  `Finding right tools...`

- What did I learn?
  `Few new tools, more practice with Crypto and Steganography and most importantly I met my boi Stego again! That listening QR code is something else ngl...` 🦕

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
