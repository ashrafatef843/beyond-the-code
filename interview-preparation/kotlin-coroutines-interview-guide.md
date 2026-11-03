# Kotlin Coroutines: Execution Behaviour, Dispatchers, Cancellation, Exception Handling & Interview Questions

## 1. Introduction

Kotlin Coroutines provide a way to write asynchronous and concurrent code in a sequential, readable style.

Unlike traditional thread-based programming, coroutines can suspend their execution without blocking the underlying thread. While one coroutine is suspended, another coroutine can execute using the available thread.

Understanding coroutines requires understanding five important concepts:

1. Coroutine execution and scheduling.
2. Coroutine contexts and dispatchers.
3. Coroutine lifecycle and structured concurrency.
4. Cancellation behaviour.
5. Exception propagation and handling.

This article explores these concepts with practical examples, expected outputs, and interview questions.

---

# PART 1: COROUTINE FUNDAMENTALS

## 2. What is a Coroutine?

A coroutine is a suspendable computation that can pause its execution and resume later.

A coroutine is not a thread.

A thread is an operating-system execution resource, whereas a coroutine is a unit of work that is scheduled to execute using threads.

### Thread vs Coroutine

| Thread | Coroutine |
| --- | --- |
| Managed by the operating system | Managed by the coroutine runtime |
| More expensive to create | Lightweight |
| Blocking operations occupy the thread | Suspending operations can release the thread |
| Context switching can be expensive | Coroutine suspension and resumption are relatively lightweight |
| Thread has its own execution stack | Coroutine state is maintained across suspension |

Example:

```kotlin
runBlocking {
    launch {
        delay(1000)
        println("Coroutine completed")
    }

    println("Main coroutine")
}
```

Output:

```text
Main coroutine
Coroutine completed
```

`delay()` suspends the coroutine without blocking its thread.

## 3. What is runBlocking?

`runBlocking` is a coroutine builder that bridges regular blocking code and suspending code.

It:

- Creates a coroutine.
- Creates a coroutine scope.
- Blocks the calling thread until its coroutine and children complete.
- Uses an event-loop dispatcher by default when no dispatcher is specified.

Example:

```kotlin
fun main() {
    println("Before")

    runBlocking {
        delay(1000)
        println("Inside runBlocking")
    }

    println("After")
}
```

Output:

```text
Before
Inside runBlocking
After
```

Important distinction:

`runBlocking` creates a coroutine, but it does not necessarily create a new thread.

It normally executes on the calling thread.

**When should you use it?**

Use `runBlocking` primarily at boundaries where regular blocking code needs to call suspending code, such as a command-line application's `main` function or certain tests.

Avoid using it inside Android UI code because it blocks the calling thread.

## 4. Coroutine Builders: launch, async and withContext

These are three important coroutine APIs.

### 4.1 launch

`launch` starts a coroutine and returns a `Job`.

Use it when you want to perform an operation without returning a result.

```kotlin
runBlocking {
    val job = launch {
        delay(1000)
        println("Task completed")
    }

    println("Main continues")

    job.join()
}
```

Output:

```text
Main continues
Task completed
```

Important properties:

- Creates a coroutine.
- Returns `Job`.
- Starts immediately by default (scheduled for execution).
- Does not return a computation result.
- Supports cancellation.
- Can be awaited using `join()`.

### 4.2 async

`async` starts a coroutine and returns a `Deferred<T>`.

Use it when you need a result from concurrent work.

```kotlin
runBlocking {
    val deferred = async {
        delay(1000)
        "Hello"
    }

    println("B")

    val result = deferred.await()

    println(result)
}
```

Output:

```text
B
Hello
```

Important properties:

- Creates a coroutine.
- Returns `Deferred<T>`.
- Starts immediately by default.
- `await()` suspends the caller until the result is available.
- `await()` returns the result.
- If the computation fails, `await()` throws its exception.

**Important:** `async` starts eagerly by default, but actual execution depends on the dispatcher and scheduling.

### 4.3 withContext

`withContext` executes a block with a modified coroutine context and returns its result.

Unlike `launch` and `async`, it does not create an independently managed child task.

Example:

```kotlin
runBlocking {
    println("A")

    val result = withContext(Dispatchers.Default) {
        println("B")
        "Result"
    }

    println("C")
    println(result)
}
```

Output:

```text
A
B
C
Result
```

Execution:

1. A executes using the original dispatcher.
2. Execution switches to `Dispatchers.Default`.
3. B executes.
4. The block completes.
5. Execution resumes in the original context.
6. C executes.

### Builder comparison

| Feature | launch | async | withContext |
| --- | --- | --- | --- |
| Creates coroutine | Yes | Yes | Executes in current coroutine context |
| Returns | Job | Deferred | Result |
| Returns computation result | No | Yes | Yes |
| Starts by default | Immediately scheduled | Immediately scheduled | Executes when called |
| Waiting mechanism | join() | await() | Automatically waits for block |
| Typical use | Background operation | Concurrent result | Context switching |

---

# PART 2: EXECUTION AND SCHEDULING

## 5. Understanding Coroutine Execution Order

One of the most important interview topics is understanding when a coroutine actually executes.

Consider:

```kotlin
runBlocking {
    launch {
        print("A")
    }

    print("B")
}
```

Expected output with the default `runBlocking` event loop:

```text
BA
```

Why?

1. `runBlocking` starts executing.
2. `launch` creates and schedules a child coroutine.
3. The parent continues executing.
4. The parent prints B.
5. The parent reaches completion and waits for its child.
6. The child executes and prints A.

Important: Calling `launch` does not mean that the child must execute immediately before the next statement in the parent.

### Example: Adding delay

```kotlin
runBlocking {
    launch {
        delay(10)
        print("A")
    }

    print("B")
}
```

Output:

```text
BA
```

The child suspends at `delay(10)`, allowing the parent to continue.

### Example: Parent and child

```kotlin
runBlocking {
    launch {
        print("B")
    }

    print("A")
}
```

Output:

```text
AB
```

The parent prints A before the child gets its execution opportunity.

**Important interview concept:** Never assume that all coroutine execution orders are guaranteed. The dispatcher, suspension points, and scheduling determine the actual order.

## 6. What is a Dispatcher?

A dispatcher determines which thread or threads are used to execute coroutine code.

A coroutine dispatcher is part of the coroutine context.

Important dispatchers:

### Dispatchers.Main

Used for UI-related operations.

```kotlin
withContext(Dispatchers.Main) {
    // Update UI
}
```

### Dispatchers.IO

Designed for blocking I/O operations.

```kotlin
withContext(Dispatchers.IO) {
    // File or blocking network operation
}
```

### Dispatchers.Default

Designed for CPU-intensive operations.

```kotlin
withContext(Dispatchers.Default) {
    // Sorting, calculations, processing
}
```

### Dispatchers.Unconfined

Starts execution in the current call-frame thread until the first suspension point.

After suspension, execution resumes on the thread selected by the suspending operation.

It is generally not recommended for ordinary application code.

### Dispatcher comparison

| Dispatcher | Typical purpose |
| --- | --- |
| Main | UI operations |
| IO | Blocking I/O |
| Default | CPU-intensive work |
| Unconfined | Special-purpose execution |

## 7. Does withContext Permanently Change the Dispatcher?

No.

Consider:

```kotlin
runBlocking {
    print("A")

    withContext(Dispatchers.Default) {
        print("B")
    }

    print("C")
}
```

Dispatcher execution:

| Statement | Dispatcher |
| --- | --- |
| A | runBlocking event loop |
| B | Dispatchers.Default |
| C | Original runBlocking dispatcher |

`withContext` temporarily changes the coroutine context for the block.

After the block completes, the calling coroutine resumes in its original context.

Also remember that a dispatcher does not guarantee a specific thread identity. For example, Dispatchers.Default uses a shared pool of threads.

---

# PART 3: SUSPENSION AND COOPERATIVE SCHEDULING

## 8. What is a Suspension Point?

A suspension point is a location where a coroutine can suspend its execution and resume later.

Examples include:

```kotlin
delay(1000)
deferred.await()
job.join()
yield()
```

A suspending function does not necessarily suspend every time it is called.

For example, `await()` does not need to suspend if the Deferred is already completed.

This is an important distinction:

**Suspending does not mean blocking.**

Suspending releases the executing thread so that other work can use it.

## 9. What is delay()?

`delay()` suspends the current coroutine for a specified duration.

```kotlin
runBlocking {
    println("A")

    delay(1000)

    println("B")
}
```

Output:

```text
A
B
```

During the delay:

- The coroutine is suspended.
- The thread is not blocked by `delay`.
- Other coroutines can execute.

Unlike `Thread.sleep()`, `delay()` is coroutine-friendly and cancellable.

## 10. What is yield()?

`yield()` suspends the current coroutine and gives the dispatcher an opportunity to execute other coroutines.

Example:

```kotlin
runBlocking {
    launch {
        repeat(3) {
            print("A")
            yield()
        }
    }

    launch {
        repeat(3) {
            print("B")
            yield()
        }
    }
}
```

A possible output:

```text
ABABAB
```

The exact output is not guaranteed.

### Without yield()

```kotlin
runBlocking {
    launch {
        repeat(3) {
            print("A")
        }
    }

    launch {
        repeat(3) {
            print("B")
        }
    }
}
```

With the default runBlocking event loop, a possible output is:

```text
AAABBB
```

### Important properties of yield()

- It is a suspending function.
- It does not block the thread.
- It gives other coroutines an opportunity to execute.
- It does not guarantee that another coroutine will execute.
- It checks cancellation.

### delay vs yield

| delay() | yield() |
| --- | --- |
| Suspends for a specified duration | Suspends to yield execution |
| Usually waits for a timer | Does not intentionally wait for a specified duration |
| Cancellation-aware | Cancellation-aware |
| Useful for delays and timeouts | Useful for cooperative scheduling |

---

# PART 4: WAITING FOR COROUTINES

## 11. What is join()?

`join()` is a suspending function available on `Job`.

It waits until the target coroutine completes.

```kotlin
runBlocking {
    val job = launch {
        delay(1000)
        println("Child completed")
    }

    println("Before join")

    job.join()

    println("After join")
}
```

Output:

```text
Before join
Child completed
After join
```

Important:

- `join()` does not cancel the coroutine.
- It suspends the calling coroutine.
- It does not block the thread.
- It does not return the coroutine's result.
- It does not rethrow the target Job's failure.

## 12. What is cancel()?

`cancel()` requests cancellation of a coroutine.

```kotlin
runBlocking {
    val job = launch {
        repeat(10) {
            println("Working $it")
            delay(500)
        }
    }

    delay(1000)

    job.cancel()
}
```

The coroutine stops when it observes cancellation.

Important: `cancel()` does not wait for cancellation to finish.

## 13. What is cancelAndJoin()?

`cancelAndJoin()` combines cancellation and waiting.

```kotlin
runBlocking {
    val job = launch {
        try {
            repeat(10) {
                println("Working $it")
                delay(500)
            }
        } finally {
            println("Cleaning up")
        }
    }

    delay(1000)

    job.cancelAndJoin()

    println("Coroutine completely finished")
}
```

Possible output:

```text
Working 0
Working 1
Cleaning up
Coroutine completely finished
```

The sequence is:

1. Request cancellation.
2. Wait for the coroutine to complete.
3. Continue executing.

### Comparison

| API | Cancels? | Waits? |
| --- | --- | --- |
| join() | No | Yes |
| cancel() | Yes | No |
| cancelAndJoin() | Yes | Yes |

## 14. What is await()?

`await()` is a suspending function available on `Deferred<T>`.

It waits for the result of an asynchronous computation.

```kotlin
runBlocking {
    val deferred = async {
        delay(1000)
        "A"
    }

    print("B")

    print(deferred.await())
}
```

Output:

```text
BA
```

Execution:

1. `async` schedules the child coroutine.
2. The child suspends at `delay`.
3. Parent prints B.
4. Parent calls `await()`.
5. Parent suspends until the result is available.
6. Child completes and returns A.
7. Parent resumes and prints A.

### join vs await

| join() | await() |
| --- | --- |
| Used with Job | Used with Deferred |
| Waits for completion | Waits for result |
| Returns Unit | Returns result |
| Does not rethrow target's failure | Rethrows target's failure |

**Important:** `Deferred` extends `Job`, so it also supports `join()`.

---

# PART 5: CANCELLATION BEHAVIOUR

## 15. What is Coroutine Cancellation?

Cancellation is the process of stopping a coroutine before it completes normally.

Cancellation is cooperative in Kotlin.

This means that a coroutine must cooperate with the cancellation mechanism.

It does not mean that the runtime forcibly terminates the coroutine at an arbitrary instruction.

### Example

```kotlin
runBlocking {
    val job = launch {
        repeat(10) {
            println("Working $it")
            delay(1000)
        }
    }

    delay(2500)

    job.cancel()
}
```

The coroutine observes cancellation at the cancellable `delay()` suspension point.

It stops instead of continuing through all ten iterations.

## 16. Why is Cancellation Cooperative?

Consider:

```kotlin
runBlocking {
    val job = launch {
        while (true) {
            println("Working")
        }
    }

    delay(1000)
    job.cancel()
}
```

This is problematic.

The child never suspends or checks cancellation.

With the default single-threaded runBlocking event loop, it can prevent the parent from progressing to the cancellation call.

A CPU-intensive coroutine needs explicit cancellation checks.

### Solution 1: isActive

```kotlin
runBlocking {
    val job = launch {
        while (isActive) {
            println("Working")
        }
    }

    delay(1000)
    job.cancelAndJoin()
}
```

### Solution 2: ensureActive()

```kotlin
runBlocking {
    val job = launch {
        while (true) {
            ensureActive()
            // Perform work
        }
    }

    delay(1000)
    job.cancelAndJoin()
}
```

### Solution 3: yield()

```kotlin
runBlocking {
    val job = launch {
        while (true) {
            yield()
            // Perform work
        }
    }

    delay(1000)
    job.cancelAndJoin()
}
```

`yield()` provides a scheduling opportunity and checks for cancellation.

## 17. What Happens When a Coroutine Is Cancelled?

When a coroutine reaches a cancellable suspension point after cancellation has been requested, it typically throws `CancellationException`.

Example:

```kotlin
runBlocking {
    val job = launch {
        try {
            delay(Long.MAX_VALUE)
        } catch (e: CancellationException) {
            println("Coroutine cancelled")
        }
    }

    delay(100)
    job.cancelAndJoin()
}
```

Output:

```text
Coroutine cancelled
```

Cancellation is represented internally using `CancellationException`.

This exception is treated as a cancellation signal, not an ordinary application failure.

## 18. What Happens to finally During Cancellation?

The `finally` block executes when a coroutine is cancelled, just as it does for other exceptional exits.

```kotlin
runBlocking {
    val job = launch {
        try {
            delay(Long.MAX_VALUE)
        } finally {
            println("Cleanup executed")
        }
    }

    delay(100)
    job.cancelAndJoin()
}
```

Output:

```text
Cleanup executed
```

This is extremely important for resource cleanup.

Use `finally` to release resources such as:

- File handles.
- Database resources.
- Temporary resources.
- Locks.
- Other acquired resources.

## 19. What is NonCancellable?

Normally, suspending functions inside a cancellation-triggered `finally` block may immediately throw another `CancellationException`.

If cleanup itself requires suspension, use `NonCancellable`.

```kotlin
runBlocking {
    val job = launch {
        try {
            delay(Long.MAX_VALUE)
        } finally {
            withContext(NonCancellable) {
                delay(1000)
                println("Cleanup completed")
            }
        }
    }

    delay(100)
    job.cancelAndJoin()

    println("Job completed")
}
```

Output:

```text
Cleanup completed
Job completed
```

`NonCancellable` allows the cleanup block to suspend without being cancelled by the Job's existing cancellation.

Use it only for short, necessary cleanup operations.

Do not use it to make ordinary application work ignore cancellation.

## 20. Should We Catch CancellationException?

Usually, no. Do not swallow cancellation exceptions.

Bad practice:

```kotlin
launch {
    try {
        delay(Long.MAX_VALUE)
    } catch (e: Exception) {
        println("Something went wrong")
    }
}
```

Why is this dangerous?

`CancellationException` extends `Exception`.

Therefore, the catch block can accidentally swallow cancellation.

Better:

```kotlin
launch {
    try {
        delay(Long.MAX_VALUE)
    } catch (e: CancellationException) {
        throw e
    } catch (e: Exception) {
        println("Handle actual failure")
    }
}
```

Or, if you do not need special cancellation handling, simply let cancellation propagate.

**Rule:** Never accidentally convert cancellation into successful completion.

---

# PART 6: EXCEPTION HANDLING AND PROPAGATION

## 21. How Does Exception Handling Work in Coroutines?

Exception handling in coroutines is different from ordinary synchronous code.

Consider:

```kotlin
try {
    scope.launch {
        throw IOException("failed")
    }
} catch (e: IOException) {
    println("Caught")
}
```

The outer catch does not catch the exception thrown inside the asynchronously executing coroutine.

Why?

The exception occurs during coroutine execution, not during the synchronous invocation of `launch`.

The exception is handled according to the coroutine's Job hierarchy and exception-handling rules.

## 22. Exception Behaviour of launch

`launch` returns a Job.

If a coroutine launched under a regular parent Job throws an uncaught exception other than `CancellationException`, it fails and propagates the failure to its parent.

Example:

```kotlin
fun main() = runBlocking {
    try {
        coroutineScope {
            launch {
                throw IOException("Failed")
            }
        }
    } catch (e: IOException) {
        println("Caught: ${e.message}")
    }
}
```

Output:

```text
Caught: Failed
```

Here, `coroutineScope` propagates the child failure to its caller, where the exception is caught.

Important: The parent and sibling coroutines may also be cancelled because of this failure.

## 23. Exception Behaviour of async

`async` returns `Deferred<T>`.

Its exception is stored in the Deferred and is rethrown when `await()` retrieves the result.

Example:

```kotlin
runBlocking {
    val deferred = async {
        throw IOException("Failed")
    }

    try {
        deferred.await()
    } catch (e: IOException) {
        println("Caught: ${e.message}")
    }
}
```

Be careful: In a regular structured coroutine scope, an uncaught failure in a child `async` also propagates to its parent Job.

Therefore, the parent may be cancelled before it reaches `await()`.

For independently handling multiple child failures, use supervision where appropriate.

## 24. Difference Between launch and async Exception Handling

| launch | async |
| --- | --- |
| Returns Job | Returns Deferred |
| Uncaught failure propagates through regular parent Job | Uncaught failure also propagates through regular parent Job |
| Does not expose a result | Exposes result through await |
| Failure can reach CoroutineExceptionHandler for an appropriate root coroutine | Failure is stored in Deferred and observed through await |

Important: `CoroutineExceptionHandler` is not a replacement for `try-catch`.

It is a last-resort mechanism for reporting uncaught exceptions.

It cannot recover a failed coroutine.

## 25. What is CoroutineExceptionHandler?

`CoroutineExceptionHandler` handles uncaught exceptions from eligible coroutines, typically root coroutines.

Example:

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    println("Exception: ${exception.message}")
}

val scope = CoroutineScope(
    SupervisorJob() + Dispatchers.Default + handler
)

scope.launch {
    throw IOException("Network failure")
}
```

The handler reports the uncaught exception.

Important points:

- It is not invoked for every exception.
- It does not replace local exception handling.
- It cannot recover the failed coroutine.
- Ordinary child coroutines generally delegate failure handling to their parent.
- Exceptions from `async` are normally observed through `await()`.

## 26. What is the Difference Between coroutineScope and supervisorScope?

This is one of the most important coroutine interview topics.

### coroutineScope

In a regular coroutine scope, a child failure cancels the parent and its other children.

```kotlin
runBlocking {
    try {
        coroutineScope {
            launch {
                delay(100)
                throw IOException("Failed")
            }

            launch {
                try {
                    delay(Long.MAX_VALUE)
                } finally {
                    println("Second child cancelled")
                }
            }
        }
    } catch (e: IOException) {
        println("Caught exception")
    }
}
```

Output:

```text
Second child cancelled
Caught exception
```

### supervisorScope

In a supervisor scope, failure of one child does not automatically cancel its siblings.

```kotlin
runBlocking {
    supervisorScope {
        launch {
            throw IOException("Failed")
        }

        launch {
            delay(100)
            println("Second child continues")
        }
    }
}
```

The second child can continue even though the first child fails.

The failure of the first child still needs appropriate exception handling or reporting.

### Comparison

| coroutineScope | supervisorScope |
| --- | --- |
| Child failure cancels parent | Child failure does not automatically cancel parent |
| Sibling children are cancelled on child failure | Siblings can continue |
| Parent failure cancels children | Parent failure cancels children |
| Used for tightly related tasks | Used for independent tasks |

---

# PART 7: STRUCTURED CONCURRENCY

## 27. What is Structured Concurrency?

Structured concurrency ensures that coroutines have a well-defined parent-child relationship and lifecycle.

A parent coroutine does not complete until its children complete.

If a parent is cancelled, its children are also cancelled.

If a child fails with an ordinary exception under a regular Job, the failure propagates upward.

Example:

```kotlin
runBlocking {
    launch {
        launch {
            delay(1000)
            println("Grandchild")
        }

        println("Child")
    }

    println("Parent")
}
```

Possible output:

```text
Parent
Child
Grandchild
```

`runBlocking` waits for all its children and descendants to finish.

### Why is structured concurrency important?

It prevents:

- Uncontrolled background tasks.
- Unnecessary resource consumption.
- Coroutines continuing after their owner is destroyed.
- Difficult-to-manage cancellation.
- Unhandled lifecycle-related failures.

For Android development, lifecycle-aware scopes such as `viewModelScope` are particularly useful.

---

# PART 8: IMPORTANT EXECUTION RULES

## 28. Quick Reference Table

| Concept | Behaviour |
| --- | --- |
| runBlocking | Starts a coroutine and blocks the calling thread |
| launch | Starts a coroutine and returns Job |
| async | Starts a coroutine and returns Deferred |
| await | Suspends until a Deferred result is available |
| join | Suspends until a Job completes |
| cancel | Requests cancellation |
| cancelAndJoin | Cancels and waits for completion |
| delay | Suspends for a specified duration |
| yield | Gives other coroutines an execution opportunity |
| withContext | Executes a block in a modified context |
| ensureActive | Throws if the current coroutine is cancelled |
| isActive | Checks whether the coroutine is active |
| NonCancellable | Allows necessary cleanup to suspend during cancellation |
| CoroutineExceptionHandler | Reports eligible uncaught coroutine exceptions |
| supervisorScope | Prevents child failure from automatically cancelling siblings |

---

# PART 9: INTERVIEW MCQ QUESTIONS

Try answering these questions before checking the answers.

## MCQ 1: Coroutine vs Thread

Which statement is correct?

A. Every coroutine creates a new thread.
B. A coroutine is the same as a thread.
C. Multiple coroutines can execute using the same thread.
D. A coroutine always executes on Dispatchers.IO.

**Correct answer: C**

*Explanation:* Coroutines are lightweight computations that can share threads.

## MCQ 2: runBlocking

What does runBlocking do?

A. Creates a new thread every time.
B. Creates a coroutine and blocks the calling thread until completion.
C. Starts a coroutine without waiting.
D. Executes only on Dispatchers.Default.

**Correct answer: B**

## MCQ 3: launch Execution

What is the output?

```kotlin
runBlocking {
    launch {
        print("A")
    }

    print("B")
}
```

A. AB
B. BA
C. AABB
D. Unpredictable in this exact default setup.

**Correct answer: B**

*Explanation:* With the default runBlocking event loop, the parent continues and prints B before the scheduled child executes.

## MCQ 4: async

When does async start executing by default?

A. Only when await() is called.
B. Only when join() is called.
C. It is scheduled immediately by default.
D. It never starts automatically.

**Correct answer: C**

*Explanation:* The default start mode is CoroutineStart.DEFAULT.

## MCQ 5: await()

What does await() do?

A. Blocks the current thread.
B. Cancels the coroutine.
C. Suspends the caller until the Deferred result is available.
D. Starts a new thread.

**Correct answer: C**

## MCQ 6: join()

What happens when you call job.join()?

A. The target coroutine is cancelled.
B. The target coroutine's result is returned.
C. The calling coroutine suspends until the target Job completes.
D. The calling thread is always blocked.

**Correct answer: C**

## MCQ 7: Cancellation

Which statement about cancellation is correct?

A. Cancellation forcibly kills the coroutine at any instruction.
B. Cancellation is cooperative.
C. Cancellation always blocks the thread.
D. Cancellation is impossible during delay().

**Correct answer: B**

## MCQ 8: finally

Does finally execute when a coroutine is cancelled?

A. Never.
B. Only when using async.
C. Yes, during normal cancellation cleanup.
D. Only when using runBlocking.

**Correct answer: C**

## MCQ 9: CancellationException

What should generally happen to CancellationException?

A. It should always be swallowed.
B. It should normally be rethrown if caught.
C. It should be converted into IOException.
D. It should be ignored using CoroutineExceptionHandler.

**Correct answer: B**

## MCQ 10: Exception Handling

Can an outer synchronous try-catch catch an exception thrown later inside launch?

A. Yes, always.
B. No, not merely because launch was called inside the try block.
C. Only when using Dispatchers.IO.
D. Only when using delay().

**Correct answer: B**

## MCQ 11: SupervisorScope

What happens if one child fails inside supervisorScope?

A. All siblings are automatically cancelled.
B. The entire application terminates.
C. Siblings are allowed to continue.
D. The failed child automatically restarts.

**Correct answer: C**

## MCQ 12: yield()

What does yield() do?

A. Blocks the thread for one second.
B. Creates another coroutine.
C. Suspends the current coroutine and gives the dispatcher an opportunity to execute other work.
D. Cancels all child coroutines.

**Correct answer: C**

## MCQ 13: Dispatcher

Which dispatcher is typically suitable for CPU-intensive work?

A. Dispatchers.Main
B. Dispatchers.IO
C. Dispatchers.Default
D. Dispatchers.Unconfined

**Correct answer: C**

## MCQ 14: cancelAndJoin()

What is the correct behaviour?

A. Only cancels the coroutine.
B. Only waits for completion.
C. Cancels the coroutine and waits for it to complete.
D. Starts the coroutine again.

**Correct answer: C**

## MCQ 15: withContext

What happens to the original dispatcher after withContext(Dispatchers.Default) completes?

A. It permanently changes to Default.
B. It resumes in the original coroutine context.
C. It changes to IO.
D. It creates a new scope that never finishes.

**Correct answer: B**

---

# PART 10: OUTPUT-BASED INTERVIEW CHALLENGES

These questions are designed to test whether you understand coroutine scheduling rather than simply memorising definitions.

## Challenge 1

```kotlin
runBlocking {
    launch {
        print("A")
    }

    launch {
        print("B")
    }

    print("C")
}
```

**Question:** What is the output with the default runBlocking dispatcher?

**Answer:**

```text
CAB
```

*Explanation:* The parent prints C before the scheduled children execute. The children are then processed in their scheduling order.

## Challenge 2

```kotlin
runBlocking {
    val deferred = async {
        delay(100)
        "A"
    }

    print("B")
    print(deferred.await())
    print("C")
}
```

**Answer:**

```text
BAC
```

*Explanation:* B executes before await suspends the parent. After the Deferred completes, A is printed, followed by C.

## Challenge 3

```kotlin
runBlocking {
    launch {
        print("A")
        delay(100)
        print("B")
    }

    print("C")
}
```

**Answer:**

```text
CAB
```

*Explanation:* The parent prints C. The child then prints A, suspends at delay, and prints B after resuming.

## Challenge 4

```kotlin
runBlocking {
    launch {
        print("A")
        yield()
        print("B")
    }

    print("C")
}
```

**Answer:**

```text
CAB
```

*Explanation:* The parent prints C, the child prints A, yields, and then resumes to print B. The exact scheduling can vary if other work is introduced.

## Challenge 5

```kotlin
runBlocking {
    val job = launch {
        try {
            delay(Long.MAX_VALUE)
        } finally {
            print("A")
        }
    }

    job.cancelAndJoin()
    print("B")
}
```

**Answer:**

```text
AB
```

*Explanation:* Cancellation triggers finally. `cancelAndJoin()` waits for completion before the parent prints B.

## Challenge 6

```kotlin
runBlocking {
    val deferred = async {
        "A"
    }

    print(deferred.await())
    print("B")
}
```

**Answer:**

```text
AB
```

*Explanation:* await() returns the completed result before execution continues.

---

# CONCLUSION

Understanding Kotlin Coroutines is not just about knowing `launch`, `async`, or `delay`.

The most important concepts are:

1. **Execution:** Coroutines are scheduled units of work, not threads.
2. **Suspension:** A coroutine can pause without blocking its executing thread.
3. **Dispatchers:** Dispatchers determine where coroutine code executes.
4. **Structured concurrency:** Parent and child coroutines have linked lifecycles.
5. **Cancellation:** Cancellation is cooperative and must be handled correctly.
6. **Exception handling:** Exceptions propagate according to Job hierarchy and coroutine builder semantics.
7. **Supervision:** `supervisorScope` and `SupervisorJob` allow independent child failures to be handled separately.

The best way to master coroutines is to predict execution order, understand cancellation, and trace exception propagation through parent-child relationships.

**Remember:** A coroutine can be suspended without blocking a thread, cancelled without being forcibly terminated, and failed without the exception being caught by the surrounding synchronous try-catch.

---

## Official Kotlin Documentation

For further reading and deeper technical details:

- [Kotlin Coroutines Basics](https://kotlinlang.org/docs/coroutines-basics.html)
- [Coroutine Cancellation](https://kotlinlang.org/docs/cancellation-and-timeouts.html)
- [Coroutine Exception Handling](https://kotlinlang.org/docs/exception-handling.html)
- [Coroutine Builders API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/)
- [kotlinx.coroutines GitHub](https://github.com/Kotlin/kotlinx.coroutines)
