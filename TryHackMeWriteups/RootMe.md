"A ctf for beginners, can you root me?"

Target IP: 10.10.1.26

-----------------
# Task 2
Lets take a look at the tasks:
```
Scan the machine, how many ports are open?

What version of Apache is running?

What service is running on port 22?

Find directories on the web server using the GoBuster tool.  

What is the hidden directory?
```

Okay, so let's start our basic recon
first off with a standard "aggressive" **NMAP** using: `nmap -A -T4 10.10.1.26`
![](../attachments/ea0951982dbb7e6174934e0fe4120ffc.png)
hmm this should already answer our first 3 questions:
We got 2 ports open, port 80 is running Apache/2.4.29 and port 22 is running an **SSH** service.

now we'll run a standard **gobuster** with a default wordlist in `dir` mode
`gobuster dir -u 10.10.1.26 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt`
![](../attachments/5f3cad0e6b72d2557c09d6f18187e915.png)
and we found something already, `/panel`

---
# Task 3
here they want to have us read a file on the target, so we'll need shell access.
we'll launch burpsuite and its attached browser to check out the URL we found earlier.

Attempting to upload a php shell didnt seem to work
![](../attachments/a3022e0b1b96263ae3dbd3c40ad31ec6.png)

Lets try some other file extensions first by adding the post request for this to the BurpSuite Intruder.
![](../attachments/3ef71301e2cf55fa8972e43e43a1f310.png)

Here we set the filename extension as a variable
![](../attachments/dbeaa82e8b63799c176d0205e462a67e.png)
And for payloads we'll try some common php alternative extensions
![](../attachments/8697f85ebd435aaafe3eecdb180ec847.png)

Results seem to say it was a success! and we can see the URL where it is available now!:
(I am certain there is a easier way to check success but still gotta learn a lot myself :p )
![](../attachments/5c97401deb1efe20e9a03a97d9fb4c69.png)

Before we open that we should set up our netcat to listen for connections
`nc -nlvp 1235`
![](../attachments/a4e721c6251aca6b5cb8005081fae710.png)

and after opening the file through the browser we got our shell!
(I had to use the `.phtml` to get a successful connection, the other ones despite being on the server would not execute the shell properly)
![](../attachments/b578a693f1c7743a2c3d696c0ef18371.png)

With our connection lets start looking for this file lazily
`file -name user.txt -print 2>/dev/null` 
( sidenote, the `2>/dev/null` is simply to hide all the "permission denied" files etc.)
and we found it in `/var/www/user.txt` !
![](../attachments/abeffad0369f801be1ae3fc8372563cf.png)
I wont show the flag here, but you can simply read it with `cat`

Next up we will try to escalate our shell by finding files with SUID perms:
`find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null`
![](../attachments/f278d8d2c13f6c6c1b07a01bfcba0cf3.png)
Python for sure stands out here..
Lets see what GTFOBins says about python with SUID 
( https://gtfobins.github.io/gtfobins/python/#suid )

We learn that we can run a shell and keep privileges by simply running 
`python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`
![](../attachments/68b8e0dad09183455dd0730167575063.png)
after this simply navigate to the root folder and get our final flag!