# 1. Trivial Flag Transfer Protocol

> Figure out how they moved the flag.

## Solution:

- ran the pcapng file in wireshark and wasted a bunch of time just scrolling through it
- then realised that you can export objects, so i exported all the objects with tftp
- got 5 files: 2 txt, 3 pics and one program
- there was garbled text inside instructions.txt

```
GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA
```
- messed around with a few different types of ciphers and found out that it was rot13 and got
```
TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN
```
- it hinted to check whats in plan.txt
```
VHFRQGURCEBTENZNAQUVQVGJVGU-QHRQVYVTRAPR.PURPXBHGGURCUBGBF
```
- applied rot13 on this again to find out
```
IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
```
- had a hunch that there might be some steganography involved so i ran steghide on the images
``` bash
┌──(shaunakkli㉿shaunakkali)-[~/Downloads/picogym/forensics]
└─$ steghide extract -sf picture1.bmp -p DUEDILIGENCE
steghide: could not extract any data with that passphrase!
                                                                                
┌──(shaunakkli㉿shaunakkali)-[~/Downloads/picogym/forensics]
└─$ steghide extract -sf picture2.bmp -p DUEDILIGENCE
steghide: could not extract any data with that passphrase!
                                                                                
┌──(shaunakkli㉿shaunakkali)-[~/Downloads/picogym/forensics]
└─$ steghide extract -sf picture3.bmp -p DUEDILIGENCE
wrote extracted data to "flag.txt".
                                                                                
┌──(shaunakkli㉿shaunakkali)-[~/Downloads/picogym/forensics]
└─$ cat flag.txt        
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
                                                                                
┌──(shaunakkli㉿shaunakkali)-[~/Downloads/picogym/forensics]
└─$ 
```
- and boom there was the flag
## Flag:

```
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
```

## Concepts learnt:

- whole lotta time spent on learning what wireshark is and how it works

## Notes:

- wasted a lot of time reading the pcapng file

## Resources:

- https://www.varonis.com/blog/how-to-use-wireshark
- https://www.youtube.com/watch?v=qTaOZrDnMzQ
- https://www.youtube.com/watch?v=OU-A2EmVrKQ


***
# 2. tunn3l v1s10n

> We found this file. Recover the flag.

## Solution:

- first i checked the type using file

```
┌──(shaunakkli㉿shaunakkali)-[~/Downloads]
└─$ file tunn3l_v1s10n
tunn3l_v1s10n: data

```
- i saw that the file type was just data, so it must be a corrupted file. i tried opening it as well but it didnt open
```
┌──(shaunakkli㉿shaunakkali)-[~/Downloads]
└─$ exiftool tunn3l_v1s10n 
ExifTool Version Number         : 13.25
File Name                       : tunn3l_v1s10n
Directory                       : .
File Size                       : 2.9 MB
File Modification Date/Time     : 2025:10:26 15:42:41+05:30
File Access Date/Time           : 2025:10:26 15:42:44+05:30
File Inode Change Date/Time     : 2025:10:26 15:42:44+05:30
File Permissions                : -rw-rw-r--
File Type                       : BMP
File Type Extension             : bmp
MIME Type                       : image/bmp
BMP Version                     : Unknown (53434)
Image Width                     : 1134
Image Height                    : 306
Planes                          : 1
Bit Depth                       : 24
Compression                     : None
Image Length                    : 2893400
Pixels Per Meter X              : 5669
Pixels Per Meter Y              : 5669
Num Colors                      : Use BitDepth
Num Important Colors            : All
Red Mask                        : 0x27171a23
Green Mask                      : 0x20291b1e
Blue Mask                       : 0x1e212a1d
Alpha Mask                      : 0x311a1d26
Color Space                     : Unknown (,5%()
Rendering Intent                : Unknown (826103054)
Image Size                      : 1134x306
Megapixels                      : 0.347

```
- then i decided to check the header for this file
```
┌──(shaunakkli㉿shaunakkali)-[~/Downloads]
└─$ head tunn3l_v1s10n 
BM�&,����n2X&,%%#␦'␦1(%5,)3*'8/,/&#3*&-$ ;2.2)%0'#3*&8,(6+'9-+/&##)U=1�vf�fR�mV�pX�oT�oT�~c��m��iȗq��q��t��s��r��o��n��k��j��d�tU�wZ�oVvR:qR=lO@mRDnSIw^TS93pXRvaYs_T~k^�tc~jYvbPv^LzbP�m]�iY�sc�4$M6'J3$F, H."F."D."<& 02#␦6'<+">+$B,&^D>fLF63␦?0-K5*?)B.#K7,E1&?+ C/$C/$@*H2'K2(G.$@'E,"L4(L4(K3'J2&L2$N4&P5'R7)S6(U8*K0"]B4cI9I/D+
( "␦ 
```
- then i decided to open the file in a hex editor https://hexed.it/
- according to the bmp file format the header format is like this
  ![68747470733a2f2f7777772e64796e616d736f66742e636f6d2f636f6465706f6f6c2f696d672f323031362f31322f424d502d66696c652d7374727563747572652e504e47](https://github.com/user-attachments/assets/56116c8d-6a5f-4e4f-960d-6babbce7991b)

- the size is 2893454 bytes so the hex should be 8E 26 2C 00 (0X2C268E)
- now we have the magic bytes 42 4d then the size 0x2c268e and then the offset 0xd0ba
- then theres the dib header, it indicates a size of 53434 (0xd0ba) instead of 40 bytes
- so we replace ba d0 by 28 00 (0x0028)
- also the image dimensions are wrong so we fix it by changing the height
- the height should be (0x2c268e-54(for the header))/(3*1134(1134 pixels width and 3 bytes per pixel))=850
- 850 is 0x352 that shoukd be 52 03 in the header
- in the end the header hex should look like this
  <img width="1830" height="550" alt="image" src="https://github.com/user-attachments/assets/2500c482-b8e4-4e8f-a375-afa087854aaa" />
- opening the image we get the flag
  [tunn3l_v1s10n (4).bmp](https://github.com/user-attachments/files/23150589/tunn3l_v1s10n.4.bmp)

## Flag:

```
picoCTF{qu1t3_a_v13w_2020}
```

## Concepts learnt:

- learnt a shitload about how to hex edit and what that header represents
- learnt some unnessasary amount of info about the bmp format 
- also messed around with file sizes and stuff

## Notes:

- oh dont get me started on this, i tried everything from stegano to strings to hexdump and i actually tried to manually read that hex dump
- spent unholy amount of time just figuring out the header

## Resources:

- http://www.fastgraph.com/help/bmp_header_format.html
- https://en.wikipedia.org/wiki/BMP_file_format
- http://www.ece.ualberta.ca/~elliott/ee552/studentAppNotes/2003_w/misc/bmp_file_format/bmp_file_format.htm
- https://www.youtube.com/watch?v=kpHFFFu9qeU
- http://www.ue.eti.pg.gda.pl/fpgalab/zadania.spartan3/zad_vga_struktura_pliku_bmp_en.html
- https://examplefiles.org/example-image-files/sample-bmp-files


***
# 3. m00nwalk

> Decode this message from the moon

## Solution:

- my first thought whenever i see an image is to just put it on ableton and see the spectogram, but that wast useful at all
- then i thought of applying some filters removing the noise, spent a whole lotta time doing that
- then i opened it in a hex editor and saw that it had JFIF in the strings
- now i figured that there might be an image hidden in the audio
- so i used a stegano decoder but alas that didnt work as well
- then i fell asleep
- next morning i gave in and decided to see the clue
- it said something about how images from moon were transmitted
- got to know about sstv
- got to know about qsstv and how to decode stuff, and spent a lot of time learning how it worked
- wasted a bunch of time trying to make qsstv work on my linux vm
- found an onine sstv decoder https://sstv-decoder.mathieurenaud.fr/ and just dragged and droppped the audio
- boom there was the picture
<img width="320" height="256" alt="download (4)" src="https://github.com/user-attachments/assets/2abe3f8b-7d7a-438b-89c7-32a441e53a71" />


## Flag:

```
picoCTF{beep_boop_im_in_space}
```

## Concepts learnt:

- SSTV and how to decode that shi
  
## Notes:

- life is much easier just using online tools, i spent wayyy too much time trying to get qsstv to work

## Resources:
- https://en.wikipedia.org/wiki/Apollo_11_missing_tapes
- https://github.com/ON4QZ/QSSTV
- https://github.com/MarioKlebsch/QSSTV-macos
- https://sstv-decoder.mathieurenaud.fr/
- 
***
