# `Result<T, E>`

## Gist
- `Result<T, E>` is like `std::optional<T>` in C++ (but instead of having no value, have error instead)
- `Ok<T>` means good, `Err<E>` means bad
- Get value with `match` or use `?`

## `Ok<T>` and `Err<E>`
`Result<T, E>` can be expressed as `Ok<T>` or `Err<E>`:
```rust
let good_value = Ok("no error");
let bad_value = Err("has error");
```

## Using `match`
Use `match` to determine if a `Result<T, E>` is good or bad:
```rust
fn match_result(res: Result<&str, &str>) {
    match res {
        Ok(good) => println!("good value: {}", good),
        Err(bad) => println!("bad value: {}", bad),
    }
}

fn main() {
    match_result(Ok("no error"));       // prints "good value: no error"
    match_result(Err("has error"));     // prints "bad value: has error"
}
```

## Using `?`
`?` is a syntax sugar for `match Result<T, E>`. `?` means "if error then return error, else continue".

Code with `match`:
```rust
fn foo() -> Result<&'static str, &'static str> {
    let res = get_some_result();
    let value = match res {
        Ok(good) => good,
        Err(bad) => return Err(bad),
    };

    // do stuff with `value`
    Ok(value)
}
```

Code with `?`:
```rust
fn foo() -> Result<&'static str, &'static str> {
    let value = get_some_result()?;

    // do stuff with `value`
    Ok(value)
}
```
