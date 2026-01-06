## script
```
from pwn import *
context.update(arch='amd64')
code = asm("""
mov rax, rdi
div rsi
mov rax, rdx
""")
p = process('/challenge/run')
p.write(code)
print(p.readall())
```
## flag
```
pwn.college{QVDjF4p4vmz56WUCFOW2vlcUW0y.dhTOxwSMwEzNzEzW}
```
