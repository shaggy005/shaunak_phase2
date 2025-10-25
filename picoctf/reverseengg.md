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

# 2. ARMssembly 1

> For what argument does this program print `win` with variables 68, 2 and 3? File: chall_1.S Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})

## Solution:

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

# 2. Vault Door 3

> This vault uses for-loops and byte arrays. The source code for this vault is here: VaultDoor3.java

## Solution:

- 

```

```

## Flag:

```
picoCTF{}
```

## Concepts learnt:

- 

## Notes:
- 

## Resources:

- none
