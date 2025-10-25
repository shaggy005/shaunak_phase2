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
- none

## Resources:

- 

