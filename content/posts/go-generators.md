---
title: "Go generator pattern"
date: "2020-05-02"
description: "Basic introduction to Go concurrency patterns"
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
generator can be synchronous or asynchronous. This pattern can be used for implementing iterators and improving concurrency of our programs.

### Hello world generator

To ilustrate the pattern I'd like to start really simple. Let's take [fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number) and I'll show you basic generator implementation
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

You might be wondering, why loop that's actualy doing a job is wrapped in a separate [goroutine](https://golangbot.com/goroutines/).

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
If we're operating on a single goroutine, it will cause deadlock and you'll see this error message.

```bash
fatal error: all goroutines are asleep - deadlock!
```
If we're speaking about deadlock, I'll show you another common pitfall. In following example
I ommited `close(numbers)` line in goroutine and we again end up with deadlock. Why?
Beacause for-range loop still expects that new values will be returned from channel and
it paused main goroutine. That's why producer should take care of closing channel, when his job is done.

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

Generating fibonacci number wasn't very interesing case. Now I want to show you something closer to real life example of usage generator patterns
and we could improve concurrency of our program.

Let's imagine that you need to perform a lot of really heavy operations and then you need to aggregate results of those operations. It could be e.g
data aggregation from multiple files on disk or fetching data from multiple services. For simplicity, I'll show you utility for performing healthchecks.

```go
type Result struct {
	Status int
	Error  error
}

func CheckServices(urls []string) <-chan *Result {
	results := make(chan *Result)
	go func() {
		for _, url := range urls {
			response, err := http.Get(url)
			results <- &Result{response.StatusCode, err}
		}
		close(results)
	}()

	return results
}

```

At first glance, this implementation looks ok. We're not blocking main goroutine and still can aggregate results of our http calls in for-range loop.
But if we get huge slice of url to check, most probalby it won't be the best choice in terms of performance. To get some improvment, we definitely should
wrap each http call in a separate goroutine.

Ok, but if I spin up multiple goroutines, how I will know that they finish their jobs? It's fundamental question,
because `ChechService`, as a producer, should close channel gracefully. I will show you one possible solution for this problem, which uses `sync.WaitGroup` and
helps me to synchonize all created goroutines.

```go
func CheckServices(urls []string) <-chan *Result {
	var wg sync.WaitGroup
	results := make(chan *Result)
	wg.Add(len(urls))
	for _, url := range urls {
		go func(url string) {
			response, err := http.Get(url)
			results <- &Result{response.StatusCode, err}
			wg.Done()
		}(url)
	}

	go func() {
		wg.Wait()
		close(results)
	}()
	return results
}
```

First, we're using `Add` method on `sync.WaitGroup` to define desired number of goroutines, that we want to spin up.
Main purpose of `Add` method is setting internal `WaitGroup` counter.
When particular goroutine finishes http call, we should invoke `wg.Done()`, which basically decreases counter on `WaitGroup`.
As you can see, there's an additional goroutine, which main purpose is to close results channel. To achive that in appropriate time,
`wg.Wait()` blocks execution of the process, until all http calls will be finished and internal counter on `WaitGroup` will be again set to 0. Then we can safely close results channel.

Also you should see quite big performance boost, comparing to previous implementation.
Here I prepared some [benchmarks](https://github.com/wbira/golang.concurreny.examples/blob/master/healthcheck/healthcheck_test.go)
that compare both implementations. In my case, implementation with separate goroutines per http call was 2x faster than the first one,
but it was tested on a small number of urls. Most probably difference will grow with number of urls.
Goroutines are very lightweight, so I would say, that we can perform healthchecks quite safely on a several hundred urls or maybe even several thousands. For bigger numbers (e.g milion) I would look for more sophisticated solution.

