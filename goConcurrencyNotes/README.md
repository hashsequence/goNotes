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

## cond

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

## once

once only allow the function passed to once.do to be called once
```go
var count int
increment := func() {
	count++
}
var once sync.Once
var increments sync.WaitGroup
increments.Add(100)
for i := 0; i < 100; i++ {
	go func() {
		defer increments.Done()
		once.Do(increment)
	}()
}
increments.Wait()
fmt.Printf("Count is %d\n", count)

```
output:

count is 1

heres another example:

```go
var count int
increment := func() { count++ }
decrement := func() { count-- }
var once sync.Once
once.Do(increment)
once.Do(decrement)
fmt.Printf("Count: %d\n", count)

```

output:
count: 1

This is because sync.Once only counts the number of
times Do is called, not how many times unique functions passed into Do are called. 

heres another example where you can deadlock with once:

```go
var onceA, onceB sync.Once
var initB func()
initA := func() { onceB.Do(initB) }
initB = func() { onceA.Do(initA) }
onceA.Do(initA)
```

its calling itself to call itself once, but it can't exit since it needs to call itself to exit

## pool

Pool is a concurrent-safe implementation of the object pool pattern.

At a high level, a the pool pattern is a way to create and make available a fixed number, or pool, of things

for use. It’s commonly used to constrain the creation of things that are expensive (e.g., database

connections) so that only a fixed number of them are ever created, but an indeterminate number of

operations can still request access to these things.

```go
myPool := &sync.Pool{
	New: func() interface{} {
		fmt.Println("Creating new instance.")
		return struct{}{}
	},
}
//1 Here we call Get on the pool. These calls will invoke the New function defined on the pool since
//instances haven’t yet been instantiated.
myPool.Get()
instance := myPool.Get()
//2 Here we put an instance previously retrieved back in the pool. This increases the available number
//of instances to one.
myPool.Put(instance)
//3 When this call is executed, we will reuse the instance previously allocated and put it back in the
//pool. The New function will not be invoked.
myPool.Get()

```

here we only allocate 4kb of items in the pool, so we can never grab more than that, in this example if

we didnt use a pool we would potentialy use more 1 gb of memory.

```go
var numCalcsCreated int
calcPool := &sync.Pool{
	New: func() interface{} {
		numCalcsCreated += 1
		mem := make([]byte, 1024)
		return &mem
	},
}
// Seed the pool with 4KB
calcPool.Put(calcPool.New())
calcPool.Put(calcPool.New())
calcPool.Put(calcPool.New())
calcPool.Put(calcPool.New())
const numWorkers = 1024 * 1024
var wg sync.WaitGroup
wg.Add(numWorkers)
for i := numWorkers; i > 0; i-- {
	go func() {
		defer wg.Done()
		mem := calcPool.Get().(*[]byte)
		defer calcPool.Put(mem)
		// Assume something interesting, but quick is being done with
		// this memory.
	}()
}
wg.Wait()
fmt.Printf("%d calculators were created.", numCalcsCreated)

```

Another common situation where a Pool is useful is for warming a cache of pre-allocated objects for
operations that must run as quickly as possible. In this case, instead of trying to guard the host machine’s
memory by constraining the number of objects created, we’re trying to guard consumers’ time by front-
loading the time it takes to get a reference to another object. 


```go
func connectToService() interface{} {
	time.Sleep(1 * time.Second)
	return struct{}{}
}

func startNetworkDaemon() *sync.WaitGroup {
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		server, err := net.Listen("tcp", "localhost:8080")
		if err != nil {
			log.Fatalf("cannot listen: %v", err)
		}
		defer server.Close()
		wg.Done()
		for {
			conn, err := server.Accept()
			if err != nil {
				log.Printf("cannot accept connection: %v", err)
				continue
			}
			connectToService()
			fmt.Fprintln(conn, "")
			conn.Close()
		}
	}()
	return &wg
}

```


using a sync.Pool to host connections to our fictitious service:

```go
func connectToService() interface{} {
	time.Sleep(1 * time.Second)
	return struct{}{}
}

func warmServiceConnCache() *sync.Pool {
	p := &sync.Pool{
		New: connectToService,
	}
	for i := 0; i < 10; i++ {
		p.Put(p.New())
	}
	return p
}

func startNetworkDaemon() *sync.WaitGroup {
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		connPool := warmServiceConnCache()
		server, err := net.Listen("tcp", "localhost:8080")
		if err != nil {
			log.Fatalf("cannot listen: %v", err)
		}
		defer server.Close()
		wg.Done()
		for {
			conn, err := server.Accept()
			if err != nil {
				log.Printf("cannot accept connection: %v", err)
				continue
			}
			svcConn := connPool.Get()
			fmt.Fprintln(conn, "")
			connPool.Put(svcConn)
			conn.Close()
		}
	}()
	return &wg
}

```
use this to benchmark:

```go
func init() {
	daemonStarted := startNetworkDaemon()
	daemonStarted.Wait()
}
func BenchmarkNetworkRequest(b *testing.B) {
	for i := 0; i < b.N; i++ {
		conn, err := net.Dial("tcp", "localhost:8080")
		if err != nil {
			b.Fatalf("cannot dial host: %v", err)
		}
		if _, err := ioutil.ReadAll(conn); err != nil {
			b.Fatalf("cannot read: %v", err)
		}
		conn.Close()
	}
}

```

```bash

cd src/gos-concurrency-building-blocks/the-sync-package/pool/ && \
go test -benchtime=10s -bench=.

```

if you compare the times, using pool is faster


the object pool design pattern is best used either when you have concurrent processes that

require objects, but dispose of them very rapidly after instantiation, or when construction of these objects

could negatively impact memory.


when working with a Pool , just remember the following points:

* When instantiating sync.Pool , give it a New member variable that is thread-safe when called.


* When you receive an instance from Get , make no assumptions regarding the state of the object you receive back.


* Make sure to call Put when you’re finished with the object you pulled out of the pool. Otherwise, the Pool is useless. Usually this is done with defer .


* Objects in the pool must be roughly uniform in makeup.

## Channels

making a basic channel

```go
var dataStream chan interface{}
dataStream = make(chan interface{}
```

read only channel (as in you get messages from the channel):

```go
var dataStream <-chan interface{}
dataStream := make(<-chan interface{})

```

send only channel ( as in you take in messages)

```go
var dataStream chan<- interface{}
dataStream := make(chan<- interface{})

```
Go will implicitly convert bidirectional channels to unidirectional channels when needed. 


Channels in Go are said to be blocking. This means that any goroutine that

attempts to write to a channel that is full will wait until the channel has been emptied, and any goroutine

that attempts to read from a channel that is empty will wait until at least one item is placed on it.


 we can read from a closed channel as well

```go
intStream := make(chan int)
close(intStream)
integer, ok := <- intStream
fmt.Printf("(%v): %v", ok, integer)

```

we can also range over a channel:

```go
intStream := make(chan int)
go func() {
	defer close(intStream)
	for i := 1; i <= 5; i++ {
		intStream <- i
	}
}()
for integer := range intStream {
	fmt.Printf("%v ", integer)
}

```
loop will break when the channel closes


you can use channels to signal multiple routines simultaneously:

```go
begin := make(chan interface{})
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
	wg.Add(1)
	go func(i int) {
		defer wg.Done()
		//1 here the goroutine waits til it can read
		<-begin
		fmt.Printf("%v has begun\n", i)
	}(i)
}
fmt.Println("Unblocking goroutines...")
//2 when we close the channel the channel can be read and unblocks all go routines
close(begin)
wg.Wait()

```

output:


Unblocking goroutines...

4 has begun

2 has begun

3 has begun

0 has begun

1 has begun


sync.Cond type to perform the same behavior, above


* buffered channels


Buffered channels are channels that are given a capacity when they’re
instantiated. This means that even if no reads are performed on the channel, a goroutine can still perform n
writes, where n is the capacity of the buffered channel. 



```go
var dataStream chan interface{}
//Here we create a buffered channel with a capacity of four. This means that we can place four things
//onto the channel regardless of whether it’s being read from.
dataStream = make(chan interface{}, 4)
```

these two are the same:

```go
a := make(chan int)
b := make(chan int, 0)
```
channels block if we are writing into a full channel or reading from an empty channel

An unbuffered channel has a capacity of zero and so it’s already full before any writes. 

buffered channels is like an in memory fifo

example:

```go
c := make(chan rune, 4)
c <- 'A'
c <- 'B'
c <- 'C'
c <- 'D' 
//now the buffer is full and if we try to do a fifth write it will block
c <- 'E' 
//we can only unblock if there is a read somewhere like

//... in some other goroutine:
<- c
```
differences between capacity 0 and 1 in channels:


If the channel is unbuffered (capacity is zero), then communication succeeds only when the sender and receiver are both ready.

If the channel is buffered (capacity >= 1), then send succeeds without blocking if the channel is not full and receive succeeds without blocking if the buffer is not empty.

When putting a value to the intChannelZero like intChannelZero <- 1, where the value be saved?

The value is copied from the sender to the receiver. The value is not saved anywhere other than whatever temporary variables the implementation might use.

The differences between intChannelZero and intChannelOne when putting a value to them.

Send on intChannelZero blocks until a receiver is ready.

Send on intChannelOne blocks until space is available in the buffer.



example of buffered channels:

```go
var stdoutBuff bytes.Buffer
defer stdoutBuff.WriteTo(os.Stdout)
intStream := make(chan int, 4)
go func() {
	defer close(intStream)
	defer fmt.Fprintln(&stdoutBuff, "Producer Done.")
	for i := 0; i < 5; i++ {
		fmt.Fprintf(&stdoutBuff, "Sending: %d\n", i)
		intStream <- i
	}
}()
for integer := range intStream {
	fmt.Fprintf(&stdoutBuff, "Received %v.\n", integer)
}

```

output:


Sending: 0

Sending: 1

Sending: 2

Sending: 3

Sending: 4

Producer Done.

Received 0.

Received 1.

Received 2.

Received 3.

Received 4.


nil channels will deadlock and panic:

```go
var dataStream chan interface{}
<-dataStream
```

```
fatal error: all goroutines are asleep - deadlock!
goroutine 1 [chan receive (nil chan)]:
main.main()
/tmp/babel-23079IVB/go-src-23079O4q.go:9 +0x3f
exit status 2
```

tidbits:

goroutine that owns a channel should:


1. Instantiate the channel.

2. Perform writes, or pass ownership to another goroutine.

3. Close the channel.

4. Ecapsulate the previous three things in this list and expose them via a reader channel.


as a consumer:


1. Knowing when a channel is closed.

2. Responsibly handling blocking for any reason.


example:

```go
chanOwner := func() <-chan int {
	resultStream := make(chan int, 5)
	go func() {
		defer close(resultStream)
		for i := 0; i <= 5; i++ {
			resultStream <- i
		}
	}()
	return resultStream
}
resultStream := chanOwner()
//using range makes sure we are done reading after closing
for result := range resultStream {
	fmt.Printf("Received: %d\n", result)
}
fmt.Println("Done receiving!")

```

## select

 channels are the glue that binds goroutines together

  select statement is the glue that binds channels together



sample layout

```go
var c1, c2 <-chan interface{}
var c3 chan<- interface{}
select {
case <- c1:
// Do something
case <- c2:
// Do something
case c3<- struct{}{}:
// Do something
}

```
Unlike switch blocks, case statements in a select block aren’t tested sequentially, and execution won’t
automatically fall through if none of the criteria are met.


If none of the channels are ready, the entire select statement blocks.

in the go spec:

```
If one or more of the communications can proceed, a single one that can proceed is chosen via a uniform pseudo-random selection. 
Otherwise, if there is a default case, that case is chosen. If there is no default case, the "select" statement blocks until
 at least one of the communications can proceed.
```

this is how you timeout channels:

```go
var c <-chan int
select {
case <-c:
case <-time.After(1 * time.Second):
	fmt.Println("Timed out.")
}

```

we can also have default behaviour if no channels are ready:

```go
start := time.Now()
var c1, c2 <-chan int
select {
case <-c1:
case <-c2:
default:
	fmt.Printf("In default after %v\n\n", time.Since(start))
}

```

this is how we put everything together in to a loop:

```go
one := make(chan interface{})
go func() {
	time.Sleep(5 * time.Second)
	close(done)
}()
workCounter := 0
loop:
for {
	select {
	case <-done:
		break loop
	default:
	}
	// Simulate work
	workCounter++
	time.Sleep(1 * time.Second)
}
fmt.Printf("Achieved %v cycles of work before signalled to stop.\n", workCounter)

```

## GOMAXPROCS

we can control the number of OS threads that wil host the Go work queues:

```go
runtime.GOMAXPROCS(runtime.NumCPU())
```

# 4 Concurrency Patterns in Go

## confinement 

these are the options for safe operations:

1. Synchronization primitives for sharing memory (sync.mutex)
2. Synchronization via communication (e.g. channels)
3. immutable data
4. Data protected by confinement

Confinement is the simple yet powerful idea of ensuring information is only ever available from one concurrent process.


There are two kinds of confinement possible: ad hoc and lexical.


adhoc confinement is acheived through convention, meaning the coder enforces themselves

here is an example:

```go
data := make([]int, 4)
loopData := func(handleData chan<- int) {
	defer close(handleData)
	for i := range data {
		handleData <- data[i]
	}
}
handleData := make(chan int)
go loopData(handleData)
for num := range handleData {
	fmt.Println(num)
}

```

We can see that the data slice of integers is available from both the loopData function and the loop over the handleData channel,
but over time developers might make mistakes and break their own convention


lexical confinement involves using lexical scope to expose only the correct data and concurrency
primitives for multiple concurrent processes to use.

example of lexical confinement:

```go
chanOwner := func() <-chan int {
	//1 Here we instantiate the channel within the lexical scope of the chanOwner function. This limits the
	//scope of the write aspect of the results channel to the closure defined below it. In other words, it
	//confines the write aspect of this channel to prevent other goroutines from writing to it.
	results := make(chan int, 5)
	go func() {
		defer close(results)
		for i := 0; i <= 5; i++ {
			results <- i
		}
	}()
	return results
}
consumer := func(results <-chan int) {
	//3 Here we receive a read-only copy of an int channel. By declaring that the only usage we require is
	// read access, we confine usage of the channel within the consume function to only reads.
	for result := range results {
		fmt.Printf("Received: %d\n", result)
	}
	fmt.Println("Done receiving!")
}
//2 Here we receive the read aspect of the channel and we’re able to pass it into the consumer, which
//can do nothing but read from it. Once again this confines the main goroutine to a read-only view of
//the channel.
results := chanOwner()
consumer(results)
```

Why pursue confinement if we have synchronization available to us? 


The answer is improved performance and reduced cognitive load on developers. Synchronization comes with a cost, and if you can avoid it you won’t have any critical sections, and therefore you won’t have to pay the cost of synchronizing them.

## The for-select Loop

```go
for { // Either loop infinitely or range over something
	select {
	// Do some work with channels
	}
}

```

* scenarios you'll use this pattern
	* Sending iteration variables out on a channel
		```go
		for _, s := range []string{"a", "b", "c"} {
			select {
			case <-done:
				return
			case stringStream <- s:
			}
		}
			
		```
	* Looping infinitely waiting to be stopped
		```go
		for {
			select {
			case <-done:
				return
			default:
				// Do non-preemptable work
			}
			// Do non-preemptable work or here
		}
		```
	
## Preventing Goroutine Leaks

* The goroutine has a few paths to termination:
	* When it has completed its work.
	* When it cannot continue its work due to an unrecoverable error.
	* When it’s told to stop working.

example of bad goroutine:

```go
doWork := func(strings <-chan string) <-chan interface{} {
	completed := make(chan interface{})
	go func() {
		defer fmt.Println("doWork exited.")
		defer close(completed)
		for s := range strings {
			// Do something interesting
			fmt.Println(s)
		}
	}()
	return completed
}
doWork(nil)
// Perhaps more work is done here
fmt.Println("Done.")

```

the strings channel will never actually gets any strings written onto it, and the goroutine containing doWork will remain in memory for the lifetime of this process (we would even deadlock if we joined the goroutine within doWork and the main goroutine).

we can prevent this by using a signal between the parent and child to signal the child to terminate:


```go
doWork := func(
	done <-chan interface{},
	strings <-chan string,
) <-chan interface{} {
	//1 Here we pass the done channel to the doWork function. As a convention, this channel is the first parameter.
	terminated := make(chan interface{})
	go func() {
		defer fmt.Println("doWork exited.")
		defer close(terminated)
		for {
			select {
			case s := <-strings:
				// Do something interesting
				fmt.Println(s)
			//2 On this line we see the ubiquitous for-select pattern in use. One of our case statements is checking whether our done channel has been signaled. If it has, we return from the goroutine.
			case <-done:
				return
			}
		}
	}()
	return terminated
}
done := make(chan interface{})
terminated := doWork(done, nil)
//3 Here we create another goroutine that will cancel the goroutine spawned in doWork if more than one second passes.
go func() {
	// Cancel the operation after 1 second.
	time.Sleep(1 * time.Second)
	fmt.Println("Canceling doWork goroutine...")
	close(done)
}()
//4 This is where we join the goroutine spawned from doWork with the main goroutine.
<-terminated
fmt.Println("Done.")

```

heres another example where of where the goroutine ias blocked on attempting to write a value to a channel:

```go
newRandStream := func() <-chan int {
	randStream := make(chan int)
	go func() {
		defer fmt.Println("newRandStream closure exited.")
		defer close(randStream)
		for {
			randStream <- rand.Int()
		}
	}()
	return randStream
}
randStream := newRandStream()
fmt.Println("3 random ints:")
for i := 1; i <= 3; i++ {
	fmt.Printf("%d: %d\n", i, <-randStream)
}

```

output:

```console
3 random ints:
1: 5577006791947779410
2: 8674665223082153551
3: 6129484611666145821
```

notice "newRandStream closure exited." was never printed 

we can fix this by using the for-select and having a read only channel done <-chan interface{},
closing the done channel will unblock the channel, and return in the goroutine

```go
newRandStream := func(done <-chan interface{}) <-chan int {
	randStream := make(chan int)
	go func() {
		defer fmt.Println("newRandStream closure exited.")
		defer close(randStream)
		for {
			select {
			case randStream <- rand.Int():
			case <-done:
				return
			}
		}
	}()
	return randStream
}
done := make(chan interface{})
randStream := newRandStream(done)
fmt.Println("3 random ints:")
for i := 1; i <= 3; i++ {
	fmt.Printf("%d: %d\n", i, <-randStream)
}
close(done)
// Simulate ongoing work
time.Sleep(1 * time.Second)


```

```console
3 random ints:
1: 5577006791947779410
2: 8674665223082153551
3: 6129484611666145821
newRandStream closure exited
```

REMEMBER:  If a goroutine is responsible for creating a goroutine, it is also responsible for ensuring it can stop the goroutine.

## The or-channel

At times you may find yourself wanting to combine one or more done channels into a single done channel
that closes if any of its component channels close.


This pattern creates a composite done channel through recursion and goroutines:

```go
//1 Here we have our function, or , which takes in a variadic slice of channels and returns a single channel.
var or func(channels ...<-chan interface{}) <-chan interface{}
or = func(channels ...<-chan interface{}) <-chan interface{} {
	switch len(channels) {
	//2 Since this is a recursive function, we must set up termination criteria. The first is that if the variadic
	//slice is empty, we simply return a nil channel. This is consistant with the idea of passing in no
	//channels; we wouldn’t expect a composite channel to do anything.
	case 0:
		return nil
	//3 Our second termination criteria states that if our variadic slice only contains one element, we just
	//return that element.
	case 1:
		return channels[0]
	}
	orDone := make(chan interface{})
	//4 Here is the main body of the function, and where the recursion happens. We create a goroutine so that
	//we can wait for messages on our channels without blocking.
	go func() {
		defer close(orDone)
		switch len(channels) {
		//5 Because of how we’re recursing, every recursive call to or will at least have two channels. As an
		//optimization to keep the number of goroutines constrained, we place a special case here for calls to
		//or with only two channels.
		case 2:
			select {
			case <-channels[0]:
			case <-channels[1]:
			}
		default:
			select {
			case <-channels[0]:
			case <-channels[1]:
			case <-channels[2]:
			//6 Here we recursively create an or-channel from all the channels in our slice after the third index, and
			//then select from this. This recurrence relation will destructure the rest of the slice into or-channels to
			//form a tree from which the first signal will return. We also pass in the orDone channel so that when
			//goroutines up the tree exit, goroutines down the tree also exit.
			case <-or(append(channels[3:], orDone)...):
			}
		}
	}()
	return orDone
}

```
This is a fairly concise function that enables you to combine any number of channels together into a single
channel that will close as soon as any of its component channels are closed, or written to. 


here is how we use it in this example:


```go
//1 This function simply creates a channel that will close when the time specified in the after elapses.
sig := func(after time.Duration) <-chan interface{} {
	c := make(chan interface{})
	go func() {
		defer close(c)
		time.Sleep(after)
	}()
	return c
}
//2 Here we keep track of roughly when the channel from the or function begins to block.
start := time.Now()
<-or(
	sig(2*time.Hour),
	sig(5*time.Minute),
	sig(1*time.Second),
	sig(1*time.Hour),
	sig(1*time.Minute),
)
fmt.Printf("done after %v", time.Since(start))


```

output:

```console
done after 1.000216772s
```

the channel created by 
```go 
sig(1*time.Second) 
```
causes the channel to all close

we can to use this channel to optimize memory because the channels after one 
channel already closes or is read wont be created as it is not neccessary anymore


the context package way gives an alternative to th or - way

## error handling

example of we’ve coupling the potential result with the potential error.

```go
type Result struct {
	Error    error
	Response *http.Response
}
checkStatus := func(done <-chan interface{}, urls ...string) <-chan Result {
	results := make(chan Result)
	go func() {
		defer close(results)
		for _, url := range urls {
			var result Result
			resp, err := http.Get(url)
			result = Result{Error: err, Response: resp}
			select {
			case <-done:
				return
			case results <- result:
			}
		}
	}()
	return results
}
done := make(chan interface{})
defer close(done)
urls := []string{"https://www.google.com", "https://badhost"}
for result := range checkStatus(done, urls...) {
	if result.Error != nil {
		fmt.Printf("error: %v", result.Error)
		continue
	}
	fmt.Printf("Response: %v\n", result.Response.Status)
}

```

## Pipelines


A pipeline is just an abstraction in your system, usually in the form 
of processing streams, or batches of data and breaks down a process or worflow in
terms of stages.


what are the properties of stages?


* A stage consumes and returns the same type

* a stage must be reified by the language so that it may be passed around,
(in go functions are reified and can be passed around, first class functions, etc)


we could think of this as functional programming in one perspective:

here is an example of transforming an array of ints:

```go
	multiply := func(values []int, multiplier int) []int {
		multipliedValues := make([]int, len(values))
		for i, v := range values {
			multipliedValues[i] = v * multiplier
		}
		return multipliedValues
	}

	add := func(values []int, additive int) []int {
		addedValues := make([]int, len(values))
		for i, v := range values {
			addedValues[i] = v + additive
		}
		return addedValues
	}
	ints := []int{1, 2, 3, 4}
	for _, v := range add(multiply(ints, 2), 1) {
		fmt.Println(v)
	}

```

the above example is what known as batch processing since 
we process a batch a time. in stream processing, we process
the values one at a time:

```go
multiply := func(value, multiplier int) int {
		return value * multiplier
	}
	add := func(value, additive int) int {
		return value + additive
	}
	ints := []int{1, 2, 3, 4}
	for _, v := range ints {
		fmt.Println(multiply(add(multiply(v, 2), 1), 2))
	}

```

* So now let's use channels to make pipelines


```go
//generates a recieve only stream, and runs a goroutine that closes the stream
//if you send a value into the done stream, otherwise it will keep
//sending the integers into the stream
generator := func(done <-chan interface{}, integers ...int) <-chan int {
		intStream := make(chan int)
		go func() {
			defer close(intStream)
			for _, i := range integers {
				select {
				case <-done:
					return
				case intStream <- i:
				}
			}
		}()
		return intStream
	}
	//high level of what is going on
	//basically we are returning the transformation of the streams
	//and in each function we run a go routine that iterates through
	//the streams at each stage
	//when we are done printing all the numbers that are completely transformed
	//we close the done channels which would in turn cleanup the goroutines
	multiply := func(
		done <-chan interface{},
		intStream <-chan int,
		multiplier int,
	) <-chan int {
		multipliedStream := make(chan int)
		go func() {
			defer close(multipliedStream)
			for i := range intStream {
				select {
				case <-done:
					return
				case multipliedStream <- i * multiplier:
				}
			}
		}()
		return multipliedStream
	}
	add := func(
		done <-chan interface{},
		intStream <-chan int,
		additive int,
	) <-chan int {
		addedStream := make(chan int)
		go func() {
			defer close(addedStream)
			for i := range intStream {
				select {
				case <-done:
					return
				case addedStream <- i + additive:
				}
			}
		}()
		return addedStream
	}
	done := make(chan interface{})
	defer close(done)
	intStream := generator(done, 1, 2, 3, 4)
	pipeline := multiply(done, add(done, multiply(done, intStream, 2), 1), 2)
	for v := range pipeline {
		fmt.Println(v)
	}

```

## handy generators 

here are some useful code you might reuse a lot in writing concurrent programs:


* this repeat function will repeat the values passed forever until you stop it:


```go
repeat := func(
		done <-chan interface{},
		values ...interface{},
	) <-chan interface{} {
		valueStream := make(chan interface{})
		go func() {
			defer close(valueStream)
			for {
				for _, v := range values {
					select {
					case <-done:
						return
					case valueStream <- v:
					}
				}
			}
		}()
		return valueStream
	}
```

* this take function will only take n items from the stream, and will finish,
and close the done channel

```go

take := func(
		done <-chan interface{},
		valueStream <-chan interface{},
		num int,
	) <-chan interface{} {
		takeStream := make(chan interface{})
		go func() {
			defer close(takeStream)
			for i := 0; i < num; i++ {
				select {
				case <-done:
					return
				case takeStream <- <-valueStream:
				}
			}
		}()
		return takeStream
	}

```

## Fan-Out, Fan-In

Basically, what happens when one of the stages of the pipeline
is computationally expensive and you want to create parallel 
goroutines to split the task up pulls from an upstream stage.
And then we could then merge the results back in,
this pattern is called the fan-out, fan-in pattern

* you can use this pattern when both apply:

	* it doesn't rely on values from the stages before
	* it takes a long time to run


lets start with this piece of code:

this code takes the first 10 random numbers generated 
by a stream, and is filtered by primeFinder, and 
so only spits out the primes

```go
rand := func() interface{} { return rand.Intn(50000000) }
	done := make(chan interface{})
	defer close(done)
	start := time.Now()
	randIntStream := toInt(done, repeatFn(done, rand))
	fmt.Println("Primes:")
	for prime := range take(done, primeFinder(done, randIntStream), 10) {
		fmt.Printf("\t%d\n", prime)
	}
	fmt.Printf("Search took: %v", time.Since(start))
```

there are two stages:

* generating numbers

* checking if the number is a prime

so lets try fanning out:


```go
numFinders := runtime.NumCPU()
finders := make([]<-chan int, numFinders)
for i := 0; i < numFinders; i++ {
	finders[i] = primeFinder(done, randIntStream)
}
```

now we have x amount of goroutines depending on your pc, that is 
pulling from random generator checking if it is prime


but now we have a multiplexer that joins back the multiple streams,
so we have a fan-in function:


```go
fanIn := func(
		done <-chan interface{}, //we have a done channel to allow the goroutines to be torned down
		channels ...<-chan interface{}, // we have a variadic slice of channels to handle the multiple channels that were fanned out
	) <-chan interface{} {
		var wg sync.WaitGroup // we need to use the sync.waitgroup so that we can wait for all channels to be drained 
		multiplexedStream := make(chan interface{})
		multiplex := func(c <-chan interface{}) { // the multiplex function reads from the channel passed and sends it to the multiplexed stream
			defer wg.Done()
			for i := range c {
				select {
				case <-done:
					return
				case multiplexedStream <- i:
				}
			}
		}
		// Select from all the channels
		wg.Add(len(channels))// we add all the channels that are beinf fanned out to the waitgroup to be tracked of
		for _, c := range channels {
			go multiplex(c)
		}
		// Wait for all the reads to complete
		go func() {
			wg.Wait()
			close(multiplexedStream)
		}()
		return multiplexedStream // returns the multiplexed stream 
	}

```

## or-done channel

when we want to encapsulate the function of handling when 
the stream is done we could put it in a orDone function

```go

orDone := func(done, c <-chan interface{}) <-chan interface{} {
		valStream := make(chan interface{})
		go func() {
			defer close(valStream)
			for {
				select {
				case <-done:
					return
				case v, ok := <-c:
					if ok == false {
						return
					}
					select {
					case valStream <- v:
					case <-done:
					}
				}
			}
		}()
		return valStream
	}

```

Doing this allows us to get back to simple for loops, like so:
```go
for val := range orDone(done, myChan) {
// Do something with val
}
```

## the tee channel

* when you want to split values coming in from a channel and send it
into another part of your code base


```go
////in the function you can only write twice, and once one channel get
//a value, it gets blocked with nil 
//when two values are sent, then it should close the done 
//channel and cleanup everything
//this can be expanded to be more than just two
tee := func(
		done <-chan interface{},
		in <-chan interface{},
	) (_, _ <-chan interface{}) {
		out1 := make(chan interface{})
		out2 := make(chan interface{})
		go func() {
			defer close(out1)
			defer close(out2)
			for val := range orDone(done, in) {
				var out1, out2 = out1, out2 // we want to use local versions of out1 and out2 so we  shadow out1 and out2
				for i := 0; i < 2; i++ {
					select {
					case <-done:
					case out1 <- val:
						out1 = nil
					case out2 <- val:
						out2 = nil
					}
				}
			}
		}()
		return out1, out2
	}
```


## the bridge channel


in go we can use a sequence of channels:

```go
<-chan <-chan interface{}
```

```go

bridge := func(
		done <-chan interface{},
		chanStream <-chan <-chan interface{},
	) <-chan interface{} {
		valStream := make(chan interface{})// this channel that will return all values from bridge
		go func() {
			defer close(valStream)
			for { // the loop is responsible for pulling channels off of chanStream and providing them to a nested loop for use
				var stream <-chan interface{}
				select {
				case maybeStream, ok := <-chanStream:
					if ok == false {
						return
					}
					stream = maybeStream
				case <-done:
					return
				}
				for val := range orDone(done, stream) { // this loop is responsible for reading vbalues off the channel it has been given and repating those values onto valstream. when the stream we're currently looping over is closed, we break out of the loop performing the reads from this channel, and continue with the next iteration of the loop, selecting channels to read from. this provides us with an unbroken stream of values.
					select {
					case valStream <- val:
					case <-done:
					}
				}
			}
		}()
		return valStream
	}

	//Now we can use bridge to help present a single-channel facade over
	//a channel of channels. Here’s an example that creates a series of 10 
	//channels, each with one element
	//written to them, and passes these channels into the bridge function:

genVals := func() <-chan <-chan interface{} {
		chanStream := make(chan (<-chan interface{}))
		go func() {
			defer close(chanStream)
			for i := 0; i < 10; i++ {
				stream := make(chan interface{}, 1)
				stream <- i
				close(stream)
				chanStream <- stream
			}
		}()
		return chanStream
	}
	for v := range bridge(nil, genVals()) {
		fmt.Printf("%v ", v)
	}

```

## Queuing 

We can implement a queuing system with buffered channels, but usually it is our
last resort in optimization just  because it can cause sync issues.


It is important to know queuing never really speeds up the program, just changes
the behavior.

lets consider this example 

```go
done := make(chan interface{})
defer close(done)
zeros := take(done, 3, repeat(done, 0))
short := sleep(done, 1*time.Second, zeros)
long := sleep(done, 4*time.Second, short)
pipeline := long

```

this took 13 seconds to finish

This pipeline chains together four stages:


1. A repeat stage that generates an endless stream of 0s.

2. A stage that cancels the previous stages after seeing three items.

3. A “short” stage that sleeps one second.

4. A “long” stage that sleeps four seconds.


What happens if we modify the pipeline to include a buffer?

```go

zeros := take(done, 3, repeat(done, 0))
short := sleep(done, 1*time.Second, zeros)
buffer := buffer(done, 2, short) // Buffers sends from short by 2
long := sleep(done, 4*time.Second, short)
pipeline := long

```

here the pipeline still took 13 seconds, but the short's stage runtime is cut from 9 seconds to 3 seconds


so the  time it spent in a blocking state has been reduced.



the true utility of queues is to decouple stages so that the runtime of one stage has no impact on the runtime of another. Decoupling stages in this manner then cascades to alter the runtime behavior of
the system as a whole, which can be either good or bad depending on your system


Let’s begin by analyzing situations in which queuing can increase the overall performance of your system.

The only applicable situations are:

* If batching requests in a stage saves time.

* If delays in a stage produce a feedback loop into the system.


queuing should be implemented either:


* At the entrance to your pipeline.

* In stages where batching will lead to higher efficiency.


## The Context Package