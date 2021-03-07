# Using Enum

- [Gist](#gist)
- [Without `using`](#without-using)
- [With `using`](#with-using)

## Gist
When using the `using` keyword on enums, the enum class does not need to be present when using the enum values. This can lead to cleaner code.

## Without `using`
Prior to C++20:
```cpp
enum class foo {
    val1,
    val2
};
```

Now, to access values in `foo`, `foo::` needs to be typed in the front:
```cpp
void bar(foo f) {}

bar(foo::val1);     // OK
bar(foo::val2);     // OK
bar(val1);          // ERROR
bar(val2);          // ERROR
```

## With `using`
By using `using`, `foo::` can be omitted:
```cpp
void bar(foo f) {}

using enum foo;
bar(val1);          // OK
bar(val2);          // OK
bar(foo::val1);     // OK, but redundant foo::
bar(foo::val2);     // OK, but redundant foo::
```

This can be used inside any code blocks, for example:

**Classes and Structs**:
```cpp
class bar {
    using enum foo;

    foo member1 = val1;         // OK
    foo member2 = foo::val2;    // OK, but redundant foo::
};
```

**Functions**:
```cpp
void bar(foo f) {
    using enum foo;

    switch (f) {
        case val1:          // OK
            // ...
        case foo::val2:     // OK, but redundant foo::
            // ...
    }
}
```

**Namespaces**:
``` cpp
namespace bar {
    using enum foo;
};

auto member1 = bar::val1;   // OK
```
