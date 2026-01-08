## script
```
mov al, byte [0x404000]
mov bx, word [0x404000]
mov ecx, dword [0x404000]
mov rdx, qword [0x404000]
```
```
nasm -f elf64 16.asm && ld -s -o 16 16.o && /challenge/run 16
```
## flag
```
pwn.college{kQvUgProGAjuRR6INnp6Bp1vlU3.dRDMywSMwEzNzEzW}
```
