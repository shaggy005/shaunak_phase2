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
