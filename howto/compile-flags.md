# Helpful compiler flags

By default the compiler will only show warning/error messages that it thinks are most important for you to see. Usually this is fine, however on occasion your code may contain bugs which **are** critical enough to cause problems, but which the compiler does **not** believe are important enough to inform you about.  Asking the compiler to be more verbose in the warnings and errors that it reports may help identify subtle problems.

This guide assumes you are using the GNU Compiler Collection (`gcc`), however these options will likely work with other compilers as well, such as LLVM Clang (`clang`).

## Show ALL errors and warnings

To request that the compiler display **all** errors and warnings, no matter how seemingly insignificant, use the `-Wall` and `-Wextra` flags.  For instance run:

`gcc -Wall -Wextra yourcode.c`

## Example

For instance, consider the following code which is meant to read and print one integer.

```
#include <stdio.h>
int main() {
  int i;
  scanf("%d", i);
  printf("i: %d\n", i);
}
```

This code looks reasonable at first, but upon closer inspection note the missing ampersand (`&`) in the `scanf()` function call.  A common mistake amongst newer C programmers. By default GCC will silently allow this, and it will result in a segmentation fault at runtime.

```
andrew@dev:~$ gcc example.c
andrew@dev:~$ ./a.out 
1337
Segmentation fault
```

However when compiled with these additional flags, GCC warns the programmer of the problem, even giving the exact line and location where the issue was noted!  The error message is less helpful (maybe we haven't yet learned what `int *` means), but this gives you a clue as to where the problem is and where a fix is likely needed.

```
andrew@dev:~$ gcc -Wall -Wextra example.c
example.c: In function ‘main’:
example.c:4:11: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat=]
    4 |   scanf("%d", i);
      |          ~^   ~
      |           |   |
      |           |   int
      |           int *
...
example.c:3:7: note: ‘i’ was declared here
    3 |   int i;
      |       ^
```

