## script
```
from pwn import *

context.clear(arch='amd64')

code = asm("""
    mov [0x404000], rax
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')          # tell the program we're done
print(p.recvall(timeout=2))
```

## flag
```
pwn.college{kyrt0cZVUgW42JQR-zKHHZsaw62.QXzEDOzwSMwEzNzEzW}
```
