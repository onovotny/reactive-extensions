# reactive-extensions

<a href='https://travis-ci.org/akarnokd/reactive-extensions/builds'><img src='https://travis-ci.org/akarnokd/reactive-extensions.svg?branch=master'></a>
<a href='https://www.nuget.org/packages/akarnokd.reactive_extensions'><img src='https://img.shields.io/nuget/v/akarnokd.reactive_extensions.svg' alt="reactive-extensions NuGet version"/></a>

Extensions to the [dotnet/reactive](https://github.com/dotnet/reactive) library.

### Setup

```
Install-Package akarnokd.reactive_extensions -Version 0.0.2-alpha
```

## Operators

These operators are available as extension methods on `IObservable` via the
`akarnokd.reactive_extensions.ReactiveExtensions` static class.

- Side-effecting sequences
  - [DoAfterNext](#doafternext), [DoAfterTerminate](#doafterterminate)
  - [DoOnSubscribe](#doonsubscribe), [DoOnDispose](#doondispose)
  - [DoFinally](#dofinally)
- Custom asynchronous boundaries
  - [ObserveOn](#observeon)
  - [ToSerialized](#toserialized)
- Combinators
  - [ConcatMap](#concatmap)
  - [ConcatMapEager](#concatmapeager)
  - [ConcatMany](#concatmany)
  - [MergeMany](#mergemany)
- Aggregators
  - [Collect](#collect)
- Composition
  - [Compose](#compose)
- Termination handling
  - [Repeat](#repeat)
  - [RepeatWhen](#repeatwhen)
  - [Retry](#retry)
  - [RetryWhen](#retrywhen)
- Test support
  - [Test](#test)

### Collect

Collects upstream items into a per-observer collection object created
via a function and added via a collector action and emits this collection
when the upstream completes.

```cs
Observable.Range(1, 5)
.Collect(() => new List<int>(), (a, b) => a.Add(b))
.Test()
.AssertResult(new List<int>() { 1, 2, 3, 4, 5 });
```

### Compose

Applies a function to the source at assembly-time and returns the
observable returned by this function.
This allows creating reusable set of operators to be applied on observables.

```cs
Func<IObservable<int>, IObservable<int>> applySchedulers =
    o => o.SubscribeOn(Scheduler.NewThread).ObserveOn(Scheduler.UI);

var o1 = Observable.Range(1, 5).Compose(applySchedulers);
var o2 = Observable.Range(10, 5).Compose(applySchedulers);
var o3 = Observable.Range(10, 5).Compose(applySchedulers);
```

### ConcatMap

Maps the upstream values into enumerables and emits their items in-order.

### ConcatMany

Concatenates (flattens) a sequence of observables.

### ConcatMapEager

Maps the upstream values into observables, runs some or all of them at once and emits items of
one such observable until it completes, then switches to the next observable, and so on until no
more observables are running.

### DoAfterNext

Call a handler after the current item has been emitted to the downstream.

### DoAfterTerminate

Call a handler after the upstream terminated normally or with an error.

### DoOnSubscribe

Call a handler just before the downstream subscribes to the upstream.

### DoOnDispose

Call a handler when the downstream disposes the sequence.

### DoFinally

Call a handler after the upstream terminates normally, with an error or the
downstream disposes the sequence.

### MergeMany

Merges some or all inner observables provided via an observable sequence and emits their
items in a sequential manner.

### ObserveOn

Observe the signals of the upstream on a specified scheduler and optionally
delay any error after all upstream items have been delivered.

(The standard `ObserveOn` by default allows errors to cut ahead.)

### Repeat

Repeatedly re-subscribes to the source observable if the predicate
function returns true upon the completion of the previous
subscription.

### RepeatWhen


### Retry

Repeatedly re-subscribes to the source observable if the predicate
function returns true upon the failure of the previous
subscription.

### RetryWhen



### Test

Subscribe a `TestObserver` to the observable sequence to perform various
convenient assertions.

### ToSerialized

Wraps an `IObserver` or an `ISubject` and serializes the calls to its `OnNext`, `OnError` and `OnCompleted`
methods by making sure they are called non-overlappingly and non-concurrently.

## Other classes

### TestObserver

An `IObserver` that offers a bunch of `AssertX` methods to check what signals
has been received from the upstream as well as have some trivial output checks
in a more convenient fashion.

```cs
var to = new TestObserver<int>();

Observable.Range(1, 5).Subscribe(to);

to.AssertResult(1, 2, 3, 4, 5);
```

There is the `Test()` extension method to make this more convenient in some cases:

```cs
Observable.Range(1, 5)
.Test()
.AwaitDone(TimeSpan.FromSeconds(5))
.AssertResult(1, 2, 3, 4 ,5)
```

### UnicastSubject

A special `ISubject` that supports exactly one `IObserver` during the subject's lifetime
and buffers signals until such observer subscribes to it.