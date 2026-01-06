## script
```
from pwn import *
context.update(arch='amd64')
code = asm("""
mov ah, 0x42
""")
p = process('/challenge/run')
p.write(code)
print(p.readall())
```
## flag
```
pwn.college{0xdcaQtyZ1Yr_Q6xh_yKNJOE9zJ.QXxEDOzwSMwEzNzEzW}
```
