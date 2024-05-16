# Beautifying your code

"Beautify"ing your code is the process of cleaning up the formatting to make it more readable. In many cases compiler errors and warnings are the result of simple syntax problems which are difficult to spot due to bad formatting, but become immediately obvious when the code is cleanly formatted.

This might seem like an arduous task, but luckily there are automated tools which can handle it for us! It will absolutely be worth your time and effort to cleanly format your code.

## Example

Consider the following program (poorly formatted), which is meant to print a grid of 10x10 numbers, with each cell representing the product (x) of the row and column.

```
#include <stdio.h>
int main() {int i, j;
  for (i=0; i<10; i++)
{
           for (j=0; j<10; j++) {   printf("%3d ", i*j);
		}}
         printf("\n"); {
	}
}
```

When run, this program instead prints the items all in one long list.  Due to the poor formatting it is not immediately evident why, or how to best fix it.

```
andrew@dev:~$ gcc example.c 
andrew@dev:~$ ./a.out 
  0   0   0   0   0   0   0   0   0   0   0   1   2   3   4   5   6   7   8   9   0   2   4   6   8  10  12  14  16  18   0   3   6   9  12  15  18  21  24  27   0   4   8  12  16  20  24  28  32  36   0   5  10  15  20  25  30  35  40  45   0   6  12  18  24  30  36  42  48  54   0   7  14  21  28  35  42  49  56  63   0   8  16  24  32  40  48  56  64  72   0   9  18  27  36  45  54  63  72  81
```

However by formatting the code we can quickly see the problem!  The `printf("\n");` is **outside** the last `for` loop, and there is an unneeded set of curly brackets at the end.

```
#include <stdio.h>
int main() {
  int i, j;
  for (i = 0; i < 10; i++) {
    for (j = 0; j < 10; j++) {
      printf("%3d ", i * j);
    }
                        // ...should be here
  }
  printf("\n");         //  <-- this... ^^^
  {}                    //  And this is unneeded
}
```

In this case by simply beautifying the code we were able to identify the problem.

The following tools are demonstrated below.

 - GNU Indent
 - Astyle
 - ClangFormat

## GNU Indent

To quickly view a formatted version of your code **without** altering the file, simply run:

`indent < yourcode.c`

By default this uses the _GNU_ formatting style, but you might also prefer the _K & R_ or _Linux_ formatting styles.

 - GNU style (default): `indent < yourcode.c`
 - K & R style: `indent -kr < yourcode.c`
 - Linux style: `indent -linux < yourcode.c`

If you **do** wish to alter the original file, updating it's contents in-place, you can omit the `<` operator and just run:

`indent yourcode.c`

Note that this will alter the file!  However a copy will be stored in a new file called `yourcode.c~`

## Astyle

To quickly view a formatted version of your code **without** altering the file, simply run:

`astyle < yourcode.c`

If you **do** wish to alter the original file, updating it's contents in-place, you can omit the `<` operator and just run:

`astyle yourcode.c`

Note that this will alter the file!  However a copy will be stored in a new file called `yourcode.c.orig`

## ClangFormat

To quickly view a formatted version of your code **without** altering the file, simply run:

`clang-format yourcode.c`

By default this uses the _LLVM_ formatting style, but you might also prefer the _Google_ or _Microsoft_ formatting styles.

 - LLVM style (default): `clang-format yourcode.c`
 - Google style: `clang-format --style=Google yourcode.c`
 - Microsoft style: `clang-format --style=Microsoft yourcode.c`

If you **do** wish to alter the original file, updating it's contents in-place, you can include the `-i` (edit in-place) operator.

`clang-format -i yourcode.c`

Note that this will alter the file, and will not store a backup of the original!

## Requirements

Note that these may not be installed by default!  If you are using the `cs.dsunix.net` or `dev.hostbin.org` machines, you may need to request that they be installed; ask your teacher. However if you wish to run these on your own machine you should be able to install them by running one of the following commands (depending on your operating system).

 - Debian/Ubuntu: `sudo apt install indent astyle clang-format`
 - Fedora/Redhat/CentOS: `sudo dns install indent astyle clang clang-tools-extra`
 - Arch: `sudo pacman -S indent astyle clang`
