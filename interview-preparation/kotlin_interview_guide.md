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
