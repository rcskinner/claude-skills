# Best Practices Reference

This file elaborates on the checklist items in SKILL.md. Use it to guide your analysis — not to
copy-paste into reviews. Read the relevant sections for the domain you're reviewing.

---

## Table of Contents

1. [Correctness & Safety](#correctness--safety)
2. [Naming & Readability](#naming--readability)
3. [Functions & Methods](#functions--methods)
4. [Architecture & Design](#architecture--design)
5. [Error Handling](#error-handling)
6. [Concurrency](#concurrency)
7. [Security](#security)
8. [Performance](#performance)
9. [Testing](#testing)
10. [Language-Specific Patterns](#language-specific-patterns)

---

## Correctness & Safety

### Null / undefined handling
- Any dereference of a potentially null/undefined value without a guard is a bug waiting to happen.
- Optional chaining (`?.`) and nullish coalescing (`??`) are not always sufficient — sometimes a
  null value signals a logic error that should be surfaced, not silently swallowed.
- In typed languages: avoid `!` (non-null assertion) unless you have ironclad proof the value is set.

### Off-by-one errors
- Loop bounds: `< length` vs `<= length`, `>= 0` vs `> 0`
- Slice/substring: check whether end index is exclusive or inclusive in the language
- Pagination: off-by-one in page/offset calculation leads to skipped or duplicated records

### Unhandled edge cases
- Empty collections: does the code behave correctly on `[]` or `{}`?
- Single-element collections: does code that assumes "at least two items" break?
- Extreme values: `0`, `-1`, `INT_MAX`, empty strings, very long strings
- Concurrent modification: iterating a collection while mutating it

---

## Naming & Readability

### Variable names
- Boolean variables: use `is`, `has`, `should`, `can` prefix (`isLoading`, `hasError`)
- Collections: plural (`users`, not `user`); maps: `userById`, `configByKey`
- Avoid single-letter names except for obvious loop counters (`i`, `j`) or coordinates (`x`, `y`)
- Avoid abbreviations unless domain-standard (`dto`, `http`, `id`, `ctx`)
- Avoid noise words: `data`, `info`, `temp`, `obj`, `result` without qualification

### Function names
- Verb + noun: `fetchUser`, `validateEmail`, `buildPayload`
- Boolean-returning functions: `is`, `has`, `can` prefix
- Avoid overloaded names: `process`, `handle`, `manage`, `do` — be specific about what

### Type / class names
- Noun phrases: `UserRepository`, `PaymentProcessor`
- Avoid `Manager`, `Handler`, `Helper`, `Util` — these signal a grab-bag class
- Interfaces: noun (what it is), not verb-gerund unless the interface represents a capability
  (`Serializable`, `Comparable` — but `UserRepository` not `UserRepositoryInterface`)

### Magic numbers and strings
- Any literal number other than `0` and `1` should be a named constant
- String literals used as identifiers, status codes, or keys should be enums or constants
- Exception: obvious mathematical formulas where the literal is self-explanatory

### Comments
- Good comments explain *why*, not *what* — the code shows what, comments show intent
- Avoid restating the code in English (`// increment i` above `i++`)
- Prefer self-documenting code over comments where possible
- TODOs are fine; TODOs with no owner or ticket are debt — flag them

---

## Functions & Methods

### Single Responsibility Principle
- A function should do one thing. If you need "and" to describe it, it does too much.
- A function longer than ~40 lines is a candidate for extraction (context-dependent)
- A function with more than ~4 parameters is hard to call correctly — consider a config object

### Pure functions preferred
- Functions without side effects are easier to test, reason about, and compose
- Prefer returning a value over mutating an argument
- Where mutation is necessary, make it explicit in the name or signature

### Early returns
- Guard clauses at the top of a function reduce nesting and make the happy path clear
- Prefer `if (invalid) return/throw` at the top over `if (valid) { ... }` wrapping the whole body

### Function length and complexity
- Cyclomatic complexity > 10 is a warning sign (too many branches)
- Deeply nested conditionals (> 3 levels) should be refactored
- Strategies: extract method, early return, polymorphism, lookup table

---

## Architecture & Design

### Separation of Concerns
- UI components should not contain business logic or direct data-fetching
- Business logic should not know about HTTP, databases, or UI
- Data access (queries, ORM) should be isolated from business logic

### Dependency Inversion
- High-level modules should depend on abstractions, not concretions
- Inject dependencies rather than instantiating them inside functions/classes
- Testability is a good proxy: if you can't test the unit without spinning up a DB, it's too coupled

### DRY vs WET
- Don't repeat yourself — but only when the duplication represents the same *concept*
- Incidental duplication (two things that happen to look alike) should sometimes stay duplicated
- Over-abstraction ("wrong abstraction") is worse than duplication — don't abstract prematurely
- Rule of three: abstract on the third duplication, not the second

### Modularity
- Modules should have high cohesion (related things together) and low coupling (few dependencies)
- Circular dependencies are always a design smell — resolve by extracting a shared module or
  inverting the dependency

### API design (for functions, classes, REST, etc.)
- Public interfaces should be minimal — expose only what consumers need
- Principle of least surprise: behaviour should match the name and common intuition
- Once a public API is used externally, changing it is a breaking change — be conservative early

---

## Error Handling

### Handle errors at the right level
- Errors should be caught by code that knows how to respond to them
- Don't catch errors just to log and re-throw — let them propagate unless you add value
- Don't catch `Exception`/`Error` at the top level and silently continue — crash or recover

### Never silently swallow errors
- `catch (_) {}`, `except: pass`, `_ = err` — these hide bugs
- At minimum, log. Preferably, propagate or convert to a domain error.

### Error types and messages
- Use typed/structured errors when you need to distinguish error types programmatically
- Error messages for developers: include context (what you were doing, what went wrong)
- Error messages for users: no stack traces, no internal paths, no system details

### Failing fast
- Validate inputs at the boundary (API handler, CLI entrypoint, constructor)
- Surface invalid state immediately rather than letting it propagate and cause confusing failures later

---

## Concurrency

### Race conditions
- Shared mutable state accessed from multiple goroutines/threads/async contexts without
  synchronisation is a race condition
- Use locks, channels, atomic operations, or immutable data as appropriate
- In JavaScript/TypeScript: sequential `await` vs `Promise.all` — understand which you need

### Async patterns (JS/TS)
- `await` inside a loop (sequential) vs `Promise.all` (parallel) — use parallel when order doesn't matter
- Never `await` a void-returning async function and ignore the promise — errors will be swallowed
- Unhandled promise rejections crash Node processes in modern versions

### Deadlocks
- Any code that acquires multiple locks must do so in a consistent order
- Long-held locks blocking fast paths

---

## Security

### Injection
- Any user input interpolated into SQL, shell commands, HTML, or eval is an injection risk
- Parameterised queries / prepared statements for SQL — always
- `dangerouslySetInnerHTML`, `innerHTML =`, `eval()` — always suspect

### Secrets
- Hardcoded API keys, passwords, tokens — always flag these as Critical
- Secrets should come from environment variables or a secrets manager, never committed to source

### Authentication & Authorisation
- Auth checks belong in middleware/guard layers, not scattered in business logic
- Verify ownership before accessing resources (e.g., ensure `resource.userId === currentUser.id`)
- Don't trust client-supplied IDs for sensitive operations without re-fetching from DB

### Input validation
- Validate and sanitise at every trust boundary (API surface, file uploads, user-provided URLs)
- Be explicit about accepted types, formats, and lengths

---

## Performance

> Only flag performance issues when they are likely to matter at realistic scale.
> Micro-optimisations in non-hot-paths are noise.

### Database
- N+1 query pattern: fetching N records then querying for each one separately in a loop
- Missing indexes on columns used in WHERE, JOIN, ORDER BY with large tables
- Fetching entire rows when only a few columns are needed
- Large transactions holding locks longer than necessary

### Memory
- Accumulating large collections in memory when streaming is possible
- Unbounded caches with no eviction policy

### Computation
- Expensive operations inside tight loops (regex compilation, object construction, sorting)
- Redundant work: computing the same derived value multiple times

### Frontend-specific
- Unnecessary re-renders (missing memoisation for expensive components)
- Fetching full datasets when pagination or lazy loading is appropriate
- Large bundle size from importing entire libraries when only a few functions are needed

---

## Testing

### Coverage priorities
- Test the happy path, at least one error path, and edge cases (empty, null, max)
- Prioritise testing business logic and integration points over trivial getters/setters

### Test quality
- Tests should assert on observable behaviour, not internal implementation details
- Test names should describe the scenario and expected outcome
- Avoid testing the same thing twice in different tests (increases maintenance burden)
- Flaky tests (intermittent failures) are worse than no tests — they erode trust in the suite

### Testability
- If a function is hard to test, it's usually a design problem, not a test problem
- Injectable dependencies (not hardcoded singletons) make unit testing possible
- Pure functions are trivially testable — prefer them

### Test doubles
- Mocks, stubs, fakes: use the simplest one that lets you test the behaviour
- Avoid over-mocking — tests that mock everything don't test integration behaviour

---

## Language-Specific Patterns

### TypeScript / JavaScript
- Prefer `const` over `let`; avoid `var` entirely
- Avoid `any` — use `unknown` and type-narrow, or model the type properly
- Use strict null checks — `strictNullChecks: true` in tsconfig
- `==` vs `===` — always use `===` in JS
- Optional chaining (`?.`) and nullish coalescing (`??`) over manual null checks
- Array methods (`map`, `filter`, `reduce`) over imperative loops for transformations
- Async/await over raw Promise chains for readability

### Python
- Use type hints (Python 3.5+) for function signatures in any non-trivial code
- Prefer list/dict comprehensions over explicit loops for simple transformations
- Use `dataclasses` or `pydantic` models over plain dicts for structured data
- Context managers (`with`) for resource management — files, locks, DB connections
- Avoid mutable default arguments (`def f(lst=[]):`) — use `None` and initialise inside

### Go
- Errors are values — check them, don't ignore them (`_ = err` is almost always wrong)
- Defer for cleanup — always `defer` close/unlock near the open/lock
- Goroutine leaks: every goroutine needs a clear exit path; context cancellation propagation
- Prefer small interfaces (1–3 methods); accept interfaces, return structs
- Use `errors.Is` / `errors.As` for error comparison, not string matching

### Java / Kotlin
- Prefer immutable types; use `final` fields, `val` in Kotlin
- Checked vs unchecked exceptions — don't swallow checked exceptions by converting to RuntimeException
- Use streams for collection operations in modern Java
- Kotlin: prefer data classes, sealed classes, null-safety operators

### Rust
- `unwrap()` and `expect()` in non-test code are red flags — use `?` or proper error handling
- Unnecessary cloning — often signals a lifetime or ownership design issue
- Prefer `impl Trait` return types over `Box<dyn Trait>` when possible
