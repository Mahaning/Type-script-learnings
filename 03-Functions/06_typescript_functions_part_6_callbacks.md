# TypeScript Functions (Part 6): Callbacks

Welcome to **Part 6** of our TypeScript Functions series. In this guide, we will explore **Callbacks**—a foundational concept in JavaScript and TypeScript for handling asynchronous behavior, event-driven programming, and functional patterns.

---

## Table of Contents
1. [Definition & Introduction](#1-definition--introduction)
2. [Basic Implementation & Type Annotations](#2-basic-implementation--type-annotations)
3. [Internal Working & Execution Mechanics](#3-internal-working--execution-mechanics)
4. [Advanced Callback Concepts](#4-advanced-callback-concepts)
5. [Real-time Scenarios](#5-real-time-scenarios)
6. [Advanced Callback Scenarios & Patterns](#6-advanced-callback-scenarios--patterns)
7. [Summary & Best Practices](#7-summary--best-practices)

---

## 1. Definition & Introduction

### What is a Callback Function?
A **callback function** is a function passed into another function as an argument, which is then executed inside the outer function to complete an action or process data.

In JavaScript and TypeScript, functions are **First-Class Citizens**. This means:
- Functions can be assigned to variables.
- Functions can be passed as parameters to other functions.
- Functions can be returned from other functions.

```
+-------------------------------------------------------+
|                    Outer Function                     |
|                                                       |
|  1. Performs some work / task                         |
|  2. Invokes Callback Function ---------------------+  |
+----------------------------------------------------|--+
                                                     |
                                                     v
                                       +---------------------------+
                                       |     Callback Function     |
                                       | (Executes with result)    |
                                       +---------------------------+
```

### Why Use Callbacks?
1. **Asynchronous Operations**: Wait for network requests, timer delays, or file I/O to complete without blocking execution.
2. **Reusability & Customization**: Pass custom logic into a function (e.g., custom sorting or array filtering).
3. **Event-Driven Programming**: React to user interactions (clicks, keypresses) or system events.

---

## 2. Basic Implementation & Type Annotations

To implement callbacks safely in TypeScript, we define explicit function types for the parameters.

### Syntax 1: Inline Type Annotation

```typescript
// Define a function that accepts a callback parameter
function processUser(
  name: string,
  callback: (message: string) => void
): void {
  const greeting = `Hello, ${name}!`;
  callback(greeting); // Invoking the callback
}

// Usage with a named function
function printLog(msg: string): void {
  console.log(`LOG: ${msg}`);
}

processUser("Alice", printLog);

// Usage with an anonymous arrow function
processUser("Bob", (msg) => {
  console.log(`NOTIFICATION: ${msg}`);
});
```

### Syntax 2: Using `type` Aliases (Recommended)

When callback signatures repeat across your codebase, define a `type` or `interface` for clarity:

```typescript
// Type Alias for Callback
type SuccessCallback = (data: string, statusCode: number) => void;
type ErrorCallback = (error: Error) => void;

function fetchData(
  url: string,
  onSuccess: SuccessCallback,
  onError: ErrorCallback
): void {
  if (url === "https://api.example.com/data") {
    onSuccess("Data fetched successfully!", 200);
  } else {
    onError(new Error("Invalid URL"));
  }
}

// Usage
fetchData(
  "https://api.example.com/data",
  (data, code) => console.log(`[${code}] ${data}`),
  (err) => console.error(`Error: ${err.message}`)
);
```

---

## 3. Internal Working & Execution Mechanics

To understand how callbacks execute internally, we need to inspect the **Call Stack**, **Web APIs**, and the **Event Loop**.

### Synchronous vs. Asynchronous Callbacks

#### 1. Synchronous Callbacks
Synchronous callbacks run immediately on the main thread during execution.

```typescript
const numbers = [1, 2, 3, 4];

// Array.prototype.map uses a synchronous callback
const doubled = numbers.map((num) => num * 2);

console.log(doubled); // [2, 4, 6]
```
*Execution Order:* `numbers.map()` calls the callback function repeatedly within the same stack frame before returning control.

#### 2. Asynchronous Callbacks
Asynchronous callbacks are offloaded to background threads (Web APIs / Node C++ APIs) and pushed to the **Task Queue**.

```typescript
console.log("Step 1: Start");

setTimeout(() => {
  console.log("Step 2: Callback executed");
}, 1000);

console.log("Step 3: End");

// Output:
// Step 1: Start
// Step 3: End
// Step 2: Callback executed
```

### Internal Execution Diagram

```
1. Call Stack              2. Web APIs / Node APIs             3. Task Queue
+-------------------+      +-------------------------------+   +-------------------+
| console.log("S1") | ---> |                               |   |                   |
| setTimeout(...)   | ---> | Timer (1000ms counter)        |   |                   |
| console.log("S3") |      +-------------------------------+   |                   |
+-------------------+                                          +-------------------+
        |                                                                |
        | Stack clears                                                   | Timer expires
        v                                                                v
+-------------------+                                          +-------------------+
|                   | <--- Event Loop moves callback <-------- | callback()        |
+-------------------+      when stack is empty                 +-------------------+
```

---

## 4. Advanced Callback Concepts

### A. Generic Callbacks
Generics allow dynamic typing for callback input and output data.

```typescript
// Generic transformation callback
type Transformer<T, U> = (input: T) => U;

function transformArray<T, U>(arr: T[], callback: Transformer<T, U>): U[] {
  const result: U[] = [];
  for (const item of arr) {
    result.push(callback(item));
  }
  return result;
}

// Example 1: Transform numbers to strings
const numList = [10, 20, 30];
const stringList = transformArray(numList, (n) => `Value: $${n}`);

// Example 2: Transform objects to booleans
const users = [{ age: 15 }, { age: 22 }, { age: 18 }];
const adults = transformArray(users, (user) => user.age >= 18);
```

### B. Node.js Style: Error-First Callbacks
A standard convention in Node.js where the first argument is reserved for an error object.

```typescript
type NodeCallback<T> = (err: Error | null, result?: T) => void;

function readFileSimulated(path: string, callback: NodeCallback<string>): void {
  if (!path.endsWith(".txt")) {
    callback(new Error("Unsupported file format"));
    return;
  }
  
  // Simulate async delay
  setTimeout(() => {
    callback(null, "File content: Hello World!");
  }, 500);
}

// Usage
readFileSimulated("notes.txt", (err, content) => {
  if (err) {
    console.error("Failed:", err.message);
    return;
  }
  console.log("Success:", content);
});
```

### C. Optional Callbacks
Callbacks can be made optional using `?` and guarded before invocation.

```typescript
type ConfigOptions = {
  silent?: boolean;
  onComplete?: (status: boolean) => void;
};

function runTask(taskName: string, config?: ConfigOptions): void {
  console.log(`Running ${taskName}...`);
  
  const success = true;

  // Safe call using optional chaining
  config?.onComplete?.(success);
}

runTask("Backup Job"); // Works without callback
runTask("Data Sync", { onComplete: (res) => console.log(`Done: ${res}`) });
```

---

## 5. Real-time Scenarios

### Scenario 1: Custom Event Emitter
Handling DOM-style dynamic subscriptions.

```typescript
type EventListener<T = any> = (data: T) => void;

class SimpleEventEmitter {
  private listeners: Record<string, EventListener[]> = {};

  on<T>(event: string, callback: EventListener<T>): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback as EventListener);
  }

  emit<T>(event: string, data: T): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach((listener) => listener(data));
    }
  }
}

// Real-time usage
const emitter = new SimpleEventEmitter();

emitter.on<{ userId: string }>("userLoggedIn", (data) => {
  console.log(`User ${data.userId} logged in.`);
});

emitter.emit("userLoggedIn", { userId: "usr_9981" });
```

### Scenario 2: Array Filter Engine
Writing a custom filtering system powered by predicates (boolean-returning callbacks).

```typescript
type Predicate<T> = (item: T) => boolean;

function filterItems<T>(items: T[], predicate: Predicate<T>): T[] {
  const filtered: T[] = [];
  for (const item of items) {
    if (predicate(item)) {
      filtered.push(item);
    }
  }
  return filtered;
}

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

const inventory: Product[] = [
  { id: 1, name: "Laptop", price: 1200, inStock: true },
  { id: 2, name: "Mouse", price: 25, inStock: false },
  { id: 3, name: "Keyboard", price: 75, inStock: true },
];

const availableAffordable = filterItems(
  inventory,
  (p) => p.inStock && p.price < 100
);

console.log(availableAffordable); // Keyboard
```

---

## 6. Advanced Callback Scenarios & Patterns

### 1. Avoiding Callback Hell (Promisification)
Deeply nested callbacks create unreadable code ("Callback Hell" / "Pyramid of Doom"). We convert callbacks into Promises using TypeScript typing.

```typescript
// Callback-based function
function legacyFetch(id: number, callback: (err: Error | null, res?: string) => void) {
  setTimeout(() => {
    if (id <= 0) callback(new Error("Invalid ID"));
    else callback(null, `Item ${id}`);
  }, 300);
}

// Wrapper function returning a Promise
function fetchAsPromise(id: number): Promise<string> {
  return new Promise((resolve, reject) => {
    legacyFetch(id, (err, res) => {
      if (err) reject(err);
      else resolve(res!);
    });
  });
}

// Usage with Async/Await
async function loadData() {
  try {
    const result = await fetchAsPromise(10);
    console.log("Promisified output:", result);
  } catch (err) {
    console.error("Caught error:", err);
  }
}

loadData();
```

### 2. Callback Retries with Exponential Backoff
A resilient retry pattern utilizing delayed callbacks.

```typescript
type AsyncOperation = (
  attempt: number,
  callback: (err: Error | null, result?: string) => void
) => void;

function retryWithBackoff(
  operation: AsyncOperation,
  maxRetries: number,
  delayMs: number,
  onFinalResult: (err: Error | null, result?: string) => void
): void {
  function attemptRetry(currentAttempt: number) {
    operation(currentAttempt, (err, result) => {
      if (!err) {
        return onFinalResult(null, result);
      }

      if (currentAttempt >= maxRetries) {
        return onFinalResult(
          new Error(`Failed after ${maxRetries} attempts: ${err.message}`)
        );
      }

      const nextDelay = delayMs * Math.pow(2, currentAttempt - 1);
      console.log(`Attempt ${currentAttempt} failed. Retrying in ${nextDelay}ms...`);

      setTimeout(() => {
        attemptRetry(currentAttempt + 1);
      }, nextDelay);
    });
  }

  attemptRetry(1);
}

// Example Execution
const unstableNetworkCall: AsyncOperation = (attempt, cb) => {
  if (attempt < 3) {
    cb(new Error("Network Timeout"));
  } else {
    cb(null, "Connected Successfully!");
  }
};

retryWithBackoff(unstableNetworkCall, 3, 200, (err, res) => {
  if (err) console.error("Final Fail:", err.message);
  else console.log("Final Success:", res);
});
```

---

## 7. Summary & Best Practices

| Topic | Best Practice |
| :--- | :--- |
| **Type Definitions** | Prefer `type` aliases for reusable callbacks instead of raw inline types. |
| **Return Types** | Use `void` as return type if callback output will be ignored by caller. |
| **Null Checks** | Use optional chaining (`cb?.()`) for optional callbacks. |
| **Error Handling** | Always follow standard error handling (e.g., Error-First Callbacks). |
| **Asynchrony** | Convert complex nested callbacks to `Promises` or `async/await` to improve readability. |