---
name: code-critic
description: >
  Perform a focused, opinionated code review on a specific file, component, or directory. Use this skill
  whenever the user asks to review, critique, audit, analyze, or improve code quality — even if they
  phrase it casually ("take a look at this", "what do you think of this code", "is this well written",
  "review my service", "check my component"). Trigger on: "code review", "critique this", "review this
  file/directory/component", "audit my code", "improve this code", "is this good code". Always prefer
  this skill over ad-hoc review when the user has a specific file or scope in mind.
---

# Code Critic Skill

A narrowly-scoped, opinionated code review skill that critiques code against SWE best practices,
produces a structured summary of findings, and enables the user to interactively review and apply changes.

---

## Step 1 — Establish Scope

Before reading any code, pin down the review scope. If the user has already provided a file path,
directory, or pasted code, skip straight to Step 2.

Otherwise, ask:
- **What to review?** — a file path, a directory, a pasted snippet, or an uploaded file
- **Any focus areas?** — performance, security, readability, architecture, test coverage, etc. (optional;
  if not given, apply the full checklist from Step 2)
- **Any project-specific conventions?** — coding standards, frameworks, patterns in use (optional;
  if not given, apply universal best practices)

Collect all of this before reading code. One ask, not multiple rounds.

---

## Step 2 — Read the Code

### For a single file or snippet
Read it fully. Note: file length, language, apparent purpose, and any imports/dependencies visible.

### For a directory
```bash
find <dir> -type f | grep -E '\.(ts|tsx|js|jsx|py|go|rs|java|cs|rb|swift|kt)$' | head -60
```
Identify the key files. Prioritise:
1. Entry points / main modules
2. Core business logic
3. Shared utilities / helpers
4. Config and types

Read the highest-impact files. You don't need to read every file — be selective and explicit about
what you skipped and why.

### What to note while reading
- What the code is doing (build a mental model before critiquing)
- Smells that surface immediately
- Anything that requires a second pass

---

## Step 3 — Evaluate Against Best Practices

Apply the universal checklist below. **You do not need to enumerate every item** — only raise issues
that are actually present. Skip categories with no findings.

### Quick-reference checklist

**Correctness & Safety**
- [ ] Off-by-one errors, null/undefined dereferences, unhandled edge cases
- [ ] Race conditions, unsafe concurrency patterns
- [ ] Injection vectors (SQL, shell, XSS), hardcoded secrets, overly broad permissions

**Readability & Maintainability**
- [ ] Naming: variables, functions, types should be intention-revealing
- [ ] Function / method length and single-responsibility
- [ ] Magic numbers / strings — should be named constants
- [ ] Deeply nested logic — should be flattened or extracted
- [ ] Dead code, commented-out code, TODO debt

**Architecture & Design**
- [ ] Separation of concerns — UI, business logic, data access
- [ ] Dependency direction — no circular deps, no leaky abstractions
- [ ] Duplication (DRY) — but not over-abstracted (WET is sometimes fine)
- [ ] Appropriate use of language idioms and standard library

**Error Handling**
- [ ] Errors caught and handled at the right level
- [ ] No silent swallowing of errors (`catch {}`, `_ = err`)
- [ ] User-facing messages don't leak internals

**Performance (flag only if likely to matter)**
- [ ] N+1 queries, unbounded loops over large datasets
- [ ] Unnecessary re-computation in hot paths
- [ ] Missing indexes, missing caching for expensive ops

**Testability & Tests (if tests exist or are expected)**
- [ ] Functions are unit-testable (no hidden side effects, injectable deps)
- [ ] Test coverage of happy path, edge cases, error cases
- [ ] Tests assert on behaviour, not implementation details

---

## Step 4 — Produce the Review

Structure your output as follows. Keep it scannable — the user should be able to triage quickly.

---

### 📋 Review Summary

> 2–4 sentence overall assessment: what the code does well, what the main concerns are, and the
> severity profile (e.g., "mostly style issues with one correctness concern" vs "significant
> architectural problems").

**Scope reviewed:** `<files or paths>`
**Lines reviewed:** ~N
**Severity breakdown:** 🔴 Critical: N | 🟠 Major: N | 🟡 Minor: N | 💡 Suggestion: N

---

### 🔍 Findings

For each finding, use this format:

```
#### [SEVERITY] Short title  (e.g., `src/api/user.ts:42`)

**What:** One sentence describing the issue.
**Why it matters:** One sentence on the risk or impact.
**Recommended fix:**
\`\`\`<lang>
// before
<problematic code>

// after
<improved code>
\`\`\`
**Effort:** [Trivial | Low | Medium | High]
```

Severity levels:
- 🔴 **Critical** — correctness bugs, security vulnerabilities, data loss risk
- 🟠 **Major** — design issues, significant maintainability debt, missing error handling
- 🟡 **Minor** — style, naming, small clarity improvements
- 💡 **Suggestion** — optional improvements, nice-to-haves

Group findings by severity (Critical first).

---

### ✅ What's Working Well

Brief bullets on genuine strengths. Don't pad this — only include real positives.

---

### 🗺️ Recommended Priority Order

Numbered list of the top changes to make, ordered by impact/effort ratio. This is the
actionable takeaway the user should act on first.

---

## Step 5 — Invite Interaction

After delivering the review, offer these explicit next steps:

> **What would you like to do next?**
> - Ask me to **implement any of these fixes** directly
> - **Go deeper** on a specific finding
> - **Expand the scope** to related files
> - **Rewrite a section** with the fixes applied
> - **Explain the reasoning** behind a recommendation

Wait for the user's direction. Don't preemptively apply changes unless asked.

---

## Tone & Style Guidelines

- Be direct and specific. Vague feedback ("this could be cleaner") is useless.
- Always show the fix, not just the problem.
- Don't nitpick trivially when bigger issues exist — prioritise ruthlessly.
- Acknowledge context: a quick script has different standards than production infrastructure.
- Don't moralize. One clear statement of the problem is enough — no lecturing.
- If the code is genuinely good, say so. Don't manufacture issues.
