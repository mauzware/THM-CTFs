## Investigating Windows - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/investigatingwindows)

This is a challenge that is exactly what is says on the tin, there are a few challenges around investigating a windows machine that has been previously compromised.

Connect to the machine using RDP. The credentials the machine are as follows:

Username: Administrator

Password: letmein123!

Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up.

**IP: 10.10.241.61**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Windows <br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Investigation starts now

- First connect to the machine with RDP `rdesktop 10.10.241.61`

- Q : `Whats the version and year of the windows machine?` <br>

  A : `Windows Server 2016` <br>

  You can find the details in Settings -> About or with CMD.

  ```
  systeminfo | findstr /B /C:"OS Name" /B /C:"OS Version"
  
  C:\Users\Administrator>systeminfo | findstr /B /C:"OS Name" /B /C:"OS Version"
  OS Name:                   Microsoft Windows Server 2016 Datacenter
  OS Version:                10.0.14393 N/A Build 14393
  ```

- Q : `Which user logged in last? ; Hint - That's you just now. But, who logged in before you?` <br>

  A : `Administrator` <br>

  Open `Event Viewer` and go to `Security` and filter ID to 4624. You can also get the info with PowerShell, you are `SYSTEM`, list is long tho.

  ```
  $Logins = Get-WinEvent -LogName "Security" | Where-Object {$_.Id -eq "4624"}

  ForEach($Login in $Logins)
  {
  $Login.Properties[5]
  }

  PS C:\Users\Administrator> $Logins = Get-WinEvent -LogName "Security" | Where-Object {$_.Id -eq "4624"}
  PS C:\Users\Administrator>
  PS C:\Users\Administrator> ForEach($Login in $Logins)
  >> {
  >> $Login.Properties[5]
  >> }
  
  Value
  -----
  SYSTEM
  SYSTEM
  SYSTEM
  SYSTEM
  Administrator
  DWM-2
  DWM-2
  SYSTEM
  ANONYMOUS LOGON
  SYSTEM
  SYSTEM
  SYSTEM
  SYSTEM
  SYSTEM
  DWM-1
  DWM-1
  LOCAL SERVICE
  SYSTEM
  .
  .
  .
  
  ```

- Q : `When did John log onto the system last? Answer format: MM/DD/YYYY H:MM:SS AM/PM ; Hint - Try using cmd to find this out. Please keep the answer format in mind before giving the answer.` <br>

  A : `03/02/2019 5:48:32 PM` <br>
  
  If you still have Event Viewer on, click on `Find` and type `John` then click `Find next` and you'll get the details.

  ```
  Log Name:      Security
  Source:        Microsoft-Windows-Security-Auditing
  Date:          3/2/2019 5:40:44 PM
  Event ID:      4624
  Task Category: Logon
  Level:         Information
  Keywords:      Audit Success
  User:          N/A
  Computer:      EC2AMAZ-I8UHO76
  Description:
  An account was successfully logged on.
  
  Subject:
  	Security ID:		NULL SID
  	Account Name:		-
  	Account Domain:		-
  	Logon ID:		0x0
  
  Logon Information:
  	Logon Type:		3
  	Restricted Admin Mode:	-
  	Virtual Account:		No
  	Elevated Token:		No
  
  Impersonation Level:		Impersonation
  
  New Logon:
  	Security ID:		EC2AMAZ-I8UHO76\John
  	Account Name:		John
  	Account Domain:		EC2AMAZ-I8UHO76
  	Logon ID:		0x39FB4D
  	Linked Logon ID:		0x0
  	Network Account Name:	-
  	Network Account Domain:	-
  	Logon GUID:		{00000000-0000-0000-0000-000000000000}
  
  Process Information:
  	Process ID:		0x0
  	Process Name:		-
  
  Network Information:
  	Workstation Name:	cloud
  	Source Network Address:	88.104.10.206
  	Source Port:		0
  
  Detailed Authentication Information:
  	Logon Process:		NtLmSsp 
  	Authentication Package:	NTLM
  	Transited Services:	-
  	Package Name (NTLM only):	NTLM V2
  	Key Length:		128
  ```

  This is a PowerShell command for same question. `($Logins | Where-Object {$_.Message -like “*John*”} | Select-Object -First 1).TimeCreated`

  ```
  PS C:\Users\Administrator> ($Logins | Where-Object {$_.Message -like “*John*”} | Select-Object -First 1).TimeCreated

  Saturday, March 2, 2019 5:48:32 PM
  ```

- Q : `What IP does the system connect to when it first starts?` <br>

  A : `10.34.2.3` <br>

  You can find this IP whenever `TMP\p.exe` runs, IP is reflected in CMD. You can also get with `System Information -> Software Environment -> Startup Apps`, there's only one value.

- Q : `What two accounts had administrative privileges (other than the Administrator user)? Answer format: List them in alphabetical order.` <br>

  A : `Guest, Jenny` <br>

  I got it with PowerShell.

  ```
  (Get-LocalGroupMember “Administrators”).Name

  PS C:\Users\Administrator> (Get-LocalGroupMember “Administrators”).Name
  EC2AMAZ-I8UHO76\Administrator
  EC2AMAZ-I8UHO76\Guest
  EC2AMAZ-I8UHO76\Jenny
  
  
  net localgroup "Administrators" 
  
  PS C:\Users\Administrator> net localgroup "Administrators"
  Alias name     Administrators
  Comment        Administrators have complete and unrestricted access to the computer/domain
  
  Members
  
  -------------------------------------------------------------------------------
  Administrator
  Guest
  Jenny
  The command completed successfully.
  ```

- Q : `Whats the name of the scheduled task that is malicous.` <br>

  A : `Clean file system` <br>

  I found this one with `Task Scheduler`, you can also get it with PowerShell. In Task Scheduler, `Clean File System` is the only program that has description.

  ```
  Task Scheduler
  
  Amazon Ec2 Launch - Instance I... Disabled
  check logged in                   Ready
  Clean file system                 Ready
  falshupdate22                     Ready
  GameOver                          Ready
  update windows                    Ready
  ```

  ```
  Get-ScheduledTask

  PS C:\Users\Administrator> Get-ScheduledTask
  
  TaskPath                                       TaskName                          State
  --------                                       --------                          -----
  \                                              Amazon Ec2 Launch - Instance I... Disabled
  \                                              check logged in                   Ready
  \                                              Clean file system                 Ready
  \                                              falshupdate22                     Ready
  \                                              GameOver                          Ready
  \                                              update windows                    Ready
  \Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319    Ready
  \Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319 64 Ready
  \Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319... Disabled
  \Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319... Disabled
  \Microsoft\Windows\Active Directory Rights ... AD RMS Rights Policy Template ... Disabled
  \Microsoft\Windows\Active Directory Rights ... AD RMS Rights Policy Template ... Ready
  \Microsoft\Windows\AppID\                      EDP Policy Manager                Ready
  \Microsoft\Windows\AppID\                      PolicyConverter                   Disabled
  \Microsoft\Windows\AppID\                      SmartScreenSpecific               Ready
  \Microsoft\Windows\AppID\                      VerifiedPublisherCertStoreCheck   Disabled
  \Microsoft\Windows\Application Experience\     Microsoft Compatibility Appraiser Ready
  \Microsoft\Windows\Application Experience\     ProgramDataUpdater                Ready
  .
  .
  .
  .
  ```

- OK, so the following command will give you answers for next two questions, `Execute` shows the task, `Arguments` shows the port. Alternatively, Task Scheduler will do the same, select malicious file from previous task and click on `Actions`. <br>

  Q : `What file was the task trying to run daily?` <br>

  A : `nc.ps1` <br>
  
  Q : `What port did this file listen locally for?` <br>

  A : `1348` <br>

  ```
  (Get-ScheduledTask -TaskName “Clean file system”).Actions 

  PS C:\Users\Administrator> (Get-ScheduledTask -TaskName “Clean file system”).Actions
  
  
  Id               :
  Arguments        : -l 1348
  Execute          : C:\TMP\nc.ps1
  WorkingDirectory :
  PSComputerName   :
  ```

- Q : `When did Jenny last logon?` <br>

  A : `Never` <br>

  In Event Viewer you do the same thing like you did with John, it prints error. PowerShell command also showed nothing, but when I ran `net user` I found the answer.

  ```
  net user Jenny

  PS C:\Users\Administrator> net user Jenny
  User name                    Jenny
  Full Name                    Jenny
  Comment
  User's comment
  Country/region code          000 (System Default)
  Account active               Yes
  Account expires              Never
  
  Password last set            3/2/2019 4:52:25 PM
  Password expires             Never
  Password changeable          3/2/2019 4:52:25 PM
  Password required            Yes
  User may change password     Yes
  
  Workstations allowed         All
  Logon script
  User profile
  Home directory
  Last logon                   Never
  
  Logon hours allowed          All
  
  Local Group Memberships      *Administrators       *Users
  Global Group memberships     *None
  The command completed successfully.
  ```

- Q : `At what date did the compromise take place? Answer format: MM/DD/YYYY` <br>

  A : `03/02/2019` <br>

  Since I know that all malicious files are in `TMP` directory, I ran the following code in PowerShell.

  ```
  Get-ChildItem C:\TMP | Select-Object Name, CreationTime


  PS C:\Users\Administrator> Get-ChildItem C:\TMP | Select-Object Name, CreationTime
  
  Name                  CreationTime
  ----                  ------------
  d.txt                 3/2/2019 4:44:51 PM
  mim-out.txt           3/2/2019 4:45:27 PM
  mim.exe               3/2/2019 4:45:19 PM
  moutput.tmp           3/2/2019 4:39:33 PM
  nbtscan.exe           3/2/2019 4:46:03 PM
  nc.ps1                3/2/2019 4:45:03 PM
  p.exe                 3/2/2019 4:46:37 PM
  scan1.tmp             3/2/2019 4:40:08 PM
  scan2.tmp             3/2/2019 4:40:08 PM
  scan3.tmp             3/2/2019 4:40:08 PM
  schtasks-backdoor.ps1 3/2/2019 4:47:04 PM
  somethingwindows.dmp  3/2/2019 4:45:13 PM
  sys.txt               3/2/2019 4:40:13 PM
  WMIBackdoor.ps1       3/2/2019 4:45:08 PM
  xCmd.exe              3/2/2019 4:46:51 PM
  ```

- Q : `During the compromise, at what time did Windows first assign special privileges to a new logon? Answer format: MM/DD/YYYY HH:MM:SS AM/PM ; Hint - 00/00/0000 0:00:49 PM` <br>

  A : `03/02/2019 4:04:49 PM` <br>

  OK, this one will be tricky to find if there was no hint, since hint said that assignment time ended with `:49 PM` that made my job much easier. I found this one with PowerShell but you can also find it with Event Viewer by filterin time and ID's.

  ```
  $Specials = Get-WinEvent -LogName "Security" | Where-Object {$_.Id -eq "4672"}

  PS C:\Users\Administrator> $Specials.TimeCreated

  .
  .
  .
  .
  Saturday, March 2, 2019 4:25:46 PM
  Saturday, March 2, 2019 4:25:46 PM
  Saturday, March 2, 2019 4:09:51 PM
  Saturday, March 2, 2019 4:09:34 PM
  Saturday, March 2, 2019 4:09:07 PM
  Saturday, March 2, 2019 4:06:53 PM
  Saturday, March 2, 2019 4:04:57 PM
  Saturday, March 2, 2019 4:04:53 PM
  Saturday, March 2, 2019 4:04:53 PM
  Saturday, March 2, 2019 4:04:52 PM
  Saturday, March 2, 2019 4:04:52 PM
  Saturday, March 2, 2019 4:04:49 PM
  Saturday, March 2, 2019 4:04:40 PM
  Saturday, March 2, 2019 4:04:39 PM
  Saturday, March 2, 2019 4:04:39 PM
  Saturday, March 2, 2019 4:04:39 PM
  Saturday, March 2, 2019 4:04:39 PM
  Saturday, March 2, 2019 4:04:39 PM
  Saturday, March 2, 2019 4:04:39 PM
  Saturday, March 2, 2019 4:04:38 PM
  Saturday, March 2, 2019 4:03:07 PM
  Saturday, March 2, 2019 4:03:07 PM
  Saturday, March 2, 2019 4:03:07 PM
  Saturday, March 2, 2019 4:03:04 PM
  Saturday, March 2, 2019 4:03:00 PM
  Saturday, March 2, 2019 4:02:59 PM
  Saturday, March 2, 2019 4:02:59 PM
  Saturday, March 2, 2019 4:02:59 PM
  Saturday, March 2, 2019 4:02:59 PM
  Saturday, March 2, 2019 4:02:59 PM
  Saturday, March 2, 2019 4:02:58 PM
  Saturday, March 2, 2019 4:02:58 PM
  Wednesday, February 13, 2019 8:14:31 AM
  Wednesday, February 13, 2019 8:14:31 AM
  Wednesday, February 13, 2019 8:14:30 AM
  ```

- Q : `What tool was used to get Windows passwords?` <br>

  A : `Mimikatz` <br>

  So for this one I typed Mimikatz by habit but you can find the answer with PowrShell or by manually going to `C:\TMP` and opening the file `mim.out`.

  ```
  Get-Content C:\TMP\mim-out.txt

  PS C:\Users\Administrator> Get-Content C:\TMP\mim-out.txt
  
    .#####.   mimikatz 2.0 alpha (x86) release "Kiwi en C" (Feb 16 2015 22:17:52)
   .## ^ ##.
   ## / \ ##  /* * *
   ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
   '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
    '#####'                                     with 15 modules * * */
  
  
  mimikatz(powershell) # sekurlsa::logonpasswords
  
  Authentication Id : 0 ; 195072 (00000000:0002fa00)
  Session           : Interactive from 1
  User Name         : Ion
  Domain            : Ion-PC
  SID               : S-1-5-21-2367887663-2567669145-1589166190-1000
          msv :
           [00000003] Primary
           * Username : Ion
           * Domain   : Ion-PC
           * NTLM     : a4a9436b46f7e948b2417435b63d6cac
           * SHA1     : 6c69a6cc3e5313a83fde6d27256b17e5020ffcb5
           [00010000] CredentialKeys
           * NTLM     : a4a9436b46f7e948b2417435b63d6cac
           * SHA1     : 6c69a6cc3e5313a83fde6d27256b17e5020ffcb5
          tspkg :
          wdigest :
           * Username : Ion
           * Domain   : Ion-PC
           * Password : MySecretP4ass
          kerberos :
           * Username : Ion
           * Domain   : Ion-PC
           * Password : (null)
          ssp :
          credman :
  
  Authentication Id : 0 ; 195024 (00000000:0002f9d0)
  Session           : Interactive from 1
  User Name         : Ion
  Domain            : Ion-PC
  SID               : S-1-5-21-2367887663-2567669145-1589166190-1000
          msv :
           [00010000] CredentialKeys
           * NTLM     : a4a9436b46f7e948b2417435b63d6cac
           * SHA1     : 6c69a6cc3e5313a83fde6d27256b17e5020ffcb5
           [00000003] Primary
           * Username : Administrator
           * Domain   : SUDOTS35
           [00000003] Primary
           * Username : Ion
           * Domain   : Ion-PC
           * NTLM     : a4a9436b46f7e948b2417435b63d6cac
           * SHA1     : 6c69a6cc3e5313a83fde6d27256b17e5020ffcb5
          tspkg :
          wdigest :
           * Username : Ion
           * Domain   : Ion-PC
           * Password : MySecretP4ass
          kerberos :
           * Username : Ion
           * Domain   : Ion-PC
           * Password : (null)
          ssp :
          credman :
  
  Authentication Id : 0 ; 997 (00000000:000003e5)
  Session           : Service from 0
  User Name         : LOCAL SERVICE
  Domain            : NT AUTHORITY
  SID               : S-1-5-19
          msv :
          tspkg :
          wdigest :
           * Username : (null)
           * Domain   : (null)
           * Password : (null)
          kerberos :
           * Username : (null)
           * Domain   : (null)
           * Password : (null)
          ssp :
          credman :
  
  Authentication Id : 0 ; 996 (00000000:000003e4)
  Session           : Service from 0
  User Name         : ION-PC$
  Domain            : WORKGROUP
  SID               : S-1-5-20
          msv :
          tspkg :
          wdigest :
           * Username : ION-PC$
           * Domain   : WORKGROUP
           * Password : (null)
          kerberos :
           * Username : ion-pc$
           * Domain   : WORKGROUP
           * Password : (null)
          ssp :
          credman :
  
  Authentication Id : 0 ; 30678 (00000000:000077d6)
  Session           : UndefinedLogonType from 0
  User Name         : (null)
  Domain            : (null)
  SID               :
          msv :
          tspkg :
          wdigest :
          kerberos :
          ssp :
          credman :
  
  Authentication Id : 0 ; 999 (00000000:000003e7)
  Session           : UndefinedLogonType from 0
  User Name         : ION-PC$
  Domain            : WORKGROUP
  SID               : S-1-5-18
          msv :
          tspkg :
          wdigest :
           * Username : ION-PC$
           * Domain   : WORKGROUP
           * Password : (null)
          kerberos :
           * Username : ion-pc$
           * Domain   : WORKGROUP
           * Password : (null)
          ssp :
          credman :
  
  mimikatz(powershell) # exit
  Bye!
  ```

- The following command will provide answers for both this qusetion and the last question. `Get-Content C:\Windows\System32\drivers\etc\hosts`. You can also manually check `hosts` in `C:\Windows\System32\drivers\etc`. <br>

  Q : `What was the attackers external control and command servers IP?` <br>

  A : `76.32.97.132` <br>

  Q : `Check for DNS poisoning, what site was targeted?` <br>

  A : `google.com` <br>

  ```
  Get-Content C:\Windows\System32\drivers\etc\hosts 

  PS C:\Users\Administrator> Get-Content C:\Windows\System32\drivers\etc\hosts
  # Copyright (c) 1993-2009 Microsoft Corp.
  #
  # This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
  #
  # This file contains the mappings of IP addresses to host names. Each
  # entry should be kept on an individual line. The IP address should
  # be placed in the first column followed by the corresponding host name.
  # The IP address and the host name should be separated by at least one
  # space.
  #
  # Additionally, comments (such as these) may be inserted on individual
  # lines or following the machine name denoted by a '#' symbol.
  #
  # For example:
  #
  #      102.54.94.97     rhino.acme.com          # source server
  #       38.25.63.10     x.acme.com              # x client host
  
  # localhost name resolution is handled within DNS itself.
  #       127.0.0.1       localhost
  #       ::1             localhost
  10.2.2.2        update.microsoft.com
  127.0.0.1  www.virustotal.com
  127.0.0.1  www.www.com
  127.0.0.1  dci.sophosupd.com
  10.2.2.2        update.microsoft.com
  127.0.0.1  www.virustotal.com
  127.0.0.1  www.www.com
  127.0.0.1  dci.sophosupd.com
  10.2.2.2        update.microsoft.com
  127.0.0.1  www.virustotal.com
  127.0.0.1  www.www.com
  127.0.0.1  dci.sophosupd.com
  76.32.97.132 google.com
  76.32.97.132 www.google.com
  ```

- Q : `What was the extension name of the shell uploaded via the servers website?` <br>

  A : `.jsp` <br>

  For this question, same rule applies like for previous ones. You can either use PowerShell or manually check `C:\inetpub\wwwroot`. Honestly, I prefer scripting since I love Python and I also wanted to practice PowerShell.

  ```
  Get-ChildItem C:\inetpub\wwwroot | Select-Object Name, CreationTime

  PS C:\Users\Administrator> Get-ChildItem C:\inetpub\wwwroot | Select-Object Name, CreationTime
  
  Name      CreationTime
  ----      ------------
  b.jsp     3/2/2019 4:47:26 PM
  shell.gif 3/2/2019 4:47:26 PM
  tests.jsp 3/2/2019 4:47:26 PM
  ```

- Q : `What was the last port the attacker opened? ; Hint - Firewall` <br>

  A : `1337` <br>

  PowerShell command is `Get-NetFirewallRule -Direction Inbound` and manually you can open `Firewall -> Inbound Rules -> Select the first rule -> Properties -> Protocols and ports` and you'll find it. PowerShell output was insanely long so for this one I did it manually.

---

## 🏁 Flags

- **Whats the version and year of the windows machine?**: `Windows Server 2016`
- **Which user logged in last?**: `Administrator`
- **When did John log onto the system last? Answer format: MM/DD/YYYY H:MM:SS AM/PM**: `03/02/2019 5:48:32 PM`
- **What IP does the system connect to when it first starts?**: `10.34.2.3`
- **What two accounts had administrative privileges (other than the Administrator user)? Answer format: List them in alphabetical order.**: `Guest, Jenny`
- **Whats the name of the scheduled task that is malicous.**: `Clean file system`
- **What file was the task trying to run daily?**: `nc.ps1`
- **What port did this file listen locally for?**: `1348`
- **When did Jenny last logon?**: `Never`
- **At what date did the compromise take place? Answer format: MM/DD/YYYY**: `03/02/2019`
- **During the compromise, at what time did Windows first assign special privileges to a new logon? Answer format: MM/DD/YYYY HH:MM:SS AM/PM**: `03/02/2019 4:04:49 PM`
- **What tool was used to get Windows passwords?**: `Mimikatz`
- **What was the attackers external control and command servers IP?**: `76.32.97.132`
- **What was the extension name of the shell uploaded via the servers website?**: `.jsp`
- **What was the last port the attacker opened?**: `1337`
- **Check for DNS poisoning, what site was targeted?**: `google.com`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Oh man, a lot of things, when it comes to Windows, there is a big difference between invesitgating Windows as a Blue Teamer and attacking Windows as a Red Teamer.`

- What did I learn?
  `I learner a lot ngl, especially for Blue side, I got so much practice with both PowerShell and Windows in general, I'm so glad that I did this room. Practice makes it perfect!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
