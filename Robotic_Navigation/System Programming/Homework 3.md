## Q1 : What does `pthread_join(pthread_t thread, void ** retval)` do ?
### 格式如下
> `int pthread_join(pthread_t thread, void ** retval);`
### 功用
It ==waits== for the thread specified by thread to terminate.  If that thread has already terminated, then  `pthread_join()` returns immediately.  The thread specified by thread must be joinable.

---
## Q2 : What does `pthread_yield(void)` do ?
### 格式如下
> `[[deprecated]] int pthread_yield(void)`
### 功用
It causes the calling thread to ==relinquish the CPU==. The thread is placed at the ==end of the run queue== for its static priority and another thread is scheduled to run.
### keyword 
**relinquish :** to give up something such as a responsibility or claim
> She relinquished control of the family investments to her son.

---
## Q3 : What does `pthread_exit(void *retval)` do ?
### 格式如下
> `void pthread_exit(void *retval);`
### 功用
It terminates the calling thread and returns a value ==via `retval`== that (if the thread is joinable) is available to another thread ==in the same process that calls `pthread_join()`.   ==

---
## Q4 : Why we deprecate `pthread_yield()` and use `sched_yield(2)` instead ?
The key reason is that `pthread_yield()` is not POSIX standard .  On the other hand, `sched_yield(2)` is universally supported across POSIX-compliant operating systems.
