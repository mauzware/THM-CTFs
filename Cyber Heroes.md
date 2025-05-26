## Cyber Heroes - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/cyberheroes)

<i>Want to be a part of the elite club of CyberHeroes? Prove your merit by finding a way to log in!

Access this challenge by deploying both the vulnerable machine by pressing the green "Start Machine" button located within this task, and the TryHackMe AttackBox by pressing the  "Start AttackBox" button located at the top-right of the page.

Navigate to the following URL using the AttackBox: http://MACHINE_IP</i>

**IP: 10.10.147.12**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Login Bypass <br>

**Tools Used**: / <br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Bypassing login

- When you visit the login page, in the source code you can find this:

  ```
    <script>
    function authenticate() {
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) { 
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("flag").innerHTML = this.responseText ;
            document.getElementById("todel").innerHTML = "";
            document.getElementById("rm").remove() ;
          }
        };
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_"+a.value+"_"+b.value+".txt", true);
        xhttp.send();
      }
      else {
        alert("Incorrect Password, try again.. you got this hacker !")
      }
    }
  </script>
  ```
  
- Reverse the string `54321@terceSrepuS` and login with newly found credentials to get the flag.

  ```
  Congrats Hacker, you made it !! Go ahead and nail other challenges as well :D flag{edb0be532c540b1a150c3a7e85d2466e}
  ```

---

## 🏁 Flags

- **Flag**: `flag{edb0be532c540b1a150c3a7e85d2466e}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing.`

- What did I learn?
  `Nothing new.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
