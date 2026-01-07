## script
```
and rdi, 1
xor rax, rax
xor rdi, 1
or rax, rdi
```
```
hackhacker@assembly-crash-course~check-even:~$ nasm -f elf64 11.asm && ld -s -o 11 11.o && /challenge/run 11
```
## flag
```
pwn.college{sKwmQL9rxgqyAV-seBwPSusmsRW.dJDMywSMwEzNzEzW}
```
