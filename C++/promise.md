# Future and Promise

- [Gist](#gist)
- [Using `std::future`](#using-stdfuture)
- [Using `std::promise`](#using-stdpromise)
- [Using `std::shared_future`](#using-stdshared_future)
- [Summary](#summary)
- [Resources](#resources)

## Gist
We want to have easier asynchronous operations (such as getting the return value of a thread) without using mutexes and conditional variables.

## Using `std::future`
`std::future` will store the result of an asynchronous operation. A typical example is to get a child thread's function return value:
```cpp
int foo(int x) {
    // do lots of work
    std::this_thread::sleep_for(3s);
    return x + 1;
}

int main() {
    auto fut = std::async(foo, 5);
    std::cout << fut.get() << '\n';     // prints "6" after 3 seconds
    return 0;
}
```
We call `std::async` to invoke `foo` and pass in a value of `5`. A future of `int` will be returned, which we will get the value via `get`. `get` will wait and return until the future value is ready, however it can only be called once on a `std::future`:
```cpp
auto fut = std::async(foo, 5);
std::cout << fut.get() << '\n';
std::cout << fut.get() << '\n';     // calling more than once will throw std::future_error
```

## Using `std::promise`
`std::promise` is used to store a value which can be later acquired by a `std::future`. A typical example is main thread wants to pass a future value to a child thread:
```cpp
int foo(std::future<int>& x) {
    int val = x.get();
    // do lots of work
    std::this_thread::sleep_for(3s);
    return val + 1;
}

int main() {
    std::promise<int> p;
    auto param = p.get_future();
    auto ret = std::async(foo, std::ref(param));

    int future_val = 5;
    // do lots of work
    std::this_thread::sleep_for(2s);
    p.set_value(future_val);
    std::cout << ret.get() << '\n';     // prints "6" after 5 seconds
    return 0;
}
```
In the main thread we call `std::async` to invoke `foo`, however the value of `x` is not known at this point of time (hence the type of `x` is `std::future`). We call `get_future` and store it in `param`, and pass it to `foo`.

When `foo` runs, it will wait on the line `int val = x.get();` since the value is not ready.

After main thread does lots of work and computed this future value (stored in `future_val`), we call `set_value` to set the future value for `param`. Once future value is set, `foo` will be awoken.

If a `std::promise` does not call `set_value`, then an exception will be thrown:
```cpp
int main() {
    std::promise<int> p;
    auto param = p.get_future();
    auto ret = std::async(foo, std::ref(param));
    // forgot to send a value
    return 0;   // throws std::future_error
}
```

## Using `std::shared_future`
Sometimes we want a function to be called asynchronously lots of times:
```cpp
std::promise<int> p;
auto param = p.get_future();

auto ret1 = std::async(foo, std::ref(param));
auto ret2 = std::async(foo, std::ref(param));   // throws std::future_error
auto ret3 = std::async(foo, std::ref(param));   // throws std::future_error
auto ret4 = std::async(foo, std::ref(param));   // throws std::future_error
auto ret5 = std::async(foo, std::ref(param));   // throws std::future_error
```
However we cannot do this, since `std::future`'s `get` can only be called once. In this situation we can use `shared_future` which allows `get` to be called multiple times:
```cpp
int foo(std::shared_future<int> x) {
    int val = x.get();
    // do lots of work
    std::this_thread::sleep_for(3s);
    return val + 1;
}

int main() {
    std::promise<int> p;
    auto param = p.get_future();
    auto shared = param.share();
    auto ret1 = std::async(foo, shared);
    auto ret2 = std::async(foo, shared);
    auto ret3 = std::async(foo, shared);
    auto ret4 = std::async(foo, shared);
    auto ret5 = std::async(foo, shared);

    int future_val = 5;
    // do lots of work
    p.set_value(future_val);
    return 0;
}
```
In `foo` we change the parameter type to `std::shared_future` (which can be copied). Calling `share` on a `std::future` will return the `std::shared_future` for that future. Now we can invoke `foo` lots of times using the same future value.

## Summary
- `std::future` is used to store a value that will be known in the future
- `std::promise` is used to "promise" the code that a value will be available in the future
- Use `std::future` and `std::promise` to simplify asynchronous operations

## Resources
| What           | URL                                              |
| :------------- | :----------------------------------------------- |
| `std::future`  | https://en.cppreference.com/w/cpp/thread/future  |
| `std::promise` | https://en.cppreference.com/w/cpp/thread/promise |
