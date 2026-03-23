We may reference the [6.2.11 Designated Initializers](https://gcc.gnu.org/onlinedocs/gcc/Designated-Inits.html) in the using the GNU Compiler Collection. This articale also reference [[Linux Kernel慢慢學]探討Designated Initializers](https://meetonfriday.com/posts/39485259/)
## What is designator
>The `[index]` or `.fieldname` is known as a _designator_.

The `[index]` is used to initialize the array and `.filename` is used to initialize the element in the structure.

---
## Why we need designated initializer
>Standard C90 requires the elements of an initializer to appear in a fixed order, the same as the order of the elements in the array or structure being initialized.

>In ISO C99 you can give the elements in any order, specifying the ==array indices== or ==structure field names== they apply to

---
## How to use it 
### Array
```c
int a[6] = { [4] = 29, [2] = 15 };
```
is equivalent to 
```c
int a[6] = { 0, 0, 15, 0, 29, 0 };
```
>The index values must be constant expressions, even if the array being initialized is automatic.
#### Question : In the declaration `int a[6] = { [4] = 29, [2] = 15 };`, how does the C standard ensure the entire array is initialized, and what specific values are assigned to the elements not explicitly designated?
In the C99 standard 6.7.8.21, they say 
>If there are fewer initializers in a brace-enclosed list than there are elements or members of an aggregate, or fewer characters in a string literal used to initialize an array of known size than there are elements in the array, the ==remainder of the aggregate shall be initialized implicitly the same as objects that have static storage duration==.

The remaining elements will be viewed as objects that have static storage duration, which will be initialized as 0.
### Structure
For this structure
```c
struct point { int x, y; };
```
The following initialization
```c
struct point p = { .y = yvalue, .x = xvalue };
```
is equivalent to 
```c
struct point p = { xvalue, yvalue };
```
### Combine these two data structure
>You can also write a series of `.filename` and`[index]` designators before an ‘=’ to specify a nested subobject to initialize; the list is taken relative to the subobject corresponding to the closest surrounding brace pair.
```c
struct point ptarray[10] = { [2].y = yv2, [2].x = xv2, [0].x = xv0 };
```
### Combine this technique with ordinary C initialization of successive elements
```c
int a[6] = { [1] = v1, v2, [4] = v4 };
```
is equivalent to 
```c
int a[6] = { 0, v1, v2, 0, v4, 0 };
```
### Labeling the elements of an array initializer is especially useful when the indices are characters or belong to an `enum` type.
```c
int whitespace[256]
  = { [' '] = 1, ['\t'] = 1, ['\h'] = 1,
      ['\f'] = 1, ['\n'] = 1, ['\r'] = 1 };
```

---
## Q1 : How about the element is initialized multiple times ? 
>If the same field is initialized multiple times, or overlapping fields of a union are initialized, the value from the ==last initialization is used==.

