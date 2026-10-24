# 🧬 The Ultimate Guide to Data Types in TypeScript & JavaScript

Data types define the nature, size, and behavioral rules of values stored in memory. They dictate what operations can be legally performed on a variable and how the engine processes the underlying binary data.

---

## 📑 Table of Contents
1. [Definition & Introduction](#1-definition--introduction)
2. [Rules of Usage & Type Discipline](#2-rules-of-usage--type-discipline)
3. [Dynamic Typing (JS) vs. Static Typing (TS)](#3-dynamic-typing-js-vs-static-typing-ts)
4. [Type Safety: The Compile-Time Shield](#4-type-safety-the-compile-time-shield)
5. [Type Annotations vs. Interfaces](#5-type-annotations-vs-interfaces)
6. [TypeScript Data Type Categories](#6-typescript-data-type-categories)
7. [Core Primitive Types](#7-core-primitive-types)
   - [Number](#a-number)
   - [String](#b-string)
   - [Boolean](#c-boolean)
   - [Null & Undefined](#d-null--undefined)
8. [Advanced & Special Types](#8-advanced--special-types)
   - [Union Types](#a-union-types)
   - [Void](#b-void)
   - [Function Types](#c-function-types)
9. [5 Real-Time Enterprise Examples](#9-5-real-time-enterprise-examples)

---

## 1. Definition & Introduction

In computer science, a **data type** is an attribute of data that tells the compiler or interpreter how the programmer intends to use the data.

```
       Raw Binary Data (01000001)
                   │
    ┌──────────────┴──────────────┐
    ▼                             ▼
Interpreter Context:        Interpreter Context:
"Number" ──► 65             "ASCII String" ──► 'A'
```

Without types, memory consists merely of contiguous streams of bits ($0$s and $1$s). Type systems assign meaning to these bit patterns, enabling runtime engines (V8, SpiderMonkey) and compilers (`tsc`) to enforce logical boundaries and optimize memory layout.

---

## 2. Rules of Usage & Type Discipline

When working with data types in TypeScript, adhere to these fundamental engineering rules:

1. **Explicit Intent over Implicit Assumptions:** Prefer explicit type declarations or leverage strong type inference rather than relying on structural ambiguity.
2. **Immutability First:** Treat value primitives as immutable values. Mutating properties on complex object references should be governed by strict structural contracts (`readonly`, `interface`, or `type`).
3. **Strict Null Checks:** Always keep `strictNullChecks: true` enabled in `tsconfig.json` to avoid the infamous "cannot read property of undefined" runtime errors.
4. **Avoid `any`:** Suppressing the type checker with `any` effectively strips TypeScript of its main value proposition, falling back to unchecked JavaScript execution.

---

## 3. Dynamic Typing (JS) vs. Static Typing (TS)

The fundamental difference between JavaScript and TypeScript lies in **when** and **how** types are verified.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Dynamic Typing (JavaScript)                                             │
│                                                                         │
│  Source Code ────► Runtime Execution ────► Type Error! (App Crashes)    │
│                     (V8 checks types)                                   │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│ Static Typing (TypeScript)                                              │
│                                                                         │
│  Source Code ────► TSC Compiler ────► Type Error! (Fails Build)         │
│                     (Checks types)                                      │
│                           │                                             │
│                           ▼ Passes                                      │
│                    Runtime Execution (Clean JS, Zero Overhead)          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Comparative Analysis

| Feature | JavaScript (Dynamic) | TypeScript (Static) |
| :--- | :--- | :--- |
| **Type Binding** | Bound to the **value** at runtime | Bound to the **identifier/variable** at compile-time |
| **Variable Re-assignment** | A variable can change its type freely during runtime | A variable retains its declared or inferred type |
| **Error Discovery** | Discovered at runtime (often by end users) | Discovered during development / compilation |
| **Performance Impact** | Engine performs continuous runtime checking & optimization | Zero runtime performance penalty (types erased during build) |
| **Refactoring** | Prone to hidden breakage | Safe, automated refactoring supported by language servers |

---

## 4. Type Safety: The Compile-Time Shield

**Type safety** is the extent to which a programming language prevents or discourages type errors. 

In JavaScript, operations like `1 + "2"` lead to implicit coercion (`"12"`), and calling a non-existent method (`user.getName()`) results in a runtime crash (`TypeError: user.getName is not a function`).

TypeScript enforces strict type safety by checking structural compatibility against defined type contracts before emitting JavaScript.

```typescript
// JavaScript (Silent runtime failure or unexpected behavior)
function calculateTotal(price, tax) {
  return price + tax; 
}
calculateTotal(100, "15"); // Returns "10015" (String concatenation bug!)

// TypeScript (Guaranteed Compile-Time Safety)
function calculateTotalTS(price: number, tax: number): number {
  return price + tax;
}
// calculateTotalTS(100, "15"); 
// ❌ Argument of type 'string' is not assignable to parameter of type 'number'.
```

---

## 5. Type Annotations vs. Interfaces

TypeScript offers two primary ways to define data structures and bind types: **Type Annotations / Type Aliases (`type`)** and **Interfaces (`interface`)**.

### Type Annotations & Type Aliases
Used to bind explicit primitives, unions, tuples, or object shapes directly.

```typescript
// Explicit Annotation
let score: number = 95;

// Type Alias
type Coordinates = {
  x: number;
  y: number;
};

type ID = string | number; // Union types are supported in aliases!
```

### Interfaces
Used primarily to describe the structure of objects, contracts, and OOP abstractions. Interfaces excel at declaration merging and class implementation.

```typescript
interface UserProfile {
  id: string;
  username: string;
  email: string;
  getFormattedName(): string;
}

// Declaration Merging (Interfaces can be extended across files)
interface UserProfile {
  avatarUrl?: string; // Merging new optional field into UserProfile
}
```

### Key Differences
- **Unions:** Type Aliases can model primitive unions (`string | number`); Interfaces cannot directly represent scalar unions.
- **Declaration Merging:** Interfaces automatically merge duplicate declarations; Type Aliases throw a duplicate identifier error.
- **Extensibility:** Interfaces extend other interfaces via `extends`; Type Aliases use intersection operators (`&`).

---

## 6. TypeScript Data Type Categories

TypeScript's type landscape can be categorized into four distinct layers:

```
                          ┌───────────────────────────┐
                          │         top type          │
                          │      (unknown / any)      │
                          └─────────────┬─────────────┘
                                        │
        ┌───────────────────────────────┴───────────────────────────────┐
        ▼                                                               ▼
┌───────────────────────────────┐               ┌───────────────────────────────┐
│        Primitive Types        │               │        Reference Types        │
├───────────────────────────────┤               ├───────────────────────────────┤
│ • number      • string        │               │ • Object       • Array        │
│ • boolean     • symbol        │               │ • Function     • Tuple        │
│ • null        • undefined     │               │ • Map / Set    • Class        │
│ • bigint                      │               │                               │
└───────────────────────────────┘               └───────────────────────────────┘
        │                                                               │
        └───────────────────────────────┬───────────────────────────────┘
                                        │
                          ┌─────────────▼─────────────┐
                          │        Bottom Type        │
                          │          (never)          │
                          └───────────────────────────┘
```

---

## 7. Core Primitive Types

Primitives are single immutable values stored on the call stack.

### A. Number
Represents all numeric values—both integers and floating-point numbers. Under the hood, JS/TS stores numbers using IEEE 754 double-precision floating-point format ($64$-bit).

```typescript
let count: number = 42;
let decimal: number = 3.14159;
let hex: number = 0xff; // 255
let binary: number = 0b1010; // 10
```

### B. String
Represents textual data encoded as UTF-16 code units.

```typescript
let firstName: string = "Jane";
let headline: string = 'Software Architect';

// Template Literal String (Interpolation)
let greeting: string = `Hello, ${firstName}! Role: ${headline}`;
```

### C. Boolean
Represents a logical entity with two states: `true` or `false`.

```typescript
let isProcessing: boolean = false;
let hasPermission: boolean = true;
```

### D. Null & Undefined
- `undefined`: Variable has been declared, but no value has been assigned yet (uninitialized).
- `null`: Explicit representation of "no value" or an intentionally empty object pointer.

```typescript
let unassigned: undefined = undefined;
let emptyPointer: null = null;

// With strictNullChecks enabled:
let title: string = "Hello";
// title = null; // ❌ Type 'null' is not assignable to type 'string'.
```

---

## 8. Advanced & Special Types

### A. Union Types
Allows a variable to hold values of multiple specified types using the pipe (`|`) operator.

```typescript
type Status = "pending" | "approved" | "rejected";
let currentStatus: Status = "pending";

// Union with primitives
type TransactionID = string | number;

function processTransaction(id: TransactionID) {
  if (typeof id === "string") {
    // Type Narrowing: TypeScript knows 'id' is a string here
    console.log(`Processing UUID: ${id.toUpperCase()}`);
  } else {
    // Type Narrowing: TypeScript knows 'id' is a number here
    console.log(`Processing Numeric ID: ${id.toFixed(0)}`);
  }
}
```

### B. Void
Represents the complete absence of a return value in functions.

```typescript
function logTelemetry(event: string): void {
  console.log(`Telemetry event dispatched: ${event}`);
  // No return statement
}
```

### C. Function Types
Functions can be typed using arrow function signatures or full call signatures.

```typescript
// Function Signature Type Alias
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (x, y) => x + y;
const multiply: MathOperation = (x, y) => x * y;

// Function parameter with callback type annotation
function executeComputation(
  a: number, 
  b: number, 
  operation: (res: number) => void
): void {
  const result = a * b;
  operation(result);
}
```

---

## 9. 5 Real-Time Enterprise Examples

Here are 5 real-world scenarios demonstrating best practices for leveraging TypeScript types in production architectures.

### Example 1: E-Commerce Payment Gateway State Machine
*Demonstrates Union Types, Literal Types, and Type Safeguards.*

```typescript
type PaymentMethod = "credit_card" | "paypal" | "crypto";

interface PaymentPending {
  status: "PENDING";
  transactionId: string;
}

interface PaymentSuccess {
  status: "SUCCESS";
  transactionId: string;
  amountPaid: number;
  paidAt: Date;
}

interface PaymentFailed {
  status: "FAILED";
  transactionId: string;
  errorMessage: string;
  errorCode: number;
}

// Discriminated Union for State Safety
type PaymentState = PaymentPending | PaymentSuccess | PaymentFailed;

function renderPaymentNotification(state: PaymentState): string {
  switch (state.status) {
    case "PENDING":
      return `Payment ${state.transactionId} is processing...`;
    case "SUCCESS":
      return `Successfully received $${state.amountPaid} on ${state.paidAt.toISOString()}`;
    case "FAILED":
      return `Error (${state.errorCode}): ${state.errorMessage}`;
  }
}
```

### Example 2: E-Commerce REST API Response Handling
*Demonstrates Interfaces, Generics, and Nullable Types.*

```typescript
interface ApiResponse<TData> {
  statusCode: number;
  success: boolean;
  timestamp: string;
  data: TData | null;
  error?: {
    code: string;
    details: string;
  };
}

interface Product {
  id: string;
  sku: string;
  price: number;
  inStock: boolean;
}

async function fetchProductCatalog(): Promise<ApiResponse<Product[]>> {
  try {
    const response = await fetch("https://api.store.com/v1/products");
    const products: Product[] = await response.json();

    return {
      statusCode: 200,
      success: true,
      timestamp: new Date().toISOString(),
      data: products,
    };
  } catch (err) {
    return {
      statusCode: 500,
      success: false,
      timestamp: new Date().toISOString(),
      data: null,
      error: {
        code: "FETCH_FAILED",
        details: err instanceof Error ? err.message : "Unknown error occurred",
      },
    };
  }
}
```

### Example 3: User Authentication & Feature Flagging System
*Demonstrates String Unions, Optional Properties, and Void Callbacks.*

```typescript
type UserRole = "SuperAdmin" | "Manager" | "Guest";

interface FeatureFlags {
  enableBetaDashboard: boolean;
  enableAIAnalytics?: boolean; // Optional flag
}

interface AppUser {
  readonly id: string;
  email: string;
  role: UserRole;
  flags: FeatureFlags;
  onLogout: () => void; // Void function type
}

function configureUserAccess(user: AppUser): void {
  if (user.role === "SuperAdmin" && user.flags.enableAIAnalytics) {
    console.log(`Granting AI access to ${user.email}`);
  } else {
    console.log(`Standard access granted for ${user.email}`);
  }
}
```

### Example 4: Real-Time WebSockets Event Listener Architecture
*Demonstrates Function Types, Unions, and Custom Callbacks.*

```typescript
type WebSocketEvent = "CONNECT" | "DISCONNECT" | "MESSAGE_RECEIVED";

type EventPayload = 
  | { event: "CONNECT"; socketId: string }
  | { event: "DISCONNECT"; socketId: string; reason: string }
  | { event: "MESSAGE_RECEIVED"; socketId: string; body: string };

type SocketEventHandler = (payload: EventPayload) => void;

class SocketManager {
  private handlers: Map<WebSocketEvent, SocketEventHandler[]> = new Map();

  public subscribe(event: WebSocketEvent, handler: SocketEventHandler): void {
    const existing = this.handlers.get(event) || [];
    this.handlers.set(event, [...existing, handler]);
  }

  public dispatch(payload: EventPayload): void {
    const eventHandlers = this.handlers.get(payload.event);
    if (eventHandlers) {
      eventHandlers.forEach((handler) => handler(payload));
    }
  }
}
```

### Example 5: Financial Currency Formatting & Conversion Engine
*Demonstrates Primitives, Exact Return Typing, and Math Functions.*

```typescript
type CurrencyCode = "USD" | "EUR" | "GBP" | "INR";

interface CurrencyRateMap {
  [code: string]: number; // Index signature
}

const exchangeRatesFromUSD: CurrencyRateMap = {
  USD: 1.0,
  EUR: 0.92,
  GBP: 0.78,
  INR: 83.12,
};

function convertCurrency(
  amount: number,
  from: CurrencyCode,
  to: CurrencyCode
): number {
  const amountInUSD = amount / exchangeRatesFromUSD[from];
  const convertedAmount = amountInUSD * exchangeRatesFromUSD[to];
  return Number(convertedAmount.toFixed(2));
}

function formatCurrencyDisplay(
  amount: number, 
  currency: CurrencyCode
): string {
  const formatted = convertCurrency(amount, "USD", currency);
  return `${currency} ${formatted.toLocaleString()}`;
}

// Usage
const priceInINR = formatCurrencyDisplay(150, "INR"); // Returns formatted converted amount
```