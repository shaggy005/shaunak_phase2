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
when i ran the binary normally it just said it was thinking of a random number and wanted me to guess it. if i guessed right, it would give a flag. obviously i’m not going to brute force a rand() number, so i opened the binary in ida and looked at the pseudocode. it was pretty straightforward: they called time(), seeded the prng, called rand() once, stored the result in a variable, and compared it with my input. if they were equal they read /home/h3/flag.txt.

there was no trick, the solution was just read the return value of rand() using a debugger.

### finding the value in gdb
first i checked architecture:
```
file time:
ELF 64-bit x86-64
```
so i used normal gdb:
```
gdb ./time
```
then i set a breakpoint on rand:
```
break rand
```
next i started the program:
```
run
```
it stopped inside libc at rand(). i didn’t care about stepping inside the library code, so i just finished the current function:
```
finish
```
this returned execution back to my main function, right after:
```
v6 = rand();
```
gdb actually printed the return value automatically:
```
Value returned is $1 = 494044218
```
to be sure, i printed eax explicitly:
```
p/d $eax
494044218
```
this is the exact number that rand() produced
### feeding the number back
then i just resumed the program:
```
continue
```
the binary asked for my guess, so i typed: ``494044218``
### output:
```
Your guess was 494044218.
Looking for 494044218.
You won. Guess was right! Here's your flag:
Flag file not found!  Contact an H3 admin for assistance.
```
```
ganesh-pavankumar-marada@ganesh-pavankumar-marada-Latitude-E5470:~/Downloads$ gdb ./time
GNU gdb (Ubuntu 15.0.50.20240403-0ubuntu1) 15.0.50.20240403-git
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./time...

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
(No debugging symbols found in ./time)
(gdb) break rand
Breakpoint 1 at 0x400790
(gdb) run
Starting program: /home/ganesh-pavankumar-marada/Downloads/time
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Download failed: Invalid argument.  Continuing without source file ./stdlib/./stdlib/rand.c.
Download failed: Invalid argument.  Continuing without source file ./stdlib/./stdlib/rand.c.

Breakpoint 1, 0x00007ffff7c4a0a8 in rand () at ./stdlib/rand.c:27
warning: 27 ./stdlib/rand.c: No such file or directory
(gdb) finish
Run till exit from #0  0x00007ffff7c4a0a8 in rand () at ./stdlib/rand.c:27
0x000000000040095f in main ()
Value returned is $1 = 494044218
(gdb) p/d $eax
$2 = 494044218
(gdb) continue
Continuing.
Welcome to the number guessing game!
I'm thinking of a number. Can you guess it?
Guess right and you get a flag!
Enter your number: 494044218
Your guess was 494044218.
Looking for 494044218.
You won. Guess was right! Here's your flag:
Flag file not found!  Contact an H3 admin for assistance.
[Inferior 1 (process 24225) exited normally]
(gdb) Quit
(gdb) 
```
so the logic was correct, the check succeeded. the actual flag file wasn’t included in my copy, so it printed the error message, but if the file existed it would’ve dumped the contents.
### Notes
had to use my roomie's laptop here cause my docker and kali vm were both tweaking out
regular mac user crashout

## Alternate Solve:
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

# 4. VerdisQuo
here we have an android apk and the flag is hidden somewhere in it
- my first thought was to disassemble it in ida, but that didnt bring u anything meaningful
- then i opened it in android studio
- i emulated the app and opened it but it only said "too slow"
- that should have been my first clue that the app is erasing something as soon as i open the app
- then i open the apk in jadx
- out of everything what looked most interesting was the folder called ``byuctf.downwithefrench``
- so this must be where the stuff happens
- i open the main file
```
package byuctf.downwiththefrench;

import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;

/* loaded from: classes3.dex */
public class MainActivity extends AppCompatActivity {
    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Utilities util = new Utilities(this);
        util.cleanUp();
        TextView homeText = (TextView) findViewById(R.id.homeText);
        homeText.setText("Too slow!!");
    }
}
```
so the app opens up whatever was inside utilities then cleans it up then prints "too slow"
i open up utilities to see this
```
package byuctf.downwiththefrench;

import android.app.Activity;
import android.widget.TextView;

/* loaded from: classes3.dex */
public class Utilities {
    private Activity activity;

    public Utilities(Activity activity) {
        this.activity = activity;
    }

    public void cleanUp() {
        TextView flag = (TextView) this.activity.findViewById(R.id.flagPart1);
        flag.setText("");
        TextView flag2 = (TextView) this.activity.findViewById(R.id.flagPart2);
        flag2.setText("");
        TextView flag3 = (TextView) this.activity.findViewById(R.id.flagPart3);
        flag3.setText("");
        TextView flag4 = (TextView) this.activity.findViewById(R.id.flagPart4);
        flag4.setText("");
        TextView flag5 = (TextView) this.activity.findViewById(R.id.flagPart5);
        flag5.setText("");
        TextView flag6 = (TextView) this.activity.findViewById(R.id.flagPart6);
        flag6.setText("");
        TextView flag7 = (TextView) this.activity.findViewById(R.id.flagPart7);
        flag7.setText("");
        TextView flag8 = (TextView) this.activity.findViewById(R.id.flagPart8);
        flag8.setText("");
        TextView flag9 = (TextView) this.activity.findViewById(R.id.flagPart9);
        flag9.setText("");
        TextView flag10 = (TextView) this.activity.findViewById(R.id.flagPart10);
        flag10.setText("");
        TextView flag11 = (TextView) this.activity.findViewById(R.id.flagPart11);
        flag11.setText("");
        TextView flag12 = (TextView) this.activity.findViewById(R.id.flagPart12);
        flag12.setText("");
        TextView flag13 = (TextView) this.activity.findViewById(R.id.flagPart13);
        flag13.setText("");
        TextView flag14 = (TextView) this.activity.findViewById(R.id.flagPart14);
        flag14.setText("");
        TextView flag15 = (TextView) this.activity.findViewById(R.id.flagPart15);
        flag15.setText("");
        TextView flag16 = (TextView) this.activity.findViewById(R.id.flagPart16);
        flag16.setText("");
        TextView flag17 = (TextView) this.activity.findViewById(R.id.flagPart17);
        flag17.setText("");
        TextView flag18 = (TextView) this.activity.findViewById(R.id.flagPart18);
        flag18.setText("");
        TextView flag19 = (TextView) this.activity.findViewById(R.id.flagPart19);
        flag19.setText("");
        TextView flag20 = (TextView) this.activity.findViewById(R.id.flagPart20);
        flag20.setText("");
        TextView flag21 = (TextView) this.activity.findViewById(R.id.flagPart21);
        flag21.setText("");
        TextView flag22 = (TextView) this.activity.findViewById(R.id.flagPart22);
        flag22.setText("");
        TextView flag23 = (TextView) this.activity.findViewById(R.id.flagPart23);
        flag23.setText("");
        TextView flag24 = (TextView) this.activity.findViewById(R.id.flagPart24);
        flag24.setText("");
        TextView flag25 = (TextView) this.activity.findViewById(R.id.flagPart25);
        flag25.setText("");
        TextView flag26 = (TextView) this.activity.findViewById(R.id.flagPart26);
        flag26.setText("");
        TextView flag27 = (TextView) this.activity.findViewById(R.id.flagPart27);
        flag27.setText("");
        TextView flag28 = (TextView) this.activity.findViewById(R.id.flagPart28);
        flag28.setText("");
    }
}
```
- to view the text we need to see the layout
- so i navigate to the layout xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/homeText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="b"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.066"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.022"/>
    <TextView
        android:id="@+id/flagPart1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="420dp"
        android:text="}"
        android:layout_marginEnd="216dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="616dp"
        android:text="t"
        android:layout_marginEnd="340dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="556dp"
        android:text="a"
        android:layout_marginEnd="332dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="676dp"
        android:text="y"
        android:layout_marginEnd="368dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="500dp"
        android:text="c"
        android:layout_marginEnd="252dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart6"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="636dp"
        android:text="c"
        android:layout_marginEnd="348dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart7"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="436dp"
        android:text="d"
        android:layout_marginEnd="364dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart8"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="496dp"
        android:text="r"
        android:layout_marginEnd="348dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart9"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="536dp"
        android:text="n"
        android:layout_marginEnd="336dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart10"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="456dp"
        android:text="i"
        android:layout_marginEnd="360dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart11"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="536dp"
        android:text="0"
        android:layout_marginEnd="276dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart12"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="516dp"
        android:text="d"
        android:layout_marginEnd="340dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart13"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="460dp"
        android:text="k"
        android:layout_marginEnd="232dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart14"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="656dp"
        android:text="u"
        android:layout_marginEnd="356dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart15"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="452dp"
        android:text="p"
        android:layout_marginEnd="320dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart16"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="476dp"
        android:text="o"
        android:layout_marginEnd="352dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart17"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="500dp"
        android:text="c"
        android:layout_marginEnd="300dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart18"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="596dp"
        android:text="f"
        android:layout_marginEnd="332dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart19"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="484dp"
        android:text="e"
        android:layout_marginEnd="308dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart20"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="436dp"
        android:text="_"
        android:layout_marginEnd="328dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart21"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="516dp"
        android:text="e"
        android:layout_marginEnd="292dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart22"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="536dp"
        android:text="_"
        android:layout_marginEnd="284dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart23"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="536dp"
        android:text="f"
        android:layout_marginEnd="268dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart24"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="468dp"
        android:text="i"
        android:layout_marginEnd="316dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart25"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="516dp"
        android:text="_"
        android:layout_marginEnd="260dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart26"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="480dp"
        android:text="4"
        android:layout_marginEnd="240dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart27"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="440dp"
        android:text="e"
        android:layout_marginEnd="224dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/flagPart28"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="576dp"
        android:text="{"
        android:layout_marginEnd="324dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```
so these are the texts that the app is clearing, so see it in order we copy paste this xml in a blank app
<img width="1799" height="1071" alt="Screenshot 2025-12-04 at 7 17 56 AM" src="https://github.com/user-attachments/assets/35e65511-23b3-460c-a312-6236e7214733" />
### and we get our flag
``byuctf{android_piece_0f_c4ke}``

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

The binary was a Rust program, so the decompiled code looked huge and confusing.
I couldn’t understand the functions or the naming, so instead of trying to read everything, I focused only on the parts that looked important.

### Finding the Important Part

When I scrolled through the decompiler, I saw two things that stood out:
Something inside the program was taking bytes of my input and changing them
The program compared the result to exactly 21 bytes that were stored inside the binary
So I realized:
If I can find those 21 bytes and whatever table the program uses to change my input, I can reverse it and get the correct input then that should be the flag.

### Finding the 21 Secret Bytes

In the decompiled code, I found constants like:
```
0x7097E6D32231D9EA
0x6876FC611BA8A216
0x27B8AB7B
0x96
```
These were grouped right before the comparison, so I assumed they were the 21 output bytes the program wanted.
I used Python to unpack them into real bytes.
This gave me:``ead93122d3e6977016a2a81b61fc76687babb82796``
So that’s the 21 output bytes the program expects.

### Finding the Table the Program Uses

In the code I kept seeing a reference to:
unk_61298
Looked like some kind of array.
I checked .rodata in the binary and found that starting at file offset 0x61298 there are 256 bytes, which looked like a full lookup table.
So I dumped those 256 bytes using Python.

### Realizing How the Program Transforms My Input

Even though the code was confusing, I noticed the important lines:
There is a variable v13 starting at 117
Every time a byte of input is processed, the code does:
```
v13 = v13 + input_byte   (mod 256)
output = table[v13]
```
So basically the program:
Takes your input byte
Adds it to a running value
Uses the result as an index into the table
That then becomes the output byte
Since I know the output and I know the table, I can reverse the math.

### Reversing It (Step by Step)

For every expected output byte:
Look inside the table for where that byte appears
That table index is the state value
Since state = (previous_state + input_byte) mod 256
I can solve:
```
input_byte = (state - previous_state) mod 256
```
then do this 21 time over till i recover all the input bytes.

### Writing a python script (wow how predictable)

I put everything into a Python script that:
- Opens the binary
- Reads the 256-byte table
- Builds the 21 expected bytes
- Loops through each expected byte and reverses the math
- Prints the flag
```
import sys, struct

b = open(sys.argv[1], "rb")
# grab sub table (0x61298)
b.seek(0x61298)
tbl = b.read(256)
print("sub table (first 64 bytes):")
print(tbl[:64].hex())
print()
# expected bytes (from decompiler)
exp = b''
exp += struct.pack("<Q", 0x7097E6D32231D9EA)
exp += struct.pack("<Q", 0x6876FC611BA8A216)
exp += struct.pack("<I", 0x27B8AB7B)
exp += struct.pack("<B", 0x96)
print("expected 21 bytes:")
print(exp.hex())
print()
# invert transform
rev = {}
for i in range(256):
    x = tbl[i]
    if x not in rev:
        rev[x] = []
    rev[x].append(i)

state = 117
out = []

for e in exp:

    # find table index producing e

    for s in rev[e]:
        inp = (s - state) & 0xff
        if ((state + inp) & 0xff) == s:
            out.append(inp)
            state = s
            break

print("recovered 21 bytes (hex):")
print(bytes(out).hex())
print()

try:
    print("FLAG:", bytes(out).decode())
except:
    print("flag bytes:", bytes(out))

```
```
shaunak@Shaunaks-MacBook-Pro dusty % python3 intersolve.py dust_intermediate
sub table (first 64 bytes):
9fd2d6a89976b875e20e5067c93aa0b515ee59be7da3fb51df7cd90de72dad28eddc3d141379af27d1d5a1f937c0ef253877ff1b40608f456f086dd3353fb42f

expected 21 bytes:
ead93122d3e6977016a2a81b61fc76687babb82796

recovered 21 bytes (hex):
446177674354467b53303030305f434c34334e217d

FLAG: DawgCTF{S0000_CL43N!}
shaunak@Shaunaks-MacBook-Pro dusty % 
```
### Final Flag
``DawgCTF{S0000_CL43N!}``

## dust_pro
similar to the last two but a few caveats here and there
the program prints some text and asks for an input. it reads a line, tries to parse it as an integer, and then does something with it.
i saw this loop in the code:
```
while ( 1 ) {
    // ... bounds checks ...
    v50[v57] ^= *((_BYTE *)&v54 + v22);
}
```
- variable v50 was a big array of numbers (bytes), and v54 was the number i typed in. the ^= meant it was XORing the array with my input number.
- after the loop, it does a SHA256 hash check.
- at first, i thought, "can i just patch the jump?" like, find the jz instruction where it checks the hash and change it so it always says i win?
- but then i realized if i did that, the flag would still be encrypted because the program uses my input to actually decrypt the text. if i put in the wrong number, the XOR math produces garbage.
- so i had to find the correct input number that decrypts the flag.
- i copied the bytes from the code (IDA showed them as negative numbers like -49, so I had to convert them to normal bytes).
- since it's XOR, I knew that: ``Encrypted_Byte XOR Key = Real_Flag_Letter``
- i wrote a python script to brute force it. I figured the flag probably started with DawgCTF since the last two challenges also had the same format (i cheated a bit but come on, this much should be allowed since if sm1 was in the real ctf this would be a valid solution)
```
def solve():
    # These are the bytes I got from IDA
    encrypted_bytes = [
        207, 9, 30, 179, 200, 60, 47, 175, 191, 36, 37, 139, 
        217, 61, 92, 227, 212, 38, 89, 139, 200, 92, 59, 245, 246
    ]

    # Trying to find the key by guessing the header is "Dawg"
    known_header = "Dawg"
    key = []
    for i in range(4):
        key.append(encrypted_bytes[i] ^ ord(known_header[i]))

    print("Key found:", key)

    # Now print the whole thing
    out = ""
    for i, byte in enumerate(encrypted_bytes):
        out += chr(byte ^ key[i % 4])
    print("Flag:", out)

solve()
```
```
shaunak@Shaunaks-MacBook-Pro dusty % python3 prosolve.py 
Key found: [139, 104, 105, 212]
Flag: DawgCTF{4LL_RU57_N0_C4R!}
shaunak@Shaunaks-MacBook-Pro dusty % 
```
### Flag: ``DawgCTF{4LL_RU57_N0_C4R!}``
