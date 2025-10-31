# 1. GDB baby step 1

> Can you figure out what is in the eax register at the end of the main function? Put your answer in the picoCTF flag format: picoCTF{n} where n is the contents of the eax register in the decimal number base. If the answer was 0x11 your flag would be picoCTF{17}.
Disassemble debugger0_a

## Solution:

- downloaded the file debugger0_a
- opened it in ghidra coderunner
- searched for the main fuction

```
undefined8 main(void)

{
  return 0x86342;
}

```
- converting 0x86342 to decimal we get 549698
- hence the flag is picoCTF{549698}

## Flag:

```
picoCTF{549698}
```

## Concepts learnt:

- how to use ghidra decompiler
- searching functions inside decompiled code

## Notes:
- none

## Resources:

- the malware analysis workshop

***

# 2. ARMssembly 1

> For what argument does this program print `win` with variables 68, 2 and 3? File: chall_1.S Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})

## Solution
- We’re given a disassembled ARM64 source file chall_1.S
  ```
    	.arch armv8-a
  	.file	"chall_1.c"
  	.text
  	.align	2
  	.global	func
  	.type	func, %function
  func:
  	sub	sp, sp, #32
  	str	w0, [sp, 12]
  	mov	w0, 68
  	str	w0, [sp, 16]
  	mov	w0, 2
  	str	w0, [sp, 20]
  	mov	w0, 3
  	str	w0, [sp, 24]
  	ldr	w0, [sp, 20]
  	ldr	w1, [sp, 16]
  	lsl	w0, w1, w0
  	str	w0, [sp, 28]
  	ldr	w1, [sp, 28]
  	ldr	w0, [sp, 24]
  	sdiv	w0, w1, w0
  	str	w0, [sp, 28]
  	ldr	w1, [sp, 28]
  	ldr	w0, [sp, 12]
  	sub	w0, w1, w0
  	str	w0, [sp, 28]
  	ldr	w0, [sp, 28]
  	add	sp, sp, 32
  	ret
  	.size	func, .-func
  	.section	.rodata
  	.align	3
  .LC0:
  	.string	"You win!"
  	.align	3
  .LC1:
  	.string	"You Lose :("
  	.text
  	.align	2
  	.global	main
  	.type	main, %function
  main:
  	stp	x29, x30, [sp, -48]!
  	add	x29, sp, 0
  	str	w0, [x29, 28]
  	str	x1, [x29, 16]
  	ldr	x0, [x29, 16]
  	add	x0, x0, 8
  	ldr	x0, [x0]
  	bl	atoi
  	str	w0, [x29, 44]
  	ldr	w0, [x29, 44]
  	bl	func
  	cmp	w0, 0
  	bne	.L4
  	adrp	x0, .LC0
  	add	x0, x0, :lo12:.LC0
  	bl	puts
  	b	.L6
  .L4:
  	adrp	x0, .LC1
  	add	x0, x0, :lo12:.LC1
  	bl	puts
  .L6:
  	nop
  	ldp	x29, x30, [sp], 48
  	ret
  	.size	main, .-main
  	.ident	"GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
  	.section	.note.GNU-stack,"",@progbits

  ```
- It defines two functions — func and main
- this is the important part in the assembly (ive added comments to explain what they do)
```
.global func
func:
    sub sp, sp, #32
    str w0, [sp, 12]      // store argument (input)
    mov w0, 68
    str w0, [sp, 16]
    mov w0, 2
    str w0, [sp, 20]
    mov w0, 3
    str w0, [sp, 24]

    ldr w0, [sp, 20]      // w0 = 2
    ldr w1, [sp, 16]      // w1 = 68
    lsl w0, w1, w0        // w0 = 68 << 2 = 272
    str w0, [sp, 28]

    ldr w1, [sp, 28]      // w1 = 272
    ldr w0, [sp, 24]      // w0 = 3
    sdiv w0, w1, w0       // w0 = 272 / 3 = 90 (integer division)
    str w0, [sp, 28]

    ldr w1, [sp, 28]      // w1 = 90
    ldr w0, [sp, 12]      // w0 = input
    sub w0, w1, w0        // w0 = 90 - input
    str w0, [sp, 28]

    ldr w0, [sp, 28]
    add sp, sp, 32
    ret

```
- so functionally result = ((a<<b)/c)-input
- then it checks if function(input)==0
- then it pronts you win
- hence func(input) = 90 - input and that must be 0
- so our input should be 90
- 90 is 0x5a in hex, and padding that according to the instructions we get 0000005a
- and we have our flag
## Flag
```
picoCTF{0000005a}

```


# Alternate Solution for ARMssembly 1:

- the .s extension means that the file is an assembly file
- i uploaded it to an online assembly to c converter and got the following code

```
#include <stdio.h>
#include <stdlib.h>

int func(int input) {
    int a = input;
    int b = 68;
    int c = 2;
    int d = 3;

    int temp = b << c;
    temp = temp / d;
    temp = temp - a;

    return temp;
}

int main(int argc, char *argv[]) {
    int val = atoi(argv[1]);
    int result = func(val);

    if (result == 0) {
        puts("You win!");
    } else {
        puts("You Lose :(");
    }

    return 0;
}

```
this means that temp = 68 * 2 * 2 = 272, then we divide temp by 3, temp becomes 90, then temp becomes temp-input.
so if we take input 90, the fuction returns 0. and in the main function we can see that it prints win when the fuction returns 0. hence 90 is the argument we need and 90 is 0x5a in hex and according to instructions, by padding it in 32 bits, our flag is picoCTF{0000005a}

## Flag:

```
picoCTF{0000005a}
```

## Concepts learnt:

- decompilation and following logic

## Notes:
- tried messing around with ghidra, couldnt get it to work, hence had to use an online tool https://www.codeconvert.ai/assembly-to-c-converter
- need to learn ghidra properly

## Resources:

- none

***

# 3. Vault Door 3

> This vault uses for-loops and byte arrays. The source code for this vault is here: VaultDoor3.java

## Solution:

- here we have a java file VaultDoor3.java with this logic

```
public boolean checkPassword(String password) {
    if (password.length() != 32) {
        return false;
    }
    char[] buffer = new char[32];
    int i;
    for (i=0; i<8; i++) {
        buffer[i] = password.charAt(i);
    }
    for (; i<16; i++) {
        buffer[i] = password.charAt(23-i);
    }
    for (; i<32; i+=2) {
        buffer[i] = password.charAt(46-i);
    }
    for (i=31; i>=17; i-=2) {
        buffer[i] = password.charAt(i);
    }
    String s = new String(buffer);
    return s.equals("jU5t_a_sna_3lpm18g947_u_4_m9r54f");
}

```
- this means it will check the final string against 
```
jU5t_a_sna_3lpm18g947_u_4_m9r54f
```
- so all i had to do now is trace back the string operations and i would have the input string that matches up with "jU5t_a_sna_3lpm18g947_u_4_m9r54f"
- the first for loop just copies the first 8 characters
- the second loop reverses the next 8 characters
- the third and fourth loop interleave the next 16 characters
- tracing back the operations on "jU5t_a_sna_3lpm18g947_u_4_m9r54f" i got the input string required
```
jU5t_a_s1mpl3_an4gr4m_4_u_79958f
```

## Flag:

```
picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_79958f}
```

## Concepts learnt:

- reading java code i guess, it wasnt too hard tbh, just a whole lotta syntax

## Notes:
- none

## Resources:

- none
***
# ARMssembly 0
## Solution
- Ill explain whats happening line by line in the assembly
```
	.arch armv8-a
	.file	"chall.c"

	.text
	.align	2
	.global	func1
	.type	func1, %function

func1:
	sub	sp, sp, #16          // allocates 16 bytes on the stack
	str	w0, [sp, 12]         // stores first argument (arg1) at sp+12
	str	w1, [sp, 8]          // stores second argument (arg2) at sp+8
	ldr	w1, [sp, 12]         // loads arg1 into w1
	ldr	w0, [sp, 8]          // loads arg2 into w0
	cmp	w1, w0               // compares arg1 and arg2
	bls	.L2                  // if arg1 <= arg2, branches to .L2
	ldr	w0, [sp, 12]         // else, w0 = arg1 (returns arg1)
	b	.L3                   // jumps to .L3 to return
.L2:
	ldr	w0, [sp, 8]          // w0 = arg2 (returns arg2 if arg1 <= arg2)
.L3:
	add	sp, sp, 16          
	ret                     
	.size	func1, .-func1  

//----------------------------------------------------

	.section	.rodata
	.align	3
.LC0:
	.string	"Result: %ld\n"      // printf format string

//----------------------------------------------------

	.text
	.align	2
	.global	main
	.type	main, %function

main:
	stp	x29, x30, [sp, -48]!  
	add	x29, sp, 0            
	str	x19, [sp, 16]        
	str	w0, [x29, 44]         // stores argc
	str	x1, [x29, 32]         // stores argv

	// ----- get argv[1] -----
	ldr	x0, [x29, 32]         // loads argv (char **)
	add	x0, x0, 8             // points to argv[1]
	ldr	x0, [x0]              // loads argv[1] (string)
	bl	atoi                  // converts to integer
	mov	w19, w0               // stores as first integer argument in w19

	// ----- get argv[2] -----
	ldr	x0, [x29, 32]         // loads argv again
	add	x0, x0, 16            // points to argv[2]
	ldr	x0, [x0]              // loads argv[2] (string)
	bl	atoi                  // converts to integer
	mov	w1, w0                // w1 = second integer argument
	mov	w0, w19               // w0 = first integer argument

	// ----- call func1(a,b) -----
	bl	func1                 // returns max(a,b) (unsigned compare)

	// ----- print result -----
	mov	w1, w0                // moves return value into w1 (printf arg)
	adrp	x0, .LC0              // load address of format string
	add	x0, x0, :lo12:.LC0
	bl	printf                // print "Result: <value>"

	mov	w0, 0                
	ldr	x19, [sp, 16]         
	ldp	x29, x30, [sp], 48    
	ret                       

	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
	.section	.note.GNU-stack,"",@progbits

```
- So effectively:
  ```
  int a = atoi(argv[1]);
  int b = atoi(argv[2]);
  printf("Result: %ld\n", func1(a,b));
  ```
- a = 3854998744 b = 915131509
- Unsigned compare → 3854998744 > 915131509
- Return 3854998744 = e5c69cd8
## FLag
```
picoCTF{e5c69cd8}
```

  
