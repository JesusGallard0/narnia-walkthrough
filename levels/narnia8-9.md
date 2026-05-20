# Narnia OverTheWire Level8-9


## Problem Explanation and Relevant Backgroung Knowledge


-This level gives to us a binary that takes an argument and copies it into a fixed-size buffer,we are about to perform an stack-based buffer overflow

### Source Code

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    // gcc's variable reordering fucked things up
    // to keep the level in its old style i am
    // making "i" global until i find a fix
    // -morla
    int i;

    void func(char *b){
	    char *blah=b;
 	    char bok[20];
	    //int i=0;

	    memset(bok, '\0', sizeof(bok));
	    for(i=0; blah[i] != '\0'; i++)
	  	    bok[i]=blah[i];

	    printf("%s\n",bok);
    }

    int main(int argc, char **argv){

  	    if(argc > 1)
		    func(argv[1]);
	    else
 	    printf("%s argument\n", argv[0]);

	    return 0;
    }

-The vulnerable function does a loop that copies bytes from input into bok[20] byte by byte until it hits a null byte \0

-If the input is longer than 20 bytes it overwrites adjacent dtack data (The *blah pointer saved ebp  and the return address


-The stack from of func() is:

-bok -20bytes
-*blah - 4 bytes
-ebp - 4bytes
-return address - 4 bytes

-We can say that 28 bytes reach the return address and the next 4 bytes overwrite it


## Solution Process


-In gdb we set a breakpoint at func+110 ,run the program with something like AAAA and examinate the stack with x/20wx $esp

-We confirm the values AAAA 0x41414141 ,in info registers we can reveal ebp too in this case at 0xffffda8c and the next value was 0xffffdcd1 which is main+23 confirming the return address location


-By adding one extra character (AAAAA) we observed that each character added sets the *blah  pointer one byte lower in memory,so we know how to calculate the address to place it in the payload



-Next we export a shellcode as an environment variable with a large NOP sled(\x90) that acts like a landing pad and points to the return address and slides execution into the shellcode

-In gdb with a breakpoint at main and running with no arguments and examining environ with x/s *((char **)environ) showed the SHELL=/bin/bash ,by adding +1 we get the next environment variable, the one we exported

-Adding +10 to its base address we skipped the SHELLCODE= landing directly in the NOP sled, the shellcode addres resulted to be 0xffffdc68

-Now to calculate the correct *blah value ,we run the program with 20 A characters and the output hesdumped xxd (out gdb),the blah pointer showed as 0xffffdc56 substracting 12 bytes of payload offset gave 0xffffdc4a as the value to place in the payload

-The final payloas structure was 20: A's to fill bok ,then the new *blah address ,then 4 A's for ebp and then the shellcode address targeting between the NOP sled

-the last 4 bytes of the payload were iterated through nearby addresses until one landed inside the NOP sled

### The payload

-This was the final payload used:

    ./narnia8 $(perl -e 'print "A"x20 . "\x4a\xdc\xff\xff" . "AAAA" . "\x80\xdc\xff\xff"')


-Now we just retrieve the password from the known directory



    $cat /etc/narnia_pass/narnia9


## Key Concepts Learned


- esp and ebp
   esp is the live stack pointer and ebp is a snapshot os esp taken at the start of the function and overwritable with overflow

- Return address control

   the saved return address on the stack tell the CPU where to go when a function ends ,overwriting it redirects execution wherever you want

-NOP sled ,a sequence of \x90 bytes placed before a shellcode,it creates a large target window so we dont need to hit the EXACT first byte of the shellcode

-Shellcode in environment variables

 placing shellcode in an env var keeps it in a predicatble high area of the stack,separate from the buffer being corrupted,making it easier to locate and target




