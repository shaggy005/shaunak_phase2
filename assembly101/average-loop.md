
```
Extracting binary code from provided ELF file...
Executing your code...
---------------- CODE ----------------
0x400000:	mov   	rcx, rsi
0x400003:	xor   	rax, rax
0x400006:	test  	rcx, rcx
0x400009:	je    	0x40001c
0x40000b:	add   	rax, qword ptr [rdi]
0x40000e:	add   	rdi, 8
0x400012:	dec   	rcx
0x400015:	jne   	0x40000b
0x400017:	cqo   	
0x400019:	idiv  	rsi
--------------------------------------
pwn.college{QoaSQnGej5ujyuGVC0nW9dcBOP6.dNTMywSMwEzNzEzW}
```
