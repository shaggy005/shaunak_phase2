just wrote a python script in nano to set the register to the given value and ran it

```
from pwn import *
context.update(arch='amd64')
code = asm("""
mov rdi, 0x1337
""")
p = process('/challenge/run')
p.write(code)
print(p.readall())
```
```
hacker@assembly-crash-course~set-register:~$ nano script.py
hacker@assembly-crash-course~set-register:~$ python script.py 
[+] Starting local process '/challenge/run': pid 155
[+] Receiving all data: Done (571B)
[*] Process '/challenge/run' stopped with exit code 0 (pid 155)
b'\nIn this level you will be working with registers. You will be asked to modify\nor read from registers.\n\n\nIn this level you will work with registers! Please set the following:\n  rdi = 0x1337\n\nYou ran me without an argument. You can re-run with `/challenge/run /path/to/your/elf` to input an ELF file, or just give me your assembled and extracted code in bytes (up to 0x1000 bytes): \nExecuting your code...\n---------------- CODE ----------------\n0x400000:\tmov   \trdi, 0x1337\n--------------------------------------\npwn.college{g7INDIGagOJ_b5qQRKYfl3Ri4ez.dRTOxwSMwEzNzEzW}\n\n'
hacker@assembly-crash-course~set-register:~$ 
```
## Flag
```
pwn.college{g7INDIGagOJ_b5qQRKYfl3Ri4ez.dRTOxwSMwEzNzEzW}
```
