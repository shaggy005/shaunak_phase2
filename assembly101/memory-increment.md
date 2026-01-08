## script
```
mov rax, [0x404000]
add qword [0x404000], 0x1337
```
```
nasm -f elf64 14.asm && ld -s -o 14 14.o && /challenge/run 14
```
## flag
```
pwn.college{kPz9m3Y4z-nZy-vqGS6_laSJpSe.dNDMywSMwEzNzEzW}
```
