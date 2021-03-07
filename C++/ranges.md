# Ranges

- [Gist](#gist)
- [Using Ranges](#using-ranges)
    - [Example - Sorting](#example---sorting)
- [Chaining Ranges](#chaining-ranges)
    - [Example - Remove Starting Elements by Predicate](#example---remove-starting-elements-by-predicate)
    - [Example - Filter Even Numbers](#example---filter-even-numbers)
    - [Example - Multiply Even Numbers By 2](#example---multiply-even-numbers-by-2)
- [Resources](#resources)

## Gist
We want to use containers as a whole rather than using iterators. Wouldn't it be nice to have `std::sort(vec)` rather than `std::sort(vec.begin(), vec.end())`?

## Using Ranges
Ranges are introduced in C++20, lives in the header `<ranges>`. Most of the STL algorithms can be used with ranges.

### Example - Sorting
The old way:
```cpp
std::vector<int> nums{5, 4, 3, 2, 1};
std::sort(nums.begin(), nums.end());
```

The ranges way:
```cpp
std::vector<int> nums{5, 4, 3, 2, 1};
auto sorted = std::ranges::sort(nums);
```

## Chaining Ranges
Ranges can be chained using the `|` operator, similar to CLI pipelines.

### Example - Remove Starting Elements by Predicate
Use `drop_while` to drop N numbers from the start of the collection until the predicate becomes false.
```cpp
std::vector<int> nums{5, 4, 3, 2, 1};

// [3, 2, 1]
auto dropped = nums | std::ranges::views::drop_while([](int n){ return n > 3; });
```

### Example - Filter Even Numbers
Use `filter` to filter out even numbers.
```cpp
std::vector<int> nums{5, 4, 3, 2, 1};

// [4, 2]
auto even = nums | std::ranges::views::filter([](int n){ return n % 2 == 0; });
```

### Example - Multiply Even Numbers By 2
Use `filter` to filter out even numbers then use `transform` to perform multiplication.
```cpp
std::vector<int> nums{5, 4, 3, 2, 1};

// [8, 4]
auto even = nums | std::ranges::views::filter([](int n){ return n % 2 == 0; })
                 | std::ranges::views::transform([](int n){ return n * 2; });
```

## Resources
| What           | URL                                      |
| :------------- | :--------------------------------------- |
| Ranges Library | https://en.cppreference.com/w/cpp/ranges |
