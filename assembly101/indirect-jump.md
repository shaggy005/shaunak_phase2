```
cmp rdi, 3
ja some
jmp [rsi + rdi*8]
some: 
    jmp [rsi + 4*8]
```

```
Extracting binary code from provided ELF file...
Executing your code...
---------------- CODE ----------------
0x400000:	cmp   	rdi, 3
0x400004:	jbe   	0x40000b
0x400006:	mov   	edi, 4
0x40000b:	jmp   	qword ptr [rsi + rdi*8]
--------------------------------------
Completed test 10
Completed test 20
Completed test 30
Completed test 40
Completed test 50
Completed test 60
Completed test 70
Completed test 80
Completed test 90
Completed test 100
pwn.college{4VJ1tSKkk1vFUPzG_FBkhTWzz6_.dJTMywSMwEzNzEzW}
```
