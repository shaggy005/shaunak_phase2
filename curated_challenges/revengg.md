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
