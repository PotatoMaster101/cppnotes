# `Box<T>`

- [Gist](#gist)
- [What is `Box<T>`](#what-is-boxt)
- [Dereference a `Box<T>`](#dereference-a-boxt)
- [Auto Dereference for Functions](#auto-dereference-for-functions)

## Gist
- `Box<T>` is like `std::unique_ptr<T>` in C++
- Use `.` instead of `->`

## What is `Box<T>`
Think of `Box<T>` as `std::unique_ptr<T>`:
```rust
let num = Box::new(100);        // with type: `let num = Box::<isize>::new(100);`
```

This is equivalent to below in C++:
```cpp
auto num = std::make_unique<int>(100);
```

## Dereference a `Box<T>`
Use `*` to dereference a `Box<T>`, same as C++:
```rust
let num = Box::new(Box::new(Box::new(Box::new(Box::new(100)))));
println!("{}", *****num + 1);   // prints "101"
println!("{}", num + 1);        // compile error: cannot add `{integer}` to `Box<Box<Box<Box<Box<{integer}>>>>>`
```

## Auto Dereference for Functions
When calling a function from the type in `Box<T>`, Rust will auto dereference:
```rust
let mut some_string = Box::new(String::from("Hello World!"));
some_string.push_str(" Again!");
```

This is equivalent to below in C++:
```cpp
auto some_string = std::make_unique<std::string>("Hello World!");
some_string->append(" Again!");
```

However, there is no `->` in Rust, use `.` instead.
