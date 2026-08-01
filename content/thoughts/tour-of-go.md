---
publish: true
title: Tour of Go
created: 2026-07-24
modified: 2026-08-02T04:45:13.598+05:30
tags:
  - learning
  - go
---

# Tour of Go

## 1. Packages and the Capital Letter Rule

### **Q: How does Go decide what's "public" and what's "private"?**

Think about it, most languages make you write `public`, `private`, or `protected` in front of everything. That's a lot of typing for something that rarely changes. Go asks: what if visibility were baked into the _name itself_?

So Go just looks at the first letter.

- `Vertex` (capital V) → anyone outside the package can see and use it.
- `vertex` (lowercase v) → it's a secret, only visible inside its own package.

**Why does this matter in practice?** Every time you name something, you're forced to make a tiny decision: "Should the outside world see this?" That's a nice side effect, it keeps your public API small and intentional instead of accidentally leaking internals.

## 2. Functions: Why the Type Comes Second

### **Q: Why does Go write `func add(x int, y int) int` instead of `int add(int x, int y)` like C?**

Try reading a gnarly C function signature with five parameters, each preceded by its type. Your eyes have to bounce back and forth. Go flips it: name first, type second. Once you get used to it, it's actually easier to scan, you find the name first, then the type detail if you need it.

### **Q: Why can Go functions return more than one value?**

Ask yourself: how do most languages return _both_ a result and an error? Usually you throw an exception, or pass in a pointer to write the error into. Go says, why not just return both, as a pair?

```go
result, err := doSomething()
```

That's it. No hidden exception jumping through your call stack, no out-parameters. You get a value and an error, sitting right there, side by side, impossible to ignore.

### **Q: What about "naked returns"?**

If you name your return values up front (`func split(sum int) (x, y int)`), you can just write `return` with nothing after it, and Go sends back whatever those named variables currently hold. Handy for a three-line function. In a long function with lots of branches? It gets confusing, you lose track of what's being returned. So: fine for short functions, best avoided elsewhere.

## 3. Variables and "Zero Values"

### **Q: What happens if you declare a variable and never assign it anything?**

In C, the answer is genuinely scary, you get garbage memory, and reading it is undefined behavior. Go refuses to let that happen. Every type has a default "zero value" it's automatically set to:

- numbers → `0`
- booleans → `false`
- strings → `""`
- pointers, slices, maps, channels, interfaces → `nil`

So there's no such thing as "uninitialized garbage" in Go. There's always a safe, predictable starting point.

### **Q: Why won't Go let you mix an `int` and a `float64` without casting?**

Because silent conversions are exactly how bugs sneak in, a float quietly getting truncated to an int, losing precision nobody noticed. Go makes you write the conversion explicitly (`float64(x)`), which forces you to _see_ the moment data could be lossy.

### **Q: Then why do constants behave differently?**

Constants aren't stored in memory the way variables are, they're more like symbolic placeholders the compiler resolves at compile time. Because of that, they can carry arbitrarily high precision until the moment they're actually assigned to a typed variable. Small, but a genuinely clever piece of design.

### **Q: What are the actual basic types Go gives you to work with?**

Nothing exotic, just a clean, explicit set:

- **Booleans:** `bool`
- **Strings:** `string`
- **Integers:** `int`, `int8`, `int16`, `int32`, `int64` (and their unsigned cousins `uint`, `uint8`, `uint16`, `uint32`, `uint64`)
- **Floating point:** `float32`, `float64`
- **Complex numbers:** `complex64`, `complex128`
- **Two friendly aliases:** `byte` (just another name for `uint8`) and `rune` (another name for `int32`, used to represent a single Unicode character)

The reason `int` doesn't specify a bit-width by default is deliberate, it's sized to whatever's most natural for the machine you're compiling for (32 or 64 bits), and you only reach for the explicit-width types (`int32`, `uint8`, etc.) when you specifically care about memory layout, overflow behavior, or binary compatibility.

## 4. Control Flow: One Loop to Rule Them All

### **Q: Where's `while` and `do-while` in Go?**

They don't exist. Go looked at `for`, `while`, and `do-while` and asked: aren't these really the same idea in three costumes? So it kept just `for`, and let it flex into all three shapes:

```go
for i := 0; i < 10; i++ { }   // classic counting loop
for condition { }              // this is your "while"
for { }                        // infinite loop
```

One keyword, three jobs. Less to memorize.

### **Q: What's `range` for?**

It's how you walk across a slice, map, or channel without manually tracking an index. It hands you both the position and the value at each step. If you only want one of the two, Go forces you to explicitly throw away the other with `_`, because an unused variable is a compile error, not just a linter warning.

### **Q: Why doesn't Go's `switch` fall through to the next case automatically, like C's does?**

Because in C, forgetting a `break` is one of the most common, most silent bugs there is. Go decided: falling through should be the _unusual_ thing you opt into (with `fallthrough`), not the default you have to remember to prevent.

### **Q: What does `defer` actually buy you?**

Picture opening a file, and needing to close it, but your function has five different `return` statements scattered through error checks. Are you going to remember to close the file at every single exit point? `defer` says: schedule this cleanup _now_, right next to where you opened the resource, and Go guarantees it runs right before the function actually exits, no matter which `return` you hit, or even if you panic.

```go
f, _ := os.Open("file.txt")
defer f.Close()
```

One more wrinkle: if you stack multiple `defer` calls, they run in reverse order, last one deferred, first one executed. Like a stack of plates: last one you put down is the first one you pick back up.

## 5. Structs: Grouping Data Together

### **Q: What's a struct, at its simplest?**

Just a bundle of fields, grouped under one name, Go's answer to "I need one value that's actually made of several related pieces":

```go
type Vertex struct {
    X int
    Y int
}
```

Create one with a struct literal (`v := Vertex{1, 2}`), and access fields with a dot (`v.X`). Nothing fancier than that, no constructors, no required boilerplate.

### **Q: Can I take a pointer to a struct field and modify it directly?**

Yes, and this is where it gets genuinely convenient. `v.X = 4` works whether `v` is a struct value or `v` is a pointer to one, which leads straight into the next question.

## 6. Pointers and the Auto-Dereference Trick

### **Q: If I have a pointer to a struct, do I have to write `(*p).Name` every time I want a field?**

You'd think so, that's the "correct" low-level way to say "go to what this pointer points at, then grab the field." But Go quietly does that dereferencing _for_ you. Write `p.Name`, and the compiler expands it into `(*p).Name` behind the scenes.

**The one place this shortcut doesn't apply:** if you want to replace the _entire_ struct the pointer refers to, you do need the explicit star:

```go
*p = Employee{"New Name", 100}
```

Because here you're not reaching into a field, you're overwriting what the pointer points at, entirely. That's a bigger action, so Go wants you to spell it out.

## 7. Arrays vs. Slices: A Label vs. the Actual Box

### **Q: What's actually different between `[5]int` and `[]int`?**

An array's size is locked into its type, a `[5]int` and a `[6]int` aren't even the same type. Rigid, fixed, predictable.

A slice is something else entirely: it's a small _descriptor_, three fields (pointer, length, capacity), that points at a backing array somewhere else. It's less "the data" and more "a label describing where the data is and how much of it you're looking at."

### **Q: If I pass a slice into a function and modify an element, does the caller see that change?**

Yes, and here's why that surprises people at first. Go passes the slice's descriptor _by value_ (a copy of the label), but that copy still points at the _same_ backing array. Change an element through the slice, and you're editing shared memory. No pointer-to-slice required.

### **Q: Why does `append` sometimes jump capacity in weird amounts, like from 2 to 6, not straight to what you need?**

Say you've got a slice with length and capacity both at 2, and you append 3 more items, you now need room for 5. Go's growth strategy first tries doubling the capacity (2 → 4). Not enough. So it recalculates: 5 elements × 8 bytes each = 40 bytes needed. But the memory allocator doesn't hand out arbitrary byte counts, it works in fixed "size classes" to avoid fragmenting memory, and the next size class up from 40 happens to be 48 bytes. 48 ÷ 8 = 6. So you end up with room for 6, not 5. It looks arbitrary until you realize it's really just rounding to the allocator's nearest bucket.

### **Q: Why can't I do 2D slices in one line, the way I'd declare a 2D array in C?**

Because a `[][]int` isn't one contiguous block of memory, it's a slice _of_ slices, where each row is its own, independently allocated chunk scattered across the heap. That's actually a feature: it means each row can have a _different_ length (a "jagged" array), which a true 2D array could never do. The cost is you have to allocate each row in a loop:

```go
grid := make([][]uint8, rows)
for i := range grid {
    grid[i] = make([]uint8, cols)
}
```

And this is also exactly why you can't do manual pointer-arithmetic tricks to "walk" from row to row, row 1 isn't sitting right after row 0 in memory. It's off somewhere else entirely.

### **Q: What happens if I try to write to a map I never initialized?**

A `nil` map isn't an empty map, it's _no_ map. Reading from it is fine (you'll just get zero values), but writing to it panics. You need `make(map[K]V)` first.

**Q: How do I check if a key exists without a value being ambiguous (e.g. is `0` a real value or a missing one)?**

The comma-ok idiom:

```go
val, ok := m["key"]
```

`ok` tells you the truth regardless of what `val` looks like. No guessing.

## 8. Closures: Functions with a Memory

### **Q: What's actually happening when a function returns another function?**

The inner function doesn't just borrow the outer function's variables temporarily, it _captures_ them, keeping them alive on the heap even after the outer function has technically finished running. Each time you call the outer function again, you get a brand-new, independent bundle of state.

```go
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}
```

Call `counter()` twice, and you get two totally separate counters. Neither can see the other's `count`. That's private state, without needing a class or a global variable.

### **Q: Why does `a, b = b, a+b` matter for something like a Fibonacci generator?**

Because Go evaluates the _entire right-hand side first_, using the old values, and only then assigns everything on the left, all at once. That erases the classic need for a temp variable (`temp = a; a = b; b = temp + b`). It's a small syntax feature, but it removes an entire category of "did I update these in the wrong order" bugs.

## 9. Methods and the Same Auto-Dereference Magic

### **Q: What's a "method" in Go, really, is it different from a function?**

Barely. It's a function with one extra thing attached: a receiver (`func (v Vertex) Abs() float64`), which just tells Go "this function belongs to this type."

### **Q: If a method has a pointer receiver, do I need to manually take the address every time I call it?**

No, and this trips people up because it seems inconsistent with how regular functions behave. A _function_ expecting `*Vertex` will flatly refuse a plain `Vertex` value; no auto-conversion happens there. But a _method_ is more forgiving: call `v.Scale(5)` on a plain value, and if `Scale` needs a pointer receiver, Go automatically rewrites it as `(&v).Scale(5)` for you. The reverse also works, calling a value-receiver method through a pointer auto-dereferences it. Methods get an ergonomic pass that plain functions don't.

### **Q: So when should I actually choose a pointer receiver over a value receiver?**

Ask two questions:

1. **Does this method need to modify the receiver?** If yes, it _must_ be a pointer receiver, a value receiver only ever gets a copy, so any change you make vanishes the moment the method returns.
2. **Is the struct large, or does copying it get expensive?** Even for read-only methods, a big struct is cheaper to pass as a pointer (8 bytes on a 64-bit machine) than to copy wholesale every single call.

The practical rule of thumb: once _any_ method on a type needs a pointer receiver, it's idiomatic to make _all_ the methods on that type use pointer receivers too, for consistency, so callers don't have to remember which methods mutate and which don't.

## 10. Interfaces: Contracts Nobody Signs

### **Q: How does a type "implement" an interface in Go, where's the `implements` keyword?**

There isn't one. Go uses _structural typing_: if your type happens to have all the methods an interface asks for, it satisfies that interface automatically. No declaration, no explicit link. This means you can even define an interface _after_ the type already exists, from a completely different package, which is exactly what lets Go code stay so decoupled.

### **Q: Does having extra methods beyond what the interface needs disqualify a type?**

Not at all. The interface only sets a _minimum bar_. Extra methods are just extra methods, irrelevant to whether the contract is satisfied.

### **Q: This is the trickiest part of Go, why can `err != nil` be true even when `err` looks like it should be nil?**

Here's the mental model that fixes it: an interface value isn't just "a thing", it's secretly _two_ things bolted together: a **type** slot and a **value** slot. It only counts as truly `nil` when _both_ slots are empty.

Now imagine this:

```go
var p *MyError = nil     // a nil pointer, but it HAS a type: *MyError
var err error = p        // stuffed into an interface
```

The interface's type slot is now filled with `*MyError`, even though the value slot is nil. So when you check `err == nil`, Go looks at both slots, sees the type slot isn't empty, and says "not nil." Which feels wrong until you internalize: a nil pointer _wrapped in an interface_ is not the same thing as a bare nil interface. The fix is simple once you know it, return a literal `nil`, not a nil-valued typed pointer.

### **Q: How do I get the concrete type back out of an interface?**

A type assertion:

```go
t, ok := i.(SomeType)
```

Just like the map lookup, `ok` tells you honestly whether it worked, instead of panicking on a bad guess. For handling several possible types at once, a type switch (`switch v := i.(type)`) reads much more cleanly than a chain of assertions.

### **Q: What's `Stringer`, and why does `fmt.Println` sometimes print something custom?**

If your type has a `String() string` method, `fmt` notices and calls it automatically whenever it needs to print your value. You're not fighting the formatter, you're just handing it a translation.

### **Q: What's the classic trap when writing a custom `error` type?**

Calling `fmt.Sprint(e)` _inside_ your own `Error()` method. Because `fmt` will notice your type satisfies `error`, and, to format it, call `.Error()` again. Which calls `fmt.Sprint(e)` again. Infinite loop. The fix: cast to a primitive first before formatting it, so `fmt` doesn't see the interface anymore.

### **Q: Why do most custom error types use a pointer receiver instead of a value receiver?**

Because two structs with identical field values are considered equal when compared as values, so two _unrelated_ errors from two different packages could accidentally compare as "the same error." A pointer is a unique memory address; no two error instances can accidentally collide.

### **Q: With `io.Reader`, why does `Read` return an integer, isn't that awkward compared to just returning the data?**

Because `Read` doesn't allocate new memory for you, it fills in a buffer you already handed it. Old data might still be sitting in unused parts of that buffer from a previous read. The returned integer `n` tells you exactly how many _fresh_ bytes actually landed, so you slice it as `b[:n]` to get only the valid part. Miss this, and you'll end up processing stale garbage.

### **Q: What is the "empty interface" (`interface{}`), and why does it feel like it can hold anything?**

Because it can. `interface{}`, often written today with its alias `any`, specifies _zero_ required methods. And if a type needs to satisfy zero methods to qualify… every single type in Go automatically qualifies. So a variable of type `interface{}` can hold a string, an int, a struct, another interface, literally anything.

The tradeoff: once something's stuffed into an `interface{}`, you've lost the compiler's ability to check what you do with it. You have to type-assert your way back to something concrete before you can meaningfully use it. It's flexible, but it's flexibility you pay for with safety, which is exactly the gap generics were introduced to close for most common cases.

### **Q: How does `image.Image` let you generate a picture out of nothing but code?**

An `image.Image` is just another interface, this time asking for three things: `ColorModel()`, `Bounds()`, and `At(x, y int)`. You don't need actual pixel data sitting in memory anywhere; you just need a type that can _answer these three questions_ for any coordinate.

- `Bounds()` reports the rectangle the image covers, typically returned via the built-in helper `image.Rect(0, 0, width, height)`.
- `At(x, y)` is called for every pixel, and you compute a `color.RGBA{…}` value on the fly, maybe based on a formula, a gradient, or pure randomness.

Nothing is precomputed or stored. The image "exists" purely as a function that knows how to answer "what color is pixel (x, y)?" whenever asked.

## 11. Generics: One Function, Many Types

### **Q: What problem do generics actually solve?**

Before generics, if you wanted an `Index` function that worked for `[]int` _and_ `[]string`, you either wrote it twice, or gave up type safety with `interface{}` and cast things by hand. Generics let you write it once, with a placeholder type:

```go
func Index[T comparable](s []T, x T) int
```

### **Q: Why does that `comparable` word matter, can't `T` just be anything?**

Because `Index` needs to check `s[i] == x`, and not every type in Go supports `==`. Slices, for instance, can't be compared with `==` at all. So if you tried to call `Index` with a `[][]int`, the compiler stops you immediately, `comparable` is the gatekeeper making sure the operation you're about to use is actually legal for whatever type shows up.

### **Q: When I write methods on a generic struct, do I have to repeat the constraint every time?**

No, you declare the constraint once, on the struct itself (`type List[T any] struct`), and after that you just reference `[T]` on its methods without restating what `T` is allowed to be. The constraint lives with the definition, not with every usage.

## 12. Concurrency: Goroutines, Channels, and the GMP Model

### **Q: What actually is a goroutine, a thread?**

Not quite, it's much lighter. An OS thread might reserve megabytes of stack space up front. A goroutine starts around 2KB and grows as needed. That's why Go programs can casually spin up thousands, even millions, of goroutines without buckling.

### **Q: How does Go run a huge number of goroutines on just a handful of CPU cores?**

This is the GMP model:

- **G**oroutine, the actual task and its stack
- **M**achine, a real OS thread
- **P**rocessor, a logical scheduling slot, holding a queue of goroutines ready to run

`GOMAXPROCS` controls how many P's exist (usually one per CPU core). The scheduler's clever move: if a goroutine blocks on something slow, a network call, a syscall, its P doesn't just sit there waiting. The scheduler detaches that P from the stuck M and hands it to a different, free M, so every other goroutine in the queue keeps making progress. Nothing sits idle just because one task is stuck.

### **Q: What's the actual philosophy behind channels?**

Go's famous line: "Don't communicate by sharing memory; share memory by communicating." Instead of multiple goroutines all reaching into the same variable and hoping nothing collides, you hand data through a channel, a typed pipe, and only one goroutine touches it at a time by construction, not by convention.

```go
ch := make(chan int)
ch <- 5        // blocks until someone receives
v := <-ch      // blocks until someone sends
```

An unbuffered channel forces sender and receiver to meet at exactly the same moment, a handoff, not a mailbox. A buffered channel (`make(chan int, 3)`) relaxes that, letting sends succeed without a receiver present, up to the buffer's capacity.

### **Q: How does a receiver know when a sender is completely done sending, rather than just pausing?**

That's exactly what `close(ch)` is for. Closing a channel is a deliberate signal from the sender: "nothing more is coming." A receiver can check this explicitly with the two-value form,

```go
v, ok := <-ch
```

— where `ok` becomes `false` once the channel is closed _and_ drained. Even more conveniently, `range` understands this natively:

```go
for v := range ch {
    // runs once per value, and exits cleanly on its own
    // the moment the channel is closed
}
```

No need to manually check `ok` yourself, the loop just ends gracefully. One important guardrail: only the _sender_ should ever close a channel, never the receiver, and closing a channel more than once is a hard panic. Closing isn't actually mandatory, either; you only need it when a receiver genuinely needs to know "the stream has ended," similar to how a file naturally signals `io.EOF`.

### **Q: What does `select` do that a regular `if` can't?**

`select` waits on _multiple_ channels at once, and reacts to whichever one becomes ready first. If more than one is ready simultaneously, Go picks randomly between them, so no single channel can starve the others by always winning. Add a `default` case, and `select` stops blocking altogether, instead of waiting, it falls through instantly if nothing's ready, which is how you build non-blocking sends and receives.

### **Q: If channels handle communication, why does Go still have `sync.Mutex`?**

Because not everything is naturally a message being passed, sometimes you genuinely just have one shared piece of state (a cache, a counter) that multiple goroutines need to touch directly. For that, you lock:

```go
mu.Lock()
defer mu.Unlock()
```

Pairing `Lock()` with `defer Unlock()` immediately means the lock _always_ releases when the function exits, even if it panics partway through. No deadlock left behind because someone forgot to unlock on one exit path.
