# Range For Initializer

- [Gist](#gist)
- [Without Range For Initializer](#without-range-for-initializer)
- [With Range For Initializer](#with-range-for-initializer)

## Gist
Allowing you to initialize a variable (such as a container) inside a range for itself.

## Without Range For Initializer
```cpp
auto nums = get_nums();
for (auto n : nums) {
    // ...
}
```

## With Range For Initializer
```cpp
for (auto nums = get_nums(); auto n : nums) {
    // ...
}
```

The initializer can be anything:
```cpp
auto nums = get_nums();
for (std::size_t i = 0; auto n : nums) {
    // ...
    i++;
}
```
