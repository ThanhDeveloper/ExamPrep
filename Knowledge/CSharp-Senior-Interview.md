# Senior C# Interview Knowledge Base

> Target: C# 14 on .NET 10 (LTS)  
> Compatibility window: .NET 5 through .NET 10  
> Research review date: 2026-09-09  
> Source policy: Microsoft Learn and official `dotnet` repositories only

## How to use this handbook

This is an interview reference, not a syntax tutorial. Read each topic at four levels: **what** contract the source code expresses, **why** the feature exists, **how** observable behavior follows, and **what the compiler/runtime does conceptually**. The stars are study priority, not difficulty:

- ⭐ Must Know — expected in almost every senior interview.
- ⭐⭐ Important Senior Knowledge — design, correctness, or production implications.
- ⭐⭐⭐ Deep Senior / Runtime Knowledge — useful for explaining internals without pretending implementation details are language guarantees.

Unless a version note says otherwise, the described behavior is unchanged across .NET 5–10. C# and .NET versions are related but distinct: the compiler supplies language features; the target framework/runtime supplies required types and execution behavior.

| Runtime | Default contemporary C# | Release character |
|---|---:|---|
| .NET 5 | C# 9 | Records, `init`, top-level statements, relational/logical patterns |
| .NET 6 | C# 10 | Record structs, global usings, file-scoped namespaces, interpolated-string handlers |
| .NET 7 | C# 11 | Required members, raw/UTF-8 literals, list patterns, generic math, `scoped` |
| .NET 8 | C# 12 | Primary constructors, collection expressions, `ref readonly` parameters |
| .NET 9 | C# 13 | `System.Threading.Lock`, `params` collections, ref/unsafe improvements |
| .NET 10 | C# 14 | Extension members, `field`, null-conditional assignment, more partial members |

Do not casually set `<LangVersion>latest</LangVersion>` in reproducible builds: `latest` changes when a newer compiler is installed. Target the framework’s default or pin an explicit version when policy requires it.

## Guide map

- [I. Language fundamentals](#part-i--c-language-fundamentals)
- [II. Object-oriented C#](#part-ii--object-oriented-c)
- [III. Equality](#part-iii--object-equality)
- [IV. Strings](#part-iv--strings)
- [V. Collections](#part-v--collections)
- [VI. Generics](#part-vi--generics)
- [VII. Delegates, events, and lambdas](#part-vii--delegates-events-and-lambdas)
- [VIII. Exceptions](#part-viii--exception-handling)
- [IX. Disposal](#part-ix--disposal-and-resource-management)
- [X. Garbage collection](#part-x--garbage-collection)
- [XI. Memory and performance](#part-xi--memory-and-performance)
- [XII. Async/await](#part-xii--async-and-await)
- [XIII. Multithreading and concurrency](#part-xiii--multithreading-and-concurrency)
- [XIV. LINQ](#part-xiv--linq)
- [XV. Nullability](#part-xv--nullability)
- [XVI. Modern C#](#part-xvi--modern-c-features)
- [XVII–XXIV. Platform APIs](#part-xvii--reflection-and-metadata)
- [XXV–XXVIII. Runtime, performance, diagnostics](#part-xxv--compilation-and-runtime)
- [XXIX. Interview traps](#part-xxix--senior-interview-traps)
- [XXX. What happens internally?](#part-xxx--what-happens-internally)
- [XXXI. Comparison cheat sheets](#part-xxxi--comparison-cheat-sheets)
- [XXXII. Questions by level](#part-xxxii--interview-questions-by-level)
- [XXXIII. Rapid review](#part-xxxiii--senior-c-rapid-review)
- [XXXIV. Priority map](#part-xxxiv--knowledge-priority)

---

# Part VI — Generics

## 20. Why generics exist ⭐

Generics express algorithms and data structures in terms of type parameters while preserving compile-time type safety. Compared with an `object`-based API, they normally avoid casts and avoid boxing value-type arguments.

```csharp
static T Echo<T>(T value) => value;

int number = Echo(42);       // inferred as Echo<int>
string text = Echo("hello"); // inferred as Echo<string>
```

The type parameter is part of the constructed type: `List<int>` and `List<string>` are different closed types. An open type such as `Dictionary<,>` still contains unassigned type parameters.

## 21. Constraints ⭐

Constraints state what operations generic code may legally perform and reject unsuitable type arguments at compile time.

| Constraint | Meaning / important nuance |
|---|---|
| `where T : class` | Non-nullable reference type in a nullable-enabled context |
| `where T : class?` | Nullable or non-nullable reference type |
| `where T : struct` | Non-nullable value type; excludes `Nullable<T>` |
| `where T : notnull` | Non-nullable value or reference type; violation is generally a nullable warning |
| `where T : unmanaged` | Non-nullable unmanaged type; enables pointer-oriented operations |
| `where T : Base` | `Base` or a derived type |
| `where T : IFoo` / `IFoo?` | Implements the interface, with nullable annotation semantics |
| `where T : new()` | Public parameterless constructor; it must be last among ordinary constraints |
| `where T : U` | `T` derives from or implements another type parameter `U` |
| `where T : allows ref struct` | Anti-constraint: permits ref-like type arguments; generic code must obey ref-safety rules |

Certain constraints are mutually exclusive, and ordering is specified by the language. Multiple interface constraints are permitted. A base-class constraint, when present, precedes interface constraints.

### Example

```csharp
static T CreateAndValidate<T>() where T : class, IValidatable, new()
{
    T value = new();
    value.Validate();
    return value;
}

interface IValidatable { void Validate(); }
```

`new T()` is legal only because of `new()`. The interface member is callable because of the interface constraint.

## 22. Variance ⭐⭐

Variance applies to reference conversions for interfaces and delegates whose type parameters are declared variant. It does not make arbitrary generic classes variant, and it does not apply through value-type substitutions.

```csharp
IEnumerable<string> names = ["Ada"];
IEnumerable<object> objects = names; // covariance: out T

Action<object> printObject = Console.WriteLine;
Action<string> printString = printObject; // contravariance: in T

Func<object, string> describe = x => x.ToString()!;
Func<string, object> widened = describe;  // in argument, out result
```

- **Covariance (`out`)** permits a more-derived result type to be viewed as a less-derived one. The parameter appears only in output-safe positions.
- **Contravariance (`in`)** permits a less-derived consumer to stand in for a more-derived consumer. The parameter appears only in input-safe positions.
- **Invariance** permits neither conversion. Mutable collections such as `List<T>` must be invariant: treating `List<string>` as `List<object>` would allow adding an `object` that is not a string.

## 23. Runtime representation ⭐⭐⭐

The CLR supports generics directly; constructed generic types retain type information at runtime. Microsoft runtime documentation describes shared code for many reference-type instantiations and specialized native code for value-type instantiations. The implementation may use dictionaries and helper lookups to supply type-specific facts to shared code. Treat exact native-code layout as an implementation detail, not a C# guarantee.

Practical consequence: `List<int>` stores integers inline in its internal array without boxing. Generic code is not equivalent to compiler-generated `object` code.

### Interview check

**Basic — Q:** What problem do generics solve?  
**A:** Reusable, compile-time type-safe code without routine casts and value-type boxing.

**Intermediate — Q:** Why is `List<string>` not a `List<object>`?  
**A:** The class is invariant; otherwise callers could insert a non-string and violate the original list's type safety.

**Senior — Q:** Contrast `class`, `class?`, and `notnull` constraints.  
**A:** `class` requires a non-nullable reference annotation, `class?` accepts either reference annotation, and `notnull` accepts non-nullable reference or value types.

**Deep Dive — Q:** Does the runtime produce identical machine code for every `T`?  
**A:** No guaranteed one-size rule. CoreCLR generally shares compatible reference-type instantiations and specializes value-type instantiations, with exact details left to the runtime.

### Official Microsoft references

- [Generics in .NET](https://learn.microsoft.com/en-us/dotnet/standard/generics/)
- [Constraints on type parameters](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)
- [Variance in generic interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-generic-interfaces)
- [Variance in delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates)
- [CoreCLR generics design](https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/generics.md)

---

# Part VII — Delegates, Events, and Lambdas

## 24. Delegates and multicast invocation ⭐

A delegate is a type-safe reference to a method. A delegate value combines a target (possibly `null` for a static method) with method information. Delegates can be combined into an invocation list. A non-void multicast call returns the last handler's result; if a handler throws, later handlers are not invoked unless the publisher explicitly invokes and isolates them one by one.

- `Action<...>` returns `void`.
- `Func<..., TResult>` returns `TResult`; its final type argument is the result.
- `Predicate<T>` represents `bool (T)` and is used by some older BCL APIs.

```csharp
Action handlers = () => Console.Write("A");
handlers += () => Console.Write("B");
handlers();
```

**Output**

```text
AB
```

## 25. Lambdas, anonymous methods, and local functions ⭐

Lambdas and anonymous methods create anonymous functions that can convert to compatible delegate or expression-tree types. A local function is a named function scoped inside another member. Local functions support recursion, iterator syntax, attributes, and generic parameters; a `static` local function cannot capture local state. Prefer a local function when naming, recursion, immediate argument validation, or avoiding a delegate conversion makes the code clearer.

### Closures and captured variables ⭐⭐

Captured variables are variables, not frozen values. Their lifetime is extended as needed. The compiler can represent captured state in a generated closure object; a delegate may therefore allocate and keep the captured object graph alive.

### Example

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
{
    int copy = i;
    actions.Add(() => Console.Write(copy));
}

foreach (Action action in actions) action();
```

### Output

```text
012
```

### Explanation

Each iteration creates a distinct `copy`. Capturing the single `i` variable instead would cause all delegates to observe its final value (`3`). Modern `foreach` iteration variables are distinct per iteration, but other captured mutable variables still obey variable-capture semantics.

## 26. Events ⭐

An event exposes controlled subscription and unsubscription while restricting ordinary consumers from replacing or invoking the underlying delegate. Only the declaring type can normally raise the event.

```csharp
sealed class Clock
{
    public event EventHandler? Tick;
    public void RaiseTick() => Tick?.Invoke(this, EventArgs.Empty);
}
```

**Delegate versus event:** a delegate property/value lets callers assign and invoke it; an event normally lets outside callers only add/remove handlers. Use the standard `EventHandler`/`EventHandler<TEventArgs>` pattern for conventional notifications.

### Subscriber lifetime trap ⭐⭐

The publisher holds delegate references to subscriber targets. If a long-lived publisher outlives a short-lived subscriber, failing to unsubscribe can keep the subscriber reachable. Use deterministic unsubscription, a returned `IDisposable` subscription, or a design whose publisher lifetime is no longer than subscribers. An anonymous lambda cannot be conveniently unsubscribed unless its delegate instance was stored.

Events are not inherently thread-safe. The null-conditional invocation pattern obtains a stable local evaluation, but application-level subscription, ordering, reentrancy, handler failure, and state synchronization remain design concerns.

### Interview check

**Basic — Q:** `Action` versus `Func`?  
**A:** `Action` has no return value; `Func`'s final generic argument is its return type.

**Intermediate — Q:** What does a closure capture?  
**A:** The variable/storage location, not a snapshot of its value.

**Senior — Q:** How can an event cause a managed memory leak?  
**A:** A reachable publisher's invocation list strongly references subscriber targets, so forgotten subscriptions extend subscriber lifetime.

**Deep Dive — Q:** Must every lambda allocate?  
**A:** No. Noncapturing lambdas can be cached and some uses can be optimized; capturing commonly requires closure state and a delegate. Do not promise an allocation without measuring generated/runtime behavior.

### Official Microsoft references

- [Delegates and lambdas](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/delegates-lambdas)
- [Delegates programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Lambda expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions)
- [Local functions](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions)
- [Events programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)
- [Subscribe to and unsubscribe from events](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/how-to-subscribe-to-and-unsubscribe-from-events)

---

# Part VIII — Exception Handling

## 27. Mechanics and design ⭐

Exceptions separate failure propagation from the normal return path. Catch an exception only when the current boundary can recover, translate it meaningfully, add context while retaining the original, or perform required logging/cleanup without duplicating responsibility. Validate common bad input with ordinary control flow; exceptions are comparatively expensive to throw and should represent exceptional conditions.

`finally` runs when control leaves its `try` in the ordinary exception/return paths, making it suitable for cleanup. `using` is preferable for disposable resources. A catch should normally be ordered from most specific to most general.

### `throw;` versus `throw ex;` ⭐

```csharp
try
{
    Save();
}
catch (IOException ex)
{
    Log(ex);
    throw;       // preserves the original stack trace
    // throw ex; // restarts the trace at this statement
}
```

To translate abstraction boundaries, include the original as `InnerException`:

```csharp
catch (IOException ex)
{
    throw new RepositoryException("Saving the order failed.", ex);
}
```

Custom exceptions should derive from `Exception`, use a meaningful domain name, and provide the constructors required by the library's compatibility needs. Do not invent a custom type when an existing exception precisely expresses the contract.

## 28. Exception filters ⭐⭐

Filters decide whether a catch handles an exception and run before stack unwinding for that handler. They are useful for conditional handling and for observing a failure while preserving the original stack state.

```csharp
catch (HttpRequestException ex) when (ex.StatusCode is HttpStatusCode.NotFound)
{
    return null;
}
```

Avoid side effects whose correctness depends on a filter running exactly once. If a filter throws, that filter is treated as false and exception search continues.

## 29. Async and aggregate failures ⭐⭐

An exception from an `async Task`/`Task<T>` method is stored in its returned task and rethrown when awaited, preserving the async causal stack information. Code before the first incomplete await may execute synchronously, but callers should reason through the returned task's completion. `async void` sends failures to the current synchronization environment and cannot be awaited by the caller.

`await Task.WhenAll(...)` completes after every supplied task completes. Its returned task contains aggregate failure information. Awaiting throws one of the exceptions; inspect the completed task's `Exception` when all failures matter. `AggregateException` is especially visible in TPL/blocking APIs and supports `Flatten` and selective `Handle`.

Cancellation is normally represented by `OperationCanceledException` associated with the token, and tasks may transition to Canceled. Do not confuse cancellation with timeout or arbitrary failure.

### Interview check

**Basic — Q:** What is `finally` for?  
**A:** Cleanup that must run as control leaves a protected region, whether by normal flow or exception.

**Intermediate — Q:** Why prefer `throw;` when rethrowing?  
**A:** It retains the original stack trace; `throw ex;` resets the apparent throw point.

**Senior — Q:** Why are exception filters more than syntactic sugar?  
**A:** They evaluate before the selected handler unwinds the stack and permit conditional handling without catch/rethrow stack effects.

**Deep Dive — Q:** Does awaiting `WhenAll` directly throw an `AggregateException` containing everything?  
**A:** The aggregate task records all faults, but `await` rethrows one exception. Retain and inspect the task's `Exception` if every fault is required.

### Official Microsoft references

- [Exceptions and exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)
- [Exception-handling statements](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements)
- [Best practices for exceptions](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Using user-filtered exception handlers](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/using-user-filtered-exception-handlers)
- [`AggregateException`](https://learn.microsoft.com/en-us/dotnet/api/system.aggregateexception?view=net-10.0)
- [`Task.WhenAll`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall?view=net-10.0)

---

# Part IX — Disposal and Resource Management

## 30. GC is not resource cleanup ⭐

The GC reclaims unreachable **managed memory**. It does not know when to release scarce external resources such as native handles, sockets, file descriptors, database leases, or unmanaged buffers. `IDisposable.Dispose` provides deterministic cleanup; `IAsyncDisposable.DisposeAsync` supports cleanup that itself requires asynchronous work.

```csharp
using FileStream stream = File.OpenRead(path);
// stream is disposed at the end of the containing scope

await using AsyncResource resource = await AsyncResource.OpenAsync();
```

A `using` statement/declaration is lowered conceptually to a `try/finally` that calls disposal, including when the body throws. `await using` awaits `DisposeAsync`.

## 31. The modern dispose pattern ⭐⭐

- A sealed type that merely owns other disposables can usually implement `Dispose` directly and idempotently.
- An unsealed base intended for inheritance generally exposes `protected virtual Dispose(bool disposing)`.
- Dispose owned managed objects when `disposing` is true.
- Release directly owned unmanaged state on both paths if the type has a finalizer.
- Call `GC.SuppressFinalize(this)` after explicit cleanup when a finalizer exists.
- Do not dispose dependencies the object does not own.

```csharp
sealed class BufferOwner : IDisposable
{
    private IMemoryOwner<byte>? _owner = MemoryPool<byte>.Shared.Rent();

    public void Dispose() => Interlocked.Exchange(ref _owner, null)?.Dispose();
}
```

Thread-safe idempotence may matter when ownership crosses threads; the exact synchronization should match the type's documented concurrency contract.

## 32. Finalizers and `SafeHandle` ⭐⭐

A finalizable object requires extra GC work: when first found unreachable it is queued for finalization and normally survives until a later collection can reclaim it. Finalizer timing and order are nondeterministic; finalizers must not depend on other finalizable managed objects still being usable.

Prefer wrapping an OS handle in `SafeHandle` rather than writing a finalizer. `SafeHandle` centralizes reliable handle release, reduces custom finalization complexity, and cooperates with interop lifetime rules. A class that owns a `SafeHandle` normally disposes it and does not need its own finalizer.

`IAsyncDisposable` does not replace `IDisposable` automatically. A type may implement one or both. Consumers use the contract appropriate to the resource and calling context.

### Interview check

**Basic — Q:** Why does a file stream need disposal if it is managed?  
**A:** It owns scarce OS resources whose timely release cannot wait for nondeterministic GC.

**Intermediate — Q:** What does `using` guarantee?  
**A:** It invokes the selected dispose operation as the scope exits, including exceptional exit; it does not make the object thread-safe.

**Senior — Q:** Why prefer `SafeHandle` to a custom finalizer?  
**A:** It encapsulates reliable native-handle finalization and avoids much error-prone finalizer logic in the owning class.

**Deep Dive — Q:** Why can finalization increase retention?  
**A:** An unreachable finalizable object is queued and must survive long enough for its finalizer to run before later reclamation.

### Official Microsoft references

- [Implement a `Dispose` method](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose)
- [Implement a `DisposeAsync` method](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-disposeasync)
- [Cleaning up unmanaged resources](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/unmanaged)
- [`using` statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)
- [`SafeHandle`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.safehandle?view=net-10.0)

---

# Part X — Garbage Collection

## 33. Reachability and generations ⭐

The managed heap is organized into segments and generations. Allocation is normally very cheap because the GC can advance an allocation pointer. Collection cost comes later: the GC determines reachability, reclaims dead objects, and may relocate live objects and update references.

```text
Roots (stack/register references, statics, GC handles, finalization machinery)
                       │
                       ▼
             reachable object graph
                       │
         unreachable objects are reclaimable
```

- **Generation 0:** newest, usually short-lived objects; collected most frequently.
- **Generation 1:** buffer between short- and long-lived objects.
- **Generation 2:** long-lived objects; a full collection includes generations 0–2.
- **Ephemeral generations:** generations 0 and 1, normally placed on an ephemeral segment.

Surviving objects can be promoted. Generation is an adaptive collection strategy, not a statement that an object permanently moves on every collection or that old objects are leaks.

## 34. LOH, POH, pinning, and fragmentation ⭐⭐

The **large object heap (LOH)** contains allocations of at least **85,000 bytes** according to current Microsoft documentation. LOH collection occurs with generation 2. Because copying large objects is expensive, the LOH is ordinarily swept rather than compacted; compaction can be requested for the next blocking collection. The threshold is an implementation/runtime policy, not a C# language rule.

The **pinned object heap (POH)** supports objects allocated as pinned. Pinning prevents relocation, which native code sometimes requires, but pinned gaps can inhibit compaction and increase fragmentation. Pin for the shortest practical time; stable interop abstractions and dedicated pinned allocations can reduce damage.

Fragmentation means sufficient total free space may exist but not in suitable contiguous ranges. It can increase committed memory and allocation difficulty. Diagnose before forcing collections or compaction.

## 35. Collection modes ⭐⭐

- **Workstation GC** targets client responsiveness and uses one managed heap; concurrent/background behavior is configurable.
- **Server GC** targets throughput on multiprocessor server workloads, normally using a heap and dedicated collection thread per logical processor participating in the process.
- **Background GC** performs much of generation 2 collection concurrently with managed execution; ephemeral collections can occur during it.

Selection is deployment/workload-sensitive. Server GC is not universally faster and can use more memory. Use runtime configuration and measurements representative of production.

## 36. Finalization and weak references ⭐⭐

Finalizable objects enter finalization machinery rather than being reclaimed immediately. A finalizer can resurrect an object by making it reachable, but this is fragile and should be avoided. A `WeakReference<T>` observes an object without keeping it strongly reachable; the target can disappear between checks, so obtain and use the result of `TryGetTarget` locally.

Weak references are useful for optional caches or mappings only when recomputation/loss is acceptable. They do not impose eviction policy and are not a general cure for cache design.

## 37. Managed leaks and GC pressure ⭐

A managed leak is commonly unintended reachability, not forgotten manual deallocation. Typical roots include:

- a growing static collection;
- a cache without size/expiration policy;
- a long-lived publisher retaining event subscribers;
- a closure or task retaining a large object graph;
- queued work or timers retaining state;
- native resources awaiting finalization because disposal was skipped.

High **allocation rate** produces GC pressure even if objects die quickly. Symptoms include excess pause time, CPU in GC, higher generations, LOH churn, or high retained size. The correct remedy depends on evidence: reduce avoidable allocation, shorten references, bound caches, reuse buffers where ownership is safe, and dispose promptly. Calling `GC.Collect()` routinely usually disrupts the collector's heuristics and is not a leak fix.

### Example: accidental event retention

```csharp
sealed class Publisher { public event Action? Changed; }
sealed class Subscriber
{
    private readonly byte[] _state = new byte[1_000_000];
    public Subscriber(Publisher p) => p.Changed += OnChanged;
    private void OnChanged() { }
}
```

If `Publisher` remains rooted, its event delegate can retain `Subscriber` and `_state`. Provide a matching unsubscribe/disposable subscription.

### Version notes

The generational model and documented LOH threshold apply throughout .NET 5–10. The POH is available in this window. Runtime defaults and container-aware heuristics can evolve, so verify deployed runtime configuration rather than infer it from source code alone.

### Interview check

**Basic — Q:** What makes an object collectable?  
**A:** It is no longer reachable from GC roots through strong references (subject to finalization/other runtime machinery).

**Intermediate — Q:** Why are generations effective?  
**A:** Most objects die young, so collecting a small young region often reclaims substantial memory cheaply.

**Senior — Q:** What is the current documented LOH threshold?  
**A:** 85,000 bytes or larger; remember this is a runtime threshold, not a guarantee about physical placement or collection timing.

**Deep Dive — Q:** Why does pinning hurt?  
**A:** The collector cannot relocate pinned objects, so holes around them can prevent effective compaction and contribute to fragmentation.

### Official Microsoft references

- [Fundamentals of garbage collection](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)
- [The large object heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap)
- [Garbage collector configuration](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector)
- [Workstation and server garbage collection](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc)
- [Background garbage collection](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/background-gc)
- [Weak references](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/weak-references)
- [`GC.AllocateArray`](https://learn.microsoft.com/en-us/dotnet/api/system.gc.allocatearray?view=net-10.0)

---

# Part XI — Memory and Performance

## 38. Three storage domains ⭐

- A thread's **stack** holds frames and implementation-selected locals/temporaries. Frames unwind with calls; stack space is bounded.
- The **managed heap** holds GC-managed objects and may also contain boxed values, arrays of value types, and value-type fields embedded in objects.
- **Native memory** is outside GC-managed storage and needs explicit ownership, safe wrappers, or interop contracts.

Never substitute “value type = stack” for storage analysis. A value lives wherever its containing storage lives, and the JIT can enregister or transform locals while preserving observable semantics.

## 39. `Span<T>` and `Memory<T>` ⭐

`Span<T>` / `ReadOnlySpan<T>` are stack-only `ref struct` views over contiguous memory. Slicing creates another view rather than copying elements. A span can view an array, stack allocation, string (`ReadOnlySpan<char>`), or unmanaged memory under the appropriate safety rules.

`Memory<T>` / `ReadOnlyMemory<T>` are ordinary structs that can be stored on the heap and cross asynchronous suspension points. Obtain a short-lived `.Span` when synchronous access is needed.

| Need | Prefer |
|---|---|
| Synchronous, short-lived contiguous view | `Span<T>` / `ReadOnlySpan<T>` |
| Store the view in an object or use across `await` | `Memory<T>` / `ReadOnlyMemory<T>` |
| Own a variable-size buffer | Array, `IMemoryOwner<T>`, or another documented owner |
| Expose read-only intent | Read-only variant; note it is a view contract, not deep immutability |

```csharp
static int ParsePrefix(ReadOnlySpan<char> text)
{
    int comma = text.IndexOf(',');
    return int.Parse(comma < 0 ? text : text[..comma]);
}

Console.WriteLine(ParsePrefix("123,rest"));
```

**Output:** `123`. The slice does not allocate a substring.

## 40. `stackalloc`, ref safety, and `scoped` ⭐⭐

`stackalloc` allocates a block whose lifetime is bounded by the current method activation. Prefer assignment to a span so the compiler enforces safe access. Stack space is limited: use it only for small, bounded buffers and avoid loops that accumulate stack allocations until return.

```csharp
Span<byte> scratch = stackalloc byte[128];
scratch.Clear();
```

Ref-like values cannot ordinarily be boxed, captured, used as class fields, or survive an `await`/`yield` boundary in a way that could outlive referenced storage. `scoped` narrows the permitted escape scope of a parameter or local, allowing APIs to express that a reference will not be returned or stored beyond the call. These are compile-time ref-safety rules; do not describe them as speculative JIT escape analysis.

## 41. Pools and ownership ⭐⭐

`ArrayPool<T>` rents arrays, often larger than requested. Return the exact rented instance in `finally`; never use it after return. Contents are not guaranteed clear. Clearing may be required for references, secrets, or invariants, and has a cost. Pooling helps when repeated allocation of meaningful buffers is measured as a problem; for small/rare buffers it can add complexity, retained capacity, and data-lifetime risk.

```csharp
byte[] rented = ArrayPool<byte>.Shared.Rent(4096);
try
{
    Use(rented.AsSpan(0, 4096));
}
finally
{
    ArrayPool<byte>.Shared.Return(rented, clearArray: true);
}
```

`MemoryPool<T>` returns an `IMemoryOwner<T>`. Disposing the owner returns/releases the memory. This explicit ownership composes well with APIs built around `Memory<T>`.

## 42. Unsafe code, pointers, `fixed`, and pinning ⭐⭐⭐

Unsafe code permits pointer operations outside normal managed memory safety. `fixed` can pin a movable managed object for a bounded region and produce a stable pointer. Pinning is sometimes necessary for native interop, but long pins can fragment the heap. Prefer spans, `SafeHandle`, generated/built-in marshalling, and documented APIs until pointer access is demonstrably needed.

`ref readonly` exposes an alias that cannot be assigned through that reference. It can avoid copying a large struct, but member access on a non-readonly struct can still require defensive copies. A `readonly struct` declares that its instance fields are readonly and helps make value semantics explicit.

### Interview check

**Basic — Q:** Span versus array?  
**A:** An array owns managed storage; a span is a non-owning, bounded view over contiguous storage.

**Intermediate — Q:** Why can `Memory<T>` cross `await` but `Span<T>` cannot normally do so?  
**A:** `Memory<T>` is storable ordinary state; span is a ref-like stack-only type whose references must obey stricter lifetime rules.

**Senior — Q:** When is `ArrayPool<T>` worse than allocation?  
**A:** When buffers are small/infrequent or ownership and clearing costs outweigh measured GC savings; misuse also creates corruption/data exposure risks.

**Deep Dive — Q:** Is `scoped` proof of a runtime escape-analysis optimization?  
**A:** No. It is a language/compiler restriction on reference escape; the runtime remains free to optimize without changing behavior.

### Official Microsoft references

- [Memory- and span-related types](https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/)
- [`Memory<T>` and `Span<T>` usage guidelines](https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines)
- [Reduce memory allocations using C#](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/)
- [`stackalloc` expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc)
- [Unsafe code, pointer types, and function pointers](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)
- [`ArrayPool<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.buffers.arraypool-1?view=net-10.0)
- [`MemoryPool<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.buffers.memorypool-1?view=net-10.0)

---

# Part I — C# Language Fundamentals

## 1. The type system ⭐

C# is statically and strongly typed. Every expression has a compile-time type; an object also has a runtime type. The compile-time type controls member lookup, overload resolution, and available conversions. The runtime type controls virtual dispatch and runtime type tests.

The Common Type System (CTS) defines how .NET languages declare and use types. Every C# type is a **value type** or a **reference type**. The Common Language Specification (CLS) is the smaller interoperability contract libraries can follow so other CLS languages can consume their public API. For example, unsigned types exist in the CTS but are not CLS-compliant in public contracts.

| Category | Variable contains | Assignment/pass by value | Examples |
|---|---|---|---|
| Value type | The value itself | Copies the value, including each field; reference-typed fields copy their references | numeric types, `bool`, `char`, enum, struct, tuple, record struct, nullable value type |
| Reference type | A reference or `null` | Copies the reference; both variables can designate the same object | class, interface, array, delegate, `string`, record class, `dynamic` at runtime |

All types can be treated through `object` (`System.Object`). Value types derive conceptually from `System.ValueType`, which derives from `object`, but converting a non-nullable value to `object` normally boxes it.

### Stack versus heap—use the accurate model ⭐⭐

“Value types live on the stack; reference types live on the heap” is false.

- A class instance is normally allocated on the managed heap; a variable referring to it might be a stack local, field inside another heap object, static storage, register, or compiler-generated state-machine field.
- A value is stored **inline in its containing storage**. A value-type local may be in a stack slot or register; a value-type field is inside its containing class object on the heap; an array of structs stores elements inline in the array; a boxed value is copied into a heap object.
- Captured locals and locals that survive `await`/`yield` can become fields of compiler-generated heap objects. JIT optimizations can remove, split, or enregister locals. Storage location is not a C# semantic promise.
- The thread stack holds call frames and is reclaimed by unwinding. The managed heap is traced by the GC. Native allocations are outside both and require their owner’s release protocol.

The senior answer should lead with **copy and identity semantics**, then discuss storage only where performance or lifetime requires it.

## 2. Boxing, unboxing, and conversions ⭐

**Boxing** converts a value type to `object`, `System.ValueType`, an implemented interface, or another permitted reference type. The runtime allocates a box and copies the value. Boxing `Nullable<T>` produces `null` when `HasValue` is false; otherwise it boxes the underlying `T`. **Unboxing** checks that the object contains exactly a compatible boxed value type and then copies the value out; it is not a numeric conversion.

### Example

```csharp
int n = 42;
object box = n;          // allocation + copy
n = 7;
Console.WriteLine(box);  // boxed copy remains 42

long wrong = (long)box;  // InvalidCastException: box contains Int32
```

### Output

```text
42
Unhandled exception: System.InvalidCastException ...
```

### Explanation

The box has runtime type `System.Int32`; changing `n` cannot change its copied payload. Unbox to `int`, then convert: `long ok = (int)box;`.

Avoid hidden boxing in hot paths: value passed to `object`, nongeneric collections, interface calls that require boxing, composite formatting through `object[]`, and unconstrained generic operations can allocate. Confirm with measurement; modern generic APIs and interpolated-string handlers often avoid it.

**Conversions:**

- Identity/implicit conversions are proven safe by the language (for example `int` to `long`, derived reference to base reference).
- Explicit conversions might lose data or fail at runtime (for example `long` to `int`, base reference to derived reference).
- Reference conversions never change the object’s runtime type or identity.
- `as` performs a permitted reference/nullable conversion and returns `null` on failure; it cannot represent a required failure well. Pattern matching (`if (x is Customer c)`) tests and introduces a definitely typed variable.
- User-defined conversions may be `implicit` or `explicit`; keep implicit ones unsurprising and non-lossy.

### Checked arithmetic ⭐

An integral operation or conversion in a `checked` context throws `OverflowException` when overflow is detected; `unchecked` discards high-order bits for nonconstant integral operations. Constant expressions are checked at compile time by default. Floating-point overflow follows IEEE behavior (`Infinity`/`NaN`), not `OverflowException`; `decimal` overflow throws. Project-level overflow checking can change the default, so use an explicit context where correctness depends on it.

```csharp
int max = int.MaxValue;
Console.WriteLine(unchecked(max + 1));
try { Console.WriteLine(checked(max + 1)); }
catch (OverflowException) { Console.WriteLine("overflow"); }
```

```text
-2147483648
overflow
```

## 3. Variables, fields, and initialization ⭐

| Construct | Initialization/lifetime | Key point |
|---|---|---|
| Local | Must be definitely assigned before read; normally lives for the invocation/scope | Compiler analysis, not default zeroing as a language promise to the local |
| Instance field | Default-initialized, then field initializers and constructor logic | One per instance; `readonly` assignable at declaration or instance constructor |
| Static field | One per closed constructed type; initialized before first relevant use | Can root an object for process/load-context lifetime |
| `const` | Compile-time substituted value; implicitly static | Public constants are copied into consuming assemblies—versioning risk |
| `static readonly` | Runtime value assigned at declaration/static constructor | Consumers read the field, so safer for evolving library values |

`readonly` prevents reassignment of a field after construction; it does not deep-freeze the referenced object. A `readonly List<int>` can still be mutated. `readonly struct` prevents mutation through its instance members and helps avoid defensive copies when used correctly.

## 4. Parameter passing and managed references ⭐

Parameters are **passed by value by default**. For a reference type, that value is a reference. Therefore a method can mutate the referred object but cannot replace the caller’s variable unless the variable itself is passed by reference.

```csharp
sealed class Person { public string Name { get; set; } = "Ada"; }

static void Change(Person person)
{
    person.Name = "Grace";       // mutates shared object
    person = new() { Name = "Linus" }; // replaces only local parameter copy
}

static void Replace(ref Person person) =>
    person = new() { Name = "Margaret" };

var p = new Person();
Change(p);
Console.WriteLine(p.Name);
Replace(ref p);
Console.WriteLine(p.Name);
```

```text
Grace
Margaret
```

| Modifier | Caller initializes? | Callee reads? | Callee assigns? | Primary intent |
|---|---:|---:|---:|---|
| none | yes | yes | local parameter only | Copy value/reference |
| `ref` | yes | yes | optional | Read/write caller storage |
| `out` | no | only after assignment | required before normal return | Produce an additional value |
| `in` | yes | yes | no | Flexible readonly-by-reference input; compiler may create a temporary |
| `ref readonly` parameter | yes | yes | no | API expects an actual location/read-only reference |

`ref` locals alias storage. A `ref` return exposes an alias to caller-visible storage; use it only when mutation/zero-copy benefits justify a tighter lifetime contract. `ref readonly` exposes an alias without permitting assignment through it. `scoped` restricts a managed reference or ref-like value so it cannot escape the permitted context. The compiler’s safe-context and ref-safe-context rules reject returning a reference to a dead local.

```csharp
static ref int Find(int[] values, int target)
{
    for (int i = 0; i < values.Length; i++)
        if (values[i] == target) return ref values[i];
    throw new KeyNotFoundException();
}

int[] values = [10, 20, 30];
ref int slot = ref Find(values, 20);
slot = 99;
Console.WriteLine(values[1]);
```

```text
99
```

### Version notes

- .NET 5/C# 9 through .NET 6/C# 10 already support the core ref features.
- C# 11 adds `scoped` and ref fields in `ref struct` types.
- C# 12 adds `ref readonly` parameters.
- C# 13 relaxes some `ref struct`, iterator, async, and generic restrictions, but a ref-like value still cannot live across an `await` or `yield` boundary where its lifetime would be unsafe.

### Interview check

**Basic — Q:** Does passing a class instance to a method pass the object by reference?  
**A:** The reference value is passed by value. Both sides initially designate the same object, but reassignment of the parameter does not replace the caller’s variable.

**Intermediate — Q:** Can a struct be on the managed heap?  
**A:** Yes—inline inside a class, in an array, in a boxed object, or as part of a compiler-generated object.

**Senior — Q:** Why can `const` be a public-library versioning hazard?  
**A:** Consumers embed the value at compile time; changing the library without recompiling consumers can leave old values in use.

**Deep Dive — Q:** What makes returning `ref` to a local illegal?  
**A:** The returned alias could outlive the local’s storage. C# ref-safety escape analysis rejects that program at compile time.

### Official Microsoft references

- [The C# type system](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/)
- [Common Type System](https://learn.microsoft.com/en-us/dotnet/standard/base-types/common-type-system)
- [Language independence and the CLS](https://learn.microsoft.com/en-us/dotnet/standard/language-independence)
- [Conversions, casting, and boxing](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions)
- [Method parameters and modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters)
- [Reduce allocations with ref safety](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/)

---

# Part II — Object-Oriented C#

## 5. Type forms ⭐

| Form | Semantics and best fit | Avoid/misuse |
|---|---|---|
| `class` | Reference identity, behavior, mutable/large state, inheritance | Value-like data whose equality must be hand-written |
| `struct` | Value semantics, inline storage, small self-contained values | Large mutable values, inheritance needs, frequent boxing |
| `record class` | Reference type with synthesized value equality, printing, `with` copy | Identity-based entities; assuming deep immutability |
| `record struct` | Value type with synthesized equality and `with` | Large or identity-bearing models |
| `readonly struct` | Value type whose instance state cannot be mutated through `this` | Containing mutable references and calling it deeply immutable |
| `ref struct` | Ref-like, scope-limited value such as `Span<T>` that can safely contain managed refs | Storage in ordinary heap fields; use across unsafe suspension points |
| `abstract class` | Shared state/implementation plus an extensible inheritance contract | Unrelated types or a need for multiple contracts |
| `interface` | Capability contract; multiple implementation; may have default/static members | Shared instance state or fragile “fat” contracts |
| `sealed class` | Closed inheritance, explicit design boundary; can enable optimization | When supported subclassing is part of the contract |
| `static class` | Stateless grouping; cannot be instantiated or inherited | Hidden global mutable state and hard-to-test dependencies |
| `partial` type/member | One logical declaration split across generated/handwritten files | Treating file split as runtime modularity |

Structs inherit from `System.ValueType` and can implement interfaces but cannot inherit another class/struct. Record class inheritance must stay within record classes; record structs cannot inherit. `init` and positional record properties provide **shallow** immutability: a referenced array or list can still mutate.

## 6. The OO principles in interview terms ⭐

- **Encapsulation:** expose invariants through a stable API and hide representation. A public setter is not encapsulation merely because it is a property.
- **Abstraction:** present the capability a caller needs and suppress irrelevant implementation detail.
- **Inheritance:** establish an “is-a” substitutability relationship and reuse/extend base behavior.
- **Composition:** build behavior from owned/collaborating objects. Prefer it when behavior must vary independently or the relationship is not truly substitutable.
- **Polymorphism:** invoke a contract while runtime type selects an override/implementation. Generic parametric polymorphism and overloads are other meanings, but OO interviews usually mean subtype polymorphism.

**Composition versus inheritance.** Inheritance couples derived code to base contracts, initialization, protected surface, and virtual behavior. Composition makes dependency and lifetime explicit, supports independent testing, and avoids fragile base-class changes. Inherit when the base deliberately supports extension and substitutability holds; compose for “has-a/uses-a.”

## 7. Interface versus abstract class ⭐

An interface supports multiple capability contracts and implementation by classes or structs. Modern interfaces can contain default implementations and static abstract members, but they still do not carry per-instance fields. An abstract class supports constructors, instance state, protected members, implementation reuse, and a single base-class lineage.

Choose an interface for a role shared by otherwise unrelated types or a consumer-facing seam. Choose an abstract base when implementations share identity, invariant state, and a controlled evolution model. Default interface methods help evolve interfaces but can make dispatch and conflict resolution harder; they do not turn an interface into a stateful base class.

## 8. Virtual, abstract, override, and new ⭐

- `virtual`: base supplies an implementation that derived classes may override.
- `abstract`: no base implementation; a concrete derived type must implement it. Abstract members are virtual.
- `override`: participates in the same virtual slot; runtime type selects the most-derived override.
- `sealed override`: supplies the final override and prevents further override.
- `new`: hides a member; selection is normally based on the expression’s compile-time type. It does not replace a virtual slot.

```csharp
class Base
{
    public virtual string V() => "Base.V";
    public string N() => "Base.N";
}
class Derived : Base
{
    public override string V() => "Derived.V";
    public new string N() => "Derived.N";
}

Base x = new Derived();
Console.WriteLine(x.V());
Console.WriteLine(x.N());
```

```text
Derived.V
Base.N
```

Conceptually, a virtual call uses the object’s runtime type information and the member’s virtual slot/dispatch mapping to locate the target. Interface calls may use runtime dispatch stubs and caches. The JIT may devirtualize when it can prove the target; this is an optimization, never behavior to depend on.

## 9. Compile-time dispatch and variance ⭐⭐

Overload resolution, extension-member selection, and hidden nonvirtual member selection are compile-time operations. Override/interface dispatch is runtime. `dynamic` postpones binding to runtime.

**Covariance** preserves assignment direction for outputs: `IEnumerable<Dog>` → `IEnumerable<Animal>`. **Contravariance** reverses it for inputs: `Action<Animal>` → `Action<Dog>`. Variance applies to generic interface/delegate type parameters and reference types; ordinary generic classes such as `List<T>` are invariant. Arrays are covariant for historical reasons but enforce runtime stores and can throw `ArrayTypeMismatchException`.

### Interview check

**Basic — Q:** What does `sealed` do?  
**A:** On a class it prevents inheritance; on an override it prevents further overriding of that virtual member.

**Intermediate — Q:** Is a record immutable?  
**A:** Not necessarily. Record class positional properties are init-only by default, record struct positional properties are mutable by default, and referenced objects can still mutate.

**Senior — Q:** Why prefer composition?  
**A:** It avoids forcing an is-a hierarchy, narrows coupling, and lets behavior and lifetime vary behind an explicit dependency.

**Deep Dive — Q:** `new` versus `override`?  
**A:** `override` reuses a virtual slot and dispatches by runtime type. `new` introduces a hiding member selected from the compile-time view.

### Official Microsoft references

- [Classes, structs, and records](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/)
- [Inheritance](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Polymorphism](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/polymorphism)
- [Record types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
- [Covariance and contravariance](https://learn.microsoft.com/en-us/dotnet/standard/generics/covariance-and-contravariance)
- [CoreCLR virtual stub dispatch](https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/virtual-stub-dispatch.md)

---

# Part III — Object Equality

## 10. Identity, value, and structural equality ⭐

| Mechanism | Default meaning | Binding |
|---|---|---|
| `ReferenceEquals(a,b)` | Same reference, or both `null`; boxed value arguments become distinct boxes | Static, cannot be overloaded |
| `object.Equals(a,b)` | Null-safe call into virtual `Equals` | Runtime override |
| `instance.Equals(other)` | Class default identity; struct default field value comparison; type may override | Virtual |
| `==` / `!=` | Operator chosen from compile-time operand types; class default identity, string/record value semantics | Static operator overload resolution |
| `IEquatable<T>.Equals` | Strongly typed equality, used by `EqualityComparer<T>.Default` when available | Generic/virtual interface dispatch |
| Structural equality | Corresponding elements compared through an `IEqualityComparer` | `IStructuralEquatable` on tuples/arrays and related types |

`==` is not universally an alias for `Equals`. A base-typed variable can select a different static operator than expected. Use an explicit comparer when equality policy is domain-specific, especially for string keys.

## 11. The equality/hash contract ⭐

A correct equality relation is reflexive, symmetric, and transitive, and consistent while relevant state is unchanged. If `a.Equals(b)` is true, `a.GetHashCode()` **must equal** `b.GetHashCode()`. Unequal objects may collide. Hash codes are bucket selectors, not unique IDs, persistence keys, stable cross-process values, or security hashes.

For a value-equality type:

1. Prefer immutable equality components.
2. Implement `IEquatable<T>`.
3. Override `Equals(object?)` consistently.
4. Override `GetHashCode`, typically with `HashCode.Combine`.
5. If operators are supplied, make `==`/`!=` consistent.
6. For inheritable classes, equality across runtime types becomes subtle; sealed value objects or records are safer defaults.

```csharp
public sealed class ProductCode : IEquatable<ProductCode>
{
    public ProductCode(string value) => Value = value;
    public string Value { get; }

    public bool Equals(ProductCode? other) =>
        other is not null &&
        StringComparer.OrdinalIgnoreCase.Equals(Value, other.Value);

    public override bool Equals(object? obj) => obj is ProductCode other && Equals(other);
    public override int GetHashCode() => StringComparer.OrdinalIgnoreCase.GetHashCode(Value);
    public static bool operator ==(ProductCode? x, ProductCode? y) => Equals(x, y);
    public static bool operator !=(ProductCode? x, ProductCode? y) => !Equals(x, y);
}
```

### How bad hashing breaks collections ⭐

`Dictionary`/`HashSet` first select a bucket from a hash code, then use equality within candidates. If an equality component changes after insertion, lookup searches using the new hash and might not find the object in its old bucket.

```csharp
sealed class MutableKey
{
    public int Id { get; set; }
    public override bool Equals(object? o) => o is MutableKey k && k.Id == Id;
    public override int GetHashCode() => Id;
}

var key = new MutableKey { Id = 1 };
var set = new HashSet<MutableKey> { key };
key.Id = 2;
Console.WriteLine(set.Contains(key));
```

```text
False
```

The object is still stored, but its current hash routes lookup elsewhere. Never mutate fields used by a key’s equality while it is in a hash collection.

## 12. Record equality ⭐⭐

Records synthesize `IEquatable<T>`, `Equals`, `GetHashCode`, `==`, `!=`, printing, and copying support. Equality includes runtime record type and member equality. It is recursively only as “value-like” as each member: arrays and most mutable collections use reference equality, so two separate arrays with equal elements are not automatically equal. `with` performs a shallow copy.

### Interview check

**Basic — Q:** Must unequal objects have unequal hashes?  
**A:** No. Equal objects must have equal hashes; collisions between unequal objects are permitted.

**Intermediate — Q:** Why implement `IEquatable<T>` for a struct?  
**A:** It supplies strongly typed equality and can avoid boxing/reflection costs of `ValueType.Equals(object)`.

**Senior — Q:** Why are mutable dictionary keys dangerous?  
**A:** A mutation can change the hash/equality relation after bucket placement and make entries unreachable through normal lookup.

**Deep Dive — Q:** Does record equality deeply compare a `List<T>` property?  
**A:** No. Synthesized equality calls the member’s equality; `List<T>` retains reference equality.

### Official Microsoft references

- [Define value equality](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)
- [C# equality comparisons](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/equality)
- [`IEquatable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1?view=net-10.0)
- [`Object.GetHashCode`](https://learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode?view=net-10.0)
- [C# record types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
- [`IStructuralEquatable`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.istructuralequatable?view=net-10.0)

---

# Part IV — Strings

## 13. Representation and immutability ⭐

`string` is the C# alias for `System.String`, a sealed reference type. A string stores a contiguous read-only sequence of UTF-16 code units. `char` is one 16-bit code unit—not necessarily a Unicode character, scalar value, or displayed grapheme. A supplementary scalar (many emoji) needs a surrogate pair. Use `Rune` to process Unicode scalar values and `StringInfo`/text-element APIs where grapheme clusters matter.

Strings are immutable: operations that seem to edit them produce another string or return an existing one. Benefits include safe sharing, stable hashing, and interning; the cost is allocation when repeatedly building changing text.

```csharp
string emoji = "🐂";
Console.WriteLine(emoji.Length);
Console.WriteLine(emoji.EnumerateRunes().Count());
```

```text
2
1
```

`ReadOnlySpan<char>` can provide an allocation-free view over part of a string for synchronous parsing. It does not own or copy the text and cannot be stored in an ordinary class field.

## 14. Equality, ordering, and culture ⭐

String `==` performs ordinal, case-sensitive value equality. Still choose overloads with an explicit `StringComparison` so intent is reviewable:

- `Ordinal` / `OrdinalIgnoreCase`: protocol tokens, identifiers, paths where the platform rules allow it, security-sensitive nonlinguistic data; usually fastest and most predictable.
- `CurrentCulture` variants: user-visible linguistic comparison/sorting.
- `InvariantCulture` variants: stable linguistic handling, not a substitute for ordinal comparison of symbolic data.

Do not normalize via `ToLower()` merely to compare: it allocates, can be culturally wrong, and hides intent. Use the corresponding ignore-case comparer. Persist formatted numbers/dates with invariant culture; display them with the user’s culture.

## 15. Interning ⭐⭐

The runtime maintains an intern pool for canonical string references. Literal interning can allow equal literals to share an instance, but automatic interning is not an identity contract to build business logic on. `String.Intern` adds/finds a canonical reference; `String.IsInterned` tests without adding. Interned references can stay alive for a long time, so manually interning unbounded external data is a memory-retention risk. Always use value comparison for content.

## 16. Building and formatting strings ⭐

| Tool | Use | Performance note |
|---|---|---|
| `+` / `string.Concat` | A small fixed number of pieces | Compiler/runtime can combine efficiently; literals can fold |
| Interpolation | Readable formatting | Modern handlers can avoid boxing and skip work in handler-aware APIs |
| `string.Format` | Composite format or dynamic format strings | Arguments may box; .NET 8 `CompositeFormat` can cache repeated parsing |
| `StringBuilder` | Many/unknown mutations, especially loops | Mutable buffer reduces intermediate strings; capacity can still grow/copy |
| `string.Create` / span formatting | Measured hot paths with known layout | More complex; use only with evidence |

```csharp
var builder = new StringBuilder(capacity: 32);
for (int i = 1; i <= 3; i++)
    builder.Append(i).Append(i == 3 ? "" : ",");
Console.WriteLine(builder.ToString());
```

```text
1,2,3
```

`StringBuilder` is not automatically faster for two or three concatenations, especially when one `Concat` can allocate the final string once. It is useful when repeated concatenation would copy an ever-growing prefix and create many temporary strings. Set a reasonable initial capacity when size is predictable, but avoid oversized retained builders.

Raw string literals (C# 11) reduce escaping for JSON, regex-like text, quotes, and multiline content. UTF-8 literals (`"GET"u8`, C# 11) have type `ReadOnlySpan<byte>` and encode bytes at compile time. They are not `string` constants.

### Interview check

**Basic — Q:** Why is `string` immutable?  
**A:** Its content cannot change after construction; apparent mutations return another value, enabling safe sharing and stable equality/hash behavior.

**Intermediate — Q:** Is `string.Length` a user-perceived character count?  
**A:** No, it counts UTF-16 `char` code units.

**Senior — Q:** When is `StringBuilder` unnecessary?  
**A:** For a small fixed concatenation that the compiler/runtime can combine into one result allocation; measure the real path.

**Deep Dive — Q:** Why not intern every repeated input?  
**A:** Pool entries can extend lifetimes dramatically; unbounded values can trade short-term deduplication for lasting memory growth.

### Official Microsoft references

- [Strings and string literals](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/)
- [Best practices for comparing strings](https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings)
- [Character encoding in .NET](https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-encoding-introduction)
- [`StringBuilder`](https://learn.microsoft.com/en-us/dotnet/api/system.text.stringbuilder?view=net-10.0)
- [`String.Intern`](https://learn.microsoft.com/en-us/dotnet/api/system.string.intern?view=net-10.0)
- [Composite formatting](https://learn.microsoft.com/en-us/dotnet/standard/base-types/composite-formatting)

---

# Part V — Collections

## 17. Choose by access pattern ⭐

Complexities below describe normal BCL behavior; `n` is collection size. “Amortized O(1)” means occasional resize is O(n). Hash operations also include the cost/quality of the comparer.

| Collection | Implementation concept | Lookup/access | Insert/remove | Ordering, memory, thread safety |
|---|---|---:|---:|---|
| `T[]` | Fixed contiguous storage | index O(1), search O(n) | Cannot resize; element shift if managed manually | Lowest overhead, stable order, not synchronized |
| `List<T>` | Resizable contiguous array | index O(1), search O(n) | end amortized O(1); middle O(n) | Cache-friendly; spare capacity; not synchronized |
| `LinkedList<T>` | Doubly linked nodes | search/index O(n) | O(1) with a known node | Per-node allocations/references; stable linked order; not synchronized |
| `Dictionary<K,V>` | Hash table with buckets/entries | average/close O(1), worst O(n) | average O(1), resize O(n) | Unspecified contract for order; capacity overhead; readers only safe with no writes |
| `SortedDictionary<K,V>` | Balanced search tree | O(log n) | O(log n) | Key sorted; node overhead; not synchronized |
| `SortedList<K,V>` | Sorted key/value arrays | binary lookup O(log n) | O(n) shifts | Key sorted, compact memory, good for mostly-read data |
| `HashSet<T>` | Hash table of unique values | average O(1), worst O(n) | average O(1) | No sorted-order contract; not synchronized |
| `SortedSet<T>` | Balanced tree | O(log n) | O(log n) | Sorted unique values; node overhead |
| `Queue<T>` | Circular resizable buffer | ends O(1) amortized | enqueue/dequeue amortized O(1) | FIFO; not synchronized |
| `Stack<T>` | Resizable array | top O(1) | push/pop amortized O(1) | LIFO; not synchronized |
| `PriorityQueue<E,P>` | Array-backed quaternary min-heap | peek O(1) | enqueue/dequeue O(log n) | Lowest priority dequeues first; equal priorities are not FIFO-guaranteed |
| `ConcurrentDictionary<K,V>` | Concurrent hash map | near O(1) typical | thread-safe compound APIs | Fine-grained synchronization; value factories can run more than once |
| `ConcurrentQueue<T>` | Concurrent FIFO | ends effectively O(1) | thread-safe enqueue/dequeue | Snapshot-like enumeration semantics; no external lock required for its operations |
| `ConcurrentStack<T>` | Concurrent LIFO | top effectively O(1) | thread-safe push/pop | Batch operations available |
| `ConcurrentBag<T>` | Unordered, thread-local-friendly bag | no keyed lookup | optimized when same threads add/take | No ordering; cross-thread steals can cost more |
| Immutable collections | Persistent trees/arrays with structural sharing | type-dependent, often O(log n) | returns a new collection | Safe to share; immutable does not mean zero allocation; use builders for batches |

### Array versus List; List versus LinkedList ⭐

Use an array for fixed-size, dense, index-heavy data or an API contract that naturally owns a fixed buffer. Use `List<T>` for the default growable sequence. Prefer `List<T>` over `LinkedList<T>` unless profiling and the algorithm prove frequent insertion/removal through already-known nodes; linked lists lose locality, allocate nodes, and still require O(n) to find a position.

### HashSet versus List ⭐

Use `HashSet<T>` for uniqueness and repeated membership/set algebra. Use `List<T>` for order, duplicates, indexing, and small sequences where hashing overhead is unnecessary. The right comparer is part of correctness.

## 18. Capacity, resizing, and collisions ⭐⭐

A resizable collection has **Count** (used elements) and **Capacity** (space before growth). Growth allocates new backing storage and copies/rebuilds entries, so pre-size when a reliable estimate exists. Do not pre-size wildly: unused capacity consumes memory and large arrays may enter the LOH.

A dictionary computes a hash, maps it to a bucket, then tests candidate keys with equality. Multiple keys can map to one bucket—a collision. Good distribution keeps chains/probes short; adversarial or poor hashing degrades toward O(n). This is why lookup is “usually O(1),” not an unconditional physical constant. Never change a key in a way that changes its hash while stored.

## 19. Concurrency and immutability ⭐⭐

`Dictionary` supports concurrent readers only while no writer modifies it. `Dictionary + lock` is appropriate when several operations must form one larger invariant/transaction. `ConcurrentDictionary` provides scalable thread-safe operations such as `TryAdd`, `TryUpdate`, `GetOrAdd`, and `AddOrUpdate`, but delegate factories execute outside internal locks and may run multiple times; the insertion/update is atomic, not the side effects inside the delegate.

```csharp
var cache = new ConcurrentDictionary<string, Lazy<Widget>>();
Widget value = cache.GetOrAdd(
    "key",
    _ => new Lazy<Widget>(LoadWidget, LazyThreadSafetyMode.ExecutionAndPublication)
).Value;
```

Use immutable collections for published snapshots and sharing without coordination. An update creates a new logical version using structural sharing; it does not mutate the old one. Use a `Builder` for many changes before freezing. Read-only wrappers are not necessarily immutable: another holder may still mutate the underlying collection.

### Version notes

Core collection semantics are stable across .NET 5–10. `PriorityQueue<TElement,TPriority>` was introduced in .NET 6. New LINQ/collection APIs have accumulated, but they do not change the fundamental selection rules.

### Interview check

**Basic — Q:** Why is `List<T>.Add` described as amortized O(1)?  
**A:** Most appends fill an existing slot; occasional capacity growth allocates and copies O(n) elements.

**Intermediate — Q:** SortedList or SortedDictionary?  
**A:** `SortedList` is denser and good when mutations are rare; `SortedDictionary` has O(log n) inserts/removes without array shifts.

**Senior — Q:** Is `GetOrAdd(key, factory)` guaranteed to invoke the factory once?  
**A:** No. Competing calls can execute factories outside locks; one produced value wins.

**Deep Dive — Q:** Why can hash lookup become O(n)?  
**A:** Collision candidates still require equality checks; poor or adversarial distribution can put many keys in the same search path.

### Official Microsoft references

- [Collections and data structures](https://learn.microsoft.com/en-us/dotnet/standard/collections/)
- [Selecting a collection class](https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class)
- [`Dictionary<TKey,TValue>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2?view=net-10.0)
- [`ConcurrentDictionary<TKey,TValue>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2?view=net-10.0)
- [When to use generic collections](https://learn.microsoft.com/en-us/dotnet/standard/collections/when-to-use-generic-collections)
- [`PriorityQueue<TElement,TPriority>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.priorityqueue-2?view=net-10.0)

---

# Part XII — Async and Await

## 43. The model: asynchronous work, not “a new thread” ⭐

`Task` and `Task<T>` represent eventual completion: successful, faulted, or canceled. They do not imply how the operation is implemented. For scalable I/O, the OS/runtime can notify completion without dedicating a blocked managed thread. `async`/`await` composes that completion without blocking the caller and does **not** automatically create a thread.

```text
Caller ──calls──> async method
                    │ runs synchronously until it must suspend
                    ▼
              incomplete awaitable
                    │ completion schedules/invokes continuation
                    ▼
              method resumes and completes returned Task
```

- **I/O-bound:** call the naturally asynchronous API and await it.
- **CPU-bound:** perform the computation where appropriate; in responsive/client or request-orchestration code, `Task.Run` can offload CPU work to the thread pool, but it does not reduce total CPU cost.
- **Server request code:** wrapping already-asynchronous I/O in `Task.Run` wastes scheduling/thread-pool capacity.

## 44. Compiler transformation and await semantics ⭐⭐⭐

An async method is transformed conceptually into a state machine. Locals that must survive suspension become fields. The builder produces the returned task-like value. At each `await`:

1. The awaitable's awaiter is obtained.
2. If it is already complete, execution can continue synchronously.
3. Otherwise, the current state and required locals are saved, a continuation is registered, and control returns to the caller.
4. On completion, the state machine resumes and `GetResult` returns a value or rethrows failure.

This conceptual lowering is the stable interview model. Exact generated fields, boxing, pooling, and JIT optimizations are implementation details.

### Example

```csharp
static async Task<int> ReadLengthAsync(string path, CancellationToken token)
{
    string text = await File.ReadAllTextAsync(path, token);
    return text.Length;
}
```

The `string` result and continuation state survive only if the read is incomplete. The calling thread is not synchronously held while the I/O is outstanding.

## 45. Continuation context, scheduler, and `ConfigureAwait` ⭐⭐

By default, task awaiting attempts to resume through a current `SynchronizationContext`; absent one, scheduling uses the current/default `TaskScheduler` according to await infrastructure. UI contexts often serialize work to the UI thread. Classic ASP.NET had a request context. **ASP.NET Core does not install a custom `SynchronizationContext` by default**, so there is generally no request-thread context for `ConfigureAwait(false)` to avoid.

`await task.ConfigureAwait(false)` says the await need not resume on the captured context. It is useful in reusable library internals that do not depend on a caller's context. It is not a magic performance switch, does not move work to a background thread, does not affect already-running work, and applies to that await—not the whole call tree.

Application code that must update UI state normally allows capture. Library code should expose async APIs and remain context-independent where practical.

`TaskScheduler` controls task scheduling, particularly explicitly scheduled tasks and continuations. It is not interchangeable with `SynchronizationContext`, though await can interact with both.

## 46. Blocking, sync-over-async, and deadlocks ⭐

`.Result`, `.Wait()`, and `.GetAwaiter().GetResult()` all block the current thread until completion.

| Form | Principal extra behavior |
|---|---|
| `await task` | Nonblocking composition; unwraps and rethrows task failure |
| `task.Result` | Blocks; returns result; faults surface through `AggregateException` |
| `task.Wait()` | Blocks; faults surface through `AggregateException` |
| `task.GetAwaiter().GetResult()` | Blocks; rethrows underlying exception rather than aggregate wrapper |

The latter does **not** make blocking safe. A classic deadlock occurs when a context thread blocks on a task whose continuation is waiting to run on that same context. Even with no context (typical ASP.NET Core), blocking consumes threads and can cause thread-pool starvation under load. The robust design is “async all the way” across the call chain.

```csharp
// UI/context-sensitive anti-pattern
string text = DownloadAsync().Result;

// Preferred
string text = await DownloadAsync();
```

## 47. `Task`, `ValueTask`, and `async void` ⭐⭐

`Task` is the default asynchronous return. It supports multiple awaits and multiple consumers after completion. A completed result can sometimes reuse cached task instances; do not assume each call allocates.

`ValueTask`/`ValueTask<T>` can avoid a task allocation when completion is frequently synchronous or when backed by a reusable source. The trade-offs are larger values and stricter consumption: unless documentation states otherwise, await a returned `ValueTask` once, do not concurrently await it, and do not mix `AsTask` with other consumption. Use it only after measurement or when implementing an API pattern that calls for it.

`async void` exists primarily for event handlers. It cannot be awaited, represented in a task graph, or naturally composed; callers cannot observe its completion/failure normally. Prefer `Task` even for methods with no result.

## 48. Cancellation ⭐

Cancellation is cooperative. A `CancellationToken` is a request, not forced thread termination.

```csharp
static async Task ProcessAsync(CancellationToken token)
{
    token.ThrowIfCancellationRequested();
    await StepAsync(token);
}
```

Good API behavior:

- accept a token, often last, and pass it to downstream operations;
- periodically observe it in long CPU loops;
- register callbacks only when needed and dispose registrations whose lifetime is bounded;
- after irreversible side effects, define whether cancellation is still honored, compensated, or ignored;
- use linked token sources to combine caller cancellation with lifetime/timeout signals, and dispose the source;
- throw `OperationCanceledException` with the relevant token so task cancellation is represented correctly.

Cancellation is not failure recovery. A timeout policy can be implemented using a cancellation source or newer timeout APIs, but communicate distinctly whether the caller canceled or a deadline expired.

## 49. Coordination: `WhenAll`, `WhenAny`, and bounded concurrency ⭐

Start independent operations before awaiting `WhenAll` to enable concurrency; do not parallelize dependencies.

```csharp
Task<User> userTask = GetUserAsync(id, token);
Task<Order[]> orderTask = GetOrdersAsync(id, token);

await Task.WhenAll(userTask, orderTask);
return (await userTask, await orderTask);
```

`WhenAll` does not create threads and does not serialize the inputs. It returns a task complete when all finish. An empty input completes immediately.

`WhenAny` completes when one input finishes, including fault/cancellation. You must still await the returned winning task to observe its result/failure, and decide what happens to losers. `WhenAny` does not cancel them.

Creating one task per item can overwhelm downstream services. Bound concurrency with `Parallel.ForEachAsync`, `SemaphoreSlim`, Channels, or a worker design, and honor the actual bottleneck.

## 50. `TaskCompletionSource<T>` ⭐⭐⭐

`TaskCompletionSource<T>` adapts callback/event-based completion into a task without scheduling an operation itself. The producer calls a `TrySet...` method; consumers await `.Task`.

```csharp
var source = new TaskCompletionSource<int>(
    TaskCreationOptions.RunContinuationsAsynchronously);

void Complete(int value) => source.TrySetResult(value);
```

Prefer `TrySetResult`, `TrySetException`, and `TrySetCanceled` when racing completions are possible. `RunContinuationsAsynchronously` prevents arbitrary consumer continuations from running inline inside the producer's critical path, reducing reentrancy and lock-coupling surprises. Still document exactly-once completion and unregister callbacks/cancellation registrations.

## 51. Async streams ⭐

`IAsyncEnumerable<T>` represents asynchronously produced sequences. `await foreach` awaits successive moves; it does not materialize the sequence.

```csharp
static async IAsyncEnumerable<int> CountAsync(
    [EnumeratorCancellation] CancellationToken token = default)
{
    for (int i = 0; i < 3; i++)
    {
        await Task.Delay(10, token);
        yield return i;
    }
}

await foreach (int item in CountAsync(token))
    Console.WriteLine(item);
```

**Output**

```text
0
1
2
```

The compiler supplies async-iterator machinery. Dispose enumeration with `await foreach`/`await using` semantics. Cancellation commonly flows through `.WithCancellation(token)` and `[EnumeratorCancellation]` on the iterator parameter.

## 52. Async design checklist ⭐

- Use the `Async` suffix for task-returning operations by convention.
- Avoid hidden concurrency: state whether calls may overlap and whether results preserve order.
- Never hold a monitor-style `lock` across `await`; use an async-compatible gate if serialization is required.
- Do not pass `CancellationToken.None` when a caller token should flow.
- Observe all tasks; intentional fire-and-forget needs explicit lifetime and exception handling.
- Do not use `Task.Run` merely to make a synchronous I/O API look scalable.
- Use `Task.Yield` only for its documented scheduling behavior, not as a general “make async” or performance tool.
- Prefer `Task.Delay` over `Thread.Sleep` in asynchronous workflows.

### Version notes

The core task/await model is stable across .NET 5–10. Async streams predate this range and remain current. API overloads and runtime allocation/scheduling optimizations have expanded; they do not alter the semantic rules. .NET 8 added `ConfigureAwaitOptions` overloads for more explicit configuration. Measure on the deployed runtime rather than inheriting historical allocation assumptions.

### Interview check

**Basic — Q:** Does `async` start a new thread?  
**A:** No. It enables suspension/composition around awaitables; the underlying operation determines resource/thread use.

**Intermediate — Q:** CPU-bound versus I/O-bound async strategy?  
**A:** Await naturally asynchronous I/O; use explicit thread-pool offload for CPU work only where responsiveness/orchestration warrants it.

**Senior — Q:** Why can `.Result` deadlock?  
**A:** It can block a context thread while the awaited continuation is queued back to that same context; without a context it still harms scalability.

**Deep Dive — Q:** Why use `RunContinuationsAsynchronously` with `TaskCompletionSource`?  
**A:** It decouples producer completion from inline consumer continuation execution, reducing reentrancy and accidental execution under producer locks.

### Official Microsoft references

- [Asynchronous programming with async and await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Asynchronous programming scenarios](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios)
- [`Task`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task?view=net-10.0)
- [`ValueTask<TResult>`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.valuetask-1?view=net-10.0)
- [CA2012: Use ValueTasks correctly](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2012)
- [`TaskCompletionSource<TResult>`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcompletionsource-1?view=net-10.0)
- [Task cancellation](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-cancellation)
- [Cancellation in managed threads](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)
- [`Task.WhenAll`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall?view=net-10.0)
- [`Task.WhenAny`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany?view=net-10.0)
- [Generate and consume async streams](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream)
- [`Task.ConfigureAwait`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.configureawait?view=net-10.0)
- [ASP.NET Core SynchronizationContext clarification](https://github.com/dotnet/aspnetcore/issues/31132)

---

# Part XIII — Multithreading and Concurrency

## 53. Execution abstractions ⭐

| Abstraction | What it represents | Senior guidance |
|---|---|---|
| `Thread` | A managed view of an OS thread | Use for rare dedicated-thread requirements; expensive and manually managed |
| Thread pool | Reusable worker threads managed by the runtime | Avoid long blocking work that starves workers; do not change pool settings casually |
| `Task` / TPL | A composable unit of eventual work/completion | Preferred orchestration abstraction; a task is not necessarily a thread |
| `Parallel.For` / `ForEach` | Synchronous data parallelism | CPU-bound independent iterations; caller participates and blocks until completion |
| `Parallel.ForEachAsync` | Bounded asynchronous iteration | Async delegates and controlled parallelism; default max concurrency is processor count |
| PLINQ | Parallel query execution over in-memory sequences | Opt-in with `AsParallel`; ordering/merge overhead can exceed gains |

Thread-pool threads are background threads. The pool adjusts worker availability; flooding it with blocking calls delays unrelated work and can appear as latency spikes. A dedicated thread can be justified for long-running blocking integration, apartment affinity, or specialized priority—but document lifecycle and failure behavior.

## 54. Correctness vocabulary ⭐

- A **race condition** makes correctness depend on timing/interleaving.
- A **data race** involves unsynchronized conflicting access to shared memory.
- A **critical section** accesses a shared invariant that must be coordinated.
- **Atomicity** means an operation appears indivisible relative to relevant observers.
- **Visibility/order** describe when writes become observable and what reordering is permitted.
- A **deadlock** is a cycle of waits with no possible progress.
- **Livelock** performs work/retries but makes no useful progress.
- **Starvation** indefinitely denies a participant CPU or a resource.
- **Thread-safe** is incomplete unless scope is stated: which operations, combinations, callbacks, enumeration, and disposal?

Thread safety is a design property, not something obtained by replacing one collection. Protect invariants, not individual statements.

## 55. `lock`, `System.Threading.Lock`, and `Monitor` ⭐

For a reference expression other than `System.Threading.Lock`, `lock (gate) { body }` lowers conceptually to `Monitor.Enter` in a `try` and `Monitor.Exit` in `finally`. Monitor locks are mutually exclusive and reentrant for the owning thread. Lock a private, dedicated object—never `this`, a `Type`, or an interned/public string that unrelated code could also lock.

From C# 13/.NET 9, when the compile-time type is `System.Threading.Lock`, `lock` uses its scope-based API conceptually like:

```csharp
using (gate.EnterScope())
{
    // critical section
}
```

```csharp
private readonly Lock _gate = new();
private int _count;

public void Increment()
{
    lock (_gate) _count++;
}
```

C# warns when a known `Lock` is converted to another type and then used with `lock`, because that changes semantics. Do not `await` in a `lock` body. `Monitor` exposes capabilities such as `TryEnter`, `Wait`, `Pulse`, and `PulseAll`; use them only with an explicit condition-loop protocol because wakeups and condition changes require rechecking under the lock.

## 56. Synchronization primitives ⭐⭐

| Primitive | Capacity / scope | Important points |
|---|---|---|
| `Mutex` | One owner; can be named/cross-process | Kernel-backed, thread-affine ownership, reentrant; abandoned mutex must be handled |
| `Semaphore` | Counted; can be named/cross-process | Kernel synchronization; limits concurrent entrants rather than protecting one owner |
| `SemaphoreSlim` | Counted, in-process | Lightweight, supports `WaitAsync`; no guaranteed FIFO ordering; not thread-affine |
| `ReaderWriterLockSlim` | Multiple readers or one writer | Useful only when measured read concurrency outweighs complexity; upgradeable mode avoids common upgrade deadlocks |
| `SpinLock` | Busy-wait mutual exclusion | Specialized short waits; consumes CPU and is a mutable struct that must not be copied |

### Async gate example

```csharp
private readonly SemaphoreSlim _gate = new(1, 1);

public async Task UpdateAsync(CancellationToken token)
{
    await _gate.WaitAsync(token);
    try { await PersistAsync(token); }
    finally { _gate.Release(); }
}
```

Always release only after a successful wait. A semaphore with count one can serialize async operations, but unlike `lock` it is not reentrant and does not enforce release by the acquiring thread.

## 57. `Interlocked`, `volatile`, and memory barriers ⭐⭐

`Interlocked` performs atomic read-modify-write operations such as increment, exchange, and compare-exchange and provides appropriate memory ordering. It is ideal for a single counter/reference transition or as a building block for carefully designed lock-free algorithms.

```csharp
if (Interlocked.CompareExchange(ref _initialized, 1, 0) == 0)
    InitializeOnce();
```

`volatile` restricts certain compiler/runtime reordering and gives acquire/release-like visibility for reads/writes of supported field types. It does **not** make compound actions such as `_count++` atomic and does not protect multi-field invariants. Prefer `Interlocked`, `lock`, or higher-level primitives for coordination.

`Thread.MemoryBarrier` prevents memory operations around the call from being reordered across it. It is a low-level tool; higher-level synchronization already supplies required ordering. Using barriers without a proven algorithm can still leave code incorrect.

## 58. Concurrent collections versus locking ⭐

`ConcurrentDictionary<TKey,TValue>` coordinates individual and compound APIs designed by the type, but not an arbitrary sequence of caller operations. Its delegates for `GetOrAdd`/`AddOrUpdate` execute outside internal locks and may run more than once. Factories therefore should be side-effect-safe; use `Lazy<T>` or another explicit scheme when single initialization matters.

`Dictionary<TKey,TValue>` plus one lock can be simpler when several keys/structures form one invariant or when a transaction-like operation is required. Never perform unbounded callbacks or I/O while holding the lock.

```csharp
// Race: ContainsKey and assignment are not one operation.
if (!map.ContainsKey(key)) map[key] = Create();

// Atomic dictionary operation, but Create may be invoked speculatively more than once.
Value value = concurrent.GetOrAdd(key, _ => Create());
```

## 59. Parallelism traps ⭐⭐

Parallel loops are not automatically faster. Partitioning, synchronization, cache contention, delegate calls, ordering, and merging are overhead. Iterations must be independent unless access is coordinated. Do not write to non-thread-safe collections from parallel bodies. Prefer thread-local aggregation followed by a merge.

PLINQ does not preserve source order unless `AsOrdered` is requested, which may cost performance. `ForAll` can avoid a merge when side effects are safely parallel. Nested parallelism and tiny work items often oversubscribe/overhead the workload.

### Deadlock prevention checklist

- Establish and follow a global lock order.
- Keep critical sections bounded and callback-free.
- Avoid waiting synchronously while holding a lock.
- Prefer timeouts/cancellation when a wait can legitimately fail.
- Do not mix monitor locks and async waits without a deliberate protocol.
- Capture dumps/traces when diagnosing; timing logs alone may perturb the issue.

### Version notes

`.NET 9 / C# 13` introduced the dedicated `System.Threading.Lock` language path; it remains the preferred lock object on .NET 10 for new code. `Parallel.ForEachAsync` arrived in .NET 6. Existing monitor-based `lock` behavior remains supported across .NET 5–10.

### Interview check

**Basic — Q:** Task versus thread?  
**A:** A task models eventual work/completion; a thread is an execution resource. Many tasks can share threads, and I/O tasks may need none while pending.

**Intermediate — Q:** `volatile int` makes `count++` safe—true?  
**A:** False. Increment is read-modify-write and needs `Interlocked.Increment` or synchronization.

**Senior — Q:** `lock` versus `SemaphoreSlim(1,1)`?  
**A:** `lock` is synchronous, thread-owned/reentrant monitor-style exclusion; `SemaphoreSlim` supports async waits, is count-based and not thread-affine/reentrant.

**Deep Dive — Q:** What changed for `lock` in C# 13?  
**A:** A compile-time `System.Threading.Lock` operand uses its ref-struct scope API; other reference operands retain Monitor-based lowering.

### Official Microsoft references

- [Threading objects and features](https://learn.microsoft.com/en-us/dotnet/standard/threading/threading-objects-and-features)
- [The managed thread pool](https://learn.microsoft.com/en-us/dotnet/standard/threading/the-managed-thread-pool)
- [`lock` statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/lock)
- [`Monitor`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.monitor?view=net-10.0)
- [Overview of synchronization primitives](https://learn.microsoft.com/en-us/dotnet/standard/threading/overview-of-synchronization-primitives)
- [Semaphore and `SemaphoreSlim`](https://learn.microsoft.com/en-us/dotnet/standard/threading/semaphore-and-semaphoreslim)
- [`Interlocked`](https://learn.microsoft.com/en-us/dotnet/api/system.threading.interlocked?view=net-10.0)
- [`volatile`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile)
- [Data parallelism with TPL](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/data-parallelism-task-parallel-library)
- [Potential pitfalls in data and task parallelism](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-in-data-and-task-parallelism)
- [PLINQ introduction](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq)

---

# Part XIV — LINQ

## 60. Enumeration model ⭐

`IEnumerable<T>.GetEnumerator()` produces an `IEnumerator<T>`. Repeated `MoveNext()` calls advance the cursor and expose `Current`; disposal releases enumeration resources. `foreach` is compiler syntax over this pattern (and pattern-based alternatives).

An iterator method containing `yield return` is transformed into an iterator state machine. Calling it normally creates/returns an enumerable representation; the body executes as enumeration advances, not necessarily at the call.

```csharp
static IEnumerable<int> Evens(IEnumerable<int> source)
{
    foreach (int item in source)
        if ((item & 1) == 0)
            yield return item;
}
```

## 61. Deferred, immediate, streaming, and buffering ⭐

- **Deferred execution:** query construction records operators; work occurs upon enumeration. Most sequence-returning operators are deferred.
- **Immediate execution:** terminal/materialization operators such as `ToList`, `ToArray`, `Count`, and `First` enumerate now.
- **Streaming:** an operator can yield an item without first consuming all input (`Where`, `Select`).
- **Buffering:** an operator needs substantial/all input before its first result (`OrderBy`; commonly `GroupBy` organizes groups).

Deferred execution observes the source at enumeration time and repeats work/side effects on every enumeration.

### Example: multiple enumeration

```csharp
IEnumerable<int> Query()
{
    Console.Write("E");
    yield return 1;
    yield return 2;
}

var query = Query().Where(x => x > 0);
Console.Write(query.Count());
Console.Write(query.Count());
```

### Output

```text
E2E2
```

### Explanation

Construction does not enumerate. Each `Count()` re-runs the iterator and predicate. This can duplicate database/network work, produce inconsistent snapshots, repeat side effects, or fail for one-shot sources. Materialize once when a stable snapshot/reuse is intended.

## 62. Operator map and cost ⭐

Assume an in-memory source of `n` elements and ordinary comparer behavior. Provider-backed queries may have entirely different plans.

| Operators | Semantics / typical cost notes |
|---|---|
| `Where`, `Select` | Deferred streaming; O(n) to fully enumerate |
| `SelectMany` | Flattens nested sequences; cost is proportional to produced/visited elements |
| `Aggregate` | Immediate fold; O(n), empty behavior depends on overload |
| `GroupBy` | Deferred but buffers/builds lookup when enumerated; O(n) expected hashing plus storage |
| `Join`, `GroupJoin` | Correlate keys; Enumerable implementation builds lookup for inner sequence |
| `OrderBy`, `ThenBy` | Stable ordering; buffers then sorts, generally O(n log n) |
| `Distinct`, `DistinctBy` | Deferred set semantics; expected O(n) hashing and O(n) auxiliary storage |
| `Union`, `Intersect`, `Except` | Set operations using equality; expected linear hashing characteristics |
| `Any`, `All` | Short-circuit; may inspect only a prefix |
| `First`, `FirstOrDefault` | Stop at first match; default can be ambiguous |
| `Single`, `SingleOrDefault` | Must detect a second match; validates cardinality |
| `Count` | Uses cheap count when exposed, otherwise enumerates; predicate overload visits source |
| `Contains` | Uses collection/provider optimization where available, otherwise linear equality scan |
| `ToList`, `ToArray`, `ToDictionary` | Immediate materialization; dictionary throws on duplicate keys |
| `Chunk` | Produces arrays of at most the requested size; final chunk may be smaller |
| `Zip` | Pairs sequences until one ends (overload-dependent result form) |

`FirstOrDefault` does not distinguish “no element” from “first element equals default”; use `Any`, a nullable/domain result, or an explicit result type when absence matters. `Single` expresses and checks an invariant, but it is not a faster version of `First`.

## 63. `IEnumerable<T>` versus `IQueryable<T>` ⭐

`IEnumerable<T>` operators execute .NET delegates over objects. `IQueryable<T>` carries an expression tree plus an `IQueryProvider`; the provider translates supported expression shapes into another query language/execution plan.

```csharp
Func<User, bool> predicate = u => u.IsActive;                  // executable code
Expression<Func<User, bool>> expression = u => u.IsActive;    // code represented as data
```

Calling `AsEnumerable()` changes subsequent binding to Enumerable operators; it does not itself materialize. Calling `ToList()` executes/materializes. Arbitrary .NET methods inside an `IQueryable` expression may be untranslatable or have provider-specific semantics. Keep provider queries within supported operations, inspect generated behavior using that provider's official tooling, and switch deliberately to in-memory evaluation.

## 64. Mutation and enumeration traps ⭐⭐

Most general-purpose mutable collections invalidate enumerators when structurally modified and throw. Deferred queries can also observe mutations made before enumeration. Enumerator thread safety is not implied by collection read safety. Preserve a snapshot or synchronize the entire enumerate/mutate protocol when consistency matters.

Custom iterator cleanup belongs in `finally`/`using`; enumerator disposal runs when enumeration completes or exits early through a well-formed consumer.

### Version notes

`DistinctBy` and `Chunk` were added in .NET 6. New overloads/operators appeared through later releases; the fundamental enumerable/queryable and deferred-execution models are unchanged across .NET 5–10.

### Interview check

**Basic — Q:** When does `source.Where(...)` execute?  
**A:** Normally when enumerated, and again on each new enumeration.

**Intermediate — Q:** `First` versus `Single`?  
**A:** `First` needs at least one and stops; `Single` asserts exactly one and must detect a second.

**Senior — Q:** Why can multiple enumeration be dangerous?  
**A:** It can repeat expensive/remote work and side effects, observe different data, or consume a one-shot sequence twice.

**Deep Dive — Q:** Why can an `IQueryable` lambda not call every valid C# method?  
**A:** It is represented as an expression tree for a provider; the provider can translate only supported nodes/methods into its target system.

### Official Microsoft references

- [LINQ overview](https://learn.microsoft.com/en-us/dotnet/csharp/linq/)
- [Deferred execution and lazy evaluation](https://learn.microsoft.com/en-us/dotnet/standard/linq/deferred-execution-lazy-evaluation)
- [`yield` statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)
- [`IEnumerable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=net-10.0)
- [`Enumerable`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable?view=net-10.0)
- [`Queryable`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable?view=net-10.0)
- [Expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)

---

# Part XV — Nullability

## 65. Nullable value and reference types ⭐

`T?` for a value type is `Nullable<T>` and adds `HasValue`/`Value` semantics. Boxing a nullable with no value produces `null`; boxing one with a value boxes the underlying `T`, not a `Nullable<T>` object.

Nullable reference types (NRT) are compile-time annotations plus flow analysis. Enabling them does not create a new CLR reference type, inject runtime checks, or prevent reflection/older code from passing `null`.

```csharp
#nullable enable
string required = "ok";
string? optional = Find();

if (optional is not null)
    Console.WriteLine(optional.Length); // flow state: not-null
```

Public annotations are part of the API contract for compiler-aware consumers. Treat warnings as correctness signals; blanket suppression defeats the feature.

## 66. Operators and patterns ⭐

- `x?.Member` short-circuits member access when `x` is null; it is not atomic across threads.
- `a ?? b` evaluates `b` only if `a` is null.
- `a ??= b` assigns only if `a` is null.
- `x!` changes compiler null-state for that expression; it performs no runtime validation.
- `x is null` / `is not null` use pattern semantics and are not redirected through overloaded `==`.

```csharp
string? value = GetValue();
int length = value?.Length ?? 0;
```

Use `!` only where an invariant exists that analysis cannot express. Prefer validation or annotations when the uncertainty belongs at an API boundary.

## 67. Nullable analysis attributes ⭐⭐

These attributes refine contracts the ordinary type annotation cannot express:

| Attribute | Contract |
|---|---|
| `[AllowNull]` | Non-null return/reads, but callers may supply null |
| `[DisallowNull]` | Nullable return/reads, but callers should not supply null |
| `[MaybeNull]` | Return may be null even if type parameter/annotation otherwise suggests non-null |
| `[NotNull]` | Return or `ref`/`out` value is non-null on normal return |
| `[NotNullWhen(bool)]` | Parameter is non-null when method returns specified bool |
| `[MaybeNullWhen(bool)]` | Parameter may be null when method returns specified bool |
| `[NotNullIfNotNull("p")]` | Return is non-null when named argument is non-null |
| `[MemberNotNull(...)]` | Listed members are non-null after method returns |
| `[MemberNotNullWhen(bool,...)]` | Listed members are non-null for specified result |
| `[DoesNotReturn]` / `[DoesNotReturnIf(bool)]` | Flow does not continue under stated condition |

### Example

```csharp
static bool TryNormalize(
    string? input,
    [NotNullWhen(true)] out string? result)
{
    result = input?.Trim();
    return !string.IsNullOrEmpty(result);
}

if (TryNormalize(raw, out string? normalized))
    Console.WriteLine(normalized.Length); // known non-null
```

Attributes should truthfully describe every normal return path; a false annotation silences warnings while creating unsound callers.

## 68. Migration and generic nuance ⭐⭐

Nullable context has separate annotation and warning settings. When migrating, enable warnings deliberately, annotate public boundaries, initialize invariants, and then fix/suppress narrow cases with justification. `required`, constructors, `MemberNotNull`, and guard clauses can align initialization with analysis.

For unconstrained `T`, `T?` means “possibly default” under modern nullable analysis and is governed by the generic context; it is not always `Nullable<T>`. Constraint choice (`class`, `class?`, `notnull`, `struct`) determines valid arguments and annotation meaning.

### Interview check

**Basic — Q:** Do NRT annotations stop null at runtime?  
**A:** No. They produce metadata/compile-time analysis and warnings; runtime reference representation is unchanged.

**Intermediate — Q:** What does the null-forgiving operator do?  
**A:** It suppresses/adjusts nullable analysis for an expression and does nothing at runtime.

**Senior — Q:** When is `NotNullWhen` useful?  
**A:** For try-pattern methods whose boolean result proves that an argument or `out` value is non-null.

**Deep Dive — Q:** What happens when `(int?)null` is boxed?  
**A:** The result is a null reference; a non-null nullable boxes its underlying `int`.

### Official Microsoft references

- [Nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references)
- [Nullable value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)
- [Nullable static analysis attributes](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis)
- [Update a codebase with nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-migration-strategies)
- [The null-forgiving operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving)

---

# Part XVI — Modern C# Features

## 69. Version map and durable use cases ⭐

| C# version | Runtime generation | High-value interview features |
|---:|---:|---|
| 9 | .NET 5 | Records, `init`, top-level statements, target-typed `new`, relational/logical patterns |
| 10 | .NET 6 | Record structs, global usings, file-scoped namespaces, extended property patterns, interpolated-string handlers |
| 11 | .NET 7 | Required members, raw and UTF-8 literals, list patterns, generic math/static abstract interface members, `scoped`, ref fields |
| 12 | .NET 8 | Primary constructors, collection expressions, inline arrays, optional lambda parameters, `ref readonly` parameters |
| 13 | .NET 9 | `params` collections, `System.Threading.Lock`, ref-like interface/generic improvements, partial properties/indexers |
| 14 | .NET 10 | Extension members, `field`, null-conditional assignment, implicit Span conversions, more partial members |

The mapping is the SDK's default relationship, not a promise that every language feature runs against any earlier framework. Pin language versions for reproducibility.

## 70. Data-shaping features ⭐

**Records (C# 9; record structs C# 10)** provide synthesized value equality, printing, deconstruction, and nondestructive mutation with `with`. They suit value-oriented models, not entities whose identity is independent of current data. `with` performs a shallow copy unless members themselves implement deeper semantics.

**`init` (C# 9)** permits assignment during object initialization while preventing ordinary later assignment. **`required` (C# 11)** tells callers that a member must be initialized, subject to constructors marked with appropriate contracts; it is compile-time enforcement, not automatic runtime validation.

```csharp
public sealed record Request
{
    public required string Id { get; init; }
}
```

**Primary constructors (C# 12)** put parameters on class/struct declarations. Parameters are in scope throughout the type but are not automatically properties (record positional parameters are different). The compiler captures a parameter only when needed by instance members.

## 71. Concise program structure ⭐

- **Top-level statements (C# 9):** compiler supplies an entry point; one compilation unit may contain them and they precede namespace/type declarations.
- **Global usings (C# 10):** apply across the compilation; implicit SDK usings are project configuration, not the same feature.
- **File-scoped namespace (C# 10):** one namespace declaration applies to the rest of the file and reduces nesting.
- **Target-typed `new` (C# 9):** derives constructed type from context; avoid where the type becomes obscure.

## 72. Pattern matching and switch expressions ⭐

Modern patterns express type tests, deconstruction, property/positional/list shape, relational comparisons, and logical composition. They are compiled semantics, not reflection.

```csharp
static string Classify(int[] values) => values switch
{
    [] => "empty",
    [< 0, ..] => "starts negative",
    [1, 2, ..] => "prefix 1,2",
    _ => "other"
};
```

Switch expressions should be exhaustive. The compiler warns when it can prove missing cases, but open hierarchies and unusual inputs still demand a fallback. Property patterns safely reject null before nested matching. Logical `and`, `or`, `not` patterns require attention to binding; add parentheses for clarity.

## 73. Literals and collection construction ⭐

**Raw string literals (C# 11)** reduce escaping and support multiline indentation rules. Dollar count controls interpolation delimiter depth. **UTF-8 literals (C# 11)** append `u8` and produce `ReadOnlySpan<byte>` UTF-8 data; they are not `string`.

```csharp
string json = """
    { "name": "Ada" }
    """;

ReadOnlySpan<byte> header = "HTTP/1.1"u8;
int[] numbers = [1, 2, 3]; // collection expression, C# 12
```

Collection expressions are target-typed and can use spread elements (`[..source]`). Their concrete construction strategy depends on the target and compiler rules; do not assume every expression creates a `List<T>` or has identical allocation behavior.

## 74. Generic math and static abstract interface members ⭐⭐

C# 11 permits static abstract/virtual interface members. .NET generic-math interfaces such as `INumber<TSelf>` allow algorithms to use operators and identities through constraints.

```csharp
static T Sum<T>(ReadOnlySpan<T> values) where T : INumber<T>
{
    T total = T.Zero;
    foreach (T value in values) total += value;
    return total;
}
```

This provides compile-time numeric abstraction without `dynamic`. Use the narrowest numeric interface required by the algorithm.

Default interface methods (C# 8, before this version window) allow an interface member implementation, mainly supporting API evolution. They do not turn interfaces into stateful base classes; classes do not inherit interface members as ordinary class members, and resolution rules can become complex across interface diamonds.

## 75. C# 13 features on .NET 9+ ⭐⭐

- **`params` collections:** the final parameter can be a supported collection/span type, not only an array. Overload resolution and construction costs depend on the target type.
- **`System.Threading.Lock`:** a dedicated efficient lock type recognized by the `lock` statement.
- Ref-like types can implement interfaces; `allows ref struct` lets generic code opt into ref-like arguments while obeying safety rules.
- `ref` locals and ref-like types are permitted in async/iterator methods where they do not cross an `await`/`yield` boundary.
- Partial properties and indexers extend source-generator-friendly declaration/implementation separation.

## 76. C# 14 features on .NET 10 ⭐⭐

**Extension members** expand extension declarations beyond classic methods, including extension properties and static members through extension blocks. Use them for discoverable operations closely related to a receiver when you cannot modify the type; they remain static members resolved at compile time, not virtual members injected into the type.

```csharp
public static class EnumerableExtensions
{
    extension<T>(IEnumerable<T> source)
    {
        public bool IsEmpty => !source.Any();
    }
}
```

**Field-backed properties** let an accessor use the contextual `field` keyword for the compiler-generated backing field. A real member named `field` may need `@field` to disambiguate.

```csharp
public string Name
{
    get;
    set => field = string.IsNullOrWhiteSpace(value)
        ? throw new ArgumentException("Name is required.")
        : value;
}
```

**Null-conditional assignment** permits assignment through `?.`/`?[]`; the right side is evaluated only when the receiver is non-null. Compound assignments are supported, but increments/decrements are not.

```csharp
customer?.Name = Normalize(raw);
```

Other C# 14 additions include implicit conversions between arrays and spans in more scenarios, `nameof(List<>)` for unbound generic types, parameter modifiers on simple lambda parameters, partial instance constructors/events, and user-defined compound assignment operators.

Interceptors are not presented here as a normal stable C# 14 application feature: official documentation describes them as experimental and subject to change. Do not make production/interview claims beyond the currently documented experimental contract.

### Interview check

**Basic — Q:** Is a primary-constructor parameter automatically a property?  
**A:** No for ordinary classes/structs; positional record parameters have separate synthesized-member behavior.

**Intermediate — Q:** What type is `"abc"u8`?  
**A:** UTF-8 byte data exposed as `ReadOnlySpan<byte>`, not a `string`.

**Senior — Q:** Are extension members dynamically dispatched?  
**A:** No. Like extension methods, they are statically declared and selected through compile-time binding rules.

**Deep Dive — Q:** Why are static abstract interface members important?  
**A:** They allow constrained generic code to invoke type-level operations/operators without runtime `dynamic` binding, enabling generic math.

### Official Microsoft references

- [C# version history](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history)
- [What's new in C# 14](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14)
- [What's new in C# 13](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13)
- [What's new in C# 12](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12)
- [What's new in C# 11](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11)
- [What's new in C# 10](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10)
- [What's new in C# 9](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9)
- [Records](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
- [Pattern matching](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)
- [Generic math](https://learn.microsoft.com/en-us/dotnet/standard/generics/math)
- [Extension members](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-members)

---

# Part XVII — Reflection and Metadata

## 77. Metadata model ⭐

A managed assembly contains CIL plus metadata describing assemblies, modules, types, members, signatures, references, and custom attributes. `Assembly`, `Module`, `Type`, and `MemberInfo` (`MethodInfo`, `PropertyInfo`, and others) expose that model at runtime.

```csharp
Type type = typeof(Dictionary<string, int>);
Console.WriteLine(type.IsGenericType);                  // True
Console.WriteLine(type.GetGenericTypeDefinition());    // Dictionary`2[TKey,TValue]
foreach (Type argument in type.GetGenericArguments())
    Console.WriteLine(argument);
```

`typeof(T)` and `obj.GetType()` acquire type information without name-based lookup. `Assembly.GetType`/`Type.GetType` name resolution depends on qualification and loading context; avoid assuming every type name is globally unique or loaded.

## 78. Invocation, construction, and generics ⭐⭐

`MethodInfo.Invoke` performs runtime access, argument checks/conversion, and wraps a target exception in `TargetInvocationException`. `Activator.CreateInstance` constructs types selected at runtime. These are valuable for frameworks and tooling, but normal direct calls/generic factories provide compile-time checking and usually lower overhead.

Open generic types contain parameters (`List<>`); closed types supply arguments (`List<int>`). Use `MakeGenericType`/`MakeGenericMethod` only after validating generic definitions and constraints; errors otherwise move from compile time to runtime.

Reflection-heavy hot paths should cache correctly scoped metadata or create strongly typed delegates where appropriate. Cache keys must account for type/load context and collectible assemblies; careless static caches can prevent unloading.

## 79. Trimming and source-generation implications ⭐⭐

Reflection can make members appear unused to static analysis. Trimming and Native AOT therefore require analyzable patterns, annotations such as `DynamicallyAccessedMembers`, generated metadata, or explicit preservation. Suppressing a trim warning without proving the required members remain is unsafe.

Source generators can inspect compile-time symbols and emit ordinary source so runtime code avoids discovery/reflection. They cannot inspect runtime-only object state. Prefer reflection for genuine runtime dynamism; prefer generated/direct code when the set of types is known at compilation, startup/performance matters, or AOT/trimming compatibility is required.

### Interview check

**Basic — Q:** `typeof(T)` versus `obj.GetType()`?  
**A:** `typeof` names a compile-time type; `GetType` returns an object's actual runtime type.

**Intermediate — Q:** Why is reflective invocation slower and less safe?  
**A:** Lookup, runtime validation, indirect invocation, and exception wrapping replace direct compile-time binding.

**Senior — Q:** Why can reflection break trimming?  
**A:** Static analysis may not see which dynamically selected members are required and can remove them.

**Deep Dive — Q:** Can a static `Type` cache prevent plugin unload?  
**A:** Yes. Strong references to types/assemblies from a collectible `AssemblyLoadContext` can keep its graph alive.

### Official Microsoft references

- [Reflection overview](https://learn.microsoft.com/en-us/dotnet/fundamentals/reflection/reflection)
- [Viewing type information](https://learn.microsoft.com/en-us/dotnet/fundamentals/reflection/viewing-type-information)
- [Metadata and self-describing components](https://learn.microsoft.com/en-us/dotnet/standard/metadata-and-self-describing-components)
- [`MethodInfo.Invoke`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.methodinfo.invoke?view=net-10.0)
- [`Activator`](https://learn.microsoft.com/en-us/dotnet/api/system.activator?view=net-10.0)
- [Prepare libraries for trimming](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/prepare-libraries-for-trimming)

---

# Part XVIII — Dynamic

## 80. `var`, `object`, and `dynamic` ⭐

| Form | Compile-time type | Binding/checking |
|---|---|---|
| `var x = expr` | Inferred once from `expr`; still static | Normal compile-time member/overload checks |
| `object x = expr` | `object` | Only `object` members statically available; cast/pattern needed for more |
| `dynamic x = expr` | `dynamic` (represented as `object` plus metadata) | Operations deferred to runtime binder |

```csharp
var a = "hello";       // static type string
object b = "hello";    // static type object
dynamic c = "hello";   // runtime-bound operations

Console.WriteLine(a.Length);
Console.WriteLine(((string)b).Length);
Console.WriteLine(c.Length);
```

All print `5`, but reach it through different binding paths. `var` is not dynamic and cannot be used without an initializer (except contextual constructs with separate rules).

## 81. Runtime binding and traps ⭐⭐

The Dynamic Language Runtime infrastructure supports dynamic binding and interoperability. A dynamic operation is resolved using runtime types and applicable binder rules; invalid operations throw a runtime binding exception instead of a compile error. The runtime can cache call-site binding rules, but dynamic remains less analyzable and generally costs more than direct static code.

If any relevant expression is `dynamic`, overload resolution may be postponed. The result of most dynamic operations is itself dynamic. Conversions back to a static type can also fail at runtime.

Use dynamic at genuine dynamic boundaries such as COM automation or dynamic-language integration. Contain it behind a statically typed adapter. Do not use it merely to avoid designing contracts, generics, or reflection validation.

### Interview check

**Basic — Q:** Is `var` runtime typing?  
**A:** No. The compiler infers a static type and checks uses normally.

**Intermediate — Q:** `object` versus `dynamic`?  
**A:** Both can hold any object; `object` operations are statically checked against `object`, while `dynamic` member binding is deferred.

**Senior — Q:** When is `dynamic` appropriate?  
**A:** At a truly dynamic/interop boundary, ideally isolated behind a typed API—not as a general substitute for design.

**Deep Dive — Q:** Does `dynamic` define a distinct CLR storage type?  
**A:** No. It is represented using `object` with compiler metadata that directs dynamic binding.

### Official Microsoft references

- [Using type `dynamic`](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic)
- [Dynamic language runtime overview](https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/dynamic-language-runtime-overview)
- [Implicitly typed local variables](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/implicitly-typed-local-variables)

---

# Part XIX — Expression Trees

## 82. Code as data ⭐⭐

`Expression<TDelegate>` represents a lambda as an immutable tree of expression nodes. `Func<T,...>` represents executable delegate behavior.

```csharp
Func<User, bool> compiled = u => u.IsActive;
Expression<Func<User, bool>> model = u => u.IsActive;

Func<User, bool> fromTree = model.Compile();
```

The tree can be inspected, translated, or rewritten by creating a new tree; `Compile()` produces a delegate that executes its semantics. Compilation has cost, so reuse compiled delegates where the lifetime and expression identity justify caching.

## 83. Query providers ⭐

`IQueryable<T>` carries an expression plus an `IQueryProvider`. Operators append method-call nodes, and enumeration/terminal operations ask the provider to execute/translate. Database-style providers need the expression structure—member access, constants, comparisons—not an opaque compiled delegate, so they can translate it to another query language.

Expression trees support a documented subset/model of language constructs. New C# syntax may be represented as older equivalent nodes or prohibited. A lambda with statement body, assignment, many pattern constructs, or optional/named argument forms may not convert. Provider support is narrower still.

Parameter nodes are identified by node objects, not only names. When composing trees, consistently substitute parameters with an `ExpressionVisitor`; simply combining bodies with differently owned `ParameterExpression` nodes produces an invalid/unbound shape.

### Interview check

**Basic — Q:** Delegate versus expression tree?  
**A:** A delegate is executable behavior; an expression tree is an inspectable immutable representation of code.

**Intermediate — Q:** Why does an ORM-like provider want an expression tree?  
**A:** It needs structure it can translate, rather than opaque IL/native delegate behavior.

**Senior — Q:** Does `Compile()` make provider translation better?  
**A:** No. It removes the inspectable representation by producing executable code; providers need the tree.

**Deep Dive — Q:** How do you combine two predicates safely?  
**A:** Rebind their parameter nodes to one shared parameter via an expression visitor, then compose bodies.

### Official Microsoft references

- [Expression trees overview](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)
- [Execute expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-execution)
- [Build expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-building)
- [`IQueryable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.iqueryable-1?view=net-10.0)

---

# Part XX — Attributes and Source Generation

## 84. Attributes are metadata ⭐

An attribute class derives from `System.Attribute`. Applying it records constructor arguments and named values in metadata; it does not itself execute application behavior. A consumer—compiler, runtime, reflection library, analyzer, or framework—assigns meaning.

```csharp
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false, Inherited = true)]
sealed class AuditedAttribute(string category) : Attribute
{
    public string Category { get; } = category;
}

[Audited("billing")]
sealed class InvoiceService { }
```

Attribute positional arguments and named values are restricted to metadata-supported types/forms. `AttributeUsage` controls valid targets, repetition, and inheritance behavior; reflective inheritance semantics differ by member/API, so use the chosen retrieval API deliberately.

## 85. Roslyn and generators ⭐⭐

Roslyn exposes compiler syntax trees, symbols, semantic models, diagnostics, code fixes, and compilation APIs. A source generator participates in compilation and adds source; generated code is compiled with the rest of the program. It does not rewrite existing user syntax and should not depend on runtime state.

Incremental generators declare transformations over compiler-provided values. The driver can cache/recompute only affected steps, so pipelines should be deterministic, value-comparable, cancellation-aware, and free of ambient side effects. Generate stable hint names and deterministic text; report actionable diagnostics for invalid inputs.

```text
User source + referenced symbols
             ↓
        Generator pipeline
             ↓
        Generated C# source
             ↓
       Normal compilation → IL + metadata
```

Generators are strong alternatives to runtime reflection for known-at-build-time registries, serializers, and mappings. An analyzer reports/enforces design; a generator emits code. They can be packaged together but solve different jobs.

### Interview check

**Basic — Q:** Does applying an attribute run its constructor during compilation?  
**A:** The compiler encodes metadata; normal runtime attribute objects are created when a consumer materializes them.

**Intermediate — Q:** Analyzer versus generator?  
**A:** An analyzer inspects and reports diagnostics; a generator contributes new source to compilation.

**Senior — Q:** Why prefer an incremental generator?  
**A:** Its declared pipeline lets the driver avoid recomputing unaffected work, improving IDE/build scalability when designed with stable values.

**Deep Dive — Q:** Can a source generator rewrite a method body?  
**A:** Not directly. It adds source; analyzers/code fixes can guide or apply source changes through different mechanisms.

### Official Microsoft references

- [Attributes](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
- [Create custom attributes](https://learn.microsoft.com/en-us/dotnet/standard/attributes/writing-custom-attributes)
- [.NET Compiler Platform SDK](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/)
- [Source generator cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md)
- [Incremental generators design](https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.md)

---

# Part XXI — Files, Streams, and I/O

## 86. Stream abstractions and ownership ⭐

`Stream` is a byte-oriented abstraction with capability properties such as `CanRead`, `CanWrite`, and `CanSeek`. Not every stream has a length or position. Partial reads are legal: a read can return fewer bytes than requested without reaching end-of-stream. Loop until the protocol's required count is obtained or zero signals completion.

| Type | Role |
|---|---|
| `FileStream` | File handle plus sync/async byte I/O and seek where supported |
| `MemoryStream` | Seekable stream over managed memory; useful for moderate in-memory payloads |
| `BufferedStream` | Adds buffering around another stream; avoid redundant layering where already buffered |
| `StreamReader` / `StreamWriter` | Text plus encoding/character buffering over a stream |
| `BinaryReader` / `BinaryWriter` | Primitive values in a specified binary representation over a stream |

Dispose the outermost owner and use `leaveOpen` when ownership of an inner stream belongs elsewhere. `Flush` pushes buffered data through layers; `Dispose` normally flushes writers. Flush is not automatically a durable-storage guarantee unless the specific API/OS option promises it.

### Correct exact-length read

```csharp
static async Task ReadExactlyAsync(
    Stream stream, Memory<byte> buffer, CancellationToken token)
{
    int offset = 0;
    while (offset < buffer.Length)
    {
        int read = await stream.ReadAsync(buffer[offset..], token);
        if (read == 0) throw new EndOfStreamException();
        offset += read;
    }
}
```

Modern streams also expose `ReadExactly`/`ReadExactlyAsync` APIs; use them when the target framework supplies the needed overload.

## 87. Async I/O, buffering, and scale ⭐

Use asynchronous file/network APIs when waiting would otherwise block valuable threads. Pass cancellation and avoid a sync wrapper over an async core. Choose buffer sizes based on evidence and protocol; enormous buffers increase memory/LOH pressure and concurrent working set.

`File.ReadAllBytes/Text` is convenient for known-small content. For very large or untrusted input it materializes the entire payload and can cause high peak memory, LOH allocations, latency, or denial of service. Stream/process incrementally and enforce length limits.

Text readers decode bytes using an encoding; byte offsets do not generally equal character indices. Be explicit about encoding at external boundaries and understand BOM detection options.

## 88. Pipelines ⭐⭐

`System.IO.Pipelines` helps implement high-performance streaming parsers. A `PipeReader` provides one or more memory segments; consumers examine `ReadOnlySequence<byte>`, advance the consumed and examined positions, and loop. A `PipeWriter` obtains memory, advances written count, and flushes.

Correct advancement is crucial: retaining buffers too long increases memory, advancing too far loses data, and failing to inspect completion/cancellation can spin. Pipelines manage buffers and backpressure but do not define an application protocol. Prefer simpler stream APIs unless profiling and parser complexity justify pipelines.

### Interview check

**Basic — Q:** Can `Stream.Read` return less than requested?  
**A:** Yes. Callers must loop when a protocol requires an exact length; zero means end of stream.

**Intermediate — Q:** Who disposes a wrapped stream?  
**A:** The documented owner. Outer readers/writers normally dispose inner streams unless constructed with `leaveOpen`.

**Senior — Q:** Why not `ReadAllBytes` for an upload?  
**A:** It makes peak memory proportional to full input and can drive LOH/GC pressure; stream with enforced bounds.

**Deep Dive — Q:** What do `PipeReader.AdvanceTo(consumed, examined)` positions express?  
**A:** What can be released and how far parsing inspected; incorrect values cause retention, repeated reads, or lost data.

### Official Microsoft references

- [File and stream I/O](https://learn.microsoft.com/en-us/dotnet/standard/io/)
- [`Stream`](https://learn.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-10.0)
- [`FileStream`](https://learn.microsoft.com/en-us/dotnet/api/system.io.filestream?view=net-10.0)
- [How to read text from a file](https://learn.microsoft.com/en-us/dotnet/standard/io/how-to-read-text-from-a-file)
- [System.IO.Pipelines](https://learn.microsoft.com/en-us/dotnet/standard/io/pipelines)

---

# Part XXII — Serialization

## 89. `System.Text.Json` contracts ⭐

`JsonSerializer` maps JSON to .NET contracts and back. Configure `JsonSerializerOptions` once and reuse it; metadata caches make an options instance effectively immutable after first use. Web defaults and general defaults differ, so do not assume case sensitivity, naming, number, or enum policy.

```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = false
};

string json = JsonSerializer.Serialize(new Person("Ada"), options);
Person? value = JsonSerializer.Deserialize<Person>(json, options);
```

Model absence, JSON `null`, required members, constructor binding, and default CLR values deliberately. Never treat successful deserialization as domain validation.

## 90. Converters and low-level readers/writers ⭐⭐

A `JsonConverter<T>` handles a type or type family when built-in mapping is insufficient. Converter order and `CanConvert` scope matter; a broad converter can intercept unintended types. Read exactly one JSON value and leave the reader at the documented token position. Avoid recursively calling the serializer with options that select the same converter indefinitely.

`Utf8JsonReader` is a forward-only `ref struct` reader over UTF-8 input; it supports low-allocation parsing and incremental state but cannot be stored across async suspension. `Utf8JsonWriter` writes valid UTF-8 JSON to buffers/streams and must be flushed/disposed according to ownership. Use them for protocol-level control or measured hotspots, not routine object mapping.

## 91. Source-generated serialization ⭐⭐

Source generation creates serialization metadata/logic at compile time. It improves startup, reduces runtime reflection, and supports trimming/Native AOT scenarios. Define a `JsonSerializerContext` with `[JsonSerializable]` declarations and pass its type information/context.

```csharp
[JsonSerializable(typeof(Person))]
internal partial class AppJsonContext : JsonSerializerContext { }

string json = JsonSerializer.Serialize(person, AppJsonContext.Default.Person);
```

Metadata-based generation supports broad customization; fast-path generation can optimize supported serialization shapes but has documented limitations and can fall back depending on configuration. Ensure every root/polymorphic type needed at runtime is included.

Security/robustness: bound input depth and payload size at the hosting layer, treat property names/values as untrusted, and do not enable polymorphic materialization without an explicit allow-list contract.

### Interview check

**Basic — Q:** Should a new `JsonSerializerOptions` be created per call?  
**A:** Usually no. Configure and reuse options so cached metadata is reused.

**Intermediate — Q:** Converter versus naming policy?  
**A:** A naming policy transforms property/dictionary names; a converter controls reading/writing a type's JSON representation.

**Senior — Q:** Why source-generate JSON metadata?  
**A:** Lower reflection/startup overhead and better trimming/Native AOT compatibility for known contracts.

**Deep Dive — Q:** Why can't a `Utf8JsonReader` local simply cross `await`?  
**A:** It is a ref-like type over external memory and must obey lifetime restrictions; preserve reader state/data through an async-safe design instead.

### Official Microsoft references

- [System.Text.Json overview](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/overview)
- [How to write custom converters](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/converters-how-to)
- [Source generation in System.Text.Json](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation)
- [Reflection versus source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/reflection-vs-source-generation)
- [`Utf8JsonReader`](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.utf8jsonreader?view=net-10.0)
- [`Utf8JsonWriter`](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.utf8jsonwriter?view=net-10.0)

---

# Part XXIII — Date and Time

## 92. Choose the semantic type ⭐

| Type | Represents | Typical use |
|---|---|---|
| `DateTime` | Date/time with `Kind`: Utc, Local, or Unspecified | Local calendar values or APIs explicitly standardized on UTC |
| `DateTimeOffset` | Date/time plus offset from UTC; identifies an instant | Timestamps crossing machines/services; default general timestamp choice |
| `TimeSpan` | Duration/interval | Elapsed or configured duration (not a time zone) |
| `TimeZoneInfo` | Rules for a geographic/system time zone | Convert/schedule using DST and historical adjustment rules |
| `DateOnly` | Calendar date without time/zone | Birthday, billing date |
| `TimeOnly` | Clock time without date/zone | Daily opening time |

An offset is not a time-zone identity. Several zones share an offset at one instant and change differently later. Persist the zone identifier as well as an appropriate instant/local schedule when future civil-time rules matter.

## 93. `DateTime.Kind` and conversions ⭐

`Kind` directs conversion behavior:

- `Utc` is treated as UTC.
- `Local` is associated with the machine's local zone.
- `Unspecified` has no UTC/local designation and conversion APIs make context-specific assumptions.

`DateTime.SpecifyKind` changes the label without converting ticks. That is correct only when external knowledge says the existing fields already represent that kind.

```csharp
DateTimeOffset instant = DateTimeOffset.UtcNow;
DateTimeOffset inZone = TimeZoneInfo.ConvertTime(instant, targetZone);
```

For elapsed time, prefer `Stopwatch`, whose timestamp source is intended for duration measurement; wall clocks can be adjusted.

## 94. DST ambiguity and invalid local times ⭐⭐

When clocks move backward, a local time can be **ambiguous** (two possible offsets). When clocks move forward, a local time can be **invalid** (never occurs). `TimeZoneInfo.IsAmbiguousTime`/`IsInvalidTime` allow explicit policy. Never silently choose an occurrence for financial, scheduling, or audit logic without a documented rule.

For future recurring events, store the civil date/time and time-zone identifier, then resolve with current rules. For historical/audit events, store an instant (often `DateTimeOffset`/UTC) and any original zone/offset needed for presentation or evidence.

### Example

```csharp
DateTimeOffset vietnam = new(2026, 9, 9, 9, 0, 0, TimeSpan.FromHours(7));
Console.WriteLine(vietnam.UtcDateTime.ToString("HH:mm"));
```

### Output

```text
02:00
```

The offset makes the instant unambiguous. It does not say which named `TimeZoneInfo` produced `+07:00`.

### Interview check

**Basic — Q:** `DateTimeOffset` versus `TimeSpan`?  
**A:** The former is a timestamp plus UTC offset; the latter is a duration.

**Intermediate — Q:** Why often prefer `DateTimeOffset` for distributed timestamps?  
**A:** It unambiguously identifies an instant while retaining the observed offset.

**Senior — Q:** Is an offset a time zone?  
**A:** No. A zone includes rule identity and transitions; an offset is only one displacement at an instant.

**Deep Dive — Q:** What is wrong with adding 24 hours to schedule “same local time tomorrow”?  
**A:** A DST transition can make the civil day 23 or 25 hours; schedule in the intended zone and resolve ambiguity/invalidity explicitly.

### Official Microsoft references

- [Choose between `DateTime`, `DateTimeOffset`, `TimeSpan`, and `TimeZoneInfo`](https://learn.microsoft.com/en-us/dotnet/standard/datetime/choosing-between-datetime)
- [Perform arithmetic with dates and times](https://learn.microsoft.com/en-us/dotnet/standard/datetime/performing-arithmetic-operations)
- [Use time zones in arithmetic](https://learn.microsoft.com/en-us/dotnet/standard/datetime/use-time-zones-in-arithmetic)
- [Resolve ambiguous times](https://learn.microsoft.com/en-us/dotnet/standard/datetime/resolve-ambiguous-times)
- [`DateTime.Kind`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.kind?view=net-10.0)
- [`DateOnly`](https://learn.microsoft.com/en-us/dotnet/standard/datetime/how-to-use-dateonly-timeonly)

---

# Part XXIV — Numeric Types

## 95. Integer, binary floating point, and decimal ⭐

| Type family | Key model | Good fit |
|---|---|---|
| `int`, `long` | Fixed-width signed integers | Counts, IDs where arithmetic is meaningful, exact integral values |
| `float`, `double`, `Half` | IEEE 754 binary floating point | Scientific/graphics/statistical values; `double` is normal default |
| `decimal` | 128-bit decimal representation with 28–29 digits precision | Financial/base-10 quantities under an explicit rounding policy |
| `BigInteger` | Arbitrarily large signed integer constrained by memory | Cryptography/math/identifiers requiring integer range beyond 64 bits |

`Half` is a 16-bit IEEE 754 binary floating-point type. It reduces storage/bandwidth but has limited range and precision; compute/convert deliberately.

### Floating-point representation trap

```csharp
Console.WriteLine(0.1 + 0.2 == 0.3);
Console.WriteLine(0.1 + 0.2);
```

### Output

```text
False
0.30000000000000004
```

Most decimal fractions have no finite binary representation. Equality policy should follow the domain: absolute/relative tolerance, ULP-oriented comparison, rounding to a defined unit, or exact decimal arithmetic. One universal epsilon is not correct for all magnitudes.

`decimal` represents many base-10 fractions exactly, but division and scale still require rounding. “Use decimal for money” is incomplete: also define currency minor units, midpoint rounding, allocation/reconciliation, overflow, and database/serialization precision.

## 96. Overflow and conversions ⭐

Integral arithmetic can be checked or unchecked. Constant overflow is normally diagnosed at compile time; runtime behavior depends on checked context and operation. Floating-point overflow produces infinities according to IEEE behavior rather than `OverflowException`. Conversions among floating types, decimal, and integers follow distinct rules—do not generalize one family to all.

Use explicit checked boundaries when silent wrap would corrupt data:

```csharp
int total = checked(unitPrice * quantity);
```

## 97. Generic math ⭐⭐

Interfaces such as `INumber<TSelf>`, `IBinaryInteger<TSelf>`, and `IFloatingPoint<TSelf>` expose static abstract operations, identities, parsing, and conversions to constrained generic algorithms. Choose the narrowest interface: an algorithm requiring only addition should not claim every number operation.

Generic conversion APIs distinguish checked, saturating, and truncating policies. This makes conversion intent part of reusable numeric code rather than hidden casts.

### Interview check

**Basic — Q:** Why can `0.1 + 0.2 != 0.3` for `double`?  
**A:** Those decimal fractions are approximated in finite binary floating-point, and the rounded results differ.

**Intermediate — Q:** Decimal or double for money?  
**A:** Usually decimal for base-10 quantities, plus an explicit currency/rounding policy; decimal alone does not solve domain rules.

**Senior — Q:** Does `checked` make floating-point overflow throw?  
**A:** No. Floating-point arithmetic follows IEEE behavior; checked context principally affects integral/decimal operations and conversions as documented.

**Deep Dive — Q:** Why use generic math rather than `dynamic`?  
**A:** Static abstract constraints preserve compile-time checking and enable runtime/JIT specialization without runtime binder failure.

### Official Microsoft references

- [Integral numeric types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types)
- [Floating-point numeric types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types)
- [`BigInteger`](https://learn.microsoft.com/en-us/dotnet/api/system.numerics.biginteger?view=net-10.0)
- [`Half`](https://learn.microsoft.com/en-us/dotnet/api/system.half?view=net-10.0)
- [Generic math](https://learn.microsoft.com/en-us/dotnet/standard/generics/math)
- [`checked` and `unchecked`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/checked-and-unchecked)

---

# Part XXV — Compilation and Runtime

## 98. From source to execution ⭐

```text
C# source
    ↓ Roslyn compiler
CIL (IL) + metadata + resources in an assembly
    ↓ CLR loader and type system
method selected for execution
    ↓ JIT, ReadyToRun code, or Native AOT strategy
native machine code
    ↓
processor execution under runtime services (GC, exceptions, threading, diagnostics)
```

Roslyn validates language rules and emits Common Intermediate Language plus metadata. An assembly is a deployment/versioning unit, commonly a `.dll` or `.exe`; extension does not mean “native DLL” or “raw machine-code executable.” The CLR loads metadata/types, verifies/coordinates managed execution, and supplies services.

## 99. JIT and tiered compilation ⭐⭐

JIT compilation translates a method's IL to native code for the current process architecture when execution needs it. That enables runtime-specific optimizations but contributes startup/warm-up cost. Generic instantiations, dynamic methods, and tiering affect when/which bodies compile.

Tiered compilation can first produce code quickly, then replace frequently executed methods with more optimized code. Tiered PGO can use runtime profile data. The exact thresholds and generated instructions are runtime policy—benchmark the exact supported runtime, environment, and workload; do not promise that a source construct is always inlined/vectorized/devirtualized.

## 100. ReadyToRun ⭐⭐

ReadyToRun (R2R) adds ahead-of-time-compiled native code to assemblies to improve startup by reducing JIT work. Trade-offs include larger binaries and code that may be less optimized than code generated with runtime knowledge. JIT may still be used for methods without suitable R2R code and tiering may replace R2R code for hot paths.

R2R is not the same as Native AOT: it retains the normal managed runtime deployment/execution model and IL/metadata needed by it.

## 101. Native AOT ⭐⭐

Native AOT publishes an application as native code ahead of time, with no JIT at runtime. Benefits can include startup, memory, and self-contained deployment characteristics. It imposes closed-world/static-analysis constraints:

- no runtime generation of new executable IL through `Reflection.Emit`;
- dynamic assembly loading and some reflection patterns are limited/incompatible;
- trimming warnings indicate code that may not work safely;
- platform/architecture-specific binaries are produced;
- size/performance results depend on workload and publish settings.

Libraries should be tested/annotated for trimming and AOT. Source generation and analyzable generic/direct access often replace runtime discovery. Native AOT is a deployment choice, not an automatic universal speedup.

## 102. Trimming and single-file deployment ⭐⭐

Trimming removes code static analysis believes unreachable. Reflection, serializers, dependency injection scanning, COM/interop, and dynamically loaded features can hide required members. Fix warnings by making access analyzable, generating code, or accurately annotating requirements; broad roots/suppressions trade size for safety and can conceal bugs.

Single-file publishing bundles application-dependent files into one deployment artifact, subject to documented exclusions/extraction behavior. Runtime APIs that assume `Assembly.Location` or adjacent physical files may behave differently. Locate content through explicit configuration/resources rather than assembly layout assumptions.

### Comparison

| Strategy | Startup | Runtime flexibility | Binary size | Runtime JIT |
|---|---|---|---|---|
| Normal IL + JIT | Warm-up required | Highest | Baseline | Yes |
| ReadyToRun | Often improved | High | Larger | Still possible/expected |
| Native AOT | Strong startup potential | Restricted closed world | Workload/config-dependent | No |

### Version notes

Native AOT publishing became supported for console applications in .NET 7 and broadened in later releases. Current .NET 10 tooling adds runtime/compiler improvements, but compatibility must be verified from build warnings and tests. Tiered compilation and ReadyToRun exist across .NET 5–10 with evolving defaults/quality.

### Interview check

**Basic — Q:** What does the C# compiler normally emit?  
**A:** An assembly containing IL, metadata, and related resources—not generally final native instructions for every target.

**Intermediate — Q:** ReadyToRun versus JIT?  
**A:** R2R supplies precompiled code to reduce startup work, but JIT can still compile/replace methods and optimize hot paths.

**Senior — Q:** Why does trimming generate warnings?  
**A:** Static analysis cannot prove dynamically accessed code will remain; the warning points to a possible publish-time removal/runtime failure.

**Deep Dive — Q:** Is Native AOT just “JIT earlier”?  
**A:** No. It uses a closed-world native publishing model with no runtime JIT and meaningful dynamic-code/loading/reflection restrictions.

### Official Microsoft references

- [Managed execution process](https://learn.microsoft.com/en-us/dotnet/standard/managed-execution-process)
- [Metadata and self-describing components](https://learn.microsoft.com/en-us/dotnet/standard/metadata-and-self-describing-components)
- [Tiered compilation runtime configuration](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/compilation)
- [ReadyToRun compilation](https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run)
- [Native AOT deployment](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/)
- [Trim self-contained deployments](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trim-self-contained)
- [Single-file deployment](https://learn.microsoft.com/en-us/dotnet/core/deploying/single-file/overview)

---

# Part XXVI — CLR

## 103. Runtime responsibilities ⭐

The Common Language Runtime executes managed code and provides type safety, memory management, exception handling, threading, interop, assembly loading, and diagnostics. The CTS defines shared type behavior; the CLS defines a cross-language public subset. “Managed” means runtime services govern execution—it does not mean the code has no native dependencies or external resources.

At startup, the host locates/configures the runtime, the loader resolves the entry assembly and dependencies, types are loaded/initialized as required, and the entry point executes. Exact loading/JIT timing is demand- and implementation-dependent.

## 104. Type system and method calls ⭐⭐

Runtime type data describes inheritance, implemented interfaces, methods, fields, GC layout, and dispatch information. CoreCLR's `MethodTable` is a key implementation structure, but its field layout is not a public contract.

For a virtual call, the runtime uses the object's actual type and dispatch metadata to select the most-derived override. Interface calls need interface-to-implementation resolution. The JIT may devirtualize when it can prove the target without changing observable behavior.

Type initialization (`.cctor`) occurs according to CLI/C# initialization rules. `beforefieldinit` gives the runtime more freedom than an explicit static constructor. Do not build correctness on an exact eager/lazy moment beyond the documented guarantees.

## 105. Assembly loading and `AssemblyLoadContext` ⭐⭐

`AssemblyLoadContext` (ALC) provides loading isolation and version-resolution scopes. The default ALC loads ordinary application dependencies. Custom ALCs support plugin scenarios and can be collectible.

Each ALC can load at most one version per simple assembly name; resolution succeeds when a loaded version is compatible with the requested version under documented rules. The same named/type-shaped assembly loaded into different contexts can produce types that are not assignment-compatible—their assembly identities/contexts differ.

Unloading is **cooperative**: call `Unload` on a collectible ALC, remove all strong references to its assemblies/types/instances/delegates/threads, then GC can collect it. Static caches, event handlers, running threads, and reflection metadata often prevent unload.

## 106. Managed/unmanaged interop ⭐⭐

Platform invocation (P/Invoke) lets managed code call exports in native libraries. The interop layer marshals supported data between managed and native representations. Correctness requires matching calling convention, character encoding, sizes/layout, ownership, and lifetime.

```csharp
internal static partial class NativeMethods
{
    [LibraryImport("mylibrary", StringMarshalling = StringMarshalling.Utf8)]
    internal static partial int transform(ReadOnlySpan<byte> input);
}
```

Use the source-generated `[LibraryImport]` path on supported modern .NET when suitable; `[DllImport]` remains supported. The actual signature above is illustrative—native declarations must match the real ABI. Prefer `SafeHandle` for owned handles and avoid exposing raw `IntPtr` ownership.

COM interop projects COM types/members into managed code and handles identity/reference/marshalling. Release behavior and apartment threading require platform-specific design; do not sprinkle manual release calls without following the official ownership model.

### Interview check

**Basic — Q:** CTS versus CLS?  
**A:** CTS defines .NET's complete common type model; CLS is a consumer-friendly subset for cross-language public APIs.

**Intermediate — Q:** What makes code managed?  
**A:** It executes under CLR services such as GC, type/exception/threading infrastructure; it may still call native code.

**Senior — Q:** Why can identical-looking plugin types fail casts?  
**A:** Type identity includes assembly identity/loading context; separate ALC-loaded definitions are different runtime types.

**Deep Dive — Q:** Why is collectible ALC unload cooperative?  
**A:** The context cannot disappear while live references, code, threads, delegates, or metadata from it remain reachable/active.

### Official Microsoft references

- [Common Language Runtime](https://learn.microsoft.com/en-us/dotnet/standard/clr)
- [Common Type System](https://learn.microsoft.com/en-us/dotnet/standard/base-types/common-type-system)
- [Language independence and CLS](https://learn.microsoft.com/en-us/dotnet/standard/language-independence)
- [About `AssemblyLoadContext`](https://learn.microsoft.com/en-us/dotnet/core/dependency-loading/understanding-assemblyloadcontext)
- [Unloadability](https://learn.microsoft.com/en-us/dotnet/standard/assembly/unloadability)
- [Native interoperability](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/)
- [Source-generated P/Invoke](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke-source-generation)
- [CoreCLR introduction](https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/intro-to-clr.md)
- [Virtual stub dispatch](https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/virtual-stub-dispatch.md)

---

# Part XXVII — Performance

## 107. Optimization is an evidence loop ⭐

```text
Define user/SLO problem → measure representative workload → find dominant cause
          ↑                                              ↓
validate correctness/regressions ← change one meaningful constraint
```

State the metric: latency percentile, throughput, CPU, allocation rate, pause time, working set, startup, or binary size. Use release builds without debugger distortion and production-like data/concurrency. A faster micro-operation can make the system worse through complexity, batching latency, contention, or retained memory.

## 108. High-probability allocation costs ⭐

| Source | Better option when measured | When the “optimization” hurts |
|---|---|---|
| Boxing value types | Generic API / constrained call | Extra generic complexity for cold code |
| Repeated string intermediates | Span parsing, one interpolation, `StringBuilder` for loops | Obscures simple concatenation compiler already optimizes |
| Capturing delegates | Static lambda + explicit state, static local function | API becomes awkward; delegate cost insignificant |
| LINQ iterator/delegate/materialization | Fused loop, avoid unnecessary `ToList` | Loses clarity/composability in non-hot path |
| Repeated medium/large arrays | `ArrayPool<T>` with strict ownership | Retains large buffers, clearing/correctness cost |
| Async completion objects | Fast synchronous path / carefully justified `ValueTask` | Consumption hazards and larger state machine |
| Reflection per item | Cached metadata/delegate or generated code | Cache prevents unload or adds startup/memory |
| Serializer metadata | Reused options/source generation | Generated contract maintenance/build complexity |

Allocation is not inherently bad—Gen 0 is designed for short-lived objects. Optimize allocation rate/retention only when it affects the target metric.

## 109. Structs, copies, and dispatch ⭐⭐

Small immutable structs can improve locality and avoid per-element object allocations. Large mutable structs create hidden copies, defensive copies through readonly receivers, bigger arrays, and expensive passing. Measure and consider `in`/`ref readonly` only when copy cost is material; aliasing can cost more than a simple copy and may inhibit optimizations.

Virtual/interface calls provide abstraction. The JIT can devirtualize/in-line proven targets, but source code must not depend on it. Replacing virtual design with manual type switches is rarely justified without a hot-path profile and maintainable boundary.

## 110. Span, pooling, and unsafe code ⭐⭐

Span-based parsing can avoid substrings/temporary arrays and work over caller-owned buffers. It does not eliminate the source buffer and cannot cross arbitrary lifetime/async boundaries. Pooling trades allocation for ownership complexity and retained capacity. Unsafe code can remove checks or bridge native APIs, but can also introduce memory corruption and prevent runtime reasoning.

Prefer algorithmic and I/O improvements before micro-optimizations:

1. Remove unnecessary remote calls and repeated database/query enumeration.
2. Choose appropriate algorithms/data structures.
3. Bound concurrency and avoid contention.
4. Reduce large retention/allocation.
5. Tune local instructions only with a remaining hotspot.

## 111. Async and concurrency performance ⭐⭐

Async improves scalability/responsiveness for waiting; it does not make CPU work faster. Common problems include synchronously blocking tasks, creating unbounded concurrent operations, thread-pool starvation, holding locks during callbacks, and allocating tasks/state for operations that need not be async.

`ValueTask` is useful when synchronous completion is frequent and measurement shows task allocation matters. It is not the default “faster Task.” `Task.Run` can preserve UI responsiveness for CPU work, but on a server it consumes pool threads and can worsen overload.

## 112. Measurement ladder ⭐

- `dotnet-counters`: live first-look rates and runtime metrics.
- `dotnet-trace`: sampled/event timeline for CPU, GC, tasks, runtime/providers.
- `dotnet-gcdump`: managed heap type/retention snapshot with lower footprint than a full dump in many scenarios.
- `dotnet-dump`: full process dump analysis for hangs, crashes, roots, threads.
- Application `Activity`, metrics, and logs: request-level correlation and production trend.
- `Stopwatch`: focused elapsed-time experiments, not a substitute for statistically rigorous end-to-end measurement.

### Interview check

**Basic — Q:** What is the first optimization step?  
**A:** Define the performance problem and measure a representative workload.

**Intermediate — Q:** Is zero allocation always the goal?  
**A:** No. Short-lived allocation is efficient; optimize when allocation rate, retention, or GC affects required metrics.

**Senior — Q:** When can a struct harm performance?  
**A:** When large/mutable values are repeatedly copied, boxed, stored densely at excessive size, or trigger defensive copies.

**Deep Dive — Q:** Can you promise an interface call costs a virtual dispatch?  
**A:** Semantically it dispatches through the interface, but the JIT may devirtualize/in-line when proven; measure generated behavior rather than promise an implementation.

### Official Microsoft references

- [Performance-oriented C# features](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/)
- [Diagnose high CPU](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/debug-highcpu)
- [Diagnose high memory](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/debug-memory-leak)
- [.NET diagnostics tools overview](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/tools-overview)
- [Garbage collection performance](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/performance)
- [Runtime configuration options for threading](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/threading)

---

# Part XXVIII — Diagnostics

## 113. Tool selection ⭐

| Tool/API | Best first use | Produces / key caution |
|---|---|---|
| `dotnet-counters` | Live health triage: CPU, GC, exceptions, thread pool | Streaming metrics; low detail, not causal proof |
| `dotnet-trace` | CPU/runtime/event timeline | `.nettrace`; collection has overhead and provider choice matters |
| `dotnet-dump` | Crash/hang/deadlock/deep root analysis | Full dump; large and sensitive, platform analysis limits apply |
| `dotnet-gcdump` | Managed heap composition/root investigation | GC dump; collection can induce runtime pause/GC behavior |
| EventPipe | Cross-platform event transport underneath tools | In-process/out-of-process sessions and provider configuration |
| `EventSource` | Custom high-performance event stream | Provider/event schema needs stable naming and levels/keywords |
| `Activity` | Distributed operation context/traces | Propagate context and stop/dispose activities correctly |
| `System.Diagnostics.Metrics` | Counters/histograms/observable gauges | Define stable units/names; avoid high-cardinality tags |
| OpenTelemetry | Vendor-neutral collection/export | SDK/exporter/configuration determine sampling and destinations |

Start with low-cost signals, narrow the hypothesis, then collect detail. Diagnostic artifacts can contain secrets, request bodies, paths, and customer data; protect access and retention.

## 114. Practical triage playbooks ⭐⭐

**High CPU**

1. Confirm process CPU and duration with counters/OS telemetry.
2. Check GC CPU, exception rate, thread-pool queue/thread count.
3. Capture a bounded trace with CPU sampling and relevant runtime providers.
4. Identify hot stacks; relate them to request/activity dimensions.
5. Reproduce and validate the change under the same workload.

**Memory growth**

1. Separate GC heap size, working set, native memory, and allocation rate.
2. Observe Gen 2/LOH behavior and whether memory plateaus after full collections.
3. Capture comparable GC dumps at meaningful points; compare types and roots.
4. Use a full dump when thread/native/precise root context is necessary.
5. Fix ownership/reachability; do not declare a leak from working set alone.

**Hang/starvation**

1. Check request latency, thread-pool queue length, thread count, locks/contention.
2. Capture a trace for scheduling/contention or dump for thread stacks.
3. Look for sync-over-async, circular lock waits, blocked I/O, unbounded queueing.

## 115. Instrumentation model ⭐⭐

`ActivitySource` creates activities only when a listener records them; check/create with the standard pattern and attach low-cardinality semantic tags. `Meter` creates instruments. Counters add deltas, histograms record distributions, and observable gauges report sampled current values.

Use logs for discrete records, metrics for aggregated trend/alerts, and traces for causal request paths. Correlation IDs alone do not replace trace context. Avoid user IDs, raw URLs, and exception messages as metric tags because cardinality can become unbounded.

OpenTelemetry integrates logs, metrics, and distributed tracing using .NET diagnostic APIs and an SDK/export pipeline. The API/SDK separation lets libraries instrument without selecting a vendor exporter.

### Interview check

**Basic — Q:** First tool for live runtime health?  
**A:** Often `dotnet-counters`; it quickly shows rates/levels before a more expensive trace or dump.

**Intermediate — Q:** GC dump versus process dump?  
**A:** A GC dump focuses on managed heap graph/type data; a full dump includes threads and broader process memory/state but is larger/sensitive.

**Senior — Q:** How do you investigate memory growth?  
**A:** Distinguish managed/native/working set and allocation/retention, take comparable evidence, then inspect dominant types and roots.

**Deep Dive — Q:** Why are high-cardinality metric tags dangerous?  
**A:** Each unique tag combination becomes a time series, causing backend/client memory and cost explosion.

### Official Microsoft references

- [.NET diagnostics tools overview](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/tools-overview)
- [`dotnet-counters`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-counters)
- [`dotnet-trace`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)
- [`dotnet-dump`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump)
- [`dotnet-gcdump`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-gcdump)
- [EventPipe overview](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/eventpipe)
- [`EventSource`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventsource?view=net-10.0)
- [.NET observability with OpenTelemetry](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)
- [.NET metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/metrics)
- [Distributed tracing concepts](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/distributed-tracing-concepts)

---
