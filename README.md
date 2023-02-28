![image](https://www.yakuter.com/wp-content/yuklemeler/1_mdkQasa9ipcJZrSGajSU1A.jpeg)

# Golang Concurrency

- [goroutines](#goroutines)
- [channels](#channels)
- [goscheduler](Goroutine scheduler)
## Concurrency
- Concurrency is about to handle numerous tasks at once.
- Aspect of the problem domain — where your program needs to handle numerous simultaneous events.
- Through concurrency, you want to define a proper structure for your program.

## Parallelism 
- Parallelism is about doing lots of tasks at once.
- It is an aspect of the solution domain — where you want to make your program faster by processing different portions of the problem in parallel.
- Parallelism is a run-time property where two or more tasks are being executed simultaneously.

A concurrent program has multiple logical threads of control.  These threads may or may not run in parallel. 

## Concurrency vs Parallelism 
- Concurrency is not Parallelism by Rob Pike https://www.youtube.com/watch?v=oV9rvDllKEg
	<img src="https://img.youtube.com/vi/oV9rvDllKEg/0.jpg" alt="Concurrency is not Parallelism by Rob Pike" style="height: 500px; width:500px;"/>
- 
- Concurrency can use parallelism for getting its job done but remember parallelism is not the ultimate goal of concurrency.
- Concurrency is when two or more tasks can start, run, and complete in overlapping time periods. It doesn't necessarily mean they'll ever both be running at the same instant. For example, multitasking on a single-core machine.
- Parallelism is when tasks literally run at the same time, e.g., on a multicore processor.

	<img src="https://res.cloudinary.com/practicaldev/image/fetch/s--uXfbq8xZ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ju47qnvgm7d832q5nfev.png" alt="Concurrency" style="height: 500px; width:500px;"/>

- https://dev.to/harinathar/concurrency-and-channels-in-go-jh4

## OS Scheduler
Every program you run creates a Process and each Process is given an initial Thread. Threads have the ability to create more Threads. 
 ### Thread States
- Waiting
- Runnable
- Executing
### Types of Work
- CPU-Bound
- IO-Bound
### Context Switching
The physical act of swapping Threads on a core is called a context switch. A context switch happens when the scheduler pulls an Executing thread off a core and replaces it with a Runnable Thread. 


## Concurrency in Golang 
https://go.dev/tour/concurrency
Concurrency in Golang is the ability for functions to run independently of each other.
Go has rich support for concurrency using goroutines and channels.

### goroutines
- A goroutine is a lightweight thread managed by the Go runtime.
- A goroutine is a function that runs independently of the function that started it. 

```go
go f(x, y, z)
```

```go
func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```

### Channels
- https://www.youtube.com/watch?v=KBZlN0izeiY GopherCon 2017: Kavya Joshi - Understanding Channels
	<img src="https://img.youtube.com/vi/KBZlN0izeiY/0.jpg" alt="Kavya Joshi - Understanding Channels" style="height: 500px; width:500px;"/>
- Channels are the pipes that connect concurrent goroutines.
- You can send values into channels from one goroutine and receive those values into another goroutine.
- A channel in Go provides a connection between two goroutines, allowing them to communicate.
```go
ch := make(chan int)
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, andassign value to v.
```
- The data flows in the direction of the arrow.
- By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.
```go
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
```
#### Buffered channels
- By default channels are unbuffered, meaning that they will only accept sends (chan <-) if there is a corresponding receive (<- chan) ready to receive the sent value.
- Channels can be buffered. Provide the buffer length as the second argument to make to initialize a buffered channel:
- Buffered channels accept a limited number of values without a corresponding receiver for those values.
```go
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
```

https://go.dev/src/runtime/chan.go

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

##### Channel Directions
 if a channel is meant to only send or receive values. This specificity increases the type-safety of the program.
```go
func ping(pings chan<- string, msg string) {
    pings <- msg
}

func pong(pings <-chan string, pongs chan<- string) {
    msg := <-pings
    pongs <- msg
}

func main() {
    pings := make(chan string, 1)
    pongs := make(chan string, 1)
    ping(pings, "passed message")
    pong(pings, pongs)
    fmt.Println(<-pongs)
}
```

 
##### Select
Go’s select lets you wait on multiple channel operations. Combining goroutines and channels with select is a powerful feature of Go.
```go
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        c1 <- "one"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        c2 <- "two"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
```



##### Closing a channel
indicates that no more values will be sent on it. This can be useful to communicate completion to the channel’s receivers.
```go
    jobs := make(chan int, 5)
    done := make(chan bool)

    go func() {
        for {
            j, more := <-jobs
            if more {
                fmt.Println("received job", j)
            } else {
                fmt.Println("received all jobs")
                done <- true
                return
            }
        }
    }()

    for j := 1; j <= 3; j++ {
        jobs <- j
        fmt.Println("sent job", j)
    }
    close(jobs)
    fmt.Println("sent all jobs")

    <-done
```

##### WaitGroups
To wait for multiple goroutines to finish, we can use a wait group.
```go
func worker(id int) {
    fmt.Printf("Worker %d starting\n", id)

    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {

    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)

        i := i

        go func() {
            defer wg.Done()
            worker(i)
        }()
    }

    wg.Wait()

}
```





# Goroutine scheduler


## OS Scheduler
The system scheduler controls multitasking by determining which of the competing threads receives the next processor time slice. 
![image](https://upload.wikimedia.org/wikipedia/commons/thumb/2/25/Concepts-_Program_vs._Process_vs._Thread.jpg/800px-Concepts-_Program_vs._Process_vs._Thread.jpg)

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
- https://appliedgo.net/concurrencyslower/
#### goals
for scheduling goroutines onto kernel threads.
- use a small number of kernel threads. kernel threads are expensive to create.
- support high concurrency. Go programs should be able to run lots and lots of goroutines.
- leverage parallelism i.e. scale to N cores.
- On an N-core machine, Go programs should be able to run N goroutines in parallel.

#### Scheduler Tracing In Go
https://www.ardanlabs.com/blog/2015/02/scheduler-tracing-in-go.html

#### GopherCon 2018: Kavya Joshi - The Scheduler Saga - https://www.youtube.com/watch?v=YHRO5WQGh0k
  <img src="https://img.youtube.com/vi/YHRO5WQGh0k/0.jpg" alt="The Scheduler Saga " style="height: 500px; width:500px;"/>



### Concurrency patterns in Golang
