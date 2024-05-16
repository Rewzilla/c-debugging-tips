# How to use CPPCheck

CPPCheck is a static code analysis tool for detecting bugs and errors in C/C++ code.  This means that CPPCheck can scan your code files and find potential problems before you compile or run in.

CPPCheck is designed to be very conservative in what it reports, meaning that it won't necessarily find and report every bug, however the bugs it does report are very unlikely to be false positives.  This makes it a good tool for programmers to run periodically, to identify any obvious problems as the code is being developed.

Running CPPCheck is very easy, simply run `cppcheck` and pass your C file(s) as argument(s).  For instance you might run:

```
cppcheck yourcode.c
```

You can also run CPPCheck over an entire project directory.  In this case it will search for C/C++ files in that directory and scan everything it finds.  For instance:

```
cppcheck ./myproject/
```

Or even just:

```
cppcheck .
```

## Example

Consider the following program which has an array of 3 integers (10, 20, 30) and uses a for loop to print them.  At first glance this program looks fine, and in fact it will _probably_ compile without errors and run without crashing, however it seems to print some kind of _junk_ data at the end as well.

```c
#include <stdio.h>
#include <stdlib.h>

int main() {

	int i, nums[3] = {10, 20, 30};

	for (i=0; i<=3; i++)
		printf("%d: %d\n", i, nums[i]);

}
```

```
andrew@dev:~$ gcc example.c
andrew@dev:~$ ./a.out 
0: 10
1: 20
2: 30
3: 3       <-- what???
```

Clearly we have a bug.  Do you see the problem?  The for loop begins at zero (`for (i=0;`) but runs until the counter is **equal** to 3 (`i<=3; i++)`).  This means the loop will actually run **four** times, not three, (0, 1, 2, 3) resulting in an out-of-bounds read off the end of the array.  A classic off-by-one error, common among new programmers.

CPPCheck can help us identify this problem though!  Let's run CCPCheck on our code and see what it reports.

```
andrew@dev:~$ cppcheck example.c 
Checking example.c ...
example.c:9:29: error: Array 'nums[3]' accessed at index 3, which is out of bounds. [arrayIndexOutOfBounds]
  printf("%d: %d\n", i, nums[i]);
                            ^
example.c:8:13: note: Assuming that condition 'i<=3' is not redundant
 for (i=0; i<=3; i++)
            ^
example.c:9:29: note: Array index out of bounds
  printf("%d: %d\n", i, nums[i]);
```

Nice!  CPPCheck has identified that there is an array out-of-bounds access in `example.c` on line `9` at column `29` where `nums[i]` was accessed.  Additionally it noted that the problem was likely due to the term `i<=3` in our loop in `example.c` on line `8` at column `13`.  CPPCheck has found the problem.

## Requirements

Note that CPPCheck may not be installed by default!  If you are using the `cs.dsunix.net` or `dev.hostbin.org` machines, you may need to request that they be installed; ask your teacher. However if you wish to run CPPCheck on your own machine you should be able to install it by running one of the following commands (depending on your operating system).

 - Debian/Ubuntu: `sudo apt install cppcheck`
 - Fedora/Redhat/CentOS: `sudo dnf install cppcheck`
 - Arch: `sudo pacman -S cppcheck`
