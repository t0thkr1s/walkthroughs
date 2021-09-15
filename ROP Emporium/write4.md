# write4

On completing our usual checks for interesting strings and symbols in this binary we're confronted with the stark truth that our favourite string `/bin/cat flag.txt` is not present this time. Although you'll see later that there are other ways around this problem, such as resolving dynamically loaded libraries and using the strings present in those, we'll stick to the challenge goal which is learning how to get data into the target process's virtual address space via the magic of ROP.


## Read & Write

The important thing to realise is that ROP is just a form of arbitrary code execution and if we're creative we can leverage it to do things like write to or read from memory. The question is what mechanism are we going to use to solve this problem, is there any built-in functionality to do the writing or do we need to use gadgets? In this challenge we won't be using built-in functionality since that's too similar to the previous challenges, instead we'll be looking for gadgets that let us write a value to memory such as `mov [reg], reg`. Nonetheless it is possible to solve this challenge by leveraging functions like `fgets()` to write to memory locations of your choosing so it's worth trying to do it that way once you've solved it using the intended technique.


## What & Where

Perhaps the most important thing to consider in this challenge is __where__ we're going to write our string. Use rabin2 or readelf to check out the different sections of this binary and their permissions. Learn a little about ELF sections and their purpose. Consider how much space each section might give you to work with and whether corrupting the information stored at these locations will cause you problems later if you need some kind of stability from this binary.


## Decisions, decisions

Once you've figured out how to write your string into memory and where to write it, go ahead and call `system()` with its location as your only argument. Are you going to cat flag.txt or drop a shell with `/bin/sh`? Try to wrap some of your functionality in helper functions, if you can write a 4 or 8 byte value to a location in memory, can you craft a function (in python using pwntools for example) that takes a string and a memory location and returns a ROP chain that will write that string to your chosen location? Crafting templates like this will make your life much easier in the long run.


There are indeed three very different ways to solve the 64 bit version of this challenge, including the intended method. Built-in functionality will give you a win if you're willing to borrow a technique from the 'pivot' challenge and an oversight in how the `pwnme()` function was constructed can get you a shell in a single link chain 🤫

```
t0thkr1s@kali ~/Downloads> rabin2 -I write4
arch     x86
baddr    0x400000
binsz    7150
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
t0thkr1s@kali ~/Downloads> r2 write4
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
0x004008a0    1 2            sym.__libc_csu_fini
0x004008a4    1 9            sym._fini
0x00400830    4 101          sym.__libc_csu_init
0x00400746    1 111          main
0x00400630    1 6            sym.imp.setvbuf
0x004005a0    3 26           sym._init
[0x00400650]> s sym.usefulFunction
[0x00400807]> pdf
┌ 17: sym.usefulFunction ();
│           0x00400807      55             push rbp
│           0x00400808      4889e5         mov rbp, rsp
│           0x0040080b      bf0c094000     mov edi, str.bin_ls         ; 0x40090c ; "/bin/ls" ; const char *string
│           0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
│           0x00400815      90             nop
│           0x00400816      5d             pop rbp
└           0x00400817      c3             ret
[0x00400807]>
```

Checking the `sym.usefulFunction` reveals that there's a systemcall, but it calls `/bin/ls`. So the goal is to write `/bin/sh\x00` somewhere in the binary, and use the `system()` to execute the string.

## Checking Section Permissions

As you can see, we have read and write permissions on the `.data` section.

```
0x00400807]> iS
[Sections]

nth paddr        size vaddr       vsize perm name
―――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000000    0x0 0x00000000    0x0 ---- 
1   0x00000238   0x1c 0x00400238   0x1c -r-- .interp
2   0x00000254   0x20 0x00400254   0x20 -r-- .note.ABI_tag
3   0x00000274   0x24 0x00400274   0x24 -r-- .note.gnu.build_id
4   0x00000298   0x30 0x00400298   0x30 -r-- .gnu.hash
5   0x000002c8  0x120 0x004002c8  0x120 -r-- .dynsym
6   0x000003e8   0x74 0x004003e8   0x74 -r-- .dynstr
7   0x0000045c   0x18 0x0040045c   0x18 -r-- .gnu.version
8   0x00000478   0x20 0x00400478   0x20 -r-- .gnu.version_r
9   0x00000498   0x60 0x00400498   0x60 -r-- .rela.dyn
10  0x000004f8   0xa8 0x004004f8   0xa8 -r-- .rela.plt
11  0x000005a0   0x1a 0x004005a0   0x1a -r-x .init
12  0x000005c0   0x80 0x004005c0   0x80 -r-x .plt
13  0x00000640    0x8 0x00400640    0x8 -r-x .plt.got
14  0x00000650  0x252 0x00400650  0x252 -r-x .text
15  0x000008a4    0x9 0x004008a4    0x9 -r-x .fini
16  0x000008b0   0x64 0x004008b0   0x64 -r-- .rodata
17  0x00000914   0x44 0x00400914   0x44 -r-- .eh_frame_hdr
18  0x00000958  0x134 0x00400958  0x134 -r-- .eh_frame
19  0x00000e10    0x8 0x00600e10    0x8 -rw- .init_array
20  0x00000e18    0x8 0x00600e18    0x8 -rw- .fini_array
21  0x00000e20    0x8 0x00600e20    0x8 -rw- .jcr
22  0x00000e28  0x1d0 0x00600e28  0x1d0 -rw- .dynamic
23  0x00000ff8    0x8 0x00600ff8    0x8 -rw- .got
24  0x00001000   0x50 0x00601000   0x50 -rw- .got.plt
25  0x00001050   0x10 0x00601050   0x10 -rw- .data
26  0x00001060    0x0 0x00601060   0x30 -rw- .bss
27  0x00001060   0x34 0x00000000   0x34 ---- .comment
28  0x00001ae2  0x10c 0x00000000  0x10c ---- .shstrtab
29  0x00001098  0x768 0x00000000  0x768 ---- .symtab
30  0x00001800  0x2e2 0x00000000  0x2e2 ---- .strtab
```

It looks the binary is passing `/bin/ls` to the `system()` function call.

```
[0x004007b5]> s sym.usefulFunction
[0x00400807]> pdf
┌ 17: sym.usefulFunction ();
│           0x00400807      55             push rbp
│           0x00400808      4889e5         mov rbp, rsp
│           0x0040080b      bf0c094000     mov edi, str.bin_ls         ; 0x40090c ; "/bin/ls" ; const char *string
│           0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
│           0x00400815      90             nop
│           0x00400816      5d             pop rbp
└           0x00400817      c3             ret
[0x00400807]>
```

Now we have to find gadgets that will allow us to place that string into the section. I’m going to use `ropper` once again.


```
t0thkr1s@darlene ~/Downloads> ropper -f write4 --search 'pop rdi'
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop rdi

[INFO] File: write4
0x0000000000400893: pop rdi; ret; 

t0thkr1s@darlene ~/Downloads>
```

But what exactly are we looking for? Firstly, we have to find a gadget that will place value at memory address. In assembly this operation looks like this `MOV [R0], R1`, which directly translates to move value from register `R1` to memory at address held by register `R0`.
The best candidate would gadget at address `0x0000000000400820` which is `mov qword ptr [r14], r15; ret;` So we can now place a value into memory, but we also need to place values into these registers with `pop`, which we can do with this gadget `pop r14; pop r15; ret;` at addtess `0x0000000000400890`. Now to finally finish collecting addresses, we have to come back and get the address of system() and in order to place the address of the string as the argument to this call, pop rdi will be needed. This gadget can be found at address 0x0000000000400893.

Great, now we can start writing our exploit.

Use `pop` gadget to place the address of our string, and string in registers
Use `mov` gadget to place the string into the memory at provided address
Use `pop rdi` gadget to place the address of the string into register.
Call address of `system()` which uses `rdi` as the argument register holding address of our string.

