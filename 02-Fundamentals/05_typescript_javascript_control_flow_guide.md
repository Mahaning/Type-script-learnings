# 🔀 The Ultimate Guide to Conditional & Control Flow Statements in TypeScript & JavaScript

Conditional statements direct execution flow by evaluating logical expressions to determine which branch of code to execute.

---

## 📑 Table of Contents

1. [Definition & Core Concepts](#1-definition--core-concepts)
2. [Internal Working & Engine Execution](#2-internal-working--engine-execution)
3. [`if`, `else if`, and `else` Statements](#3-if-else-if-and-else-statements)
4. [The Ternary Operator (`? :`)](#4-the-ternary-operator---)
5. [The `switch` Statement](#5-the-switch-statement)
6. [Short-Circuit Evaluation (`&&`, `||`, `??`)](#6-short-circuit-evaluation---)
7. [Type-Level Conditionals & Guarding (TypeScript)](#7-type-level-conditionals--guarding-typescript)
8. [Summary & Best Practices Matrix](#8-summary--best-practices-matrix)

---

## 1. Definition & Core Concepts

Conditional statements evaluate boolean expressions to branch execution paths.

```
                     ┌────────────────────────┐
                     │ Evaluate Expression   │
                     └───────────┬────────────┘
                                 │
                   ┌─────────────┴─────────────┐
             Truthy│                           │Falsy
                   ▼                           ▼
        ┌────────────────────┐      ┌────────────────────┐
        │ Execute Branch A   │      │ Execute Branch B   │
        └────────────────────┘      └────────────────────┘
```

In JavaScript and TypeScript, values are implicitly coerced to boolean truthiness during conditional evaluations. Falsy values include:

* `false`
* `0`, `-0`, `0n` (BigInt zero)
* `""` (Empty string)
* `null`
* `undefined`
* `NaN`

*All other values—including empty arrays `[]` and empty objects `{}`—are truthy.*

---

## 2. Internal Working & Engine Execution

At the V8 engine / bytecode level, conditional statements are converted into **conditional branching instructions** or **jump tables**:

### A. Linear Evaluation (`if-else`)
Evaluates conditions sequentially using jump instructions (e.g., `JumpIfFalse`). In long `if-else` chains, performance scales linearly with $O(N)$ conditions checked.

```
Bytecode Flow for if (x > 10):
1. Compare x with 10         ---> Output: Boolean Flag Register
2. JumpIfFalse Offset [Target Block B]
3. [Execute Block A]
4. Jump [End of Statement]
5. Target Block B: [Execute Block B]
```

### B. Jump Table Optimization (`switch`)
When a `switch` statement evaluates a dense set of integer or string values, modern engines optimize execution from $O(N)$ linear checks down to an **$O(1)$ constant-time Jump Table** or binary search tree.

```
       Switch Value (e.g., Status Code 2)
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
     [ Jump Table Index ] ──► [ Address Target 2 ] ──► Direct Jump to Code Block
```

---

## 3. `if`, `else if`, and `else` Statements

### Definition
The fundamental control flow structure that executes a block of code if its condition evaluates to a truthy value.

### Use Case
Ideal for dynamic ranges, multiple unrelated conditions, or non-discrete boolean evaluations.

### Internal Working
Evaluates expressions sequentially from top to bottom. As soon as one branch evaluates to truthy, its block executes, and execution jumps directly to the end of the `if-else` chain.

### Code Example

```typescript
interface UserSession {
  isLoggedIn: boolean;
  role: "admin" | "editor" | "subscriber";
  loginAttempts: number;
}

function determineAccessLevel(user: UserSession): string {
  // 1. Primary Security Guard
  if (!user.isLoggedIn) {
    return "ACCESS_DENIED: User must be authenticated.";
  } 
  
  // 2. Account Lockout Check
  if (user.loginAttempts >= 5) {
    return "ACCOUNT_LOCKED: Too many failed attempts.";
  }

  // 3. Role-Based Hierarchy (Sequential Evaluation)
  if (user.role === "admin") {
    return "FULL_ACCESS: System control granted.";
  } else if (user.role === "editor") {
    return "LIMITED_ACCESS: Content editing granted.";
  } else {
    return "READ_ONLY: Basic view access.";
  }
}

// Example Execution
console.log(determineAccessLevel({ isLoggedIn: true, role: "editor", loginAttempts: 0 }));
// Output: "LIMITED_ACCESS: Content editing granted."
```

---

## 4. The Ternary Operator (`? :`)

### Definition
An inline conditional operator that accepts three operands: a condition, an expression to execute if truthy, and an expression to execute if falsy.

### Use Case
Ideal for inline assignments, JSX/TSX view templates, and returning values without verbose `if` blocks.

### Internal Working
Evaluates the condition once, pushing the evaluated value of the chosen expression onto the stack. *Both branches are not evaluated*, ensuring sub-expressions with side-effects in the unused branch are skipped.

### Code Example

```typescript
type SubscriptionPlan = "free" | "pro" | "enterprise";

interface UserProfile {
  name: string;
  plan: SubscriptionPlan;
  apiCredits: number;
}

function getRateLimit(user: UserProfile): number {
  // Inline Ternary Assignment
  const baseLimit = user.plan === "enterprise" ? 10000 : 1000;

  // Nested Ternary for Multi-State Evaluation
  const maxConcurrency = user.plan === "enterprise" 
    ? 50 
    : user.plan === "pro" 
      ? 10 
      : 2;

  return baseLimit + user.apiCredits;
}

// Example Execution
const proUser: UserProfile = { name: "Alex", plan: "pro", apiCredits: 250 };
console.log(`Rate Limit: ${getRateLimit(proUser)}`); 
// Output: "Rate Limit: 1250"
```

---

## 5. The `switch` Statement

### Definition
Evaluates an expression against multiple matching `case` clauses, executing statements associated with the first matching case using **strict equality (`===`)**.

### Use Case
Best used when comparing a single value against multiple fixed, discrete values (like enums or string unions).

### Internal Working
Evaluates the key expression once and matches its result sequentially or via an internal lookup/jump table against `case` values using strict equality (`===`). A `break` statement exits the block; omitting `break` results in **fallthrough execution**, running subsequent case blocks regardless of match status.

### Code Example

```typescript
enum HttpStatusCode {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  Unauthorized = 401,
  NotFound = 404,
  InternalServerError = 500,
}

function handleResponse(status: HttpStatusCode): string {
  switch (status) {
    case HttpStatusCode.OK:
    case HttpStatusCode.Created: 
      // Shared fallthrough logic for 20x Success Codes
      return "SUCCESS: Action processed successfully.";

    case HttpStatusCode.BadRequest:
      return "CLIENT_ERROR: Invalid parameters provided.";

    case HttpStatusCode.Unauthorized:
      return "SECURITY_ERROR: Missing authentication token.";

    case HttpStatusCode.NotFound:
      return "RESOURCE_ERROR: Requested entity does not exist.";

    case HttpStatusCode.InternalServerError:
      return "SERVER_ERROR: An unexpected error occurred on our end.";

    default:
      // Exhaustive checking for unknown codes
      return `UNKNOWN_RESPONSE: Handled status ${status}`;
  }
}

// Example Execution
console.log(handleResponse(HttpStatusCode.Created));
// Output: "SUCCESS: Action processed successfully."
```

---

## 6. Short-Circuit Evaluation (`&&`, `||`, `??`)

### Definition
Leverages logical operators to evaluate or assign expressions conditionally based on truthiness or nullish values.

### Use Case
Ideal for default values, optional function calls, and concise guard clauses.

### Short-Circuit Execution Rules
* **Logical AND (`&&`):** Evaluates from left to right; returns the **first falsy value** encountered, or the last value if all are truthy.
* **Logical OR (`||`):** Evaluates from left to right; returns the **first truthy value** encountered, or the last value if all are falsy.
* **Nullish Coalescing (`??`):** Returns the right-hand value **only if** the left-hand value is `null` or `undefined`.

### Code Example

```typescript
interface AppConfig {
  title?: string;
  timeout?: number;
  enableLogs?: boolean;
}

function initializeApp(config: AppConfig) {
  // 1. Short-circuit execution for function calls (Logical AND)
  config.enableLogs && console.log("Initializing application modules...");

  // 2. Logical OR (||) fallback vs Nullish Coalescing (??)
  // BAD: || treats 0 as falsy, overwriting valid 0-second timeouts with 5000
  const badTimeout = config.timeout || 5000;

  // GOOD: ?? preserves valid 0 or false, triggering ONLY on null/undefined
  const safeTimeout = config.timeout ?? 5000;
  const appTitle = config.title ?? "Untitled Application";

  return { appTitle, safeTimeout };
}

// Example Execution with 0 as explicit parameter
console.log(initializeApp({ timeout: 0, enableLogs: true }));
// Logs: "Initializing application modules..."
// Output: { appTitle: "Untitled Application", safeTimeout: 0 }
```

---

## 7. Type-Level Conditionals & Guarding (TypeScript)

### Definition
TypeScript enhances runtime conditionals with **Type Narrowing** (using `typeof`, `instanceof`, `in`, or custom type predicates) and introduces compile-time **Conditional Types**.

### Use Case
Ensuring compile-time type safety when working with union types or polymorphic function signatures.

### Code Example

```typescript
// --- A. Runtime Type Guarding & Narrowing ---
type MediaItem = 
  | { kind: "image"; url: string; resolution: [number, number] }
  | { kind: "video"; url: string; durationSeconds: number };

function processMedia(item: MediaItem): string {
  // Discriminated Union Control Flow
  switch (item.kind) {
    case "image":
      // TypeScript automatically narrows 'item' to image variant here
      return `Image [${item.resolution[0]}x${item.resolution[1]}] loaded from ${item.url}`;
    case "video":
      // TypeScript automatically narrows 'item' to video variant here
      return `Video (${item.durationSeconds}s) streaming from ${item.url}`;
  }
}

// --- B. Type-Level Conditional Types (Compile-time) ---
type IsString<T> = T extends string ? "Yes" : "No";

type Test1 = IsString<string>; // "Yes"
type Test2 = IsString<number>; // "No"
```

---

## 8. Summary & Best Practices Matrix

| Conditional | Best Used For | Time Complexity | Readability Impact |
| :--- | :--- | :--- | :--- |
| **`if / else`** | Range checks, complex logical conditions, guard clauses | $O(N)$ linear evaluation | High for small checks; degraded by deep nesting |
| **`Ternary (? :)`** | Simple 2-way branch assignments, inline JSX/TSX expressions | $O(1)$ evaluation | High for single-level; poor when deeply nested |
| **`switch`** | Single discrete value matching against 3+ fixed cases | $O(1)$ to $O(N)$ (optimized via Jump Tables) | Excellent for discrete unions and enums |
| **`Logical (&&, \|\|, ??)`** | Guarding execution, establishing fallbacks, default assignment | $O(1)$ short-circuit | High when used simply; obscure if overused |
| **Type Guards** | Narrowing TypeScript union types safely at runtime | $O(1)$ evaluation | Crucial for enterprise TypeScript safety |

### Key Guidelines
1. **Prefer Guard Clauses:** Return early from functions instead of deeply nesting `if-else` blocks.
2. **Use `??` Over `||` For Defaults:** Avoid bug patterns where valid falsy numbers (`0`), booleans (`false`), or strings (`""`) get overwritten.
3. **Keep Ternaries Shallow:** Avoid nesting ternary expressions past 1 level deep; refactor to a `switch` or `if-else` block instead.