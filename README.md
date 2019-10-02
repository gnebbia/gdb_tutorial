# GDB Tutorial

## Intro

Debuggins is the art and science of finding and eliminating bugs 
in software, bugs can be simple functional issues or can have 
security implications. When we compile a program automatically we 
have no debugging symbols, in order to open a program with gdb we 
do:

```sh
 gdb ./programName
 # opens programName with gdb
```

Once we are in GDB we can run the program by doing: 
```sh
 run parameter1 parameter2 
 # in this case we run the program passing the specified parameters
```

if the program was compiled with the so called "debugging symbols", we will will
have useful information about variables, functions, and other stuff.
Debug symbols can either be integral part of the binary file or can be placed 
in a separate file.

We can disassemble a program by doing:

```sh
 disassemble main
 # this will disassemble main, an arrow drawn like "=>" will show what EIP
 # is pointing to, so what is the next instruction which will be executed
```

we can attach gdb to a running process by doing:
```sh
 attach pid
 # attaches GDB to the specified process ID
```

we can also attach to an existing process by launching gdb in quiet mode, so
that the boilerplate initial information does not appear in the output by doing:
```sh
gdb -q -p <PID>
```

if the program crashes we can run:
```sh
 backtrace
 # show the current stack, an alias is "bt"
```


### Debugging Symbols

We need to be explicitly mention the intention to create debug 
symbols at compile time, we have different kind of debug symbol 
file types:

* DWARF 2
* COFF
* XCOFF
* Stabs

we have two options with gcc in order to compile with debug 
symbols:

* "-g" flag: in order to compile a program with debug symbols 
  with a format taken from the Operating System;
* "-ggdb" flag: in order to compile a program with GDB specific 
  debug symbols , these are the best one understood by gdb.

In a practical compilation scenario we can include debug symbold by doing:
```sh
 gcc -ggdb programName.c -o programName
```

### What Symbol Files tell us ?


The information provided by debug files and debug symbols can be summarized as
follows:

- info on **sources**: so we will have the source code available, 
    done with "list 1" or "list", or we can see the source file 
    name with "info sources"
    N.B.: if we rename or delete the source code file/files we won't 
    be able to list source code
- info on **functions**: list of functions available with relative name of the source,
    we can do this with "info functions";
- info on **(global) variables**: list of variables available, we can do this
    with "info variables", this will list only global variables;
    Notice that in order to get info on local variables we have to specify 
    the scope, we can do this by with "info scope" + tab tab will give us the list
    of available scopes and we do for example "info scope functionName"
    or "info scope main".

In order to detach (or strip) symbols from a binary file and save them in a separate 
file we can do:
```sh
 objcopy --only-keep-debug <BinaryFile> <OutputDebugFile>
 # we will put debug symbols of the program binaryFile in the file 
 # called debugFile, without deleting them from the binary
```

we can attach debug symbols to a binary by doing:
```sh
 objcopy --add-gnu-debuglink=<InputDebugFile> <BinaryFile>
 # this will add the debug symbol file called "InputDebugFile"
 # to the binary program called "BinaryFile"
```

we can simply strip debug symbols from a binary by doing:
```sh
 strip --strip-debug <BinaryName> 
 # this will delete debug symbols from <BinaryName>
```

Anyway, remember that also after doing this removal operation there will still
be additional informations that can be used by reverse engineers.

We can delete everything which is unnecessary by doing:
```sh
 strip --strip-debug --strip-unneeded <ProgramName>
 #this will delete all the debug symbols, we won't see even the function names
```

so in order to not let other view my code and make the 
life of reverse engineers harder we can use the second strip command, 
this will also make the binary smaller.

So summarizing with debugging symbols we can:
- `list 1`, list source code
- `info sources`, list information on the source file, such as name, and possibly other informations
- `info functions`, list functions
- `info variables`, list flobal variables
- `info scope <FunctionName>`, list local variables in the specified function
- `info files`, lists all the sections and their addresses, like ".text", ".bss", ".data", etc...



#### Loading a symbol file in GDB


In order to load a symbol file in GDB we do:
```sh
 symbol-file fileName 
 # this command inside gdb will load the symbol file called "fileName"
```

## Inspecting Symbols with "nm"

The program "nm" will list symbols contained in an object file, so we can 
inspect symbols related to a file by doing:
```sh
 nm ./programName 
 # this will list all the symbols of the program "programName"
```

By default the list will be composed by 3 rows where:

1. the first, represents the virtual address;
2. the second, denotes the symbol type;
3. the third, denotes the symbol name.


### Symbol Types

For symbol types we have:


| Symbol Table |                 Meaning                 |
|:------------:|:---------------------------------------:|
|       A      |             absolute symbol             |
|       B      | in the uninitialized data section (BSS) |
|       D      |     in the initialized data section     |
|       N      |             debugging symbol            |
|       T      |           in the text section           |
|       U      |        symbol undefined right now       |


We moreover can encounter both "uppercase" or "lowercase" symbols:

* lowercase symbol is a "local" symbol
* uppercase symbol is an "external" symbol

for a complete list of symbols we do `man nm`.

### Other nm examples

```sh
 nm -a | grep functionName
```

  -- this can be useful if we want to find a specific function to 
    which executable is owned

  -- this can be useful even by specifying a specific symbol type 
    instead of function name

```sh
 nm -n #display all the symbols in sorted order
```
```sh
 nm -g #displays all the external symbols
```
```sh
 nm -S #displays the size of the corresponding object for each 
  symbol
```


## Strace

The "strace" tool will help us understand how our program 
interacts with the OS, this tool traces all system calls made by 
the program, it even tells us about arguments passed and has 
great filtering capabilities. Let's see some examples:

```sh
 strace ./programName 20 30
```

```sh
 strace -o report.txt ./programName 
 # in this case with the option "-o" we store the output to another file
```

```sh
 strace -t ./programName 
 # this will save the time at which each system call is called
```

```sh
 strace -r ./programName 
 # this will save the relative time taken by each system call,
 # this can be helpful even to understand which system call takes 
 # more time and which takes less time
```

```sh
 strace -r -e write ./programName 
 # this will filter the output by putting only the system call "write"
```

```sh
 strace -e connect nc www.google.it 80 
 # this will filter the output by putting only the system call "connect"
```

```sh
 strace -e send,recv nc www.google.it 80
 # this will filter the output by putting only the system calls
 # "send" and "recv"
```

We can even attach "strace" to a running process:

```sh
 strace -p processID
 # in this case we attach strace to a running process, we have to specify
 # the process ID we want to attach to
```

we can get even list and statistics of system calls used, in this 
case we do:

```sh
 strace -c ./programName
 # this allows to print statistics on system calls and let us understand
 # which system call a program is using, for example if we see 
 # "send", "connect", etc... we understand that this is a program 
 # which communicates over the network
```

all of this is important because when we are in GDB we can set "breakpoints"
on one of these system calls.


## Breakpoints, Registers and Memory

A breakpoint is a technique used to "pause" the program during 
execution based on certain criteria, these criteria can be for 
example "before to execute a specific instruction" (which we want 
to examine).

### Breakpoints


There are different ways to set a breakpoint:

* breakpoints on functions
  * `break functionName` : will set a breakpoint on the function 
* breakpoints on instruction addresses
  * `break *0x080484cd` : will set a breakpoint on an instruction 
    address, for example as shown by "disassemble main", we need 
    the asterisk before the address
* breakpoint on line number
  * `break 54` : will set a breakpoint on line number 54

once a breakpoint is set, we can run the program with:

```sh
 run param1 
 # this will run the program, using as parameter "param1"
```

once the program is frozen we can inspect various informations, 
for example using:

```sh
 info registers
```
or with:

```sh
 info all-registers 
 # this will dump all registers to screen
```

we can see the list of active breakpoints with:

```sh
 info breakpoints
 # this will list breakpoints
```

we can enable/disable breakpoints once we have seen the id number 
by doing:

```sh
 disable 1 
 # this will disable the breakpoint with id number equal to 1
```

```sh
 enable 1
 # this will enable breakpoint 1
```

```sh
 delete 1
 # this will delete breakpoint 1
```

we can continue the execution after a breakpoint with the 
instructions:

```sh
 continue
 # continues until the end or until the next breakpoint (if there are any)
```

```sh
 step 
 # will execute and point to the next line of C code, we can pass an argument
 # "n" to specify the number of code lines to step
```

```sh
 stepi
 # steps one assembly instruction exactly, we can pass an argument "n" to
 # specify the number of code lines to step
```

```sh
 finish
 # run until the end of the current function
```

```sh
 advance <location> 
 # in this case we advance to a specific location, which can be "somefunction"
 # or "5" (a line number) or "hello.c:23" a line number in a specific file
```

#### Conditional Breakpoints


Sometimes we want to use breakpoints only if certain conditions 
are met, these conditional breakpoints could be very handy for 
example in case of loops. Let's see some example, let's say we 
put a breakpoint on line "10" of our code for example with:

```sh
 break 10
 # set a breakpoint at line 10
```
we can now view its id number with:

```sh
 info breakpoints
```
now let's say our breakpoint has as id number "1" we can do:

```sh
 condition 1 counter == 5 
 # in this case we are telling GDB to 
 # breakpoint (at line 10) only when the variable counter is equal 
 # to 5
```

let's see another example:

```sh
 break *0x08048478 
```

```sh
 condition 1 $eax != 0
 # this will stop the program only when $eax is different from zero at the 
 # instruction specified at the address mentioned in the break instruction
```

#### Watch Breakpoints


We can even set breakpoints when certain events happen, for 
example when a variable is written/read or written or read from 
it, let's see some example:

```sh
 watch variable1 
 # in this case we break when a variable is written to
```

```sh
 rwatch variable1
 # in this case we break when a variable is read from
```

```sh
 awatch variable1
 # in this case we break when a variable is written to or read from
```

```sh
 info watch
 # is the same thing as info break
```

### Examine Memory


In order to examine memory we use the commands: 

* x: to examine memory
* print: which is useful to print variables
* display: it is useful to look at the evolution of a certain variable

the "x" command has a very specific format which has to follow, 
the general format is:

```sh
 x/<number><suffix>
 # where instead of "number" we specify the numerical quantity of data, 
 # and with "suffix" we specify actually the suffix
```
Let's make some example:

```sh
 x/s argv[1]
 # this will print the string argv[1]
```

```sh
 x/i $eip 
 # this will show which instruction is the register eip pointing to
```

```sh
 x/10i $eip 
 # this will show which instruction is the register eip pointing to 
 # and additional following 9 instructions (in total 10 instructions)
```

```sh
 x/10xw $esp 
 # this will show 10 words (32bit) in hexadecimal format starting from
 # the esp register
```

```sh
 x/10xg $rsp 
 # this will show 10 giants (64bit) in hexadecimal format starting 
 # from the rsp register
```

```sh
 x/5i $pc
 # shows the next 5 assembly instructions, this is useful to avoid doing 
 # everytime "disassemble main" in order to inspect the code, 
 # another common instruction is "layout asm"
```

```sh
 print argv[0]
 # this will print the argv[0] variable
```

Notice that "print" is equivalent to "x/s".

Let's see an application of "display" and "undisplay":

```sh
 display variable1
 # in this case at every step we print out the value of the variable "variable1"
```

```sh
 disp/i $pc 
 # this will show the next instruction at each step, useful if combined 
 # with stepi
```

we can look at the current displays at:

```sh
 info display
```

and we can undisplay with:

```sh
 undisplay 1
 # where "1" in this case is the id number of the interested display
```

#### Modify Memory


Let's take an example, in which we take start a program with gdb 
and want to modify some content of the memory. We would do:

```sh
 gdb ./programName AAAA 10 20 
 # in this case our program let's say for the sake of the example 
 # that takes 3 arguments as inputs
```

```sh
 break main
 # set the breakpoint to main
```

now we can for example look at one of the passed arguments with:

```sh
 x/s argv[1]
```

or view it character by character with:

```sh
 x/5c argv[5] 
```

the above instructions will show us the starting memory address 
of the argument, we can change it by doing:

```sh
 set {char} 0xbffff7e6 = 'B'
 # in this example we have put the starting memory address of argv[0] 
 # and put it in a char (1Byte) the character "B" we will see a number 
 # the next time in memory at that address, that number is the ascii code 
 # of 'B', notice that we can view the ascii table by doing "man ascii"
```

alternatively we can achieve the same thing by inserting directly 
the ascii code of 'B', for example:

```sh
 set {char} 0xbffff7e6 = 66
```

we can even set more bytes at a time, for example:

```sh
 set {int} 0xbffff7e6 = 12
 # in this example we will have the first char as 12, but the following 
 # set at zero, since the int takes more bytes
```
now if we want to write at consecutive memory addresses (e.g., we 
want to put all B's in the same memory address and following 3 
bytes) we do:

```sh
 set {char} 0xbffff7e6 = 'B'
```

```sh
 set {char} (0xbffff7e6 + 1) = 'B'
```

```sh
 set {char} (0xbffff7e6 + 2) = 'B'
```

```sh
 set {char} (0xbffff7e6 + 3) = 'B'
```

we can even change the value of other variables, for example 
let's suppose there is a variable called "sum" and we want to 
change its value, in this case we would do:

```sh
 set sum = 2000 
 # this will set the value of the variable called "sum" to 2000
```

we can even change the value of registers:

```sh
 set $eax = 10 
 # this will set the value of the register eax to 10
```

```sh
 set $eip = 0x80484c4 
 # this will set the instruction pointer to the address mentioned, 
 # we took this address by making "print functionName" in this way we know at 
 # which address the function is, it's very probable that this will make a 
 # "segmentation fault" since when the function will terminate, 
 # it will return to the address specified at the top of the stack, so it's very
 # probable that at the stack there is garbage, since we forced 
 # the loading of an arbitrary function so when the function will 
 # terminate we will have a segmentation fault, there is an 
 # exception to this, in case the function calls the "exit()" system call
```

### Modify and Patch Binary


We can rewrite a binary file in multiple ways, for example by 
launching gdb with the option "--write" or by executing specific 
commands inside gdb, let's see both examples:

```sh
 gdb --write -q ./a.out 
 # in this case we run the binary a.out with write permissions, 
 # the flag "-q" it's used to suppress introductory messages and copyright 
 # notice, so it's a "quiet mode"
```

or if we launch gdb without the option "--write" we can execute:

```sh
 set write on 
 # in this case we set the binary in write mode, so if we rewrite instructions
 # the binary will be modified, it is always a good idea to make a copy 
 # of our binary before doing this
```

```sh
 set write off 
 # in this case we disable the write mode, so the binary becomes again read-only
```

```sh
 show write 
 # displays whether executable files and core files are opened 
 # for writing as well as reading
```

once in write mode we can put:

```sh
 set {unsigned char}0x00000000004004b9 = 22
 # modifying part of the binary
```

## Convenience Variables and Routines

We can create variables in GDB to hold data, these variables are 
called "convenience variables", and can be set through the "set" 
command, let's see some example:

```sh
 set $i = 10
```
```sh
 set $dyn = (char *)malloc(10)
```

  -- we can do this only after the program is running, and we 
    could inspect this for example by doing "x/10xb $dyn" that 
    will show us the first ten bytes starting from $dyn, so the 
    content of "dyn"

```sh
 set $demo = "joe"
```

```sh
 set argv[1] = $demo
```
we can even call C routines for example:

```sh
 call strcpy($dyn, argv[1])
 # in this case we are copying the content of argv[1] into $dyn,
 # we can inspect $dyn by doing "x/10xb $dyn" or "x/10c $dyn"
```

we can even call local functions listed with "info functions", 
for example we can do something like:

```sh
 call AddNumbers(10,20)
 # this will print on the screen the result given by the function
```

```sh
 call AddNumbers($i, $j)
 # this will print on the screen the result given by the mentrioned function,
 # notice that this time we passed as arguments two convenience variables
```

Notice that when we start a program and want to check different 
functions what we do is usually setting a breakpoint at "main" 
and then use the "call" instruction.

## Window Commands

We can run gdb in a windowed TUI mode with:

```sh
 gdb ./programName -tui
```
from here we can list windows and do other operations:

```sh
 info win
 # we list windows
```

```sh
 focus winname 
 # set focus to a particular window by name or position "next" or "prev",
 # we can even use "fs" as alias for "focus"
```

```sh
 layout type
 # where the layout type can be "asm", "src", "split" or "reg"
```

```sh
 layout asm 
 # very useful for navigating assembly source code
```

```sh
 tui reg typ
 # set the register window layout "general", "float", "system" or "next"
```

```sh
 winheight val
 # set the window height to the chosen value, an alias is "wh"
```

## Practical Scenarios

### Practical Scenario: Cracking a Simple Binary with Debug 

#### Symbols

We have a program that wants us to insert the secret password as 
parameter, in a poorly written code what we could simply do is:

```sh
 strings programName 
 # the "strings" program will show the strings contained in the binary
```
this can show us sometimes interesting strings about the program 
in our case for example there were interesting strings and we 
tried one of those to crack the program. Poorly coded programs 
may reveal private/secret informations. Secrets can be easily 
hidden by encryption/encoding. 

We do some inspection generally with:

```sh
 info functions
```
```sh
 info variables
```
```sh
 info scope interestingFunction
```
```sh
 break main 
 # this is followed by some "call interestingFunction()"
```
```sh
 print interestingVariableName
```
### Practical Scenario: Disassembling and Cracking a Simple 

####  Binary

Here we deal with assembly, now there are two main syntaxes:

* AT&T
* Intel

GDB as the GNU as assembly uses by default the AT&T syntax. We 
can change the assembly syntax by doing:

```sh
 set disassembly-flavor intel 
 # here we cahnge the syntax to intel, but we can check the 
 alternatives by just pressing "tab-tab" after "disassembly-flavor"
```

In this case what we do is:

```sh
 disassemble main 
 # here we will see the assembly instructions of our program
```

we always have to remember that the assembly code we see is 
dependant on the architecture for which the program was written, 
for example in a qemu virtual machine with debian armel 
application we would see arm assembly code, while on an x86_64 
architecture we see an x86_64 assembly code.

### Practical Scenario 2


```sh
 layout asm
```
```sh
 disp/i $pc
```
```sh
 stepi
```

#### Assembly Tips

```sh
 printf '#include <stdio.h>\nint main(void) { printf("Hello, world!\\n");
 return 0; }\n' | gcc -x c -S - -o - -fno-asynchronous-unwind-tables
```

where:

* `-x c` selects C as it cannot be determined otherwise when getting 
  source code from standard input 
* `-S` tells the compiler to stop compilation after generating the assembly,
  but before assembling it 
* `-` means standard input 
* `-o` means write to standard output 
* `-fno-asynchronous-unwind-tables` disables special exception 
  suggest that generates lots of noise like .cfi_offset 

If you do this, then you get a minimal hello world program in 
assembly, which uses the standard library You can write your own 
".s" file and assemble it. Use -m32 if you're on a 64-bit machine 
and want to try 32-bit assembly.
