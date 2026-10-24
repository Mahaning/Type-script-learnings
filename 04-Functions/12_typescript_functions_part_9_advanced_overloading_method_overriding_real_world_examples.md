# TypeScript Functions (Part 9): Advanced Overloading, Method Overriding & Real-World Examples

Welcome to **Part 9** of our TypeScript Functions series. In this section, we will delve deeper into **Advanced Function Overloading**, explore **Method Overriding** (and clarify why standalone function overriding does or does not exist), and work through real-world production-grade programming examples.

---

## Table of Contents

1. [Does Overriding Exist in TypeScript? (Overloading vs. Overriding)](#1-does-overriding-exist-in-typescript-overloading-vs-overriding)
2. [Why Standalone Function Overriding Doesn't Exist in JavaScript/TypeScript](#2-why-standalone-function-overriding-doesnt-exist-in-javascripttypescript)
3. [Method Overriding in Object-Oriented TypeScript](#3-method-overriding-in-object-oriented-typescript)
4. [Advanced Overloading Techniques](#4-advanced-overloading-techniques)
   * [Conditional Return Types with Overloads](#a-conditional-return-types-with-overloads)
   * [Interface & Type Alias Overloads](#b-interface--type-alias-overloads)
   * [Combining Generics with Overloads](#c-combining-generics-with-overloads)
5. [Comprehensive Real-World Examples](#5-comprehensive-real-world-examples)
   * [Example 1: Strongly-Typed HTTP/REST Client (`fetch` Wrapper)](#example-1-strongly-typed-httprest-client-fetch-wrapper)
   * [Example 2: Method Overriding & Overloading in a Payment Gateway Engine](#example-2-method-overriding--overloading-in-a-payment-gateway-engine)
6. [Summary & Key Takeaways](#6-summary--key-takeaways)

---

## 1. Does Overriding Exist in TypeScript? (Overloading vs. Overriding)

The answer is **YES, but only in Class Inheritance (Object-Oriented Programming)**. Standalone function overriding does **NOT** exist in JavaScript or TypeScript.

To avoid confusion, let's compare the two concepts:

| Feature | Function / Method Overloading | Method Overriding |
| :--- | :--- | :--- |
| **Definition** | Defining multiple signatures for a **single function** with different parameter counts/types. | Redefining a method in a **child/derived class** that was already defined in a **parent/base class**. |
| **Scope** | Within a single function scope or class. | Across class inheritance hierarchies (`extends`). |
| **Target** | Standalone functions or Class Methods. | **Class Methods only**. |
| **Runtime Behavior** | Compiled into a **single JavaScript function** that handles different argument paths. | The child class method **replaces/shadows** the parent method at runtime via prototype chaining. |

---

## 2. Why Standalone Function Overriding Doesn't Exist in JavaScript/TypeScript

In languages like C++ or Java, you can declare two functions with the same name in the same file/namespace if their parameters differ. At runtime, the engine selects the correct memory address for execution based on the signature.

### In JavaScript / TypeScript:

1. **Variables and Functions Share a Lexical Scope**: Functions in JS are first-class objects assigned to identifiers.
2. **Name Shadowing/Redeclaration**: Declaring a function with the same name overwrites the previous definition in that scope.

```typescript
// ❌ If JavaScript supported traditional runtime function overriding, this would work:
function calculate(a: number): number { return a * 2; }
function calculate(a: number, b: number): number { return a + b; }

// BUT IN JAVASCRIPT RUNTIME:
// The second 'calculate' completely replaces/overwrites the first one!
// Calling calculate(5) at runtime will pass b = undefined and return NaN!
```

**Why TypeScript handles it this way:**
TypeScript compiles down to standard JavaScript. Because standard JavaScript engine mechanics only allow **one** function implementation per identifier, TypeScript uses a **single implementation signature** for overloads rather than true runtime function overriding.

---

## 3. Method Overriding in Object-Oriented TypeScript

While standalone function overriding does not exist, **Method Overriding** exists via OOP class inheritance using the `override` keyword (introduced in TypeScript 4.3).

```typescript
abstract class Logger {
  // Base implementation
  public log(message: string): void {
    console.log(`[BASE LOG]: ${message}`);
  }
}

class TimestampLogger extends Logger {
  // ✅ METHOD OVERRIDING
  // The child class provides its own implementation of 'log'
  public override log(message: string): void {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${message}`);
  }
}

class FileLogger extends Logger {
  // ✅ METHOD OVERRIDING with 'super' invocation
  public override log(message: string): void {
    super.log(message); // Invoke base method
    console.log("-> Saved log entry to disk file.");
  }
}

// Execution
const loggers: Logger[] = [new TimestampLogger(), new FileLogger()];
loggers.forEach(l => l.log("System initialized."));
```

> **Note on `override` keyword:** Using `override` ensures that if the parent class changes its method name, TypeScript will throw a compile-time error warning you that you are no longer overriding an existing method.

---

## 4. Advanced Overloading Techniques

### A. Conditional Return Types with Overloads

Overloads can map precise argument configurations to distinct return types.

```typescript
// Overloads mapping primitive flags to return formats
function serialize(data: Record<string, any>, format: "json"): string;
function serialize(data: Record<string, any>, format: "buffer"): Buffer;
function serialize(data: Record<string, any>, format: "map"): Map<string, any>;

// Implementation
function serialize(
  data: Record<string, any>,
  format: "json" | "buffer" | "map"
): string | Buffer | Map<string, any> {
  switch (format) {
    case "json":
      return JSON.stringify(data);
    case "buffer":
      return Buffer.from(JSON.stringify(data));
    case "map":
      return new Map(Object.entries(data));
  }
}

const jsonStr = serialize({ name: "Alice" }, "json");    // Type: string
const buf = serialize({ name: "Alice" }, "buffer");      // Type: Buffer
const mapObj = serialize({ name: "Alice" }, "map");      // Type: Map<string, any>
```

---

### B. Interface & Type Alias Overloads

You can declare overload signatures inside Interfaces and Type Aliases for object methods or callable function types.

```typescript
interface DocumentParser {
  // Callable overload signatures within an interface
  parse(rawText: string): Record<string, unknown>;
  parse(rawText: string, strict: true): Record<string, unknown>;
  parse(buffer: Buffer): Record<string, unknown>;
}

const parser: DocumentParser = {
  parse(input: string | Buffer, strict?: boolean): Record<string, unknown> {
    if (Buffer.isBuffer(input)) {
      return JSON.parse(input.toString("utf-8"));
    }
    return JSON.parse(input);
  }
};
```

---

### C. Combining Generics with Overloads

Generics can be combined with overloads to provide type-safe operations across varying shapes of data.

```typescript
// Generic Overload 1: Extract array item by index
function getItem<T>(list: T[], index: number): T | undefined;

// Generic Overload 2: Extract object value by key
function getItem<T extends object, K extends keyof T>(obj: T, key: K): T[K];

// Implementation
function getItem(container: any, keyOrIndex: any): any {
  if (Array.isArray(container)) {
    return container[keyOrIndex];
  }
  return container[keyOrIndex];
}

const numArr = [10, 20, 30];
const firstNum = getItem(numArr, 0); // Inferred Type: number | undefined

const userObj = { id: 101, username: "dev_john" };
const username = getItem(userObj, "username"); // Inferred Type: string
```

---

## 5. Comprehensive Real-World Examples

### Example 1: Strongly-Typed HTTP/REST Client (`fetch` Wrapper)

This example models a web API client that uses function overloading to return raw JSON response bodies, Blob objects, or parsed custom DTO models depending on how `apiCall` is invoked.

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
}

// Overload 1: Return raw text response
function apiCall(url: string, responseType: "text"): Promise<string>;

// Overload 2: Return binary Blob response
function apiCall(url: string, responseType: "blob"): Promise<Blob>;

// Overload 3: Return typed JSON with wrapper schema
function apiCall<T>(url: string, responseType: "json"): Promise<ApiResponse<T>>;

// Implementation
async function apiCall<T>(
  url: string,
  responseType: "text" | "blob" | "json"
): Promise<string | Blob | ApiResponse<T>> {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`HTTP Error: ${response.status}`);
  }

  if (responseType === "text") {
    return await response.text();
  }

  if (responseType === "blob") {
    return await response.blob();
  }

  const data = await response.json();
  return { data, status: response.status };
}

// --- Real-world Usage ---
async function runExamples() {
  // Overload 1 -> inferred return type: string
  const htmlContent = await apiCall("https://example.com/page", "text");

  // Overload 2 -> inferred return type: Blob
  const imageBlob = await apiCall("https://example.com/avatar.png", "blob");

  // Overload 3 -> inferred return type: ApiResponse<{ id: number; name: string }>
  interface UserDto { id: number; name: string; }
  const userResponse = await apiCall<UserDto>("https://api.example.com/user/1", "json");

  console.log(`User Name: ${userResponse.data.name}`);
}
```

---

### Example 2: Method Overriding & Overloading in a Payment Gateway Engine

This example demonstrates **Method Overloading** and **Method Overriding** working together within an enterprise payment processing system.

```typescript
// --- Base Payment Gateway ---
abstract class PaymentGateway {
  public abstract gatewayName: string;

  // OVERLOAD SIGNATURES: Processing payments with card details OR stored token
  public processPayment(amount: number, cardNumber: string, cvc: string): Promise<boolean>;
  public processPayment(amount: number, paymentToken: string): Promise<boolean>;

  // IMPLEMENTATION SIGNATURE
  public async processPayment(
    amount: number,
    cardOrToken: string,
    cvc?: string
  ): Promise<boolean> {
    if (cvc) {
      return this.executeCardTransaction(amount, cardOrToken, cvc);
    }
    return this.executeTokenTransaction(amount, cardOrToken);
  }

  protected abstract executeCardTransaction(amount: number, card: string, cvc: string): Promise<boolean>;
  protected abstract executeTokenTransaction(amount: number, token: string): Promise<boolean>;
}

// --- Stripe Gateway Implementation ---
class StripeGateway extends PaymentGateway {
  public override gatewayName = "Stripe";

  // OVERRIDING abstract protected execution methods
  protected override async executeCardTransaction(amount: number, card: string, cvc: string): Promise<boolean> {
    console.log(`[Stripe] Charged $${amount} via Card ending in ${card.slice(-4)}`);
    return true;
  }

  protected override async executeTokenTransaction(amount: number, token: string): Promise<boolean> {
    console.log(`[Stripe] Charged $${amount} via Saved Token: ${token}`);
    return true;
  }
}

// --- PayPal Gateway Implementation ---
class PayPalGateway extends PaymentGateway {
  public override gatewayName = "PayPal";

  protected override async executeCardTransaction(amount: number, card: string, cvc: string): Promise<boolean> {
    console.log(`[PayPal] Processing direct card charge for $${amount}`);
    return true;
  }

  protected override async executeTokenTransaction(amount: number, token: string): Promise<boolean> {
    console.log(`[PayPal] Executing Billing Agreement Token: ${token}`);
    return true;
  }
}

// --- Client Execution ---
async function checkout(gateway: PaymentGateway) {
  // Invoking Overload 1 (Direct Card)
  await gateway.processPayment(150.0, "4111111111111111", "123");

  // Invoking Overload 2 (Saved Token)
  await gateway.processPayment(75.5, "tok_stripe_abc123");
}

const stripe = new StripeGateway();
checkout(stripe);
```

---

## 6. Summary & Key Takeaways

1. **Standalone Function Overriding Does NOT Exist**: In JavaScript runtime, declaring multiple functions with the same name replaces earlier definitions.
2. **Method Overriding DOES Exist in OOP**: Classes in TypeScript support overriding inherited methods using the `override` keyword.
3. **Function Overloading is a Compile-Time Concept**: TypeScript allows multiple overload signatures for a single shared function implementation.
4. **Implementation Visibility**: Callers can only invoke the explicit public **overload signatures**—the broader implementation signature remains hidden.
5. **Use the `override` Keyword**: Always annotate overridden class methods with `override` to catch breaking base-class changes during compilation.
