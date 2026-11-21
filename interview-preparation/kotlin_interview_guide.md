# Kotlin Interview Preparation

A focused study guide covering Kotlin collection operations, functional-style aggregation, performance trade-offs, interface delegation, and property delegation. It includes code examples, interview-ready answers, a knowledge check, and a final revision checklist.

## Contents

1. [Functional programming and Kotlin collections](#1-functional-programming-and-kotlin-collections)
   - `fold()` vs `foldRight()`
   - `Sequence` vs `List`
   - Intermediate collections and aggregation
   - Mutable accumulators and performance
   - `fold()` vs imperative loops
2. [Kotlin interface delegation](#2-kotlin-delegation--deeper-explanation)
   - The problem delegation solves
   - The `by` keyword and generated forwarding methods
   - Benefits, decorators, and testability
   - Inheritance vs delegation
   - Overriding delegated methods
3. [Property delegation](#3-dont-confuse-interface-delegation-with-property-delegation)
   - `getValue()` and `setValue()`
   - Custom and built-in delegates
4. [Delegation interview challenge](#4-interview-challenge-test-your-understanding-of-delegation)
5. [Final interview revision checklist](#5-what-i-would-memorize-for-the-thermondo-interview)
6. [Inline functions: `inline`, `noinline`, and `crossinline`](#6-inline-functions-inline-noinline-and-crossinline)
7. [Generics, PECS, covariance, and contravariance](#7-generics-pecs-covariance-and-contravariance)
8. [Threads, concurrency, and parallelism](#8-threads-concurrency-and-parallelism)
9. [Offline-first Android design with SSOT](#9-offline-first-android-design-with-ssot)
10. [Kotlin exception handling and the Throwable hierarchy](#10-kotlin-exception-handling-and-the-throwable-hierarchy)

---

# 1. Functional programming and Kotlin collections
Let's go through the six questions from an interview perspective, including how the Kotlin compiler and collection operations behave.

## Q1. What is the difference between `fold()` and `foldRight()`?
Both functions accumulate collection elements into a single result. The difference is the **direction of iteration and the order of the lambda parameters**.

### `fold()` — left to right

```
val numbers = listOf(1, 2, 3)

val result = numbers.fold("Start") { accumulator, element ->
    "($accumulator + $element)"
}

println(result)
```

Output:

```
(((Start + 1) + 2) + 3)
```

Execution:

| Step    | Accumulator       | Element | New accumulator         |
| ------- | ----------------- | ------- | ----------------------- |
| Initial | Start             |         | Start                   |
| 1       | Start             | 1       | (Start + 1)             |
| 2       | (Start + 1)       | 2       | ((Start + 1) + 2)       |
| 3       | ((Start + 1) + 2) | 3       | (((Start + 1) + 2) + 3) |

### `foldRight()` — right to left

```
val numbers = listOf(1, 2, 3)

val result = numbers.foldRight("Start") { element, accumulator ->
    "($element + $accumulator)"
}

println(result)
```

Output:

```
(1 + (2 + (3 + Start)))
```

Execution:

| Step    | Element | Accumulator       | New accumulator         |
| ------- | ------- | ----------------- | ----------------------- |
| Initial |         | Start             | Start                   |
| 1       | 3       | Start             | (3 + Start)             |
| 2       | 2       | (3 + Start)       | (2 + (3 + Start))       |
| 3       | 1       | (2 + (3 + Start)) | (1 + (2 + (3 + Start))) |

### Main differences
| Feature                | fold                  | foldRight                    |
| ---------------------- | --------------------- | ---------------------------- |
| Direction              | Left to right         | Right to left                |
| First lambda argument  | Accumulator           | Element                      |
| Second lambda argument | Element               | Accumulator                  |
| Initial value          | Required              | Required                     |
| Empty list             | Returns initial value | Returns initial value        |
| Common use             | Accumulation          | Right-associative operations |

Both are O(n) time operations.

**When would you use `foldRight()`?**

When the order of combining elements matters and you need right-associative processing.

For example, constructing a nested expression, processing a list from the end, or composing functions in a particular order.

One important detail: Kotlin's `foldRight()` is defined for `List` and arrays, whereas `fold()` is available for general `Iterable` collections as well as sequences.

Reference: [Kotlin foldRight documentation](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/fold-right.html?utm_source=chatgpt.com) and [Kotlin fold documentation](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/fold.html?utm_source=chatgpt.com).

## Q2. When would you prefer `Sequence` over `List`?
This question is about **eager versus lazy evaluation**.

A List normally evaluates each operation immediately and creates a result collection.

A Sequence evaluates operations when you request the final result.

### Example 1: List (eager evaluation)
```
val numbers = listOf(1, 2, 3, 4, 5)

val result = numbers
    .filter {
        println("Filtering $it")
        it > 2
    }
    .map {
        println("Mapping $it")
        it * 2
    }

println(result)
```

Execution:

```
Filtering 1
Filtering 2
Filtering 3
Filtering 4
Filtering 5

Mapping 3
Mapping 4
Mapping 5

[6, 8, 10]
```

Why?

1. `filter()` traverses the entire List and creates a new List.
2. `map()` traverses the filtered List and creates another List.
3. The final result contains the mapped values.

### Example 2: Sequence (lazy evaluation)
```
val numbers = listOf(1, 2, 3, 4, 5)

val result = numbers
    .asSequence()
    .filter {
        println("Filtering $it")
        it > 2
    }
    .map {
        println("Mapping $it")
        it * 2
    }

println("Sequence created")

println(result.toList())
```

Execution:

```
Sequence created

Filtering 1
Filtering 2
Filtering 3
Mapping 3
Filtering 4
Mapping 4
Filtering 5
Mapping 5

[6, 8, 10]
```

Notice the difference.

A Sequence processes an element through the entire pipeline before moving to the next element.

**How execution differs**

List

**First operation:** Filter all 5 elements

**Second operation:** Map all 3 matching elements

Final List

Sequence

Element 1 → Filter

Element 2 → Filter

Element 3 → Filter → Map

Element 4 → Filter → Map

Element 5 → Filter → Map

Final result

### When is Sequence better?
Consider this example:

```
val result = numbers
    .asSequence()
    .filter { it > 100 }
    .map { it * 2 }
    .take(5)
    .toList()
```

Imagine you have one million records, but you only need the first five matching records.

With a List pipeline, filtering and mapping happen for all matching items before `take(5)` is applied.

With a Sequence, processing can stop once five matching items have been found.

**Prefer Sequence when:**

- Processing large collections.
- Applying multiple transformations.
- You only need a limited number of results.
- You want lazy evaluation.
- You want to avoid intermediate collections.

**Prefer List when:**

- Working with small collections.
- Operations are simple.
- You need to reuse the materialized result.
- The overhead of lazy processing is unnecessary.

Important: Sequence is not automatically faster. It introduces iterator and pipeline overhead, which can make it slower for small collections.

## Q3. Does `filter().map().sumOf()` create intermediate collections?
**Yes, when you use the regular List/Iterable operations.**

Consider:

```
val total = transactions
    .filter { it.isPaid }
    .map { it.amount }
    .sumOf { it }
```

Let's understand what happens internally.

**Collection pipeline**

**Original List**

10,000 transactions

**filter **

Traverses the original List and creates a new List of paid transactions.

**map **

Creates another List containing amounts.

**sumOf **

Traverses the mapped List and produces one numeric result.

Let's count the traversals:

| Operation  | Traversals | Intermediate allocation |
| ---------- | ---------- | ----------------------- |
| `filter()` | 1          | Filtered List           |
| `map()`    | 1          | Mapped List             |
| `sumOf()`  | 1          | No result collection    |

The complexity remains O(n), but the operations make multiple passes and allocate intermediate collections.

### Alternative 1: Sequence
```
val total = transactions
    .asSequence()
    .filter { it.isPaid }
    .map { it.amount }
    .sumOf { it }
```

No intermediate result Lists are created by the filtering and mapping pipeline. The terminal `sumOf` processes the sequence.

### Alternative 2: Fold
```
val total = transactions.fold(0L) { acc, transaction ->
    if (transaction.isPaid) {
        acc + transaction.amount
    } else {
        acc
    }
}
```

This performs:

- One traversal.
- No intermediate collection.
- One accumulator.

### Alternative 3: sumOf (my preferred solution here)
```
val total = transactions.sumOf { transaction ->
    if (transaction.isPaid) transaction.amount else 0L
}
```

This also performs one traversal and avoids intermediate collections.

**Interview takeaway:** If the task is simply filtering by a condition and adding values, `sumOf` is often the clearest solution. Fold is useful when the aggregation logic is more complex.

## Q4. What happens when your accumulator is mutable?
This is an important question because it tests your understanding of functional programming.

A fold accumulator does not have to be immutable.

You can use a mutable object and modify it during each iteration.

Example:

```
val numbers = listOf(1, 2, 3, 4)

val result = numbers.fold(mutableListOf<Int>()) { acc, number ->
    acc.add(number * 2)
    acc
}

println(result)
```

Output:

```
[2, 4, 6, 8]
```

What happens?

1. `mutableListOf()` creates the initial accumulator.
2. The first element is processed.
3. The accumulator is mutated.
4. The same accumulator is returned.
5. This repeats for each element.

Notice that `acc.add()` modifies the existing collection. We explicitly return `acc` because the lambda must return the accumulator type.

### Is this still functional programming?
Not in the strict pure-functional sense.

A pure fold typically uses immutable transformations:

```
val result = numbers.fold(emptyList<Int>()) { acc, number ->
    acc + (number * 2)
}
```

But this implementation has a disadvantage.

`acc + value` creates a new List each time. With a regular List, repeated copying can make the total work O(n²).

The mutable implementation can be O(n), with O(n) storage.

### Is a mutable accumulator dangerous?
Not necessarily.

It is safe and often efficient when:

- The accumulator is created inside the operation.
- It is not shared with other threads.
- You don't expose intermediate states.
- You don't reuse the same mutable accumulator across unrelated calculations.

One subtle detail:

```
val initial = mutableListOf<Int>()

val result = emptyList<Int>().fold(initial) { acc, value ->
    acc.apply { add(value) }
}

println(result === initial) // true
```

For an empty collection, `fold()` returns the exact initial accumulator object.

Therefore, an externally shared mutable initial value can introduce unexpected aliasing.

**Staff-level takeaway:** Mutability is not inherently bad. The important thing is ownership, encapsulation, and whether the accumulator can be accessed by other code.

## Q5. Can fold introduce performance problems?
Yes. The problem is usually not `fold()` itself, but what you do inside its lambda.

### Example 1: Inefficient immutable accumulation
```
val result = numbers.fold(emptyList<Int>()) { acc, number ->
    acc + number
}
```

Why can this be inefficient?

`acc + number` creates a new List and copies the previous elements.

If you process 100,000 elements, you can end up copying a large number of elements repeatedly.

Approximate time complexity can become O(n²).

### Example 2: Efficient mutable accumulation
```
val result = numbers.fold(mutableListOf<Int>()) { acc, number ->
    acc.apply {
        add(number)
    }
}
```

This is approximately O(n) for appending to an ArrayList-backed mutable list.

### Example 3: Repeated string concatenation
```
val result = words.fold("") { acc, word ->
    acc + word
}
```

Repeated string concatenation can cause repeated allocation and copying because Strings are immutable.

Better:

```
val result = buildString {
    words.forEach { word ->
        append(word)
    }
}
```

Or:

```
val result = words.joinToString("")
```

### Other considerations
| Potential issue             | Explanation                                                 |
| --------------------------- | ----------------------------------------------------------- |
| Repeated collection copying | Can turn an O(n) operation into O(n²).                      |
| Large accumulator           | Stores substantial state in memory.                         |
| Complex lambda              | Makes performance and code behavior harder to reason about. |
| Unnecessary object creation | Creates additional garbage collection pressure.             |
| Side effects                | Makes the accumulator harder to reason about and test.      |

Note that Kotlin's standard `fold()` is an inline function for Iterable collections, so the lambda itself does not necessarily introduce an extra lambda-object allocation on the JVM.

**The important distinction:** `fold()` is not automatically expensive. The accumulator implementation determines much of its performance.

## Q6. Is fold always faster than an imperative loop?
No.

Consider these two implementations.

**Using fold:**

```
fun sum(numbers: List<Int>): Int {
    return numbers.fold(0) { acc, number ->
        acc + number
    }
}
```

**Using a traditional loop:**

```
fun sum(numbers: List<Int>): Int {
    var total = 0

    for (number in numbers) {
        total += number
    }

    return total
}
```

Both are O(n) time and O(1) auxiliary space.

The standard `fold()` on Iterable is inline, so its lambda can be inlined by the compiler. Therefore, the difference may be very small.

But an imperative loop can sometimes be easier for the compiler to optimize and avoids the abstraction when complex control flow is required.

For example:

```
fun findFirstPaidTransaction(
    transactions: List<Transaction>
): Transaction? {
    for (transaction in transactions) {
        if (transaction.isPaid) {
            return transaction
        }
    }

    return null
}
```

You could use `firstOrNull`, but fold would be a less natural solution because it cannot stop iterating early using an ordinary non-local return in the same way.

### When to use which?
| Situation                        | Preferred approach |
| -------------------------------- | ------------------ |
| Simple sum                       | `sumOf()`          |
| Complex accumulation             | `fold()`           |
| Transforming values              | `map()`            |
| Finding first match              | `firstOrNull()`    |
| Early exit and complex branching | `for` loop         |
| Lazy pipeline                    | `Sequence`         |
| Simple collection processing     | List operations    |

**My answer in an interview:**

"Fold isn't inherently faster than a loop. Both can have O(n) time complexity. Kotlin's fold is inline, which reduces abstraction overhead. However, for straightforward aggregation, I prefer sumOf, while for more complicated accumulations I use fold. When I need early exits or complicated control flow, I prefer a traditional loop. For performance-critical code, I benchmark rather than assuming one approach is faster."

# 2. Kotlin Delegation — deeper explanation
Very high priority for your Thermondo interview

Delegation is one of the most useful Kotlin features for implementing **composition over inheritance**.

The key idea is:

> Instead of inheriting functionality from another class, your class receives an object that already implements the required functionality and delegates some or all of its behavior to that object.

Let's understand why this matters.

## Q7. What problem does delegation solve?
Imagine you have a payment system.

You start with an interface:

```
interface PaymentProcessor {
    fun pay(amount: Double)
    fun refund(transactionId: String)
    fun cancel(transactionId: String)
}
```

And an implementation:

```
class StripeProcessor : PaymentProcessor {

    override fun pay(amount: Double) {
        println("Stripe payment: $amount")
    }

    override fun refund(transactionId: String) {
        println("Stripe refund: $transactionId")
    }

    override fun cancel(transactionId: String) {
        println("Stripe cancellation: $transactionId")
    }
}
```

Now imagine you want to add logging to payment operations.

One approach is inheritance.

```
class LoggingProcessor : StripeProcessor() {
    // Add logging
}
```

But this approach has problems:

1. `LoggingProcessor` is tightly coupled to `StripeProcessor`.
2. You cannot easily replace Stripe with another payment provider.
3. You cannot extend a final class without changing its design.
4. You inherit implementation details that might not belong to your class.
5. You are limited by Kotlin's single class inheritance.

What if you want to support Stripe, PayPal, or a mock implementation?

This is where composition and delegation become useful.

## Q8. How does Kotlin delegation work with `by`?
First, let's implement composition manually.

```
class LoggingProcessor(
    private val processor: PaymentProcessor
) : PaymentProcessor {

    override fun pay(amount: Double) {
        println("Starting payment")

        processor.pay(amount)
    }

    override fun refund(transactionId: String) {
        processor.refund(transactionId)
    }

    override fun cancel(transactionId: String) {
        processor.cancel(transactionId)
    }
}
```

Notice something?

We have to implement all three methods, even though we only wanted to add logging to one of them.

This is boilerplate.

Kotlin solves this with interface delegation.

### Using `by`

```
class LoggingProcessor(
    private val processor: PaymentProcessor
) : PaymentProcessor by processor {

    override fun pay(amount: Double) {
        println("Starting payment")

        processor.pay(amount)
    }
}
```

That's it!

Kotlin automatically delegates `refund()` and `cancel()` to the supplied processor.

You only implement the methods you want to customize.

### What does the compiler effectively generate?
Conceptually, Kotlin transforms the class into something similar to:

```
class LoggingProcessor(
    private val processor: PaymentProcessor
) : PaymentProcessor {

    override fun pay(amount: Double) {
        println("Starting payment")
        processor.pay(amount)
    }

    override fun refund(transactionId: String) {
        processor.refund(transactionId)
    }

    override fun cancel(transactionId: String) {
        processor.cancel(transactionId)
    }
}
```

The compiler generates forwarding implementations for the delegated interface members that you haven't overridden.

This is why the feature reduces boilerplate.

## Q9. What are the real benefits of delegation?
Let's look at them from an architectural perspective.

### Benefit 1: Composition instead of inheritance
You can inject any implementation of the interface.

```
val stripe = StripeProcessor()

val processor = LoggingProcessor(stripe)
```

Or:

```
val paypal = PayPalProcessor()

val processor = LoggingProcessor(paypal)
```

You reuse the logging behavior without requiring a subclass for every provider.

### Benefit 2: Runtime flexibility
The same wrapper can work with different implementations.

```
interface Logger {
    fun log(message: String)
}

class ConsoleLogger : Logger {
    override fun log(message: String) {
        println(message)
    }
}

class FileLogger : Logger {
    override fun log(message: String) {
        println("Writing to file: $message")
    }
}
```

```
class ApplicationLogger(
    private val logger: Logger
) : Logger by logger
```

You can inject a different implementation according to your environment.

For example:

- Production: FileLogger.
- Development: ConsoleLogger.
- Unit tests: FakeLogger.

This makes testing and dependency injection easier.

### Benefit 3: Implement multiple interfaces without multiple inheritance
Kotlin does not support multiple class inheritance.

But you can implement and delegate multiple interfaces.

```
interface Logger {
    fun log(message: String)
}

interface Analytics {
    fun track(event: String)
}

class ConsoleLogger : Logger {
    override fun log(message: String) {
        println(message)
    }
}

class FirebaseAnalytics : Analytics {
    override fun track(event: String) {
        println("Tracking: $event")
    }
}
```

Now:

```
class ApplicationService(
    logger: Logger,
    analytics: Analytics
) : Logger by logger,
    Analytics by analytics
```

Your class now has both capabilities.

It doesn't need to inherit implementation from either concrete class.

### Benefit 4: Decorator pattern
Delegation is particularly useful for adding behavior without changing the original implementation.

For example:

```
class CachedPaymentProcessor(
    private val processor: PaymentProcessor
) : PaymentProcessor by processor {

    private val cache = mutableMapOf<String, Boolean>()

    // You can override selected methods
}
```

You can build:

- Logging decorators.
- Caching decorators.
- Metrics decorators.
- Validation decorators.
- Retry decorators.

Each decorator can add one responsibility while forwarding the rest.

This is a good example of the Open/Closed Principle: you can extend behavior without modifying the original implementation.

### Benefit 5: Testability and dependency isolation
Suppose your service requires a repository.

```
interface UserRepository {
    fun getUser(id: Int): String
}
```

You can inject a fake implementation:

```
class FakeUserRepository : UserRepository {
    override fun getUser(id: Int): String {
        return "Test User"
    }
}
```

Then:

```
class LoggingUserRepository(
    private val repository: UserRepository
) : UserRepository by repository
```

You can test the wrapper independently of the real database or network implementation.

### Summary: Delegation benefits
| Benefit               | Why it matters                                    |
| --------------------- | ------------------------------------------------- |
| Loose coupling        | Depends on interfaces instead of concrete classes |
| Reusability           | Reuses existing implementations                   |
| Flexibility           | Different implementations can be injected         |
| Testability           | Easily replace dependencies with fakes or mocks   |
| Less boilerplate      | Compiler generates forwarding methods             |
| Multiple capabilities | Delegate multiple interfaces                      |
| Decorator pattern     | Add behavior without modifying original classes   |

## Q10. What is the difference between inheritance and delegation?
This is an important follow-up interview question.

| Inheritance                             | Delegation                             |
| --------------------------------------- | -------------------------------------- |
| Extends a parent class                  | Uses another object                    |
| Establishes an inheritance relationship | Establishes a composition relationship |
| Inherits implementation                 | Forwards interface behavior            |
| Can override inherited methods          | Can override delegated methods         |
| Coupled to parent implementation        | Coupled to the delegated interface     |
| Kotlin supports one class superclass    | Can delegate multiple interfaces       |

Consider:

```
class A : ParentClass()
```

This says A inherits from ParentClass.

Whereas:

```
class A(
    private val dependency: SomeInterface
) : SomeInterface by dependency
```

This says A implements SomeInterface and delegates its implementation to another object.

**Important:** Kotlin's `by` interface delegation only delegates interface implementations. It is not a replacement for inheriting from a concrete class.

## Q11. What happens when you override a delegated method?
Consider this example:

```
interface Printer {
    fun print()
    fun printLine()
}

class BasePrinter : Printer {

    override fun print() {
        println("Base print")
    }

    override fun printLine() {
        print()
    }
}
```

Now:

```
class CustomPrinter(
    private val printer: Printer
) : Printer by printer {

    override fun print() {
        println("Custom print")
    }
}
```

What happens?

```
val printer = CustomPrinter(BasePrinter())

printer.print()
printer.printLine()
```

Output:

```
Custom print
Base print
```

Why doesn't `printLine()` call `CustomPrinter.print()`?

This is a very important implementation detail.

1. `printLine()` is delegated to `BasePrinter`.
2. `BasePrinter.printLine()` executes using the BasePrinter instance as its receiver.
3. Its internal call to `print()` therefore calls `BasePrinter.print()`.
4. It doesn't dispatch back through the wrapper.

The delegated object doesn't automatically know about the wrapper's overridden methods.

This matters when implementing decorators for logging, monitoring, caching, and validation.

# 3. Don't confuse interface delegation with property delegation
Both use the `by` keyword, but they solve different problems.

| Feature      | Interface delegation           | Property delegation           |
| ------------ | ------------------------------ | ----------------------------- |
| Purpose      | Reuse interface implementation | Delegate property access      |
| Syntax       | `Interface by object`          | `property by delegate`        |
| Main methods | Interface methods              | `getValue()` and `setValue()` |
| Example      | `Logger by logger`             | `name by lazy {}`             |

## Q12. What is property delegation?
Consider:

```
val database by lazy {
    createDatabase()
}
```

Normally, you would need to implement the lazy initialization logic yourself:

```
class DatabaseManager {

    private var database: Database? = null

    fun getDatabase(): Database {
        if (database == null) {
            database = createDatabase()
        }

        return database!!
    }
}
```

With property delegation:

```
class DatabaseManager {

    val database by lazy {
        createDatabase()
    }
}
```

Kotlin takes responsibility for delegating the property's initialization and access behavior.

### How does it work internally?
A property delegate provides the following operator functions:

```
operator fun getValue(
    thisRef: Any?,
    property: KProperty<*>
): T
```

For mutable properties, it also provides:

```
operator fun setValue(
    thisRef: Any?,
    property: KProperty<*>,
    value: T
)
```

The compiler uses these functions when accessing or assigning the delegated property.

## Q13. How can you implement your own property delegate?
Let's create a delegate that automatically trims all assigned strings.

```
import kotlin.reflect.KProperty

class TrimmedStringDelegate {

    private var value: String = ""

    operator fun getValue(
        thisRef: Any?,
        property: KProperty<*>
    ): String {
        return value
    }

    operator fun setValue(
        thisRef: Any?,
        property: KProperty<*>,
        newValue: String
    ) {
        value = newValue.trim()
    }
}
```

Use it:

```
class User {

    var name: String by TrimmedStringDelegate()
}
```

Now:

```
val user = User()

user.name = "  Ashraf  "

println(user.name)
```

Output:

```
Ashraf
```

The delegate controls how the property is read and written.

## Q14. What are the most common built-in property delegates?
**1. lazy**

Initializes a property only on its first access.

```
val configuration by lazy {
    loadConfiguration()
}
```

By default, lazy initialization is synchronized.

**2. observable**

Notifies you after a property changes.

```
var username: String by Delegates.observable("") {
    property, oldValue, newValue ->

    println("$oldValue -> $newValue")
}
```

**3. vetoable**

Allows you to reject an attempted property change.

```
var age: Int by Delegates.vetoable(18) {
    property, oldValue, newValue ->

    newValue >= 18
}
```

**4. Map delegation**

Useful for reading values from a Map.

```
class User(
    private val values: Map<String, Any>
) {
    val name: String by values
    val age: Int by values
}
```

The map must contain the appropriate keys and values.

# 4. Interview challenge: Test your understanding of delegation

Try answering each question before reading the explanation.

### 1. What does Kotlin generate for `Logger by logger`?

```kotlin
class AppLogger(private val logger: Logger) : Logger by logger
```

- A. A new `Logger` implementation with no methods
- B. Forwarding implementations for delegated interface members
- C. A subclass of the logger class
- D. A property getter only

**Correct answer: B.** The compiler generates forwarding methods for interface members that the class has not overridden.

### 2. Can Kotlin delegate multiple interfaces?

```kotlin
class Service(a: A, b: B) : A by a, B by b
```

- A. No
- B. Yes
- C. Only if `A` extends `B`
- D. Only with abstract classes

**Correct answer: B.** Kotlin allows a class to delegate multiple interfaces to different objects.

### 3. What happens in this code?

```kotlin
class Wrapper(private val printer: Printer) : Printer by printer {
    override fun print() = println("Wrapper")
}
```

- A. All methods print `Wrapper`
- B. Only the overridden `print()` method uses the wrapper implementation
- C. Compilation error
- D. The delegate is replaced

**Correct answer: B.** Unoverridden methods are still forwarded to the delegate.

### 4. Which operator does a read-only property delegate need?

```kotlin
val name: String by delegate
```

- A. `setValue()` only
- B. `getValue()`
- C. `invoke()`
- D. `provideDelegate()` only

**Correct answer: B.** A read-only delegated property requires `getValue()`. A mutable property also needs `setValue()`.

### 5. What is the primary advantage of composition over inheritance?

```kotlin
class Service(private val repository: Repository)
```

- A. More inheritance levels
- B. Runtime flexibility and reduced coupling
- C. Automatic deep copying
- D. No need for interfaces

**Correct answer: B.** Composition allows interchangeable implementations without inheriting from a concrete class.

### 6. What happens when a delegated object's method internally calls another interface method?

```kotlin
class Base : Printer {
    override fun print() = println("Base")
    override fun printLine() = print()
}

class Wrapper(private val printer: Printer) : Printer by printer {
    override fun print() = println("Wrapper")
}
```

- A. `Wrapper.print()` is always called
- B. `Base.print()` is called by `Base.printLine()`
- C. Compilation error
- D. Both are called

**Correct answer: B.** The internal call uses the delegate's own receiver, not the wrapper's override.

---

**Key point:** Delegation forwards calls to the delegate object. An override in the wrapper does not rewrite the delegate object's own internal method calls.

## 5. What I would memorize for the Thermondo interview
For the functional programming topic:

- `fold()` = left-to-right accumulation.
- `foldRight()` = right-to-left accumulation.
- `Sequence` = lazy evaluation.
- `List` operations = normally eager evaluation.
- `filter().map()` = intermediate Lists.
- `fold()` = general aggregation.
- `sumOf()` = preferred for straightforward sums.
- Mutable accumulators = potentially efficient, but require controlled ownership.
- `fold()` is not necessarily faster than loops.

For delegation:

- `by` allows interface implementation to be delegated to another object.
- It is an implementation of composition over inheritance.
- Kotlin generates forwarding methods.
- You can override selected delegated methods.
- You can delegate multiple interfaces.
- Property delegation is different: `getValue()` and `setValue()`.
- Understand the difference between the wrapper's overridden methods and internal calls made by the delegate.

**The most valuable Staff-level distinction:** Delegation is not just a way to avoid writing forwarding methods. It's an architectural tool to separate responsibilities, reduce coupling, and make implementations interchangeable.

---

# 6. Inline functions: `inline`, `noinline`, and `crossinline`

This topic tests how Kotlin handles lambdas, compiler inlining, and non-local control flow.

## Q15. What is a non-local return?

An ordinary lambda passed to an inline function can use `return` to return from the *enclosing function*, not just from the lambda. This is called a non-local return.

```kotlin
inline fun execute(action: () -> Unit) {
    println("Before")
    action()
    println("After")
}

fun main() {
    execute {
        println("Inside lambda")
        return
    }

    println("Main finished")
}
```

Output:

```text
Before
Inside lambda
```

The `return` exits `main()`, so neither `After` nor `Main finished` is printed. This is possible because the lambda parameter is inlined into the caller's control flow.

## Q16. What problem does `crossinline` solve?

`crossinline` tells the compiler that a lambda parameter must not make a non-local return. This is needed when the lambda is captured or invoked from another execution context, such as a nested lambda or an object expression.

```kotlin
inline fun execute(crossinline action: () -> Unit) {
    val runnable = Runnable {
        action()
    }

    runnable.run()
}
```

This is allowed because `action` is invoked inside the `Runnable` implementation.

But this call is not allowed:

```kotlin
fun main() {
    execute {
        println("Inside")
        return // Compilation error: non-local return is prohibited
    }

    println("Finished")
}
```

A local, labeled return is still allowed:

```kotlin
fun main() {
    execute {
        println("Inside")
        return@execute // Returns from this lambda only
    }

    println("Finished")
}
```

Output:

```text
Inside
Finished
```

Important: the nested execution context does not have to be asynchronous or run on another thread. A `Runnable` can be executed immediately on the same thread and still be a different control-flow context.

## Q17. How are `inline`, `crossinline`, and `noinline` different?

```kotlin
inline fun process(
    action: () -> Unit,
    noinline callback: () -> Unit,
    crossinline nestedAction: () -> Unit
) {
    action()

    // A noinline lambda is an ordinary function value and can be stored.
    val storedCallback: () -> Unit = callback
    storedCallback()

    // A crossinline lambda can be invoked from a nested lambda/object.
    val runnable = Runnable {
        nestedAction()
    }
    runnable.run()
}
```

| Feature | `inline` parameter | `crossinline` parameter | `noinline` parameter |
|---|---|---|---|
| Inlined by compiler | Yes | Yes | No |
| Non-local `return` | Allowed when otherwise valid | Prohibited | Prohibited |
| Can be captured in a nested lambda/object | Not without `crossinline` when this would permit an invalid non-local return | Yes | Yes |
| Can be stored/passed as an ordinary function value | No | Not as an ordinary escaping function value | Yes |
| Main reason to use it | Reduce lambda overhead and enable non-local returns/reified types | Preserve inlining while restricting non-local returns | Store, return, or pass the lambda as a normal value |

A useful mental model:

- `inline`: inline the function and its eligible lambda arguments.
- `crossinline`: inline the lambda, but prohibit a non-local return because it may be invoked through another execution context.
- `noinline`: do not inline this lambda; treat it as a regular function value.

## Q18. Why use `crossinline` in an Android callback helper?

```kotlin
inline fun View.setSafeClickListener(
    crossinline onClick: () -> Unit
) {
    setOnClickListener {
        onClick()
    }
}
```

The click lambda is captured by the listener and invoked later when a click occurs. It must not be able to return from the function that registered the listener.

Other common use cases include `Runnable`, callback registration, and object expressions that capture a lambda.

**Interview answer:**

"`crossinline` is used on an inline lambda parameter when the lambda needs to be captured or invoked inside another execution context. It preserves the possibility of inlining but prevents non-local returns, because the lambda may execute through a nested callback or object rather than directly in the caller's control flow. A labeled local return such as `return@execute` is still allowed."

---

# 7. Generics, PECS, covariance, and contravariance

The PECS principle is a practical way to remember how variance works:

> **Producer Extends, Consumer Super.**

In Kotlin, this maps to:

- Producer: `out T` (covariance).
- Consumer: `in T` (contravariance).
- Both producer and consumer: usually invariant `T`.

Start with this hierarchy:

```kotlin
open class Animal
class Dog : Animal()
class Cat : Animal()
```

## Q19. What is covariance (`out`)?

A producer returns values of type `T`.

```kotlin
interface Producer<out T> {
    fun get(): T
}

class DogProducer : Producer<Dog> {
    override fun get(): Dog = Dog()
}
```

Because `Dog` is an `Animal`, a producer of Dogs can safely be used where a producer of Animals is expected:

```kotlin
val dogProducer: Producer<Dog> = DogProducer()
val animalProducer: Producer<Animal> = dogProducer // Valid
```

Why is this safe? The caller expects an `Animal` and receives a `Dog`, which is an `Animal`.

A covariant `T` can be returned, but it cannot be accepted as an ordinary input parameter of a public member:

```kotlin
interface Producer<out T> {
    fun get(): T
    // fun add(value: T) // Compiler error: T is in an input position
}
```

If a `Producer<Dog>` were widened to `Producer<Animal>`, allowing `add(Cat())` would be unsafe for the underlying Dog producer.

## Q20. What is contravariance (`in`)?

A consumer accepts values of type `T`.

```kotlin
interface Consumer<in T> {
    fun add(value: T)
}

class AnimalConsumer : Consumer<Animal> {
    override fun add(value: Animal) {
        println("Consuming an animal")
    }
}
```

An `AnimalConsumer` can safely be used where a `Consumer<Dog>` is expected:

```kotlin
val animalConsumer: Consumer<Animal> = AnimalConsumer()
val dogConsumer: Consumer<Dog> = animalConsumer // Valid
```

It can also be used as a `Consumer<Cat>`:

```kotlin
val catConsumer: Consumer<Cat> = animalConsumer // Also valid
```

An Animal consumer can accept Dogs and Cats because both are Animals.

But a Cat-only consumer cannot be used as a Dog consumer:

```kotlin
class CatConsumer : Consumer<Cat> {
    override fun add(value: Cat) {
        println("Consuming a cat")
    }
}

val onlyCats: Consumer<Cat> = CatConsumer()
// val dogs: Consumer<Dog> = onlyCats // Compiler error
```

The reverse assignment is unsafe too: a Dog-only consumer cannot guarantee that it accepts Cats.

The subtype relationships are:

```text
Dog <: Animal
Cat <: Animal

Producer<Dog> <: Producer<Animal>
Consumer<Animal> <: Consumer<Dog>
Consumer<Animal> <: Consumer<Cat>
```

Covariance preserves the subtype direction. Contravariance reverses it.

## Q21. Can `in` and `out` be used together?

They cannot both be applied to the *same type parameter*:

```kotlin
// Invalid Kotlin syntax/variance declaration:
// interface Box<in out T>
```

But they can be used on different type parameters of the same declaration. This is a common pattern:

```kotlin
interface Transformer<in Input, out Output> {
    fun transform(input: Input): Output
}
```

- `Input` is consumed, so it is `in`.
- `Output` is produced, so it is `out`.

Example:

```kotlin
class AnimalToDog : Transformer<Animal, Dog> {
    override fun transform(input: Animal): Dog = Dog()
}

val transformer: Transformer<Dog, Animal> = AnimalToDog()
```

This is safe: the implementation can accept a Dog because it accepts any Animal, and it returns a Dog, which is an Animal.

## Q22. Why is `MutableList<T>` invariant?

`List` is declared as `List<out T>` because its public API only exposes read operations. It is a read-only interface, though the underlying collection is not necessarily immutable.

`MutableList<T>` both produces and consumes `T`:

- `get(index)` returns `T`.
- `add(element: T)` accepts `T`.

Therefore, it is invariant.

```kotlin
val dogs: MutableList<Dog> = mutableListOf(Dog())
// val animals: MutableList<Animal> = dogs // Compiler error
```

If that assignment were allowed, someone could add a Cat through `animals`, leaving a Cat inside a list that is supposed to contain only Dogs.

## Q23. What is use-site variance (type projection)?

Declaration-site variance (`out` or `in` on a type parameter) defines how a generic type behaves generally. Use-site variance restricts how a particular argument is used at a call site.

Kotlin's `Array<T>` is invariant, but a copy function can safely project its source and destination:

```kotlin
fun <T> copy(
    source: Array<out T>,
    destination: Array<in T>
) {
    for (index in source.indices) {
        destination[index] = source[index]
    }
}

val dogs = arrayOf(Dog(), Dog())
val animals = arrayOf(Animal(), Animal())

copy(dogs, animals)
```

- `Array<out T>` is a producer view: read values as `T`.
- `Array<in T>` is a consumer view: write values of `T`.

This corresponds to Java's `? extends T` and `? super T` wildcards.

## PECS interview cheat sheet

| Type | Meaning | Safe use |
|---|---|---|
| `Producer<out T>` | Produces `T` | `Producer<Dog>` can be used as `Producer<Animal>` |
| `Consumer<in T>` | Consumes `T` | `Consumer<Animal>` can be used as `Consumer<Dog>` or `Consumer<Cat>` |
| `Box<T>` | Reads and writes `T` | Invariant; no subtype widening by default |
| `Array<out T>` | Read-only projected view | Read `T` values |
| `Array<in T>` | Write-only projected view | Write `T` values; reads are only safely available as `Any?` |

**Interview answer:**

"PECS means Producer Extends, Consumer Super. In Kotlin, a producer is declared with `out` because it returns values, and a consumer is declared with `in` because it accepts values. `out` preserves the subtype relationship, while `in` reverses it. If a type both reads and writes the same generic type, it generally needs to be invariant. We can use `in` and `out` on separate type parameters, such as `Transformer<in Input, out Output>`, or use type projections at a call site."

---

# 8. Threads, concurrency, and parallelism

These concepts are related but not interchangeable.

## Q24. What is a thread?

A thread is an execution path within a process. A process can contain multiple threads. Threads in the same process share the process's memory and resources, while each thread has its own execution stack and execution state.

An illustrative JVM example:

```kotlin
fun main() {
    val worker = Thread {
        println("Worker: ${Thread.currentThread().name}")
    }

    worker.start()
    println("Main: ${Thread.currentThread().name}")
}
```

`start()` asks the runtime/operating system to start a new thread. Calling `run()` directly would just invoke the method on the current thread; it would not start a new thread.

## Q25. What is concurrency?

Concurrency is the structuring and execution of multiple tasks whose periods of progress overlap. It is about managing multiple tasks, not necessarily running them at the exact same instant.

A single CPU core can interleave tasks by switching between them. A single-threaded event loop can also manage multiple outstanding tasks by starting work, handling another event while the first operation waits, and resuming when an event arrives.

Therefore, concurrency does not require multiple threads.

## Q26. What is parallelism?

Parallelism means multiple tasks are executing at the same time, for example on different CPU cores.

| Concept | Meaning |
|---|---|
| Thread | An execution path in a process |
| Concurrency | Multiple tasks make progress during overlapping periods |
| Parallelism | Multiple tasks execute simultaneously |
| Multithreading | Using multiple threads in one process |
| Synchronization | Coordinating access/order around shared resources |

Concurrency is about dealing with multiple tasks. Parallelism is about simultaneous execution. Parallel execution is one way to achieve concurrent progress, but concurrency can exist without parallel execution.

## Q27. What problems arise when threads share mutable state?

Consider:

```kotlin
var counter = 0

val threads = List(100) {
    Thread {
        repeat(1_000) {
            counter++
        }
    }
}

threads.forEach { it.start() }
threads.forEach { it.join() }

println(counter) // May be less than 100_000
```

`counter++` is a read-modify-write operation, not one atomic operation. Threads can read the same old value and overwrite one another's updates. This is a race condition.

### Option 1: `synchronized`

```kotlin
var counter = 0
val lock = Any()

val threads = List(100) {
    Thread {
        repeat(1_000) {
            synchronized(lock) {
                counter++
            }
        }
    }
}

threads.forEach { it.start() }
threads.forEach { it.join() }

println(counter) // 100_000
```

The same lock provides mutual exclusion: only one thread at a time can enter the protected section. Monitor unlock/lock operations also establish memory-visibility guarantees between threads using that monitor.

### Option 2: `AtomicInteger`

```kotlin
val counter = AtomicInteger(0)

val threads = List(100) {
    Thread {
        repeat(1_000) {
            counter.incrementAndGet()
        }
    }
}

threads.forEach { it.start() }
threads.forEach { it.join() }

println(counter.get()) // 100_000
```

`incrementAndGet()` performs an atomic increment. It is often a good fit for a simple counter, while a lock is more flexible when several related operations must be protected together.

## Q28. Does `volatile` make `counter++` thread-safe?

No. A volatile field helps ensure reads and writes are visible across threads, but it does not make a compound read-modify-write operation atomic. Use an atomic type or synchronization when increments must not be lost.

## Q29. What is the difference between concurrency and multithreading?

Multithreading is one implementation technique for concurrent work. Concurrency can also be achieved on one thread through event-loop or cooperative scheduling. Conversely, having multiple threads does not automatically make shared-state access correct; it introduces the need to reason about races, visibility, atomicity, and synchronization.

## Important concurrency vocabulary

| Term | Explanation |
|---|---|
| Race condition | Result depends on unpredictable timing/interleaving |
| Deadlock | Threads wait indefinitely for resources held by each other |
| Starvation | A task/thread is repeatedly denied needed resources |
| Thread safety | Code remains correct when accessed concurrently |
| Atomicity | An operation behaves as one indivisible operation |
| Mutual exclusion | Only one thread enters a protected section at a time |
| Visibility | One thread's writes become observable by another |
| Thread pool | A managed set of reusable worker threads |
| Context switch | CPU switches execution from one thread/task to another |

**Interview answer:**

"A thread is an execution path. Concurrency is about multiple tasks making progress during overlapping periods, while parallelism means they literally execute at the same time. We can have concurrency on a single thread, for example with an event loop. When multiple threads share mutable state, we must handle race conditions and visibility using appropriate synchronization or atomic types."

---

# 9. Offline-first Android design with SSOT

This section captures the offline-first design discussion for a field-technician application. The example is illustrative: technicians can view assigned installations, update tasks, add notes, and work without a reliable connection.

## Q30. What does Single Source of Truth (SSOT) mean in an offline-first app?

For the Android client, the local database (for example, Room) is the canonical source that the UI reads. The Repository coordinates data sources and updates Room from the remote API. The backend remains authoritative for shared server-side data and business rules; the local database may temporarily contain a newer, unconfirmed local change or an older server snapshot.

The core rule is:

> The UI observes local persisted data. Remote responses are written into the local database; the UI does not maintain a separate competing copy of remote data.

Typical architecture:

```text
Compose UI
    ↓ observes
ViewModel / StateFlow
    ↓
Repository
    ├── Local data source: Room (client SSOT)
    ├── Remote data source: REST API
    └── Synchronization coordinator
              ↓
         WorkManager
```

## Q31. How do you return cached items immediately, then refresh from the API?

Use Room's observable query from the beginning. A Room `Flow` emits the current query result when collection starts and emits again when the observed table changes. There is no need to call `getAllOnce()` and then switch to a second observable query.

### DAO

```kotlin
@Dao
interface ItemsDao {
    @Query("SELECT * FROM items ORDER BY title")
    fun observeAll(): Flow<List<ItemEntity>>

    @Query("DELETE FROM items")
    suspend fun deleteAll()

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(items: List<ItemEntity>)
}
```

### Repository

```kotlin
class ItemsRepository(
    private val database: AppDatabase,
    private val dao: ItemsDao,
    private val api: ItemsApi
) {
    private val refreshMutex = Mutex()

    // Every consumer observes the same local source of truth.
    fun observeItems(): Flow<List<Item>> =
        dao.observeAll()
            .map { entities -> entities.map(ItemEntity::toDomain) }
            .distinctUntilChanged()

    // Fetch first; only replace the cache after a successful response.
    suspend fun refreshItems() = refreshMutex.withLock {
        val remoteItems = api.getItems()

        database.withTransaction {
            dao.deleteAll()
            dao.insertAll(remoteItems.map(ItemDto::toEntity))
        }
    }
}
```

### ViewModel: subscribe first, then refresh after the first local emission

```kotlin
class ItemsViewModel(
    private val repository: ItemsRepository
) : ViewModel() {
    private var initialRefreshTriggered = false

    private val _refreshStatus =
        MutableStateFlow<RefreshStatus>(RefreshStatus.Idle)
    val refreshStatus = _refreshStatus.asStateFlow()

    val items: StateFlow<List<Item>> =
        repository.observeItems()
            .onEach {
                if (!initialRefreshTriggered) {
                    initialRefreshTriggered = true
                    viewModelScope.launch {
                        refreshItemsSafely()
                    }
                }
            }
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.Eagerly,
                initialValue = emptyList()
            )

    private suspend fun refreshItemsSafely() {
        _refreshStatus.value = RefreshStatus.Loading
        try {
            repository.refreshItems()
            _refreshStatus.value = RefreshStatus.Success
        } catch (e: IOException) {
            _refreshStatus.value = RefreshStatus.Failed("Network unavailable")
        } catch (e: HttpException) {
            _refreshStatus.value = RefreshStatus.Failed("Server error: ${e.code()}")
        }
    }
}

sealed interface RefreshStatus {
    data object Idle : RefreshStatus
    data object Loading : RefreshStatus
    data object Success : RefreshStatus
    data class Failed(val message: String) : RefreshStatus
}
```

The important sequence is:

1. Start collecting the Room Flow.
2. Room emits the current cached list.
3. The ViewModel exposes that list.
4. Trigger the remote refresh separately, without blocking Room collection.
5. Fetch remote data.
6. Replace Room's rows in a transaction.
7. Room emits the updated list automatically.

There is no need for the Repository to manually emit the remote list. The remote response becomes visible through the same Room Flow after persistence.

**Why not put the network call in `onStart {}`?** `onStart` runs before upstream collection begins. If the network call is performed there, a slow request can delay the initial local database emission. Triggering the refresh after the first Room emission avoids that problem.

## Q32. Why use a database transaction when replacing the cache?

For a complete authoritative remote snapshot:

```kotlin
database.withTransaction {
    dao.deleteAll()
    dao.insertAll(remoteItems.map(ItemDto::toEntity))
}
```

Without a transaction, observers may see an intermediate empty table between deletion and insertion. A transaction makes the replacement atomic from the perspective of database observers; if insertion fails, the deletion is rolled back.

Fetch the remote response *before* starting the transaction. If the network request fails, do not delete the existing cache.

This full-replacement strategy is appropriate only when the API response represents the complete authoritative collection and there are no pending local edits that must be preserved. It is not suitable for a paginated response or a collection containing unsynchronized local changes.

## Q33. What if the user changes data while offline?

A local-first write should persist the user's change immediately and record what must be synchronized. A common design is a transactional outbox.

```kotlin
suspend fun completeTask(taskId: String) {
    database.withTransaction {
        taskDao.updateStatus(taskId, TaskStatus.COMPLETED)

        outboxDao.insert(
            PendingOperation(
                id = UUID.randomUUID().toString(),
                entityId = taskId,
                operationType = "COMPLETE_TASK",
                status = SyncStatus.PENDING
            )
        )
    }

    syncScheduler.enqueueSync()
}
```

The local update and outbox insertion must be in the same transaction. Otherwise, a crash between the two operations could leave a changed row without a record of the work that must be sent to the server.

The UI should distinguish states such as:

- Saved locally / pending synchronization.
- Synchronization in progress.
- Confirmed by server.
- Rejected by server and requiring user attention.

Do not tell the user that a business operation is server-confirmed merely because it was saved locally.

## Q34. How do you make synchronization reliable?

A robust synchronization mechanism typically has two directions:

- **Push:** send locally queued changes to the server.
- **Pull:** retrieve server changes made by other devices/users.

Useful design elements:

| Element | Purpose |
|---|---|
| Transactional outbox | Persist pending local operations atomically with local changes |
| Stable operation ID / idempotency key | Make retries safe when the server processed a request but the response was lost |
| Retry with backoff | Recover from transient network/server failures |
| Incremental cursor or revision | Fetch only server changes since the last successful sync |
| Server-side versioning | Detect stale edits and concurrent updates |
| Operation ordering/dependencies | Ensure parent records exist before dependent uploads |
| Recovery of in-progress work | Reclaim operations left in progress after process death |
| WorkManager | Schedule persistent work with constraints such as network availability |

Do not assume a request failed just because the client timed out. The server may have processed it. Retry with the same idempotency key and require server support for deduplication.

Do not hold a Room transaction open while making a network request. Claim the operation in a short local transaction, perform the network call outside the transaction, then persist the acknowledgement/result in another short transaction.

## Q35. How should concurrent edits be handled?

For important records, use optimistic concurrency control. For example, a technician edits server version 5 while another technician updates it to version 6. When the first technician synchronizes with `baseVersion = 5`, the server detects the stale revision and responds with a conflict rather than silently overwriting version 6.

Then the client can:

1. Fetch the latest server record.
2. Preserve the pending local operation.
3. Merge fields that are independent and safely mergeable.
4. Ask the user to resolve conflicts in important fields.

Do not use last-write-wins for every field. Notes and attachments may be appendable, while installation status, safety checks, or measurements may require explicit validation.

The server must participate in conflict detection and enforce business rules. A client-only strategy cannot guarantee consistency across devices.

## Q36. How should photos and large attachments be handled?

Do not normally store image bytes directly in Room. Store the file in app-private storage and keep metadata in Room:

```kotlin
@Entity
data class AttachmentEntity(
    @PrimaryKey val id: String,
    val installationId: String,
    val localPath: String,
    val remoteUrl: String?,
    val uploadStatus: UploadStatus,
    val checksum: String
)
```

Track attachment uploads separately from the installation record. Consider orphaned files, missing files, duplicate uploads, interrupted uploads, insufficient storage, and the case where the upload succeeds but saving its acknowledgement fails.

Saving a file and updating Room cannot be one atomic transaction across both storage systems. A temporary-file/finalization strategy and recovery cleanup can handle partial failures.

## Q37. What should be tested?

| Test case | Expected result |
|---|---|
| First Room collection with cached rows | Cached rows are emitted without waiting for the API |
| API returns a new complete snapshot | Room is replaced transactionally and Flow emits the new rows |
| API fails | Existing cache remains intact |
| API returns a legitimate empty snapshot | Cache becomes empty, if the endpoint contract defines it as a complete snapshot |
| Failure during database insertion | Transaction rolls back; old cache remains |
| Local update while offline | Local row and outbox operation commit together |
| Process dies during synchronization | Pending or expired in-progress operation is recovered |
| Server processed request but client timed out | Retry with same idempotency key does not duplicate the operation |
| Two technicians edit the same record | Version conflict is detected and handled by business rules |
| Remote refresh arrives while local changes are pending | Local pending changes are not blindly overwritten |
| Photo upload is interrupted | Attachment remains pending/retryable |

## Offline-first interview answer

"I would use a local-first architecture with Room as the Android client's source of truth. The UI observes Room through the Repository, so it can display cached data immediately and receive updates automatically. The Repository fetches remote data and persists it into Room; it doesn't send a separate remote copy directly to the UI.

For local edits, I would update the entity and insert a pending operation into an outbox in one database transaction. WorkManager can later push those operations when network constraints are met. I would use stable idempotency keys for safe retries and server revisions for optimistic concurrency. I would preserve local pending changes during pulls and use business-specific conflict resolution rather than blindly replacing everything.

I would test cache-first emission, transaction rollback, process death, retries, duplicate requests, concurrent edits, and attachment failures."

## Final SSOT reminders

- Room's observable Flow already emits the current query result; a separate `getAllOnce()` is unnecessary when the same query can be observed from the start.
- Keep observation and refresh as separate responsibilities: `observeItems()` reads Room; `refreshItems()` fetches remote data and updates Room.
- Avoid doing a slow network request in `onStart` if it delays subscription to the local Flow.
- Do not manually emit remote results when the database Flow will emit them after persistence.
- Replace the entire cache only when the response is a complete authoritative snapshot and no pending local changes will be lost.
- SSOT applies to each kind of state: Room for persisted business data, outbox for pending operations, filesystem for large attachments, secure storage for credentials, and ViewModel for transient UI state.

---

# 10. Kotlin exception handling and the Throwable hierarchy

This topic is useful because Kotlin's exception model differs from Java in one important way: Kotlin does not enforce checked exceptions at compile time.

## Q38. What is the difference between `Throwable`, `Error`, and `Exception`?

`Throwable` is the root of the throwable hierarchy. Both `Error` and `Exception` inherit from it.

```text
Throwable
├── Error
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── LinkageError
└── Exception
    ├── RuntimeException
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   └── IllegalStateException
    └── Other exceptions
        └── IOException
```

| Type | Meaning | Examples |
|---|---|---|
| `Throwable` | Root type for anything that can be thrown | Base class |
| `Error` | Serious runtime or environment problem that applications generally should not try to recover from | `OutOfMemoryError`, `StackOverflowError` |
| `Exception` | Exceptional condition an application may be able to handle | `IOException`, `RuntimeException` |
| `RuntimeException` | Unchecked exception commonly representing programming or runtime conditions | `NullPointerException`, `IllegalStateException` |

Do not confuse Kotlin's `error("message")` function with the `Error` class. The function throws an `IllegalStateException`; it does not throw a `java.lang.Error`.

## Q39. Does Kotlin have checked exceptions?

No. Kotlin does not require callers to catch or declare exceptions such as `IOException`.

```kotlin
fun readFile(): String {
    return File("data.txt").readText()
}
```

The function can throw an `IOException`, but Kotlin does not require a `try/catch` or a `throws` declaration.

For Java interoperability, Kotlin provides `@Throws`:

```kotlin
@Throws(IOException::class)
fun readFile(): String {
    return File("data.txt").readText()
}
```

This exposes the declared exception in the generated Java method signature so Java callers/compiler tooling can treat it as a checked exception.

## Q40. What is the difference between `require()`, `check()`, and `error()`?

```kotlin
fun createUser(age: Int) {
    require(age >= 18) {
        "User must be an adult"
    }
}
```

`require()` throws `IllegalArgumentException` when its condition is false. Use it to validate input or arguments.

```kotlin
fun startPayment() {
    check(isInitialized) {
        "Payment service is not initialized"
    }
}
```

`check()` throws `IllegalStateException` when its condition is false. Use it to validate the current state of an object or operation.

```kotlin
fun roleName(role: String): String = when (role) {
    "admin" -> "Administrator"
    "user" -> "Regular user"
    else -> error("Unknown role: $role")
}
```

`error()` throws `IllegalStateException` and is useful when a code path should be impossible or represents an unexpected state.

| Function | Exception | Typical purpose |
|---|---|---|
| `require(condition)` | `IllegalArgumentException` | Invalid input/argument |
| `check(condition)` | `IllegalStateException` | Invalid current state |
| `error(message)` | `IllegalStateException` | Impossible or unexpected code path |

## Q41. What is the difference between `Nothing`, `Unit`, and `Any`?

```kotlin
fun logMessage(): Unit {
    println("Hello")
}

fun fail(): Nothing {
    throw IllegalStateException("Failed")
}
```

- `Any` is the common supertype of all non-nullable Kotlin types.
- `Unit` represents a function that completes without returning a meaningful value. It has one value, `Unit`.
- `Nothing` is a type with no values. A function returning `Nothing` never completes normally, for example because it always throws or loops forever.

`Nothing` is useful in expressions such as the Elvis operator:

```kotlin
val name: String = nullableName ?: error("Name is missing")
```

The compiler knows that the right-hand side cannot complete normally, so after the Elvis expression `name` is non-null.

## Q42. What happens if `finally` throws another exception?

```kotlin
fun test() {
    try {
        throw IllegalArgumentException("First")
    } finally {
        throw IllegalStateException("Second")
    }
}
```

The exception propagated from this example is `IllegalStateException("Second")`. The exception thrown by `finally` replaces the original exception as the primary exception, so the first failure can be lost.

Avoid throwing new exceptions from `finally` when possible. If cleanup can fail, design the code to preserve the original failure and attach cleanup failures as suppressed exceptions where appropriate.

Also remember that `finally` executes whether the `try` block completes normally or exceptionally (subject to process termination or other abnormal runtime termination).

## Q43. What does `runCatching` do, and what should you be careful about?

`runCatching` executes a block and returns a `Result<T>` containing either its value or a failure.

```kotlin
val result: Result<Int> = runCatching {
    10 / 0
}

result
    .onSuccess { println("Result: $it") }
    .onFailure { println("Failed: ${it.message}") }
```

Important detail: the standard `runCatching` catches `Throwable`, not just `Exception`. Therefore, it also captures `Error` subclasses such as `OutOfMemoryError`.

Do not use `runCatching` indiscriminately around large parts of an application. Catch the failures your code can meaningfully handle, and avoid converting fatal runtime problems into ordinary application failures unintentionally.

## Q44. What is the difference between `throw` and `@Throws`?

- `throw` is a Kotlin keyword that throws an exception at runtime.
- `@Throws` is an annotation primarily used to declare exceptions in generated JVM method signatures for Java interoperability.

```kotlin
throw IllegalArgumentException("Invalid input")
```

```kotlin
@Throws(IOException::class)
fun loadData(): String {
    return File("data.txt").readText()
}
```

## Q45. What are exception causes and suppressed exceptions?

A cause records the underlying exception that explains why another exception was created:

```kotlin
try {
    readRemoteFile()
} catch (e: IOException) {
    throw IllegalStateException("Could not load configuration", e)
}
```

The `IllegalStateException` has the original `IOException` as its `cause`.

Suppressed exceptions record additional failures that occurred while handling a primary failure, commonly during resource cleanup. On the JVM, `Throwable` exposes them through `addSuppressed()` and `suppressed`.

**Interview answer:**

"`Throwable` is the root of the hierarchy. `Error` generally represents serious runtime problems, while `Exception` represents conditions application code may be able to handle. Kotlin does not enforce checked exceptions, unlike Java. I use `require` for argument validation, `check` for state validation, and `error` for impossible states. A useful Kotlin-specific detail is that `error()` returns `Nothing` and throws `IllegalStateException`. I also remember that `runCatching` catches `Throwable`, so it should be used thoughtfully."

---

