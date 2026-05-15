- Debugging for optimisation might be still in progress gcc and clang
- -g , --gdb3 for debugging
- load the program with the output , and then use *start* cmd in gdb
- *ignore* ( ignore invocations in a loop for a while maybe) and *until* for navigating faster
- *info locals* and *bt full* and *bt 'nummber'*
- Custom cmd to execute when we encounter a breakpoint
- God 's feature , dynamic printf , add printf without recompile of output , *dprintf loc , format-string expr1 , expr2 , ...... *
- *watch 'var'*
- *info threads * , thread apply all bt
- *save breakpoints* , save breakpoints to a file
- God 's feature , recording *record* , is moving through the program in reverse order .. *rc,rf,rn,rni,rs* .... , this is slow as we need to record everything to a buffer.
- .gdbinit , config for the debugger
- Repeat program through GDB until failure
	Occasionally the bug doesn’t reproduce every time, instead you will need to run the program several times to reproduce it. To repeat program until failure (with optional recording), you will set up a breakpoint at exit function (called when system exit) and instruct it to restart the program.

```
(gdb) start
...
Temporary breakpoint 1, main (argc=1, argv=0x7fffffffdec8) at linked_list_test.cpp:80
80	int main(int argc, char* argv[]) {
(gdb) set pagination off
(gdb) break exit
Breakpoint 2 at 0x7ffff7267120
(gdb) command 2
Type commands for breakpoint(s) 2, one per line.
End with a line saying just "end".
>run
>end
(gdb) c
Continuing.
...
```

- Debugging memory corruption with record and watchpoints

 Watchpoints are extremely useful for debugging memory corruption when paired with recording. The idea is to run your program through GDB with recording on and when a memory corruption happens your program will stop. It should be easy to figure out which memory address got corrupted. Then you need to set up a watchpoint on that address and then reverse-continue it. When the watchpoint hits, you will see the line that corrupts your memory.

```
(gdb) set pagination off
(gdb) start
Temporary breakpoint 1 at 0x4005a0: file /usr/include/x86_64-linux-gnu/bits/stdio2.h, line 104.
Starting program: /home/ivica/Projects/johnysswlab/a.out 

Temporary breakpoint 1, main () at test.c:9
9	    printf("\n i = %d \n", i);
(gdb) record
(gdb) c
...
(gdb) watch *(long**) 0x7fffffffddd8
Hardware watchpoint 2: *(long**) 0x7fffffffddd8
(gdb) reverse-continue
...
```

- Above is a shortened description of what needs to be done. Unfortunately, getting the full address that needs to be watched requires some skill and a full description of this technique would require a whole article. Luckily there is a live demonstration of this technique available
- 
## STRACE

- System calls and signals , signal maskable which means we can overwrite the default action of signals incase the need arises , and their are also non-maskable signals which are not updatable 
- Here is a short list of a few most important signals taken from [edspresso](https://www.educative.io/edpresso/what-are-linux-signals)
- Every time a program receives a signal, a default action will be performed automatically unless the program has explicitly overwritten it. Default action depends on the signal type and can be: **Core** (create the core dump and terminate the receiving process), **Term** (terminate the receiving process), **Ign** (ignore the received signal), **Stop** (pause the program) and **Cont** (resume the stopped program).

|         |                                                                          |                |
| ------- | ------------------------------------------------------------------------ | -------------- |
| Signal  | Description                                                              | Default Action |
| SIGINT  | Interrupt from keyboard (Ctrl + C)                                       | Term           |
| SIGKILL | Force the program to terminate                                           | Term           |
| SIGTERM | Politely ask the program to terminate                                    | Term           |
| SIGBUS  | Access to memory that doesn’t exist                                      | Core           |
| SIGSEGV | Access to forbidden part of the memory                                   | Core           |
| SIGSTOP | Pauses the execution of the program                                      | Stop           |
| SIGCONT | Continues the execution of the paused program                            | Cont           |
| SIGILL  | Program executed illegal or non-existing instruction                     | Core           |
| SIGFPE  | Arithmetic instruction caused the exception (e.g. division by zero etc.) | Core           |
| SIGTRAP | Program executed the trap instruction                                    | Core           |

- Primary fegature of strace is to trace the system calls in linux to monitor the resources and info for debugging ...
- Lookout for mmap , brk , munmap in memory & heap allocations
- strace ls , strace -e trace=file ... , strace -e -f ....

- Transfer-Encoding: chunked\r
