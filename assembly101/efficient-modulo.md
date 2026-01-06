## script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
    mov al, dil
    mov bx, si
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## flag
```
pwn.college{412gX6Xq9XsgM7bojtiWt1eNTG7.dlTOxwSMwEzNzEzW}
```
