## skript
```
from pwn import *

context.clear(arch='amd64')

code = asm("""
    push rdi
    push rsi
    pop  rdi
    pop  rsi
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## fahalagaagggg
```
pwn.college{sBeZGABIhD1Dzpc82VLJK2NV5CX.dhDMywSMwEzNzEzW}
```
