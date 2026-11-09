# Lab 12 — Producer, Consumer and the Other Tools

## Part 1
1. sem_wait(&empty) stops the producer: empty starts at 4 and counts free
   slots; at 0 the producer blocks. "in buffer" never exceeds 4.
2. The consumer waits first: full starts at 0, so sem_wait(&full) blocks.
3. Locking the mutex before sem_wait(&empty): when the buffer is full the
   producer sleeps holding the mutex, the consumer can never take an item,
   nobody posts empty — deadlock after 4 puts (timeout killed it, exit 124).

## Part 2
1. The ordinary variable `ready` (protected by the mutex) remembers the state;
   the condition variable remembers nothing — it is only where B sleeps.
2. With `if` it still worked here (one waiter, one signal).
3. `while` is still required: wake-ups can be spurious, and with several
   waiters another thread can consume the condition between signal and wake-up
   — the condition must be re-checked under the mutex.

## Part 3
1. if (++readers == 1) sem_wait(&room); and if (--readers == 0)
   sem_post(&room); — first reader locks the room, last one frees it.
2. ++readers/--readers are themselves three-step operations; without rlock the
   count can corrupt and the room is never freed or is taken twice.
3. Writer 2 waits forever while readers keep arriving — starvation.

## Part 4
1. one holds A and waits for B; two holds B and waits for A: a cycle.
2. Without sleep(1) it hung in 9 of 10 runs — intermittent, which is worse:
   the bug survives testing and strikes rarely.
3. With one global lock order a cycle cannot form: whoever holds B already
   holds A, so no B-holder ever waits for A.
