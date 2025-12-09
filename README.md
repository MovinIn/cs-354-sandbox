# cs-354-sandbox
The c programming language is not object oriented. It only uses functions. 
## main method
```
#include <stdio.h>
int main(void){
  return 0; //returns 0 after the program finishes successfully. 
}
```
## pointers
```
int* m; 
int *m2; // These two statements are the same.
int i=42;
m=&i; //the & is the address-of operator. m now holds the address to i.
int j=*m; //the * is the dereference operator. we get the value at the address m, which is of course, 42.
int a[3] = {1,2,3}; //array of integers. an array is a non-movable label at the address of the first element. 
int* p1 = a; //this pointer points to the address of the array.
*(p1+1) = 4; //We just set the value of the 1st index of the array. 2 => 4. a[1] and *(p1+1) are equivalent.
// ---------- 2d arrays --------------
int a1[2][3] = {{1,2,3},{4,5,6}}; //this is a 2d array.
// Doing the same thing with pointers:
int** a2 = malloc(2*sizeof(int*)); //allocate space for 2 pointers on the heap. malloc returns the address pointing to the first element of this space.
*(a2+0) = malloc(3*sizeof(int)); // allocate space for 3 ints on the heap. malloc returns the address pointing to the first element of this space, which the pointer at a2+0 is assigned to.
*(a2+1) = malloc(3*sizeof(int)); // allocate space for 3 ints on the heap. malloc returns the address pointing to the first element of this space, which the pointer at a2+1 is assigned to.
(*(a2+1)+2) = 6; // a2+1 is the address of the 2nd pointer. We then get the address of the pointee by doing *(a2+1). We then add 2 to that address to get the 3rd pointer. we then dereference and assign 6.
// the above is the same as a1[1][2] = 6;
//We cannot free the static a1 2d array because it lives on the stack. after the variable is popped off of the stack, stack memory is freed. 
//Doing the same with pointers:
free(*(a2+0));
free(*(a2+1));
free(a2);
```

## c-strings
```
char *sptr = "CS 354";  
char str[9]="CS 354";
sptr = "qwmdooamds"; //COMPILES; OK
str = "xoqwmoxm"; //DOES NOT COMPILE
```
Why? sptr is a pointer. We can change what the pointer is pointing to.  
str is NOT a pointer! It is an array. Because an array is a label to a memory address, we cannot change such label.  

`string.h` - library of useful functions to manipulate c strings. 

1. `int strlen(const char *str)` - returns the length of string `str` up to but not including the null character.  
2. `int strcmp(const char *str1, const char *str2)` - compares alphabetically `str1` pointee value with `str2` pointee value. 
3. `char* strcpy(char *dest, const char *src)` - copies the string pointed to by `src` to the memory pointed to by `dest` and terminates with the null character `\0`.
4. `char* strcat(char *dest, const char *src)` - Appends the string pointed to by `src` to the end of the string pointed to by `dest` and terminates with the null character.   

`Buffer Overflow`: when the array is not big enough to store all of the data. <br/>
`char a[3] = "asd"; //Buffer Overflow: compiler replaces "asd" with "asd\0" which is of length 4, but we try to fit it in an array of length 3.`



## command line arguments & program arguments
`command line arguments` -   
`program arguments` - enables information to be passed to the program before it begins



# Midterm 2
## Memory
### Virtual Address Space (VAS)
An illusion that a program uses. 
### System View
OS maps virtual address spaces to physical address spaces (PAS). The OS uses pages. 
### Hardware View
The pyramid: (Registers, Cache, Cache, Cache, Main Memory, Local Secondary Storage, Network Storage). Registers are fastest but smallest, Network is biggest but slowest. 
Stack grows down, heap grows up  
### DIY Heap using Posix API (Portable OS Interface)
`int brk(void* addr)`: sets top of heap to specified address `addr`  
Returns 0 if successful else -1 and sets `errno`  
`void* sbrk(intptr_t incr)`: changes top of heap ptr by `incr` bytes  
### Heap
A `heap` is a collection of various sized memory blocks that are managed by allocator.   
An `allocator` is a program that allocates and frees heap blocks as well as splits and merges them.   
Wants to maximize throughput and maximize memory utilization.   
If we run out of memory, we can:
1. coalesce adjacent free blocks (immediately \[after we free a block\] or delayed \[only when we need it\]).
2. ask linux kernel for more space.
3. return `NULL`
#### Implicit Free List
Used in Java.   
`new` operator implicitly determines the number of bytes needded.   
Garbage collector implicitly determines unused bytes and frees them.   
1. There is no explicit list.
2. Each block of memory has a header. Free blocks have a footer (which holds the length of the entire block).
3. We split and coalesce ONLY FREE BLOCKS (both are $$O(1)$$). 
4. Malloc is $$O(N)$$ and free is $$O(1)$$.
5. Must not move previously allocated blocks.
6. `a-bit`: status of block. `p-bit`: status of previous block. `0 = free`.
7. Types: first-fit, next-fit, best-fit.
    1. `first-fit`: traverse list until we find a block that fits.
    2. `next-fit`: given that the previous malloc function fit a block at address `x`, traverse implicit list starting from address of `x` (with wrap around).
    3. `best-fit`: tries to fit the block in the smallest free block.
8. `internal-fragmentation`: wasted memory by allocated block overhead (think: not-splitting blocks vs. splitting blocks).
9. `false-fragmentation / external-fragmentation`: wasted memory by allocated blocks not condensed, but spread out so that a big block of memory cannot fit, even if we have enough space (but its split up into chunks!)
#### Explicit Free List
Used in C. 
1. Uses a data structure containing just free blocks.
2. We traverse this free list in order when `malloc` is called, and find a free block that is suitable.
3. Worse in space, better in time (We only search free blocks in comparison to implicit free list).
4. Structure: Each free block node in the data structure has the following elements:
    1. A header containing the block-size + 0pa (with the last 3 bits) `// 4 bytes`
    2. An address pointing to the predecessor free block `//4 bytes`
    3. An address pointing to the successor free block `//4 bytes`
    4. A footer containing the block size (not including 0pa) `//4 bytes`. This is useful for faster coalescing with a free previous block. 
5. Ordering of the explicit free list
    1. address order: order blocks in free list from low to high addresses. 
    3. last-in order: most recently freed block is pushed to front of linked list.
6. Data structures
    1. Simple segregation: one free list for each block size (8, 16, 24, ...) `//A lot faster, but less memory efficient: cannot split nor coalesce adjacent free blocks.`
    2. Fitted segregation: one free list for small, medium, large block sizes. `//Slower, but more memory efficient: can split and coalesce blocks.`
### Locality and Caching
1. Temporal locality: When recently accessed memory is repeatedly accessed in near future. 
2. Spatial locality: When recently accessed memory location is followed by accessing memory that is nearby.
3. Stride: step size in words between sequential access in blocks.
4. Working set: set of blocks accessed during some time interval.
5. Cache miss: block not found in this cache.
    1. Cold miss: cache location is empty or invalid. 
    2. Capacity miss: cache is too small for working set. 
    3. Conflict miss: two or more blocks map to the same location.
6. Cache hit: block is in this cache!
7. Victim block: block that is going to be replaced. 
8. Placement policies:
    1. unrestrictive policy: memory block address can be placed in any line in a cache. 
    2. restrictive policy: `block# % size`.
9. Cache blocks must be big enough to capture spatial locality, but small enough to minimize latency (to copy block from memory to cache or register)
10. $$B = 2^b$$. $$S = 2^s$$. $$E$$ is associativity, or # of lines in a set. $$C = B * S * E$$. $$C$$ is the cache size.
#### Designing a Cache
1. Using sets and tags: this is a unrestrictive policy. We limit and restrict locations where a block can be stored in cache to a specific set so we can improve searching for blocks within a cache. A set is a collection of memory blocks somewhere in memory.
2. We put the b-bits and s-bits in the least significant bits to improve spatial locality (adjacent blocks within a set will be close together, and adjacent sets will be close together).
3. t-bits: bits of address stored in cache to identify which block is in the set.
4. A line is a location in cache that can store 1 block of memory.
6. v-bit: a status bit. If $$v=1$$, we have a valid set.
7. Processing a request
    1. Extract s-bits from address to get set number.
    2. Extract t-bits from address and compare with stored tag for cache line.
    3. If $$v=0$$ or t-bits do not match, it is going to be a `cache-miss`. Fetch from next lowest level!
    4. If $$v=1$$ and t-bits match, `cache-hit`!
##### Basic Cache / Direct Mapped Cache
1. Each cache set has only one line.
2. Conflict misses will occur very frequently; `thrashing`: we will continually exchange blocks in the same set.
##### Fully Associative Cache
1. A cache is only one set with $$E$$ lines.
##### Set Associative Caches
1. A cache having $$S$$ sets with each set having $$E$$ lines.
2. If $$E=4$$, the cache is known as a `4 way associative cache`. $$E$$ determines the associativity of a cache.
3. $$C = (S, E, B, m)$$.
##### Replacement Policies
1. Random replacement
2. Least recently used (LRU): track when block was last accessed.
    1. Use LRU queue (priority queue).
    2. Use status bits in each line to track
3. Least frequently used (LFU): track how often a line is used.
##### Writing Policies
When writing to a block in memory, we write it to the cache first. We can then update the lower levels of memory to reflect the changed value in the cache.   
`Write Through` is paired with `No Write Allocate`. `Write Back` is paired with `Write Allocate`. 
###### Write Hits
1. Write Through (Immediate)
    1. Slower; must wait. 
3. Write Back (Later)
    1. Must track that the block has changed (needs dirty bit). If it has, then we write it back in memory.
    2. Faster: No wait
###### Write Misses
1. No Write Allocate: Write directly into the lower cache, bypassing this cache.
    1. (+) Less bus traffic
    2. (-) Slower! Must wait for lower level to write.
2. Write Allocate: Read block into this cache and then write to it.
    1. (-) More bus traffic
    2. (-) Must wait for read from next lower level.
    3. (+) The read block will be at higher level, which reduces write misses and memory accesses.
##### Cache Performance
1. `hit-rate`: percentage of cache hits per accesses.   
2. `hit-time`: time to determine a hit.   
3. `miss-penalty`: additional time to process a cache miss.

We have options: 
1. Larger Blocks ((+) hit-rate, (-) miss penalty)
2. More Sets ((+) hit-rate: (+) temporal locality \[more sets, less blocks per set\], (-) hit-time: slower set selection)
3. More Lines per Set ((+) hit-rate: (+) temporal locality, (-) hit-time, (-) miss penalty)
A larger stride will result in a decreased `hit-rate`, because of bad `spatial locality`.
## Assembly
### Suffixes
A word in cache is 4 bytes, but in assembly is 2 bytes!  
`b`: 1 byte, `w`: 2 bytes, `l`: 4 bytes  
`mov`: moves bytes from one place to another 
### Operand specifiers
1. Immediate: `$13 = 13 //this is a constant. We can also do $0x13 for hexadecimal`
2. Register: `%ebx`
3. Memory: `(%ebx), (0xABCDEF), 4(%ebx), etc.`
    1. $$Imm(E_b,E_i,s) = Base + E_i * s + Offset$$
    2. $$E_b$$ and $$E_i$$ must be 32-bit registers.
    3. `Imm` can only be 1, 2, 4, or 8 bytes.
### Registers
`%eip`: Program Counter (PC)
`e-flags`: condition code registers. 
### Instructions
`MOV, PUSH, POP`: instructions to move from `S` to `D` (from REG/IMMEDIATE/MEM to MEM/REG. EXCLUDES MEM to MEM).   

// MIDTERM 2 END

#### MOV
1. `S` and `D` must be the same size.
2. We can use `movs` (sign-extend) or `movz` (zero-extend) if we want to copy from `S` to `D` with different sizes. As an example: `movswl`: moves `S` of size word to `D` of size double word.
#### PUSH
Add data to the stack.   
This pushes `S` to the top of the stack. 
1. make room on stack.
2. copy S to top of stack. 
#### POP
Remove data from the stack.   
1. copy top of stack to `D`. 
2. update stack pointer to shrink stack. 
#### LEAL
similar to MOV, but simply copies the address of `S` to `D`. So, `D <--- &S`.   
1. S must be a memory operand (must be in memory).   
2. Memory is not accessed.   
3. Condition Code registers are not set.
#### Shifts
##### Logical Shift (zero-fill)
##### Arithmetic Shift (sign-fill)
#### CMP and TEST
`CMP` compares values arithmetically. We subtract the two values to see which is bigger, smaller, or if they are equal.  
`TEST` compares values logically. We take the logical `AND` to see if the two values are equal or not equal.  
Both of these instructions only set `CC` registers; it does not change either operand. 
#### Condition Codes
1. `ZF`: zero flag; `ZF` = 1 if result = 0.
2. `CF`: carry flag; if result has unsigned overflow. 
3. `SF`: sign flag; `SF` = 1 if result < 0.
4. `OF`: overflow flag; `OF` = 1, if result has 2's complement overflow.
#### SET
set a byte register to 1 if a condition is true, set to 0 if false.  
```
sete D // == equal
setne D //!= not equal
sets D // < 0 signed (negative)
setns D // >= not signed (nonnegative)
setz // sets byte register if zero flag is 1
setnz // sets byte register if zero flag is 0
```
#### JMP
jump to a specific instruction. 
##### Unconditional Jumps
`jmp *Operand`: jumps to effective address at `*Operand` `//indirect jump`
`jmp Label`: jumps to label  `//direct jump`
##### Conditional Jumps
`je, jne, jg, ja, jb, etc`
### Stack and Stack Frames (SF)
`Stack Frame`: A block of stack memory used by a single function call.   
`%ebp`: base pointer. Points to the base of `SF`.  
`%esp`: stack pointer. Points to the top of `SF`.   
When pushing stack frames: ie. function X calls function Y, then the return address of `X` is pushed onto the stack. `%ebp` is pushed onto the stack. Set `%ebp` to `%esp`, so points to old `%ebp`. Then add local variables on stack by subtracting from `%esp`. After function Y ends, then we only need to set `%esp` to `%ebp`, effectively popping Y off the stack. Then we pop old `%ebp` off the stack and set that to `%ebp`. Then we pop the return address and set that to `%eip% (Program Counter).  
### Transferring Control
1. Flow Control
    1. function call like unconditional jumps (call => jmp)
3. Stack Frames
    1. return (ret => popl %eip)
    2. leave (frees stack frame) `movl %ebp, %esp \\ popl %ebp`
### Register Usage Conventions
1. `%eax` usually stores return values.
2. Callee uses `%ebp` to access:
    1. Callee's arguments `8(%ebp)`
    2. local variables `-4(%ebp)`
3. Caller uses `%esp` to:
    1. setup args to fn call `// movl ARG, 4(%esp)`
    2. save return address `// ret`
4. Caller-save: caller must save/restore registers before calling callee function. `// %eax, %ecx, %edx`.
5. Callee-save: callee must save/restore registers before using them. `// %ebx, %esi, %edi`. 
### Recursion
`Direct Recursion`: when function calls itself.   
`Recursive Case`: same as math240 `//case when function calls itself.`
`Base Case`: same as math240 `//stops the recursion.`
`Infinite Recursion`: we never reach the base case. This causes `Stack Overflow`, because each recursive call consumes more memory in stack. 
### Stack Allocated Arrays
The first element of an array is at the top of the stack (lower addresses). So we add bytes to the starting address to index through the array.   
### Stack Allocated Multidimensional Arrays


## Function Pointers
A `function pointer` is a pointer to code, storing the address of the first instruction of a function.   
Allows functions to be stored in arrays and allows functions to be passed to and returned from other functions.   
### Example
```
int func(char c) { }
int (*my_function_ptr) (char); //creates a function pointer called my_function_ptr, holding an address of the first instruction of a function with a return type of int and parameter of char
my_function_ptr = func;
int x = my_function_ptr('c');
```
## Buffer Overflow / Stack Smashing
`Buffer Overflow` is when we exceed the bounds of the array. Dangerous for stack allocated arrays.   
Buffer Overflow can overwrite data outside the buffer and also the state of execution.   
## Flow of Execution
`Exceptional Control Flow`: special execution that enables the system to react to an unusual event.  
An `event` is a change in processor state that may or may not be related to current instruction.   
`processor state`: a processor's internal memory elements, registers, flags, signals, etc.
## Exception Usage

<img width="896" height="538" alt="image" src="https://github.com/user-attachments/assets/f05c5ff7-22a3-435b-b510-4c5fbb74b19e" />

## Exceptions
An `exception` is an event that sidesteps the usual logical flow.   
`Exceptions` can originate from hardware or software. The response is an indirect function call that abruptly changes the flow of execution.   
`Asynchronous Exception`: results from some event unrelated to the current flow of execution.   
`Synchronous Exception`: results from the current instruction.   
### General Exceptional Flow
Normal Flow => Exceptional Event Occurs => Transfer control to Exceptional Handler => Exceptional Handler => Return control (to $$I_{curr}$$, $$I_{next}$$, doesn't return (seg fault))  
### Kinds of Exceptions
1. Abort
    1. cleanly ends a process
    2. synchronous
    3. does not return
2. Fault
    1. handles problems with current instruction
    2. potentially recoverable error
    3. synchronous
    4. might return to $$I_{curr}$$ and re-execute it.
3. Interrupt
    1. Enables devices to notify or signal that it needs attention. 
    2. Signal from external device
        1. Device signals an interrupt.
        2. OS finishes the current instruction.
        3. transfer control to appropriate exception handler
        4. transfer control back to interrupted process's next instruction
        5. Kind of like push notifications (vs. polling which is like RPC: constantly checking periodically \[wasteful\])
    4. asynchronous
    5. returns to $$I_{next}$$.
4. Trap
    1. Enables a process to interact with OS
    2. intentional exception
        1. process indicates that it needs an OS service.
        2. `int`: interrupt assembly x86 instruction.
        3. transfer control to the OS system call handler which calls the service function.
        4. transfer control back to process's next instruction. 
    4. synchronous
    5. returns to $$I_{next}$$.
### Transferring Control via Exception Table
Exceptions transfer control to the kernel.  
1. Push return address to top of kernel stack ($$I_{curr}$$ or $$I_{next}$$.   
2. push interrupted process's state so it can be restarted
3. Do indirect function call which runs the appropriate exception handler
    1. `indirect function call`: uses an exception table to determine the exception handler
    2. `exception table`: a jump table for exceptions that is allocated memory by OS on system boot.

<img width="1595" height="296" alt="image" src="https://github.com/user-attachments/assets/fabb80c9-5f35-4dc9-94c2-3dca58fb9c16" />

### Exceptions and System calls in IA-32 & Linux
0 - 31 are defined by processor
Page Fault - we need a data value that is in lower levels of memory

32 - 255 are defined by OS


### Context Switch
A context switch may result from some exception. This happens when the OS decides to switch to another process while the current process is handling an exception.  
Switching from kernel A to kernel B: `Save context of "A", Restore context of "B", Transfer Control to "B"`  

## Multifile Coding
divide programs into functional units (header file `.h` and source file `.c`).   
One Definition Rule: an identifier can be defined only once in the global scope  

<img width="742" height="404" alt="image" src="https://github.com/user-attachments/assets/4121a408-db3a-445c-b294-5c9e92762e7b" />

### Reasoning
Include guards  
#ifndef: if not defined: otherwise, skip to endif
#define: define the identifier  

### Multifile Compilation

<img width="1788" height="471" alt="image" src="https://github.com/user-attachments/assets/100436ff-87ef-4925-b11d-2e5142831bb0" />

### Executable and Linkable Format

<img width="518" height="576" alt="image" src="https://github.com/user-attachments/assets/66de0e62-0f94-4255-870f-4f3a0590e8e6" />


`.symtab`: linker symbol table - global vars and extern functions  
`.strtab`: names of symbols in `.symtab` are stored in `.strtab`  

### Static Linking 
Static linking generates a complete EOF with no variable or function identifiers remaining in the OF

#### Comparison with Dynamic Linking
Dynamic Linking: does not replace external functions with actual code, but dynamically linked during runtime
Executable size: smaller; but all library code is dynamically linked.
### Linking
Symbol Resolution: makes sure variables and identifiers follow ODR  
Relocation: variables and function identifiers are replaced with addresses  
#### Making things private
Functions and global variables only in a source file and not the header are "intended" as private, but not actually always private. They can still be accessed by units that declare the variable or fn with "extern".   
We can truly make a variable ALWAYS private by declaring them as static.   
#### Linker Symbols
Variables located in the `.data` segment need linker symbols  
Linker Symbols are symbols managed by the linker, created by compiler.   

<img width="1308" height="635" alt="image" src="https://github.com/user-attachments/assets/e082f6ee-f427-4849-8ecb-7e813b491970" />

All functions need linker symbols for relocation, likely all for resolution  

<img width="1218" height="485" alt="image" src="https://github.com/user-attachments/assets/be25e57f-cbfa-49b7-9ce1-25eb1dbbecb2" />

