## sscccharaappttt
```
from pwn import *

context.clear(arch='amd64')

code = asm("""
    mov rax, [rsp]
    add rax, [rsp + 8]
    add rax, [rsp + 16]
    add rax, [rsp + 24]
    shr rax, 2
    push rax
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## phlagg
```
pwn.college{YcP49D_Q65GHd5PIcI3TB0JfGUX.dlDMywSMwEzNzEzW}
```
