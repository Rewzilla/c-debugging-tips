# How to use GDB

The GNU Debugger (GDB) is a tool for introspecting a program at run time. Essentially, this means that GDB is able to observe a program while it runs, identify where and how it misbehaves, and inform you about what went wrong.

For more advanced users GDB can also be used to manually stop/start/pause/restart a program as it runs and inspect specific pieces along the way.

This makes GDB a perfect tool for identifying programming problems!

## Running a program with GDB

In order to take full advantage of the power of GDB you will need to compile your program with debugging symbols. GCC supports this with the `-g` parameter.  For instance, compile your program like:

`gcc -g yourcode.c`

You should then be able to load your program in GDB by running:

`gdb ./a.out`

This will drop you into a GDB shell with the prompt `(gdb)`.  You can now execute your program by typing:

`run`

The program will now run until completion, or until it crashes.  If you see a message stating "_(process ...) exited normally_", the program ran to completion and did not crash.  If instead the program crashes, you may see a message such as "_Program received signal SIGSEGV_".  In this case GDB identified that the program crashes and has paused to let you inspect its state.

## Identifying why a program crashed

Assuming the program crashed, GDB should display the exact line from your source file where the crash occurred.  For instance, consider the following example program which contains a bug (incomplete function).

```c
#include <stdio.h>
#include <stdlib.h>

struct person {
	char *name;
	int age;
};

struct person * new_person(char *n, int a) {
	return NULL; // Not yet implemented
}

void print_person(struct person *p) {
	printf("name: %s\n", p->name);
	printf("age: %d\n", p->age);
}

int main() {
	struct person *p;
	p = new_person("Bob", 100);
	print_person(p);
}
```

When run this program will crash with a segmentation fault.

```
andrew@dev:~$ ./a.out 
Segmentation fault
```

GDB will give us more information though!

```
andrew@dev:~$ gcc -g example.c
andrew@dev:~$ gdb ./a.out 
  ... GDB output removed for brevity ...
(gdb) run
Starting program: /home/andrew/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x000055555555515b in print_person (p=0x0) at example.c:14
14		printf("name: %s\n", p->name);
(gdb)
```

This let's us know that the problem occurred in `example.c` on line `14`, where the `printf()` function was used to print the person's name. Further, we can see that the `p` variable was set to `0x0` (i.e. NULL) when passed as an argument to the `print_person()` function, which likely explains the problem, a null pointer dereference.

To check the value of a specific variable such as `p`, you may use the `print` command, like:

```
(gdb) print p
$1 = (struct person *) 0x0
```

If you would like to see a back trace (i.e. list of functions calls showing how we got to this point in the program), you can simply type `backtrace`.  Here we see that `main()` called `print_person()` from `example.c` at line `21`, and `print_person()` reached line `14` before crashing.  If many more functions had been called along the way to reach this point, they would be shown here.

```
(gdb) backtrace 
#0  0x000055555555515b in print_person (p=0x0) at example.c:14
#1  0x00005555555551c1 in main () at example.c:21
```

## Command line arguments and file input

If your program takes any command line arguments, (for instance maybe a file called `input.txt`), you can add those with the `--args` parameter at load time, like:

`gdb --args ./a.out input.txt`

Alternatively, if your program needs to read input from a file via stdin (with the `<` operator), you can do this after loading the program when you run it, like:

```
gdb ./a.out
(gdb) run < input.txt
```

## Breakpoints

For more advanced users, GDB can also be used to pause and inspect a program at any point.  In order to to this you will need to set _breakpoints_ where you'd like the program to pause.  These can either be set on specific functions or on specific lines in the program.

To set a breakpoint on a specific function (such as the `print_person()` function above), run the following command **before** using `run` to start the program.

`break print_person`

To set a breakpoint on a specific line in your code (such as `example.c` line `14` from the code above), run the following command **before** using `run to start the program.

`break example.c:14`

You may also get a list of breakpoints by running:

`info breakpoints`

You may also delete breakpoints by running:

`delete N` (where `N` is the breakpoint number, like `1` or `2`)

Any time a breakpoint is hit you will be dropped back into the `(gdb)` prompt to inspect the program at that point in its execution.  If you would like to single-step to the next line of C code you can simply run:

`next`

If instead you would like to allow the program to continue on until it hits the next breakpoint (or ends, or crashes), you can run:

`continue`

## Making GDB more user friendly

TODO (GEF, PEDA, etc) ?

## Requirements

Note that GDB may not be installed by default!  If you are using the `cs.dsunix.net` or `dev.hostbin.org` machines, you may need to request that they be installed; ask your teacher. However if you wish to run GDB on your own machine you should be able to install it by running one of the following commands (depending on your operating system).

 - Debian/Ubuntu: `sudo apt install gdb`
 - Fedora/Redhat/CentOS: `sudo dnf install gdb`
 - Arch: `sudo pacman -S gdb`

