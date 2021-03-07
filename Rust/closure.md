# Closures

- [Gist](#gist)
- [Verbose Version](#verbose-version)
- [Simplified Version](#simplified-version)
- [Returning Closures](#returning-closures)
- [Generic Closures](#generic-closures)

## Gist
- Closure is a lambda expression

## Verbose Version
Doing simple increments:
```rust
let inc = |num: i32| -> i32 { num + 1 };
println!("{}", inc(3));         // prints "4"
```

This is equivalent to below in C++:
```cpp
auto inc = [](int num) { return num + 1; };
std::cout << inc(3) << '\n';    // prints "4"
```

## Simplified Version
Types can be ignored, curly brace can be ignored if only 1 line in body.
```rust
let inc = |num| num + 1;        // note: ; is for variable not for closure body
println!("{}", inc(3));         // prints "4"
```

## Returning Closures
Currently, closures can be returned in a `Box<T>`. Other methods might appear in newer versions of rust.
```rust
fn get_inc_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|num| num + 1)
}

println!("{}", get_inc_closure()(3));   // prints "4"
```

This is equivalent to:
```cpp
auto get_inc_closure() {
    return [](int num) { return num + 1; };
}

std::cout << get_inc_closure()(3) << '\n';  // prints "4"
```

## Generic Closures
Currently, there are no ways as of yet to create generic closures, so you can't do the below in rust:
```cpp
auto inc = [](const auto& num) { return num + 1; };
std::cout << inc(3.1) << '\n';     // prints "4.1"
```
