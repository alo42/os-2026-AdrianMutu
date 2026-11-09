# Lab 11 — The Race and the Semaphore

## Part 1
1. counter++ is three CPU steps: LOAD from memory, ADD 1, STORE back.
2. A loads 100, B loads 100, A stores 101, B stores 101 — two increments,
   the counter grew by one. Our runs lost nearly half: ~1.03–1.15M of 2M.
3. The scheduler interleaves the threads differently every run, so the number
   of collisions differs — a race is nondeterministic.

## Part 2
1. mutex and sem: three runs each, always exactly 2 000 000.
2. sem_init(&s, 0, 2): two units, so BOTH threads enter the critical section —
   the race returns (~1.94M in our runs). A semaphore is a lock only at 1.
3. The locked version pays lock/unlock on every iteration and serializes the
   threads: 0.024 s (mutex) vs 0.003 s (race, wrong).

## Part 3
1. With start value 1, B passes immediately and prints B: I read "" — the
   empty string, because A has not written yet.
2. A semaphore starting at 0 means "the event has not happened yet".

## Part 4
1. Largest inside = 3; sem_wait(&places) guarantees it (places starts at 3).
2. PLACES = 1 turns the counting semaphore into a mutex.
