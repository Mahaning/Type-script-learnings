# JavaScript Functions — Part 7: More About Callbacks & Programming Examples

Continuing our deep dive into JavaScript functions, Part 7 focuses on applying callbacks across practical, real-world programming scenarios—from foundational synchronous operations to complex asynchronous workflows.

---

## Table of Contents
1. [Core Mechanics Refresh & Nuances](#1-core-mechanics-refresh--nuances)
2. [Basic Programming Examples](#2-basic-programming-examples)
3. [Intermediate Practical Patterns](#3-intermediate-practical-patterns)
4. [Advanced Asynchronous Control Flow](#4-advanced-asynchronous-control-flow)
5. [Summary Checklist](#5-summary-checklist)

---

## 1. Core Mechanics Refresh & Nuances

Before building complex examples, let's highlight key nuances when working with callbacks:

### A. Context (`this`) Binding Loss
When passing object methods as callbacks, they often lose their original `this` context when invoked inside another function.

```javascript
const userProfile = {
  name: "Alex",
  greet: function () {
    console.log(`Hello, my name is ${this.name}`);
  }
};

function executeCallback(cb) {
  cb(); // Executed in global context or undefined in strict mode
}

// ❌ BAD: 'this.name' will be undefined
executeCallback(userProfile.greet);

// ✅ GOOD: Bind explicit context or use an arrow function wrapper
executeCallback(userProfile.greet.bind(userProfile));
executeCallback(() => userProfile.greet());
```

---

## 2. Basic Programming Examples

### Example 1: Array Transformation Engine (Custom Map)
Implement your own version of `Array.prototype.map` to understand how array iteration callbacks function under the hood.

```javascript
function mapArray(arr, transformCallback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    // Pass element, index, and array instance to the callback
    const transformedValue = transformCallback(arr[i], i, arr);
    result.push(transformedValue);
  }
  return result;
}

// Usage
const rawPrices = [10, 20, 30, 40];
const formattedPrices = mapArray(rawPrices, (price, index) => {
  return `Item ${index + 1}: $${price.toFixed(2)}`;
});

console.log(formattedPrices);
// Output: ["Item 1: $10.00", "Item 2: $20.00", "Item 3: $30.00", "Item 4: $40.00"]
```

### Example 2: Input Sanitization and Formatting
Using callbacks to create flexible data pipelines for validating or transforming user inputs.

```javascript
function processInput(input, sanitizerCallback) {
  if (typeof input !== "string") {
    throw new TypeError("Input must be a string");
  }
  return sanitizerCallback(input);
}

// Custom sanitizer callbacks
const trimAndLowercase = (str) => str.trim().toLowerCase();
const removeSpecialChars = (str) => str.replace(/[^a-zA-Z0-9 ]/g, "");

console.log(processInput("  Hello World!  ", trimAndLowercase)); 
// Output: "hello world!"

console.log(processInput("User_123#$$", removeSpecialChars)); 
// Output: "User123"
```

---

## 3. Intermediate Practical Patterns

### Example 3: Event Emitter / Pub-Sub Pattern
Callbacks are fundamental to building publish-subscribe (Pub/Sub) event architecture.

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  // Subscribe to an event with a callback
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }

  // Emit an event and invoke all associated callbacks
  emit(eventName, data) {
    const eventCallbacks = this.events[eventName];
    if (eventCallbacks) {
      eventCallbacks.forEach((callback) => callback(data));
    }
  }
}

// Usage
const emitter = new EventEmitter();

emitter.on("userLoggedIn", (user) => {
  console.log(`[LOG]: User ${user.username} logged in.`);
});

emitter.on("userLoggedIn", (user) => {
  console.log(`[ANALYTICS]: Tracked session for ID ${user.id}`);
});

// Triggering the event
emitter.emit("userLoggedIn", { id: 402, username: "dev_john" });
```

---

## 4. Advanced Asynchronous Control Flow

In production code, you often need to coordinate multiple asynchronous operations using callbacks.

### Example 4: Asynchronous Parallel Execution (`asyncParallel`)
Execute multiple async operations at once and run a final completion callback when all operations are done.

```javascript
function asyncParallel(tasks, finalCallback) {
  const results = [];
  let completedCount = 0;
  let hasErrored = false;

  if (tasks.length === 0) {
    return finalCallback(null, []);
  }

  tasks.forEach((task, index) => {
    task((err, data) => {
      if (hasErrored) return;

      if (err) {
        hasErrored = true;
        return finalCallback(err, null);
      }

      results[index] = data; // Preserve task execution order
      completedCount++;

      if (completedCount === tasks.length) {
        finalCallback(null, results);
      }
    });
  });
}

// Simulated Async Operations
const fetchUser = (cb) => setTimeout(() => cb(null, { id: 101, name: "Maria" }), 300);
const fetchPosts = (cb) => setTimeout(() => cb(null, ["Post 1", "Post 2"]), 100);
const fetchSettings = (cb) => setTimeout(() => cb(null, { theme: "dark" }), 200);

// Run all in parallel
asyncParallel([fetchUser, fetchPosts, fetchSettings], (err, results) => {
  if (err) {
    console.error("Failed to load initial app data:", err);
    return;
  }
  const [user, posts, settings] = results;
  console.log("App Initialized Successfully:", { user, posts, settings });
});
```

### Example 5: Retrying Failed Asynchronous Operations
A resilience pattern that retries an operation up to $N$ times with a delay before executing the final callback.

```javascript
function retryOperation(task, maxRetries, delay, finalCallback) {
  let attempts = 0;

  function attempt() {
    attempts++;
    task((err, result) => {
      if (!err) {
        return finalCallback(null, result);
      }

      console.warn(`Attempt ${attempts} failed: ${err.message}`);

      if (attempts >= maxRetries) {
        return finalCallback(new Error(`Operation failed after ${maxRetries} attempts`), null);
      }

      setTimeout(attempt, delay);
    });
  }

  attempt();
}

// Simulated Flaky Network Task
let failureCount = 0;
function unstableNetworkRequest(callback) {
  setTimeout(() => {
    if (failureCount < 2) {
      failureCount++;
      callback(new Error("Connection Timeout"), null);
    } else {
      callback(null, { status: 200, data: "Success Data" });
    }
  }, 100);
}

// Execution
retryOperation(unstableNetworkRequest, 4, 200, (err, response) => {
  if (err) {
    console.error("Final Result:", err.message);
  } else {
    console.log("Final Result:", response);
  }
});
```

---

## 5. Summary Checklist

When working with callbacks in production code:

1. **Check Callback Existence**: Verify `typeof cb === 'function'` before calling it.
2. **Handle Context**: Mind the value of `this` inside callbacks—use `.bind()` or arrow functions when needed.
3. **Follow Conventions**: Always place error parameters first in asynchronous callbacks (`(err, result) => {}`).
4. **Prevent Multiple Execution**: Ensure callbacks inside loops or conditional blocks execute exactly once per invocation.