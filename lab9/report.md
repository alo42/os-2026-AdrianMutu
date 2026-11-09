# Lab 9 — Watching System Calls with strace

## Part 1
ls prints with the `write` system call, to descriptor 1 (standard output):
the trace ends with `write(1, "total 0\n", 8) = 8`. Before that, a burst of
openat/read/close loads the shared libraries and reads the directory.

## Part 2
The first openat returns 3 because 0, 1 and 2 are already taken — every
process inherits stdin (0), stdout (1) and stderr (2) from the shell, and the
kernel always returns the smallest free descriptor number.

## Part 3
`strace -f ls -l 2>&1 >/dev/null | wc -l` counts ALL the system calls of a
simple ls -l (the trace goes to stderr and wc counts its lines): 269 on this
machine (~288 on the reference machine; the exact number depends on libc,
locales and the directory). A two-letter command costs hundreds of kernel
crossings, most of them before main() even starts.
