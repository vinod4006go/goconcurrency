# Golang Concurrency

- [goroutines](#goroutines)
- [channels](#channels)
- [goscheduler](#goscheduler)
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



### goscheduler
- The scheduler's job is to distribute ready-to-run goroutines over worker threads
- ![image](https://user-images.githubusercontent.com/124967310/220011873-6debb254-4bde-44b1-9053-3e7ead1f0e20.png)
- https://github.com/golang/go/blob/master/src/runtime/proc.go
- ![image](https://i0.wp.com/golangbyexample.com/wp-content/uploads/2020/08/Scheduling.jpg?w=759&ssl=1)



- GopherCon 2018: Kavya Joshi - The Scheduler Saga - https://www.youtube.com/watch?v=YHRO5WQGh0k
	<img src="https://img.youtube.com/vi/YHRO5WQGh0k/0.jpg" alt="The Scheduler Saga " style="height: 500px; width:500px;"/>
- https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html


### Concurrency patterns in Golang
