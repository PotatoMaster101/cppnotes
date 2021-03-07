# `async` and `await`

- [Gist](#gist)
- [Using `async` and `await`](#using-async-and-await)
- [Avoid `async void` Methods](#avoid-async-void-methods)
- [Avoid `.Result` or `.Wait()`](#avoid-result-or-wait)
- [`await` Exception Handling](#await-exception-handling)
- [Summary](#summary)
- [References](#references)

## Gist
Use `async` to specify a method is asynchronous, allowing task based asynchronous programming.

## Using `async` and `await`
Marking a method of `async` does not magically make the method run asynchronously. The only purpose of `async` is to enable the `await` keyword in the method:
```cs
async Task Foo() {
    // now you can use `await` keyword in here
}
```

When a method is marked as `async`, change return type to `Task` (or `Task<T>` where `T` is the method return type). To call an `async` method, place `await` method before the call (this also means the calling method needs to be `async` too):
```cs
async Task CallFoo() {
    await Foo();
}
```

## Avoid `async void` Methods
Never mark a `void` method as `async`, the only exception being event handlers (e.g., handling button click in WPF):
```cs
async void Button1_Clicked(object sender, RoutedEventArgs e) {
    // async void method
}
```
In the case of event handlers, changing `void` to `Task` will cause a compile error. Marking a `void` method as `async` has a few disadvantages such as:

The method can't be awaited:
```cs
async void Foo() {
    Console.WriteLine("async void");
}

await Foo();    // compile error
```

The exception can never be caught:
```cs
async void Foo() {
    throw new Exception("try catch me");
}

try {
    Foo();      // exception can't be caught in try/catch leading to crash
} catch { }
```

Best practice: the code in `async void` should be as short as possible and wrapped in try/catch to avoid exceptions:
```cs
async void Button1_Click(object sender, RoutedEventArgs e) {
    try {
        await DoSomeStuff();
    } catch { }
}
```

## Avoid `.Result` or `.Wait()`
Using `.Result` (or `.Wait()`) on `Task<T>` will cause the task to run synchronously and potentially cause a deadlock with the GUI thread. In most cases, you want to use `await` instead.

**Good**:
```cs
var result = await Foo();
```

**Bad**:
```cs
var task = Foo();
var result = task.Result;
```

It is recommended to use `.Result` only when the task has already completed (already used `await`):
```cs
var task = Foo();
var result = await task;
// ...
var result_using_result = task.Result;     // OK, since task has already completed
```

## `await` Exception Handling
Handle exceptions normally when using `async` and `await`. The exception will appear on the current thread when using `await`:
```cs
async Task<IEnumerable<string>> ThrowSomeException() {
    return await File.ReadAllLinesAsync("this file does not exist");    // throws FileNotFoundException
}

try {
    var lines = await ThrowException();     // the exception can be caught on the current thread
} catch { }
```

## Summary
- Never mark an `async` method as `void` unless it is an event handler, in which case keep the method as short as possible and use try/catch
- Don't use `.Result` or `.Wait()` on tasks, it will make the task run synchronously

## References
| What      | URL                                                                               |
| :-------- | :-------------------------------------------------------------------------------- |
| `async`   | https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async  |
| `await`   | https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await |
| `Task`    | https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task           |
| `Task<T>` | https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1         |
