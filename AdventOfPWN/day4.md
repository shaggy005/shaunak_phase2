> The program loads a special kernel watcher (eBPF) that listens every time I use ln.
> It checks the filenames I use and only succeeds if I follow a hidden correct order.

- If I give the right sequence, the program prints the flag.

### What I figured out

- By dumping and reading the eBPF assembly, I noticed it compares filenames character-by-character.

- It expects:
  ```
  source file must always be:
  sleigh
  ```
- destination filenames must be in this exact order:
```
dasher
dancer
prancer
vixen
comet
cupid
donner
blitzen
```
## shell script
```
#!/bin/sh
touch sleigh

for name in dasher dancer prancer vixen comet cupid donner blitzen
do
    ln sleigh $name
done
```

```
$ chmod +x solve.sh && ./solve.sh
🎅 🎄 🎁 Ho Ho Ho, Merry Christmas!
pwn.college{ED2tUUelC7OsTVEYy7TzVlEzSGO.0lM5gTMywSMwEzNzEzW}
```
