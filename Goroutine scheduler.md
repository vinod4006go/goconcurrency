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


### go runtime
![image](https://user-images.githubusercontent.com/124967310/220011873-6debb254-4bde-44b1-9053-3e7ead1f0e20.png)

### goscheduler

- Why create a scheduler when the operating system can schedule threads for you?
#### Different Threading Models (https://yuriktech.com/2020/03/07/User-Space-Scheduling/#Threading-Models)
>
> ##### 1-1
> When we call to create a Green Thread in our program, it invokes a system call to spawn an Kernel Thread.
This type of Threading model does not need its own scheduler in user space.
Examples would be: Java (JVM), C, Rust
>
> Benefits -
> No need for a user space scheduler as we reuse the preemptive OS scheduler.
>
> Drawbacks -
> Creating a large number of threads consumes a lot of system resources.
> The same drawbacks to context switching apply here to due using the same OS scheduler.

> ###### N-1
> We have multiple Green Threads and only one Kernel Thread.
This type of Threading model does need its own scheduler in user space.
The prime example would be Node.js.
>
> Benefits -
> Due to running on a single Kernel Thread, there is no overhead thinking about race conditions and mutexes.
>
> Drawbacks -
> Cannot leverage multi core processors

> ##### M-N
>With this Threading model we create multiple Green Threads that run on multiple Kernel Threads.
This type of Threading model does need its own scheduler in user space.
Examples would be: RxJava, Akka, Go
>
> Benefits -
Leveraging the best of both worlds
>
>Drawbacks -
You need a really good implementation of a user space scheduler to make this threading model utilize the most out of your system
Data races and sync issues can occur, so it is up to the developers to handle them.
 

- https://github.com/golang/go/blob/master/src/runtime/proc.go
- ![image](https://i0.wp.com/golangbyexample.com/wp-content/uploads/2020/08/Scheduling.jpg?w=759&ssl=1)

GopherCon 2018: Kavya Joshi - The Scheduler Saga - https://www.youtube.com/watch?v=YHRO5WQGh0k
  <img src="https://img.youtube.com/vi/YHRO5WQGh0k/0.jpg" alt="The Scheduler Saga " style="height: 500px; width:500px;"/>
- https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html
