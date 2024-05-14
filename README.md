# C Debugging Tips

## Overview

The following guide provides tips and tricks for debugging C programs.

## Intended audience

This guide is intended for students who are new to C programming and need help determining the cause of segmentation faults or other runtime problems.

The guide is written and maintained by DSU faculty, primarily for DSU students in CSC-150, CSC-250 or CSC-300, but may be useful to other students in other classes or universities as well.

## How to use this guide

The tools below are all explained and demonstrated in this guide.  If you'd just like to dive in and learn more about debugging C program problems, feel free to go through these in any order.  The time and effort you invest in learning this tools now will save you countless hours and frustration in the future!

 - [Beautify your code](howto/beautify.md).
 - [Compile with verbose warning/error messages](howto/compile-flags.md).
 - [Run with GDB to isolate crashes](howto/gdb.md).
 - [Run with Valgrind to detect heap memory corruption](howto/valgrind.md).
 - [Compile with ASAN to detect memory corruption](howto/asan.md).
 - [Compile with scan-build to detect subtle bugs](howto/scan-build.md).
 - [Run CCPCheck to detect subtle bugs](howto.md).

Alternatively, if you have a specific problem or question, the next section lists common problems that students encounter and links to specific resources that might be helpful to resolve it.  If you need immediate help on a specific issue, check the list below for the relevant guide(s).

## Specific problems and questions

 - **Q:** Help! My program compiles fine, but does something unexpected or wrong when run.

   **A:** Sounds like a problem with your code structure or algorithm. If you haven't already, consider [beautifying](howto/beautify.md) your code, which will make it easier to reason about.

 - **Q:** Help! My program says "Segmentation fault" and crashes.

   **A:** Likely a memory corruption bug.  Try [GDB](howto/gdb.md), [Valgrind](howto/valgrind.md), or [ASAN](howto/asan.md).

 - **Q:** Help! My program prints _junk_ data.

   **A:** Sounds like you may be using an uninitialized variable, or printing from invalid memory.  Try [scan-build](howto/scan-build.md) to identify potential causes.  [Valgrind](howto/valgrind.md) or [ASAN](howto/asan.md) might also be able to detect where/why this is happening at runtime.

 - **Q:** Help!  My program locks up or runs forever.

   **A:** This may be an infinite loop.  Try [scan-build](howto/scan-build.md) to identify potential causes.

## Questions or suggestions

This resource is primarily maintained by Andrew Kramer (andrew.kramer at dsu dot edu), with help from other DSU faculty.  Feel free to send any questions, comments, or suggestions my way.

Pull requests are also welcome!  This is a living document.  If you see a way to improve this guide, please send us a PR!

Thanks and happy debugging!
