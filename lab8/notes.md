# Lab 8 — The Same Calls, Inside Your Own OS (xv6)

## Part 1 — run, inside xv6
1. xv6: `fork()`, `exec()`, `wait()`. Linux (Lab 7): `fork()`, `execlp()`,
   `waitpid()`. The kernel operations are the same; only the C-library wrappers
   differ (`execlp` searches PATH and builds argv, xv6's `exec` takes the path
   and argv array directly).
2. `echo` prints it. After a successful exec the code of `run` no longer exists
   in the child; the words are just echo's arguments.
3. The status is whatever the program passes to exit after the failed exec:
   xv6's run.c calls `exit(1)`, the Linux version calls `exit(127)`. The kernel
   attaches no meaning to the number.

## Part 2 — pingpong
1. A pipe is one-directional. With a single pipe for both directions a process
   could read back its own byte; two pipes give each direction its own channel.
2. The blocking `read(c2p[0], &b, 1)`: read on an empty pipe blocks until the
   child has written its reply, so "received pong" cannot appear early. The
   final `wait(0)` only collects the child afterwards.

Optional: `primes.c` — a chain of processes, one per prime; each stage prints
its first number and forwards the non-multiples. Prints prime 2 … prime 31.
