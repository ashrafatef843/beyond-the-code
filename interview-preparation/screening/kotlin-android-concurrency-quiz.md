# Kotlin & Android Concurrency Quiz Questions

## Question 1: Singleton Context Leak

```kotlin
class AdManager private constructor(private val context: Context) {
    companion object {
        private var instance: AdManager? = null

        fun getInstance(context: Context): AdManager {
            if (instance == null) instance = AdManager(context)
            return instance!!
        }
    }
}
```

### Options:
- **A.** Nothing — this is a standard singleton.
- **B.** `instance!!` can throw a null-pointer exception.
- **C.** It can hold an Activity context for the app's lifetime. *(Correct)*
- **D.** Companion objects can't hold mutable references.

### Explanation:
The `instance` property lives in the static companion object scope for the lifetime of the application process. Passing an Activity context into `getInstance()` stores it inside `private val context: Context`, preventing the Activity from being garbage-collected when destroyed and creating a memory leak.

---

## Question 2: Handler Memory Leak

What's wrong here?

```kotlin
class AdActivity : AppCompatActivity() {
    private val handler = Handler(Looper.getMainLooper())

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        handler.postDelayed({ showAd() }, 60_000)
    }
}
```

### Options:
- **A.** The pending Runnable keeps the Activity alive. *(Correct)*
- **B.** Handler can't be built in a field initializer.
- **C.** `postDelayed` runs the block on a background thread.
- **D.** Nothing — the Runnable is cancelled automatically.

### Explanation:
The lambda `{ showAd() }` passed into `postDelayed` captures an implicit reference to the enclosing `AdActivity` instance. The delayed `Runnable` remains in the main thread's `MessageQueue` for 60 seconds. If `AdActivity` is destroyed during this window, the pending message prevents it from being garbage-collected.

---

## Question 3: Double-Checked Locking

What's wrong with this double-checked lock?

```kotlin
object ConfigCache {
    private var config: Config? = null

    fun get(): Config {
        if (config == null) {
            synchronized(this) {
                if (config == null) config = load()
            }
        }
        return config!!
    }
}
```

### Options:
- **A.** It's correct as written.
- **B.** `synchronized(this)` isn't allowed inside an object.
- **C.** It can deadlock under contention.
- **D.** Config needs `@Volatile` for the unsynchronized read. *(Correct)*

### Explanation:
Double-Checked Locking relies on an unsynchronized read check (`if (config == null)`). Without marking `config` with `@Volatile`, memory reordering by the compiler or CPU can cause a secondary thread to read a non-null reference before its fields are fully initialized, leading to instability or crashes.

---

## Question 4: Coroutine Exception Propagation

What happens here?

```kotlin
try {
    scope.launch {
        throw IOException("failed")
    }
} catch (e: IOException) {
    log(e)
}
```

### Options:
- **A.** The catch block logs the exception.
- **B.** The catch never runs; the exception propagates through the scope. *(Correct)*
- **C.** It won't compile — `launch` can't throw a checked exception.
- **D.** The exception is discarded with no further effect.

### Explanation:
`scope.launch` starts a coroutine asynchronously and immediately returns a `Job`. The surrounding synchronous `try-catch` block completes long before the coroutine body executes. When the exception is thrown, it propagates up the `CoroutineScope` hierarchy rather than being caught by the synchronous `try-catch`.

---

## Question 5: Uncooperative Cancellation

`cancel()` is called. What happens?

```kotlin
val job = scope.launch(Dispatchers.Default) {
    while (true) {
        computeFrame() // pure CPU work, never suspends
    }
}
delay(100)
job.cancel()
```

### Options:
- **A.** The loop stops at the next iteration.
- **B.** `computeFrame()` throws `CancellationException`.
- **C.** The dispatcher's thread is interrupted.
- **D.** The loop keeps running; cancellation needs cooperation. *(Correct)*

### Explanation:
Coroutine cancellation is cooperative. The loop performs continuous CPU work without calling suspending functions (e.g., `yield()`) or explicitly checking `isActive`. Therefore, calling `job.cancel()` sets the job state to canceled, but the executing loop never checks this state and continues running indefinitely.

---

## Question 6: Executor Exception Handling

`send()` throws. What happens to the exception?

```kotlin
private val executor = Executors.newSingleThreadExecutor()

fun track(event: Event) {
    executor.submit { send(event) }
}
```

### Options:
- **A.** It's captured in the Future and silently lost. *(Correct)*
- **B.** It crashes the process via the default handler.
- **C.** It's rethrown on the calling thread.
- **D.** It shuts the executor down permanently.

### Explanation:
When executing tasks via `ExecutorService.submit()`, uncaught exceptions are caught internally by the returned `FutureTask`. The exception is stored within the `Future` and is only rethrown when calling `.get()`. Because the returned `Future` is ignored in this code, the exception is silently swallowed.

---

## Question 7: StateFlow Conflation

A coroutine collects a `MutableStateFlow` whose value is updated rapidly from another thread. What is guaranteed?

### Options:
- **A.** Every value is delivered, in order.
- **B.** Values are buffered until the collector catches up.
- **C.** Nothing is delivered until the flow completes.
- **D.** Only the latest value; intermediate updates can be skipped. *(Correct)*

### Explanation:
`StateFlow` is conflated by design, meaning it only holds the single most recent state. When values are updated faster than the collector can process them, intermediate updates are dropped so that the collector always receives the newest available state.
