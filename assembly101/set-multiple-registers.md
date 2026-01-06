## python script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
    mov rax, 0x1337
    mov r12, 0xCAFED00D1337BEEF
    mov rsp, 0x31337
""")

p = process('/challenge/run')
p.send(code)
```
## flag
```
pwn.college{MsQgkWORgQMd5WR7sx1iTrCJQdx.QXwEDOzwSMwEzNzEzW}
```
