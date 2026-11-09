# Jetpack Compose: The Complete Guide to Composition, Recomposition, State, and Side Effects

**Level:** Beginner to Advanced  
**Reading time:** Approximately 45 minutes  
**Technology:** Kotlin, Android Jetpack Compose, Compose Runtime, Material 3

## Learning objectives

By the end of this tutorial, you should be able to:

1. Explain how Jetpack Compose works.
2. Understand what Composition and Recomposition mean internally.
3. Explain how Compose observes state changes and updates the UI.
4. Understand Kotlin delegated properties and the `by` keyword.
5. Use `remember`, `rememberSaveable`, and observable collections correctly.
6. Explain composable identity, lifecycle, and stable keys.
7. Apply State Hoisting and Unidirectional Data Flow.
8. Understand and choose between all major Compose effect APIs.
9. Integrate Compose with Kotlin Coroutines and Flow.
10. Understand stability, Strong Skipping, and performance optimization.
11. Design, test, and debug real Compose applications.

---

# Table of contents

| Section | Subject | Time |
|---|---|---:|
| 1 | Introduction to Jetpack Compose | 3 min |
| 2 | Composable functions and UI building | 3 min |
| 3 | Composition and Recomposition | 10 min |
| 4 | Kotlin delegation and Compose State | 5 min |
| 5 | Remember, lifecycle, identity, and keys | 4 min |
| 6 | State Hoisting, ViewModel, and UDF | 3 min |
| 7 | Side Effects and Effects APIs | 8 min |
| 8 | derivedStateOf and state observation | 3 min |
| 9 | Stability and performance optimization | 3 min |
| 10 | Architecture, animations, accessibility, and testing | 2 min |
| 11 | Practical exercises and interview questions | Optional review |

# Part 1: Introduction to Jetpack Compose

## 1.1 What is Jetpack Compose?

Jetpack Compose is Android's modern declarative UI toolkit.

Traditionally, Android applications used XML layouts and the View system.

For example, in a traditional Android application:

```kotlin
val textView = findViewById<TextView>(R.id.myTextView)

textView.text = "Hello Alex"

textView.visibility = View.VISIBLE
```

This is an **imperative approach**.

You explicitly tell Android which UI element to find and how to change it.

Jetpack Compose follows a declarative approach.

Instead of manually changing UI elements, you describe what the UI should look like for the current state.

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name")
}
```

When the `name` value changes, Compose can update the displayed text automatically.

### Imperative vs declarative

| Imperative UI | Declarative UI |
|---|---|
| Explicitly manipulates UI elements | Describes the desired UI |
| Commonly uses XML and Views | Uses composable functions |
| You manually update View properties | Compose updates UI from state |
| Focuses on how to change the UI | Focuses on what the UI should display |

The central idea:

**The UI is a function of the current application state.**

Conceptually:

```text
       Current State
             |
             v
      Composable Functions
             |
             v
         Composition
             |
             v
           Layout
             |
             v
          Drawing
             |
             v
           Screen
```

When state changes, Compose updates the affected parts of the UI.

## 1.2 How does Compose start?

A typical Android Activity creates Compose content using `setContent`.

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            MyAppTheme {
                MyScreen()
            }
        }
    }
}
```

The `setContent` function establishes the Compose UI hosted by the Activity.

When the composable content is initially evaluated, Compose performs the initial Composition.

A typical project uses:

- Kotlin.
- The Compose compiler plugin.
- Compose UI.
- Material 3.
- An Activity that hosts Compose content.

For dependency management, the Compose BOM helps coordinate compatible Compose library versions.

# Part 2: Composable Functions and UI Building

## 2.1 What is a composable function?

A composable function is a Kotlin function annotated with `@Composable`.

```kotlin
@Composable
fun WelcomeScreen() {
    Text("Welcome to Jetpack Compose")
}
```

Composable functions describe UI elements and can call other composable functions.

```kotlin
@Composable
fun ProfileScreen() {

    Column {

        Text("My Profile")

        Text("Alex")

        Button(onClick = {}) {
            Text("Edit Profile")
        }
    }
}
```

Think of composables as reusable building blocks.

## 2.2 Important built-in composables

| Composable | Responsibility |
|---|---|
| `Text` | Displays text |
| `Button` | Displays an interactive button |
| `Column` | Arranges children vertically |
| `Row` | Arranges children horizontally |
| `Box` | Layers or positions children |
| `Image` | Displays an image |
| `Icon` | Displays an icon |
| `TextField` | Accepts text input |
| `LazyColumn` | Displays a vertically scrolling list |
| `LazyRow` | Displays a horizontally scrolling list |

## 2.3 Modifiers

A Modifier changes the appearance, layout, interaction, or behavior of a composable.

```kotlin
Text(
    text = "Welcome",
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
        .background(Color.LightGray)
)
```

Common modifiers include:

- `padding()` – Adds space.
- `fillMaxWidth()` – Fills the available width.
- `fillMaxSize()` – Fills the available size.
- `size()` – Sets dimensions.
- `background()` – Adds a background.
- `clickable()` – Adds click interaction.
- `border()` – Adds a border.
- `offset()` – Changes placement.

### Modifier order matters

Compare:

```kotlin
Modifier
    .background(Color.Blue)
    .padding(16.dp)
```

And:

```kotlin
Modifier
    .padding(16.dp)
    .background(Color.Blue)
```

These can produce different results.

Modifier elements wrap each other in order. In the first example, the background includes the padded area. In the second, the padding is outside the background.

### Best practice for reusable components

Allow callers to provide a Modifier:

```kotlin
@Composable
fun WelcomeText(
    modifier: Modifier = Modifier
) {
    Text(
        text = "Welcome",
        modifier = modifier
    )
}
```

This makes components more flexible.

---

# Part 3: Composition and Recomposition

**This is the most important section of the tutorial.**

## 3.1 What is Composition?

Composition is the process through which Compose executes composable functions to produce a description of the UI.

The resulting Composition is a tree that represents the composable UI structure.

Consider:

```kotlin
@Composable
fun MyScreen() {

    Column {

        Text("Hello")

        Button(onClick = {}) {
            Text("Click")
        }
    }
}
```

Conceptually, the Composition looks like this:

```text
MyScreen
   |
   └── Column
         |
         ├── Text("Hello")
         |
         └── Button
               |
               └── Text("Click")
```

This is a conceptual representation of the UI hierarchy.

**Important:** Composition is not the same thing as drawing pixels on the screen.

It describes the UI that should exist.

## 3.2 What is Initial Composition?

Initial Composition is the first time Compose executes the composables to create the Composition.

For example:

```kotlin
setContent {
    MyScreen()
}
```

When `MyScreen()` is first evaluated, Compose creates the initial Composition.

Afterward, the Composition is updated through Recomposition.

Remember:

- Initial Composition creates the first Composition.
- Recomposition updates an existing Composition.

## 3.3 What is Recomposition?

Recomposition is the process of re-executing composable functions that may have changed in response to state or input changes.

Consider:

```kotlin
@Composable
fun Counter() {

    var count by remember {
        mutableIntStateOf(0)
    }

    Column {

        Text("Count: $count")

        Button(
            onClick = {
                count++
            }
        ) {
            Text("Increase")
        }
    }
}
```

Initially:

```text
count = 0

Screen:
Count: 0
[Increase]
```

The user clicks the button.

The following code executes:

```kotlin
count++
```

Now:

```text
count = 1
```

Compose observes the state change and schedules Recomposition for the affected composable scopes.

The updated UI becomes:

```text
Count: 1
[Increase]
```

This is the basic mechanism of Recomposition.

## 3.4 What triggers Recomposition?

The most common trigger is a change to observable Compose state that was previously read during composition.

For example:

```kotlin
var name by remember {
    mutableStateOf("John")
}
```

When the value changes:

```kotlin
name = "Alex"
```

Compose can identify the composable scopes that read this state and schedule their recomposition.

Another trigger is a parent composable running again and supplying different inputs to its children.

For example:

```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

The parent initially calls:

```kotlin
Greeting("John")
```

Later it calls:

```kotlin
Greeting("Alex")
```

The changed input can cause the affected content to update.

### Important distinction

Changing an ordinary Kotlin variable does not automatically trigger Recomposition.

```kotlin
var count = 0

count++
```

This is not observable Compose state.

For a value that must automatically update the UI, use an observable state holder or another supported observable source.

## 3.5 How does Compose know what needs to be recomposed?

This is one of the most important concepts in Compose.

**Compose tracks where observable state is read.**

Consider:

```kotlin
@Composable
fun MyScreen() {

    var count by remember {
        mutableIntStateOf(0)
    }

    Column {

        Text("Welcome")

        Text("Count: $count")

        Button(
            onClick = {
                count++
            }
        ) {
            Text("Increase")
        }
    }
}
```

The composable reads `count` here:

```kotlin
Text("Count: $count")
```

That read establishes a dependency on the observable state.

When `count` changes, the relevant scope can be invalidated.

The Compose runtime maintains information about the Composition and the state dependencies of composable scopes.

Conceptually:

```text
                MyScreen
                   |
                   v
                 Column
                   |
          +--------+---------+
          |        |         |
          v        v         v
        Text     Text      Button
      Welcome   Count      Increase
                 |
                 |
             reads count
                 |
                 v
           Observable state
```

When count changes, Compose knows that the scope that read it needs reconsideration.

This does not mean that every composable in the application must execute again.

Compose can skip eligible composables whose inputs have not changed.

### Important terminology

| Term | Meaning |
|---|---|
| State read | Creates an observable dependency |
| State write | Can invalidate scopes that read the state |
| Invalidation | Marks a scope as needing reconsideration |
| Recomposition | Re-executes affected composable code |
| Skipping | Avoids executing an eligible composable |
| Apply changes | Applies successful recomposition changes to the Composition |

## 3.6 What is the Recomposer?

The Recomposer is a runtime component responsible for coordinating recomposition.

Conceptually, the process is:

1. A composable reads observable state.
2. The state changes.
3. Affected composable scopes are invalidated.
4. The Recomposer schedules recomposition.
5. Affected composables are evaluated again.
6. Compose applies the changes to the Composition.

You normally do not need to interact with the Recomposer directly.

However, understanding its role explains how a state assignment eventually leads to an updated screen.

## 3.7 Composition, Recomposition, Layout, and Drawing

Compose updates UI through three main phases:

### Phase 1: Composition

Composition answers:

**What UI should exist?**

Compose evaluates composable functions and produces a description of the UI.

Example:

```kotlin
if (isLoggedIn) {
    HomeScreen()
} else {
    LoginScreen()
}
```

When `isLoggedIn` changes, the structure of the Composition can change.

### Phase 2: Layout

Layout answers:

**Where should the elements appear, and how much space should they occupy?**

Layout consists of two main steps:

1. Measurement – Determines the sizes of elements.
2. Placement – Determines where elements appear.

For example:

```kotlin
Column {
    Text("Title")
    Text("Description")
}
```

Compose measures and places both text elements according to available constraints.

### Phase 3: Drawing

Drawing answers:

**How should the elements be rendered?**

Compose draws text, images, shapes, backgrounds, and other visual elements.

### Complete flow

```text
             State changes
                   |
                   v
              Composition
               What to show?
                   |
                   v
                 Layout
               Where to put it?
                   |
                   v
                Drawing
              How to render it?
                   |
                   v
                 Screen
```

**Important:** Not every state change requires all three phases to run.

Compose can skip phases that do not need updating.

For example:

- A state read during composition may invalidate composition.
- A state read during layout placement can trigger layout work without requiring composition for that read.
- A state read inside a drawing lambda can trigger drawing work without requiring composition or layout for that read.

This becomes important when optimizing scrolling and animations.

## 3.8 Recomposition does not mean redrawing everything

A common beginner misunderstanding is:

"Whenever state changes, Compose redraws the entire screen."

That is not the correct mental model.

Compose tracks state dependencies and maintains the Composition structure.

When a state change happens, affected scopes are invalidated.

Eligible composables can be skipped when their inputs have not changed and the applicable runtime conditions allow it.

For example:

```kotlin
@Composable
fun Dashboard() {

    Column {

        Header()

        Counter()

        Footer()
    }
}
```

Imagine the counter owns its own state.

When that state changes, the counter's scope can be recomposed without necessarily executing the Header and Footer composables again.

The exact recomposition scope depends on where state is read and the structure of the composable calls.

## 3.9 Recomposition is optimistic

This is an important advanced concept.

Compose can start recomposition assuming that the information it is using will remain valid while the work is performed.

However, state can change again before the recomposition has finished.

Compose can cancel the current recomposition and restart with the newer information.

This means:

- A composable can execute more than once.
- A composable can be recomposed without a visible UI change.
- A recomposition can be cancelled.
- A composable may be skipped.

Therefore, you should never depend on a composable executing exactly once or in a particular order.

### Important programming rule

Composable functions should be:

- Fast.
- Predictable.
- Side-effect free.
- Idempotent.

For example, this is appropriate:

```kotlin
@Composable
fun UserName(name: String) {
    Text(name)
}
```

This is problematic:

```kotlin
@Composable
fun UserName(name: String) {

    sharedUserName = name

    Text(name)
}
```

Why?

Because Compose can re-execute the function, cancel its recomposition, or skip it.

Mutating external application data directly inside the composable body can produce unexpected behavior.

Use event callbacks and appropriate effect APIs instead.

### Section 3 summary

Composition creates the first UI description.

Recomposition updates the existing Composition when relevant inputs or observed state change.

The runtime tracks state dependencies, invalidates the necessary scopes, and can skip work that does not need to be repeated.

Layout and drawing are separate phases that Compose can also optimize independently.

**Do not try to eliminate Recomposition. Understand what triggers it and make the work performed during it efficient.**

---

# Part 4: Kotlin Delegated Properties and Compose State

This section combines Kotlin's `by` keyword, `MutableState`, `remember`, and observable collections.

## 4.1 What is State?

State is information that can change over time and influence the UI.

Examples:

- Username.
- Password.
- Selected tab.
- Shopping cart count.
- Loading status.
- Whether a dialog is visible.
- A list of contacts.

In Compose, a common way to represent observable state is:

```kotlin
var count by remember {
    mutableIntStateOf(0)
}
```

The state holder is created with `mutableIntStateOf()`.

`remember` retains it across recompositions.

The `var` allows the value to be updated.

The `by` keyword provides convenient property access.

## 4.2 What does `by` mean in Kotlin?

The `by` keyword provides **delegated property access**.

It means that another object handles the property's getter and setter operations.

### Without `by`

```kotlin
val nameState = remember {
    mutableStateOf("John")
}

// Read
println(nameState.value)

// Update
nameState.value = "Mike"
```

Here, you explicitly use the `.value` property.

### With `by`

```kotlin
var name by remember {
    mutableStateOf("John")
}

// Read
println(name)

// Update
name = "Mike"
```

Both examples use the same underlying state.

The `by` keyword lets Kotlin delegate property operations to the `MutableState` object through operator functions.

The important functions are:

- `getValue()` – Used when reading the property.
- `setValue()` – Used when assigning a new value.

Compose provides the required operator support.

### Interview takeaway

`by` does not create observable state by itself.

`mutableStateOf()` creates the observable state.

`by` only provides convenient delegated access to that state.

## 4.3 mutableStateOf and mutableIntStateOf

Examples:

```kotlin
var name by remember {
    mutableStateOf("John")
}
```

For an integer:

```kotlin
var count by remember {
    mutableIntStateOf(0)
}
```

Other specialized state holders include:

```kotlin
mutableLongStateOf(0L)
mutableFloatStateOf(0f)
mutableDoubleStateOf(0.0)
```

The specialized primitive state APIs can avoid boxing in common cases.

## 4.4 Normal Kotlin variables versus Compose State

Compare:

```kotlin
var count = 0
```

With:

```kotlin
var count by remember {
    mutableIntStateOf(0)
}
```

The first is an ordinary Kotlin variable.

The second uses observable Compose state and retains the state object across recompositions.

When the state value changes, Compose can schedule recomposition for scopes that read it.

## 4.5 remember

`remember` stores an object in the Composition.

```kotlin
@Composable
fun Counter() {

    var count by remember {
        mutableIntStateOf(0)
    }

    Button(
        onClick = {
            count++
        }
    ) {
        Text("Count: $count")
    }
}
```

Why do we need `remember`?

If a new state object were created on every recomposition, the counter could return to its initial value.

`remember` allows Compose to retrieve the stored object again at the same Composition location.

Remember these rules:

- It survives recomposition while its call remains in the Composition.
- It is associated with a particular composable location.
- It is forgotten when its owning call leaves the Composition.
- It does not automatically survive Activity recreation or process death.
- It can use keys to determine when its stored value should be recalculated.

## 4.6 remember with keys

Consider:

```kotlin
val formattedName = remember(name) {
    name.uppercase()
}
```

The calculation is remembered while `name` remains equal to its previous key.

When `name` changes, Compose recalculates the remembered value.

This is useful for caching calculations that depend on inputs.

## 4.7 rememberSaveable

What happens if an Activity is recreated, for example because of a configuration change?

`remember` alone does not guarantee restoration.

Use:

```kotlin
var name by rememberSaveable {
    mutableStateOf("")
}
```

`rememberSaveable` can preserve supported values through Android's saved-state mechanism.

| API | Across recomposition | Configuration recreation | Supported saved-state restoration |
|---|---|---|---|
| `remember` | Yes | No guarantee | No |
| `rememberSaveable` | Yes | Yes | Yes |
| ViewModel | Yes, when observed | Yes | Not by itself |
| Database or persistent storage | Yes, when observed | Yes | Yes |

`rememberSaveable` is suitable for small pieces of restorable UI state.

For unsupported custom objects, you may need a Saver.

For application data, use an appropriate state holder or persistent storage rather than trying to put everything into saved UI state.

## 4.8 Normal MutableList versus mutableStateListOf

This is a particularly important Compose topic.

A normal Kotlin mutable list is not automatically observable.

Consider:

```kotlin
@Composable
fun NamesScreen() {

    val names = remember {
        mutableListOf("John")
    }

    Column {

        Text("Number of names: ${names.size}")

        Button(
            onClick = {
                names.add("Alex")
            }
        ) {
            Text("Add name")
        }

        names.forEach { name ->
            Text(name)
        }
    }
}
```

Initially:

```text
["John"]
```

After clicking:

```kotlin
names.add("Alex")
```

The list becomes:

```text
["John", "Alex"]
```

But Compose is not automatically notified about this mutation.

Why?

Because `remember` preserves the list instance. It does not observe mutations inside an ordinary Kotlin mutable list.

### Solution 1: mutableStateListOf

```kotlin
@Composable
fun NamesScreen() {

    val names = remember {
        mutableStateListOf("John")
    }

    Column {

        Text("Number of names: ${names.size}")

        Button(
            onClick = {
                names.add("Alex")
            }
        ) {
            Text("Add name")
        }

        names.forEach { name ->
            Text(name)
        }
    }
}
```

Now collection operations are observable:

```kotlin
names.add("Alex")
names.remove("John")
names.clear()
names[0] = "Mike"
```

Changes to the observable collection can invalidate scopes that read its contents.

### Solution 2: Immutable list with mutableStateOf

```kotlin
@Composable
fun NamesScreen() {

    var names by remember {
        mutableStateOf(listOf("John"))
    }

    Button(
        onClick = {
            names = names + "Alex"
        }
    ) {
        Text("Add name")
    }

    Column {
        names.forEach { name ->
            Text(name)
        }
    }
}
```

The important point is that a new list is assigned to the observable state.

### What about mutableStateOf with a mutable list?

This is not sufficient:

```kotlin
var names by remember {
    mutableStateOf(mutableListOf("John"))
}

names.add("Alex")
```

Why?

Because the value of the state property was not reassigned.

You modified the contents of the same list instance.

### Comparison

| Approach | Collection changes automatically observed? |
|---|---|
| `mutableListOf()` | No |
| `remember { mutableListOf() }` | No |
| `mutableStateListOf()` | Yes |
| `mutableStateOf(listOf())` with reassignment | Yes |
| `mutableStateOf(mutableListOf())` with in-place mutation | No |

**Additional detail:** An observable list tracks collection changes. It does not automatically observe ordinary property mutations inside objects stored in the collection.

For example:

```kotlin
data class User(
    var name: String
)
```

Changing `user.name` does not become observable just because the User object is stored inside a `mutableStateListOf`.

The object's changing property must itself be observable, or you should replace the item with a new immutable object.

---

# Part 5: Composable Lifecycle, Identity, and Keys

## 5.1 Composable lifecycle

A composable generally has three conceptual stages:

1. It enters the Composition.
2. It can be recomposed while it remains in the Composition.
3. It leaves the Composition when it is no longer part of the UI structure.

Consider:

```kotlin
@Composable
fun Parent() {

    var showCounter by remember {
        mutableStateOf(true)
    }

    if (showCounter) {
        Counter()
    }

    Button(
        onClick = {
            showCounter = !showCounter
        }
    ) {
        Text("Toggle Counter")
    }
}
```

Suppose the counter reaches 5.

Then:

1. The user hides the counter.
2. `Counter()` leaves the Composition.
3. Remembered state owned by that counter is forgotten.
4. The user displays it again.
5. The new counter starts at 0.

Why?

Because the composable that owned the remembered state left the Composition.

### Important distinction

| Situation | What happens? |
|---|---|
| Composable recomposes | Remembered values are retained |
| A nested element disappears, but its state owner remains | Parent-owned state can remain |
| Composable owning the remember call leaves | Its remembered value is forgotten |
| Composable enters again | New remembered state is created |

## 5.2 How does Compose identify composables?

Compose tracks composable instances using their position in the Composition and the structure of their calls.

Conceptually, the call site and execution order help identify a composable group.

This matters because Compose needs to associate remembered values with the correct composable instance.

Suppose:

```kotlin
Column {
    Text("A")
    Text("B")
}
```

The two Text calls have different positions and therefore different identities.

However, when dynamic data is involved, positions can change.

## 5.3 The key composable

Consider:

```kotlin
Column {

    contacts.forEach { contact ->

        key(contact.id) {
            ContactRow(contact)
        }
    }
}
```

The `key()` composable gives the content a stable identity based on `contact.id`.

This helps Compose preserve the association between a composable instance and its remembered state when list items move.

## 5.4 Keys in LazyColumn

A LazyColumn displays large or scrolling lists efficiently.

```kotlin
LazyColumn {

    items(
        items = contacts,
        key = { contact ->
            contact.id
        }
    ) { contact ->

        ContactRow(contact)
    }
}
```

Why use stable keys?

- They identify individual items.
- They help preserve remembered item state.
- They improve behavior when items are inserted or removed.
- They help Compose keep item identity consistent during reordering.

Keys must be stable and unique among the relevant items.

You can also provide `contentType` when different item types share a lazy list, helping Compose reuse compatible item compositions.

### Position versus identity

Imagine:

```text
Position 0: Apple
Position 1: Banana
Position 2: Orange
```

Suppose Banana is expanded.

Now Apple is removed:

```text
Position 0: Banana
Position 1: Orange
```

Without stable identity, remembered state associated with a position can become associated with the wrong item.

Stable keys tell Compose:

"Banana is the same item even though its position changed."

## 5.5 Do not confuse different types of keys

There are several key mechanisms.

| API | Purpose |
|---|---|
| `remember(name)` | Recalculates a remembered value when the key changes |
| `key(id) { ... }` | Gives a group of composables a specific identity |
| `items(key = { ... })` | Gives a lazy list item a stable identity |
| `LaunchedEffect(userId)` | Cancels and restarts an effect when its key changes |

They have related terminology but different responsibilities.

---

# Part 6: State Hoisting, ViewModel, and Unidirectional Data Flow

## 6.1 What is State Hoisting?

State Hoisting is the practice of moving state ownership from a child composable to its parent.

Consider a stateful Counter:

```kotlin
@Composable
fun Counter() {

    var count by remember {
        mutableIntStateOf(0)
    }

    Button(
        onClick = {
            count++
        }
    ) {
        Text("Count: $count")
    }
}
```

The Counter owns its state.

That can be appropriate for self-contained UI.

However, if another composable must control the counter value, it is often better to move ownership to the parent.

### Hoisted version

```kotlin
@Composable
fun Counter(
    count: Int,
    onIncrement: () -> Unit
) {
    Button(
        onClick = onIncrement
    ) {
        Text("Count: $count")
    }
}
```

Now the parent owns the state:

```kotlin
@Composable
fun CounterScreen() {

    var count by rememberSaveable {
        mutableIntStateOf(0)
    }

    Counter(
        count = count,
        onIncrement = {
            count++
        }
    )
}
```

The child receives data and emits events.

The parent decides what should happen.

## 6.2 Unidirectional Data Flow

This is called Unidirectional Data Flow (UDF).

```text
             Parent
           Owns State
                |
                |
          State flows down
                |
                v
              Child
          Displays the UI
                |
                |
          Events flow up
                |
                v
              Parent
          Updates the state
```

Benefits:

- Clear ownership of state.
- Easier testing.
- Easier debugging.
- Fewer inconsistent values.
- More reusable composables.

## 6.3 Where should state live?

Useful rules:

1. Hoist state to at least the lowest common parent of composables that read it.
2. Hoist state to at least the highest level where it needs to be changed.
3. Keep related state together when it changes in response to the same events.

Do not hoist every small UI value to the Activity or ViewModel.

For example:

- A dropdown's temporary expanded state can remain inside the dropdown.
- A selected product affecting the whole shopping screen should be owned at a higher level.
- Screen-level business data generally belongs in a ViewModel or another state holder.

## 6.4 ViewModel and Compose State

A real application often uses a ViewModel to manage screen-level state.

Define a UI state model:

```kotlin
data class ProfileUiState(
    val name: String = "",
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)
```

ViewModel:

```kotlin
class ProfileViewModel : ViewModel() {

    private val _uiState =
        MutableStateFlow(ProfileUiState())

    val uiState: StateFlow<ProfileUiState> =
        _uiState.asStateFlow()
}
```

Composable:

```kotlin
@Composable
fun ProfileRoute(
    viewModel: ProfileViewModel
) {

    val uiState by viewModel.uiState
        .collectAsStateWithLifecycle()

    ProfileScreen(
        state = uiState
    )
}
```

Use `collectAsStateWithLifecycle()` for lifecycle-aware collection of Flow in Android UI.

The ViewModel owns screen-related state and business logic.

The composable receives the state and displays it.

---

# Part 7: Side Effects and Effects APIs

**This is the second most important section after Composition and Recomposition.**

## 7.1 What is a Side Effect?

A side effect is an operation that changes something outside the normal UI description.

Examples include:

- Calling a network API.
- Registering an external listener.
- Showing a Snackbar.
- Navigating to another screen.
- Sending an analytics event.
- Starting a coroutine.
- Updating an external object.

Why are side effects important in Compose?

Because composables can execute repeatedly, be skipped, or have recomposition cancelled.

You should not assume that the body of a composable runs only once.

Therefore:

**Keep composable bodies free of uncontrolled side effects.**

Compose provides several effect APIs to handle these operations safely.

## 7.2 Effects API overview

| API | Main responsibility |
|---|---|
| `LaunchedEffect` | Runs a coroutine tied to Composition and its keys |
| `rememberCoroutineScope` | Provides a Composition-aware scope for launching coroutines from events |
| `DisposableEffect` | Performs setup and cleanup for resources |
| `SideEffect` | Publishes information after successful composition |
| `rememberUpdatedState` | Provides the latest value without restarting an ongoing effect |
| `produceState` | Converts external or asynchronous data into Compose State |
| `snapshotFlow` | Converts Compose state reads into a Flow |
| `derivedStateOf` | Caches a derived value to reduce unnecessary downstream updates |

Each API solves a different problem.

---

## 7.3 LaunchedEffect

`LaunchedEffect` launches a coroutine tied to a composable's presence in the Composition.

Example:

```kotlin
@Composable
fun UserScreen(userId: String) {

    LaunchedEffect(userId) {

        val user = fetchUser(userId)

        // Update or display the result through state.
    }

    Text("User ID: $userId")
}
```

### How does LaunchedEffect work?

When the composable enters the Composition:

1. The coroutine starts.
2. The suspend operation executes.
3. The coroutine continues until it completes or is cancelled.
4. If the key changes, the previous coroutine is cancelled.
5. A new coroutine starts with the new key.
6. If the composable leaves the Composition, the coroutine is cancelled.

### What are effect keys?

Consider:

```kotlin
LaunchedEffect(userId) {
    fetchUser(userId)
}
```

The key is `userId`.

When it changes, the effect restarts.

This is important because the effect depends on the user ID.

### What about LaunchedEffect(Unit)?

```kotlin
LaunchedEffect(Unit) {
    delay(5000)
    println("Finished")
}
```

`Unit` remains equal to itself.

Therefore, ordinary recomposition does not restart the coroutine.

However, the effect is still tied to the composable's lifecycle.

It is cancelled when the call leaves the Composition and starts again if it re-enters.

### Incorrect implementation

```kotlin
@Composable
fun UserScreen(userId: String) {

    LaunchedEffect(Unit) {
        fetchUser(userId)
    }

    Text(userId)
}
```

Suppose the screen receives user ID 101 and then receives 202.

Because the effect key remains `Unit`, the coroutine is not restarted just because `userId` changed.

The effect could continue working with the old ID.

### Correct implementation

```kotlin
@Composable
fun UserScreen(userId: String) {

    LaunchedEffect(userId) {
        fetchUser(userId)
    }

    Text(userId)
}
```

Now the effect restarts when the ID changes.

### When to use LaunchedEffect

Use it for suspend work tied to a composable and its keys, such as:

- Loading data when a key changes.
- Running a timed effect.
- Collecting an event stream while the composable is present.
- Performing coroutine-based work tied to a screen or UI element.

For production API requests, handle loading, errors, cancellation, and stale data explicitly. Screen-level business operations are often better owned by a ViewModel.

---

## 7.4 rememberCoroutineScope

`rememberCoroutineScope()` returns a CoroutineScope tied to the composable's lifecycle.

It is particularly useful when you want to launch a coroutine in response to a user event.

Example:

```kotlin
@Composable
fun MessageButton(
    snackbarHostState: SnackbarHostState
) {

    val scope = rememberCoroutineScope()

    Button(
        onClick = {

            scope.launch {

                snackbarHostState.showSnackbar(
                    "Hello!"
                )
            }
        }
    ) {
        Text("Show message")
    }
}
```

### How does it work?

- The scope is remembered at the composable location.
- The button click launches a coroutine.
- The coroutine performs suspend work.
- The scope is cancelled when the owning composable leaves the Composition.

### LaunchedEffect versus rememberCoroutineScope

| LaunchedEffect | rememberCoroutineScope |
|---|---|
| Starts automatically when the effect enters Composition | Returns a scope for later use |
| Restarts when keys change | Does not automatically start a coroutine |
| Good for composition-driven work | Good for event-driven work |
| Coroutine cancelled when effect is disposed | Launched jobs cancelled when scope is disposed |

Use `LaunchedEffect` for work that should start because the composable enters or a key changes.

Use `rememberCoroutineScope` for work initiated by events such as button clicks.

---

## 7.5 SideEffect

`SideEffect` executes after every successful composition.

It is useful when you need to publish Compose-managed information to an external object.

Example:

```kotlin
class AppSettings {
    var currentTheme: String = "Light"
}
```

Composable:

```kotlin
@Composable
fun ThemeScreen() {

    var isDarkMode by remember {
        mutableStateOf(false)
    }

    val appSettings = remember {
        AppSettings()
    }

    val theme =
        if (isDarkMode) "Dark" else "Light"

    SideEffect {
        appSettings.currentTheme = theme
    }

    Button(
        onClick = {
            isDarkMode = !isDarkMode
        }
    ) {
        Text("Change theme")
    }
}
```

### What happens?

Initially:

```kotlin
isDarkMode = false
theme = "Light"
```

After the first successful composition:

```kotlin
appSettings.currentTheme = "Light"
```

The user clicks the button.

Now:

```kotlin
isDarkMode = true
theme = "Dark"
```

After successful recomposition:

```kotlin
appSettings.currentTheme = "Dark"
```

The external object is synchronized with the current Compose information.

### Is SideEffect a one-time initialization API?

No.

It executes after each successful composition.

| Situation | Does SideEffect execute? |
|---|---|
| Initial successful composition | Yes |
| State changes and recomposition succeeds | Yes |
| Another recomposition succeeds | Yes |
| No composition happens | No |
| Composition attempt is cancelled before applying changes | No |

### Incorrect usage

```kotlin
@Composable
fun UserScreen(userId: String) {

    SideEffect {
        fetchUserFromServer(userId)
    }

    Text(userId)
}
```

Why is this problematic?

- It runs after every successful recomposition.
- Unrelated state changes could trigger repeated requests.
- SideEffect is not designed for suspend operations.

### Correct approach

```kotlin
@Composable
fun UserScreen(userId: String) {

    LaunchedEffect(userId) {
        fetchUserFromServer(userId)
    }

    Text(userId)
}
```

The effect executes initially and restarts when `userId` changes.

**Interview takeaway:** `SideEffect` is for publishing information after successful composition, not for loading data or launching asynchronous operations.

---

## 7.6 rememberUpdatedState

`rememberUpdatedState` is useful when an effect should continue running without restarting, but needs access to the latest callback or value.

### Example: Splash Screen timer

Imagine a Splash Screen that waits five seconds and then navigates to another screen.

```kotlin
@Composable
fun SplashScreen(
    onTimeout: () -> Unit
) {

    LaunchedEffect(Unit) {

        delay(5000)

        onTimeout()
    }

    Text("Loading...")
}
```

Suppose:

1. Splash Screen initially receives callback A.
2. The five-second timer starts.
3. After two seconds, the parent recomposes and provides callback B.
4. The original coroutine continues running.

The running coroutine may still hold callback A.

But we want the timer to continue without restarting while invoking callback B at the end.

### Solution

```kotlin
@Composable
fun SplashScreen(
    onTimeout: () -> Unit
) {

    val currentOnTimeout by rememberUpdatedState(
        onTimeout
    )

    LaunchedEffect(Unit) {

        delay(5000)

        currentOnTimeout()
    }

    Text("Loading...")
}
```

### What happens?

| Time | Event |
|---|---|
| 0 seconds | Timer starts with callback A |
| 2 seconds | Parent provides callback B |
| 2 seconds | The remembered state holder now exposes B |
| 5 seconds | Original coroutine completes and invokes B |

The coroutine does not restart.

### Why not use LaunchedEffect(onTimeout)?

```kotlin
LaunchedEffect(onTimeout) {

    delay(5000)

    onTimeout()
}
```

When the callback changes:

1. Previous coroutine is cancelled.
2. A new coroutine starts.
3. Another five-second delay begins.

That is different from our requirement.

### Interview takeaway

- `remember` preserves an object.
- `rememberUpdatedState` maintains a state holder whose value stays up to date.
- `LaunchedEffect` runs coroutine work tied to Composition and keys.

---

## 7.7 DisposableEffect

`DisposableEffect` is used when you need to perform setup and cleanup.

This is particularly useful for:

- Registering listeners.
- Registering lifecycle observers.
- Subscribing to external services.
- Adding callbacks.
- Managing resources that need explicit cleanup.

Example:

```kotlin
@Composable
fun LifecycleScreen(
    lifecycleOwner: LifecycleOwner
) {

    DisposableEffect(lifecycleOwner) {

        val observer = LifecycleEventObserver {
                _, event ->

            println("Lifecycle event: $event")
        }

        lifecycleOwner.lifecycle.addObserver(observer)

        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

### How does it work?

When the effect enters Composition:

1. The setup code executes.
2. The observer is registered.
3. The effect remains active.

When the key changes or the effect leaves Composition:

1. `onDispose` executes.
2. The observer is removed.

This prevents external resources from retaining unnecessary references.

### Why are keys important?

Consider:

```kotlin
DisposableEffect(lifecycleOwner) {
    // Register observer.
    onDispose {
        // Remove observer.
    }
}
```

If `lifecycleOwner` changes, Compose disposes the old effect and starts the new one.

This ensures that the observer is associated with the correct owner.

---

## 7.8 produceState

`produceState` converts external or asynchronous data into Compose State.

It is useful when adapting:

- Suspend functions.
- Callback-based APIs.
- External data sources.
- Asynchronous operations.

### Example 1: Asynchronous data

Imagine a repository:

```kotlin
suspend fun loadUsername(): String {

    delay(2000)

    return "John"
}
```

Use `produceState`:

```kotlin
@Composable
fun UserScreen() {

    val username by produceState(
        initialValue = "Loading..."
    ) {

        value = loadUsername()
    }

    Text(username)
}
```

### Execution

1. The composable enters Composition.
2. The state starts with `"Loading..."`.
3. The producer coroutine starts.
4. `loadUsername()` waits two seconds.
5. It returns `"John"`.
6. `value` is updated.
7. Compose schedules recomposition.
8. The UI displays `"John"`.

A production implementation should also handle exceptions and loading or error states.

### Using keys with produceState

```kotlin
@Composable
fun UserScreen(userId: String) {

    val username by produceState(
        initialValue = "Loading...",
        userId
    ) {

        value = repository.loadUsername(userId)
    }

    Text(username)
}
```

When the key changes, the previous producer is cancelled and the new producer starts.

An important detail: `produceState` retains its state holder across key changes. The initial value is not automatically applied again on every key change. Explicitly update `value` inside the producer when a new source requires a reset.

### Example 2: Callback-based API

Imagine an external message service:

```kotlin
interface MessageSource {

    fun subscribe(
        listener: (String) -> Unit
    ): () -> Unit
}
```

The `subscribe()` method registers a callback and returns a function that removes the subscription.

Now:

```kotlin
@Composable
fun MessageScreen(
    source: MessageSource
) {

    val message by produceState(
        initialValue = "Waiting for messages...",
        source
    ) {

        val unsubscribe = source.subscribe { newMessage ->

            value = newMessage
        }

        awaitDispose {
            unsubscribe()
        }
    }

    Text(message)
}
```

### Understanding awaitDispose

```kotlin
awaitDispose {
    unsubscribe()
}
```

When the producer is cancelled or disposed, the cleanup block executes.

This matters because otherwise an external service might continue holding a reference to the callback.

### What happens when source changes?

1. Previous producer is cancelled.
2. `awaitDispose` executes.
3. Old subscription is removed.
4. New producer starts with the new source.

### Difference between produceState and DisposableEffect

Both can manage callback subscriptions.

However, they have different purposes:

| produceState | DisposableEffect |
|---|---|
| Converts produced data into Compose State | Manages external setup and cleanup |
| Provides a `value` property that updates the UI | Does not itself provide a State value |
| Supports suspend work through its producer | Best suited to resource setup and disposal |
| Uses `awaitDispose` for cleanup | Uses `onDispose` for cleanup |

Use `produceState` when external data must become Compose State.

Use `DisposableEffect` when you only need to register and unregister an external resource.

---

## 7.9 snapshotFlow

`snapshotFlow` observes Compose state reads and exposes their changes as a Kotlin Flow.

It is useful when you want to process Compose state using Flow operators.

### Example: Observe LazyColumn scrolling

```kotlin
@Composable
fun MyScreen() {

    val listState = rememberLazyListState()

    LazyColumn(state = listState) {

        items(100) { index ->
            Text("Item $index")
        }
    }

    LaunchedEffect(listState) {

        snapshotFlow {
            listState.firstVisibleItemIndex
        }.collect { index ->

            println("First visible item: $index")
        }
    }
}
```

### How does it work?

Initially:

```kotlin
firstVisibleItemIndex = 0
```

When the user scrolls, the value might change:

```text
0 → 1 → 2 → 3 → 4
```

`snapshotFlow` observes the state read and emits its results to the collector.

### Example: Detect when the user scrolls past the first item

```kotlin
LaunchedEffect(listState) {

    snapshotFlow {
        listState.firstVisibleItemIndex
    }
        .map { index ->
            index > 0
        }
        .distinctUntilChanged()
        .filter { it }
        .collect {
            println("User scrolled past the first item!")
        }
}
```

### Understanding Flow operators

| Operator | Purpose |
|---|---|
| `map` | Transforms the value |
| `distinctUntilChanged` | Suppresses consecutive equal values |
| `filter` | Allows values matching a condition |
| `collect` | Receives emitted values |

### Important properties of snapshotFlow

**A. It is a cold Flow.**

Observation starts when a collector subscribes.

**B. It initially evaluates and emits its block's result.**

For example, if the first visible index is zero, the collector initially receives zero.

**C. Equal consecutive results are suppressed.**

If the calculated value remains the same, another equal value is not emitted.

**D. It is conflated.**

Rapid state updates may be represented by the latest available value rather than preserving every intermediate state transition.

Do not use it as a guaranteed record of every change that ever happened.

### When is snapshotFlow useful?

- Scroll analytics.
- Logging state changes.
- Detecting scroll thresholds.
- Applying Flow transformations to Compose state.
- Performing coroutine-based work in response to Compose state changes.

---

# Part 8: derivedStateOf and State Observation

## 8.1 What is derivedStateOf?

`derivedStateOf` is useful when a frequently changing state produces a value that changes less frequently.

Its purpose is to reduce unnecessary downstream updates.

### Example: Show a scroll-to-top button

Imagine a scrolling list.

You want to display a button only after the user has moved beyond the first item.

```kotlin
@Composable
fun ScrollScreen() {

    val listState = rememberLazyListState()

    val showScrollButton by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex > 0
        }
    }

    Box {

        LazyColumn(state = listState) {

            items(100) { index ->
                Text("Item $index")
            }
        }

        if (showScrollButton) {

            Button(onClick = {}) {
                Text("Scroll to top")
            }
        }
    }
}
```

### Why is derivedStateOf useful?

The scroll index may change frequently:

```text
0 → 1 → 2 → 3 → 4 → 5
```

But the Boolean value changes only at the boundary:

```text
false → true
```

| Scroll index | Derived Boolean |
|---:|---|
| 0 | false |
| 1 | true |
| 2 | true |
| 3 | true |
| 4 | true |
| 0 | false |

`derivedStateOf` caches the calculated result and notifies consumers when that result changes.

It does not prevent every recomposition in the screen. It helps avoid invalidating consumers of the derived state when the derived result remains equal.

## 8.2 When should you NOT use derivedStateOf?

Consider:

```kotlin
val greeting by remember {
    derivedStateOf {
        "Hello $firstName"
    }
}
```

Whenever `firstName` changes, `greeting` changes too.

The input and output change together.

There is little benefit in using `derivedStateOf`.

Instead:

```kotlin
val greeting = "Hello $firstName"
```

Another example:

```kotlin
val doubled = number * 2
```

You do not need `derivedStateOf` here.

### Comparison

| Situation | Use derivedStateOf? |
|---|---|
| Scroll index converted into a Boolean threshold | Yes |
| Search text converted into a Boolean threshold | Often useful |
| Number converted into a doubled value | No |
| First name converted into a greeting | No |
| Frequently changing scroll position converted into visibility | Yes |

### Interview-ready answer

Use `derivedStateOf` when a frequently changing input produces a result that changes less frequently.

Avoid it when the input and output change together, because the overhead provides little benefit.

## 8.3 derivedStateOf versus snapshotFlow

These APIs are related to state observation but solve different problems.

| derivedStateOf | snapshotFlow |
|---|---|
| Produces Compose State | Produces Kotlin Flow |
| Useful for UI-derived state | Useful for coroutine-based observation |
| Reduces downstream updates when derived output remains equal | Allows Flow operators and collectors |
| Commonly used directly in composables | Commonly collected inside LaunchedEffect |

Example:

```kotlin
val showButton by remember {
    derivedStateOf {
        listState.firstVisibleItemIndex > 0
    }
}
```

This produces Boolean Compose State for the UI.

By contrast:

```kotlin
LaunchedEffect(listState) {

    snapshotFlow {
        listState.firstVisibleItemIndex
    }.collect { index ->

        // Perform side effect.
    }
}
```

This produces a Flow for coroutine-based observation.

---

# Part 9: Stability, Skipping, and Performance

## 9.1 What is Stability?

Stability helps Compose determine whether the inputs of a composable have changed and whether it can skip recomposition.

Broadly, Compose reasons about stable and unstable types.

A type is stable when Compose can reliably determine whether its value has changed.

A type is unstable when Compose cannot reliably determine changes through the normal stability contract.

Common stable values include:

- Int.
- Boolean.
- String.
- Immutable data types with stable properties.
- Compose State types.

Types that can be problematic include:

- Mutable classes.
- Standard collection interfaces when their underlying values may be mutable.
- Objects whose contents can change without observable state.

Consider:

```kotlin
data class User(
    var name: String
)
```

This object can be changed internally without notifying Compose.

A better approach for UI models is usually:

```kotlin
data class User(
    val name: String
)
```

Replace the object with a new instance when the value changes.

## 9.2 What is Skipping?

Suppose the parent recomposes and calls a child:

```kotlin
@Composable
fun UserDetails(name: String) {
    Text(name)
}
```

If the child is eligible for skipping and its relevant inputs have not changed, Compose can skip executing its body.

This avoids unnecessary work.

## 9.3 What is Strong Skipping?

Strong Skipping is a Compose compiler mode that allows more composables to be skipped.

It is enabled by default starting with Kotlin 2.0.20.

With Strong Skipping:

- Restartable composables become skippable.
- Stable parameters are compared using structural equality.
- Unstable parameters are compared using instance equality.
- Lambdas with unstable captures can be automatically remembered.

This is an important modern Compose detail.

Older explanations often say that unstable parameters automatically prevent a composable from being skipped.

That is not generally correct for current projects using Strong Skipping.

### Important example

Consider:

```kotlin
data class User(
    var name: String
)
```

Then:

```kotlin
user.name = "Alex"
```

The same object instance is still being passed.

Under Strong Skipping, an unstable parameter may be compared using instance equality.

Because the object reference has not changed, Compose can consider it unchanged and skip the composable.

The UI can remain stale because the internal mutation was not observable.

This is why observable state or immutable replacement is essential.

## 9.4 Why are ordinary Kotlin lists considered unstable?

Consider:

```kotlin
fun UserList(users: List<User>)
```

The `List` interface does not guarantee that the underlying collection cannot be modified.

Compose therefore cannot assume that an ordinary `List` is immutable.

Under Strong Skipping, unstable collections can still be skipped using instance equality when the reference stays the same.

However, changing a list in place does not automatically make its contents observable.

Use immutable replacement, observable collections, or another correct state-management mechanism.

## 9.5 What about @Stable and @Immutable?

Compose provides annotations:

```kotlin
@Stable
```

And:

```kotlin
@Immutable
```

These communicate stability contracts to Compose.

They are not general-purpose performance switches.

Only use them when the type actually satisfies the relevant contract.

Incorrect annotations can lead to Compose skipping UI that should have updated.

## 9.6 remember for expensive calculations

Consider:

```kotlin
@Composable
fun ContactList(
    contacts: List<Contact>
) {

    val sortedContacts = contacts.sortedBy {
        it.name
    }

    LazyColumn {

        items(sortedContacts) { contact ->
            Text(contact.name)
        }
    }
}
```

Every time this composable executes, it sorts the list again.

You can cache the result:

```kotlin
@Composable
fun ContactList(
    contacts: List<Contact>
) {

    val sortedContacts = remember(contacts) {
        contacts.sortedBy {
            it.name
        }
    }

    LazyColumn {

        items(
            items = sortedContacts,
            key = { it.id }
        ) { contact ->

            Text(contact.name)
        }
    }
}
```

Now the sorted result is recalculated when the key changes.

Important: This assumes the input collection is treated immutably and is replaced when its contents change. Mutating the same list instance in place can make this cache stale.

## 9.7 Defer state reads

An important optimization is to read state in the latest appropriate phase.

For example:

```kotlin
Modifier.offset {
    IntOffset(
        x = 0,
        y = listState.firstVisibleItemScrollOffset
    )
}
```

This reads the scroll state in the placement lambda instead of reading it directly in the composable body.

It can allow the state update to trigger layout work rather than recomposition for that read.

Similarly, state read inside drawing lambdas can sometimes be handled through the drawing phase alone.

This can be especially useful for:

- Scrolling.
- Animations.
- Frequently changing visual values.

## 9.8 Common performance mistakes

Avoid:

**1. Expensive calculations in composable bodies**

Cache or move suitable calculations outside composition.

**2. In-place mutation of ordinary objects**

Compose cannot automatically observe arbitrary property mutations.

**3. Missing stable keys in dynamic lists**

Provide stable IDs.

**4. Network calls directly in composable bodies**

Use ViewModels or appropriate effect APIs.

**5. Incorrect stability annotations**

Do not claim immutability if objects can mutate.

**6. Trying to eliminate every recomposition**

Recomposition is normal. Measure first.

## 9.9 How do you debug recomposition?

Android Studio provides Layout Inspector.

You can use it to:

- Inspect the Compose UI hierarchy.
- Observe recomposition counts.
- Observe skipped composables.
- Understand how interactions affect UI execution.

For complex issues, use composition tracing and benchmarks.

A high recomposition count does not automatically mean poor performance.

The actual cost of the work matters.

---

# Part 10: Architecture, Navigation, Animation, Accessibility, and Testing

## 10.1 Recommended application architecture

A common Android architecture is:

```text
              UI Layer
                 |
                 v
            Composable UI
                 |
                 v
              ViewModel
                 |
                 v
             Repository
                 |
                 v
             Data Sources
              /       \
             v         v
         API Server  Database
```

Responsibilities:

**Composable UI**
- Displays information.
- Handles interactions.
- Receives state.
- Emits events.

**ViewModel**
- Owns screen-level state.
- Handles screen-related logic.
- Exposes UI state.

**Repository**
- Coordinates data access.
- Combines local and remote sources.

**Data sources**
- Network APIs.
- Databases.
- Other external systems.

Use Navigation Compose to manage destinations and navigation.

## 10.2 Animation

Important animation APIs include:

- `animate*AsState`
- `AnimatedVisibility`
- `AnimatedContent`
- `updateTransition`
- `Animatable`

Example:

```kotlin
@Composable
fun AnimatedBox() {

    var expanded by remember {
        mutableStateOf(false)
    }

    val size by animateDpAsState(
        targetValue = if (expanded) 200.dp else 100.dp,
        label = "Box size"
    )

    Box(
        modifier = Modifier
            .size(size)
            .background(Color.Blue)
            .clickable {
                expanded = !expanded
            }
    )
}
```

When the state changes, the animation API animates the transition.

## 10.3 Accessibility

Good Compose UI should be accessible.

Remember:

- Provide meaningful content descriptions for informative images.
- Use Material components when appropriate.
- Provide useful labels and state descriptions.
- Make interactive elements understandable to screen readers.
- Test with accessibility services.

Compose uses a semantics system to describe UI elements to accessibility services and UI tests.

## 10.4 Compose testing

Compose provides UI testing APIs.

Example:

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun counterIncrements() {

    composeTestRule.setContent {
        CounterScreen()
    }

    composeTestRule
        .onNodeWithText("Increase")
        .performClick()

    composeTestRule
        .onNodeWithText("Count: 1")
        .assertIsDisplayed()
}
```

Important testing APIs:

| API | Purpose |
|---|---|
| `onNodeWithText()` | Finds an element by text |
| `onNodeWithTag()` | Finds an element using a test tag |
| `performClick()` | Performs a click |
| `assertIsDisplayed()` | Verifies visibility |
| `performTextInput()` | Enters text |
| `assertTextEquals()` | Checks displayed text |

Test actual user behavior rather than only checking that a composable renders.

---

# Part 11: Important Compose API Comparison

Use this table for revision.

| API | Main responsibility |
|---|---|
| `remember` | Retains an object across recompositions |
| `rememberSaveable` | Retains supported saved state across recreation |
| `mutableStateOf` | Creates observable mutable state |
| `mutableIntStateOf` | Creates observable integer state |
| `mutableStateListOf` | Creates an observable mutable collection |
| `SideEffect` | Publishes information after successful composition |
| `LaunchedEffect` | Launches coroutine work tied to Composition and keys |
| `rememberCoroutineScope` | Provides a scope for launching coroutines from UI events |
| `rememberUpdatedState` | Provides the latest value without restarting an ongoing effect |
| `DisposableEffect` | Performs setup and cleanup |
| `produceState` | Converts asynchronous or external data into Compose State |
| `snapshotFlow` | Converts Compose state observations into a Flow |
| `derivedStateOf` | Caches a derived value to reduce downstream updates |
| `key()` | Gives a group of composables identity |
| LazyColumn item `key` | Identifies list items |

---

# Part 12: Practical Code Review Challenges

Try solving each challenge before reading the explanation.

For every problem, ask:

1. What is wrong?
2. Why is it wrong?
3. What happens at runtime?
4. How would you fix it?

## Challenge 1: Mutable List and State

**Difficulty: Easy–Medium**

```kotlin
@Composable
fun ShoppingCart() {

    val items = remember {
        mutableListOf("Apple", "Banana")
    }

    Column {

        Text("Items: ${items.size}")

        Button(
            onClick = {
                items.add("Orange")
            }
        ) {
            Text("Add item")
        }

        items.forEach { item ->
            Text(item)
        }
    }
}
```

Questions:

1. Will the UI automatically recompose when an item is added?
2. Does `remember` make a list observable?
3. What is the difference between `mutableListOf()` and `mutableStateListOf()`?
4. Show two ways to fix this code.

### Solution

Approach 1:

```kotlin
val items = remember {
    mutableStateListOf("Apple", "Banana")
}
```

Approach 2:

```kotlin
var items by remember {
    mutableStateOf(listOf("Apple", "Banana"))
}

items = items + "Orange"
```

Both approaches make the relevant changes observable.

---

## Challenge 2: LaunchedEffect and Coroutine Lifecycle

**Difficulty: Medium**

```kotlin
@Composable
fun UserProfile(userId: String) {

    var userName by remember {
        mutableStateOf("")
    }

    LaunchedEffect(Unit) {

        delay(500)

        userName = fetchUser(userId)
    }

    Text("User: $userName")
}
```

Scenario:

1. User opens profile 101.
2. Two seconds later, the parent provides user ID 202.

Questions:

1. Will LaunchedEffect execute again?
2. What happens to the existing coroutine?
3. Could the screen display the wrong user's information?
4. How would you fix it?
5. What happens if the ID changes several times quickly?

### Solution

```kotlin
@Composable
fun UserProfile(userId: String) {

    var userName by remember {
        mutableStateOf("")
    }

    LaunchedEffect(userId) {

        userName = fetchUser(userId)
    }

    Text("User: $userName")
}
```

When the key changes, the previous coroutine is cancelled and a new one starts.

For production code, also model loading, errors, cancellation, and stale data. Consider keeping screen-level API loading in the ViewModel.

---

## Challenge 3: Recomposition and derivedStateOf

**Difficulty: Medium–Hard**

```kotlin
@Composable
fun ScrollScreen() {

    val listState = rememberLazyListState()

    val showScrollButton =
        listState.firstVisibleItemIndex > 0

    Box {

        LazyColumn(state = listState) {

            items(100) { index ->
                Text("Item $index")
            }
        }

        if (showScrollButton) {

            Button(onClick = {}) {
                Text("Scroll to top")
            }
        }
    }
}
```

Questions:

1. What happens when the user scrolls?
2. Why might this cause unnecessary recompositions?
3. How can derivedStateOf help?
4. Rewrite the code.

### Solution

```kotlin
@Composable
fun ScrollScreen() {

    val listState = rememberLazyListState()

    val showScrollButton by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex > 0
        }
    }

    Box {

        LazyColumn(state = listState) {

            items(100) { index ->
                Text("Item $index")
            }
        }

        if (showScrollButton) {

            Button(onClick = {}) {
                Text("Scroll to top")
            }
        }
    }
}
```

The Boolean only changes when the first visible item crosses the threshold.

---

## Challenge 4: Resource Management

**Difficulty: Hard**

Imagine an external location service:

```kotlin
interface LocationService {

    fun subscribe(
        callback: (String) -> Unit
    ): () -> Unit
}
```

Review this implementation:

```kotlin
@Composable
fun LocationScreen(
    locationService: LocationService
) {

    val location by produceState(
        initialValue = "Unknown"
    ) {

        locationService.subscribe { newLocation ->
            value = newLocation
        }
    }

    Text("Location: $location")
}
```

Questions:

1. What happens when LocationScreen leaves Composition?
2. Is the listener removed?
3. Could this cause a resource leak?
4. How would you fix it with awaitDispose?
5. What happens if locationService changes?

### Solution

```kotlin
@Composable
fun LocationScreen(
    locationService: LocationService
) {

    val location by produceState(
        initialValue = "Unknown",
        locationService
    ) {

        val unsubscribe =
            locationService.subscribe { newLocation ->
                value = newLocation
            }

        awaitDispose {
            unsubscribe()
        }
    }

    Text("Location: $location")
}
```

The key ensures that a new service causes the previous producer to be cancelled.

The cleanup function removes the old subscription.

---

## Challenge 5: LazyColumn Stable Keys

**Difficulty: Hard**

```kotlin
data class Product(
    val id: Int,
    val name: String
)
```

Composable:

```kotlin
@Composable
fun ProductList(
    products: List<Product>
) {

    LazyColumn {

        items(products) { product ->

            var expanded by remember {
                mutableStateOf(false)
            }

            Column {

                Text(product.name)

                Button(
                    onClick = {
                        expanded = !expanded
                    }
                ) {
                    Text("Toggle")
                }

                if (expanded) {
                    Text("Product details")
                }
            }
        }
    }
}
```

Scenario:

1. You have Apple, Banana, and Orange.
2. You expand Banana.
3. You remove Apple.
4. The list updates.

Questions:

1. What could happen to the expanded state?
2. Why can remembered state become associated with the wrong item?
3. How do stable keys help?
4. Rewrite using product ID.

### Solution

```kotlin
@Composable
fun ProductList(
    products: List<Product>
) {

    LazyColumn {

        items(
            items = products,
            key = { product ->
                product.id
            }
        ) { product ->

            var expanded by remember {
                mutableStateOf(false)
            }

            Column {

                Text(product.name)

                Button(
                    onClick = {
                        expanded = !expanded
                    }
                ) {
                    Text("Toggle")
                }

                if (expanded) {
                    Text("Product details")
                }
            }
        }
    }
}
```

Stable keys allow Compose to identify Banana using its ID rather than only its position.

---

# Part 13: Multiple-Choice Interview Questions

Choose one answer for each question.

### 1. What does remember do?

A. Triggers recomposition whenever any object changes.

B. Preserves an object across recompositions while its call remains in the Composition.

C. Stores data permanently across application restarts.

D. Creates a new object on every recomposition.

### 2. What happens when a normal mutableListOf is modified?

A. Compose always detects the mutation.

B. The entire application recomposes.

C. Compose is not automatically notified about the mutation.

D. The list becomes immutable.

### 3. When does LaunchedEffect(Unit) restart?

A. On every recomposition.

B. Whenever any variable changes.

C. When its call leaves and re-enters the Composition.

D. Whenever the UI redraws.

### 4. When does SideEffect execute?

A. Before composition.

B. After every successful composition.

C. Only once when entering Composition.

D. Only when a button is clicked.

### 5. What is rememberUpdatedState used for?

A. Restarting an effect whenever a callback changes.

B. Keeping the latest value available without restarting the ongoing effect.

C. Persisting data after the application closes.

D. Preventing recomposition completely.

### 6. What does produceState provide?

A. A normal mutable Kotlin list.

B. A Flow that must be collected manually.

C. Compose State produced from asynchronous or external data.

D. Permanent database storage.

### 7. What does snapshotFlow do?

A. Converts Kotlin Flow into a database.

B. Converts Compose state reads into a Flow.

C. Prevents state changes.

D. Runs a coroutine on every recomposition.

### 8. What happens when a snapshotFlow block produces the same result repeatedly?

A. It emits the same value every time.

B. It cancels the coroutine.

C. It suppresses consecutive equal results.

D. It triggers a new Composition.

### 9. Why use derivedStateOf?

A. To make any Kotlin object observable.

B. To convert every Compose State into a Flow.

C. To reduce unnecessary downstream updates when a derived value changes less frequently than its inputs.

D. To save state after process death.

### 10. Why use stable keys in LazyColumn?

A. To make all items immutable.

B. To identify items consistently when the list changes.

C. To prevent scrolling.

D. To force every item to recompose.

## Answer key

| Question | Answer | Explanation |
|---|---|---|
| 1 | B | remember retains an object across recompositions |
| 2 | C | A normal mutable list is not observable Compose state |
| 3 | C | Unit remains the same key; leaving and re-entering restarts the effect |
| 4 | B | SideEffect executes after successful compositions |
| 5 | B | It provides the latest value without restarting an ongoing effect |
| 6 | C | It converts produced asynchronous values into Compose State |
| 7 | B | It observes Compose State and exposes values as Flow emissions |
| 8 | C | Equal consecutive results are suppressed |
| 9 | C | It helps when inputs change more frequently than the derived result |
| 10 | B | Keys preserve item identity |

---

# Part 14: Advanced Conceptual Interview Questions

Try answering these without looking at the previous sections.

### Q1. Composition

What is the difference between initial Composition and Recomposition?

### Q2. Recomposition

Does Recomposition mean the entire application UI tree executes again?

### Q3. State Observation

How does Compose know which composable scopes depend on a particular state value?

### Q4. Optimistic Recomposition

Why can Compose cancel and restart a recomposition?

### Q5. Composable Functions

Why should composable functions avoid uncontrolled side effects?

### Q6. remember

What is the difference between retaining a value during recomposition and retaining state across Activity recreation?

### Q7. Delegated Properties

What does the Kotlin `by` keyword do when used with MutableState?

### Q8. Mutable Collections

Why does modifying a normal mutable list not automatically trigger Recomposition?

### Q9. LaunchedEffect

What happens when the key of LaunchedEffect changes?

### Q10. rememberUpdatedState

Why would you choose rememberUpdatedState rather than adding a callback as a LaunchedEffect key?

### Q11. Resource Management

What is the difference between DisposableEffect and produceState with awaitDispose?

### Q12. derivedStateOf

Why is derivedStateOf useful for scroll thresholds but often unnecessary for simple string concatenation?

### Q13. snapshotFlow

Why should you not expect snapshotFlow to record every intermediate state change?

### Q14. Stability

What is the difference between stable and unstable parameters?

### Q15. Strong Skipping

How does Strong Skipping change the behavior of composables with unstable parameters?

### Q16. LazyColumn

What is the difference between item position and item identity?

### Q17. Layout Phases

What are the three major phases in Compose?

### Q18. Performance

Why should you diagnose performance problems before applying stability annotations?

---

# Part 15: Complete Interview Preparation Checklist

Use this checklist to measure your understanding.

## Kotlin fundamentals

- [ ] Delegated properties using `by`
- [ ] Getter and setter delegation
- [ ] `getValue()` and `setValue()`
- [ ] Mutable versus immutable collections
- [ ] State holder objects

## Compose fundamentals

- [ ] Declarative UI
- [ ] Composable functions
- [ ] Modifiers
- [ ] Layout containers
- [ ] Material 3
- [ ] Preview

## Composition and Recomposition

- [ ] Initial Composition
- [ ] Recomposition
- [ ] Recomposer
- [ ] Invalidation
- [ ] State observation
- [ ] Composition tree
- [ ] Composition identity
- [ ] Optimistic recomposition
- [ ] Skipping
- [ ] Layout and drawing phases

## State management

- [ ] remember
- [ ] rememberSaveable
- [ ] mutableStateOf
- [ ] mutableIntStateOf
- [ ] mutableStateListOf
- [ ] Immutable list replacement
- [ ] State Hoisting
- [ ] Unidirectional Data Flow
- [ ] ViewModel and StateFlow

## Lifecycle and identity

- [ ] Composable lifecycle
- [ ] Remembered state disposal
- [ ] remember keys
- [ ] key composable
- [ ] LazyColumn stable keys
- [ ] State preservation across recreation

## Side Effects

- [ ] LaunchedEffect
- [ ] rememberCoroutineScope
- [ ] SideEffect
- [ ] rememberUpdatedState
- [ ] DisposableEffect
- [ ] produceState
- [ ] awaitDispose
- [ ] snapshotFlow
- [ ] Effect keys
- [ ] Coroutine cancellation

## Derived state and Flow

- [ ] derivedStateOf
- [ ] Cold Flows
- [ ] Flow operators
- [ ] Initial snapshotFlow emission
- [ ] Equality suppression
- [ ] Conflation
- [ ] State-to-Flow conversion

## Performance

- [ ] Stability
- [ ] Strong Skipping
- [ ] Immutable UI models
- [ ] Stable keys
- [ ] Expensive calculation caching
- [ ] Deferred state reads
- [ ] Recomposition inspection
- [ ] Layout Inspector
- [ ] Performance measurement

## Application development

- [ ] ViewModel
- [ ] Repository pattern
- [ ] Navigation Compose
- [ ] Animation
- [ ] Accessibility
- [ ] UI testing

---

# Part 16: Final Cheat Sheet

| Concept | Remember this |
|---|---|
| Jetpack Compose | Declarative UI toolkit |
| `@Composable` | Marks a composable function |
| Composition | Creates the initial UI description |
| Recomposition | Updates the existing Composition |
| Recomposer | Coordinates recomposition |
| Invalidation | Marks a scope for reconsideration |
| State read | Establishes an observable dependency |
| `mutableStateOf` | Creates observable mutable state |
| `by` | Delegates Kotlin property access |
| `remember` | Retains an object across recompositions |
| `rememberSaveable` | Retains supported saved state across recreation |
| `mutableStateListOf` | Creates an observable mutable collection |
| State Hoisting | Moves state ownership to a parent |
| UDF | State flows down, events flow up |
| `key()` | Provides composable identity |
| LazyColumn key | Identifies list items |
| `LaunchedEffect` | Runs coroutine work tied to Composition |
| `rememberCoroutineScope` | Launches coroutines from UI events |
| `SideEffect` | Publishes information after successful composition |
| `rememberUpdatedState` | Keeps the latest value available to an ongoing effect |
| `DisposableEffect` | Registers and cleans up external resources |
| `produceState` | Converts asynchronous data into Compose State |
| `awaitDispose` | Cleans up a producer's external resources |
| `snapshotFlow` | Converts state observations into Flow |
| `derivedStateOf` | Caches derived state |
| Stability | Helps Compose decide whether parameters changed |
| Strong Skipping | Makes more restartable composables skippable |
| Layout | Measures and positions elements |
| Drawing | Renders visual elements |

---

# Part 17: What to Learn After This Tutorial

Once you understand this guide, follow this learning sequence.

### 1. Compose Runtime Internals

Study:

- Composer.
- Recomposer.
- Slot table.
- Restart scopes.
- Snapshots.
- Compiler-generated code.
- How the runtime applies changes.

### 2. Advanced State Management

Study:

- ViewModel.
- StateFlow.
- SharedFlow.
- SavedStateHandle.
- State restoration.
- UI state and UI events.
- Repository integration.

### 3. Advanced Layouts

Study:

- Custom Layout.
- SubcomposeLayout.
- Measurement and constraints.
- Intrinsics.
- Custom Modifier.
- Graphics and drawing.

### 4. Performance Engineering

Study:

- Stability reports.
- Strong Skipping.
- Deferred state reads.
- Composition tracing.
- Macrobenchmark.
- Profiling and frame timing.

### 5. Advanced UI Development

Study:

- AnimationSpec.
- Animatable.
- updateTransition.
- Navigation Compose.
- Adaptive layouts.
- Accessibility.
- Screenshot testing.

---

# Final Takeaways

The most important idea in Jetpack Compose is understanding the relationship between **state, observation, Composition, Recomposition, and lifecycle**.

Remember these rules:

1. Composition creates the initial UI description.
2. Recomposition updates the existing Composition.
3. Compose tracks where observable state is read.
4. State changes can invalidate the scopes that read that state.
5. Recomposition does not necessarily execute the entire UI tree.
6. `remember` preserves an object but does not make ordinary internal mutations observable.
7. `mutableStateListOf` makes collection operations observable.
8. `by` is Kotlin delegated property access, not a state-management mechanism on its own.
9. `rememberSaveable` supports restoring appropriate UI state across recreation.
10. `LaunchedEffect` manages coroutine work according to its keys and Composition lifecycle.
11. `SideEffect` runs after successful compositions.
12. `rememberUpdatedState` provides the latest value without restarting an ongoing effect.
13. `produceState` converts asynchronous or external data into Compose State.
14. `snapshotFlow` converts Compose state observations into Kotlin Flow.
15. `derivedStateOf` is useful when a frequently changing input produces a less frequently changing result.
16. Stable keys help Compose preserve the identity of dynamic list items.
17. Strong Skipping allows more composables to be skipped, but it does not make ordinary object mutations observable.
18. Composition, Layout, and Drawing are distinct phases.

## The most important mental model

```text
                APPLICATION STATE
                        |
                        v
                 STATE OBSERVATION
                        |
                        v
                  INVALIDATION
                        |
                        v
                  RECOMPOSITION
                        |
                        v
                UPDATED COMPOSITION
                        |
                        v
                     LAYOUT
                        |
                        v
                    DRAWING
                        |
                        v
                  UPDATED SCREEN
```

**A strong Compose developer does not merely memorize which API to use. They understand why it exists, what triggers it, how its lifecycle works, and what happens when the Composition changes.**
