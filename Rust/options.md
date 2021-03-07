# `Option<T>`

- [Gist](#gist)
- [`Some<T>` and `None`](#somet-and-none)
- [Using `match`](#using-match)
- [Using `unwrap`](#using-unwrap)
- [Using `expect`](#using-expect)

## Gist
- `Option<T>` is like `std::optional<T>` in C++
- `Some<T>` means has value, `None` means no value
- Get value with `match` or call `unwrap`/`expect` (`panic` if no value)

## `Some<T>` and `None`
`Option<T>` can be expressed as `Some<T>` or `None`:
```rust
let value = Some("some value");
let no_value = None;
```

## Using `match`
Determine if `Some<T>` or `None` using `match`:
```rust
fn match_option(opt: Option<&str>) {
    match opt {
        Some("specific value") => println!("specific value"),
        Some(other_value) => println!("other value: {}", other_value),
        None => println!("nothing"),
    }
}

fn main() {
    match_option(Some("specific value"));      // prints "specific value"
    match_option(Some("some other value"));    // prints "other value: some other value"
    match_option(None);                        // prints "nothing"
}
```

## Using `unwrap`
Get value using `unwrap` (will panic if `None`):
```rust
fn unwrap_option(opt: Option<&str>) {
    println!("{}", opt.unwrap());
}

fn main() {
    unwrap_option(Some("specific value"));      // prints "specific value"
    unwrap_option(Some("some other value"));    // prints "some other value"
    unwrap_option(None);                        // crash: thread 'main' panicked at 'called `Option::unwrap()` on a `None` value'
}
```

## Using `expect`
Get value using `expect` (same as `unwrap`, but can provide custom panic message):
```rust
fn expect_option(opt: Option<&str>) {
    println!("{}", opt.expect("oh no, opt is none"));
}

fn main() {
    expect_option(Some("specific value"));      // prints "specific value"
    expect_option(Some("some other value"));    // prints "some other value"
    expect_option(None);                        // crash: thread 'main' panicked at 'oh no, opt is none'
}
```
