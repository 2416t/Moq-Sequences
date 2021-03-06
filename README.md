# Moq.Sequences #

Allows you to enforce that methods or property accessors are called in a specific sequence.

[![NuGet version](https://img.shields.io/nuget/v/Moq.Sequences.svg)](https://www.nuget.org/packages/Moq.Sequences)

## Features ##

* Supports method invocations, property setters and getters.
* Allows you to specify the number of times a specific call should be expected.
* Provides loops which allow you to group calls into a recurring group.
* Allows you to specify the the number of times a loop should be expected.
* Calls that are expected to be called in sequence can be inter-mixed with calls that are expected in any order.
* Multi-threaded support, as well as support for `Task`-based concurrency and `async` / `await`.

## To Use ##

Add a reference to the [NuGet package `Moq.Sequences`](https://www.nuget.org/packages/Moq.Sequences) to your .NET project.

### Sequences ###

You create a `Sequence` as an envelope for both the expectations and the corresponding mock execution.

Sequences are created by calling `Sequence.Create()`.

During mock execution, any violation of sequencing will throw a `SequenceException` immediately.
When the `Sequence` is disposed any sequencing expectations that were not fulfilled will cause a 
`SequenceException` to be thrown.

Sequences cannot be nested. An attempt to create more than one will throw a `SequenceUsageException`.

### Steps ###

To specify that a call is expected, append `.InSequence()` to the `mock.Setup()` method.
You can optionally include the number of times the call is expected - the default is once.
To enforce a simple sequence do the following:

```csharp
// using Moq.Sequences;

using (Sequence.Create())
{
    mock.Setup(_ => _.Method1()).InSequence();
    mock.Setup(_ => _.Method2()).InSequence(Times.AtMostOnce());
    …
    // Logic that triggers the above method calls should be done here.
    …
}
```

Note that the order expected is the order in which the calls to `.Setup()` execute.

### Loops ###

You can specify that a group of calls should be done in sequence multiple times.
An example could be a test that expects a resource to be opened, read and then closed where these operations should always be done in sequence the same number of times.

You create a Loop by calling `Sequence.Loop()` where any number of iterations is allowed, or `Sequence.Loop(Times)` if you want to restrict the number of times the loop executes.

```csharp
// using Moq.Sequences;

using (Sequence.Create())
{
    …
    using (Sequence.Loop(Times.Exactly(3)))
    {
        mock.Setup(_ => _.Method1()).InSequence();
        mock.Setup(_ => _.Method2()).InSequence();
    }
    …
    // Logic that triggers the above method calls should be done here.
    …
}
```

### Support for `Task`-based concurrency and `async` / `await` ###

This library was originally designed with explicit multi-threading in mind. By default, the currently
active `Sequence` is tied to (and only accessible on) the specific thread that created it.
Unfortunately, this behaviour does not play well together with `Task`-based concurrency.

You can change Moq.Sequences' behavior to a mode that is compatible with `Task`-based concurrency
and `async` / `await` by doing the following before any of your tests execute:

```csharp
Sequence.ContextMode = SequenceContextMode.Async;
```

## To Build ##

Both the source code and the NuGet package can be built with Visual Studio 2017 (Community Edition or better).
