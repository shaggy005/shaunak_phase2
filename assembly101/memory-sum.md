## script
```
from pwn import *

context.clear(arch='amd64')

code = asm("""
    mov rax, [rdi]
    mov rbx, [rdi + 8]
    add rax, rbx
    mov [rsi], rax
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## flag
```
pwn.college{8pT5-Sa2E8wU52Bgtf0aFQSYubd.dZDMywSMwEzNzEzW}
```
