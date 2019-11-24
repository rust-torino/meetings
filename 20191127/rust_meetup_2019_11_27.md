# Async all the things*!

<footnote style="font-size: 1.2rem;">*that can run asynchronously</footnote>

---

# Who are we?

Luca Barbato
 * VLC
 * Rav1e
 * Rust

Edoardo Morandi
 * Ex-bioinformatician, C++ & Python
 * Full stack dev

---

# What do you know about async/await?

* ECMAScript
* Go/Ruby
* Async in Rust pre-1.39

---

# The beginning of the story

* I/O operations make your program sit and wait
* Multithreading **is not** the solution

---

# ECMAScript approach

`Promise` is a "magic" object that defer the execution of a function. Some APIs (i.e. `fetch`) return a `Promise`, but almost everything can be _promisified_.

It costs, and the runtime is extremely bound to the ECMAScript interpreter/JIT.

Easier to work with using `await`.

---

# Go/Ruby approach

Disclaimer: not a Go/Ruby programmer.
Linear code &ne; linear execution. If the runtime needs to do I/O, it _automagically_ schedules the operation and resume the execution of things that are ready.

Code is fast to write, short and easy to understand, but programmer does not have full control.

---

# C/C++

Yes, it was not in the list. For reasons.
* Direct access to epoll/kqueue/IOCP
* Hard (C++) or impossible (C) to create a decent low-cost abstraction
* Fabulous way to shoot in your feet
* Really, don't do that

---

# Rust before 1.39
`Future` + `tokio` for the win. But...
* Linearity of code is lost
* Not easy to convince the borrowck in some cases
* In some cases you need to pay for an allocation, even if it could be theoretically avoided

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

* `Future` trait in `std`
* `future 0.3` is stable
* `async-std 1.1` is stable
* `tokio 0.2` is in alpha

---

# Unmaintained (but useful) crates

* Contribute!
* `futures::compat` FTW