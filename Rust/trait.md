# `trait`

- [Gist](#gist)
- [Basic Usage](#basic-usage)
- [Override Functions](#override-functions)
- [Static Functions and `Self`](#static-functions-and-self)
- [`dyn` Keyword](#dyn-keyword)

## Gist
- `trait` is an abstract class
- Static functions use `Self` to refer to implementor type
- Use `dyn` to mean a type is `trait`

## Basic Usage
`trait` is like an abstract class in C++, you can put functions, virtual functions and pure virtual functions in it:
```rust
// define trait `Foo`
trait Foo {
    fn do_work(&self) -> &str;              // "abstract" function
    fn do_more_work(&self) -> &str;         // "abstract" function
    fn do_even_more_work(&self) -> &str {   // normal function
        "doing even more work"
    }
}

// implement `Foo` for `Bar`
struct Bar {}
impl Foo for Bar {
    fn do_work(&self) -> &str {
        "doing work"
    }

    fn do_more_work(&self) -> &str {
        "doing more work"
    }
}

let b = Bar{};
println!("{}", b.do_work());            // prints "doing work"
println!("{}", b.do_more_work());       // prints "doing more work"
println!("{}", b.do_even_more_work());  // prints "doing even more work"
```

## Override Functions
This is like virtual functions in C++:
```rust
// define trait `Foo`
trait Foo {
    fn override_me() {
        println!("function in Foo");
    }
}

// implement `Foo` for `Bar`
struct Bar {}
impl Foo for Bar {}

// implement `Foo` for `Baz`
struct Baz {}
impl Foo for Baz {
    fn override_me() {
        println!("function in Baz");
    }
}

Bar::override_me();     // prints "function in Foo"
Baz::override_me();     // prints "function in Baz"
```

## Static Functions and `Self`
Traits can have static methods too, if refering to implementor type, use `Self`:
```rust
// define trait `Foo`
trait Foo {
    fn create_foo(name: &'static str) -> Self;
    fn do_work(&self) -> String;
}

// implement `Foo` for `Bar`
struct Bar { name: &'static str }
impl Foo for Bar {
    fn create_foo(name: &'static str) -> Bar {
        Bar { name: name }
    }

    fn do_work(&self) -> String {
        format!("{} doing work", self.name)
    }
}

let b = Bar::create_foo("bar1");
println!("{}", b.do_work());        // prints "bar1 doing work"
```

## `dyn` Keyword
Sometimes a type can be ambiguous: `&Foo` can be a `struct` or a `trait`. Use `dyn` keyword to mean a `trait`:
```rust
fn do_work(f: &dyn Foo) {
    // ...
}
```
`f` must be a `trait`.
