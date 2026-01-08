Extracting binary code from provided ELF file...
Executing your code...
---------------- CODE ----------------
0x400000:	push  	rbp
0x400001:	mov   	rbp, rsp
0x400004:	sub   	rsp, 0x200
0x40000b:	xor   	rcx, rcx
0x40000e:	mov   	r8, rsp
0x400011:	mov   	qword ptr [r8], 0
0x400018:	add   	r8, 8
0x40001c:	add   	rcx, 8
0x400020:	cmp   	rcx, 0x200
0x400027:	jl    	0x400011
0x400029:	xor   	rcx, rcx
0x40002c:	cmp   	rcx, rsi
0x40002f:	jge   	0x40004a
0x400031:	movzx 	rax, byte ptr [rdi + rcx]
0x400036:	mov   	r8, rax
0x400039:	shl   	r8, 1
0x40003c:	neg   	r8
0x40003f:	inc   	word ptr [rbp + r8 - 2]
0x400045:	inc   	rcx
0x400048:	jmp   	0x40002c
0x40004a:	xor   	rcx, rcx
0x40004d:	xor   	rbx, rbx
0x400050:	xor   	rdx, rdx
0x400053:	cmp   	rcx, 0x100
0x40005a:	je    	0x40007b
0x40005c:	mov   	r8, rcx
0x40005f:	shl   	r8, 1
0x400062:	neg   	r8
0x400065:	movzx 	rax, word ptr [rbp + r8 - 2]
0x40006b:	cmp   	ax, bx
0x40006e:	jle   	0x400076
0x400070:	mov   	bx, ax
0x400073:	mov   	rdx, rcx
0x400076:	inc   	rcx
0x400079:	jmp   	0x400053
0x40007b:	mov   	rax, rdx
0x40007e:	leave 	
0x40007f:	ret   	
--------------------------------------
pwn.college{kQFgT3yJOui062Q2EoD9LOUS3pu.dZTMywSMwEzNzEzW}
