---
title: Goroutines
date: 2026-07-24
tags:
  - go
  - concurrency
  - learning
publish: false
---

# [Gist Of Go: Goroutines](https://antonz.org/go-concurrency/goroutines/)

## Goroutines

Functions that run with the `go` prefix, are called *goroutines*. The Go runtime juggles these goroutines and distributes them among operating system threads running on CPU cores. Compared to OS threads, goroutines are lightweight, so you can create hundreds or thousands of them.

```go
func main() {
    go say(1, "go is awesome")
    go say(2, "cats are cute")
    time.Sleep(500 * time.Millisecond)
}
```

## Dependent and independent goroutines

`time.Sleep(500 * time.Millisecond)` is needed in the above code as goroutines are completely independent. When we call `go say()`, the function runs on its own. `main` doesn't wait for it. Hence, the main function finishes before the goroutines have a chance to start. The `main` function is also a goroutine.

### Wait group

Using `time.Sleep()` to wait for goroutines is a bad idea because we can't predict how long they will take. A better approach is to use a *wait group*:

```go
func main() {
    var wg sync.WaitGroup // (1)

    wg.Add(1)             // (2)
    go say(&wg, 1, "go is awesome")

    wg.Add(1)             // (2)
    go say(&wg, 2, "cats are cute")

    wg.Wait()             // (3)
}

// say prints each word of a phrase.
func say(wg *sync.WaitGroup, id int, phrase string) {
    for _, word := range strings.Fields(phrase) {
        fmt.Printf("Worker #%d says: %s...\n", id, word)
        dur := time.Duration(rand.Intn(100)) * time.Millisecond
        time.Sleep(dur)
    }
    wg.Done()             // (4)
}
```

`wg (1)` has a counter inside. Calling `wg.Add(1) (2)` increments it by one, while `wg.Done() (4)` decrements it. `wg.Wait() (3)` blocks the goroutine (in this case, main) until the counter reaches zero. This way, main waits for say(1) and say(2) to finish before it exits.

However, this approach mixes business logic (`say`) with concurrency logic (`wg`). As a result, we can't easily run `say` in regular, non-concurrent code.

```go
func main() {
    var wg sync.WaitGroup
    wg.Add(2)

    go func() {
        defer wg.Done()
        say(1, "go is awesome")
    }()

    go func() {
        defer wg.Done()
        say(2, "cats are cute")
    }()

    wg.Wait()
}
```

### WaitGroup.Go

The `WaitGroup.Go` method (Go 1.25+) automatically increments the wait group counter, runs a function in a goroutine, and decrements the counter when it's done. This means we can rewrite the example above without using `wg.Add()` and `wg.Done()`:

```go
func main() {
    var wg sync.WaitGroup

    wg.Go(func() {
        fmt.Println("go is awesome")
    })

    wg.Go(func() {
        fmt.Println("cats are cute")
    })

    wg.Wait()
    fmt.Println("done")
}
```

The implementation uses `Add` and `Done` just like we did before:

```go
// https://github.com/golang/go/blob/master/src/sync/waitgroup.go
func (wg *WaitGroup) Go(f func()) {
    wg.Add(1)
    go func() {
        defer wg.Done()
        f()
    }()
}
```

## Counting digits in words

```go
// countDigitsInWords counts the number of digits in the words of a phrase.
func countDigitsInWords(phrase string) counter {
    var wg sync.WaitGroup
    syncStats := new(sync.Map)
    words := strings.Fields(phrase)

    // Count the number of digits in words,
    // using a separate goroutine for each word.
    wg.Go(func() {

        for _, word := range words {
            count := 0;

            for _, digit := range word {
                if digit >= '0' && digit <= '9' {
                    count += 1
                }
            }

             // To store the results of the count,
             // use syncStats.Store(word, count)
            syncStats.Store(word, count)
        }
    })

    // As a result, syncStats should contain words
    // and the number of digits in each.
    wg.Wait()

    return asStats(syncStats)
}
```

## Channels

In Go, goroutines can pass values to each other through *channels*. A channel is like a window where one goroutine can throw something and another can catch it.

```text
┌─────────────┐    ┌─────────────┐
│ goroutine A │    │ goroutine B │
│             └────┘             │
│        X <-  chan  <- X        │
│             ┌────┐             │
│             │    │             │
└─────────────┘    └─────────────┘
```

*Goroutine B sends value X to goroutine A.*

```go
func main() {
	// To create a channel, use `make(chan type)`.
    // Channel can only accept values of the specified type:
    messages := make(chan string)

	// To send a value to a channel,
    // use the `channel <-` syntax.
    // Let's send "ping":
    go func() {
        fmt.Println("B: Sending message...")
        messages <- "ping"                    // (1)
        fmt.Println("B: Message sent!")       // (2)
    }()

    fmt.Println("A: Doing some work...")
    time.Sleep(500 * time.Millisecond)
    fmt.Println("A: Ready to receive a message...")

	// To receive a value from a channel,
    // use the `<-channel` syntax.
    // Let's receive "ping" and print it:
    <-messages                               //  (3)

    fmt.Println("A: Messege received!")
    time.Sleep(100 * time.Millisecond)
}
```

After sending the message to the channel (1), goroutine B gets blocked. Only when goroutine A receives the message (3) does goroutine B continue and print "message sent" (2).

So, channels not only transfer data, but also help to synchronize independent goroutines.

## Result channel

```go
// countDigitsInWords counts the number of digits in the words of a phrase.
func countDigitsInWords(phrase string) counter {
    words := strings.Fields(phrase)
    counted := make(chan int)

    go func() {
        // Loop through the words,
        // count the number of digits in each,
        // and write it to the counted channel.
        for _, word := range words {
            count := 0;

            for _, digit := range word {
                if digit >= '0' && digit <= '9' {
                    count += 1
                }
            }

            // and write it to the counted channel.
            counted <- count
        }
    }()

    // Read values from the counted channel
    // and fill stats.
    stats := make(map[string]int)
    for _, word := range words {
        stats[word] = <-counted
    }

    // As a result, stats should contain words
    // and the number of digits in each.
    return stats
}
```

## Related

- [[go-fiber-gorm-boilerplate]]
