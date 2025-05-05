## Ninja Skills - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/ninjaskills)

Let's have some fun with Linux. Deploy the machine and get started.

This machine may take up to 3 minutes to configure.

(If you prefer to SSH into the machine, use the credentials new-user as the username and password)

Answer the questions about the following files:

    8V2L
    bny0
    c4ZX
    D8B3
    FHl1
    oiMO
    PFbD
    rmfX
    SRSq
    uqyw
    v2Vb
    X1Uy

The aim is to answer the questions as efficiently as possible.

**IP: 10.10.114.64**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Commands <br>

**Tools Used**: <br>

**Author**: <br>

mauzware aka mauzinho <br>
[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🐁 I'm officialy a ninja mouse 

- Let's connect with SSH and start. This is the base syntax for find command: `find <file-name> -exec <command> \;`. With `-exec` you can execute any linux command so have that in mind.

  ```
  ssh new-user@10.10.114.64        
  The authenticity of host '10.10.114.64 (10.10.114.64)' can't be established.
  ED25519 key fingerprint is SHA256:+00WC/pAhDtahb5/yyyGVz399+f2Yw9DkVmh1KPGZwU.
  This key is not known by any other names.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
  Warning: Permanently added '10.10.114.64' (ED25519) to the list of known hosts.
  new-user@10.10.114.64's password: 
  Last login: Wed Oct 23 22:13:05 2019 from ip-10-10-231-194.eu-west-1.compute.internal
  ████████╗██████╗ ██╗   ██╗██╗  ██╗ █████╗  ██████╗██╗  ██╗███╗   ███╗███████╗
  ╚══██╔══╝██╔══██╗╚██╗ ██╔╝██║  ██║██╔══██╗██╔════╝██║ ██╔╝████╗ ████║██╔════╝
     ██║   ██████╔╝ ╚████╔╝ ███████║███████║██║     █████╔╝ ██╔████╔██║█████╗  
     ██║   ██╔══██╗  ╚██╔╝  ██╔══██║██╔══██║██║     ██╔═██╗ ██║╚██╔╝██║██╔══╝  
     ██║   ██║  ██║   ██║   ██║  ██║██║  ██║╚██████╗██║  ██╗██║ ╚═╝ ██║███████╗
     ╚═╝   ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚═╝     ╚═╝╚══════╝
          Let the games begin!
  [new-user@ip-10-10-114-64 ~]$ pwd
  /home/new-user
  [new-user@ip-10-10-114-64 ~]$ whoami
  new-user
  [new-user@ip-10-10-114-64 ~]$ id
  uid=501(new-user) gid=501(new-user) groups=501(new-user)
  ```
  
- This is the command that will find all mentioned files and I will also use it throughout my journey of becoming a ninja.

  ```
  find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) 2>/dev/null
  /mnt/D8B3
  /mnt/c4ZX
  /var/FHl1
  /var/log/uqyw
  /opt/PFbD
  /opt/oiMO
  /media/rmfX
  /etc/8V2L
  /etc/ssh/SRSq
  /home/v2Vb
  /X1Uy
  ```
  
- Which of the above files are owned by the best-group group(enter the answer separated by spaces in alphabetical order). Answer: `D8B3 v2Vb`

  ```
  find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -group "best-group"; 2>/dev/null
  /mnt/D8B3
  /home/v2Vb
  ```

- Which of these files contain an IP address? Answer: `oiMO` (you can find this either manually be greping IP from each file or with on liner, command for greping IP is `grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" <file_name>`)

  ```
  [new-user@ip-10-10-114-64 ~]$ grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /mnt/D8B3
  [new-user@ip-10-10-114-64 ~]$ grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /mnt/c4ZX 
  [new-user@ip-10-10-114-64 ~]$ grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /var/FHl1 
  [new-user@ip-10-10-114-64 ~]$ grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /var/log/uqyw 
  [new-user@ip-10-10-114-64 ~]$ grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /opt/PFbD 
  [new-user@ip-10-10-114-64 ~]$ grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /opt/oiMO 
  1.1.1.1

  find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" * {} \; 2>/dev/null 
  /opt/oiMO:1.1.1.1
  ```

- Which file has the SHA1 hash of 9d54da7584015647ba052173b84d45e8007eba94. Answer: `c4ZX`

  ```
  find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec sha1sum {} \; 2>/dev/null 
  2c8de970ff0701c8fd6c55db8a5315e5615a9575  /mnt/D8B3
  9d54da7584015647ba052173b84d45e8007eba94  /mnt/c4ZX
  d5a35473a856ea30bfec5bf67b8b6e1fe96475b3  /var/FHl1
  57226b5f4f1d5ca128f606581d7ca9bd6c45ca13  /var/log/uqyw
  256933c34f1b42522298282ce5df3642be9a2dc9  /opt/PFbD
  5b34294b3caa59c1006854fa0901352bf6476a8c  /opt/oiMO
  4ef4c2df08bc60139c29e222f537b6bea7e4d6fa  /media/rmfX
  0323e62f06b29ddbbe18f30a89cc123ae479a346  /etc/8V2L
  acbbbce6c56feb7e351f866b806427403b7b103d  /etc/ssh/SRSq
  7324353e3cd047b8150e0c95edf12e28be7c55d3  /home/v2Vb
  59840c46fb64a4faeabb37da0744a46967d87e57  /X1Uy
  ```
  
- Which file contains 230 lines? Answer: `bny0` ; it's the only file that's missing from the list.

  ```
  find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec wc -l {} \; 2>/dev/null 
  209 /mnt/D8B3
  209 /mnt/c4ZX
  209 /var/FHl1
  209 /var/log/uqyw
  209 /opt/PFbD
  209 /opt/oiMO
  209 /media/rmfX
  209 /etc/8V2L
  209 /etc/ssh/SRSq
  209 /home/v2Vb
  209 /X1Uy
  ```

- This command will give you answers for the last two questions. Which file's owner has an ID of 502? Answer: `X1Uy` 

  ```
  find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec ls -ln {} \; 2>/dev/null 
  -rw-rw-r-- 1 501 502 13545 Oct 23  2019 /mnt/D8B3
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /mnt/c4ZX
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /var/FHl1
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /var/log/uqyw
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /opt/PFbD
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /opt/oiMO
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /media/rmfX
  -rwxrwxr-x 1 501 501 13545 Oct 23  2019 /etc/8V2L
  -rw-rw-r-- 1 501 501 13545 Oct 23  2019 /etc/ssh/SRSq
  -rw-rw-r-- 1 501 502 13545 Oct 23  2019 /home/v2Vb
  -rw-rw-r-- 1 502 501 13545 Oct 23  2019 /X1Uy
  ```

- Which file's owner has an ID of 502? Answer: `X1Uy`
  
- Which file is executable by everyone? Answer: `8V2L`

- See ya in the next one ninjas!

---

## 🏁 Flags

- **Which of the above files are owned by the best-group group(enter the answer separated by spaces in alphabetical order)**: `D8B3 v2Vb`
- **Which of these files contain an IP address?**: `oiMO`
- **Which file has the SHA1 hash of 9d54da7584015647ba052173b84d45e8007eba94**: `c4ZX`
- **Which file contains 230 lines?**: `bny0`
- **Which file's owner has an ID of 502?**: `X1Uy`
- **Which file is executable by everyone?**: `8V2L`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Becoming a ninja is always challenging...`

- What did I learn?
  `I BECAME A NINJA!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
