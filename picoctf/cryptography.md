# 1. RSA Oracle

> Can you abuse the oracle?
An attacker was able to intercept communications between a bank and a fintech company. They managed to get the message (ciphertext) and the password that was used to encrypt the message.
Additional details will be available after launching your challenge instance.

## Solution:

- I downloaded the secret and the password
- The hints say that i have to use ssl to decrypt the final message (secret.enc)
- Hence i figured that the password.enc must be the key to decode secret.enc
- the netcat connection opens an oracle that can be used to encode or decode something
- on encrypting something it gives the formula it uses to encrypt
  ```
  what should we do for you?
  E --> encrypt D --> decrypt. 
  e
  enter text to encrypt (encoded length must be less than keysize): shaunak
  shaunak
  
  encoded cleartext as Hex m: 736861756e616b
  
  ciphertext (m ^ e mod n)  4348842447110155337013063398512775669504355482501591674158803281748614095541024535962389280517359075103021821449692156887685210156751407909246521453314695

  ```
- searched on the web for rsa algorithms, and found out that its the same
- searched for vulnerabilities of rsa encoding
- figured out that rsa uses this formula 

```
- first convert the the cleartext to hex then to the ciphertext just becomes m^e modn
- this can be abused by a simple mathematical property
- lets define the encryption of m by f(m) = m^e mod n
- then f(m1)*f(m2)=f(m1*m2)
- so let f(m1)=c1 , f(m2)=c2 and f(m1*m2)=c3
- also let decryption of c1,c2,c3 be g(c1),g(c2),g(c3) respectively
- then g(c3)=m1 * m2 and since we already know m2 , m1 = g(c3)/m2

```
- now i wrote a simple script using my limited knowledge in python to calculate this and put the entire thing in openssl as well cause im lazy
- then ran this script in a python venv

```python
from subprocess import run, PIPE

# read password
f = open("password.enc","r")
c = int(f.read())
f.close()
print(c)

m1 = "a"
print("using m1 =", m1)
m1_bytes = m1.encode("utf-8")
m1_int = int.from_bytes(m1_bytes,"big")

print("oracle encrypt")
c1 = int(input("c1:"))

# combine
c2 = c*c1
print(c2)

# decrypt
m2 = int(input("m2(hex):"),16)

# recover password
m_int = m2//m1_int
m_bytes = m_int.to_bytes((m_int.bit_length()+7)//8,"big")
m = m_bytes.decode("utf-8",errors="ignore")
print(m)

# decrypt secret
r = run(["openssl","enc","-aes-256-cbc","-d","-in","secret.enc","-pass","pass:"+m],stdout=PIPE,stderr=PIPE,text=True)
print(r.stdout)
```
- then i ran the script along with the oracle to input the values, i take "a" as m2 here

```bash
┌──(shaunakkli㉿shaunakkali)-[~]
└─$ python3 recover_flag.py

3567252736412634555920569398403787395170577668834666742330267390011828943495692402033350307843527370186546259265692029368644049938630024394169760506488003
using m1 = a
oracle encrypt
c1:1894792376935242028465556366618011019548511575881945413668351305441716829547731248120542989065588556431978903597240454296152579184569578379625520200356186
6759203291556042231884299567348859177930002052933581070542971197285711015106013555983327382520890013292510910732035253877762059526771378554060662044890667955724486048505501021744299550049249400949244607539943901804897303068723126577086285320298610508086645085015303403649613600120029026519364249924535836558
m2(hex):136665a6be83
3319c
picoCTF{su((3ss_(r@ck1ng_r3@_3319c817}
```
```bash
┌──(shaunakkli㉿shaunakkali)-[~]
└─$ nc titan.picoctf.net 55883
*****************************************
****************THE ORACLE***************
*****************************************
what should we do for you? 
E --> encrypt D --> decrypt. 
e
enter text to encrypt (encoded length must be less than keysize): a
a

encoded cleartext as Hex m: 61

ciphertext (m ^ e mod n) 1894792376935242028465556366618011019548511575881945413668351305441716829547731248120542989065588556431978903597240454296152579184569578379625520200356186

what should we do for you? 
E --> encrypt D --> decrypt. 
d
Enter text to decrypt: 6759203291556042231884299567348859177930002052933581070542971197285711015106013555983327382520890013292510910732035253877762059526771378554060662044890667955724486048505501021744299550049249400949244607539943901804897303068723126577086285320298610508086645085015303403649613600120029026519364249924535836558
decrypted ciphertext as hex (c ^ d mod n): 136665a6be83
decrypted ciphertext: fe¦¾

what should we do for you? 
E --> encrypt D --> decrypt. 
```
- in this way we get the flag
  
## Flag:

```
picoCTF{su((3ss_(r@ck1ng_r3@_3319c817}
```

## Concepts learnt:

- dont get me started on the amount of maths required in ts, i tried doing it by hand xd
- then wrote this script in python, had to brush up my python a bit
- overall this problem took the most time 

## Notes:
- Note for self: dont try and waste time looking at hexdumps of .enc files
- brush up on python

## Resources:
- https://www.reviversoft.com/en/file-extensions/enc
- https://www.calculator.net/big-number-calculator.html
- https://stackoverflow.com/questions/30056762/rsa-encryption-and-decryption-in-python
- https://discuss.python.org/t/how-to-use-rsa-public-key-to-decrypt-ciphertext-in-python/2919/4
- https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/

***

# 2. 
> Put in the challenge's description here

## Solution:

- Include as many steps as you can with your thought process
- You **must** include images such as screenshots wherever relevant.

```
put codes & terminal outputs here using triple backticks

you may also use ```python for python codes for example
```

## Flag:

```
picoCTF{}
```

## Concepts learnt:



## Notes:



## Resources:



***

# 3. 
> Put in the challenge's description here

## Solution:

- Include as many steps as you can with your thought process
- You **must** include images such as screenshots wherever relevant.

```
put codes & terminal outputs here using triple backticks

you may also use ```python for python codes for example
```

## Flag:

```
picoCTF{}
```

## Concepts learnt:



## Notes:



## Resources:



***



