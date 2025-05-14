## Git Happens - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/githappens)

**IP: 10.10.122.13**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Enumeration, Git <br>

**Tools Used**: Nmap, Ffuf<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.122.13

  PORT   STATE SERVICE VERSION
  80/tcp open  http    nginx 1.14.0 (Ubuntu)
  | http-git: 
  |   10.10.122.13:80/.git/
  |     Git repository found!
  |_    Repository description: Unnamed repository; edit this file 'description' to name the...
  |_http-server-header: nginx/1.14.0 (Ubuntu)
  |_http-title: Super Awesome Site!
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.122.13/FUZZ -w /usr/share/wordlists/dirb/common.txt

  .git/HEAD               [Status: 200, Size: 23, Words: 2, Lines: 2, Duration: 59ms]
  css                     [Status: 301, Size: 194, Words: 7, Lines: 8, Duration: 49ms]
  index.html              [Status: 200, Size: 6890, Words: 541, Lines: 61, Duration: 49ms]
  :: Progress: [4614/4614] :: Job [1/1] :: 836 req/sec :: Duration: [0:00:06] :: Errors: 0 ::
  ```
  
- When I visited the login page I tried to get access to it but it was unsuccessful. Downloaded the `/.git/HEAD` which pointed me to `/.git/refs/heads/master`
  
- It was a dead end so I decided to check the Git directory. I did some manual exploration but after a few minutes, I actually downloaded the full `/.git/objects` directory since it had a lot inside and I didn't want to go manually through all of them.

- The thing is, when you download Git directory, if it is named as source IP, you can actually use Git commands inside of it, so I did that.

  ```
  wget -m -k http://10.10.122.13/.git/objects/

  cd 10.10.122.13
  ```

- First I checked for all the logs.

  ```
  git log

  commit d0b3578a628889f38c0affb1b75457146a4678e5 (HEAD -> master, tag: v1.0)
  Author: Adam Bertrand <hydragyrum@gmail.com>
  Date:   Thu Jul 23 22:22:16 2020 +0000
  
      Update .gitlab-ci.yml
  
  commit d0b3578a628889f38c0affb1b75457146a4678e5 (HEAD -> master, tag: v1.0)
  Author: Adam Bertrand <hydragyrum@gmail.com>
  Date:   Thu Jul 23 22:22:16 2020 +0000
  
      Update .gitlab-ci.yml
  
  commit 77aab78e2624ec9400f9ed3f43a6f0c942eeb82d
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Fri Jul 24 00:21:25 2020 +0200
  
      add gitlab-ci config to build docker file.
  
  commit 2eb93ac3534155069a8ef59cb25b9c1971d5d199
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Fri Jul 24 00:08:38 2020 +0200
  
      setup dockerfile and setup defaults.
  
  commit d6df4000639981d032f628af2b4d03b8eff31213
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Thu Jul 23 23:42:30 2020 +0200
  
      Make sure the css is standard-ish!
  
  commit d954a99b96ff11c37a558a5d93ce52d0f3702a7d
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Thu Jul 23 23:41:12 2020 +0200
  
      re-obfuscating the code to be really secure!
  
  commit bc8054d9d95854d278359a432b6d97c27e24061d
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Thu Jul 23 23:37:32 2020 +0200
  
      Security says obfuscation isn't enough.
      
      They want me to use something called 'SHA-512'
  
  commit e56eaa8e29b589976f33d76bc58a0c4dfb9315b1
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Thu Jul 23 23:25:52 2020 +0200
  
      Obfuscated the source code.
      
      Hopefully security will be happy!
  
  commit 395e087334d613d5e423cdf8f7be27196a360459
  Author: Hydragyrum <hydragyrum@gmail.com>
  Date:   Thu Jul 23 23:17:43 2020 +0200
  
      Made the login page, boss!
  
  commit 2f423697bf81fe5956684f66fb6fc6596a1903cc
  Author: Adam Bertrand <hydragyrum@gmail.com>
  Date:   Mon Jul 20 20:46:28 2020 +0000
  
      Initial commit
  ```

- OK, one of these commits looked very suspicious to me so I checked the actual commit and found the password inside. That's the whole challenge.

  ```git diff [find-it-on-your-own]```

---

## 🏁 Flags

- **Super Secret Password**: `Th1s_1s_4_L0ng_4nd_S3cur3_P4ssw0rd!`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting my way around it. This room is pure enumeration room tho.`

- What did I learn?
  `This was my first expereience with hacking Git if you can actually call it like that.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
