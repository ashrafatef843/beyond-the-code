# Technical Code Review Challenge: QuoteViewModel Refactoring

## Exercise Context & Prompt

> *"Imagine a peer on the team wrote this ViewModel for submitting a heat pump quote from our Android app. Walk me through a live code review as if this were a production pull request, sharing your thought process aloud as you read. Suggest how you would fix, improve, and refactor this code to make it production-ready."*

**Source:** [Gist: hopeman15/QuoteViewModel.kt](https://gist.github.com/hopeman15/61fb3e208bea106c2fef568c2a111adc)

---

## 1. Original Unrefactored Code (`QuoteViewModel.kt`)

```kotlin
import android.content.Context
import android.widget.Toast
import androidx.lifecycle.ViewModel
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.launch
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.RequestBody.Companion.toRequestBody
import org.json.JSONObject

sealed interface QuoteUiState {
    object Idle : QuoteUiState
    object Loading : QuoteUiState
    data class Success(val grant: Double) : QuoteUiState
}

class QuoteViewModel(private val context: Context) : ViewModel() {
    val uiState = MutableStateFlow<QuoteUiState>(QuoteUiState.Idle)
    private val client = OkHttpClient()
    private val api = QuoteApi.create()

    fun submitQuote(quoteId: String, heatPumpPrice: Double, address: String?) {
        GlobalScope.launch(Dispatchers.Main) {
            uiState.value = QuoteUiState.Loading

            val quote = api.getQuote(quoteId)
            val installations = api.getInstallations(quote.customerId)

            var totalExistingSubsidies = 0.0
            for (inst in installations) {
                val subsidy = api.getSubsidy(inst.id)
                if (subsidy.status == "APPROVED") {
                    totalExistingSubsidies += subsidy.amount
                }
            }

            val eligibleCap = 30000.0
            val eligibleAmount = minOf(heatPumpPrice, eligibleCap)
            val grantAmount = eligibleAmount * 0.70

            try {
                val body = JSONObject()
                    .put("quote_id", quoteId)
                    .put("address", address!!)
                    .toString()
                    .toRequestBody("application/json".toMediaType())

                val request = Request.Builder()
                    .url("https://api.thermondo.de/v1/dispatch")
                    .header("Authorization", "Bearer dev_secret_key_88912")
                    .post(body)
                    .build()

                client.newCall(request).execute()
            } catch (e: Exception) {
                // ignore
            }

            api.updateQuoteStatus(quoteId, "SUBMITTED", grantAmount)
            uiState.value = QuoteUiState.Success(grantAmount)
            Toast.makeText(context, "Quote submitted!", Toast.LENGTH_SHORT).show()
        }
    }
}
```

---

## 2. Live Code Review Findings (Issues & Anti-Patterns)

During a code review, I would highlight the following 13 key issues categorized by architecture, thread safety, memory management, and error handling:

### 🚨 Critical / App-Crashing Issues
1. **Synchronous Network Call on Main Thread (`NetworkOnMainThreadException`)**:
   `client.newCall(request).execute()` is a synchronous blocking network call executed inside `GlobalScope.launch(Dispatchers.Main)`. This will immediately crash the app on Android.
2. **Unsafe Null Assertion (`address!!`)**:
   `address!!` will throw a `NullPointerException` if `address` is null. Validation should occur before network requests or accept non-nullable input.
3. **Hardcoded Dev Secret in Source Code**:
   `header("Authorization", "Bearer dev_secret_key_88912")` exposes credentials in plain text. Authorization must be handled via a secure network layer (e.g., OkHttp Interceptor or BuildConfig/encrypted storage).

### ⚠️ Memory Leaks & Lifecycle Anti-Patterns
4. **Android Context Leak in ViewModel**:
   `class QuoteViewModel(private val context: Context) : ViewModel()` holds a strong reference to `Context` (likely an Activity Context). This prevents the Activity from being garbage collected on configuration change.
5. **UI Logic Inside ViewModel (`Toast.makeText`)**:
   ViewModels should not display UI components (Toasts, Dialogs). UI side-effects belong in the UI layer (Composable / Activity / Fragment) observing state or single-shot events.
6. **`GlobalScope` Abuse**:
   `GlobalScope.launch` ignores ViewModel lifecycle cancellation. If the user leaves the screen, background network operations continue running. Use `viewModelScope`.

### 🛠️ Architecture & Domain Logic Flaws
7. **Exposed Public Mutable State**:
   `val uiState = MutableStateFlow(...)` exposes mutable state directly. External callers can mutate `uiState.value`. State should be encapsulated as `private val _uiState = MutableStateFlow(...)` and exposed as `val uiState: StateFlow`.
8. **Direct Network Execution in ViewModel**:
   The ViewModel directly builds HTTP requests, handles JSON serialization, and interacts with raw OkHttp/Retrofit clients. This violates Separation of Concerns. Logic should be delegated to a Repository and Data Sources.
9. **Dead Code / Unused Calculation**:
   `totalExistingSubsidies` is calculated in a sequential loop but never used in `grantAmount` calculation or passed downstream.
10. **N+1 Network Query Problem**:
    The `for (inst in installations)` loop executes `api.getSubsidy(inst.id)` sequentially per installation, causing heavy latency. Concurrency (`async`/`awaitAll`) or a bulk backend endpoint should be used.
11. **Hardcoded Business Rules**:
    `30000.0` (eligible cap) and `0.70` (grant rate) are hardcoded in the presentation layer. These rules belong in the domain layer / configuration.
12. **Swallowed Exceptions**:
    `catch (e: Exception) { // ignore }` hides network or parsing failures and allows the code to proceed as if submission succeeded.
13. **Deprecated Kotlin Sealed Class Objects**:
    `object Idle : QuoteUiState` should use `data object Idle : QuoteUiState` (Kotlin 1.9+). `QuoteUiState` should also include an `Error(val message: String)` state.

---

## 3. Production-Ready Refactored Architecture

To make this code production-ready, we apply **Clean Architecture** and **Unidirectional Data Flow (UDF)**:

1. **Domain Layer**: `CalculateGrantUseCase` encapsulates subsidy and grant eligibility rules.
2. **Repository Layer**: `QuoteRepository` coordinates data sources, network calls, and exception handling.
3. **Presentation Layer**: `QuoteViewModel` uses `viewModelScope`, exposes read-only `StateFlow`, and delegates business & network tasks.

---

## 4. Refactored Production Code

### A. UI State (`QuoteUiState.kt`)

```kotlin
sealed interface QuoteUiState {
    data object Idle : QuoteUiState
    data object Loading : QuoteUiState
    data class Success(val grantAmount: Double) : QuoteUiState
    data class Error(val message: String) : QuoteUiState
}
```

---

### B. Domain / Use Case (`CalculateGrantUseCase.kt`)

```kotlin
class CalculateGrantUseCase {
    companion object {
        private const val ELIGIBLE_CAP = 30000.0
        private const val GRANT_PERCENTAGE = 0.70
    }

    operator fun invoke(heatPumpPrice: Double, existingSubsidies: Double): Double {
        val remainingCap = maxOf(0.0, ELIGIBLE_CAP - existingSubsidies)
        val eligibleAmount = minOf(heatPumpPrice, remainingCap)
        return eligibleAmount * GRANT_PERCENTAGE
    }
}
```

---

### C. Repository Interface & Implementation (`QuoteRepository.kt`)

```kotlin
interface QuoteRepository {
    suspend fun submitQuote(
        quoteId: String,
        heatPumpPrice: Double,
        address: String
    ): Result<Double>
}

class QuoteRepositoryImpl(
    private val apiService: QuoteApiService,
    private val calculateGrantUseCase: CalculateGrantUseCase,
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) : QuoteRepository {

    override suspend fun submitQuote(
        quoteId: String,
        heatPumpPrice: Double,
        address: String
    ): Result<Double> = withContext(ioDispatcher) {
        runCatching {
            val quote = apiService.getQuote(quoteId)
            val installations = apiService.getInstallations(quote.customerId)

            // Fetch subsidies concurrently instead of sequential N+1 requests
            val totalExistingSubsidies = coroutineScope {
                installations.map { inst ->
                    async {
                        val subsidy = apiService.getSubsidy(inst.id)
                        if (subsidy.status == "APPROVED") subsidy.amount else 0.0
                    }
                }.awaitAll().sum()
            }

            val grantAmount = calculateGrantUseCase(heatPumpPrice, totalExistingSubsidies)

            // Dispatch quote payload
            apiService.dispatchQuote(
                DispatchRequest(quoteId = quoteId, address = address)
            )

            // Update quote status
            apiService.updateQuoteStatus(
                quoteId = quoteId,
                status = "SUBMITTED",
                grantAmount = grantAmount
            )

            grantAmount
        }
    }
}
```

---

### D. Production ViewModel (`QuoteViewModel.kt`)

```kotlin
class QuoteViewModel(
    private val quoteRepository: QuoteRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow<QuoteUiState>(QuoteUiState.Idle)
    val uiState: StateFlow<QuoteUiState> = _uiState.asStateFlow()

    fun submitQuote(quoteId: String, heatPumpPrice: Double, address: String?) {
        if (address.isNull@Blank()) {
            _uiState.value = QuoteUiState.Error("Address cannot be empty")
            return
        }

        viewModelScope.launch {
            _uiState.value = QuoteUiState.Loading

            quoteRepository.submitQuote(
                quoteId = quoteId,
                heatPumpPrice = heatPumpPrice,
                address = address
            ).onSuccess { grantAmount ->
                _uiState.value = QuoteUiState.Success(grantAmount)
            }.onFailure { exception ->
                _uiState.value = QuoteUiState.Error(
                    exception.localizedMessage ?: "Failed to submit quote"
                )
            }
        }
    }
}
```

---

## 5. Summary Checklist for Candidates

| Area | Bad Practice | Production Fix |
|---|---|---|
| **Context** | Held `Context` in ViewModel | Removed `Context`; UI handles Toasts/strings |
| **State** | Public `MutableStateFlow` | `private MutableStateFlow` + public `asStateFlow()` |
| **Lifecycle** | `GlobalScope` | `viewModelScope` |
| **Threading** | `execute()` on `Dispatchers.Main` | `Dispatchers.IO` via Repository / Retrofit |
| **Concurrency** | Sequential `for` loop N+1 requests | `async` / `awaitAll()` concurrent fetch |
| **Errors** | Empty `catch (e: Exception)` | `runCatching` / `Result` -> `QuoteUiState.Error` |
| **Security** | Hardcoded auth secret | OkHttp Auth Interceptor / Encrypted Config |
| **Safety** | `address!!` force-unwrap | Early validation (`isNullOrBlank()`) |
