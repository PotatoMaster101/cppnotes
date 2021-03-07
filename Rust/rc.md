# `Rc<T>`

- [Gist](#gist)
- [What is `Rc<T>`](#what-is-rct)
- [`Rc::Clone`](#rcclone)

## Gist
- `Rc<T>` is like `std::shared_ptr<T>` in C++
- Use `Rc::Clone` to create a new `Rc<T>` pointing to existing data
- Only use in single-threaded situations

## What is `Rc<T>`
Think of `Rc<T>` as `std::shared_ptr<T>` in C++:
```rust
let num = Rc::new(100);
```

This is equivalent to below in C++:
```cpp
auto num = std::make_shared<int>(100);
```

## `Rc::Clone`
Use `Rc::Clone` to create another `Rc<T>` pointing to an existing `Rc<T>`:
```rust
let num = Rc::new(100);
let num_copy1 = Rc::clone(&num);
let num_copy2 = Rc::clone(&num);
println!("{}", Rc::strong_count(&num));     // prints "3"
```

This is equivalent to below in C++:
```cpp
auto num = std::make_shared<int>(100);
auto num_copy1{num};
auto num_copy2{num};
std::cout << num.use_count() << std::endl;  // prints "3"
```
