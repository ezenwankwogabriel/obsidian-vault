goroutines run concurrently(same time) at blazing speeds. The sync package provides low-level control tools to prevent them from stomping on each other's memory/running out of order.

#### sync.Mutex (the door lock)
- analogy: a lock on a single-occupancy bathroom door
- use case: protects shared variable from data races (when multiple goroutines read/write same variable at exacly same time)
```
var mu sync.Mutex
count := 0

func increment() {
	mu.Lock() // grab key; others must wait in line
	defer mu.Unlock() // release key at end of function
	count++
}
```
#### sync.RWMutex
- analogy: a public bulletin board
- use case: optimizes performance when many goroutines need to read data, but only an occasional goroutine needs to write data. Multiple readers but a writer gets exclusive access.
```
var rw sync.RWMutex
var cache = map[string]string{}

func Read(key string) string {
	rw.Lock()
	defer rw.Runlock()
	return cache[key]
}
```
#### sync.WaitGroup
- analogy: a tour guide waiting for every person to get back on the bus before driving off
- use case: pauses a main progam until a batch of worker goroutine finishes execution
```
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
	wg.Add(1)
	go func() {
		defer wg.Done()
	}
}

wg.Wait()
```
#### sync.Once (one time trigger)
- analogy: a light switch that gets permanently stuck on "ON" after the first click
- use-case: for initializing a db connection/loading config. runs once, regardless of how many goroutines try to call it simultaneously.
```
var once sync.Once

func setup() {
	once.Do(func() {
		fmt.Println("Database connection initialized)
	})
}
```

#### sync.Pool (the tool shed)
- analogy: A rack of shared bowling shoes at an alley. Rent them, use them, return them for the next person instead of buying a new pair every visit.
- use-case: stores and reuses temp objects to reduce memory allocations and lower Garbage Collector (GC) pressure

---

Alternatively, an actor
One goroutine owns the data, and nobody else touches it. Everyone sends that goroutine messages through a channel, like dropping notes in a mailbox, and it handles them one at a time, more like mimicking one threadedness of nodejs

Go doesn't have built-in `actor` keyword, but its native concurrency primitives implement the core Actor principle seamlessly.
> Do not communicate by sharing memory; instead, share memory by communicating

```
package main

import (
	"fmt"
)

// 1. Define Message Types
type IncrementMsg struct{}

type GetCountMsg struct {
	ReplyChan chan int // Channel to return the response
}

// 2. Define the Actor Struct
type CounterActor struct {
	mailbox chan any
	count   int // Private state - never exposed directly!
}

func NewCounterActor() *CounterActor {
	actor := &CounterActor{
		mailbox: make(chan any, 100), // Mailbox queue
		count:   0,
	}
	// Start the actor's event loop in its own goroutine
	go actor.loop()
	return actor
}

// 3. Event Loop (Processes messages sequentially)
func (a *CounterActor) loop() {
	for msg := range a.mailbox {
		switch m := msg.(type) {
		case IncrementMsg:
			a.count++
		case GetCountMsg:
			m.ReplyChan <- a.count
		}
	}
}

// 4. Public API (Sends messages to the Mailbox)
func (a *CounterActor) Increment() {
	a.mailbox <- IncrementMsg{}
}

func (a *CounterActor) GetCount() int {
	reply := make(chan int)
	a.mailbox <- GetCountMsg{ReplyChan: reply}
	return <-reply // Wait for actor response
}

func main() {
	counter := NewCounterActor()

	// Concurrent callers safely interact without Mutexes
	go counter.Increment()
	go counter.Increment()

	fmt.Println("Current Count:", counter.GetCount())
}
```