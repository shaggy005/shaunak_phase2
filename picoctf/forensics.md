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
