Extracting binary code from provided ELF file...
Executing your code...
---------------- CODE ----------------
0x400000:	push  	rbp
0x400001:	push  	rbx
0x400002:	push  	r12
0x400004:	mov   	rbx, rdi
0x400007:	xor   	r12, r12
0x40000a:	test  	rbx, rbx
0x40000d:	je    	0x40002f
0x40000f:	movzx 	rdi, byte ptr [rbx]
0x400013:	test  	dil, dil
0x400016:	je    	0x40002f
0x400018:	cmp   	dil, 0x5a
0x40001c:	ja    	0x40002a
0x40001e:	mov   	eax, 0x403000
0x400023:	call  	rax
0x400025:	mov   	byte ptr [rbx], al
0x400027:	inc   	r12
0x40002a:	inc   	rbx
0x40002d:	jmp   	0x40000f
0x40002f:	mov   	rax, r12
0x400032:	pop   	r12
0x400034:	pop   	rbx
0x400035:	pop   	rbp
0x400036:	ret   	
--------------------------------------
pwn.college{gJ9VMuTTwsO7SYvGqanEzQDlz5_.dVTMywSMwEzNzEzW}
