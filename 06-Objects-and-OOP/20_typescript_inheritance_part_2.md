# TypeScript Inheritance - Deep Dive (Part 2)

## Table of Contents

1. [Types of Inheritance in TypeScript](#1-types-of-inheritance-in-typescript)
   - [Single Inheritance](#11-single-inheritance)
   - [Multilevel Inheritance](#12-multilevel-inheritance)
   - [Hierarchical Inheritance](#13-hierarchical-inheritance)
   - [Multiple / Hybrid Inheritance (Via Mixins & Interfaces)](#14-multiple--hybrid-inheritance-via-mixins--interfaces)
2. [Managing Constructors in Derived Classes](#2-managing-constructors-in-derived-classes)
   - [The `super()` Execution Flow](#21-the-super-execution-flow)
   - [Parameter Properties in Inherited Constructors](#22-parameter-properties-in-inherited-constructors)
   - [Default & Omitted Constructors](#23-default--omitted-constructors)
3. [Method Overriding vs Method Overloading](#3-method-overriding-vs-method-overloading)
   - [Method Overriding & The `override` Keyword](#31-method-overriding--the-override-keyword)
   - [Method Overloading in Classes](#32-method-overloading-in-classes)
4. [Advanced Concepts & Anti-Patterns](#4-advanced-concepts--anti-patterns)

---

## 1. Types of Inheritance in TypeScript

TypeScript supports classical single-dispatch inheritance models natively, as well as multiple-inheritance behaviors through mixins and interface implementation.

```
       +-----------------------------------------------------------+
       |                  INHERITANCE TYPES IN TS                 |
       +-----------------------------------------------------------+
       |                                                           |
       |  1. Single          2. Multilevel        3. Hierarchical  |
       |    [ Base ]           [ Base ]               [ Base ]     |
       |       |                  |                  /        \    |
       |    [ Child ]          [ Child ]         [ChildA]   [ChildB|
       |                          |                                |
       |                       [ Grand ]                           |
       |                                                           |
       |  4. Multiple/Hybrid (Via Interfaces & Mixins)             |
       |    [ InterfaceA ]   [ InterfaceB ]                        |
       |            \             /                                |
       |            [ ConcreteClass ]                              |
       +-----------------------------------------------------------+
```

---

### 1.1 Single Inheritance
A child class derives directly from a single parent class.

```typescript
class Vehicle {
  constructor(public brand: string) {}

  public startEngine(): void {
    console.log(`${this.brand} engine started.`);
  }
}

class Car extends Vehicle {
  public doors: number = 4;
}

const myCar = new Car("Toyota");
myCar.startEngine(); // Output: Toyota engine started.
```

---

### 1.2 Multilevel Inheritance
A child class acts as a base class for another derived class, forming an inheritance chain.

```typescript
class ElectronicDevice {
  constructor(public powerSupply: string) {}
}

class Computer extends ElectronicDevice {
  constructor(powerSupply: string, public processor: string) {
    super(powerSupply);
  }
}

class Laptop extends Computer {
  constructor(powerSupply: string, processor: string, public batteryLifeHours: number) {
    super(powerSupply, processor);
  }

  public getSpecs(): string {
    return `Power: ${this.powerSupply}, CPU: ${this.processor}, Battery: ${this.batteryLifeHours}h`;
  }
}

const macbook = new Laptop("65W USB-C", "Apple M3", 18);
console.log(macbook.getSpecs());
```

---

### 1.3 Hierarchical Inheritance
Multiple child classes extend from a single parent class.

```typescript
abstract class Employee {
  constructor(public id: string, public name: string, public baseSalary: number) {}
  public abstract calculatePay(): number;
}

class FullTimeEmployee extends Employee {
  public calculatePay(): number {
    return this.baseSalary; // Fixed monthly salary
  }
}

class ContractEmployee extends Employee {
  constructor(id: string, name: string, baseSalary: number, public hoursWorked: number) {
    super(id, name, baseSalary);
  }

  public calculatePay(): number {
    return this.baseSalary * this.hoursWorked; // Hourly rate * hours
  }
}
```

---

### 1.4 Multiple / Hybrid Inheritance (Via Mixins & Interfaces)
TypeScript **does not support multiple inheritance with classes directly** (i.e., `class C extends A, B` is illegal). However, you can achieve multiple behavioral inheritance using **Interfaces** or **Mixins**.

#### Implementing Multiple Interfaces
```typescript
interface Serializable {
  serialize(): string;
}

interface Printable {
  print(): void;
}

class DocumentReport implements Serializable, Printable {
  constructor(public content: string) {}

  public serialize(): string {
    return JSON.stringify({ data: this.content });
  }

  public print(): void {
    console.log(`Printing: ${this.content}`);
  }
}
```

#### TypeScript Mixin Pattern
Mixins allow you to compose classes from reusable behavior functions.

```typescript
// Disposable Mixin
function DisposableMixin<TBase extends new (...args: any[]) => {}>(Base: TBase) {
  return class extends Base {
    public isDisposed: boolean = false;
    public dispose(): void {
      this.isDisposed = true;
      console.log("Resource disposed.");
    }
  };
}

// Timestamp Mixin
function TimestampMixin<TBase extends new (...args: any[]) => {}>(Base: TBase) {
  return class extends Base {
    public createdAt: Date = new Date();
  };
}

// Base operational class
class UserSession {
  constructor(public userId: string) {}
}

// Composite Class combining UserSession + Disposable + Timestamp
const EnhancedSession = TimestampMixin(DisposableMixin(UserSession));

const session = new EnhancedSession("usr_1029");
console.log(session.userId);    // From UserSession
console.log(session.createdAt); // From TimestampMixin
session.dispose();              // From DisposableMixin
```

---

## 2. Managing Constructors in Derived Classes

Constructors in inherited classes follow strict initialization rules enforced by the TypeScript compiler to ensure safety.

### 2.1 The `super()` Execution Flow

When a subclass contains its own constructor:
1. `super(...)` **must** be called before referencing `this`, `super`, or returning from the constructor.
2. `super(...)` delegates execution to the parent constructor first, ensuring parent fields are allocated before child fields are initialized.

```typescript
class BaseNotification {
  public timestamp: Date;

  constructor(public recipient: string) {
    this.timestamp = new Date();
    console.log("1. BaseNotification constructor executed.");
  }
}

class EmailNotification extends BaseNotification {
  public subject: string;

  constructor(recipient: string, subject: string) {
    // ERROR if you try to use `this.subject = subject` here before `super()`!

    super(recipient); // Pass required arguments to parent constructor
    
    console.log("2. EmailNotification constructor executed.");
    this.subject = subject; // Safe to use `this` now
  }
}

const email = new EmailNotification("alice@example.com", "System Alert");
```

---

### 2.2 Parameter Properties in Inherited Constructors

TypeScript allows parameter properties shorthand (`public`, `protected`, `private`, `readonly`) inside constructor parameters to automatically generate and assign instance variables.

```typescript
class BaseLogger {
  constructor(protected prefix: string) {}
}

class ConsoleLogger extends BaseLogger {
  // `prefix` is forwarded to `super`
  // `logLevel` uses shorthand parameter property declaration
  constructor(prefix: string, public logLevel: string) {
    super(prefix);
  }

  public log(message: string): void {
    console.log(`[${this.logLevel}] ${this.prefix}: ${message}`);
  }
}
```

---

### 2.3 Default & Omitted Constructors

If a derived class **does not** define a constructor, it implicitly inherits the parent class constructor and signature.

```typescript
class ParentConfig {
  constructor(public env: string, public debug: boolean = false) {}
}

class ChildConfig extends ParentConfig {
  // No explicit constructor defined here.
  // Implicitly acts as:
  // constructor(env: string, debug?: boolean) { super(env, debug); }
  
  public getMode(): string {
    return `Mode: ${this.env.toUpperCase()}`;
  }
}

const config = new ChildConfig("production", true);
console.log(config.getMode()); // Output: Mode: PRODUCTION
```

---

## 3. Method Overriding vs Method Overloading

It is important to distinguish between **Overriding** (changing runtime implementation in subclasses) and **Overloading** (providing multiple compile-time function signatures).

---

### 3.1 Method Overriding & The `override` Keyword

Method overriding occurs when a subclass redefines a method present in its superclass.

TypeScript provides the `override` keyword (and the `noImplicitOverride` compiler option in `tsconfig.json`) to prevent accidental overrides or typos.

```typescript
class PaymentProcessor {
  public process(amount: number): void {
    console.log(`Processing generic payment of $${amount}`);
  }
}

class StripeProcessor extends PaymentProcessor {
  // Safely overriding parent method
  public override process(amount: number): void {
    console.log("Validating Stripe API keys...");
    super.process(amount); // Optionally call parent behavior
  }

  // Compiler Error under `noImplicitOverride` if typo occurs:
  // public override proces(amount: number) { ... } -> Error: No base method 'proces' to override.
}
```

---

### 3.2 Method Overloading in Classes

Method overloading in TypeScript allows a single method to accept different parameter counts or type signatures. 

Unlike Java, TypeScript class overloading consists of:
1. **Overload Signatures** (compile-time type declarations).
2. **Implementation Signature** (single method containing the concrete logic).

```typescript
class DataFormatter {
  // Overload Signature 1: Accept Date object
  public format(value: Date): string;
  
  // Overload Signature 2: Accept Epoch timestamp number
  public format(value: number): string;
  
  // Overload Signature 3: Accept ISO string with optional timezone offset
  public format(value: string, timeZone?: string): string;

  // Concrete Implementation Signature (Must be compatible with all signatures)
  public format(value: Date | number | string, timeZone?: string): string {
    if (value instanceof Date) {
      return value.toISOString();
    }
    if (typeof value === "number") {
      return new Date(value).toISOString();
    }
    if (typeof value === "string") {
      const parsed = new Date(value);
      return timeZone ? `${parsed.toLocaleString()} (${timeZone})` : parsed.toISOString();
    }
    throw new Error("Invalid input format");
  }
}

const formatter = new DataFormatter();
console.log(formatter.format(1700000000000));
console.log(formatter.format("2026-10-24T12:00:00Z", "UTC"));
```

---

## 4. Advanced Concepts & Anti-Patterns

### Common Anti-Pattern: Fragile Base Class Problem
Avoid deeply nested inheritance hierarchies (e.g., 5+ levels deep). Changes in deep parent classes can unpredictably break downstream derived classes.

```typescript
// ❌ BAD: Over-engineered deep hierarchy
class Entity {}
class DbEntity extends Entity {}
class UserEntity extends DbEntity {}
class AccountUserEntity extends UserEntity {}
class AdminAccountUserEntity extends AccountUserEntity {}

// ✅ GOOD: Favor composition or shallow inheritance hierarchies
interface Authenticatable {
  authenticate(): boolean;
}

class AdminUser {
  constructor(private userDetails: UserEntity, private authService: Authenticatable) {}
}
```

---

## Summary Matrix: Inheritance Part 2

| Feature | Support in TS | Key Mechanics |
| :--- | :--- | :--- |
| **Single Inheritance** | ✅ Native | Implemented using `class Sub extends Base`. |
| **Multilevel Inheritance** | ✅ Native | Chains classes sequentially (`A -> B -> C`). |
| **Multiple Class Inheritance** | ❌ Not Direct | Solved via **Interfaces** or **Mixins**. |
| **Constructor Delegation** | ✅ Required | Derived constructors must call `super(...)` first. |
| **Method Overriding** | ✅ Native | Explicitly marked using the `override` keyword. |
| **Method Overloading** | ✅ Compile-Time | Declarative signatures mapped to one implementation block. |