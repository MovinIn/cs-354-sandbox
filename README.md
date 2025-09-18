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
