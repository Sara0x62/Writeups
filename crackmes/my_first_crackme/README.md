# My First Crackme
crackmes link; https://crackmes.one/crackme/644af68433c5d43938912c6b

### Starting off...
We run a simple `strings crackme`
To see if we have any interesting strings visible

And we do, (as seen in the picture below)
![strings](https://github.com/Sara0x62/Writeups/assets/83826811/90ada361-9ac8-4723-8840-3848d732fba4)

These ones caught my eye;
```
P@ssword
Enter password:
Go away!
Welcome!
```


### That looks a little too easy...
![try1](https://github.com/Sara0x62/Writeups/assets/83826811/2deac1dc-96dd-4469-a6c8-43d9c43239f9)

and it was

Okay, lets try to understand what it's doing then; opening it in Ghidra
The decompiled code.. that looks very complicated and i don't understand this at all xD

![decompiled](https://github.com/Sara0x62/Writeups/assets/83826811/4fca8404-f761-49c7-a1b3-3b55bef1452f)

### Mhm... lets see if we can find something i do know 
Aha, i can understand this part

![decompiled2](https://github.com/Sara0x62/Writeups/assets/83826811/42eac490-1a94-4e0a-b966-f11651cf3198)

some more readable code here; a strncpy call, puts, fgets for our user input and strcmp to compare the passwords

This line in particular caught my eye- i have renamed the variables in the screenshots but
`line 46: local_password[1] = 'a';`

The C code is again a little confused but the assembly we can see pretty clearly it seems to load the password here
and then edit a letter!
`byte ptr [RAX + local_77],0x61` -- in this case 0x61 is 'a' and local_77 seems to be 1
so now we have `P@ssword` -> `Password`

![local_password](https://github.com/Sara0x62/Writeups/assets/83826811/5a62141a-17f3-44b7-bd1b-f1c7d6f20bec)

so it does that;
Upon closer inspection the disassembler got a litte.. confused here too for the C code
so i'll use the Assembly part for the rest

![assembly-confused](https://github.com/Sara0x62/Writeups/assets/83826811/3370625b-35bd-48e6-aee7-48b30902a130)

but this all basically comes down to 
```c
fgets(local_48, local_58, stdin);
local_54 = strcmp(local_48, local_password);
```
and from the part earlier; we know `local_password`= `password`after the edit

![try2](https://github.com/Sara0x62/Writeups/assets/83826811/e4c4be5c-09d8-4239-b5e3-9e2dde152a1f)

Success!

