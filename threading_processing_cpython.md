Multi-threading and Multi-processing in CPython

- There can only be one thread running at any given time in a python process (due to the GIL)

- Multiprocessing is parallelism. Multithreading is concurrency (not actually in parallel, but they are perceived to be because threads start and switch to other threads part way through constantly)

- Multiprocessing is best for computations. Multithreading is best for IO.

- If you have CPU heavy tasks, use multiprocessing with n_process = n_cores and never more (they will compete)

- If you have IO heavy tasks, use multithreading with n_threads = m * n_cores with m a number bigger than 1 that you can tweak on your own.

- The GIL is a single lock on the interpreter itself which adds a rule that execution of any Python bytecode requires acquiring the interpreter lock. This prevents deadlocks (as there is only one lock) and doesn’t introduce much performance overhead. But it effectively makes any CPU-bound Python program single-threaded.
- The GIL solves two major problems: there are no conflicts if two different threads (within the same process) are attempting to edit/read the same resource. Also, there is only one lock so there is no issue of deadlock.

- The GIL does not have much impact on the performance of I/O-bound multi-threaded programs as the lock is shared between threads while they are waiting for I/O.

But a program whose threads are entirely CPU-bound, e.g., a program that processes an image in parts using threads, would not only become single threaded due to the lock but will also see an increase in execution time, as seen in the above example, in comparison to a scenario where it was written to be entirely single-threaded.

***This increase is the result of acquire and release overheads added by the lock.***