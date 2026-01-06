## script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
shr rdi, 0x20
mov al, dil
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## flag 
```
pwn.college{sSYJq_8B3cimmm-de4wJ3Gzfgtc.dBDMywSMwEzNzEzW}
```
