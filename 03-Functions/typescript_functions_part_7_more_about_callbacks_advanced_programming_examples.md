# TypeScript Functions (Part 7): More About Callbacks & Programming Examples

Welcome to **Part 7** of the TypeScript Functions series. Having covered the fundamentals of callbacks in Part 6, this guide dives deeper into **advanced callback patterns**, **functional techniques**, **context binding nuances (`this`)**, and **real-world production programming examples**.

---

## Table of Contents

1. [Understanding `this` Context in Callbacks](#1-understanding-this-context-in-callbacks)
2. [Functional Programming Patterns with Callbacks](#2-functional-programming-patterns-with-callbacks)
   - [Callback Composition / Piping](#a-callback-composition--piping)
   - [Callback-based Memoization](#b-callback-based-memoization)
   - [Rate Limiting: Debounce and Throttle Callbacks](#c-rate-limiting-debounce-and-throttle-callbacks)
3. [Asynchronous Control Flow Patterns](#3-asynchronous-control-flow-patterns)
   - [Parallel Execution Engine](#a-parallel-execution-engine)
   - [Sequential / Series Execution Engine](#b-sequential--series-execution-engine)
4. [Advanced Practical Examples](#4-advanced-practical-examples)
   - [Example 1: Real-time Data Pipeline with Transformation Hooks](#example-1-real-time-data-pipeline-with-transformation-hooks)
   - [Example 2: Reactive Pub/Sub State Store with Middleware Support](#example-2-reactive-pubsub-state-store-with-middleware-support)
5. [Summary & Cheat Sheet](#5-summary--cheat-sheet)

---

## 1. Understanding `this` Context in Callbacks

One common pitfall when working with callbacks in TypeScript is losing the reference to `this` when object methods are passed as callbacks.

### The Problem

```typescript
class UserNotifier {
  private prefix = "[NOTIFICATION]";

  public notify(message: string): void {
    console.log(`${this.prefix} ${message}`);
  }
}

function executeTask(taskName: string, callback: (msg: string) => void): void {
  callback(`Task '${taskName}' finished successfully.`);
}

const notifier = new UserNotifier();

// ❌ ERROR AT RUNTIME: 'this' is undefined when executeTask calls the method!
// executeTask("Data Processing", notifier.notify);
```

### Solutions in TypeScript

#### Option 1: Arrow Function Wrapper
Arrow functions capture `this` lexically from their enclosing scope.

```typescript
executeTask("Data Processing", (msg) => notifier.notify(msg));
```

#### Option 2: Explicit Binding (`Function.prototype.bind`)

```typescript
executeTask("Data Processing", notifier.notify.bind(notifier));
```

#### Option 3: Typed `this` Parameter in Callback Signatures
TypeScript allows declaring explicit `this` parameters to enforce type safety at compile time:

```typescript
type BoundCallback<TContext> = (this: TContext, message: string) => void;

function runWithContext<T>(
  context: T,
  callback: BoundCallback<T>,
  message: string
): void {
  callback.call(context, message);
}

runWithContext(notifier, notifier.notify, "Context safely bound!");
```

---

## 2. Functional Programming Patterns with Callbacks

### A. Callback Composition / Piping

Composition combines multiple callbacks into a single processing chain where the output of one callback becomes the input for the next.

```typescript
type Callback<T> = (input: T) => T;

function composeCallbacks<T>(...callbacks: Callback<T>[]): Callback<T> {
  return (initialValue: T): T => {
    return callbacks.reduce((acc, cb) => cb(acc), initialValue);
  };
}

// Concrete Example: Text Processing Pipeline
const trim = (str: string): string => str.trim();
const toLowerCase = (str: string): string => str.toLowerCase();
const removePunctuation = (str: string): string => str.replace(/[^\w\s]/gi, "");

const sanitizeInput = composeCallbacks(trim, toLowerCase, removePunctuation);

const rawUserString = "  Hello, World!!!  ";
console.log(sanitizeInput(rawUserString)); // Output: "hello world"
```

---

### B. Callback-based Memoization

Memoization caches function call results based on supplied arguments using a lookup callback.

```typescript
type MemoizedFn<A, R> = (arg: A) => R;

function memoize<A extends string | number, R>(
  fn: (arg: A) => R
): MemoizedFn<A, R> {
  const cache = new Map<A, R>();

  return (arg: A): R => {
    if (cache.has(arg)) {
      console.log(`[Cache Hit] for key: ${arg}`);
      return cache.get(arg)!;
    }

    console.log(`[Computing Result] for key: ${arg}`);
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

// Usage
const expensiveComputation = memoize((num: number): number => {
  return num * num * 100;
});

console.log(expensiveComputation(5)); // Computed: 2500
console.log(expensiveComputation(5)); // Cache Hit: 2500
```

---

### C. Rate Limiting: Debounce and Throttle Callbacks

Rate limiting ensures callbacks are not invoked excessively during high-frequency events (e.g., window resize, keypresses).

```typescript
type GenericCallback<T extends any[]> = (...args: T) => void;

// Debounce: Delay execution until 'delayMs' has elapsed since last call
function debounce<T extends any[]>(
  callback: GenericCallback<T>,
  delayMs: number
): GenericCallback<T> {
  let timerId: ReturnType<typeof setTimeout> | null = null;

  return (...args: T): void => {
    if (timerId !== null) {
      clearTimeout(timerId);
    }
    timerId = setTimeout(() => {
      callback(...args);
    }, delayMs);
  };
}

// Usage Example
const handleSearchInput = debounce((query: string) => {
  console.log(`Executing search API call for: "${query}"`);
}, 300);

handleSearchInput("T");
handleSearchInput("Ty");
handleSearchInput("TypeS");
// Only "TypeS" will execute after 300ms delay
```

---

## 3. Asynchronous Control Flow Patterns

Before Promises and `async/await` were standard, asynchronous flow control relied entirely on structured callbacks. Understanding these patterns is essential for low-level systems programming and reactive libraries.

### A. Parallel Execution Engine

Runs multiple asynchronous operations simultaneously and executes a completion callback once all tasks complete.

```typescript
type AsyncTask<T> = (callback: (err: Error | null, result?: T) => void) => void;

function runInParallel<T>(
  tasks: AsyncTask<T>[],
  onComplete: (err: Error | null, results?: T[]) => void
): void {
  const results: T[] = [];
  let completedCount = 0;
  let hasFailed = false;

  if (tasks.length === 0) {
    onComplete(null, []);
    return;
  }

  tasks.forEach((task, index) => {
    task((err, result) => {
      if (hasFailed) return;

      if (err) {
        hasFailed = true;
        onComplete(err);
        return;
      }

      results[index] = result!;
      completedCount++;

      if (completedCount === tasks.length) {
        onComplete(null, results);
      }
    });
  });
}

// Usage
const task1: AsyncTask<string> = (cb) =>
  setTimeout(() => cb(null, "Result 1"), 200);

const task2: AsyncTask<string> = (cb) =>
  setTimeout(() => cb(null, "Result 2"), 100);

runInParallel([task1, task2], (err, results) => {
  if (err) console.error("Parallel Error:", err);
  else console.log("Parallel Complete:", results); // Output: ["Result 1", "Result 2"]
});
```

---

### B. Sequential / Series Execution Engine

Runs asynchronous tasks strictly one after the other in sequence.

```typescript
function runInSeries<T>(
  tasks: AsyncTask<T>[],
  onComplete: (err: Error | null, results?: T[]) => void
): void {
  const results: T[] = [];

  function executeNext(index: number): void {
    if (index === tasks.length) {
      onComplete(null, results);
      return;
    }

    const currentTask = tasks[index];
    currentTask((err, result) => {
      if (err) {
        onComplete(err);
        return;
      }

      results.push(result!);
      executeNext(index + 1);
    });
  }

  executeNext(0);
}

// Usage
runInSeries([task1, task2], (err, results) => {
  if (err) console.error("Series Error:", err);
  else console.log("Series Complete:", results);
});
```

---

## 4. Advanced Practical Examples

### Example 1: Real-time Data Pipeline with Transformation Hooks

A data processor that accepts custom interceptor/transform callbacks for real-time ETL (Extract, Transform, Load) tasks.

```typescript
type TransformerHook<T> = (data: T) => T;
type ValidationHook<T> = (data: T) => boolean;
type OutputCallback<T> = (err: Error | null, processedData?: T) => void;

class StreamProcessor<T> {
  private transformers: TransformerHook<T>[] = [];
  private validators: ValidationHook<T>[] = [];

  public addTransform(hook: TransformerHook<T>): this {
    this.transformers.push(hook);
    return this;
  }

  public addValidator(hook: ValidationHook<T>): this {
    this.validators.push(hook);
    return this;
  }

  public process(data: T, onDone: OutputCallback<T>): void {
    try {
      // 1. Validation Step
      for (const validator of this.validators) {
        if (!validator(data)) {
          onDone(new Error("Validation failed for record"));
          return;
        }
      }

      // 2. Transformation Step
      let result = data;
      for (const transform of this.transformers) {
        result = transform(result);
      }

      // 3. Emit Result
      onDone(null, result);
    } catch (err) {
      onDone(err instanceof Error ? err : new Error(String(err)));
    }
  }
}

// Real-world Record Type
interface OrderPayload {
  orderId: string;
  amount: number;
  currency: string;
}

// Usage
const processor = new StreamProcessor<OrderPayload>();

processor
  .addValidator((order) => order.amount > 0)
  .addTransform((order) => ({
    ...order,
    currency: order.currency.toUpperCase(),
  }))
  .addTransform((order) => ({
    ...order,
    amount: Math.round(order.amount * 100) / 100, // Rounding
  }));

processor.process(
  { orderId: "ord_101", amount: 49.998, currency: "usd" },
  (err, finalRecord) => {
    if (err) console.error("Pipeline Error:", err.message);
    else console.log("Pipeline Output:", finalRecord);
  }
);
```

---

### Example 2: Reactive Pub/Sub State Store with Middleware Support

A state container allowing reactive state updates, action dispatching, and asynchronous middleware callbacks.

```typescript
type MiddlewareCallback<S> = (
  currentState: S,
  action: string,
  next: (newState: S) => void
) => void;

type SubscriberCallback<S> = (state: S) => void;

class ReactiveStore<S> {
  private state: S;
  private middlewares: MiddlewareCallback<S>[] = [];
  private subscribers: Set<SubscriberCallback<S>> = new Set();

  constructor(initialState: S) {
    this.state = initialState;
  }

  public use(middleware: MiddlewareCallback<S>): void {
    this.middlewares.push(middleware);
  }

  public subscribe(callback: SubscriberCallback<S>): () => void {
    this.subscribers.add(callback);
    // Return unsubscribe callback
    return () => this.subscribers.delete(callback);
  }

  public dispatch(action: string, stateReducer: (prev: S) => S): void {
    const nextState = stateReducer(this.state);

    let index = 0;
    const runMiddleware = (currentState: S) => {
      if (index < this.middlewares.length) {
        const middleware = this.middlewares[index++];
        middleware(currentState, action, runMiddleware);
      } else {
        // All middleware executed; update state and inform subscribers
        this.state = currentState;
        this.notifySubscribers();
      }
    };

    runMiddleware(nextState);
  }

  private notifySubscribers(): void {
    this.subscribers.forEach((subscriber) => subscriber(this.state));
  }

  public getState(): S {
    return this.state;
  }
}

// Usage Example
interface AppState {
  counter: number;
}

const store = new ReactiveStore<AppState>({ counter: 0 });

// Logging Middleware Callback
store.use((state, action, next) => {
  console.log(`[Middleware] Action '${action}' dispatched.`);
  next(state);
});

// Subscriber Callback
const unsubscribe = store.subscribe((updatedState) => {
  console.log(`[UI Subscription] Counter is now: ${updatedState.counter}`);
});

// Dispatch actions
store.dispatch("INCREMENT", (s) => ({ counter: s.counter + 1 }));
store.dispatch("ADD_TEN", (s) => ({ counter: s.counter + 10 }));

// Unsubscribe when done
unsubscribe();
```

---

## 5. Summary & Cheat Sheet

| Pattern | Key Signature / Mechanism | Best Use Case |
| :--- | :--- | :--- |
| **Bound Callback** | `(this: TContext, ...args: T) => R` | Class methods passed as callbacks without losing `this`. |
| **Composition** | `(...fns: Callback<T>[]) => Callback<T>` | Building processing pipelines or middleware chains. |
| **Debounce** | `(...args: T) => void` with internal timer | Throttling input events (search bar, window resize). |
| **Parallel Flow** | Iterating with completion count tracking | Running independent async network requests concurrently. |
| **Series Flow** | Recursive index-driven completion calls | Executing step-by-step sequential operations (migrations, installations). |
| **Unsubscribe Pattern** | Returning `() => void` cleanup callback | Managing listeners safely in event emitters or state subscribers. |
