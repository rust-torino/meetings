# Rust-Torino 20190321

## Making C-ABI-compatible libraries in Rust
- Key concepts
- Channels in Go
- Channels in Rust
- Shortcomings
- Alternatives

---
# Key concepts

A [Channel](https://en.wikipedia.org/wiki/Channel_(programming)) is defined as:

> A model for interprocess communication and synchronization via message passing.

In my words:

> A syncronization primitive to pass around ownership of data across different threads, coroutine or such like.

> A mean to connect two endpoints.

---
# Channels in Go

Channels exist since a while but they got popularized by [Go](https://tour.golang.org/concurrency/2) as the way to pass and retrieve data across [goroutines](https://tour.golang.org/concurrency/1).

``` go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

---
# Channels in Rust

The [channel](https://doc.rust-lang.org/std/sync/mpsc/fn.channel.html) model works quite well with the Rust ownership system.

``` rust
use std::sync::mpsc::channel;
use std::thread;

let (sender, receiver) = channel();

// Spawn off an expensive computation
thread::spawn(move|| {
    sender.send(expensive_computation()).unwrap();
});

// Do some useful work for awhile

// Let's see what that answer was
println!("{:?}", receiver.recv().unwrap());
```

---
# Channels compared

We have `bounded` and `unbounded` variants in both languages.

In **Go** we specify the bound by passing an additional parameter.

``` go
// unbounded
ch := make(chan int)

// bounded
ch : = make(chan int, 100)
```

While in **Rust** we have a separate functions.
``` rust
// unbounded
let (s, r) = channel();

// bounded
let (s, r) = sync_channel(100);
```

---
# Channels compared

The **Rust** is all about the *endpoints* while the **Go** version is about the *channel* itself.
``` go
ch := make(chan int)
go func() { ch <- 42 }()
solution := <-ch
```

``` rust 
let (s, r) = channel();
thread::spawn(move|| {
    s.send(42).unwrap();
});
let solution = r.recv().unwrap();
```

---
# Ergonomy Flaws

The rust channels provided by `std::sync::mpsc` has a number of flaws.

- Confusing terminology: 
	- `tx` and `rx` aren't that intuitive (`s`ender and `r`eceiver are better and now used)
	- `sync_channel` doesn't deliver immediately the fact it is bounded.
- Usability:
	- `Sender` and `Receiver` are `!Sync`
	- Multiple producer, Single consumer

---
# Alternatives

- [crossbeam-channels](https://docs.rs/crossbeam-channel): Faster, leaner, supports [selecting](https://docs.rs/crossbeam-channel/0.3.8/crossbeam_channel/macro.select.html) and has better ergonomy. The internals are really complex though.
- [futures::sync::mpsc](https://docs.rs/futures/0.1.25/futures/sync/mpsc/index.html): Futures-based channels, better ergonomy, but somehow specialized.
- [new-channel](https://github.com/stjepang/new-channel): An attempt to make a subset of crossbeam-channels small enough it could replace `std::sync::mpsc`.