# Rust-Torino 20181025

## Bechmarking and Testing in Rust
- testing with `libtest`
- benchmarking with `criterion`
---
# Writing tests

[libtest](https://doc.rust-lang.org/book/second-edition/ch11-00-testing.html) is built-in in `rust` itself, in the future it will be [pluggable](https://github.com/rust-lang/rust/issues/50297)

- The integration tests live in `./tests`
```
$ cat tests/bar.rs
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}
```
- the unit-tests can be embedded in the source code. 
```
$ cat src/lib.rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```
---
# Running tests

Cargo has a built in command: `cargo test`
``` sh
$ cargo test -h
```
``` sh
$ cargo test -- --help
```
``` sh
$ cargo test -- --list
```
``` sh
$ cargo test -- --nocapture
```

---

# Cargo test
`cargo test` will build a test harness out of the source code and execute it, then build the integration tests and execute them.
``` sh
$ cargo test
```
```
   Compiling testcase-benchmark v0.1.0 (/private/tmp/testcase-benchmark)
    Finished dev [unoptimized + debuginfo] target(s) in 0.88s
     Running target/debug/deps/testcase_benchmark-57fdf903d343f894

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/bar-db45e2a4953e8a30

running 1 test
test it_works ... ok
...
```

---
# Benchmarks
Currently **libbench** is only available in *nightly* and it is going to be overhauled soon.

[Criterion](https://github.com/japaric/criterion.rs) is a separate crate that provides even richer benchmarking capabilities and it integrates nicely in cargo, the only caveat is that you cannot have it run `#[bench]`-marked code from the source and you have to write more code in `./benches`.

---

# Setting up for criterion
## Cargo.toml

``` toml
[lib]
bench = false

[[bench]]
name = "bench"
harness = false

[dev-dependencies]
criterion = "0.2"
```
You have to disable the possibly-built-in libbench and then add the crate.

---

# Writing benchmarks

``` rust
#[macro_use]
extern crate criterion;

fn bench_fn(b: &mut Bencher) {
   b.iter(|| {...})
}

fn some(c: &mut Criterion) {
	c.bench_function("name", |b| bench_fn(b))
}

criterion_group!(group_name, some, tests);
criterion_group!(another_group_name, other);

criterion_main!(group_name, another_group_name);
```
---
# Running benchmarks

Cargo has a built in command: `cargo bench`

``` sh
$ cargo bench -h
```
``` sh
$ cargo bench -- --help
```
``` sh
$ cargo bench -- --list
```
``` sh
$ cargo bench -- --nocapture
```
It works pretty much as `cargo test` does, the only caveat is that some options may be missing (e.g. `--list` had been added just recently)

---

# Comparing benchmarks

[critcmp](https://github.com/BurntSushi/critcmp) is a tiny utility to compare different runs and save the data for further processing or `cargo clean`-impervious storage.

``` sh
$ cargo bench -- --save-baseline before
$ cargo bench -- --save-baseline change

$ critcmp before change
```
``` sh
$ critcmp --export before > before.json
$ critcmp --export after > after.json

$ critcmp before.json after.json
```
``` sh
$ critcmp --baselines
```

---