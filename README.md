# Golang Concurrency
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
- Concurrency can use parallelism for getting its job done but remember parallelism is not the ultimate goal of concurrency.
- Concurrency is when two or more tasks can start, run, and complete in overlapping time periods. It doesn't necessarily mean they'll ever both be running at the same instant. For example, multitasking on a single-core machine.
- Parallelism is when tasks literally run at the same time, e.g., on a multicore processor.

## OS Scheduler
Every program you run creates a Process and each Process is given an initial Thread. Threads have the ability to create more Threads. 
 ### Thread States
- Waiting
- Runnable
- Executing
### Types of Work
- CPU-Bound
- IO-Bound
## Concurrency in Golang
Concurrency in Golang is the ability for functions to run independently of each other.
Go has rich support for concurrency using goroutines and channels.

### goroutines –
- A goroutine is a lightweight thread managed by the Go runtime.
- A goroutine is a function that runs independently of the function that started it. 
### Channels
- Channels are the pipes that connect concurrent goroutines.
- You can send values into channels from one goroutine and receive those values into another goroutine.
- A channel in Go provides a connection between two goroutines, allowing them to communicate.
- By default channels are unbuffered, meaning that they will only accept sends (chan <-) if there is a corresponding receive (<- chan) ready to receive the sent value.
- Buffered channels accept a limited number of values without a corresponding receiver for those values.

##### Channel Directions
 if a channel is meant to only send or receive values. This specificity increases the type-safety of the program.
 
##### Select
Go’s select lets you wait on multiple channel operations. Combining goroutines and channels with select is a powerful feature of Go.

##### Closing a channel
indicates that no more values will be sent on it. This can be useful to communicate completion to the channel’s receivers.

##### WaitGroups
To wait for multiple goroutines to finish, we can use a wait group.



### Go scheduler
### Concurrency patterns in Golang
