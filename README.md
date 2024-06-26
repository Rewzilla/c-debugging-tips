# C Debugging Tips

## Overview

The following guide provides tips and tricks for debugging C programs.

## Intended audience

This guide is intended for students who are new to C programming and need help determining the cause of segmentation faults or other run time problems.

The guide is written and maintained by DSU faculty, primarily for DSU students in CSC-150, CSC-250 or CSC-300, but may be useful to other students in other classes or universities as well.

## How to use this guide

The tools below are all explained and demonstrated in this guide.  If you'd just like to dive in and learn more about debugging C program problems, feel free to go through these in any order.  The time and effort you invest in learning this tools now will save you countless hours and frustration in the future!

 - [Beautify your code](guides/beautify.md).
 - [Compile with verbose warning/error messages](guides/compile-flags.md).
 - [Run with GDB to isolate crashes](guides/gdb.md).
 - [Run with Valgrind to detect heap memory corruption](guides/valgrind.md).
 - [Compile with ASAN to detect memory corruption](guides/asan.md).
 - [Compile with scan-build to detect subtle bugs](guides/scan-build.md).
 - [Run CCPCheck to detect subtle bugs](guides/cppcheck.md).

Alternatively, if you have a specific problem or question, the next section lists common problems that students encounter and links to specific resources that might be helpful to resolve it.  If you need immediate help on a specific issue, check the list below for the relevant guide(s).

## Specific problems and questions

 - **Q:** Help! My program compiles fine, but doesn't behave the way I think it should.

   **A:** This might be a problem with your code structure or algorithm. If you haven't already, consider [beautifying](guides/beautify.md) your code, which will make it easier to reason about.

 - **Q:** Help! My program says "Segmentation fault", "Aborted", or some other cryptic error and crashes.

   **A:** Likely a memory corruption bug.  Try [GDB](guides/gdb.md), [Valgrind](guides/valgrind.md), or [ASAN](guides/asan.md).

 - **Q:** Help! My program prints _junk_ data

   **A:** Sounds like you may be using an uninitialized variable, or printing from invalid memory.  Try [scan-build](guides/scan-build.md) to identify potential causes.  [Valgrind](guides/valgrind.md) or [ASAN](guides/asan.md) might also be able to detect where/why this is happening at run time.

 - **Q:** Help!  My program locks up or runs forever.

   **A:** This may be an infinite loop.  Try running the program in [GDB](guides/gdb.md) up to the point it gets stuck, then press `CTRL + C` to pause it, allowing you to see what part of the code it's stuck in.

## Questions or suggestions

This resource is primarily maintained by Andrew Kramer (andrew.kramer at dsu dot edu), with help from other DSU faculty.  Feel free to send any questions, comments, or suggestions my way.

Pull requests are also welcome!  This is a living document.  If you see a way to improve this guide, please send us a PR!

### Ideas to improve this document (TODO's)

The following is a list of ideas which may improve this document but haven't yet been fully explored.  If you have time and knowledge and want to help contribute to this guide, these would be good places to start!  Feel free to add to this list as well.

 - Are there other tools that really should be included here?

 - Would it be useful to include the MemorySanitizer (MSAN) and UndefinedBehaviorSanitizer (UBSAN) in addition to the AddressSanitizer (ASAN)?

 - Is there a better way to root-cause infinite loops than just breaking in GDB?

 - Is there a more effective way to organize and present these tools so they don't feel overwhelming to students?

 - Add "further reading" sections to each page, with links to useful resources or tutorials.
