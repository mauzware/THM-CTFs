## Reversing ELF - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/reverselfiles)

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Reverse Engineering <br>

**Tools Used**: Radare and GDB<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Enumeration

- First one is pretty simple, just run the binary.

  ```
  chmod +x crackme1
  ./crackme1
  flag{not_that_kind_of_elf}
  ```

- Second one is also basic one, it requires password as argument which you can get by with `strings` or `cat`. After just run the binary with newly found password.

  ```
  cat crackme2
  super_secret_password
  
  chmod +x crackme2
  ./crackme2 super_secret_password
  Access granted.
  flag{if_i_submit_this_flag_then_i_will_get_points}
  ```

- Use the same method as with previous one. `strings` to get the password then decode it.

  ```
  Usage: %s PASSWORD
  malloc failed
  ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==
  Correct password!
  Come on, even my aunt Mildred got this one!
  ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
  
  echo 'ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==' | base64 -d
  f0r_y0ur_5ec0nd_le55on_unbase64_4ll_7h3_7h1ng5
  ```

- OK, so now hell started for me since I had absolutely zero experience with reverse engineering but thankfully my CTF teammates helped me and guided me through it. If I explained something not properly, I'm sorry, I'm still new to this.
  Run the binary with `Radare`, use `aaa` followed by `afl` in order to list all functions. Then we will use `pdf` to take a better look at specific function, here will be looking for `main` function.

  ```
  chmod +x crackme4
  r2 -d crackme4

  [0x7f05848425c0]> aaa
  INFO: Analyze all flags starting with sym. and entry0 (aa)
  INFO: Analyze imports (af@@@i)
  INFO: Analyze entrypoint (af@ entry0)
  INFO: Analyze symbols (af@@@s)
  INFO: Analyze all functions arguments/locals (afva@@@F)
  INFO: Analyze function calls (aac)
  INFO: Analyze len bytes of instructions for references (aar)
  INFO: Finding and parsing C++ vtables (avrr)
  INFO: Analyzing methods (af @@ method.*)
  INFO: Recovering local variables (afva@@@F)
  INFO: Skipping type matching analysis in debugger mode (aaft)
  INFO: Propagate noreturn information (aanr)
  INFO: Use -AA or aaaa to perform additional experimental analysis

  [0x7f05848425c0]> afl
  0x004004e0    1      6 sym.imp.puts
  0x004004f0    1      6 sym.imp.__stack_chk_fail
  0x00400500    1      6 sym.imp.printf
  0x00400510    1      6 sym.imp.__libc_start_main
  0x00400520    1      6 sym.imp.strcmp
  0x00400540    1     41 entry0
  0x00400570    4     41 sym.deregister_tm_clones
  0x004005a0    4     57 sym.register_tm_clones
  0x004005e0    3     28 entry.fini0
  0x00400600    4     42 entry.init0
  0x004007d0    1      2 sym.__libc_csu_fini
  0x0040062d    4     77 sym.get_pwd
  0x004007d4    1      9 sym._fini
  0x0040067a    6    156 sym.compare_pwd
  0x00400760    4    101 sym.__libc_csu_init
  0x00400716    4     74 main
  0x004004b0    3     26 sym._init
  0x00400530    1      6 loc.imp.__gmon_start__
  0x7f058482787b    3     40 fcn.7f058482787b
  0x7f0584833660    3    160 fcn.7f0584833660
  0x7f05848402d0    5    174 fcn.7f05848402d0
  0x7f0584830c30   23    415 fcn.7f0584830c30
  0x7f0584833320    3    162 fcn.7f0584833320
  [SNIP!]

  [0x7f05848425c0]> pdf @main
            ; DATA XREF from entry0 @ 0x40055d(r)
  ┌ 74: int main (int argc, char **argv);
  │ `- args(rdi, rsi) vars(2:sp[0xc..0x18])
  │           0x00400716      55             push rbp
  │           0x00400717      4889e5         mov rbp, rsp
  │           0x0040071a      4883ec10       sub rsp, 0x10
  │           0x0040071e      897dfc         mov dword [var_4h], edi     ; argc
  │           0x00400721      488975f0       mov qword [var_10h], rsi    ; argv
  │           0x00400725      837dfc02       cmp dword [var_4h], 2
  │       ┌─< 0x00400729      741b           je 0x400746
  │       │   0x0040072b      488b45f0       mov rax, qword [var_10h]
  │       │   0x0040072f      488b00         mov rax, qword [rax]
  │       │   0x00400732      4889c6         mov rsi, rax
  │       │   0x00400735      bf10084000     mov edi, str.Usage_:__s_password_nThis_time_the_string_is_hidden_and_we_used_strcmp_n ; 0x400810 ; "Usage : %s password\nThis time the string is hidden and we used strcmp\n"
  │       │   0x0040073a      b800000000     mov eax, 0
  │       │   0x0040073f      e8bcfdffff     call sym.imp.printf         ; int printf(const char *format)
  │      ┌──< 0x00400744      eb13           jmp 0x400759
  │      │└─> 0x00400746      488b45f0       mov rax, qword [var_10h]
  │      │    0x0040074a      4883c008       add rax, 8
  │      │    0x0040074e      488b00         mov rax, qword [rax]
  │      │    0x00400751      4889c7         mov rdi, rax
  │      │    0x00400754      e821ffffff     call sym.compare_pwd
  │      │    ; CODE XREF from main @ 0x400744(x)
  │      └──> 0x00400759      b800000000     mov eax, 0
  │           0x0040075e      c9             leave
  └           0x0040075f      c3             ret
  ```

- In main function there are two functions that we want to observe, `sym.imp.printf` and `sym.compare_pwd`. Our focus is `sym.compare_pwd` since it compares passwords. Set up a breakpoint before the function is called then use `px` to extract a secret password.

  ```
  [0x7f05848425c0]> ood 'random'
  child received signal 9
  45792

  
  [0x7f9bdb3d25c0]> db 0x004006d2
  [0x7f9bdb3d25c0]> dc
  INFO: hit breakpoint at: 0x4006d2

  [0x004006d2]> px @rdi
  - offset -      6061 6263 6465 6667 6869 6A6B 6C6D 6E6F  0123456789ABCDEF
  0x7fff45e97260  6d79 5f6d 3072 335f 7365 6375 7233 5f70  my_m0r3_secur3_p
  0x7fff45e97270  7764 0000 0000 0000 003f a1dc 528b 735f  wd.......?..R.s_
  0x7fff45e97280  a072 e945 ff7f 0000 5907 4000 0000 0000  .r.E....Y.@.....
  0x7fff45e97290  b873 e945 ff7f 0000 0000 0000 0200 0000  .s.E............
  0x7fff45e972a0  0200 0000 0000 0000 a86c 1cdb 9b7f 0000  .........l......
  0x7fff45e972b0  0000 0000 0000 0000 1607 4000 0000 0000  ..........@.....
  0x7fff45e972c0  0000 0000 0200 0000 b873 e945 ff7f 0000  .........s.E....
  0x7fff45e972d0  b873 e945 ff7f 0000 8c3b 4435 75b8 64ad  .s.E.....;D5u.d.
  0x7fff45e972e0  0000 0000 0000 0000 d073 e945 ff7f 0000  .........s.E....
  0x7fff45e972f0  00c0 3edb 9b7f 0000 0000 0000 0000 0000  ..>.............
  0x7fff45e97300  8c3b 20d0 a733 9a52 8c3b 80ed 4d0e 5352  .; ..3.R.;..M.SR
  0x7fff45e97310  0000 0000 0000 0000 0000 0000 0000 0000  ................
  0x7fff45e97320  0000 0000 0000 0000 0000 0000 0000 0000  ................
  0x7fff45e97330  d073 e945 ff7f 0000 003f a1dc 528b 735f  .s.E.....?..R.s_
  0x7fff45e97340  0000 0000 0000 0000 656d 1cdb 9b7f 0000  ........em......
  0x7fff45e97350  1607 4000 0000 0000 0000 0000 ff7f 0000  ..@.............
  ```

- For next one, my teammate told me a cheat code for this one. Just run `ltrace` on the binary with any input.

  ```
  ltrace ./crackme5
  __libc_start_main(["./crackme5"] <unfinished ...>
  puts("Enter your input:"Enter your input:
  )                                               = 18
  __isoc99_scanf(0x400966, 0x7ffc54397d60, 0, 0dsdsd
  )                          = 1
  strlen("dsdsd")                                                         = 5
  strlen("dsdsd")                                                         = 5
  strlen("dsdsd")                                                         = 5
  strlen("dsdsd")                                                         = 5
  strlen("dsdsd")                                                         = 5
  strlen("dsdsd")                                                         = 5
  strncmp("dsdsd", "OfdlDSA|3tXb32~X3tX@sX`4tXtz", 28)                    = 21
  puts("Always dig deeper"Always dig deeper
  )                                               = 18
  +++ exited (status 0) +++
  ```

- With `crackme6` use the same methodology as for `crackme4` but take a closer look at function `sym.my_Secure_test`. Open it in Graph Mode with `VV` and note all the `cmp al` values, we will combine them and crack them in order to get the secret password.

  ```
  chmod +x crackme6

  ./crackme6 
  Usage : ./crackme6 password
  Good luck, read the source
  
  r2 -d crackme6
  aaa
  afl
  pdf @main
  pdf @sym.my_secure_test
  VV @sym.my_secure_test

  NOTE ALL cmp al values
  0x31
  0x33
  0x33
  0x37
  0x5f
  0x70
  0x77
  0x64
  ```

- Remove `0x` and combine all of these values and convert the combined value in CyberChef, you should get `1337_pwd` as final result.

- This one was a bit tricky. We do the same procedure like with last one.

  ```
  r2 -d crackme7
  aaa
  afl
  pdf @main
  
  │ ││││ │    0x08048665      3d697a0000     cmp eax, 0x7a69             ; 'iz'
  │ ││││ │┌─< 0x0804866a      7517           jne 0x8048683
  │ ││││ ││   0x0804866c      83ec0c         sub esp, 0xc
  │ ││││ ││   0x0804866f      68bc880408     push str.Wow_such_h4x0r_    ; 0x80488bc ; "Wow such h4x0r!"
  │ ││││ ││   0x08048674      e8f7fcffff     call sym.imp.puts           ; int puts(const char *s)
  ```

- Now our focus is on `0x7a69` since that string is pushed and converted. Take the numbers from this hex value and convert it to decimal value, then provide it as input in binary and you'll get the flag. I used Python for conversion.

  ```
  nano todec.py
  
  string = "7a69"
  decimal = int(string, 16)
  print(decimal)
  
  pyhon3 todec.py
  
  31337

  ./crackme7
  Menu:
  
  [1] Say hello
  [2] Add numbers
  [3] Quit
  
  [>] 31337
  Wow such h4x0r!
  flag{much_reversing_very_ida_wow}
  ```

- For the last one, we need to find the correct password in order to get the flag. Let's do it!

  ```
  chmod +x crackme8

   ./crackme8
  Usage: ./crackme8 password
                                                                                                                       
  ./crackme8 bimbo
  Access denied.
  
  strings crackme8
  Usage: %s password
  Access denied.
  Access granted.
  ```

- Run the binary with `Radare` and do `pdf @main`. You'll find a `sym.imp.atoi` function that converts strings to an integer. Currently, in this binary the string that we are looking for is `cafef00d`.

  ```
  │      │    0x080484dc      e89ffeffff     call sym.imp.atoi           ; int atoi(const char *str)
  │      │    0x080484e1      83c410         add esp, 0x10
  │      │    0x080484e4      3d0df0feca     cmp eax, 0xcafef00d
  ```

- Now the easy part, convert this hex into decimal. I used `RapidTables` but you can use any tool you like. If you are using Rapid Tables you need the value with 9 digits, from `Decimal from signed 2's complement (9 digits)`

  ```
  cafef00d -> from HEX to Decimal

  3405705229 -> this is decimal version
  
  -889262067 -> Decimal from signed 2's complement (9 digits) WE NEED THIS

  ./crackme8 -889262067
  Access granted.
  flag{at_least_this_cafe_wont_leak_your_credit_card_numbers}
  ```

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `Understanding all the binaries, functions and some other concepts.`

- What did I learn?
  `I learned a lot, this was my first experience with reverse engineering and I loved it, now I want to get into it even more!`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
