# Tag Dispatch

- [Gist](#gist)
- [Using Tag Dispatch](#using-tag-dispatch)
    - [Example - Check for Integral](#example---check-for-integral)
    - [Example - Type of Iterator](#example---type-of-iterator)
- [Custom Tags](#custom-tags)

## Gist
Similar to template specialisation, but uses tags instead.

## Using Tag Dispatch
There are some standard tags declared in `<type_traits>` header, for iterators they are in `<iterator>`.

### Example - Check for Integral
We want our function templates to distinguish between integers and non-integers.
```cpp
template <class T>
void foo(T t, std::true_type) { std::cout << "integral\n"; }

template <class T>
void foo(T t, std::false_type) { std::cout << "not integral\n"; }

template <class T>
void foo(T t) { foo(t, std::conditional_t<std::is_integral_v<T>, std::true_type, std::false_type>{}); }
```

We use `std::conditional_t` to output `std::true_type` or `std::false_type` depending on type `T`, which is then passed to the correct template overload. Note that the function parameter for tag does not require a name. We can test this by:
```cpp
foo(1);         // prints "integral"
foo(1.0);       // prints "not integral"
foo("abc");     // prints "not integral"
```

### Example - Type of Iterator
We want our function template to distinguish between different types of iterators.
```cpp
template <class Iter>
void foo(Iter beg, Iter end, std::forward_iterator_tag) { std::cout << "forward\n"; }

template <class Iter>
void foo(Iter beg, Iter end, std::bidirectional_iterator_tag) { std::cout << "bidirectional\n"; }

template <class Iter>
void foo(Iter beg, Iter end, std::random_access_iterator_tag) { std::cout << "random access\n"; }

template <class Iter>
void foo(Iter beg, Iter end) { foo(beg, end, typename std::iterator_traits<Iter>::iterator_category{}); }
```

We use member type `iterator_category` to determine the type of the iterator and pass it to the correct template overload. We can test this:
```cpp
std::vector<int> v;
foo(v.begin(), v.end());        // prints "random access"

std::list<int> l;
foo(l.begin(), l.end());        // prints "bidirectional"

std::forward_list<int> fl;
foo(fl.begin(), fl.end());      // prints "forward"

std::string str;
foo(str.begin(), str.end());    // prints "random access"
```

## Custom Tags
Normally a tag is just an empty class/struct:
```cpp
struct tag1 {};
struct tag2 {};
struct tag3 {};

void foo(tag1) { std::cout << "tag1 foo\n"; }
void foo(tag2) { std::cout << "tag2 foo\n"; }
void foo(tag3) { std::cout << "tag3 foo\n"; }
```

Now we can use different tags to invoke different version of `foo`:
```cpp
foo(tag1{});        // prints "tag1 foo"
foo(tag2{});        // prints "tag2 foo"
foo(tag3{});        // prints "tag3 foo"
```
