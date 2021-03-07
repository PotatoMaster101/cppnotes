# `String` and `str`

- [Gist](#gist)
- [`String`](#string)
- [`str`](#str)
- [As Function Parameter](#as-function-parameter)

## Gist
- `String` is like `std::string` in C++
- `str` is like `std::string_view` in C++
- Use `&str` as function parameter for const-ness and performance

## `String`
Think of `String` as `std::string` in C++, you can do basic string operations such as append and remove.
```rust
let mut some_string = String::from("Hello World!");
some_string += " Again!";   // "Hello World! Again!"
```

This is equivalent to below in C++:
```cpp
auto some_string = std::string{"Hello World!"};
some_string += " Again!";   // "Hello World! Again!"
```

## `str`
Think of `str` as `std::string_view` in C++, it is immutable and fixed size. Any hardcoded string in the program will be type `str` (or more specifically, `&'static str`).
```rust
let some_str = "Hello World!";
some_str += " Again!";      // compile error: cannot use `+=` on type `&str`
```

## As Function Parameter
It is preferred to take `&str` (string slice) as function parameters for safety and performance. The function cannot modify it:
```rust
fn string_func(some_string: &str) {
    // ...
}
```

If function needs to modify, take reference to mutable `String`:
```rust
fn string_func_modify(some_string: &mut String) {
    // ...
}
```

Borrowed `String` can also be passed to functions taking `&str`:
```rust
string_func(&String::from("Hello World!"));   // ok
```
