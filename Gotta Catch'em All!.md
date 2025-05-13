## Gotta Catch'em All! - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/pokemon)

<i>Remember to connect to the VPN network using OpenVPN, It may take some time for the machine to properly deploy.

You can also deploy your own Kali Linux machine, and control it in your browser using the provided Kali machine (Subscription Required).

Enjoy the room!</i>

**IP: 10.10.226.182**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Linux Privilege Escalation, Enumeration <br>

**Tools Used**: Nmap<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan. I also ran Gobuster but it found nothing.

  ```
  nmap -sC -sV -T5 -p- 10.10.226.182

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
  |   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
  |_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Can You Find Them All?
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- In the source code of web app you can find two interesting things, Pokemon names and SSH credentials.

  ```
      <script type="text/javascript">
    	const randomPokemon = [
    		'Bulbasaur', 'Charmander', 'Squirtle',
    		'Snorlax',
    		'Zapdos',
    		'Mew',
    		'Charizard',
    		'Grimer',
    		'Metapod',
    		'Magikarp'
    	];
    	const original = randomPokemon.sort((pokemonName) => {
    		const [aLast] = pokemonName.split(', ');
    	});

    	console.log(original);
    </script>


        </div>
        <pokemon>:<hack_the_pokemon>
        	<!--(Check console for extra surprise!)-->
      </div>
  ```
  
- Let's login and catch them all!

---

## 👑 Catching Them All and Root Privilege Escalation

- First Pokemon is in the ZIP file in Desktop directory, extract the file and read the note.

  ```
  pokemon@root:~$ cd Desktop/
  pokemon@root:~/Desktop$ ls -al
  total 12
  drwxr-xr-x  2 pokemon pokemon 4096 Jun 24  2020 .
  drwxr-xr-x 19 pokemon pokemon 4096 May 13 10:41 ..
  -rw-rw-r--  1 pokemon pokemon  383 Jun 22  2020 P0kEmOn.zip
  
  pokemon@root:~/Desktop$ unzip P0kEmOn.zip 
  Archive:  P0kEmOn.zip
     creating: P0kEmOn/
    inflating: P0kEmOn/grass-type.txt
   
  pokemon@root:~/Desktop$ ls -al
  total 16
  drwxr-xr-x  3 pokemon pokemon 4096 May 13 10:51 .
  drwxr-xr-x 19 pokemon pokemon 4096 May 13 10:41 ..
  drwxrwxr-x  2 pokemon pokemon 4096 Jun 22  2020 P0kEmOn
  -rw-rw-r--  1 pokemon pokemon  383 Jun 22  2020 P0kEmOn.zip
  
  pokemon@root:~/Desktop$ cd P0kEmOn/
  pokemon@root:~/Desktop/P0kEmOn$ ls -al
  total 12
  drwxrwxr-x 2 pokemon pokemon 4096 Jun 22  2020 .
  drwxr-xr-x 3 pokemon pokemon 4096 May 13 10:51 ..
  -rw-rw-r-- 1 pokemon pokemon   53 Jun 22  2020 grass-type.txt
  
  pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt 
  50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d
  ```
  
- It's encoded in hexadecimal, use any tool that you like for decoding, I have my own scripts for that.

  ```
  What you wanna do? encode or decode? decode
  Enter Hex string for decoding: 50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d
  Decode Hex value: PoKeMoN{Bulbasaur}
  ```
  
- I found another user credentials in `/Videos/Gotta/Catch/Them/ALL!`.

  ```
  pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ ls -al
  total 12
  drwxrwxr-x 2 pokemon pokemon 4096 Jun 22  2020 .
  drwxrwxr-x 3 pokemon pokemon 4096 Jun 22  2020 ..
  -rw-r--r-- 1 pokemon root      78 Jun 22  2020 Could_this_be_what_Im_looking_for?.cplusplus
  
  pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ cat Could_this_be_what_Im_looking_for\?.cplusplus 
  # include <iostream>
  
  int main() {
          std::cout << "ash : pikapika"
          return 0;
  ```

- I already tried accessing `ash` directory and reading `roots-pokemon.txt` but got denied. After logging in as `ash`, I tried it again but it was unsuccessful as well. Here's the catch, ash can run `sudo -l`.

  ```
  ash@root:/home$ sudo -l
  [sudo] password for ash: 
  Matching Defaults entries for ash on root:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User ash may run the following commands on root:
      (ALL : ALL) ALL
  ```

- Well, we can run any command as sudo, so just switch to root now. Also, read the `roots-pokemon.txt` as well.

  ```
  ash@root:/home$ sudo cat roots-pokemon.txt 
  Pikachu!

  ash@root:/home$ sudo su
  root@root:/home# whoami
  root
  ```

- Hint tells us to check website, so I went to that directory and found another Pokemon.

  ```
  root@root:/var/www/html# cat water-type.txt 
  Ecgudfxq_EcGmP{Ecgudfxq}
  ```

- This is ROT14, I use my own scripts for decoding.

- For the last one, I did some manual exploration but couldn't find it like others. Here's the one-liner `find` command that will list all files with `fire` in it.

  ```
  find / -name '*fire*' -type f 2>/dev/null

  [SNIP!]
  /etc/why_am_i_here?/fire-type.txt
  [SNIP!]
  ```

- There it is, read the file and challenge is done! String is encoded with Base64.

  ```
  root@root:/home/pokemon# cat /etc/why_am_i_here?/fire-type.txt
  UDBrM20wbntDaGFybWFuZGVyfQ==
  
  echo 'UDBrM20wbntDaGFybWFuZGVyfQ==' | base64 -d                                                    
  P0k3m0n{Charmander} 
  ```

---

## 🏁 Flags

- **Find the Grass-Type Pokemon**: `PoKeMoN{Bulbasaur}`
- **Find the Water-Type Pokemon**: `Squirtle_SqUaD{Squirtle}`
- **Find the Fire-Type Pokemon**: `P0k3m0n{Charmander}`
- **Who is Root's Favorite Pokemon?**: `Pikachu!`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Catching all Pokemons.`

- What did I learn?
  `Nothing, just sharpened my enumeration skills.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
