# 1 Introduction To Concurrency

##  race conditions


A race condition occurs when two or more operations must execute in the correct order, but the program
has not been written so that this order is guaranteed to be maintained.

basic example
```go
 var data int
2 go func() {
3 	data++
4 }()
5 if data == 0 {
6 	fmt.Printf("the value is %v.\n", data)
7 }

```
no guarantee that the printf will execute

## Atomicity


When something is considered atomic, or to have the property of atomicity, this means that within the
context that it is operating, it is indivisible, or uninterruptible

## Memory Access Synchronization

use sync.Mutex

```go
var memoryAccess sync.Mutex
var value int
go func() {
	memoryAccess.Lock()
	value++
	memoryAccess.Unlock()
}()
memoryAccess.Lock()
if value == 0 {
	fmt.Printf("the value is %v.\n", value)
} else {
	fmt.Printf("the value is %v.\n", value)
}
memoryAccess.Unlock()

```

## Deadlock


A deadlocked program is one in which all concurrent processes are waiting on one another.


Deadlock Example:

```go
type value struct {
	mu    sync.Mutex
	value int
}
var wg sync.WaitGroup
printSum := func(v1, v2 *value) {
	defer wg.Done()
	v1.mu.Lock()
	defer v1.mu.Unlock()
	time.Sleep(2 * time.Second)
	v2.mu.Lock()
	defer v2.mu.Unlock()
	fmt.Printf("sum=%v\n", v1.value+v2.value)
}
var a, b value
wg.Add(2)
go printSum(&a, &b)
go printSum(&b, &a)
wg.Wait()

```
The Coffman Conditions are as follows:

Mutual Exclusion - 
A concurrent process holds exclusive rights to a resource at any one time.


Wait For Condition - 
A concurrent process must simultaneously hold a resource and be waiting for an additional resource.


No Preemption -
A resource held by a concurrent process can only be released by that process, so it fulfills this
condition.


Circular Wait -
A concurrent process (P1) must be waiting on a chain of other concurrent processes (P2), which are
in turn waiting on it (P1), so it fulfills this final condition too.


Let’s examine our contrived program and determine if it meets all four conditions:


1. The printSum function does require exclusive rights to both a and b , so it fulfills this condition.

2. Because printSum holds either a or b and is waiting on the other, it fulfills this condition.

3. We haven’t given any way for our goroutines to be preempted.

4. Our first invocation of printSum is waiting on our second invocation, and vice versa.


## livelocks


Livelocks are programs that are actively performing concurrent operations, but these operations do
nothing to move the state of the program forward

example when you are trying to move left, the person in front of you move left 
livelock example

```go
//func NewCond(l Locker) *Cond
cadence := sync.NewCond(&sync.Mutex{})
go func() {
	for range cadence := sync.NewCond(&sync.Mutex{})
go func() {
    //func Tick(d Duration) <-chan Time
	for range time.Tick(1 * time.Millisecond) {
        //func (c *Cond) Broadcast()
        // Broadcast wakes all goroutines waiting on c.
		cadence.Broadcast()
	}
}()
takeStep := func() {
    cadence.L.Lock()
    //func (c *Cond) Wait()
    //Wait atomically unlocks c.L and suspends execution of the calling goroutine. 
    //After later resuming execution, Wait locks c.L before returning. Unlike in other systems, Wait 
    //cannot return unless awoken by Broadcast or Signal.
    //Because c.L is not locked when Wait first resumes, the caller typically cannot assume 
    //that the condition is true when Wait returns. Instead, the caller should Wait in a loop:
	cadence.Wait()
	cadence.L.Unlock()
}
tryDir := func(dirName string, dir *int32, out *bytes.Buffer) bool {
	fmt.Fprintf(out, " %v", dirName)
	atomic.AddInt32(dir, 1)
	takeStep()
	if atomic.LoadInt32(dir) == 1 {
		fmt.Fprint(out, ". Success!")
		return true
	}
	takeStep()
	atomic.AddInt32(dir, -1)
	return false
}
var left, right int32
tryLeft := func(out *bytes.Buffer) bool { return tryDir("left", &left, out) }
tryRight := func(out *bytes.Buffer) bool { return tryDir("right", &right, out) }

walk := func(walking *sync.WaitGroup, name string) {
	var out bytes.Buffer
	defer func() { fmt.Println(out.String()) }()
	defer walking.Done()
	fmt.Fprintf(&out, "%v is trying to scoot:", name)
	for i := 0; i < 5; i++ {
		if tryLeft(&out) || tryRight(&out) {
			return
		}
	}
	fmt.Fprintf(&out, "\n%v tosses her hands up in exasperation!", name)
}
var peopleInHallway sync.WaitGroup
peopleInHallway.Add(2)
go walk(&peopleInHallway, "Alice")
go walk(&peopleInHallway, "Barbara")
peopleInHallway.Wait()
1 * time.Millisecond) {
		cadence.Broadcast()
	}
}()
takeStep := func() {
	cadence.L.Lock()
	cadence.Wait()
	cadence.L.Unlock()
}
tryDir := func(dirName string, dir *int32, out *bytes.Buffer) bool {
    //1 tryDir allows a person to attempt to move in a direction and returns whether or not they were
    //successful. Each direction is represented as a count of the number of people trying to move in that
    //direction, dir 
    fmt.Fprintf(out, " %v", dirName)
    //First, we declare our intention to move in a direction by incrementing that direction by one. 
    atomic.AddInt32(dir, 1)
    // each person must move at the same rate of speed, or
    //cadence. takeStep simulates a constant cadence between all parties.
	takeStep()
	if atomic.LoadInt32(dir) == 1 {
		fmt.Fprint(out, ". Success!")
		return true
	}
    takeStep()
    //4 Here the person realizes they cannot go in this direction and gives up. We indicate this by
    //decrementing that direction by one
	atomic.AddInt32(dir, -1)
	return false
}
var left, right int32
tryLeft := func(out *bytes.Buffer) bool { return tryDir("left", &left, out) }
tryRight := func(out *bytes.Buffer) bool { return tryDir("right", &right, out) }

walk := func(walking *sync.WaitGroup, name string) {
	var out bytes.Buffer
	defer func() { fmt.Println(out.String()) }()
	defer walking.Done()
	fmt.Fprintf(&out, "%v is trying to scoot:", name)
	for i := 0; i < 5; i++ {
		if tryLeft(&out) || tryRight(&out) {
			return
		}
	}
	fmt.Fprintf(&out, "\n%v tosses her hands up in exasperation!", name)
}
var peopleInHallway sync.WaitGroup
peopleInHallway.Add(2)
go walk(&peopleInHallway, "Alice")
go walk(&peopleInHallway, "Barbara")
peopleInHallway.Wait()


```

output:

Alice is trying to scoot: left right left right left right left right left right

Alice tosses her hands up in exasperation!

Barbara is trying to scoot: left right left right left right left right left right

Barbara tosses her hands up in exasperation!

Livelocks are a subset of a larger set of problems called starvation


Starvation is any situation where a concurrent process cannot get all the resources it needs to perform
work.


starvation example:
```go
var wg sync.WaitGroup
var sharedLock sync.Mutex
const runtime = 1 * time.Second
greedyWorker := func() {
	defer wg.Done()
	var count int
	for begin := time.Now(); time.Since(begin) <= runtime; {
		sharedLock.Lock()
		time.Sleep(3 * time.Nanosecond)
		sharedLock.Unlock()
		count++
	}
	fmt.Printf("Greedy worker was able to execute %v work loops\n", count)
}
politeWorker := func() {
	defer wg.Done()
	var count int
	for begin := time.Now(); time.Since(begin) <= runtime; {
		sharedLock.Lock()
		time.Sleep(1 * time.Nanosecond)
		sharedLock.Unlock()
		sharedLock.Lock()
		time.Sleep(1 * time.Nanosecond)
		sharedLock.Unlock()
		sharedLock.Lock()
		time.Sleep(1 * time.Nanosecond)
		sharedLock.Unlock()
		count++
	}
	fmt.Printf("Polite worker was able to execute %v work loops.\n", count)
}
wg.Add(2)
go greedyWorker()
go politeWorker()
wg.Wait()

```

output:

Polite worker was able to execute 289777 work loops.

Greedy worker was able to execute 471287 work loops


Both workers do the same amount of simulated work
(sleeping for three nanoseconds), but as you can see in the same amount of time, the greedy worker got
almost twice the amount of work done!

starvation can cause your program to behave inefficiently or incorrectly

# 2 Modeling Your Code: Communicating Sequential Processes

The Difference Between Concurrency and Parallelism : Concurrency is a property of the code; parallelism is a property of the running program.


Package sync provides basic synchronization primitives such as mutual exclusion locks. Other than the
Once and WaitGroup types, most are intended for use by low-level library routines. Higher-level
synchronization is better done via channels and communication

when to use primitives vs channels

```
                      +-------------------------------------+
           Yes        |                                     |
  +-------------------+ is it a performance critical section|
  |                   |                                     |
  |                   +----------------+--------------------+
  |                                    |
  |                                    |  No
  |                                    v
  |                  +-----------------+-----------------------+
  |                  |  are you trying to transfter owneship   |        Yes
  |                  | of data                                 +--------------------+
  |                  +----------------+------------------------+                    |
  |                                   |                                             |
  |                                   |  No                                         |
  |                   +---------------v--------------------------+                  |
  |       Yes         | Are you trying to guard internal state   |                  |
  +-------------------+  of struct                               |                  |
  +<                  |                                          |                  |
  |                   +----------------+-------------------------+                  |
  |                                    |                                            |
  |                                    |                                            |
  |                                    | No                                         |
  |                                    |                                            |
  |                    +---------------v--------------------------+                 |
  |                    |  Are You trying to coordinate multiple   |                 |
  |        No          |  pieces of logic                         |       Yes       |
  +<-------------------+                                          +---------------->+
  |                    |                                          |                 |
  |                    +------------------------------------------+                 |
  |                                                                                 |
  |                                                                                 |
  |                                                                        +--------v------+
+-v--------------+                                                         |  use channels |
| use primitives |                                                         +---------------+
+----------------+

```

# 3 Go's Concurrency Building Blocks

## GoRoutines

 green threads — threads that are managed by a language’s
runtime 

Go’s mechanism for hosting goroutines is an implementation of what’s called an M:N scheduler, which
means it maps M green threads to N OS threads. Goroutines are then scheduled onto the green threads.
When we have more goroutines than green threads available, the scheduler handles the distribution of the
goroutines across the available threads and ensures that when these goroutines become blocked, other
goroutines can be run. 

Go follows a model of concurrency called the fork-join model

Go runtime is smart enough so that when a variable is stilled reference and the goroutine that declared the 
variable ends, the reference is transferred to the heap

multiple goroutines can operate against the same address space

BE CAREFUL:  the garbage collector does nothing to collect
goroutines that have been abandoned somehow

sample of benchmarking context switches

note:

Code can block waiting for something to be sent on the channel:
<-signal

```go
func BenchmarkContextSwitch(b *testing.B) {
	var wg sync.WaitGroup
	begin := make(chan struct{})
	c := make(chan struct{})
	var token struct{}
	sender := func() {
		defer wg.Done()
		//1 Here we wait until we’re told to begin. We don’t want the cost of setting up and starting each
		//goroutine to factor into the measurement of context switching.
		<-begin
		for i := 0; i < b.N; i++ {
			//2 Here we send messages to the receiver goroutine. A struct{}{} is called an empty struct and takes
			//up no memory; thus, we are only measuring the time it takes to signal a message.
			c <- token
		}
	}
	receiver := func() {
		defer wg.Done()
		<-begin
		for i := 0; i < b.N; i++ {
			//3 Here we receive a message but do nothing with it.
			<-c
		}
	}
	wg.Add(2)
	go sender()
	go receiver()
	// 4 Here we begin the performance timer.
	b.StartTimer()
	//5 Here we tell the two goroutines to begin, by closing the channel to unblock the sender and reciever goroutines
	close(begin)
	wg.Wait()
}


```

goroutine context switches is faster than OS context switches

# sync package

## WaitGroup

WaitGroup is a great way to wait for a set of concurrent operations to complete when you 

either don’t care about the result of the concurrent operation, or 

you have other means of collecting their results

```go
var wg sync.WaitGroup
wg.Add(1)
go func() {
	defer wg.Done()
	fmt.Println("1st goroutine sleeping...")
	time.Sleep(1)
}()
wg.Add(1)
go func() {
	defer wg.Done()
	fmt.Println("2nd goroutine sleeping...")
	time.Sleep(2)
}()
wg.Wait()
fmt.Println("All goroutines complete.")

```

You can think of a WaitGroup like a concurrent-safe counter

## Mutex and RWMutex

Mutex stands for “mutual exclusion” and is a way to guard
critical sections of your program.

example of using regular mutex:

```go
var count int
var lock sync.Mutex
increment := func() {
	lock.Lock()
	defer lock.Unlock()
	count++
	fmt.Printf("Incrementing: %d\n", count)
}
decrement := func() {
	lock.Lock()
	defer lock.Unlock()
	count--
	fmt.Printf("Decrementing: %d\n", count)
}
// Increment
var arithmetic sync.WaitGroup
for i := 0; i <= 5; i++ {
	arithmetic.Add(1)
	go func() {
		defer arithmetic.Done()
		increment()
	}()
}
// Decrement
for i := 0; i <= 5; i++ {
	arithmetic.Add(1)
	go func() {
		defer arithmetic.Done()
		decrement()
	}()
}
arithmetic.Wait()
fmt.Println("Arithmetic complete.")
```

always call Unlock within a defer statement

The sync.RWMutex is conceptually the same thing as a Mutex : it guards access to memory; however,
RWMutex gives you a little bit more control over the memory. You can request a lock for reading, in which
case you will be granted access unless the lock is being held for writing. This means that an arbitrary
number of readers can hold a reader lock so long as nothing else is holding a writer lock.


```go
producer := func(wg *sync.WaitGroup, l sync.Locker) {
	defer wg.Done()
	for i := 5; i > 0; i-- {
		l.Lock()
		l.Unlock()
		time.Sleep(1)
	}
}
observer := func(wg *sync.WaitGroup, l sync.Locker) {
	defer wg.Done()
	l.Lock()
	defer l.Unlock()
}
test := func(count int, mutex, rwMutex sync.Locker) time.Duration {
	var wg sync.WaitGroup
	wg.Add(count + 1)
	beginTestTime := time.Now()
	go producer(&wg, mutex)
	for i := count; i > 0; i-- {
		go observer(&wg, rwMutex)
	}
	wg.Wait()
	return time.Since(beginTestTime)
}
tw := tabwriter.NewWriter(os.Stdout, 0, 1, 2, ' ', 0)
defer tw.Flush()
var m sync.RWMutex
fmt.Fprintf(tw, "Readers\tRWMutext\tMutex\n")
for i := 0; i < 20; i++ {
	count := int(math.Pow(2, float64(i)))
	fmt.Fprintf(
		tw,
		"%d\t%v\t%v\n",
		count,
		test(count, &m, m.RLocker()), //RLocker inits a read lock for m
		test(count, &m, &m),
	)
}


```

# cond

.a rendezvous point for goroutines waiting for or announcing the occurrence
of an event.

```go
//1 Here we instantiate a new Cond . The NewCond function takes in a type that satisfies the sync.Locker
//interface. This is what allows the Cond type to facilitate coordination with other goroutines in a
//concurrent-safe way.
c := sync.NewCond(&sync.Mutex{})
//2 Here we lock the Locker for this condition. This is necessary because the call to Wait automatically
//calls Unlock on the Locker when entered.
c.L.Lock()
for conditionTrue() == false {
//3 Here we wait to be notified that the condition has occurred. This is a blocking call and the goroutine
//will be suspended
	c.Wait()
}

//4 Here we unlock the Locker for this condition. This is necessary because when the call to Wait exits,
//it calls Lock on the Locker for the condition.
c.L.Unlock()
```

Note that the call to Wait doesn’t just block, it suspends the current
goroutine, allowing other goroutines to run on the OS thread.

we use conditions to coordinate goroutines 

A few other things happen when you call
Wait : upon entering Wait , Unlock is called on the Cond variable’s Locker , and upon exiting Wait , Lock
is called on the Cond variable’s Locker

sample of coordinatign inserts ad removes in a queue:

```go
//1 First, we create our condition using a standard sync.Mutex as the Locker 
c := sync.NewCond(&sync.Mutex{})
//2 Next, we create a slice with a length of zero. Since we know we’ll eventually add 10 items, we
//instantiate it with a capacity of 10
queue := make([]interface{}, 0, 10)
removeFromQueue := func(delay time.Duration) {
	time.Sleep(delay)
	//8 We once again enter the critical section for the condition so we can modify data pertinent to the
	//condition.
	c.L.Lock()
	//9 Here we simulate dequeuing an item by reassigning the head of the slice to the second item.
	queue = queue[1:]
	fmt.Println("Removed from queue")
	//10 Here we exit the condition’s critical section since we’ve successfully dequeued an item
	c.L.Unlock()

	//11 Here we let a goroutine waiting on the condition know that something has occurred.
	c.Signal()
}
for i := 0; i < 10; i++ {
	//3 We enter the critical section for the condition by calling Lock on the condition’s Locker 
	c.L.Lock()
	// 4 Here we check the length of the queue in a loop. This is important because a signal on the condition
	//doesn’t necessarily mean what you’ve been waiting for has occurred — only that something has
	//occurred.
	for len(queue) == 2 {
		//5 We call Wait , which will suspend the main goroutine until a signal on the condition has been sent.
		c.Wait()
	}
	fmt.Println("Adding to queue")
	queue = append(queue, struct{}{})
	//6 Here we create a new goroutine that will dequeue an element after one second.
	go removeFromQueue(1 * time.Second)

	//7 Here we exit the condition’s critical section since we’ve successfully enqueued an item.
	c.L.Unlock()
}

```

we use signal to signal that one goroutine is done

we can use broadcast to broadcast all goroutines are done waiting

button click example, pretty hard to follow so read carefully:

```go
//1 We define a type Button that contains a condition, Clicked 
type Button struct {
	Clicked *sync.Cond
}
button := Button{Clicked: sync.NewCond(&sync.Mutex{})}
//2 Here we define a convenience function that will allow us to register functions to handle signals from
//a condition. Each handler is run on its own goroutine, and subscribe will not exit until that
//goroutine is confirmed to be running.
subscribe := func(c *sync.Cond, fn func()) {
	var goroutineRunning sync.WaitGroup
	goroutineRunning.Add(1)
	go func() {
		goroutineRunning.Done()
		c.L.Lock()
		defer c.L.Unlock()
		c.Wait()
		fn()
	}()
	goroutineRunning.Wait()
}

//3 Here we create a WaitGroup . This is done only to ensure our program doesn’t exit before our writes
//to stdout occur.
var clickRegistered sync.WaitGroup
clickRegistered.Add(3)


//4 Here we register a handler that simulates maximizing the button’s window when the button is
//clicked.
subscribe(button.Clicked, func() {
	fmt.Println("Maximizing window.")
	clickRegistered.Done()
})

//5 Here we register a handler that simulates displaying a dialog box when the mouse is clicked.
subscribe(button.Clicked, func() {
	fmt.Println("Displaying annoying dialog box!")
	clickRegistered.Done()
})

//6 we simulate a user raising the mouse button from having clicked the application’s button.
subscribe(button.Clicked, func() {
	fmt.Println("Mouse clicked.")
	clickRegistered.Done()
})

//3 Here we set a handler for when the mouse button is raised. It in turn calls Broadcast on the
//Clicked Cond to let all handlers know that the mouse button has been clicked (a more robust
//implementation would first check that it had been depressed).

button.Clicked.Broadcast()
clickRegistered.Wait()


```

basically,

the clickeRegistered is to ensure that the functions fn() actually run to print to stdout

the goroutineRunning is used to ensure that the go routine in the subscribe function call is actually kicked off

the button.Clicked is to ensure that the go routines in each subscribe function call is running and waiting before
calling fn()
