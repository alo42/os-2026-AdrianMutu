# Lab 17 — Threads Share Memory, and Race (Linux)

## Part 1 — Shared and racing

Program: `fire.c` — two threads of one process add 1 to the same `counter`
1,000,000 times each, with no lock.

Build and runs (3 runs):

```
gcc -Wall -Wextra -pthread -o fire fire.c
./fire   -> counter: 1413362
./fire   -> counter: 1045320
./fire   -> counter: 1110658
expected: 2000000 every time
```

**Observe — Thread 2 prints a non-zero counter at the start. What does that
prove about the memory of the two threads?**

In the first run, the second thread to start printed
`thread 1 sees counter = 283638`. The thread had not incremented anything
yet, so the 283,638 it saw could only have been written by the other thread.
This proves that both threads read and write the very same variable at the
very same address: threads of one process share one address space (one heap,
one set of globals). There is no copy-on-fork isolation as between processes.

**Observe — Why is the final counter less than 2,000,000, and different each
run?**

`counter++` is not one operation but three: (1) **read** the current value of
`counter` from memory into a register, (2) **add** one to the register,
(3) **write** the register back to memory. The two threads run in parallel on
different cores, and the scheduler can interleave these steps arbitrarily.
When both threads read the same value (say 5000), both compute 5001, and both
write 5001, two increments produce a total effect of one: an update is lost.
How often this interleaving happens depends on timing, cache traffic and
scheduling, which differ on every run — hence a different (but always short of
2,000,000) result each time.

## Part 2 — Make it worse, then think

Program: `fire4.c` — same loop, four threads (expected 4,000,000).

```
./fire4  -> counter: 1832426   (lost 2167574)
./fire4  -> counter: 1670248   (lost 2329752)
./fire4  -> counter: 2101788   (lost 1898212)
```

Yes, the loss gets bigger: with two threads about 0.6–0.95 million updates
were lost; with four threads about 1.9–2.3 million are lost. More threads
hammering the same variable means more overlapping read-add-write windows,
so a larger share of the increments is overwritten.

**Observe — In one sentence: what single idea would stop the lost updates?**

Mutual exclusion: make the read-add-write of `counter++` one indivisible
(atomic) step — e.g. surround it with a lock (`pthread_mutex_lock/unlock`) so
only one thread at a time can be between the read and the write.
