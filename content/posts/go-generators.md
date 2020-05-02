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
it paused main goroutine. That's why producer should take care for closing channel, when his when job is done.

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

Generating fibonacci number wasn't very interesing case. Now I want to show you something closer to real life example of usage generator patterns,
and we could improve concurrency of our program.
Let's imagine that you need to perform a lot of really havy operations and then you need to aggregate result of those operations. It could be e.g
data aggregation from multiple files on disk, of fetching data from multiple services. For simplicity, I show you utility for performing healthcheck's.

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

At first glance this implementation looks ok. We're not blocking main goroutine and still can aggregate results of our http calls in for-range loop.
But if we get huge slice of url to check most probalby it won't be the best choice in terms of performance. To get some improvment, we definitly should
wrap each http call in a sepperate goroutine. Ok, but if I spin up a multiple goroutines how I will know that they finish their jobs? It's fundamental question,
casue **ChechService** as a producer should close channel gracefully. I will show you one possible solution for this problem which utilize **sync.WaitGroup**, which
hell me to synchonize all created goroutines.

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

First we're using **Add** method on **sync.WaitGroup** do define desired number of goroutines that we want to spin up.
Main purpose of add method is setting internal **WaitGroup** counter.
When particular goroutine will finish http call, we should invoke **wg.Done()**, which basically decreses counter on **WaitGroup**.
As we you can see there's an additional goroutine with main purpose is to close results channel. To achive that in appropriate time,
**wg.Wait()** blocks execution of the process, until all http call will finished, and internal counter on **WaitGroup** will be again set to 0. Then we can safetly close results channel.

Also you should see quite big performance boost comparing to previous implementation.
Here I prepared some [benchmarks](https://github.com/wbira/golang.concurreny.examples/blob/master/healthcheck/healthcheck_test.go)
that's compare both implementations. In my case implementation with sepperate goroutines per http call was 2x faster than first one,
but it was tested on small number of urls. Most probably difference will growth with number of urls.
Goroutines are very lightweight, so I would say, that we can perform healthcheck quite safely on a several hundred urls, or maybe even several thousand. For bigger numbers (e.g milion) I would look for more sophisticated solution.

