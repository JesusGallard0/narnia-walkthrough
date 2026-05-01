# Narnia Overthewire level 3-4


## Problem Explanation

-Narnia3 gives us a program that reads a file you specify and copies its contents to /dev/null,basically throwing them away.

-The challenge is to make it read a protected password file and save the output somewhere we can actually read it.

-The password we need is stored in /etc/narnia_pass/narnia4 but mainly,we are not allowed to read it,but the program can read it because it runs with narnia4 privileges.

-We just need to trick it into saving the output somewhere useful.

## Background Knowledge

### Buffer overflow

-In C, variables are stored next to each other in memory.When a program copies data into a variable without checking size,it can spill over into the next variable.

-The C function strcpy stands out in this case because it copies bytes blindly until it hits a null .


### Stack Memory Layout

-Local variables in a C program are laid out sequentially, meaning if you overflow one variable,you overwrite whatever comes right after in memory.For example in this program:

    [ifile 32 bytes][ofile 16 bytes]

-ifile comes first,ofile comes right after.Overflow ifile and you control ofile


### Symbolic Links

-A symlink is a file that points to another file.When a program open a symlink ,the OS redirects its content to the target.We use this to make the program read /etc/narnia_pass/narnia4 without specifying that path as our overflow payload


### The Vulnerabiliry

-Looking at the source code the critical lines are:

     char ofile[16] = "/dev/null";
    char ifile[32];


    strcpy(ifile, argv[1]);


-ofile is the output destination (/dev/null) .ifile is where our input argument gets copied .Since ifile is only 32 bytes but sits right before ofile in memory,if we pass an argument longer than 32 bytes, we overflow ifile and overwrite ofile, replacing /dev/null with whatever we want


## Solution Process

### Create the nested directory structure

    mkdir -p /tmp/axaxaxaxaxaxaxaxaxaxaxaxaxa/tmp/

-This creates a long path on purpose

-The long part /tmp/axaxaxaxaxaxaxaxaxaxaxaxaxa is just padding to fill and overflow ifile's 32 bytes.The /tmp/ at the end is what will land inside ofile, replacing /dev/null as the output destination

### Create the symlink

-ln -s /etc/narnia_pass/narnia4 /tmp/axaxaxaxaxaxaxaxaxaxaxaxaxa/tmp/trick

-We create a symlink called trick inside that nested directory,pointing to the password file.

-When the program opens this symlink as its input file,the OS redirects it to /etc/narnia_pass/narnia4

-We can't point directly to the password file in out argument because we need the argument to be a specific lenght for the overflow to work precisely.

### We create the output file

    touch /tmp/trick
    chmod 777 /tmp/trick


-We create /tmp/trick as our output destination and make it world writable so the program can write to it.

-This is the file that ofile will point to after the overflow, replacing /dev/null.


### Execute the exploit

    ./narnia3 /tmp/axaxaxaxaxaxaxaxaxaxaxaxaxa/tmp/trick

### When this runs

-strcpy copies our long argument into ifile which is only 32 bytes

-The overflow spills into ofile, replacing /dev/null with /tmp/trick

-The program opens trick via the symlink, which redirects to the password file

-It reads the password and writes it to /tmp/trick instead of /dev/null


### Read the password

    cat /tmp/trick


## Key Concepts Learned

### strcpy is not safe

-it copies without checking size.Any time you see strcpy in C code,think potential overflow.The safe alternative is strncpy which takes a maximum lenght.

### Local variables sit to each other in the stack

-Overflowing one doesn't just corrupt that variable,it corrupts whatever lives next to it.




