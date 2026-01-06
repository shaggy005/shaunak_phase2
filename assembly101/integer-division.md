## script
```
from pwn import *

context.arch = 'amd64'

code = asm("""
mov rax, rdi
div rsi
""")

p = process('/challenge/run')
p.send(code)
p.shutdown('send')
print(p.recvall(timeout=2))
```
## flag
```
pwn.college{I1tWAAmYkYQceNChpnyPMzLITai.ddTOxwSMwEzNzEzW}
```
