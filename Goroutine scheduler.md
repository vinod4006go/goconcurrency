> ### go runtime
>![image](https://user-images.githubusercontent.com/124967310/220011873-6debb254-4bde-44b1-9053-3e7ead1f0e20.png)

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
