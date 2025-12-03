# 1. JoyDivision
First i decompile it
So the program gives us a weird looking flag.txt after it runs, but that file is not the real flag. It’s some transformed/encrypted version of the flag. I looked at the code and basically the program messes with the original flag in two ways:
- It flips the bytes (like NOT and XOR)
- It expands the bytes three times, each time doubling the size

So to get back the actual flag I need to do the same thing but in reverse.

### Understanding what expand() does

expand() takes each byte and splits it into two bytes, mixing the upper nibble (top 4 bits) and the lower nibble (bottom 4 bits).
It also switches how it mixes them every time (so first byte is done one way, second byte is done another way, and so on).

Because the program calls expand() three times:

1st time: 1 → 2 bytes

2nd time: 2 → 4 bytes

3rd time: 4 → 8 bytes

So final flag file is 8× the original length.

To undo it, I must run a function that does the opposite (I called mine rx() for reverse expand).
Each rx() halves the size.

### Understanding flipBits()

Before the expanding happens, the program does flipBits() on every byte of the real flag.

flipBits alternates between two actions:

On byte 0 → do ~byte (NOT)

On byte 1 → do byte ^ key (XOR with key = 105)

On byte 2 → do NOT again

On byte 3 → do XOR again (key increases by 32 each time)

So to undo it:

If it's a NOT byte → I do NOT again

If it’s an XOR byte → I XOR with the same key again

Since XOR is symmetrical (XOR twice cancels out), it works nicely.

### Decoding the Base64

The flag file the binary made is binary data, so I need to base64 decode it safely:
```
base64 -D -i flag.txt.b64 -o flag.bin
```

Now flag.bin contains the actual encrypted bytes.

✔ Step 4: Running the Python Script

I wrote a small Python script with:

a function to reverse expand

a loop to reverse flipBits

print the result

Here’s the  script:
```
d=open("flag.bin","rb").read()

def rx(b):
    o=bytearray(len(b)//2)
    t=0
    for i in range(len(o)):
        x=b[2*i];y=b[2*i+1]
        o[i]=(x&15)|(y&240) if t==0 else (x&240)|(y&15)
        t=1-t
    return bytes(o)

a=rx(d)
a=rx(a)
a=rx(a)

o=bytearray(len(a))
t=0;k=105
for i in range(len(a)):
    if t==0:
        o[i]=(~a[i])&255
    else:
        o[i]=a[i]^k
        k=(k+32)&255
    t=1-t

print(o.decode("utf-8","ignore"))
```
### The Output
```
shaunak@Shaunaks-MacBook-Pro JoyDivision % base64 -i flag.txt -o flag.txt.b64

shaunak@Shaunaks-MacBook-Pro JoyDivision % base64 -D -i flag.txt.b64 -o flag.bin 

shaunak@Shaunaks-MacBook-Pro JoyDivision % python3 sol.py
sunshine{C3A5ER_CR055ED_TH3_RUB1C0N}
```
Running the script prints the flag:
```
sunshine{C3A5ER_CR055ED_TH3_RUB1C0N}
```


### Notes

- I didn’t need to understand every tiny bit of the C code — just the order of operations.

- Expand grows the data. Reverse expand shrinks it.

- FlipBits alternates NOT and XOR. Undo them in the same order.

- XOR is nice because doing XOR twice with the same value gives back the original.

- Base64 was important so the bytes don’t get corrupted.


# 2. Worthy Knight
First I decompile it
It asks for a 10 letter password.
Then it checks each pair of 2 letters.

So the string is like:
```
s[0] s[1]  s[2] s[3]  s[4] s[5]  s[6] s[7]  s[8] s[9]
```

It checks 5 pairs.

### Pair 1 (s[0], s[1])

The code says:
``
(s[0] ^ s[1]) == 36
s[1] == 106 ('j')
``

So:
``
s[1] = 'j'
s[0] = 36 ^ 106 = 'N'
``

So first pair is:

**Nj**

### Pair 2 (s[2], s[3])

The code says:
```
(s[3] ^ s[2]) == 56
s[3] == 'S'
```

So:
```
s[3] = 'S'
s[2] = 56 ^ 'S' = 'k'
```

So second pair is:

**kS**

### Pair 3

This part was confusing but basically:

It takes s[5] + s[4]

Calculates MD5

Compares it to a fixed MD5 string.

So I brute-forced all combinations of letters (uppercase + lowercase).
Very small set, only 52×52 tries.
```
import hashlib
import string

t="33a3192ba92b5a4803c9a9ed70ea5a9c"
l=string.ascii_letters

for a in l:
    for b in l:
        if a.isupper()==b.isupper():
            continue
        x=(b+a).encode()
        if hashlib.md5(x).hexdigest()==t:
            print(a,b)
            exit()
```

The correct match was:
```
s[4] = 'f'
s[5] = 'T'
```

So third pair is:

***fT***

### Pair 4 (s[6], s[7])

The code says:
```
(s[7] ^ s[6]) == 56
s[7] == 'a'
```

So:
```
s[7] = 'a'
s[6] = 56 ^ 'a' = 'Y'
```

So fourth pair is:

**Ya**

### Pair 5 (s[8], s[9])

The code says:
```
(s[8] ^ s[9]) == 32
s[9] == 'i'
```

So:
```
s[9] = 'i'
s[8] = 32 ^ 'i' = 'I'
```

So last pair is:

**Ii**

### final Incantation

Putting all pairs together:

**Nj kS fT Ya Ii**


So the final 10-letter string is:
**NjkSfTYaIi**

The program prints the flag:

**KCTF{NjkSfTYaIi}**

# 3. time
I opened the binary in ida, and read the pseudocode.

inside main(), i saw that the program is a number-guessing game that generates a random number using srand(time(0)) and then rand(), asks for a guess and then prints the secret number afterwards if the guess is wrong, else it prints the flag from flag.txt

the important part of the function looked like this:
```
if (v6 == v5)
{
    puts("You won. Guess was right! Here's your flag:");
    giveFlag();
}
else
{
    puts("Sorry. Try again, wrong guess!");
}
```

v6 is the random number generated using rand(), and v5 is the number the user enters.
the flag printing happens inside giveFlag(), which simply opens: **/home/h3/flag.txt**
and prints whatever is inside it.
so the logic is:
if I guess the random number, i get the flag
if not, i don't get the flag (wow surprise)

***but the number is based on the current time and changes every run, so brute-forcing it or predicting it manually is basically impossible***
then I looked at the assembly for the if statement. The important part was:
```
cmp [rbp+var_C], eax
jnz short loc_fail
```
since I wanted to always win, I decided to patch the binary by removing this conditional jump.
in IDA, I replaced jnz with two nops.
nop is basically a null instruction
```
cmp [rbp+var_C], eax
nop
nop
```
this means the program will always go to the “You won” branch and call giveFlag().
After patching the bytes and exporting the binary, I ran it again.
Now it instantly prints:
```
root@c05751c9661d:/home# ./time
Welcome to the number guessing game!
I'm thinking of a number. Can you guess it?
Guess right and you get a flag!
Enter your number: 123
Your guess was 123.
Looking for 1277730927.
You won. Guess was right! Here's your flag:
Flag file not found!  Contact an H3 admin for assistance.
root@c05751c9661d:/home# 
```
This confirms the patch worked because the win message appears every time.

### Notes
also i had to deal with a lot of compatibilty bs because im on a mac and the binary wont run on my machine or on my kali vm either, had to make a docker x86 environment to run and test the binaries

# 5. Dusty
## dust_noob
- opened this in ida 
- never dissassembled a rust binary so never did i expect so many functions. finally after some yt tutorials realised that here the main function is supposed to be called shinyclean or something
- opened that function and generated pseudocode for it in ida to understand it faster
```
__int64 shinyclean::main::h4b15dd54e331d693()
{
  __int64 v1; // [rsp+8h] [rbp-100h]
  _BYTE s[23]; // [rsp+2Ah] [rbp-DEh] BYREF
  _BYTE v3[22]; // [rsp+41h] [rbp-C7h] BYREF
  _BYTE v4[9]; // [rsp+57h] [rbp-B1h] BYREF
  _BYTE v5[48]; // [rsp+60h] [rbp-A8h] BYREF
  _QWORD v6[4]; // [rsp+90h] [rbp-78h] BYREF
  _BYTE v7[48]; // [rsp+B0h] [rbp-58h] BYREF
  _BYTE *v8; // [rsp+E0h] [rbp-28h]
  void *v9; // [rsp+E8h] [rbp-20h]
  _BYTE *v10; // [rsp+F0h] [rbp-18h]
  void *v11; // [rsp+F8h] [rbp-10h]
  _BYTE *v12; // [rsp+100h] [rbp-8h]

  memset(s, 0, sizeof(s));
  qmemcpy(v3, "{^HX|kyDym", 10);
  v3[10] = 12;
  v3[11] = 12;
  v3[12] = 96;
  v3[13] = 124;
  v3[14] = 11;
  v3[15] = 109;
  v3[16] = 96;
  v3[17] = 104;
  v3[18] = 11;
  v3[19] = 10;
  v3[20] = 119;
  v3[21] = 30;
  strcpy(v4, "B");
  v4[2] = 0;
  *(_WORD *)&v4[3] = 0;
  *(_DWORD *)&v4[5] = 0;
  do
  {
    if ( *(_QWORD *)&v4[1] >= 0x17u )
      core::panicking::panic_bounds_check::h8307ccead484a122(*(_QWORD *)&v4[1], 23, &off_54578);
    s[*(_QWORD *)&v4[1]] = v3[*(_QWORD *)&v4[1]] ^ 0x3F;
    v1 = *(_QWORD *)&v4[1] + 1LL;
    if ( *(_QWORD *)&v4[1] == -1 )
      core::panicking::panic_const::panic_const_add_overflow::hf2f4fb688348b3b0(&off_545A8);
    ++*(_QWORD *)&v4[1];
  }
  while ( v1 != 23 );
  if ( (unsigned int)std::process::id::hcbcee05e6d949703() == 29485234 )
  {
    v10 = s;
    v11 = &core::array::_$LT$impl$u20$core..fmt..Debug$u20$for$u20$$u5b$T$u3b$$u20$N$u5d$$GT$::fmt::hf6f6e41e4948d91c;
    v12 = s;
    v8 = s;
    v9 = &core::array::_$LT$impl$u20$core..fmt..Debug$u20$for$u20$$u5b$T$u3b$$u20$N$u5d$$GT$::fmt::hf6f6e41e4948d91c;
    v6[2] = s;
    v6[3] = &core::array::_$LT$impl$u20$core..fmt..Debug$u20$for$u20$$u5b$T$u3b$$u20$N$u5d$$GT$::fmt::hf6f6e41e4948d91c;
    v6[0] = s;
    v6[1] = &core::array::_$LT$impl$u20$core..fmt..Debug$u20$for$u20$$u5b$T$u3b$$u20$N$u5d$$GT$::fmt::hf6f6e41e4948d91c;
    core::fmt::Arguments::new_v1::hfac9ebf3d99d1264(v5, &unk_545C0, v6);
    return std::io::stdio::_print::he7d505d4f02a1803(v5);
  }
  else
  {
    core::fmt::Arguments::new_const::hf72ed85907e377bb(v7, &off_545E0);
    return std::io::stdio::_print::he7d505d4f02a1803(v7);
  }
}
```
Inside it, there were a bunch of byte arrays like s, v3, etc.

noticed something interstiing
```
qmemcpy(v3, "{^HX|kyDym", 10);

v3[10] = 12;
v3[11] = 12;
v3[12] = 96;
v3[13] = 124;
v3[14] = 11;
v3[15] = 109;
v3[16] = 96;
v3[17] = 104;
v3[18] = 11;
v3[19] = 10;
v3[20] = 119;
v3[21] = 30;
```

- this was the first 22 bytes of some encoded data
- then i saw the code accidentally reads one more byte: ``s[i] = v3[i] ^ 0x3F;``
- but at index 22, v3[22] actually comes from v4[0], which was set to 'B'.
- so the full array has 23 bytes total.
- shen there was a loop: ``s[i] = v3[i] ^ 0x3F;``
- this means the real flag characters are stored in v3, but XOR-encrypted with 0x3F.
- so to decode the flag, I rebuilt v3 in Python and XORed each byte with 0x3F.

Here is the Python script I wrote:
```
base = list(b"{^HX|kyDym")
extras = [12,12,96,124,11,109,96,104,11,10,119,30]
v3 = base + extras
v3.append(ord('B'))

flag = "".join(chr(b ^ 0x3F) for b in v3)
print(flag)
```
When I ran it, I got:
```
DawgCTF{FR33_C4R_W45H!}
```
There was also some process ID check in the code, but it didn’t matter because we were extracting the bytes directly.

## dusty_intermediate
