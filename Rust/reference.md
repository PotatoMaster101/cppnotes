# Reference

- [Gist](#gist)
- [Immutable Reference](#immutable-reference)
- [Mutable Reference](#mutable-reference)
- [Lifetime](#lifetime)

## Gist
- Reference is like references and pointers in C++
- Use `&` for immutable reference, use `&mut` for mutable reference

## Immutable Reference
A reference in Rust is similar to a pointer in C++. Use `&` to create immutable reference and use `*` to dereference:
```rust
let value = 100;
let reference = &value;
```

This is equivalent to below in C++:
```cpp
const auto value = 100;
auto &reference = value;
```

## Mutable Reference
Use `&mut` to create mutable reference. Mutable reference can only refer to mutable values:
```rust
let mut value = 100;
let reference = &mut value;
*reference = 50;        // value == 50
```

You can't create a mutable reference if an immutable reference already exist:
```rust
let mut value = 100;
let immutable_reference = &value;
let mutable_reference = &mut value;
println!("{}", immutable_reference);    // compile error: cannot borrow `value` as mutable because it is also borrowed as immutable
```

You can't modify a mutable reference if it is borrowed in another scope:
```rust
let mut value = 100;

// new scope
{
    let reference = &value;
    value = 200;
    println!("{}", reference);          // compile error: cannot assign to `value` because it is borrowed
}
```

## Lifetime
Rust won't allow reference to live longer than the value:
```rust
let value = 100;
let mut reference = &value;

// new scope
{
    let new_value = 200;
    reference = &new_value;    // dangles after scope ends
}
println!("{}", reference);          // compile error: borrowed value does not live long enough
```
