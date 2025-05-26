## Dig Dug - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/digdug)

<i>Oooh, turns out, this MACHINE_IP machine is also a DNS server! If we could dig into it, I am sure we could find some interesting records! But... it seems weird, this only responds to a special type of request for a givemetheflag.com domain?

Access this challenge by deploying both the vulnerable machine by pressing the green "Start Machine" button located within this task, and the TryHackMe AttackBox by pressing the  "Start AttackBox" button located at the top-right of the page.

Use some common DNS enumeration tools installed on the AttackBox to get the DNS server on MACHINE_IP to respond with the flag.</i>

**IP: 10.10.227.247**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: DNS <br>

**Tools Used**: dig<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 DNS Enumeration

- Add IP to your `/etc/hosts` and run `dig`.

  ```
  $ dig digdug.thm

  ; <<>> DiG 9.20.7-1-Debian <<>> digdug.thm
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 37729
  ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
  
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 1232
  ; COOKIE: 500fb44a1c4e109a010000006834ac5edf40e885a6880b3c (good)
  ;; QUESTION SECTION:
  ;digdug.thm.                    IN      A
  
  ;; AUTHORITY SECTION:
  .                       14682   IN      SOA     a.root-servers.net. nstld.verisign-grs.com. 2025052601 1800 900 604800 86400
  
  ;; Query time: 4 msec
  ;; SERVER: 192.168.1.1#53(192.168.1.1) (UDP)
  ;; WHEN: Mon May 26 19:00:59 BST 2025
  ;; MSG SIZE  rcvd: 142
  ```
  
- Since we already have one subdomain provided, let's check that one out as well.

  ```
  $ dig givemetheflag.com @digdug.thm

  ; <<>> DiG 9.20.7-1-Debian <<>> givemetheflag.com @digdug.thm
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43876
  ;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
  
  ;; QUESTION SECTION:
  ;givemetheflag.com.             IN      A
  
  ;; ANSWER SECTION:
  givemetheflag.com.      0       IN      TXT     "flag{0767ccd06e79853318f25aeb08ff83e2}"
  
  ;; Query time: 48 msec
  ;; SERVER: 10.10.227.247#53(digdug.thm) (UDP)
  ;; WHEN: Mon May 26 19:02:44 BST 2025
  ;; MSG SIZE  rcvd: 86
  ```

---

## 🏁 Flags

- **Flag**: `flag{0767ccd06e79853318f25aeb08ff83e2}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `Refreshing DNS enumeration.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
