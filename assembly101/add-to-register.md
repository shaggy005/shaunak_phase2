## python script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
    add rdi, 0x331337
""")

p = process('/challenge/run')
p.send(code)
```
## flag
```
pwn.college{gDWpaa_Qg052svX2ki318mgCCqJ.dVTOxwSMwEzNzEzW}
```
