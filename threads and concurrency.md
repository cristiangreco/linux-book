Threads
=======

Concurrency concepts
====================

A race condition is a situation where the result produced by the execution of two (or more) processes/threads depends on the relative order in which they gain access to the cpu.

A critical section is a section of code which accesses shared resources and should be executed atomically. This is generally achieved by means of a mutex.

A deadlock is a situation in which two (or more) threads need to simultaneously access two or more different shared resources, each of which is governed by a separate mutex. When more than one thread is locking the same set of mutexes it may happen that threads block waiting to access a resource currently owned by the another thread (in circle) and there is no progress in the application. The simplest example is two threads acquiring two locks in reverse order (the solution is to acquire them in order or, less frequent, acquire the first lock and then lock the remaining mutexes with a trylock and release all in case of failure).

A livelock is a situation where there is no progress but, differently than deadlock, threads are not blocked: they are just too busy acting in response of each other and cannot do any other progress.

A condition variable allows a thread to notify other threads about changes in the state of a shared resource, and other threads to block waiting for such notification. It is always used in conjunction with a mutex. If no thread is waiting when the condvar is signaled, the signal is lost. Waiting on a condvar causes the associated lock to be released before going to sleep and acquired after wakeup (atomically).