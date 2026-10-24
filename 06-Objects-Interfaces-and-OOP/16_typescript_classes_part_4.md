# TypeScript Classes — Part 4

> **Focus:** Static Properties, Static Methods, Use Cases & Limitations

## Quick Overview

| Concept | Key Takeaway | Primary Use Case |
| :--- | :--- | :--- |
| **Static Keyword** | Binds properties/methods to the class, not instances | Class-level state, constants, and utilities |
| **Static Access** | Accessed using `ClassName.member` | Calling utility helpers without `new` |
| **Static Factory** | Methods returning new class instances | Alternative constructors, custom initialization |
| **Static Limitations** | Cannot access instance `this` properties directly | Shared logic independent of specific object data |

---

## 1. What are Static Properties & Their Behavior?

By default, properties defined inside a class belong to **instances** created with `new`. When a property is declared with the `static` keyword, it belongs directly to the **class itself**.

### Key Characteristics:
* **Memory Efficiency**: Only **one copy** of a static property exists in memory, shared across all usages.
* **No Instantiation Required**: Static properties can be accessed without creating an instance using `new`.
* **State Persistence**: Changes to a mutable static property affect the class-level state globally.

```typescript
class AppConfig {
  // Static property shared across the application
  static appName: string = "TypeScript Pro Engine";
  static maxConnections: number = 10;
}

// Accessing static properties directly on the class
console.log(AppConfig.appName);        // Output: "TypeScript Pro Engine"
console.log(AppConfig.maxConnections); // Output: 10
```

---

## 2. Accessing Static Members

Static members can be accessed both outside and inside the class, but always via the **class name** (or `this` inside static methods).

### Accessing Within Instance Methods
Instance methods cannot use instance `this` to access static properties. Instead, they must reference the class explicitly:

```typescript
class UserTracker {
  static totalUsers: number = 0; // Class-level property
  username: string;               // Instance-level property

  constructor(name: string) {
    this.username = name;
    // Accessing static property inside constructor via Class Name
    UserTracker.totalUsers++;
  }

  getStats(): string {
    return `User: ${this.username} | Total System Users: ${UserTracker.totalUsers}`;
  }
}

const user1 = new UserTracker("Alice");
const user2 = new UserTracker("Bob");

console.log(user1.getStats()); // Output: User: Alice | Total System Users: 2
console.log(UserTracker.totalUsers); // Output: 2
```

---

## 3. Static Methods

Static methods are functions defined on the class object rather than on class instances. They are ideal for pure utility logic, calculations, or operations that do not rely on instance state.

```typescript
class MathUtils {
  static readonly PI: number = 3.14159;

  // Static method performing a calculation
  static calculateArea(radius: number): number {
    return MathUtils.PI * radius * radius;
  }

  static isEven(value: number): boolean {
    return value % 2 === 0;
  }
}

// Directly calling static methods
console.log(MathUtils.calculateArea(5)); // Output: 78.53975
console.log(MathUtils.isEven(10));       // Output: true
```

---

## 4. Primary Uses of the `static` Keyword

### A. Utility / Helper Classes
Creating collections of related logic functions that don't need persistent state.

```typescript
class StringFormatter {
  static capitalize(str: string): string {
    return str.charAt(0).toUpperCase() + str.slice(1);
  }

  static slugify(str: string): string {
    return str.toLowerCase().replace(/\s+/g, "-");
  }
}

console.log(StringFormatter.slugify("TypeScript Classes Guide")); // Output: "typescript-classes-guide"
```

### B. Static Factory Methods
Alternative constructors that create and configure instances cleanly.

```typescript
class DateRange {
  constructor(public startDate: Date, public endDate: Date) {}

  // Factory Method: Create a range for today
  static createTodayRange(): DateRange {
    const today = new Date();
    return new DateRange(today, today);
  }

  // Factory Method: Create a range from a specific number of days ago until today
  static createPastDaysRange(days: number): DateRange {
    const end = new Date();
    const start = new Date();
    start.setDate(end.getDate() - days);
    return new DateRange(start, end);
  }
}

const lastWeek = DateRange.createPastDaysRange(7);
```

---

## 5. Practical Example: Global ID Generator & Registry

Here is a practical pattern combining static state management with instance initialization:

```typescript
class UniqueIdGenerator {
  // Shared counter & prefix
  private static currentId: number = 1000;
  private static prefix: string = "ID";

  // Static Method to configure generator behavior
  static setPrefix(newPrefix: string): void {
    UniqueIdGenerator.prefix = newPrefix;
  }

  // Static Method to generate next unique ID
  static generateNextId(): string {
    UniqueIdGenerator.currentId++;
    return `${UniqueIdGenerator.prefix}-${UniqueIdGenerator.currentId}`;
  }
}

class SystemItem {
  id: string;
  name: string;

  constructor(name: string) {
    this.id = UniqueIdGenerator.generateNextId();
    this.name = name;
  }

  display(): string {
    return `Item [${this.id}]: ${this.name}`;
  }
}

UniqueIdGenerator.setPrefix("PROD");

const item1 = new SystemItem("Monitor");
const item2 = new SystemItem("Keyboard");

console.log(item1.display()); // Output: Item [PROD-1001]: Monitor
console.log(item2.display()); // Output: Item [PROD-1002]: Keyboard
```

---

## 6. Static Properties & Methods Limitations

While static members are very powerful, they come with key architectural limitations:

### 1. Cannot Access Instance Members via `this`
Static methods operate at the class level and do not have access to individual object instances. Therefore, calling `this` inside a static method references the **Class constructor**, not an instance.

```typescript
class BadExample {
  instanceProperty: string = "Instance Data";

  static staticMethod(): void {
    // ❌ Error: Property 'instanceProperty' does not exist on type 'typeof BadExample'.
    // console.log(this.instanceProperty); 
  }
}
```

### 2. Cannot Be Accessed via Instances in TypeScript
Unlike some languages, TypeScript will throw a compile-time error if you try to access a static member through an instance variable.

```typescript
class Example {
  static systemVersion: string = "v2.0";
}

const instance = new Example();

// ❌ Error: Property 'systemVersion' does not exist on type 'Example'. 
// Did you mean to access the static member 'Example.systemVersion' instead?
// console.log(instance.systemVersion); 

// ✅ Correct Way
console.log(Example.systemVersion);
```

### 3. Lifetime and State Leakage
Static state persists for the entire lifecycle of the application runtime. Overusing mutable static properties can lead to global state bugs and make unit testing difficult.

---

## 📌 Summary Checklist

* [x] **Class Binding**: Declare shared state and helpers directly on the class using `static`.
* [x] **Direct Access**: Call static members using `ClassName.memberName`.
* [x] **Factories & Helpers**: Implement static methods for utility algorithms and object generation patterns.
* [x] **Mind the Scope**: Remember that static methods **cannot** directly access instance variables using `this`.