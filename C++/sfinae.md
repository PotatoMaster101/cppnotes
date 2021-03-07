# SFINAE

- [Gist](#gist)
- [Using SFINAE](#using-sfinae)
- [As Template Parameter](#as-template-parameter)
    - [Example](#example)
    - [Overloads](#overloads)
- [As Function Parameter](#as-function-parameter)
    - [Example](#example-1)
    - [Overloads](#overloads-1)
- [As Function Return Type](#as-function-return-type)
    - [Example](#example-2)
    - [Overloads](#overloads-2)
- [Class Template SFINAE](#class-template-sfinae)
    - [Example - Check for Numeric](#example---check-for-numeric)
    - [Example - Check for Iterable](#example---check-for-iterable)
    - [Example - Check for Container](#example---check-for-container)
- [Resources](#resources)

## Gist
**Substitution Failure is not an Error**. This is used to "enable" or "disable" templates at compile time. The usage is similar to tag dispatch and template specialisation.

## Using SFINAE
Normally, if you see `std::enable_if` or `std::void_t`, then SFINAE might be in play. SFINAE can appear in 4 places:
1. As template parameter
2. As function parameter
3. As function return type
4. In class template specialisation

## As Template Parameter
### Example
For example, only enable the function `foo` if the `T` is integral type:
```cpp
template <typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
void foo(T t) { std::cout << "integral\n"; }

foo(5);     // prints "integral"
foo(5.0);   // does not compile
foo("abc"); // does not compile
```
We used `std::enable_if_t` to only enable the function `foo` if `std::is_integral_v<T>` is satisfied. Note that we have used a nameless template parameter for `std::enable_if_t`. The code does not compile for `foo(5.0)` and `foo("abc")` since they are not integrals.

### Overloads
If in the future we want to have another function `foo` that is only enabled if `T` is floating points, we can try something like:
```cpp
template <typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
void foo(T t) { std::cout << "integral\n"; }

template <typename T, typename = std::enable_if_t<std::is_floating_point_v<T>>>
void foo(T t) { std::cout << "floating point\n"; }
```

However, this code **will not compile**, since C++ does not consider template parameters with different default values "unique". This error is equivalent to:
```cpp
// does not compile
template <std::size_t N = 0>
void foo() {}

template <std::size_t N = 1>
void foo() {}
```
Having same template parameter with different default values will not make `foo` "unique". To solve this issue, we can use SFINAE inside function parameter or return type.

## As Function Parameter
### Example
Using same example as previous:
```cpp
template <typename T>
void foo(T t, std::enable_if_t<std::is_integral_v<T>>* = nullptr) { std::cout << "integral\n"; }

foo(5);     // prints "integral"
foo(5.0);   // does not compile
foo("abc"); // does not compile
```
We have used `std::enable_if_t` in the function parameter this time, which will enable `foo` if `T` is integral. Notice that we have `* = nullptr` at the end, this is because by default `std::enable_if_t` will evaluate to `void` if the condition is true. Thus the expression `std::enable_if_t<std::is_integral_v<T>>* = nullptr` becomes `void* = nullptr` when `std::is_integral_v<T>` is satisfied. We can also do something like:
```cpp
template <typename T>
void foo(T t, std::enable_if_t<std::is_integral_v<T>, int> = 0) { std::cout << "integral\n"; }
```
This time it will evaluate to `int = 0`.

### Overloads
We now want to have another `foo` which only enables if `T` is floating point:
```cpp
template <typename T>
void foo(T t, std::enable_if_t<std::is_integral_v<T>>* = nullptr) { std::cout << "integral\n"; }

template <typename T>
void foo(T t, std::enable_if_t<std::is_floating_point_v<T>>* = nullptr) { std::cout << "floating point\n"; }

foo(5);     // prints "integral"
foo(5.0);   // prints "floating point"
foo("abc"); // does not compile
```

This time the code will compile, since `std::enable_if_t<std::is_integral_v<T>>` and `std::enable_if_t<std::is_floating_point_v<T>>` are different types. This is like having:
```cpp
void foo(const std::vector<int>& v) {}
void foo(const std::vector<float>& v) {}
```
Compiles because `std::vector<int>` and `std::vector<float>` are different types.

## As Function Return Type
### Example
Using same example as previous:
```cpp
template <typename T>
std::enable_if_t<std::is_integral_v<T>> foo(T t) { std::cout << "integral\n"; }

foo(5);     // prints "integral"
foo(5.0);   // does not compile
foo("abc"); // does not compile
```
This time we use `std::enable_if_t` in function return, but meaning is the same.

### Overloads
```cpp
template <typename T>
std::enable_if_t<std::is_integral_v<T>> foo(T t) { std::cout << "integral\n"; }

template <typename T>
std::enable_if_t<std::is_floating_point_v<T>> foo(T t) { std::cout << "floating point\n"; }

foo(5);     // prints "integral"
foo(5.0);   // prints "floating point"
foo("abc"); // does not compile
```
Works as expected.

## Class Template SFINAE
Similar to function templates, SFINAE can also work on class templates. Normally we use **partial specialisation** when playing with SFINAE and class templates.

### Example - Check for Numeric
We want to distinguish if `T` is integral or floating or not integral and floating.
```cpp
template <typename T, typename = void>
struct foo { foo() { std::cout << "default\n"; } };

template <typename T>
struct foo<T, typename std::enable_if_t<std::is_integral_v<T>>> { foo() { std::cout << "integral\n"; } };

template <typename T>
struct foo<T, typename std::enable_if_t<std::is_floating_point_v<T>>> { foo() { std::cout << "floating point\n"; } };
```

We have created 2 partial specialisations for `foo`, one accepts integral and another accepts floating point. We can test this:
```cpp
foo<int> a{};           // prints "integral"
foo<double> b{};        // prints "floating point"
foo<std::string> c{};   // prints "default"
```
`std::string` is not integral or floating point, compiler will fall-back to default.

### Example - Check for Iterable
We want to check if `T` has `begin()` and `end()` functions.
```cpp
template<typename T, typename = void>
struct can_iterate : std::false_type {};

template<typename T>
struct can_iterate<
    T,
    std::void_t<
        decltype(std::declval<T>().begin()),
        decltype(std::declval<T>().end())
    >
> : std::true_type {};
```

If `T` has `begin()` and `end()` (check using `std::void_t`), then inherit from `std::true_type`, else fall-back to `std::false_type`. We can test this by:
```cpp
std::cout << can_iterate<int>::value << '\n';               // prints "0"
std::cout << can_iterate<std::string>::value << '\n';       // prints "1"
std::cout << can_iterate<std::vector<int>>::value << '\n';  // prints "1"
```

### Example - Check for Container
We want to check if `T` is a container:
```cpp
template<typename T, typename = void>
struct is_container : std::false_type {};

template<typename T>
struct is_container<
    T,
    std::void_t<
        typename T::value_type,
        typename T::size_type,
        typename T::iterator,
        decltype(std::declval<T>().size()),
        decltype(std::declval<T>().begin()),
        decltype(std::declval<T>().end())
    >
> : std::true_type {};
```

Same as the previous example. Normally a container type will have member types `value_type`, `size_type` and `iterator`, and functions `size()`, `begin()` and `end()`. If all these members are present, then inherit from `std::true_type`, else fall-back to `std::false_type`.

## Resources
| What             | URL                                               |
| :--------------- | :------------------------------------------------ |
| `std::enable_if` | https://en.cppreference.com/w/cpp/types/enable_if |
| `std::void_t`    | https://en.cppreference.com/w/cpp/types/void_t    |
| SFINAE           | https://en.cppreference.com/w/cpp/language/sfinae |
