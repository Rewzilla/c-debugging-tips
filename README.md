# C Debugging Tips

## Overview

The following guide provides tips and tricks for debugging C programs.

## Intended audience

This guide is intended for students who are new to C programming and need help determining the cause of segmentation faults or other runtime problems.

The guide is written and maintained by DSU faculty, primarily for DSU students in CSC-150, CSC-250 or CSC-300, but may be useful to other students in other classes or universities as well.

## How to use this guide

The most common problems that students face fall into two categories:

 - **Compile time issues**: i.e. problems with the code structure which prevent it from building in the first place. These may cause the compiler to throw errors or warnings and result in no runable program being generated.

 - **Run time issues:** i.e. problems which only appear after the program has been compiled and run. These may result in segmentation faults, memory corruption warnings, non-determenistic behavior, or even _junk_ data being printed.

### Resolving compile time issues

Any/all of the following may be helpful in resolving compile time issues.

 - [Beautify the code](howto/beautify.md).
 - [Compile with verbose warning/error messages](howto/compile-flags.md).

### Resolving run time issues

Any/all of the following may be helpful in resolving run time issues.

 - [Run with GDB to isolate crashes](howto/gdb.md).
 - [Run with Valgrind to detect heap memory corruption](howto/valgrind.md).
 - [Compile with ASAN to detect memory corruption](howto/asan.md).
 - [Compile with scan-build to detect subtle bugs](howto/scan-build.md).
 - [Run CCPCheck to detect subtle bugs](howto.md).

## Specific problems and questions

 - Q: Help! My program says "Segmentation fault" and crashes.

   A: Likely a memory corruption bug.  Try [GDB](howto/gdb.md), [Valgrind](howto/valgrind.md), or [ASAN](howto/asan.md).

 - Q: Help! My program prints _junk_ data.

   A: Sounds like you may be using an uninitialized variable, or printing from invalid memory.  Try [scan-build](howto/scan-build.md) to identify potential causes.

 - Q: Help!  My program locks up or runs forever.

   A: This may be an infinite loop.  Try [scan-build](howto/scan-build.md) to identify potential causes.

## Questions or suggestions

This resource is primarily maintained by Andrew Kramer (andrew.kramer at dsu dot edu), with help from other DSU faculty.  Feel free to send any questions, comments, or suggestions my way.

Pull requests are also welcome!  This is a living document.  If you see a way to improve this guide, please send us a PR!

Thanks and happy debugging!
