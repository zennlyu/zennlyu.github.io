---
title: S081 Utilities
categories: [mit_opencourseware]
tags: [S081, Operating System]
---

## Boot xv6 [Easy]

The xv6-labs-2020 repository differs slightly from the book's xv6-riscv; it mostly adds some files. If you are curious look at the git log:

```shell
$ git log
```

The files you will need for this and subsequent lab assignments are distributed using the [Git](http://www.git-scm.com/) version control system. Above you switched to a branch (git checkout util) containing a version of xv6 tailored to this lab. To learn more about Git, take a look at the [Git user's manual](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html), or, you may find this [CS-oriented overview of Git](http://eagain.net/articles/git-for-computer-scientists/) useful. Git allows you to keep track of the changes you make to the code. For example, if you are finished with one of the exercises, and want to checkpoint your progress, you can *commit* your changes by running:

```shell
$ git commit -am 'my solution for util lab exercise 1'
	Created commit 60d2135: my solution for util lab exercise 1
	1 files changed, 1 insertions(+), 0 deletions(-)
```

You can keep track of your changes by using the git diff command. Running git diff will display the changes to your code since your last commit, and git diff origin/util will display the changes relative to the initial xv6-labs-2020 code. Here, `origin/xv6-labs-2020` is the name of the git branch with the initial code you downloaded for the class.

These are the files that `mkfs` includes in the initial file system; most are programs you can run. You just ran one of them: `ls`. If you type `ls` at the prompt, you should see output similar to the following:

xv6 has no `ps` command, but, if you type Ctrl-p, the kernel will print information about each process. If you try it now, you'll see two lines: one for `init`, and one for `sh`.

To quit qemu type: `Ctrl-a x`.

## Grading and hand-in procedure

You can run `make grade` to test your solutions with the grading program. The TAs will use the same grading program to assign your lab submission a grade. Separately, we will also have check-off meetings for labs (see [Grading policy](https://pdos.csail.mit.edu/6.S081/2020/general.html#grading)).

The lab code comes with GNU Make rules to make submission easier. After committing your final changes to the lab, type make handin to submit your lab. For detailed instructions on how to submit see [below](https://pdos.csail.mit.edu/6.S081/2020/labs/util.html#submit).



## sleep ([easy](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

> Implement the UNIX program `sleep` for xv6; your `sleep` should pause for a user-specified number of ticks. A tick is a notion of time defined by the xv6 kernel, namely the time between two interrupts from the timer chip. Your solution should be in the file `user/sleep.c`.

### Preparation

- read Chapter 1 of the [xv6 book](https://pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf).
- Look at some of the other programs in `user/` (e.g., `user/echo.c`, `user/grep.c`, and `user/rm.c`) to see how you can obtain the command-line arguments passed to a program.
- If the user forgets to pass an argument, sleep should print an error message.
- The command-line argument is passed as a string; you can convert it to an integer using `atoi` (see user/ulib.c).
- Use the system call `sleep`.
- See `kernel/sysproc.c` for the xv6 kernel code that implements the `sleep` system call (look for `sys_sleep`), `user/user.h` for the C definition of `sleep` callable from a user program, and `user/usys.S` for the assembler code that jumps from user code into the kernel for `sleep`.
- Make sure `main` calls `exit()` in order to exit your program.
- Add your `sleep` program to `UPROGS` in Makefile; once you've done that, `make qemu` will compile your program and you'll be able to run it from the xv6 shell.
- Look at Kernighan and Ritchie's book *The C programming language (second edition)* (K&R) to learn about C.

### Presentation

Run the program from the xv6 shell:

```sh
$ sleep 10
(nothing happens for a little while)
```

Your solution is correct if your program pauses when run as shown above. Run make grade to see if you indeed pass the sleep tests.

Note that make grade runs all tests, including the ones for the assignments below. If you want to run the grade tests for one assignment, type:

```shell
$ ./grade-lab-util sleep
```

This will run the grade tests that match "sleep". Or, you can type:

```shell
$ make GRADEFLAGS=sleep grade
```



## pingpong ([easy](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

> Write a program that uses UNIX system calls to ''ping-pong'' a byte between two processes over a pair of pipes, one for each direction. The parent should send a byte to the child; the child should print "<pid>: received ping", where <pid> is its process ID, write the byte on the pipe to the parent, and exit; the parent should read the byte from the child, print "<pid>: received pong", and exit. Your solution should be in the file `user/pingpong.c`.

### Preparation

- Use `pipe` to create a pipe.
- Use `fork` to create a child.
- Use `read` to read from the pipe, and `write` to write to the pipe.
- Use `getpid` to find the process ID of the calling process.
- Add the program to `UPROGS` in Makefile.
- User programs on xv6 have a limited set of library functions available to them. You can see the list in `user/user.h`; the source (other than for system calls) is in `user/ulib.c`, `user/printf.c`, and `user/umalloc.c`.

### Presentation

Run the program from the xv6 shell and it should produce the following output:

```shell
$ pingpong
4: received ping
3: received pong
```

Your solution is correct if your program exchanges a byte between two processes and produces output as shown above.



## primes ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))/([hard](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

> Write a concurrent version of prime sieve using pipes. This idea is due to Doug McIlroy, inventor of Unix pipes. The picture halfway down [this page](http://swtch.com/~rsc/thread/) and the surrounding text explain how to do it. Your solution should be in the file `user/primes.c`.
>
> Your goal is to use `pipe` and `fork` to set up the pipeline. The first process feeds the numbers 2 through 35 into the pipeline. For each prime number, you will arrange to create one process that reads from its left neighbor over a pipe and writes to its right neighbor over another pipe. Since xv6 has limited number of file descriptors and processes, the first process can stop at 35.

### Preparation

- Be careful to close file descriptors that a process doesn't need, because otherwise your program will run xv6 out of resources before the first process reaches 35.
- Once the first process reaches 35, it should wait until the entire pipeline terminates, including all children, grandchildren, &c. Thus the main primes process should only exit after all the output has been printed, and after all the other primes processes have exited.
- Hint: `read` returns zero when the write-side of a pipe is closed.
- It's simplest to directly write 32-bit (4-byte) `int`s to the pipes, rather than using formatted ASCII I/O.
- You should create the processes in the pipeline only as they are needed.
- Add the program to `UPROGS` in Makefile.

### Presentation

Your solution is correct if it implements a pipe-based sieve and produces the following output:

```shell
$ primes
prime 2
prime 3
prime 5
prime 7
prime 11
prime 13
prime 17
prime 19
prime 23
prime 29
prime 31
```



## find ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

> Write a simple version of the UNIX find program: find all the files in a directory tree with a specific name. Your solution should be in the file `user/find.c`.

### Preparation

- Look at user/ls.c to see how to read directories.
- Use recursion to allow find to descend into sub-directories.
- Don't recurse into "." and "..".
- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.
- You'll need to use C strings. Have a look at K&R (the C book), for example Section 5.5.
- Note that == does not compare strings like in Python. Use strcmp() instead.
- Add the program to `UPROGS` in Makefile.

### Presentation

Your solution is correct if produces the following output (when the file system contains the files `b` and `a/b`):

```shell
$ echo > b
$ mkdir a
$ echo > a/b
$ find . b
./b
./a/b
$ 
```



## xargs ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

> Write a simple version of the UNIX xargs program: read lines from the standard input and run a command for each line, supplying the line as arguments to the command. Your solution should be in the file `user/xargs.c`.

### Preparation

- Use `fork` and `exec` to invoke the command on each line of input. Use `wait` in the parent to wait for the child to complete the command.
- To read individual lines of input, read a character at a time until a newline ('\n') appears.
- kernel/param.h declares MAXARG, which may be useful if you need to declare an argv array.
- Add the program to `UPROGS` in Makefile.
- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.

xargs, find, and grep combine well:

```shell
$ find . b | xargs grep hello
```

will run "grep hello" on each file named b in the directories below ".".

To test your solution for xargs, run the shell script xargstest.sh. Your solution is correct if it produces the following output:

```shell
$ sh < xargstest.sh
$ $ $ $ $ $ hello
hello
hello
$ $
```

You may have to go back and fix bugs in your find program. The output has many `$` because the xv6 shell doesn't realize it is processing commands from a file instead of from the console, and prints a `$` for each command in the file.

### Presentation

The following example illustrates xarg's behavior:

```shell
$ echo hello too | xargs echo bye
bye hello too
```

Note that the command here is "echo bye" and the additional arguments are "hello too", making the command "echo bye hello too", which outputs "bye hello too".

Please note that xargs on UNIX makes an optimization where it will feed more than argument to the command at a time. We don't expect you to make this optimization. To make xargs on UNIX behave the way we want it to for this lab, please run it with the -n option set to 1. For instance

```shell
$ echo "1\n2" | xargs -n 1 echo line
line 1
line 2
```



## Optional challenge exercises

- Write an uptime program that prints the uptime in terms of ticks using the `uptime` system call. ([easy](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))
- Support regular expressions in name matching for `find`. `grep.c` has some primitive support for regular expressions. ([easy](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))
- The xv6 shell (`user/sh.c`) is just another user program and you can improve it. It is a minimal shell and lacks many features found in real shell. For example, modify the shell to not print a `$` when processing shell commands from a file ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html)), modify the shell to support wait ([easy](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html)), modify the shell to support lists of commands, separated by ";" ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html)), modify the shell to support sub-shells by implementing "(" and ")" ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html)), modify the shell to support tab completion ([easy](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html)), modify the shell to keep a history of passed shell commands ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html)), or anything else you would like your shell to do. (If you are very ambitious, you may have to modify the kernel to support the kernel features you need; xv6 doesn't support much.)
