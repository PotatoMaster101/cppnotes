# Forwarding Reference

- [Gist](#gist)
- [Reference](#reference)
- [Forwarding Reference](#forwarding-reference)
- [`std::move` vs `std::forward`](#stdmove-vs-stdforward)
- [Reference Collapsing](#reference-collapsing)
- [Using `auto&&`](#using-auto)
- [Tips](#tips)
    - [Don't "Wrap" `T`](#dont-wrap-t)
    - [Make Perfect Forwarding Constructors Function Template](#make-perfect-forwarding-constructors-function-template)
    - [Don't Use Perfect Forwarding Constructors Everywhere](#dont-use-perfect-forwarding-constructors-everywhere)
- [Summary](#summary)
- [Resources](#resources)

## Gist
We want to forward parameters perfectly to functions which handle lvalue and rvalue differently, such that lvalue will be forwarded to lvalue version, and rvalue will be forwarded to rvalue version.

## Reference
There are lvalue and rvalue references:
```cpp
void foo(int& lvalue) { std::cout << "foo(lvalue)\n"; }
void foo(int&& rvalue) { std::cout << "foo(rvalue)\n"; }

int i = 0;
foo(i);             // prints "foo(lvalue)"
foo(10);            // prints "foo(rvalue)"
```

Inside a function, any parameter is an lvalue, even if it is passed as rvalue. Now if we "forward" this parameter to some other functions that handle lvalue and rvalue differently, it will always be forwarded to the lvalue version.
```cpp
void foo(int& lvalue) { std::cout << "foo(lvalue)\n"; }
void foo(int&& rvalue) { std::cout << "foo(rvalue)\n"; }

void bar(int&& param) {
    // `param` is lvalue now
    foo(param);     // this will always print "foo(lvalue)", even though `param` is rvalue reference
}

bar(10);        // prints "foo(lvalue)"
```

## Forwarding Reference
A forwarding reference is used to forward the parameter appropriately, if it is lvalue if will be forwarded to lvalue version, if it is rvalue it will be forwarded to rvalue version. This is done using `std::forward`:
```cpp
void foo(int& lvalue) { std::cout << "foo(lvalue)\n"; }
void foo(int&& rvalue) { std::cout << "foo(rvalue)\n"; }

template <class T>
void bar(T&& param) { foo(std::forward<T>(param)); }

int i = 0;
bar(i);         // prints "foo(lvalue)"
bar(10);        // prints "foo(rvalue)", which is handled correctly now
```

**Note**: the `T&&` is a forwarding reference, not an rvalue reference. When `&&` is after a template type, it is a forwarding reference.

## `std::move` vs `std::forward`
`std::move` will always cast to rvalue no matter what, while `std::forward` will only cast to rvalue if the parameter truely is an rvalue. If you only want rvalue, then use `std::move`. If you want to handle both lvalue and rvalue, use `std::forward`.

## Reference Collapsing
`std::forward` works by using reference collapsing, which is performed when compiler sees a double reference (i.e., a reference to a reference). Double reference in code will lead to compile error, however compiler can generate them when substituting templates. Rules of reference collapsing:

| First Reference | Second Reference | Result Reference | Example               |
| :-------------- | :--------------- | :--------------- | :-------------------- |
| lvalue          | lvalue           | lvalue           | `int& &` -> `int&`    |
| lvalue          | rvalue           | lvalue           | `int& &&` -> `int&`   |
| rvalue          | lvalue           | lvalue           | `int&& &` -> `int&`   |
| rvalue          | rvalue           | rvalue           | `int&& &&` -> `int&&` |

A simple example of how compiler might perform reference collapsing:
```cpp
template <class T>
void bar(T&& param) {
    std::forward<T>(param);
}
```

When calling `bar` with an lvalue reference (such as `int&`), compiler will perform substitutes and reference collapsing:
```cpp
template <class T>
void bar(int& && param) {       // <-- before reference collapse
    std::forward<int&>(param);
}
```
```cpp
template <class T>
void bar(int& param) {          // <-- after reference collapse
    std::forward<int&>(param);
}
```

When calling `bar` with an rvalue reference (such as `int&&`):
```cpp
template <class T>
void bar(int&& && param) {      // <-- before reference collapse
    std::forward<int&&>(param);
}
```
```cpp
template <class T>
void bar(int&& param) {         // <-- after reference collapse
    std::forward<int&&>(param);
}
```

When calling `bar` with no reference (such as `int`):
```cpp
template <class T>
void bar(int&& param) {         // <-- no need for reference collapse
    std::forward<int>(param);
}
```

## Using `auto&&`
When type is `auto&&`, it can also be a forwarding reference:
```cpp
auto&& value = foo();       // `value` might be lvalue or rvalue
```

`std::forward` can also work with `auto&&`, just use `decltype` for the template parameter:
```cpp
std::forward<decltype(value)>(value);
```

Similarly, for lambda with parameter as `auto&&` it will also work:
```cpp
[](auto&& param) { std::forward<decltype(param)>(param); }
```

## Tips
### Don't "Wrap" `T`
Don't place `T` in some other templates. Below is not perfect forwarding:
```cpp
template <class T>
void foo(std::vector<T>&& vec) { ... }
```
It is only taking a `std::vector` by rvalue reference. For perfect forwarding to work, it must be `T&&` (or whatever template parameter name).

### Make Perfect Forwarding Constructors Function Template
For perfect forwarding constructors, make them function templates even if they are already in class template.

**Bad**:
```cpp
template <class T>
struct foo {
    T stuff;

    foo(T&& data)
        : stuff{std::forward<T>(data)} { }
};
```

**Good**:
```cpp
template <class T>
struct foo {
    T stuff;

    template <class U>
    foo(U&& data)
        : stuff{std::forward<U>(data)} { }
};
```

### Don't Use Perfect Forwarding Constructors Everywhere
Perfect forwarding constructors are greedy, they will "hide" other constructors (such as `const T&`) in most cases:
```cpp
template <class T>
struct foo {
    T stuff;

    foo(const T& data)
        : stuff{data} { std::cout << "const T&\n"; }

    template <class U>
    foo(U&& data)
        : stuff{std::forward<U>(data)} { std::cout << "U&&\n"; }
};

int x = 10;
int& y = x;
const int z = 20;

foo<int> a{x};      // prints "U&&"
foo<int> b{y};      // prints "U&&"
foo<int> c{z};      // prints "const T&"
foo<int> d{0};      // prints "U&&"
```

## Summary
- Use `std::move` always cast to rvalue
- Use `std::forward` to forward parameters while keeping their lvalue or rvalue status
- When using `std::forward` with `auto&&`, use `decltype` to retrieve `T`

## Resources
| What               | URL                                                                                                              |
| :----------------- | :--------------------------------------------------------------------------------------------------------------- |
| Reference          | https://en.cppreference.com/w/cpp/language/reference                                                             |
| Reference Collapse | https://stackoverflow.com/questions/13725747/concise-explanation-of-reference-collapsing-rules-requested-1-a-a-2 |
| `std::forward`     | https://en.cppreference.com/w/cpp/utility/forward                                                                |
