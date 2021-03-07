# Concepts

- [Gist](#gist)
- [Using Concepts](#using-concepts)
    - [Requires Clause](#requires-clause)
    - [Without Requires Clause](#without-requires-clause)
    - [Standard Concepts](#standard-concepts)
- [Custom Concepts](#custom-concepts)
    - [Creating Concepts](#creating-concepts)
    - [Chain Multiple Concepts](#chain-multiple-concepts)
- [Requires Expression](#requires-expression)
    - [Simple Requirements](#simple-requirements)
    - [Type Requirements](#type-requirements)
    - [Compound Requirements](#compound-requirements)
    - [Nested Requirements](#nested-requirements)
- [Anonymous Concepts](#anonymous-concepts)
- [Constrain `auto`](#constrain-auto)
- [Overloading](#overloading)
- [Compile Time Constants](#compile-time-constants)
- [Summary](#summary)
- [Resources](#resources)

## Gist
Concept is a way to constrain templates in C++20. Prior to C++20 there were a few ways to constrain templates:
- Using good names for templates such as `Num` instead of `T`, however it will not be checked by compiler
- Using SFINAE to select the right template overload
- Using `static_assert` to check template type info at compile time

Concept can work on both function and class templates.

## Using Concepts
### Requires Clause
To restrict a template, use the `requires` clause in the function header. For example:
```cpp
template <class T>
void foo(T arg) requires std::integral<T> {
    // ...
}
```
Compiler will perform a compile time check whether `T` is an integral type (i.e., `int`, `char` etc). If `T` is not an integral type, the code will not compile and a meaningful error message will be shown. The `requires` clause can also appear on the same line as the template:
```cpp
template <class T> requires std::integral<T>
void foo(T arg) {
    // ...
}
```

### Without Requires Clause
Instead of using `requires` clause, the concept can also directly go into the template itself:
```cpp
template <std::integral T>
void foo(T arg) {
    // ...
}
```
Note that in this case the template argument for `std::integral` has been deduced. If the concept requires multiple template arguments, then it needs to use the `requires` clause, for example:
```cpp
// assume concept bar takes 3 template arguments
template <class T, class U> requires bar<T, U, int>
void foo(T arg1, U arg2) {
    // ...
}
```

### Standard Concepts
The list of standard concepts can be found [here](https://en.cppreference.com/w/cpp/concepts). They are located in 3 headers:
```cpp
#include <concepts>
#include <iterators>
#include <ranges>
```

## Custom Concepts
### Creating Concepts
Custom concepts can be created:
```cpp
template <class T>
concept int_collection = std::integral<typename T::value_type>;
```
The above code creates a new `concept` named `int_collection`. `T` must have a `value_type` member and that member must be an integral type. Below is an example on using `int_collection`:
```cpp
template <int_collection T> void foo(T t) {}

foo(123);                           // ERROR: no value_type member
foo("abc");                         // ERROR: no value_type member
foo(std::vector{1, 2, 3});          // OK
foo(std::forward_list{1, 2, 3});    // OK
foo(std::string{"abc"});            // OK
foo(std::vector{"1", "2", "3"});    // ERROR: value_type not integral
```

### Chain Multiple Concepts
A `concept` can also chain multiple different `concept`s:
```cpp
template <class T>
concept int_collection_contiguous = std::integral<typename T::value_type> && std::contiguous_iterator<typename T::iterator>;
```
This is similar to the previous example, but a new constraint has been added: `std::contiguous_iterator<typename T::iterator>`. This whole concept means that `T` must have a member `value_type` that is type integral, and `T` must have a member named `iterator` that satisfies `std::contiguous_iterator`. The `&&` in the concept means short circuit **AND**, `||` can also be used to mean **OR**. Below is an example to use `int_collection_contiguous` concept:
```cpp
template <int_collection_contiguous T> void foo(T t) {}

foo(123);                           // ERROR: no value_type member and no iterator member
foo("abc");                         // ERROR: no value_type member and no iterator member
foo(std::vector{1, 2, 3});          // OK
foo(std::list{1, 2, 3});            // ERROR: not contiguous
foo(std::string{"abc"});            // OK
foo(std::vector{"1", "2", "3"});    // ERROR: value_type not integral
```

## Requires Expression
### Simple Requirements
The below example checks whether type `T` can be added to another type `T`:
```cpp
template <class T>
concept can_add = requires (T a, T b) { a + b; };
```
If `T` can be added, then the requirement will be met. Some examples to test this:
```cpp
template <can_add T> void foo(T t) {}

foo(123);                   // OK
foo(123.0);                 // OK
foo(std::string{"abc"});    // OK
foo("abc");                 // ERROR: char* can't be added to char*
```

### Type Requirements
The below example checks whether `T` has the specified member types:
```cpp
template <class T>
concept collection = requires {
    typename T::value_type;
    typename T::size_type;
    typename T::iterator;
};
```
If `T` has all 3 `value_type`, `size_type`, and `iterator`, then the requirement will be met. Some examples to test this:
```cpp
template <collection T> void foo(T t) {}

foo(std::vector{1, 2, 3});      // OK
foo(std::list{1, 2, 3});        // OK
foo(std::string{"abc"});        // OK
foo(123);                       // ERROR: int doesn't have all 3 members
```

### Compound Requirements
This looks very similar to simple requirements. The below example checks whether `T` can be multiplied and result type can be converted to `int`:
```cpp
template <class T>
concept mul_to_int = requires (T a, T b) {
    {a * b} noexcept -> std::convertible_to<int>;
};
```
With the `noexcept` keyword present, it means the multiplication does not throw. Some examples to test this:
```cpp
template <mul_to_int T> void foo(T t) {}

foo(123);       // OK
foo(123.0);     // OK
foo("abc");     // ERROR: can't multiply and cast result to int
```

### Nested Requirements
The below example also checks whether `T` can be multiplied and result type can be concerted to `int`:
```cpp
template <class T>
concept mul_to_int = requires (T a, T b) {
    requires std::convertible_to<int, decltype(a * b)>;
};
```

## Anonymous Concepts
Anonymous concepts can be created using `requires` twice:
```cpp
template <class T> requires requires (T a, T b) { a + b; }
auto foo(T a, T b) { return a + b; }
```
The first `requires` starts the `requires` clause, then the `requires (T a, T b) { a + b; }` is an anonymous concept. This is equivalent to:
```cpp
template <class T>
concept can_add = requires (T a, T b) { a + b; };

template <class T> requires can_add<T>
auto foo(T a, T b) { return a + b; }
```
However, anonymous concepts cannot be used anywhere else, but `can_add` concept can be used somewhere else.

## Constrain `auto`
Like templates, `auto` can also be constrained. This can be used to make code more readable and find subtle bugs:
```cpp
std::integral auto x = 1;               // OK
std::integral auto x = 1.0;             // ERROR: x is floating point
std::unsigned_integral auto x = -1;     // ERROR: x is signed
std::floating_point auto x = 1;         // ERROR: x is integral
std::floating_point auto x = 1.0;       // OK
std::floating_point auto x = foo();     // OK or ERROR depends on what foo returns
```

Works with functions too:
```cpp
std::integral auto foo(std::integral auto a, std::integral auto b) { return a + b; }
```
This is the same as when using `std::integral` on a template.

## Overloading
Similar to SFINAE, compiler will choose the most appropriate function to call when using function overloading:
```cpp
template <class T, class U>
void foo(T, U) { std::cout << "normal\n"; }

template <class T, class U> requires std::predicate<T, U>
void foo(T, U) { std::cout << "predicate\n"; }

template <class T, class U> requires std::invocable<T, U>
void foo(T, U) { std::cout << "invocable\n"; }

foo([](int num){ return num > 0; }, 1);                     // prints "predicate"
foo([](int num){ return std::to_string(num); }, 1);         // prints "invocable" - returned string is not a predicate
foo([](int num){ return std::to_string(num); }, "");        // prints "normal" - can't evaluate to predicate or invocable
```

## Compile Time Constants
Concepts can also be used to generate compile time constants:
```cpp
template <class T>
concept can_add = requires (T a, T b) { a + b; };

constexpr auto add1 = can_add<int>;                 // true
constexpr auto add2 = can_add<float>;               // true
constexpr auto add3 = can_add<std::string>;         // true
constexpr auto add4 = can_add<std::vector<int>>;    // false
```

## Summary
Use concepts to:
- Constrain templates
- Improve code readability (giving clear interface without having to look into function body or documentation)
- Generate better error messages

## Resources
| What                     | URL                                                    |
| :----------------------- | :----------------------------------------------------- |
| Standard Concepts        | https://en.cppreference.com/w/cpp/concepts             |
| Constraints and Concepts | https://en.cppreference.com/w/cpp/language/constraints |
