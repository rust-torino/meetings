# A pratical intro to Rust

---

# What is Rust

A language empowering everyone to build reliable and efficient software. 

---

# Why Rust

* Performance
* Reliability
* Productivity

---

# Cool, let's do it!

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Courtesy of [rustup.rs](https://rustup.rs)

---

# An objective

We **really need** an objective. Let's try a silly thing:
## Count the unique anchors inside _Appunti di informatica libera_

---

# Let's create a new project

## Cargo!

The tool you really need to handle your projects

---

# Again, Cargo!

```sh
cargo new count_a2
```
(Because the website is a2.pluto.it)

---

# Pick up your editor on steroids

A pretty valid choice is VS code with rls plugin.

I go with (Neo)Vim, sorry :D

---

# A useful (but ready to use) hello world
`count_a2/src/main.rs`
```rust
fn main() {
    println!("Hello, world!");
}
```

---

# Compile it!

```sh
cargo build
```

---

# Run it! (and also compile it)

```sh
cargo run
```

---

# Need more power?

```sh
cargo run --release
```

---

# Take all my HTML!!!!

If you have enough time, give it a look as well.

---

# Step by step

* Read an HTML file (yes, one, with the path harcoded)
* Use a regex to check if we find an anchor
* Use the regex to count the anchors
* Count only the unique anchors
* Iterate over the files to dump the counts for each file
* Count the unique anchors across the files

---

# Read the HTML file

## Where to start?

---

# https://doc.rust-lang.org/std/

And try to search `read`. And maybe `file`...

---

# Got it?

---

```rust
/* Struct std::fs::File */

/// Read all bytes until EOF in this source,
/// appending them to `buf`.
fn read_to_string(
    &mut self,
    buf: &mut String
) -> Result<usize>
```

---

# struct? self? mut? Vec? Result? usize???
Aaaaaaaaaaaahhhh

---

# Take it easy, it's simple

* Create a mutable `std::fs::File` struct (`&mut self`)
* Create a mutable `String` (`&mut String`)
* Use `read_to_string`
* Check the `Result`

---

# Create the `File`

`File::open` has a pretty useful documentation:

```rust
use std::fs::File;

fn main() -> std::io::Result<()> {
    let mut f = File::open("foo.txt")?;
    Ok(())
}
```

---

# Ok... but?

* Why `main` returns a `Result`?
* Why `Result != std::io::Result`?
* What the heck is `<()>`???
* Is the code asking me to open the file?
* Someone forgot the semicolon in the last line?
* Speaking of which... what the heck is `(())`???

---

# Eeeeeeasy

* `main` can return a `Result`. For not, we can forget about it.
* `std::io::Result<T> = Result<T, std::io::Error>`
* `()` is an empty tuple, which is a type (with zero size)
* `?` is for _early return_ in case of errors
* Omitting the semicolon in the last line is like `return`
* `Ok(    ()     )` -- Returning an `Ok` containing an empty tuple

---

# Simpler (for now)

```rust
use std::fs::File;

fn main() {
    let mut f = File::open(
        "path_to_your_html_file"
    ).unwrap();
}
```

---

# Magical recipe

* ~~Create a mutable `std::fs::File` struct (`&mut self`)~~
* Create a mutable `String` (`&mut String`)
* Use `read_to_string`
* Check the `Result`

---

# Check the docs!

---

# String != &str

```rust
let mut buffer = String::new();
```

---

# Magical recipe

* ~~Create a mutable `std::fs::File` struct (`&mut self`)~~
* ~~Create a mutable `String` (`&mut String`)~~
* Use `read_to_string`
* Check the `Result`

---

# Bitten by the compiler

```rust
f.read_to_string(&mut buffer);
```
```
error[E0599]: no method named `read_to_string` found for type `std::fs::File` in the current scope
 --> src/main.rs:6:7
  |
6 |     f.read_to_string(&mut buffer);
  |       ^^^^^^^^^^^^^^
  |
  = help: items from traits can only be used if the trait is in scope
help: the following trait is implemented but not in scope, perhaps add a `use` for it:
  |
1 | use std::io::Read;
  |
```

---

# Import the... trait?

```rust
use std::{fs::File, io::Read};
```

---

# Magical recipe

* ~~Create a mutable `std::fs::File` struct (`&mut self`)~~
* ~~Create a mutable `String` (`&mut String`)~~
* ~~Use `read_to_string`~~
* Check the `Result`

---

# rustc is pedantic

```
warning: unused `std::result::Result` that must be used
 --> src/main.rs:6:5
  |
6 |     f.read_to_string(&mut buffer);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: #[warn(unused_must_use)] on by default
  = note: this `Result` may be an `Err` variant, which should be handled
```

But it's right

---

# Fix it!

```rust
f.read_to_string(&mut buffer).unwrap();
```

---

# `mut`, `mut` everywhere ðŸ˜­

## You know that a block in Rust is an expression, don't you?

---

```rust
use std::{fs::File, io::Read};

fn main() {
    let content = {
        let mut f = File::open("test.html").unwrap();
        let mut buffer = String::new();
        f.read_to_string(&mut buffer).unwrap();
        buffer
    };
    
    // Print the head?
    println!("{}", &content[..300]);
}
```

---

# Step by step

* ~~Read an HTML file (yes, one, with the path harcoded)~~
* Use a regex to check if we find an anchor
* Use the regex to count the anchors
* Count only the unique anchors
* Iterate over the files to dump the counts for each file
* Count the unique anchors across the files

---

# Regex on crate!

## https://docs.rs/regex

---

# Edit your Cargo.toml

```toml
[dependencies]
regex = "1"
```

---

# Create a Regex object

```rust
use regex::Regex;
/* ... */

    let anchor_re = Regex::new(
    	r#"<a[^>]*? href="(?P<href>[^"]*)"[^>]*>"#,
    ).unwrap();
```

---

# How to use it?

## Answers please!

---

# The answer: depends

* `is_match`
* `find`
* `captures`
* `captures_iter`

For now `is_match` can be ok... for now.

---

# Step by step

* ~~Read an HTML file (yes, one, with the path harcoded)~~
* ~~Use a regex to check if we find an anchor~~
* Use the regex to count the anchors
* Count only the unique anchors
* Iterate over the files to dump the counts for each file
* Count the unique anchors across the files

---

# Iterate!

## In Rust you can use both imperative and functional approaches

---

# Imperative

```rust
for capture_match in anchor_re.captures_iter(&content) {
    /* ... */
}
```

---

# Functional

```rust
anchor_re
    .captures_iter(&content)
    .for_each(|capture_match| {
        /* ... */
    });
```

Don't you see the Î»?

---

# Advantages of functional approach

```rust
let anchors = anchor_re
    .captures_iter(&content)
    .count();
```

## RTFM!

---

# Step by step

* ~~Read an HTML file (yes, one, with the path harcoded)~~
* ~~Use a regex to check if we find an anchor~~
* ~~Use the regex to count the anchors~~
* Count only the unique anchors
* Iterate over the files to dump the counts for each file
* Count the unique anchors across the files

---

# Count unique elements

## When you lose your way, you'll probably need a Map

---

# The basic idea

* Map an `href` to a number
* Get an `href`
* Search for the `href` into the map
* If not found, insert the `href` with a counter set to 1
* If it's found, increment the counter

---

# Ownership and borrowing

* `href` is a `&str`
* The map must contain a `String` (not true for all the cases, if you are curious for this case, just ask me)

---

# Let's get the `href`s

```rust
anchor_re
    .captures_iter(&content)
    .for_each(|captures| {
    	let href = captures. /* Give me the answer! */
    });
```

---

# Let's get the `href`s

```rust
anchor_re
    .captures_iter(&content)
    .for_each(|captures| {
    	let href = captures
            .name("href")
            .unwrap()
            .as_str();
        unimplemented!("use href with a map");
        /* Waiting for `todo!` */	
    });
```

---

# Panic!

The code should panic with the first anchor. Let's review the whole code:

```rust
use std::{fs::File, io::Read};
use regex::Regex;

fn main() {
    let content = {
        let mut f = File::open("utilizzo.htm").unwrap();
        let mut buffer = String::new();
        f.read_to_string(&mut buffer).unwrap();
        buffer
    };
    
    let anchor_re = Regex::new(
    	r#"<a[^>]*? href="(?P<href>[^"]*)"[^>]*>"#,
    ).unwrap();

    /* continues... */
```

---


```rust
/* ... */
    anchor_re
        .captures_iter(&content)
        .for_each(|captures| {
            let href = captures
                .name("href")
                .unwrap()
                .as_ref();
            unimplemented!("use href with a map");
            /* Waiting for `todo!` */	
        });
}
```

---

# No Panic!

## Look at the HTML

---

# Uppercases everywhere!!!

## Need a case insensitive regex. Suggestions?

Read the docs!

---

## From

`<a[^>]*? href="(?P<href>[^"]*)"[^>]*>`

## To

`(?i)<a[^>]*? href="(?P<href>[^"]*)"[^>]*>`

---

# Success!! (sort of)

```
thread 'main' panicked at 'not yet implemented: use href with a map', src/main.rs:20:13
note: Run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

---

# Now, map!

## Two major map types

* `BTreeMap`
* `HashMap`

The choice is your. In the real world, benchmarks!
For our example, we will take the `HashMap` (because of 1.36 news)

---

# Init a new map

```rust
use std::collections::HashMap;
/* ... */

    let mut hrefs_occurrences = HashMap::new();
```

---

# Try to find and `href`

```rust
match hrefs_occurrences.get_mut(href) {
    Some(count) => /* ... */
    None => /* ... */
}
```
Thanks NLL

---

# When it's found...

```rust
Some(count) => *count += 1,
```

---

# ...otherwhise...
```rust
None => {
    hrefs_occurrences.insert(href.to_string(), 1);
}
```
Remember the idea of ownership and borrowing?

---
[src/main.rs:24] hrefs_occurrences = {
                  hrefs_occurrences.insert(href.to_string(), 1);                │      "a2394.htm": 1,
              }                                                                 │      "2783.jpg
# The whole code

```rust
let mut hrefs_occurrences = HashMap::new();
anchor_re.captures_iter(&content).for_each(|captures| {
    let href = captures.name("href").unwrap().as_str();
    match hrefs_occurrences.get_mut(href) {
        Some(count) => *count += 1,
        None => {
            hrefs_occurrences.insert(
                href.to_string(),
                1
            );
        }
    }
});
```

---

# Debug it!

```rust
dbg!(hrefs_occurrences);
```
```
[src/main.rs:24] hrefs_occurrences = {
    "a2394.htm": 1,
    "2783.jpg": 1,
    "utilizzo.htm": 1,
    /* ... */
```

---

* ~~Read an HTML file (yes, one, with the path harcoded)~~
* ~~Use a regex to check if we find an anchor~~
* ~~Use the regex to count the anchors~~
* ~~Count only the unique anchors~~
* Iterate over the files to dump the counts for each file
* Count the unique anchors across the files

---

# Iterate the files

## Try to give me the answer!

---

# Iterate the files

```rust
pub fn read_dir<P: AsRef<Path>>(
    path: P
) -> Result<ReadDir>
```

---

# What is `ReadDir`

## Just one click away

```rust
/// Iterator over the entries in a directory.
impl Iterator for ReadDir {
    type Item = Result<DirEntry>;
}
```

---

# Iterating over `Result`s

## Not the easiest think to do using functional approaches, let's do the easy thing

---

# A first try

```rust
use std::fs::read_dir;
/* ... */
    read_dir("appunti_linux")
        .unwrap()
        .map(Result::unwrap)
        .for_each(|entry|) {
            /* ... */
        });
```

---

# Meh

## We want just HTML files

And maybe not a `DirEntry`

---

```rust
read_dir("appunti_linux")
    .unwrap()
    .map(Result::unwrap)
    .map(|entry| entry.path())
    .filter(|file_path| {
        file_path
            .extension()
            .map(|extension| {
                extension
                    .to_str()
                    .unwrap()
                    .starts_with("htm")
            })
            .unwrap_or(false)
    })
    .for_each(|file_path| {
        /* ... */
    });
```

---

# That was a bit of code

## Don't be scared, the functional approach is helpful to focus on a single thing at a time

---

```rust
read_dir("appunti_linux")
    .unwrap() /* dir must exist */
    .map(Result::unwrap) /* dir must STILL exist */
    .map(|entry| entry.path()) /* DirEntry -> PathBuf */
    .filter(|file_path| { /* Iterate only if */
        file_path
            .extension() /* Get the extension */
            .map(|extension| { /* When exists */
                extension
                    .to_str() /* &OsStr -> &str */
                    .unwrap() /* Must be convertible */
                    .starts_with("htm") /* Err... */
            })
            .unwrap_or(false) /* Skip when extension
                               * does not exist
                               */
    })
    .for_each(|file_path| {
        /* ... */
    });
```

---

# Glue things together

```rust
/* ... */
.for_each(|file_path| {
    let content = {
        let mut f = File::open(file_path).unwrap();
        let mut buffer = String::new();
        /* ... */
});
```

## Done!

---

* ~~Read an HTML file (yes, one, with the path harcoded)~~
* ~~Use a regex to check if we find an anchor~~
* ~~Use the regex to count the anchors~~
* ~~Count only the unique anchors~~
* ~~Iterate over the files to dump the counts for each file~~
* Count the unique anchors across the files

---

# This is easy

## Just move the map outside the iteration (and the Regex) and print it in a decent way

---

```rust
fn main() {
    let anchor_re = Regex::new(
        r#"(?i)<a[^>]*? href="(?P<href>[^"]*)"[^>]*>"#
    ).unwrap();
    let mut hrefs_occurrences = HashMap::new();
    
    /* ... the whole iteration */

    hrefs_occurrences
        .into_iter()
        .for_each(|(href, count)| {
            println!("{} -> {}", href, count)
         });
}
```

---

# DONE!

## If you want to take the sum, it is pretty easy as well. Try it.

---

# Are we really done?

## Rust has more superpowers that you can think of...

---

# Multithread made easy

## Powered by `rayon`

---

# Rayon is simple

```rust
    read_dir("/home/edoval/appunti_linux/")
        .unwrap()
        .par_bridge()
```

Just remember to add it to `Cargo.toml`

---

# But Rust is safe

```
error[E0596]: cannot borrow `hrefs_occurrences` as mutable, as it is a captured variable in a `Fn` closure
  --> src/main.rs:33:56
   |
33 |             anchor_re.captures_iter(&content).for_each(|captures| {
   |                                                        ^^^^^^^^^^ cannot borrow as mutable
34 |                 let href = captures.name("href").unwrap().as_str();
35 |                 match hrefs_occurrences.get_mut(href) {
   |                       ----------------- mutable borrow occurs due to use of `hrefs_occurrences` in closure
   |
help: consider changing this to accept closures that implement `FnMut`
  --> src/main.rs:23:19
   |
23 |           .for_each(|file_path| {
   |  ___________________^
24 | |             let content = {
25 | |                 let mut f = File::open(file_path).unwrap();
26 | |                 let mut buffer = String::new();
...  |
41 | |             });
42 | |         });
   | |_________^
```

---

# VERY SAFE

```rust
let hrefs_occurrences = RefCell::new(HashMap::new());
```
```
error[E0277]: `std::cell::RefCell<std::collections::HashMap<std::string::String, i32>>` cannot be shared between threads safely
  --> src/main.rs:26:10
   |
26 |         .for_each(|file_path| {
   |          ^^^^^^^^ `std::cell::RefCell<std::collections::HashMap<std::string::String, i32>>` cannot be shared between threads safely
```

---

# Let's try a concurrent Map

## `chashmap`!

---

# Maybe it's hard...

```rust
use chashmap::CHashMap;
/* ... */
    let hrefs_occurrences = CHashMap::new();
    /* ... */
                match hrefs_occurrences.get_mut(href) {
                    Some(mut count) => *count += 1,
    /* ... */
```

## DONE!

---

# Do you want hardcore?

## Try to return early in case of error instead of `panic`ing

---

![100% center](code.svg)

---

# Thanks!
