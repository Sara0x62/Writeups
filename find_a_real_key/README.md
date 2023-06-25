# Find a real key - RE challenge
by; f0rizen
link; https://crackmes.one/crackme/629e1e5833c5d4251e72375f

### Starting off
Running strings gets us a little bit of interesting info;

```
Usage: ./crackme FLAG
Wrong flag
sup3r_s3cr3t_k3y_1337
You found a flag! %s
```

But of course this "super secret key" is not our solution..
![try1](https://github.com/Sara0x62/Writeups/assets/83826811/bc44480c-8570-4d31-82ec-c34ea8a9c908)

## Time to get to the disassembly
Quite a lot of code in our main, i will post the full code down below and just focus on the things we care about

the first block - after like main variable initialization.
I have renamed some of the variables for easier reading (instead of just local_xx everywhere)
```c
  if (argc == 1) {    // Check the argument count
    puts("Usage: ./crackme FLAG");
    return_value = 1;
  }
```

This next part we loop over our "super secret key" - 
we take each character, do `- 0x22` on that value (since in C a character is also an integer value)
and store that in an array
```c
  else {
    argv_len = strlen(*(char **)(argv + 8));
    if (argv_len == 0x15) {
      for (loop1_cntr = 0; loop1_cntr < 0x15; loop1_cntr = loop1_cntr + 1) {
        abStack_28[loop1_cntr] = "sup3r_s3cr3t_k3y_1337"[loop1_cntr] - 0x22;    
      }
```

After that we have a large block of values being loaded into another array
```c
      local_88[0] = 0x37;
      local_88[1] = 0x3f;
      local_88[2] = 0x2f;
      local_88[3] = 0x76;
      local_88[4] = 0x2b;
      local_88[5] = 0x62;
      local_88[6] = 0x28;
      local_88[7] = 0x21;
      local_88[8] = 0x34;
      local_88[9] = 0xf;
      local_88[10] = 0x77;
      local_88[11] = 0x62;
      local_88[12] = 0x48;
      local_88[13] = 0x27;
      local_88[14] = 0x75;
      local_88[15] = 8;
      local_88[16] = 0x56;
      local_88[17] = 0x6a;
      local_88[18] = 0x68;
      local_88[19] = 0x4e;
      local_88[20] = 0x68;
```

Finally we have this block; the dissassmbly made our if statement look quite unreadable..
```c
      for (loop2_cntr = 0; loop2_cntr < 0x15; loop2_cntr = loop2_cntr + 1) {
        if ((int)(char)(abStack_28[loop2_cntr] ^ *(byte *)((long)loop2_cntr + *(long *)(argv + 8)))
            != local_88[loop2_cntr]) {
          puts("Wrong flag");
          return_value = 1;
          goto LAB_00101355;
        }
      }
      printf("You found a flag! %s\n",*(undefined8 *)(argv + 8));
```
Cleaned up the if statement basically comes down to something like this
```c
if ( ( abStack_28[loop2_cntr] ^ argv[loop2_cntr] ) != local_88[loop2_cntr]) { ... }
```
so, it loads our key, with each character having 0x22 subtracted
It then loads all those other values and finally does a XOR check;
The loaded key, XORed against our input - and checks if that equals each of those values
Should be easy enough to solve, especially since the inverse of XOR is just XOR again

```
a ^ b = c
c ^ b = a
etc.
```

Our solution - a simple script in python;

```python
key = "sup3r_s3cr3t_k3y_1337"

# Copied from the disassembly
local_88 = [
    0x37, 0x3f, 0x2f, 0x76, 0x2b, 0x62, 0x28,
    0x21, 0x34, 0xf, 0x77, 0x62, 0x48, 0x27,
    0x75, 8, 0x56, 0x6a, 0x68, 0x4e, 0x68]

out = ""

for ch, k in zip(key, local_88):
    out += chr( (ord(ch) - 0x22) ^ k )

print(out)
```

![success](https://github.com/Sara0x62/Writeups/assets/83826811/b12a3101-8a41-4161-a334-0a5168bb5a3c)



Full dissassembly code by Ghidra
```c
undefined8 main(int argc,long argv)

{
  undefined8 return_value;
  size_t argv_len;
  long in_FS_OFFSET;
  int loop1_cntr;
  int loop2_cntr;
  int local_88 [24];
  byte abStack_28 [24];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  if (argc == 1) {
    puts("Usage: ./crackme FLAG");
    return_value = 1;
  }
  else {
    argv_len = strlen(*(char **)(argv + 8));
    if (argv_len == 0x15) {
      for (loop1_cntr = 0; loop1_cntr < 0x15; loop1_cntr = loop1_cntr + 1) {
        abStack_28[loop1_cntr] = "sup3r_s3cr3t_k3y_1337"[loop1_cntr] - 0x22;
      }
      local_88[0] = 0x37;
      local_88[1] = 0x3f;
      local_88[2] = 0x2f;
      local_88[3] = 0x76;
      local_88[4] = 0x2b;
      local_88[5] = 0x62;
      local_88[6] = 0x28;
      local_88[7] = 0x21;
      local_88[8] = 0x34;
      local_88[9] = 0xf;
      local_88[10] = 0x77;
      local_88[11] = 0x62;
      local_88[12] = 0x48;
      local_88[13] = 0x27;
      local_88[14] = 0x75;
      local_88[15] = 8;
      local_88[16] = 0x56;
      local_88[17] = 0x6a;
      local_88[18] = 0x68;
      local_88[19] = 0x4e;
      local_88[20] = 0x68;
      for (loop2_cntr = 0; loop2_cntr < 0x15; loop2_cntr = loop2_cntr + 1) {
        if ((int)(char)(abStack_28[loop2_cntr] ^ *(byte *)((long)loop2_cntr + *(long *)(argv + 8)))
            != local_88[loop2_cntr]) {
          puts("Wrong flag");
          return_value = 1;
          goto LAB_00101355;
        }
      }
      printf("You found a flag! %s\n",*(undefined8 *)(argv + 8));
      return_value = 0;
    }
    else {
      puts("Wrong flag");
      return_value = 1;
    }
  }
LAB_00101355:
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return return_value;
}
```
