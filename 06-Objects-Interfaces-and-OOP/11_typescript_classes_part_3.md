# TypeScript Classes — Part 3
> **Focus:** Method Syntax, Object Creation, State Management & Immutability

---

## Quick Overview

| Concept | Key Takeaway | Primary Use Case |
| :--- | :--- | :--- |
| **Standard Method** | Stored on class prototype | Default choice for performance and low memory overhead |
| **Arrow Method** | Bound to class instance | Callbacks & event listeners (preserves `this`) |
| **Method Overloads** | Multiple signatures, single implementation | Flexibly handling different argument types |
| **Object Independence** | Instances maintain unique memory locations | Isolating state across separate objects |
| **Readonly Guard** | Compile-time immutability enforcement | Protecting IDs, timestamps, and static config |

---

## 1. Method Syntax Variations

TypeScript offers multiple ways to define class methods based on how `this` scoping and function signatures should behave.

### A. Standard Method Syntax
Standard methods live on the **prototype object**, making them lightweight and memory-efficient because all instances share a single implementation.

```typescript
class Calculator {
  // Standard method syntax
  add(a: number, b: number): number {
    return a + b;
  }
}
```

---

### B. Arrow Function Methods (Instance Properties)
Arrow functions automatically bind `this` to the specific instance created. They are useful when passing methods as callbacks (e.g., event listeners or timer handlers) where `this` binding could otherwise be lost.

> ⚠️ **Trade-off:** Each instance creates its own copy of the function, using slightly more memory than standard methods.

```typescript
class Timer {
  seconds: number = 0;

  // Arrow function guarantees 'this' always points to the instance
  tick = (): void => {
    this.seconds++;
    console.log(`Current seconds: ${this.seconds}`);
  };
}

const timer = new Timer();

// Safe: 'this' context is preserved even when passed as a callback
setTimeout(timer.tick, 1000);
```

---

### C. Method Overload Signatures
Overloads allow a method to accept different argument combinations while maintaining strong type safety.

```typescript
class Formatter {
  // 1. Overload Signatures (Type Definitions)
  format(value: string): string;
  format(value: number): string;
  
  // 2. Single Implementation Signature
  format(value: string | number): string {
    if (typeof value === "number") {
      return `$${value.toFixed(2)}`;
    }
    return value.trim().toUpperCase();
  }
}

const fmt = new Formatter();
console.log(fmt.format(100));      // Output: "$100.00"
console.log(fmt.format(" hello"));  // Output: "HELLO"
```

---

## 2. Object Creation & Lifecycle

An object instance is constructed using the `new` keyword, which invokes the class `constructor`.

```typescript
class Car {
  make: string;
  model: string;
  year: number;

  constructor(make: string, model: string, year: number) {
    this.make = make;
    this.model = model;
    this.year = year;
  }
}

// Instantiation
const myCar = new Car("Honda", "Civic", 2022);
const rentalCar = new Car("Ford", "Mustang", 2024);
```

### What Happens During Instantiation?

```
  1. Allocation          2. Execution             3. Binding              4. Return
┌────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌──────────────────┐
│ Allocate fresh │ ──>│ Call class      │ ──>│ Assign parameters│ ──>│ Return instance  │
│ memory block   │    │ constructor     │    │ to 'this' fields │    │ object reference │
└────────────────┘    └─────────────────┘    └─────────────────┘    └──────────────────┘
```

---

## 3. Working with Class Objects

After instantiation, interaction with properties and execution of methods occurs via **dot notation** (`.`).

```typescript
class BankAccount {
  accountNumber: string;
  balance: number;

  constructor(accountNumber: string, initialBalance: number) {
    this.accountNumber = accountNumber;
    this.balance = initialBalance;
  }

  getBalanceInfo(): string {
    return `Account ${this.accountNumber} balance: $${this.balance}`;
  }
}

const account = new BankAccount("ACC-9012", 1500);

// Accessing fields & executing methods
console.log(account.accountNumber);    // Output: "ACC-9012"
console.log(account.getBalanceInfo()); // Output: "Account ACC-9012 balance: $1500"
```

---

## 4. Object Independence in Memory

Every instantiated object occupies its own independent block in heap memory. Modifying properties in one instance **never** alters another instance.

```typescript
class Counter {
  count: number = 0;

  increment(): void {
    this.count++;
  }
}

// Creating two distinct instances
const counterA = new Counter();
const counterB = new Counter();

counterA.increment();
counterA.increment();

console.log(counterA.count); // Output: 2
console.log(counterB.count); // Output: 0 (counterB remains isolated)

// Reference Equality Verification
console.log(counterA === counterB); // Output: false (distinct memory addresses)
```

---

## 5. Modifying Instance Properties

Properties not restricted by immutability flags can be updated directly via property assignment or safely through dedicated mutator methods.

```typescript
class Player {
  name: string;
  score: number;

  constructor(name: string, initialScore: number = 0) {
    this.name = name;
    this.score = initialScore;
  }

  // Preferred: Controlled state mutation via method
  addPoints(points: number): void {
    this.score += points;
  }
}

const player = new Player("Alex", 10);

// Option A: Direct modification
player.score = 25;

// Option B: Encapsulated modification
player.addPoints(15);
console.log(player.score); // Output: 40
```

---

## 6. Enforcing Immutability (`readonly`)

The `readonly` modifier locks property values after object instantiation. Attempts to alter a `readonly` field outside the constructor trigger compile-time errors.

### A. Basic Readonly Check

```typescript
class Device {
  readonly serialNumber: string; // Immutable field
  deviceName: string;            // Mutable field

  constructor(serialNumber: string, deviceName: string) {
    this.serialNumber = serialNumber; // Allowed in constructor
    this.deviceName = deviceName;
  }

  updateDevice(newName: string, newSerial: string): void {
    this.deviceName = newName; // OK
    
    // ❌ Error: Cannot assign to 'serialNumber' because it is a read-only property.
    // this.serialNumber = newSerial; 
  }
}

const phone = new Device("SN-9988-X", "Smartphone");

// Valid field update
phone.deviceName = "Smartphone Pro";

// ❌ Invalid Field Update
// phone.serialNumber = "SN-0000-Z"; // Error!
```

---

### B. Readonly Collections

To prevent internal mutations (like `.push()` or `.pop()`) on array properties, pair `readonly` with `ReadonlyArray<T>`.

```typescript
class AuditLog {
  readonly logs: ReadonlyArray<string>;

  constructor(initialLogs: string[]) {
    this.logs = [...initialLogs]; // Create a frozen shallow copy
  }

  addEntry(entry: string): void {
    // ❌ Error: Property 'push' does not exist on type 'readonly string[]'.
    // this.logs.push(entry); 
  }
}
```

---

## 📌 Summary Checklist

- [x] **Syntax Selection**: Choose standard methods for efficiency or arrow methods when context-binding (`this`) is needed.
- [x] **Instantiation**: Create independent instances using the `new` operator.
- [x] **Memory Isolation**: Ensure distinct objects handle their state independently without cross-leakage.
- [x] **Immutability Protection**: Guard structural keys, IDs, and base data using `readonly` and `ReadonlyArray<T>`.