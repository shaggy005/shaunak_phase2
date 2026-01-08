Extracting binary code from provided ELF file...
Executing your code...
---------------- CODE ----------------
0x400000:	xor   	rax, rax
0x400003:	test  	rdi, rdi
0x400006:	je    	0x400013
0x400008:	cmp   	byte ptr [rdi + rax], 0
0x40000c:	je    	0x400013
0x40000e:	inc   	rax
0x400011:	jmp   	0x400008
--------------------------------------
pwn.college{8w-GgGSQKKRg1or54bui3U2yUu8.dRTMywSMwEzNzEzW}
