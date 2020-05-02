---
title: "Go geneator pattern"
date: 2020-05-02T21:12:45+01:00
tags: [
  "go",
  "concurrency",
  "generators"
]
type: "post"
---

In this post I would like to present generator pattern in Go.
<!--more-->
Generator is simply a function, that returns [channel](https://tour.golang.org/concurrency/2), thus
generator can be synchronous or asynchronous. This pattern can be use for implementing iterators and improve concurrency of our programs.

### Hello world generator

To ilustrate the pattern I'd like start really simple. Let's take [fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number) and I'll show you basic generator implementation
that produces n fibonacci numbers.

```go
package main

import "fmt"

func GenerateFibonnaciSequence(n int) <-chan int {
	numbers := make(chan int)

	go func() {
		for i, f1, f2 := 0, 0, 1; i < n; i, f1, f2 = i+1, f1+f2, f1 {
			numbers <- f1
		}
		close(numbers)
	}()

	return numbers
}

func main() {
	for num := range fibonacci.GenerateFibonnaciSequence(10) {
		fmt.Println(num)
	}
	fmt.Println("Done!")
}

```

You might wondering why loop that's actualy doing a job is wrapped in a sepperate [goroutine](https://golangbot.com/goroutines/).

```go
func GenerateFibonnaciSequence(n int) <-chan int {
	numbers := make(chan int)

	for i, f1, f2 := 0, 0, 1; i < n; i, f1, f2 = i+1, f1+f2, f1 {
		numbers <- f1
	}

	return numbers
}

```

Sending and receiving are blocking operations for channels, so if we put some value to channel, we'll pause current goroutine.
If we're operating on single goroutine it will cause deadlock, and you'll see this error message.

```bash
fatal error: all goroutines are asleep - deadlock!
```
If we're speaking about deadlock, I'll show you another common pitfall. In following example
I ommited **close(numbers)** line in goroutine and we again end up with deadlock. Why?
Beacause for-range loop still expects that new values will be returned from channel, and
it paused main goroutine. That's why producer should take care for closing channel when job is done.

```go

package main

import "fmt"

func GenerateFibonnaciSequence(n int) <-chan int {
	numbers := make(chan int)

	go func() {
		for i, f1, f2 := 0, 0, 1; i < n; i, f1, f2 = i+1, f1+f2, f1 {
			numbers <- f1
		}
	}()

	return numbers
}

func main() {
	for num := range fibonacci.GenerateFibonnaciSequence(10) {
		fmt.Println(num)
	}
	fmt.Println("Done!")
}
```

2. generowanie liczb 
- przyklad patternu
- jak "czytac" z generatora
- show deadlock
- closing channels

3. healthcheck
