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
    1. $$Imm(\%E_b,\%E_i,s) = Base + \%E_i * s + Offset$$
    2. $$E_b$$ and $$E_i$$ must be 32-bit registers.
    3. `Imm` can only be 1, 2, 4, or 8 bytes.
### Instructions
`MOV, PUSH, POP`: instructions to move from `S` to `D` (from REG/IMMEDIATE/MEM to MEM/REG. EXCLUDES MEM to MEM).   

// MIDTERM 2 END

#### MOV
1. `S` and `D` must be the same size.
2. We can use `movs` (sign-extend) or `movz` (zero-extend) if we want to copy from `S` to `D` with different sizes. As an example: `movswl`: moves `S` of size word to `D` of size double word.
#### PUSH
This pushes `S` to the top of the stack. 
1. make room on stack.
2. copy S to top of stack. 
#### POP
1. copy top of stack to `D`. 
2. update stack pointer to shrink stack. 
#### LEAL
