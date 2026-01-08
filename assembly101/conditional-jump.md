Extracting binary code from provided ELF file...
Executing your code...
---------------- CODE ----------------
0x400000:	cmp   	dword ptr [rdi], 0x7f454c46
0x400006:	je    	0x400021
0x400008:	cmp   	dword ptr [rdi], 0x5a4d
0x40000e:	je    	0x40002c
0x400010:	mov   	eax, dword ptr [rdi + 4]
0x400013:	mov   	ebx, dword ptr [rdi + 8]
0x400016:	mov   	ecx, dword ptr [rdi + 0xc]
0x400019:	imul  	eax, ebx
0x40001c:	imul  	eax, ecx
0x40001f:	jmp   	0x400035
0x400021:	mov   	eax, dword ptr [rdi + 4]
0x400024:	add   	eax, dword ptr [rdi + 8]
0x400027:	add   	eax, dword ptr [rdi + 0xc]
0x40002a:	jmp   	0x400035
0x40002c:	mov   	eax, dword ptr [rdi + 4]
0x40002f:	sub   	eax, dword ptr [rdi + 8]
0x400032:	sub   	eax, dword ptr [rdi + 0xc]
--------------------------------------
pwn.college{Qiy71TIBK5h39R4uHzSJkkNc0HC.dFTMywSMwEzNzEzW}
