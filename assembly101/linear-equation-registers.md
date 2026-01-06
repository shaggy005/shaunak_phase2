## script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
imul rdi, rsi
add rdi, rdx
mov rax, rdi
""")

p = process('/challenge/run')
p.send(code)
```
## flag
```
pwn.college{0WGjZ1Vhus48X55UudSxwfEbAE5.dZTOxwSMwEzNzEzW}
```
