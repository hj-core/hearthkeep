# HearthKeep: Kotlin Multiplatform Code Review Checklist

## Instruction to the Reviewing LLM

You will code review a given subject. First, read this checklist completely. Evaluate the provided source code strictly against these rules. For every violation discovered, output a structured ticket containing:

- **Rule ID**: The alphanumeric tag violated.
- **Severity**: CRITICAL, WARNING, or STYLE.
- **Location**: The specific file and code section.
- **Explanation**: A concise description of the issue.
- **Suggested Fix**: Refactored code complying with the rule.

If you discover a structural or behavioral issue that is NOT covered by the checklist below, still report it using the same ticket format with a descriptive placeholder Rule ID (e.g., `[CUSTOM-01]`). Clearly state that it falls outside the defined checklist.

---

## 1. Architectural Integrity & Platform Boundaries [ARCH]

### [ARCH-01] Module Separation

- **Target Scope**: `commonMain/**/*`
- **Severity**: CRITICAL
- **DO**: Keep all core business logic, database definitions, entities, repository interfaces, and use cases strictly inside the shared `commonMain` directory.
- **DO NOT**: Import any platform-specific framework dependencies (e.g., `android.content.Context`, `android.util.Log`, `android.os.Parcelable`) into the `commonMain` module.

### [ARCH-02] Platform Abstraction

- **Target Scope**: `commonMain/**/*`
- **Severity**: CRITICAL
- **DO**: Abstract native capabilities (such as local file paths or local OS notification triggers) using clean interfaces or standard `expect`/`actual` declarations.
- **DO NOT**: Use conditional environment runtime checks within the shared core layer to determine platform types.

### [ARCH-03] Dependency Resolution

- **Target Scope**: All Source Sets
- **Severity**: WARNING
- **DO**: Register and resolve all application dependencies via a KMP-compatible dependency injection framework or a clean Service Locator pattern.
- **DO NOT**: Hardcode structural instantiations or couple class definitions directly to specific implementations.

### [ARCH-04] Lifecycle Binding

- **Target Scope**: `*ViewModel.kt`
- **Severity**: CRITICAL
- **DO**: Scope business logic ViewModels tightly to the hosting platform's architecture lifecycles to guarantee resource disposal.
- **DO NOT**: Allow persistent un-scoped ViewModels to survive independently across configuration changes like screen rotations.

---

## 2. Local Storage & Room Database Operations [DB]

### [DB-01] Main-Thread Safety

- **Target Scope**: `*Dao.kt`, `*Repository.kt`
- **Severity**: CRITICAL
- **DO**: Offload all database read and write tasks explicitly from the Main UI thread using appropriate background dispatchers.
- **DO NOT**: Perform blocking SQLite operations on the main thread, causing frames to drop.

### [DB-02] Asynchronous Exposure

- **Target Scope**: `*Dao.kt`
- **Severity**: CRITICAL
- **DO**: Enclose database mutations inside explicit Kotlin `suspend` functions and expose database queries as reactive data streams using Kotlin `Flow`.
- **DO NOT**: Expose raw, blocking, synchronous object collections from database access layers.

### [DB-03] Transactional Boundaries

- **Target Scope**: `*Dao.kt`, `*Repository.kt`
- **Severity**: CRITICAL
- **DO**: Annotate methods executing multi-table modifications (such as decreasing stock counts and concurrently creating an automated entry in the shopping list) with `@Transaction` to enforce ACID compliance.
- **DO NOT**: Allow multi-step relational mutations to execute as isolated database statements outside of a single transaction boundary.

### [DB-04] Relational Integrity

- **Target Scope**: `*Entity.kt`
- **Severity**: WARNING
- **DO**: Declare explicit Foreign Key relationships between related data models (such as linking an `Item` back to its parent `Category`).
- **DO NOT**: Define detached relational structures that allow orphaned child entries to persist in the local storage database.

### [DB-05] Schema Safety

- **Target Scope**: Database Infrastructure
- **Severity**: CRITICAL
- **DO**: Export the Room JSON database schema to version control and author incremental migration scripts (`Migration(1, 2)`) for structural changes.
- **DO NOT**: Use destructive database fallback behaviors (`fallbackToDestructiveMigration()`) in production code paths.

---

## 3. Concurrency, Streams, & State Management [CONC]

### [CONC-01] Structured Concurrency

- **Target Scope**: All Source Sets
- **Severity**: CRITICAL
- **DO**: Launch all asynchronous routines within a managed, lifecycle-aware coroutine context scope (such as `viewModelScope`).
- **DO NOT**: Instantiate unmanaged coroutines or utilize the global runtime scope (`GlobalScope`) under any circumstances.

### [CONC-02] Dispatcher Allocation

- **Target Scope**: All Source Sets
- **Severity**: CRITICAL
- **DO**: Explicitly route tasks to their mathematically optimal thread pool via dispatchers:
  - `Dispatchers.IO` for storage reads, writes, and database operations.
  - `Dispatchers.Default` for heavy calculations (such as parsing dates or indexing local search tokens).
  - `Dispatchers.Main` exclusively for UI updates.
- **DO NOT**: Perform computationally intense tasks on I/O pools, or I/O blockages on computation pools.

### [CONC-03] Stream Management

- **Target Scope**: `*ViewModel.kt`
- **Severity**: WARNING
- **DO**: Transform raw data streams into hot variables using `StateFlow` or `SharedFlow` before exposing them to the presentation layer.
- **DO NOT**: Expose cold data streams (`Flow`) directly to the layout layer, which forces duplicate upstream evaluations for every collector.

### [CONC-04] Suspend Function Discipline

- **Target Scope**: All Source Sets
- **Severity**: WARNING
- **DO**: Use `suspend` for operations that are inherently asynchronous or main-safe — e.g., database reads/writes, network calls, or any I/O that requires explicit dispatcher switching.
- **DO**: Keep `suspend` functions pure with respect to cancellation — they should either respond to cancellation promptly or document their cancellation behavior.
- **DO NOT**: Declare a function `suspend` if it only performs CPU-bound pure computation without suspending (e.g., arithmetic, string manipulation, in-memory transformations). Such functions should remain synchronous and run on `Dispatchers.Default` if heavy.
- **DO NOT**: Use `suspend` as a signal for "this might take a while" without actually suspending at a call site — prefer `Flow` for streaming or multi-valued async sequences.

---

## 4. Jetpack Compose & Compose Multiplatform UI [UI]

### [UI-01] Flow Direction

- **Target Scope**: `*Screen.kt`, `*Component.kt`
- **Severity**: CRITICAL
- **DO**: Enforce a strict Unidirectional Data Flow model where state values travel downward into composables, and interaction event signals travel upward into the business logic layer.
- **DO NOT**: Allow user interface layouts to directly modify state elements or invoke asynchronous side effects internally.

### [UI-02] List Optimization

- **Target Scope**: `*Screen.kt`, `*Component.kt`
- **Severity**: WARNING
- **DO**: Pass explicit stable unique keys using the `key` parameter inside lazy list iterations (`LazyColumn`, `LazyRow`).
- **DO NOT**: Rely on implicit list item index tracking, which forces complete collection re-render sequences during list sorting or updates.

### [UI-03] Render Invariance

- **Target Scope**: `*Screen.kt`, `*Component.kt`
- **Severity**: WARNING
- **DO**: Intercept costly runtime formatting (such as date transformations or string filtering operations) using `remember` blocks, or move them entirely into the ViewModel.
- **DO NOT**: Place un-cached complex computational transformations directly within UI rendering methods.

---

## 5. Domain Logic & Expiration Engine [DOM]

### [DOM-01] Temporal Engine Uniformity

- **Target Scope**: Core Engine
- **Severity**: CRITICAL
- **DO**: Perform every date abstraction and timezone math operation exclusively using the `kotlinx-datetime` multiplatform library.
- **DO NOT**: Integrate native platform time dependencies (such as `java.util.Date` or `Calendar`) within the shared core layer.

### [DOM-02] Timezone Stability

- **Target Scope**: Core Engine
- **Severity**: CRITICAL
- **DO**: Persist, map, and process inventory expiration calculations using absolute UTC timestamps internally.
- **DO NOT**: Use relative local target device timezone offsets, which cause data processing errors if the system location parameters change.

### [DOM-03] Metric Calculations

- **Target Scope**: Core Engine
- **Severity**: WARNING
- **DO**: Code expiration windows (such as the 30-day proximity alert metric) using dynamic calculation rules that respect variable month lengths.
- **DO NOT**: hardcode time duration math as static millisecond numbers (e.g., calculating a month as a rigid static block of 2,592,000,000 milliseconds).

---

## 6. Documentation & Code Maintenance [DOC]

### [DOC-01] Rationale Documentation

- **Target Scope**: Public APIs, Queries
- **Severity**: STYLE
- **DO**: Document the specific operational intent ("why", not just "what") for every public interface method, algorithmic formula, and database query strategy.
- **DO NOT**: Submit functional code changes containing complex edge cases or custom logic loops without inline documentation.

### [DOC-02] Authoritative Reference Constraints

- **Target Scope**: Inline Comments, KDoc
- **Severity**: WARNING
- **DO**: Reference architectural decision records (ADRs) when code documentation needs to cite an external design rationale. ADRs represent solidified, agreed-upon decisions.
- **DO NOT**: Reference planning checklist items, review checklist items, or any other transient planning artifact from `docs/` in code documentation. These artifacts are directional and less solid than ADRs.

---

## 7. Code Formatting & Readability [FMT]

### [FMT-01] Method Orchestration

- **Target Scope**: All Functions
- **Severity**: STYLE
- **DO**: Write high-level coordinating methods that handle application flow by cleanly dispatching explicit operations down to descriptive sub-routines.
- **DO NOT**: Pack dense procedural lines or mixed levels of abstraction directly inside a single primary controller method.

### [FMT-02] Layout Linearity

- **Target Scope**: All Source Files
- **Severity**: STYLE
- **DO**: Arrange file code in a strictly linear, top-down reading pattern. High-level coordinating entry points must sit at the top, followed progressively by intermediate sub-routines, and low-level details at the bottom.
- **DO NOT**: Scatter minor utilities or target helper methods randomly throughout the file.

---

## 8. Design Cleanness, Correctness, & BDD Testing [TST]

### [TST-01] Behavioral Verification

- **Target Scope**: Core Logic, ViewModels
- **Severity**: CRITICAL
- **DO**: Verify every non-trivial domain calculation, database operation, state transition, and processing edge case with automated unit tests.
- **DO NOT**: Ship application updates relying on manually verified feature sets.

### [TST-02] Hierarchical Structure

- **Target Scope**: `*Test.kt`
- **Severity**: WARNING
- **DO**: Model test architecture to mirror a clean Behavior-Driven Development (BDD) paradigm using JUnit 5 `@Nested` classes. Organize tests structurally following a logical hierarchy: `Target Method -> Specific Behavioral Scenario -> Expected Outcome Condition`.

  Example Structure:

  ```kotlin
  class InventoryEngineTests {
      @Nested
      inner class ProcessStateTransition {
          @Nested
          inner class GivenItemIsCurrentlyInUse {
              @Test
              fun `should transition status to Empty when inventory count falls to zero`() {
                  // Test code goes here
              }
          }
      }
  }
  ```

- **DO NOT**: Structure test classes as a flat, unorganized list of independent testing statements.

### [TST-03] Functional Naming Conventions

- **Target Scope**: `*Test.kt`
- **Severity**: STYLE
- **DO**: Write clear, descriptive test names using backticked string signatures to clearly explain the scenario and expected outcome.
- **DO NOT**: Use ambiguous, cryptic names (such as `test1()` or `checkUpdate()`) for test verification methods.
