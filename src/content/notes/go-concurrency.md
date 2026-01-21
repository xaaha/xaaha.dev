---
title: Go Concurrency Basics
date: 2026-01-21
description: Go Concurrency Basics
draft: true
category: Go
---

# Here is how the go concurrency works

Here is a simple example of how go concurrency works. Starting with waitgroups.

Waitgroup is used for co-ordination between channels. Remember **"Wait until N goroutines  are finished"**

```go
package main

import (
	"fmt"
	"sync"
)

type Result struct {
	Value string
	Err   error
}

func main() {
	fruits := []string{"apple", "ball", "cherry", "dragonfruit"}

	ch := make(chan Result)
	var wg sync.WaitGroup

	wg.Add(1)
	go printFruits(fruits, ch, &wg)

	go func() {
		wg.Wait()
		close(ch)
	}()

	for res := range ch {
		if res.Err != nil {
			fmt.Println("error:", res.Err)
			continue
		}
		fmt.Println(res.Value)
	}
}

func printFruits(str []string, ch chan Result, wg *sync.WaitGroup) {
	defer wg.Done()

	for _, s := range str {
		if s == "dog" {
			ch <- Result{Err: fmt.Errorf("not a fruit")}
			continue
		}
		ch <- Result{Value: s}
	}
}
```

## Basics of wait group



## Channel Basics

```go
package main

import (
	"fmt"
)

// Simple go routine channel example

type Result struct {
	Value string
	Err   error
}

func main() {
	fruits := []string{"apple", "ball", "cherry", "dragonfruit", "dog"}
	c := make(chan Result)

	go print(fruits, c)

	for res := range c {
		if res.Err != nil {
			fmt.Println("error:", res.Err)
		}
		fmt.Println(res.Value)
	}
}

func print(str []string, ch chan Result) {
	defer close(ch)
	for _, s := range str {
		if s == "dog" {
			ch <- Result{Err: fmt.Errorf("not a fruit")}
			continue
		}
		ch <- Result{Value: s}

	}
}
```
