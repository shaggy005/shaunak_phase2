# JoyDivision
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
```
shaunak@Shaunaks-MacBook-Pro JoyDivision % base64 -i flag.txt -o flag.txt.b64

shaunak@Shaunaks-MacBook-Pro JoyDivision % base64 -D -i flag.txt.b64 -o flag.bin 

shaunak@Shaunaks-MacBook-Pro JoyDivision % python3 sol.py
sunshine{C3A5ER_CR055ED_TH3_RUB1C0N}
```
#  Worthy Knight

I opened the code. It asks for a 10 letter password.
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

## Final Incantation

Putting all pairs together:

**Nj kS fT Ya Ii**


So the final 10-letter string is:
**NjkSfTYaIi**

The program prints the flag:

**KCTF{NjkSfTYaIi}**
