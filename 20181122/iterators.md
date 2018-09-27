# Rust-Torino 20181122

## Iterators in Rust
- Loops and iterators
- Iterators in `std`
- Writing iterators

---
# Loops and iterators

> **loop**
> A loop is a programming idiom that repeats a sequence of instructions until a specific condition is met.


> **Iterator**
> The iterator pattern allows you to perform some task on a sequence of items in turn. An iterator is responsible for the logic of iterating over each item and  determining when the sequence has finished.
> 

---

# Loops in rust
Straight from the [documentation](https://doc.rust-lang.org/book/loops.html):
``` rust
loop {
    println!("again and forever!");
}
```
```
let mut number = 3;
while number != 0 {
    println!("{}!", number);
    number -= 1;
}
```
```
for num in 0..10 {
    println!("the value is: {}", element);
}
```

---
# Loops in rust: `loop`
If something never ends you may use `loop`:
``` rust
loop {
    println!("again and forever!");
}
```
Or `break` free on condition:
``` rust
let mut condition = 0
loop {
    println!("again and for a while!");
    condition += 1;
    if condition > 5 {
    	break;
    }
}
```
---
# Loops in rust: `while`
`loop` + `break` with some sugar
``` rust
let mut cond = 0;
while cond <= 5 {
    println!("again and for a while!");
    condition += 1;
}
```
---
# Loops in rust: `for`
Not the usual C-`for`:
``` rust
let v = vec![0, 2, 4, 5];
for val in v {
    println!("{}", val);
}
```
The `for` in **rust** always go through a _"sequence"_ and presents its elements one by one, roughly it could be thought as a loop:
``` rust
let v = vec![0, 2, 4, 5];
let mut idx = 0;
let len = v.len();
while idx < len {
    println!("{}", v[idx]);
    idx += 1;
}
```
---

# Iterators and `for`
`for` is actually some sugar over something different:

``` rust
let v = vec![0, 2, 4, 5];
let mut iter = v.iter();

while let Some(el) = iter.next() {
    println!("{}", el);
}
```
- Make an [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) from [`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter) (See [IntoIterator](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html))
- Keep calling [`next()`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#tymethod.next) until there aren't elements.
---
# Iterators: minimal requirements 
`Iterator` is a `Trait`, e.g. `Vec::iter()` produces a `Iter` struct.
To implement an `Iterator` you just need to implement an associated type `Iter` and a `next()` method.

``` rust 
impl Iterator for NoOp {
    type Item = ();
    fn next(&mut self) -> Option<Self::Item> {
        None
    }
}
```
---

# Why iterators
## Speed
The compiler usually desugars the iterator code in a more efficient way.
Even if the `size_hint()` and other optional methods aren't implemented the compiler can avoid *bound-checking*, if they are it can optimize the code by loop-unrolling and/or autovectorize it.
## Clarity
Methods such as `map()`, `filter()`, `rev()`, `position()` are usually more self-descriptive than a bunch of `if`s and explicit index computation.

---

# Some useful methods

## `for_each()`
Same as `for` but takes a _closure_. Quite useful if you are on a tight loop:
``` rust
let my_closure = if cond {
    |v| { /* ... some code ...*/ }
} else {
    |v| { /* some slightly different code */ }
};

my_iter.for_each(my_closure);
```
``` rust
for el in my_iter {
    if cond {
       /* ... some code ... */
    } else {
       /* some slightly different code */
    }
}
```
---
## `by_ref()`
Can be used to not consume the iterator, but keep using from where we left.
``` rust
let first_42 = iter.by_ref().position(|v| v == 42);
let second_42 = iter.by_ref().position(|v| v == 42);

...
```
Without `by_ref()` the first `position()` would consume `iter`.

---
## Usual suspects
You will end up using more often than not and they have the same name in other languages.
- `map()`
- `fold()`
- `find()`
- `filter()`
- `zip()`
- `collect()`

---
# End summary
- Iterators are faster than bare loop quite often
- Once you get used to them you can write more compact code
- More often than not code that is easier to follow
	- But you can overdo and make the code pointlessly convoluted
- A good deal of high level APIs such as Futures follow the same patterns, so expect to see `map()` and friends in many other places.
