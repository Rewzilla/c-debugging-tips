# How to use the AddressSanitizer (ASAN)

The AddressSanitizer (ASAN) is a compile-time instrumentation module which allows a program to _immediately_ detect run time errors and print verbose diagnostic information about what went wrong. In other words, ASAN can be compiled into your program when you build it and will identify problems that occur when the program is run.  ASAN can detect all kinds of bugs, including buffer overflows, use after frees, double frees, and memory leaks.

This is useful in debugging subtle bugs because often times a bug may corrupt memory silently in the background with no immediate observable affect, and this issue won't be noticeable until later on in the program's execution when the side-effects of that corruption causes an observable crash. That makes it difficult to identify the true root cause, since we are only ever able to investigate its side-effects down the line.

ASAN solves this compiling in additional run time checks which cause the program to fail _immediately_ when the first problem occurs and print detailed information about what happened.

Debugging a program with ASAN is very easy.  Simply compile your program with the `clang` compiler, add ASAN (the `-fsanitize=address` flag), add debugging symbols (the `-g` flag) and then run it.  For instance...

```
clang -fsanitize=address -g yourcode.c
./a.out
```

Note: ASAN is similar to [Valgrind](valgrind.md) in it's capabilities, but different in how it achieves them.  ASAN needs to be compiled directly into the code itself, where-as [Valgrind](valgrind.md) can monitor the program as it runs without having been compiled in.

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

	for (i=0; i<10; i++)
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

Without a deep understanding of system internals, this error message isn't terribly helpful. Luckily ASAN can quickly identify exactly where the problem occurred and what went wrong! Let's compile the program with ASAN and run it again.

```
andrew@dev:~$ clang -fsanitize=address example.c -g
andrew@dev:~$ ./a.out 
=================================================================
==1072==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6040000000b8 at pc 0x564fcf842f16 bp 0x7fffef0d5f80 sp 0x7fffef0d5f78
WRITE of size 4 at 0x6040000000b8 thread T0
    #0 0x564fcf842f15 in main /home/andrew/example.c:11:11
    #1 0x7f48bf440249 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #2 0x7f48bf440304 in __libc_start_main csu/../csu/libc-start.c:360:3
    #3 0x564fcf785300 in _start (/home/andrew/a.out+0x20300) (BuildId: 8a6490ae64bc5257bdfb4777a6c61cbec438b8ad)

0x6040000000b8 is located 0 bytes to the right of 40-byte region [0x604000000090,0x6040000000b8)
allocated by thread T0 here:
    #0 0x564fcf80814e in __interceptor_malloc (/home/andrew/a.out+0xa314e) (BuildId: 8a6490ae64bc5257bdfb4777a6c61cbec438b8ad)
    #1 0x564fcf842eb8 in main /home/andrew/example.c:8:9
    #2 0x7f48bf440249 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-buffer-overflow /home/andrew/example.c:11:11 in main
Shadow bytes around the buggy address:
  0x0c087fff7fc0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c087fff7fd0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c087fff7fe0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c087fff7ff0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c087fff8000: fa fa fd fd fd fd fd fd fa fa 00 00 00 00 00 02
=>0x0c087fff8010: fa fa 00 00 00 00 00[fa]fa fa fa fa fa fa fa fa
  0x0c087fff8020: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c087fff8030: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c087fff8040: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c087fff8050: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c087fff8060: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==1072==ABORTING
```

There is quite a lot of information here, but we can mostly ignore the bottom half.  The most important piece for understanding our bug is right at the top:

```
==1072==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6040000000b8 at pc 0x564fcf842f16 bp 0x7fffef0d5f80 sp 0x7fffef0d5f78
WRITE of size 4 at 0x6040000000b8 thread T0
    #0 0x564fcf842f15 in main /home/andrew/example.c:11:11
    #1 0x7f48bf440249 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #2 0x7f48bf440304 in __libc_start_main csu/../csu/libc-start.c:360:3
    #3 0x564fcf785300 in _start (/home/andrew/a.out+0x20300) (BuildId: 8a6490ae64bc5257bdfb4777a6c61cbec438b8ad)

0x6040000000b8 is located 0 bytes to the right of 40-byte region [0x604000000090,0x6040000000b8)
allocated by thread T0 here:
    #0 0x564fcf80814e in __interceptor_malloc (/home/andrew/a.out+0xa314e) (BuildId: 8a6490ae64bc5257bdfb4777a6c61cbec438b8ad)
    #1 0x564fcf842eb8 in main /home/andrew/example.c:8:9
    #2 0x7f48bf440249 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
```

Let's break this down to understand what each piece means.  First, ASAN reports that this is a `heap-buffer-overflow` error.  That's helpful!  We now know that the bug was due to an array being overflowed, and that array was on the heap.

Next, ASAN tells us that this was a `WRITE` bug, meaning that we were trying to _store_ data outside the bounds of that heap buffer (as opposed to reading it).  The write occurred in the `main()` function within `/home/andrew/example.c:11:11` (i.e. our `example.c` file on line `11` at column `11`.  This is where the program misbehaved by writing outside the bounds of an array.

Further, ASAN reports that the array being overflowed was allocated by the `main()` function within `/home/andrew/example.c:8:9` (i.e. our `example.c` file on line `8` at column `9`).

Putting this all together we can begin to see what happened...

 1. In `example.c` on line `8` the `main()` function allocated a block of heap memory with `malloc()`.

    ```c
    7:
    8:    nums = malloc(10 * sizeof(int));
    9:
    ```

 2. In `example.c` on line `11` the `main()` function wrote beyond the end of that heap buffer.

    ```c
    10:    for (i=0; i<=10; i++)
    11:        nums[i] = 99;
    12:
    ```

Aha! The answer is suddenly obvious.  In order for this to occur, the for-loop must have run too many times.  Why did that happen?  Due to a simple off-by-one mistake, a common error among new C programmers.  The counter begins at 0 (`for (i=0; `), but continues up to and _including_ 10 (`; i<=10; i++)`) which means the loop actually runs 11 times. Whoops! That would have been hard to spot based on the first error message, but ASAN helped us hone in on the problem.

## Requirements

Note that ASAN may not be installed by default!  If you are using the `cs.dsunix.net` or `dev.hostbin.org` machines, you may need to request that they be installed; ask your teacher. However if you wish to use ASAN on your own machine you should be able to install it by running one of the following commands (depending on your operating system).

 - Debian/Ubuntu: `sudo apt install clang clang-tools`
 - Fedora/Redhat/CentOS: `sudo dnf install clang clang-tools-extra`
 - Arch: `TODO`

