# Collision

## Source Code

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

unsigned long hashcode = 0x21DD09EC; // decimal: 568134124
unsigned long check_password(const char *p) {
    int *ip = (int *) p;
    int res = 0;
    for (int i = 0; i < 5; i++) {
        res += ip[i];
    }
    return res;
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("usage : %s [passcode]\n", argv[0]);
        return 0;
    }
    if (strlen(argv[1]) != 20) {
        printf("passcode length should be 20 bytes\n");
        return 0;
    }

    if (hashcode == check_password(argv[1])) {
        system("/bin/cat flag");
        return 0;
    } else
        printf("wrong passcode.\n");
    return 0;
}
```

If we run it with a passcode like AAAABBBBCCCCDDDDEEEE, we can look at the pattern in the `eax` (accumulator) register. 0x41414141 -> 0x42424242 -> 0x43434343 etc... So, we need 5 things (4 bytes each) that add up to 0x21DD09EC. Let's divided 0x21DD09EC with 5.

### Quick Python Calculation

```
t0thkr1s : ~
≫ python3
Python 3.7.6 (default, Dec 30 2019, 19:38:26)
[Clang 11.0.0 (clang-1100.0.33.16)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x21DD09EC / 5
113626824.8
>>> hex(113626824)
'0x6c5cec8'
>>> 0x21dd09ec - 0x6c5cec8
454507300
>>> hex(0x21dd09ec - 0x6c5cec8 * 4)
'0x6c5cecc'
>>> hex(0x6c5cec8 * 4 + 0x6c5cecc)
'0x21dd09ec'
>>>
```

### Byte Flipping

We're dealing with a little-endiann architecture, so we need to flip the order of the bytes.

```
./collision (python -c "print '\xc8\xce\xc5\x06' * 4 + '\xcc\xce\xc5\x06'")
```

## Building The Exploit

```python3
#!/usr/bin/env python3
from pwn import *

shell = ssh('col', 'pwnable.kr', password='guest', port=2222)
payload = ('\xc8\xce\xc5\x06' * 4 + '\xcc\xce\xc5\x06').encode('latin-1')
process = shell.process(executable='./col', argv=['col', payload])
log.success('The flag is: ' + process.recvall().decode().strip())
```
