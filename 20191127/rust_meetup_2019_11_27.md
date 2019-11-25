<!--
_footer: "*that can run asyncronously"
-->

# Async all the things\*!


---

# Who are we?

Luca Barbato
 * Gentoo
 * VideoLan
 * rav1e
 * rust


Edoardo Morandi
 * Ex-bioinformatician
 * C++
 * Python
 * ... and everything else

---
# What do you know about async/await?

* ECMAScript
    * [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* Ruby
    * [concurrent.Async](http://ruby-concurrency.github.io/concurrent-ruby/master/Concurrent/Async.html)
* Go
    * "You shall not need it"
        * goroutine/channel all the things!
* Async in Rust pre-1.39
    * Old style **futures**
    * tokio-first way
    * other MIO-based experiments...

---
# The beginning of the story

* Some operations make your program sit and wait
    * I/O
        * No/little CPU involved most of the time
* You may want to do something useful meanwhile
* You want start computing as soon as the data is ready
    * You do **not** want to waste cycles in a busy-waiting.
* Multithreading **is not** the best solution.
    * Even if with channels...
* Registering callbacks manually on an event loop does not really scale well.
    * You may use MIO directly if you are *really* into this...

---
# ECMAScript approach

`Promise` is a "magic" object that defer the execution of a function. Some APIs (i.e. `fetch`) return a `Promise`, but almost everything can be _promisified_.

- It comes with some memory/computation costs
- The behaviour at runtime may depend on the ECMAScript interpreter/JIT implementation.

[then()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then), [catch()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) and [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) make its usage quite straightforward.

---
# Go approach

> **Disclaimer**: not a Go programmer.

The I/O happens in some implicit thread/goroutine, if there are other tasks/routines pending they are executed meanwhile.

``` go
	r := strings.NewReader("some io.Reader stream to be read\n")

	buf := make([]byte, 4)
	if _, err := io.ReadFull(r, buf); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", buf)
```

Linear code &ne; linear execution.

---

# Async I/O code and C/C++

* Fabulous collection of lovely [footguns](https://www.gnu.org/software/libc/manual/html_node/Asynchronous-I_002fO.html) thanks to `POSIX.1b`.
    * C++11 [has](https://en.cppreference.com/w/cpp/thread/async) [some](https://en.cppreference.com/w/cpp/thread/future) [more](https://en.cppreference.com/w/cpp/thread/promise)...
* Expect **pain**.

* Eventloops abstracting `epoll`, `kqueue` and friends are still less problematic.

---

# Rust before 1.39
`Future` + `tokio` for the win. But...
* The notation is quite cumbersome.
  ``` rust
    fn add<'a, A, B>(a: A, b: B) -> Box<Future<Item=i32, Error=A::Error> + 'a>
        where A: Future<Item=i32> + 'a,
              B: Future<Item=i32, Error=A::Error> + 'a,
    {
        Box::new(a.join(b).map(|(a, b)| a + b))
    }
  ```
* You can get confused and the borrowck might have to ask a lot if you are *really sure* about what you are doing.
* In some cases you need to pay for an heap allocation, even if it could be theoretically avoided

---
# Example from tokio

```rust
fn main() {
    let addr = "127.0.0.1:1234".parse().unwrap();

    let future = TcpStream::connect(&addr)
        .and_then(|socket| {
            io::write_all(socket, b"hello world")
        })
        .and_then(|(socket, _)| {
            // read exactly 11 bytes
            io::read_exact(socket, vec![0; 11])
        })
        .and_then(|(socket, buf)| {
            println!("got {:?}", buf);
            Ok(())
        })
        .map_err(|_| println!("failed"));

    tokio::run(future);
}
```

---

# We can do better!

## Welcome to `async/await` in Rust 1.39

---

# Example from tokio-0.2-alpha-6

```rust
#[tokio::main]
pub async fn main() -> Result<(), Box<dyn Error>> {
    // Open a TCP stream to the socket address.
    //
    // Note that this is the Tokio TcpStream, which is fully async.
    let mut stream = TcpStream::connect("127.0.0.1:6142").await?;
    println!("created stream");

    let result = stream.write(b"hello world\n").await;
    println!("wrote to stream; success={:?}", result.is_ok());

    Ok(())
}
```

---

# `Future` &ne; `Promise`

* `Future` is lazy -- does not do anything until `await`ed
* `Promise` _automagically_ allocates some resources when created, and runs as soon as it is possible
* `Future` is a **trait**, `Promise` is a concrete **object**
* `async/await` in ES is almost syntactic sugar
* `async/await` in Rust is much more than syntactic sugar

---

# Trivial example

```rust
async fn test(s: &str) -> &str {
  let data = get_data().await;
  let mut range = get_range(&data, s).await;
  drop(data);

  sync_from_remote(&mut range).await;
  &s[range]
}
```

---
# Non trivial details

* Suspension &rarr; stack must be stored, status must be stored
* Lifetime of the stored coroutine
* `Send`/`Sync` of the `Future` depends on the marker of the content
* Optimizations of the state machine

---
# Are we async yet?

---
# Ecosystem status

* [Future](https://doc.rust-lang.org/std/future/trait.Future.html) trait in `std::future` is stable.
* [future**s**](https://crates.io/crates/futures) `0.3` is stable
* [async-std](https://crates.io/crates/async-std) `1.1` is stable
* [tokio](https://crates.io/crates/tokio) `0.2` is in __alpha__

---
# What now?

* There is some overlap between `async-std` and `tokio`.
    * c.f [async-tungstenite](https://crates.io/crates/async-tungstenite) and [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite)
* [std::future::Future](https://doc.rust-lang.org/std/future) and [futures::future::Future](https://docs.rs/futures/0.3/futures/prelude/trait.Future.html) are different.
    * You want to use [crate one](https://book.async.rs/overview/std-and-library-futures.html).
* Thread-parallelism and async-parallelism are different
    * One works better for CPU intensive tasks
    * The other is geared towards I/O
    * (You are welcome to prove me wrong ;))

---
# I want to help!

Lots of useful code can be ported from futures `0.1` to `0.3` or async-std.
* Help in getting tokio out of alpha is welcome
    * [quinn](https://github.com/djc/quinn) relies on tokio, anybody wants to see how it would fare with async-std?
* Actix-web 2.0 is _almost_ out!

---
# Questions?

---
# Thank you!

- See you in Turin next year?
