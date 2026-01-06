## script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
and rdi, rsi
and rax, rdi
""")

p = process('/challenge/run')
p.send(code)
```
## flag
```
pwn.college{44lGtjlhqeSoH3abXbeSkDfAu3A.dFDMywSMwEzNzEzW}
```
