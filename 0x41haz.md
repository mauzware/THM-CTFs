## 0x41haz - TryHackMe Writeup

[Room Link](https://tryhackme.com/room/0x41haz)

<i>In this challenge, you are asked to solve a simple reversing solution. Download and analyze the binary to discover the password.

There may be anti-reversing measures in place!</i>

---

## 📌 General Information

**Room Difficulty**: Easy  <br>

**Room Type**: Reverse Engineering <br>

**Tools Used**: Radare<br>

**Author**: <br>

[<img align='center' src="https://github.com/mauzware/mauzware/blob/main/LOGO%20NEW.png"/>](https://github.com/mauzware)

[TryHackMe Profile](https://tryhackme.com/p/mauzinho) <br>
[GitHub Profile](https://github.com/mauzware)

---

## 🔍 Let's do it!

- First let's check the binary with `file` and `strings`.

  ```
  file 0x41haz.0x41haz
  0x41haz.0x41haz: ELF 64-bit MSB *unknown arch 0x3e00* (SYSV)
  ```

  ```
  strings 0x41haz.0x41haz       
  /lib64/ld-linux-x86-64.so.2
  gets
  exit
  puts
  strlen
  __cxa_finalize
  __libc_start_main
  libc.so.6
  GLIBC_2.2.5
  _ITM_deregisterTMCloneTable
  __gmon_start__
  _ITM_registerTMCloneTable
  u/UH
  2@@25$gfH
  sT&@f
  []A\A]A^A_
  =======================
  Hey , Can You Crackme ?
  =======================
  It's jus a simple binary 
  Tell Me the Password :
  Is it correct , I don't think so.
  Nope
  Well Done !!
  ;*3$"
  GCC: (Debian 10.3.0-9) 10.3.0
  .shstrtab
  .interp
  .note.gnu.build-id
  .note.ABI-tag
  .gnu.hash
  .dynsym
  .dynstr
  .gnu.version
  .gnu.version_r
  .rela.dyn
  .rela.plt
  .init
  .plt.got
  .text
  .fini
  .rodata
  .eh_frame_hdr
  .eh_frame
  .init_array
  .fini_array
  .dynamic
  .got.plt
  .data
  .bss
  .comment
  ```
  
- OK, so it requires the password, but this `file` output doesn't look good so I checked it with `hexedit`.
  When you open it with hexedit you will need to change the 6th byte from 02 to 01.

  ```00000000   7F 45 4C 46  02 02 01 00 -> 00000000   7F 45 4C 46  02 01 01 00```
  
- Now when you run `file` command it's a whole different story.

  ```
  file 0x41haz.0x41haz 
  0x41haz.0x41haz: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6c9f2e85b64d4f12b91136ffb8e4c038f1dc6dcd, for GNU/Linux 3.2.0, stripped
  ```

- Let's run the binary now.

  ```
  ./0x41haz.0x41haz 
  =======================
  Hey , Can You Crackme ?
  =======================
  It's jus a simple binary 
  
  Tell Me the Password :
  bimbo
  Is it correct , I don't think so.
  ```

- OK, pretty much what strings found, let's use Radare now.

  ```
  r2 -d ./0x41haz.0x41haz
  WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
  [0x7f4f44f885c0]> aaa
  [SNIP!]
  [0x7f4f44f885c0]> afl
  0x5646dd86c030    1      6 sym.imp.puts
  0x5646dd86c040    1      6 sym.imp.strlen
  0x5646dd86c050    1      6 sym.imp.gets
  0x5646dd86c060    1      6 sym.imp.exit
  0x5646dd86c070    1      6 sym.imp.__cxa_finalize
  0x5646dd86c080    1     42 entry0
  0x5646dd86efe0    1   4128 reloc.__libc_start_main
  0x5646dd86c165    8    219 main
  0x5646dd86c160    5     56 entry.init0
  0x5646dd86c120    5     50 entry.fini0
  [SNIP!]
  
  [0x7f4f44f885c0]> pdf @main
              ; DATA XREF from entry0 @ 0x5646dd86c09d(r)
  ┌ 219: int main (int argc, char **argv, char **envp);
  │ afv: vars(6:sp[0xc..0x48])
  │           0x5646dd86c165      55             push rbp
  │           0x5646dd86c166      4889e5         mov rbp, rsp
  │           0x5646dd86c169      4883ec40       sub rsp, 0x40
  │           0x5646dd86c16d      48b8324040..   movabs rax, 0x6667243532404032 ; '2@@25$gf'
  │           0x5646dd86c177      488945ea       mov qword [var_16h], rax
  │           0x5646dd86c17b      c745f27354..   mov dword [var_eh], 0x40265473 ; 'sT&@'
  │           0x5646dd86c182      66c745f64c00   mov word [var_ah], 0x4c ; 'L' ; 76
  │           0x5646dd86c188      488d3d790e..   lea rdi, str._nHey___Can_You_Crackme___n ; 0x5646dd86d008 ; "=======================\nHey , Can You Crackme ?\n======================="
  │           0x5646dd86c18f      e89cfeffff     call sym.imp.puts       ; int puts(const char *s)
  │           0x5646dd86c194      488d3db50e..   lea rdi, str.Its_jus_a_simple_binary__n ; 0x5646dd86d050 ; "It's jus a simple binary \n"
  │           0x5646dd86c19b      e890feffff     call sym.imp.puts       ; int puts(const char *s)
  │           0x5646dd86c1a0      488d3dc40e..   lea rdi, str.Tell_Me_the_Password_: ; 0x5646dd86d06b ; "Tell Me the Password :"
  │           0x5646dd86c1a7      e884feffff     call sym.imp.puts       ; int puts(const char *s)
  │           0x5646dd86c1ac      488d45c0       lea rax, [var_40h]
  │           0x5646dd86c1b0      4889c7         mov rdi, rax
  │           0x5646dd86c1b3      b800000000     mov eax, 0
  │           0x5646dd86c1b8      e893feffff     call sym.imp.gets       ; char *gets(char *s)
  │           0x5646dd86c1bd      488d45c0       lea rax, [var_40h]
  │           0x5646dd86c1c1      4889c7         mov rdi, rax
  │           0x5646dd86c1c4      e877feffff     call sym.imp.strlen     ; size_t strlen(const char *s)
  │           0x5646dd86c1c9      8945f8         mov dword [var_8h], eax
  │           0x5646dd86c1cc      837df80d       cmp dword [var_8h], 0xd
  │       ┌─< 0x5646dd86c1d0      7416           je 0x5646dd86c1e8
  │       │   0x5646dd86c1d2      488d3daf0e..   lea rdi, str.Is_it_correct___I_dont_think_so. ; 0x5646dd86d088 ; "Is it correct , I don't think so."
  │       │   0x5646dd86c1d9      e852feffff     call sym.imp.puts       ; int puts(const char *s)
  │       │   0x5646dd86c1de      bf00000000     mov edi, 0
  │       │   0x5646dd86c1e3      e878feffff     call sym.imp.exit       ; void exit(int status)
  │       └─> 0x5646dd86c1e8      c745fc0000..   mov dword [var_4h], 0
  │       ┌─< 0x5646dd86c1ef      eb34           jmp 0x5646dd86c225
  │      ┌──> 0x5646dd86c1f1      8b45fc         mov eax, dword [var_4h]
  │      ╎│   0x5646dd86c1f4      4898           cdqe
  │      ╎│   0x5646dd86c1f6      0fb65405ea     movzx edx, byte [rbp + rax - 0x16]
  │      ╎│   0x5646dd86c1fb      8b45fc         mov eax, dword [var_4h]
  │      ╎│   0x5646dd86c1fe      4898           cdqe
  │      ╎│   0x5646dd86c200      0fb64405c0     movzx eax, byte [rbp + rax - 0x40]
  │      ╎│   0x5646dd86c205      38c2           cmp dl, al
  │     ┌───< 0x5646dd86c207      7506           jne 0x5646dd86c20f
  │     │╎│   0x5646dd86c209      8345fc01       add dword [var_4h], 1
  │    ┌────< 0x5646dd86c20d      eb16           jmp 0x5646dd86c225
  │    │└───> 0x5646dd86c20f      488d3d940e..   lea rdi, str.Nope       ; 0x5646dd86d0aa ; "Nope"
  │    │ ╎│   0x5646dd86c216      e815feffff     call sym.imp.puts       ; int puts(const char *s)
  │    │ ╎│   0x5646dd86c21b      bf00000000     mov edi, 0
  │    │ ╎│   0x5646dd86c220      e83bfeffff     call sym.imp.exit       ; void exit(int status)
  │    │ ╎│   ; CODE XREFS from main @ 0x5646dd86c1ef(x), 0x5646dd86c20d(x)
  │    └──└─> 0x5646dd86c225      8b45fc         mov eax, dword [var_4h]
  │      ╎    0x5646dd86c228      3b45f8         cmp eax, dword [var_8h]
  │      └──< 0x5646dd86c22b      7cc4           jl 0x5646dd86c1f1
  │           0x5646dd86c22d      488d3d7b0e..   lea rdi, str.Well_Done___ ; 0x5646dd86d0af ; "Well Done !!"
  │           0x5646dd86c234      e8f7fdffff     call sym.imp.puts       ; int puts(const char *s)
  │           0x5646dd86c239      b800000000     mov eax, 0
  │           0x5646dd86c23e      c9             leave
  └           0x5646dd86c23f      c3             ret
  ```

- After checking the main function you can see some strings there.

  ```
  0x5646dd86c16d      48b8324040..   movabs rax, 0x6667243532404032 ; '2@@25$gf'
  0x5646dd86c17b      c745f27354..   mov dword [var_eh], 0x40265473 ; 'sT&@'
  0x5646dd86c182      66c745f64c00   mov word [var_ah], 0x4c ; 'L' ; 76
  ```

- So now I checked all of them, and well, got the password.

  ```
  $ ./0x41haz.0x41haz
  =======================
  Hey , Can You Crackme ?
  =======================
  It's jus a simple binary 
  
  Tell Me the Password :
  sT&@
  Is it correct , I don't think so.
                                                                                                                       
  $ ./0x41haz.0x41haz
  =======================
  Hey , Can You Crackme ?
  =======================
  It's jus a simple binary 
  
  Tell Me the Password :
  L
  Is it correct , I don't think so.

  $ ./0x41haz.0x41haz
  =======================
  Hey , Can You Crackme ?
  =======================
  It's jus a simple binary 
  
  Tell Me the Password :
  2@@25$gfsT&@
  Is it correct , I don't think so.
                                                                                                                       
  $ ./0x41haz.0x41haz
  =======================
  Hey , Can You Crackme ?
  =======================
  It's jus a simple binary 
  
  Tell Me the Password :
  2@@25$gfsT&@L
  Well Done !!
  ```

---

## 🏁 Flags

- **Flag**: `THM{2@@25$gfsT&@L}`

---

## 💬 Notes & Reflections

- What was challenging in this room for me?
  `This one was pretty basic honestly.`

- What did I learn?
  `More reverse engineering knowledge.`

- I would recommend this room or any type of CTF challenge to every Cyber Security enthusiast since it will sharpen your skills better than any book/tutorial/course or walkthrough rooms on many different platforms!

- Thank you for reading! Enjoy, Have Fun and Happy Hacking! 🤟

---
