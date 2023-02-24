## OS Scheduler
The system scheduler controls multitasking by determining which of the competing threads receives the next processor time slice. 


### Processes and Threads
- A process, in the simplest terms, is an executing program. 
- A thread is the basic unit to which the operating system allocates processor time. A thread can execute any part of the process code, including parts currently being executed by another thread.
- Each process provides the resources needed to execute a program. A process has a virtual address space, executable code, open handles to system objects, a security context, a unique process identifier, environment variables, a priority class, minimum and maximum working set sizes, and at least one thread of execution. Each process is started with a single thread, often called the primary thread, but can create additional threads from any of its threads.
- The thread context includes the thread's set of machine registers, the kernel stack, a thread environment block, and a user stack in the address space of the thread's process. Threads can also have their own security context, which can be used for impersonating clients.
- A thread pool is a collection of worker threads that efficiently execute asynchronous callbacks on behalf of the application. The thread pool is primarily used to reduce the number of application threads and provide management of the worker threads.

### Context Switches
The scheduler maintains a queue of executable threads for each priority level. These are known as ready threads. When a processor becomes available, the system performs a context switch. The steps in a context switch are:
- Save the context of the thread that just finished executing.
- Place the thread that just finished executing at the end of the queue for its priority.
- Find the highest priority queue that contains ready threads.
- Remove the thread at the head of the queue, load its context, and execute it.

The most common reasons for a context switch are:
- The time slice has elapsed.
- A thread with a higher priority has become ready to run.
- A running thread needs to wait.
When a running thread needs to wait, it relinquishes the remainder of its time slice.
### 

## User Space Scheduling
https://yuriktech.com/2020/03/07/User-Space-Scheduling/

There are 3 usual models for threading. One is N:1 where several userspace threads are run on one OS thread. This has the advantage of being very quick to context switch but cannot take advantage of multi-core systems. Another is 1:1 where one thread of execution matches one OS thread. It takes advantage of all of the cores on the machine, but context switching is slow because it has to trap through the OS.

Go tries to get the best of both worlds by using a M:N scheduler. It schedules an arbitrary number of goroutines onto an arbitrary number of OS threads. You get quick context switches and you take advantage of all the cores in your system. The main disadvantage of this approach is the complexity it adds to the scheduler.

## go runtime
![image](https://user-images.githubusercontent.com/124967310/220011873-6debb254-4bde-44b1-9053-3e7ead1f0e20.png)

### goscheduler
- https://morsmachine.dk/go-scheduler
- https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw
- https://github.com/golang/go/blob/master/src/runtime/proc.go
- https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html
- https://speakerdeck.com/kavya719/the-scheduler-saga?slide=19

#### Scheduler Tracing In Go
https://www.ardanlabs.com/blog/2015/02/scheduler-tracing-in-go.html

#### GopherCon 2018: Kavya Joshi - The Scheduler Saga - https://www.youtube.com/watch?v=YHRO5WQGh0k
  <img src="https://img.youtube.com/vi/YHRO5WQGh0k/0.jpg" alt="The Scheduler Saga " style="height: 500px; width:500px;"/>

