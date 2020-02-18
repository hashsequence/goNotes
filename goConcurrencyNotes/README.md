## 1 Introduction To Concurrency

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

## 2 Modeling Your Code: Communicating Sequential Processes

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
