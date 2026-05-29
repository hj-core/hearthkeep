# HearthKeep: Phase-by-Phase Development Planning Checklist

## Instruction to the Planning LLM

You will generate a directional, high-level engineering plan for a specific development phase. First, read this checklist completely. Every phase plan you generate must strictly address the requirements under these sections. For your output, structure the phase plan around these rules and explicitly cite the Rule IDs (e.g., `[SCOPE-01]`, `[EXEC-02]`) to prove compliance.

---

## 1. Scope Boundaries & Phase Milestones [SCOPE]

### [SCOPE-01] Value-Driven Boundary Definition

- **Target Scope**: Phase Goal Setting
- **Severity**: CRITICAL
- **DO**: Define exactly one primary user-facing value statement or functional milestone that this phase delivers (e.g., "Offline database storage of category structures is fully functional").
- **DO NOT**: Allow multi-objective phase drift that combines unrelated features, causing fragmented progress.

### [SCOPE-02] Explicit Out-of-Scope Exclusions

- **Target Scope**: Scope Control
- **Severity**: CRITICAL
- **DO**: List at least three specific related items, features, or design details that are explicitly deferred to later phases to preserve development momentum.
- **DO NOT**: Leave boundaries open-ended, allowing "scope creep" to subtly inflate the task list mid-phase.

---

## 2. System Architecture & Integration Points [PLAN-ARCH]

### [PLAN-ARCH-01] Data Flow Mapping

- **Target Scope**: Data Design
- **Severity**: CRITICAL
- **DO**: Explicitly sketch the directional flow of data through the architecture layers (e.g., `SQLite Database → Room DAO → Repository → ViewModel StateFlow → Compose UI`).
- **DO NOT**: Design isolated components without mapping how state changes flow between them in real-time.

### [PLAN-ARCH-02] Layer Intersect Analysis

- **Target Scope**: Structural Safety
- **Severity**: WARNING
- **DO**: Identify exactly which shared `commonMain` structures, models, or configurations will be touched, updated, or extended during this phase.
- **DO NOT**: Write code modifications directly into shared core modules without analyzing how they affect existing modules or platform targets.

---

## 3. Risk Assessment & Technical Spikes [RISK]

### [RISK-01] Spike Isolation

- **Target Scope**: Risk Mitigation
- **Severity**: CRITICAL
- **DO**: Identify unproven frameworks, APIs, or libraries (e.g., `WorkManager` background threading bounds) and schedule a time-boxed engineering "spike" (research task) before writing production code.
- **DO NOT**: Attempt to build production-grade features on top of untried libraries directly during primary feature sprint loops.

### [RISK-02] Mitigation Strategy Mapping

- **Target Scope**: Fallback Planning
- **Severity**: WARNING
- **DO**: Document a clear fallback plan for any high-risk integration dependencies (such as using mock file-system storage if DB locking issues arise).
- **DO NOT**: Proceed into active development assuming zero configuration, compilation, or synchronization bottlenecks.

---

## 4. Incremental Execution Sequence [EXEC]

### [EXEC-01] Top-Down Engineering Sequence

- **Target Scope**: Implementation Sequence
- **Severity**: STYLE
- **DO**: Plan implementation steps using a logical sequence that begins with foundational structures before progressing to outer UI shells:
  1. Database Schemas, Tables, and Entities.
  2. Data Access Objects (DAOs) and Transaction methods.
  3. Business Repositories and local state caching flows.
  4. ViewModels, dynamic transforms, and validation logic.
  5. UI Views, layouts, visual states, and design elements.
- **DO NOT**: Build graphical user interfaces before defining, verifying, and testing the underlying databases, domains, or mock repositories.

### [EXEC-02] Atomic Task Chunking

- **Target Scope**: Task Breakdown
- **Severity**: WARNING
- **DO**: Breakdown the phase execution roadmap into specific task units designed to represent a single, compile-safe, isolated functional change.
- **DO NOT**: Outline sprawling, multi-day engineering blocks that combine data modeling, logic, and interface rendering into a single, massive step.

---

## 5. Verification & Quality Assurance Strategy [PLAN-QA]

### [PLAN-QA-01] BDD Behavioral Test Matrix

- **Target Scope**: Testing Plan
- **Severity**: CRITICAL
- **DO**: List the precise behavior patterns and edge cases that will be covered by automated unit tests at the end of the phase (e.g., "empty fields do not crash state transition calculations").
- **DO NOT**: Defer testing strategies to the end of the project or rely purely on manual runtime verification.

### [PLAN-QA-02] Verification Sandbox Strategies

- **Target Scope**: Manual Review
- **Severity**: STYLE
- **DO**: Define a lightweight manual validation checklist (such as mock database values or print logging points) to verify that background flows behave properly before implementing complex layouts.
- **DO NOT**: Write extensive UI components solely to verify basic database state queries.
