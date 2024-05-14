# How to use Valgrind

Valgrind is a tool which can monitor a program as it runs and _immediately_ report problems as they occur. This is extremely powerful because memory corruption often happens silently in the background and won't become evident until later on when the side-effects cause observable problems. This makes it difficult to track down the root cause of the bug, since we are only investigating the eventual side-effects of the bug, not the bug itself.  Valgrind solves this by observing everything the program does and _immediately_ throwing an error when memory corruption initially occurs.

Examining a program with Valgrind is very easy.  Simply compile your program with debugging symbols (the `-g` flag) and then run it with `valgrind ...`.  For instance...

```
gcc -g yourcode.c
valgrind ./a.out
```

Note: Valgrind is similar to [ASAN](asan.md) in what it can do, but different it how it accomplishes that.  [ASAN](asan.md) needs to be compiled directly into the program itself at build time, where-as Valgrind is able to observe the program running without having been compiled into it.

## Example

Consider the following program which is meant to generate a set of 10 integers, set them all to `99`, and then print them.  These integers are stored on the heap in a block of memory allocated with `malloc()`.

```c
#include <stdio.h>
#include <stdlib.h>

int main() {

	int i, *nums;

	nums = malloc(10 * sizeof(int));

	for (i=0; i<=10; i++)
		nums[i] = 99;

	for (i=0; i<=10; i++)
		printf("%d: %d\n", i, nums[i]);

}
```

At first glance this program looks reasonable, however when run it will fail with a cryptic error message.

```
andrew@dev:~$ gcc example.c
andrew@dev:~$ ./a.out 
Fatal glibc error: malloc assertion failure in sysmalloc: (old_top == initial_top (av) && old_size == 0) || ((unsigned long) (old_size) >= MINSIZE && prev_inuse (old_top) && ((unsigned long) old_end & (pagesize - 1)) == 0)
Aborted
```

Without a deep understanding of system internals, this error message isn't terribly helpful. Luckily Valgrind can quickly identify exactly where the problem occured and what went wrong! Let's compile the program with debug symbols and run it with Valgrind.

```
andrew@dev:~$ gcc -g example.c
andrew@dev:~$ valgrind ./a.out 
==42905== Memcheck, a memory error detector
==42905== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==42905== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==42905== Command: ./a.out
==42905== 
==42905== Invalid write of size 4
==42905==    at 0x10917C: main (example.c:11)
==42905==  Address 0x4a44068 is 0 bytes after a block of size 40 alloc'd
==42905==    at 0x48407B4: malloc (vg_replace_malloc.c:381)
==42905==    by 0x10915A: main (example.c:8)
==420 bytes after a block of size 40 alloc'd905== 
0: 99
1: 99
2: 99
3: 99
4: 99
5: 99
6: 99
7: 99
8: 99
9: 99
==42905== Invalid read of size 4
==42905==    at 0x1091A9: main (example.c:14)
==42905==  Address 0x4a44068 is 0 bytes after a block of size 40 alloc'd
==42905==    at 0x48407B4: malloc (vg_replace_malloc.c:381)
==42905==    by 0x10915A: main (example.c:8)
==42905== 
10: 99
==42905== 
==42905== HEAP SUMMARY:
==42905==     in use at exit: 40 bytes in 1 blocks
==42905==   total heap usage: 2 allocs, 1 frees, 1,064 bytes allocated
==42905== 
==42905== LEAK SUMMARY:
==42905==    definitely lost: 40 bytes in 1 blocks
==42905==    indirectly lost: 0 bytes in 0 blocks
==42905==      possibly lost: 0 bytes in 0 blocks
==42905==    still reachable: 0 bytes in 0 blocks
==42905==         suppressed: 0 bytes in 0 blocks
==42905== Rerun with --leak-check=full to see details of leaked memory
==42905== 
==42905== For lists of detected and suppressed errors, rerun with: -s
==42905== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

This looks complicated, but let's break it down and see that it's really quite simple.  First, in the summary at the end notice that there were actually _two_ errors detected at run time (`ERROR SUMMARY: 2 errors from 2 contexts ...`).  We will need to understand and address both.

Starting from the top, the first relevent block of information is:

```
==42905== Invalid write of size 4
==42905==    at 0x10917C: main (example.c:11)
==42905==  Address 0x4a44068 is 0 bytes after a block of size 40 alloc'd
==42905==    at 0x48407B4: malloc (vg_replace_malloc.c:381)
==42905==    by 0x10915A: main (example.c:8)
```

Valgrind has detected an "invalid write", meaning that we wrote data to a memory location that we shouldn't have. This is most commonly caused by writing data beyond the end of an array, or occasionally at a negative index before it.

More specifically, Valgrind reported that the invalid write occured at `example.c:14` and that the data was written `0 bytes after a block of size 40 alloc'd`. This means the bug is in `example.c` on line `14`, and we wrote data just to the beyond a block of data that was allocated for us to use. Further, Valgrind reports that the allocated block which we wrote beyond was previously allocated by `malloc()`, which was called by `main()` in `example.c` on line `8`.

Putting this all together...

 1. `main()` called `malloc()` in `example.c` on line `8`, thus allocating a block of memory

    ```c
    7: 
    8:     nums = malloc(10 * sizeof(int));
    9:
    ```

 2. The program accidentally wrote data _beyond_ that block in `example.c` at line `11`.

    ```c
    10:     for (i=0; i<=10; i++)
    11:         nums[i] = 99;
    12:
	```

Now the issue is obvious! The loop must have gone too far, copying too many integers into the array and overflowing beyond its end. Do you see why? Looks like a simple off-by-one bug. The counter begins at 0 (`for (i=0; `), but continues up to and _including_ 10 (`; i<=10; i++)`) which means the loop actually runs 11 times. Whoops! That would have been hard to spot based on the first error message, but Valgrind helped us hone in on the problem.

Now let's see if we can solve the second bug. The second relevent block of information that Valgrind gave us was:

```
==42905== Invalid read of size 4
==42905==    at 0x1091A9: main (example.c:14)
==42905==  Address 0x4a44068 is 0 bytes after a block of size 40 alloc'd
==42905==    at 0x48407B4: malloc (vg_replace_malloc.c:381)
==42905==    by 0x10915A: main (example.c:8)
```

Breaking this down as we did above, we can see that...

 1. Again, `main()` called `malloc()` in `example.c` on line `8`, thus allocating a block of memory

    ```c
    7: 
    8:     nums = malloc(10 * sizeof(int));
    9:
    ```

 2. This time the program accidentally read data _beyond_ that block in `example.c` at line `14`.

    ```c
    13     for (i=0; i<=10; i++)
    14         printf("%d: %d\n", i, nums[i]);
    15
	```

Aha, looks like a second occurence of the same bug! Another simple off-by-one mistake. Removing the two equals signs from these for-loops resolves both issues.

## Requirements

Note that Valgrind may not be installed by default!  If you are using the `cs.dsunix.net` or `dev.hostbin.org` machines, you may need to request that they be installed; ask your teacher. However if you wish to run Valgrind on your own machine you should be able to install it by running one of the following commands (depending on your operating system).

 - Debian/Ubuntu: `sudo apt install valgrind`
 - Fedora/Redhat/CentOS: `TODO`
 - Arch: `TODO`

