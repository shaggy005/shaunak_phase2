# 1. Buffer overflow 0
>Let's start off simple, can you overflow the correct buffer? The program is available here. You can view source here.
Connect using:
`nc saturn.picoctf.net 52136`

## Solution:
- I open the c code and read it thorougly
- I see that the print function uses a `sigsedv_handler()` function
- It meands that the flag is printed when a segmentation fault is triggered
- Segmentation fault happens when a program tries to access memory that is restricted or that doesnt exist at all.
- Also we see that the input is taken using a `gets()` function.
- `gets()` is known to be notorious with buffer overflows, that means it will write input to a stack without knowing its allocated length
- So if just we write an input long enough we can artificially trigger a segementation fault
- So i turn on the instance and just type a very long input
- That gives away the flag

```
shaunak@Shaunaks-MacBook-Pro bare-metal-alchemist % nc saturn.picoctf.net 52136
Input: sdgfjhdajhvvhdkhdsvjdkghdjfkhdgjdkhgjdhjdsjgkdjghkdhgjdjkfdsjgsbhckh
picoCTF{ov3rfl0ws_ar3nt_that_bad_9f2364bc}
shaunak@Shaunaks-MacBook-Pro bare-metal-alchemist %
```

## Flag:
```
picoCTF{ov3rfl0ws_ar3nt_that_bad_9f2364bc}
```

## Concepts learnt:
- Segementation Fault
- Buffer overflow
- I kinda knew what these are but still got a refresher course

## Notes:
- none

## Resources:
- https://stackoverflow.com/questions/1564372/what-causes-a-sigsegv
- http://stackoverflow.com/questions/18986351/what-is-the-simplest-standard-conform-way-to-produce-a-segfault-in-c

***

# 2. 
> Put in the challenge's description here

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
-

## Resources:
-

***

# 3. 
> 

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
-

## Resources:
-

***
