## script
```
from pwn import *

context.clear(arch='amd64')

code = asm("""
    mov rdi, 0x404000
    mov rax, [rdi]
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## flag
```
pwn.college{suAFnmbalN38LTkqFJcMQziNH31.QXyEDOzwSMwEzNzEzW
```
