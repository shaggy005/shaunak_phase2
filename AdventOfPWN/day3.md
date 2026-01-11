> in this challenge, a file called /stocking appears briefly containing the flag, and then gets deleted.
> trying to cat /stocking normally doesn’t work because the file often disappears before i can access it.

- the key thing i relied on is how unix file descriptors work:
- when a process opens a file, the kernel gives it a file handle (file descriptor).
- even if the file is later deleted using unlink(), the process that already opened it can still read from it.
- the data only truly disappears once all open file handles are closed.

so the goal is not to win a timing race with cat, but instead to open the file early and keep the handle alive.

### key idea

- i used two important tricks:
- tail -F to hold the file open
- tail reads the end of a file
- -F means follow by name, so it keeps watching the file even if it gets recreated
- this effectively opens /stocking and keeps the file descriptor active while i wait for content to appear
- renice to satisfy the challenge condition
- the challenge script checks for processes that are running with a positive nice value.
- "nice" is a scheduling priority value in linux
- higher nice value = more polite process (gets less CPU priority)
- renice lets me change the priority of an already-running process
- so by making my shell process "nice", the challenge condition becomes true and the flag gets written.
```
ubuntu@2025~day-03:~$ tail -F /stocking &
[1] 1306
ubuntu@2025~day-03:~$ renice -n 1 -p $$
354 (process ID) old priority 0, new priority 1
ubuntu@2025~day-03:~$ sleep 10
pwn.college{s1yT64zZXtDqZnKzxp2rGVe36QT.0FN4gTMywSMwEzNzEzW}
```
