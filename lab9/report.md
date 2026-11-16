# Lab 9 — Watching System Calls with strace

## Part 1
ls prints with the `write` system call, to descriptor 1 (standard output):
the trace ends with `write(1, "total 0\n", 8) = 8`. Before that, a burst of
openat/read/close loads the shared libraries and reads the directory.

## Part 2
The first openat returns 3 because 0, 1 and 2 are already taken — every
process inherits stdin (0), stdout (1) and stderr (2) from the shell, and the
kernel always returns the smallest free descriptor number.

## Part 3 — Counting ONE kind of call
The filter -e trace= isolates a single kind of call; the listing goes to
/dev/null and wc -l counts the trace lines (minus the final
"+++ exited with 0 +++" marker, which is not a call):

    strace -f -e trace=openat ls -l 2>&1 >/dev/null | wc -l   -> 32 lines = 31 openat
    strace -f -e trace=read   ls -l 2>&1 >/dev/null | wc -l   -> 20 lines = 19 read
    strace -f -e trace=write  ls -l 2>&1 >/dev/null | wc -l   ->  2 lines =  1 write
    strace -f -e trace=close  ls -l 2>&1 >/dev/null | wc -l   -> 35 lines = 34 close

Cross-checked with strace -c, which tallies the calls itself: 31 openat
(1 ENOENT), 19 read, 1 write, 34 close. close > openat because sockets (4,
from the name-service lookup) also create descriptors that must be closed.
With output discarded, ls writes exactly ONCE: 31 opens of preparation for a
single write of result.
