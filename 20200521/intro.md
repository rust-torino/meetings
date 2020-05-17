# Intro to Rust, based on other languages

Different from [this presentation](https://github.com/rust-torino/meetings/blob/master/20190718/presentation.md), in which a Rust project is created from scratch, without comparisons.

---

# Completely new to coding?

Raise your hand now, in case we need to digress a bit

---

# What you will see

## Rosetta code

* Javascript
* Python
* Rust, obviously

---

# Credits

Many thanks to [rosettacode.org](http://rosettacode.org), which made my life easier for this presentation

---

# Hello world!

## Javascript

```javascript
console.log("Hello World!");
```

---

# Hello world!

## Python

```python
print("Hello World!")
```

---

# Hello world!

## Rust

```rust
fn main() {
    pritnln!("Hello World!");
}
```

---

# Hello world!

## Observations

* Explicit `main` fn -- Rust is not interpreted!
* `fn` -> _function_
* Curly braces
* Semicolons
* `println!` is not a function, is a **macro**

---

# Hello world!

## Observations -- cargo expand

```rust
#![feature(prelude_import)]
#[prelude_import]
use std::prelude::v1::*;
#[macro_use]
extern crate std;
fn main() {
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(
            &["Hello world!\n"],
            &match () {
                () => [],
            },
        ));
    };
}
```

### Many details, don't focus on this

---

# Sieve of Erathostenes

[An ancient algorithm to find prime numbers](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)

---

# Sieve of Erathostenes

## Javascript

```javascript
function eratosthenes(limit) {
    var primes = [];
    if (limit >= 2) {
        var sqrtlmt = Math.sqrt(limit) - 2;
        var nums = new Array(); // start with an empty Array...
        for (var i = 2; i <= limit; i++) // and
            nums.push(i); // only initialize the Array once...
        for (var i = 0; i <= sqrtlmt; i++) {
            var p = nums[i]
            if (p)
                for (var j = p * p - 2; j < nums.length; j += p)
                    nums[j] = 0;
        }
        /* ... */
```

---

# Sieve of Erathostenes

## Javascript (2)

```javascript
        /* ... */
        for (var i = 0; i < nums.length; i++) {
            var p = nums[i];
            if (p)
                primes.push(p);
        }
    }
    return primes;
}

console.log(eratosthenes(100));
```

---

# Sieve of Erathostenes

## Python

```python
def primes_upto(limit):
    is_prime = [False] * 2 + [True] * (limit - 1) 
    for n in range(int(limit**0.5 + 1.5)): # stop at ``sqrt(limit)``
        if is_prime[n]:
            for i in range(n*n, limit+1, n):
                is_prime[i] = False
    return [i for i, prime in enumerate(is_prime) if prime]

print(primed_upto(100))
```

---

# Sieve of Erathostenes

## Rust -- non idiomatic approach

```rust
fn primes_upto(limit: u64) -> Vec<u64> {
    let mut is_prime = vec![true; limit as usize + 1];
    is_prime[0] = false;
    is_prime[1] = false;
    for n in 0..((limit as f64).sqrt() + 1.5) as usize {
        if is_prime[n] {
            for i in (n.pow(2)..(limit as usize + 1)).step_by(n) {
                is_prime[i] = false;
            }
        }
    }
    is_prime
        .into_iter()
        .enumerate()
        .filter(|&(_, is_prime)| is_prime)
        .map(|(n, _)| n as u64)
        .collect()
}
```

---

# Sieve of Erathostenes

## Rust -- non idiomatic approach (2)

```rust
fn main() {
    println!("{:?}", primes_upto(100));
}
```

---

# Observations

* ~~Horrible~~ Non idiomatic Rust, _translated_ from Python
* Statically typed, deduction when possible (ML langs style)
* Syntax for return types + generics
* Everything's _const_ by default, `mut` keyword
* Indexing
* No implicit conversions
* Ranges
* Iterators and chaining
* Printing, `Display` and `Debug`

---

# Sieve of Erathostenes

## Idiomatic Rust

```rust
fn primes_upto(limit: usize) -> impl Iterator<Item = usize> {
    let is_prime = if limit < 2 {
        Vec::new()
    } else {
        let mut is_prime = vec![true; limit - 1];
        let limit = (limit as f64).sqrt() as usize;
        for index in 2..limit + 1 {
            let mut iter = is_prime[index - 2..].iter_mut().step_by(index);
            if let Some(true) = iter.next() {
                iter.for_each(|is_prime| *is_prime = false);
            }
        }
        is_prime
    };

    /* ... */
```

---

# Sieve of Erathostenes

## Idiomatic Rust (2)

```rust
    /* ... */
    is_prime
        .into_iter()
        .enumerate()
        .filter(|&(_, is_prime)| is_prime)
        .map(|(index, _)| index + 2)
}

fn main() {
    for prime in primes_upto(100) {
        println!("{}", prime);
    }
}
```

---

# Sieve of Erathostenes

## Observations (whoa!)

* `impl Trait`
* `Iterator`
* `use`
* Everything is an expression
* `if let`
* Iterators, iterators, iterators

---

# Stop here, for now

* Read [the book](https://doc.rust-lang.org/book/)
* Read [the std](https://doc.rust-lang.org/std/)
* Read [TWIR](https://this-week-in-rust.org/)
* Practice with [Rust by Example](https://doc.rust-lang.org/rust-by-example/index.html), [Rustlings](https://github.com/rust-lang/rustlings) and [Exercism](https://exercism.io/)
