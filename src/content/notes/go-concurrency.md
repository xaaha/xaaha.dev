---
title: Go Concurrency For Dummies
date: 2026-01-22
description: Go Concurrency For Dummies
draft: true
category: Go
---

# Go Concurrency Basics

Go concurrency is about structuring programs so multiple tasks can make progress independently using goroutines (lightweight threads) and channels (safe communication pipes).

## Goroutines

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	items := []string{"apple", "banana", "cherry"}

	go printItems(items) // starts concurrently, doesn't wait

	time.Sleep(100 * time.Millisecond) // without this, main exits before goroutine finishes
}

func printItems(items []string) {
	for _, item := range items {
		fmt.Println(item)
	}
}
```

## WaitGroup: Waiting for Goroutines

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	items := []string{"apple", "banana", "cherry"}

	var wg sync.WaitGroup
	wg.Add(1) // "I'm starting 1 goroutine"

	go printItems(items, &wg)

	wg.Wait() // blocks until printItems calls Done
}

func printItems(items []string, wg *sync.WaitGroup) {
	defer wg.Done() // "This goroutine is done"

	for _, item := range items {
		fmt.Println(item)
	}
}
```

### Multiple Goroutines in a Loop

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	items := []string{"apple", "banana", "cherry"}

	var wg sync.WaitGroup
	wg.Add(len(items)) // Add BEFORE the loop, matching count

	for _, item := range items {
		go func(v string) {
			defer wg.Done()
			fmt.Println(v)
		}(item) // pass item as argument to avoid closure bug
	}

	wg.Wait()
}
```

## Channels: Communication Between Goroutines

```go
package main

import "fmt"

func main() {
	items := []string{"apple", "banana", "cherry"}

	ch := make(chan string) // unbuffered channel

	go func() {
		for _, item := range items {
			ch <- item // send (blocks until received)
		}
		close(ch) // sender closes when done
	}()

	for msg := range ch { // receive until channel closes
		fmt.Println(msg)
	}
}
```

### Buffered Channel

```go
package main

import "fmt"

func main() {
	ch := make(chan string, 2) // buffered channel (capacity 2)

	ch <- "apple"  // doesn't block (buffer has space)
	ch <- "banana" // doesn't block (buffer has space)
	// ch <- "cherry" // would block (buffer full, no receiver)

	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

### WaitGroup + Channel Pattern

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	items := []string{"apple", "banana", "cherry"}

	var wg sync.WaitGroup
	results := make(chan string)

	wg.Add(len(items))
	for _, item := range items {
		go func(v string) {
			defer wg.Done()
			results <- v
		}(item)
	}

	go func() {
		wg.Wait()      // wait for all senders
		close(results) // then close channel
	}()

	for v := range results {
		fmt.Println(v)
	}
}
```

## Worker Pool

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	items := []string{"apple", "banana", "cherry", "date", "elderberry"}

	jobs := make(chan string)
	var wg sync.WaitGroup

	// start 3 workers
    limit := 3
	wg.Add(limit)
	for range limit {
		go worker(i, jobs, &wg)
	}

	// send jobs
	for _, item := range items {
		jobs <- item
	}
	close(jobs)

	wg.Wait()
}

func worker(id int, jobs <-chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("worker %d: %s\n", id, job)
	}
}
```

## Semaphore: Limit Concurrency

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	items := []string{"apple", "banana", "cherry", "date", "elderberry"}

	sem := make(chan struct{}, 2) // max 2 concurrent goroutines
	var wg sync.WaitGroup

	wg.Add(len(items))
	for _, item := range items {
		go func(v string) {
			defer wg.Done()
			sem <- struct{}{} // acquire slot (blocks if full)
			doExpensiveWork(v)
			<-sem // release slot
		}(item)
	}

	wg.Wait()
}

func doExpensiveWork(item string) {
	fmt.Printf("processing %s\n", item)
	time.Sleep(100 * time.Millisecond)
}
```

## Common Mistakes

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	items := []string{"apple", "banana", "cherry"}

	// BUG: Closure captures loop variable (pre-Go 1.22)
	var wg1 sync.WaitGroup
	wg1.Add(len(items))
	for _, item := range items {
		go func() {
			defer wg1.Done()
			fmt.Println(item) // prints last item multiple times
		}()
	}
	wg1.Wait()

	fmt.Println("---")

	// FIX: Pass as argument
	var wg2 sync.WaitGroup
	wg2.Add(len(items))
	for _, item := range items {
		go func(v string) {
			defer wg2.Done()
			fmt.Println(v) // correct
		}(item)
	}
	wg2.Wait()
}
```
