# TypeScript Interfaces - Deep Dive (Part 4)

## Table of Contents

1. [Classes Implementing Interfaces (`implements`)](#1-classes-implementing-interfaces-implements)

   * [Single and Multiple Interface Implementation](#11-single-and-multiple-interface-implementation)

   * [Access Modifier Restrictions with `implements`](#12-access-modifier-restrictions-with-implements)

   * [Property Initialization & Parameter Properties](#13-property-initialization--parameter-properties)

2. [Interface vs. Class: Comprehensive Comparison](#2-interface-vs-class-comprehensive-comparison)

   * [Feature Comparison Matrix](#21-feature-comparison-matrix)

   * [Compilation & Runtime Behavior](#22-compilation--runtime-behavior)

3. [When to Use Which: Architectural Guidelines](#3-when-to-use-which-architectural-guidelines)

   * [Use Cases for Interfaces](#31-use-cases-for-interfaces)

   * [Use Cases for Classes](#32-use-cases-for-classes)

   * [Combining Interfaces and Classes (Best Practices)](#33-combining-interfaces-and-classes-best-practices)

4. [Decision Flowchart](#4-decision-flowchart)

5. [Summary Checklist](#5-summary-checklist)

## 1. Classes Implementing Interfaces (`implements`)

An interface defines a contract, and a class uses the `implements` keyword to guarantee that it provides concrete implementations for all members declared in that interface.

```
+-----------------------------------------------------------------+
|                    CLASS IMPLEMENTATION MODEL                   |
+-----------------------------------------------------------------+

       [ Interface: Printable ]        [ Interface: Encryptable ]
       +--------------------+          +------------------------+
       | print(): void      |          | encrypt(): string      |
       +--------------------+          +------------------------+
                 \                        /
                  \                      /
               +------------------------------+
               | class SecureDocument         |
               |   implements Printable,      |
               |              Encryptable     |
               +------------------------------+
               | print(): void { ... }        |
               | encrypt(): string { ... }    |
               +------------------------------+

```

### 1.1 Single and Multiple Interface Implementation

A class can implement a single interface or multiple interfaces simultaneously by separating them with commas.

```
interface Repository<T> {
  findById(id: string): T | undefined;
  save(entity: T): void;
}

interface Auditable {
  lastUpdated: Date;
  updatedBy: string;
}

export interface User {
  id: string;
  name: string;
}

// Class implementing multiple interfaces
export class UserRepository implements Repository<User>, Auditable {
  public lastUpdated: Date = new Date();
  public updatedBy: string = "system";
  private storage: Map<string, User> = new Map();

  public findById(id: string): User | undefined {
    return this.storage.get(id);
  }

  public save(entity: User): void {
    this.lastUpdated = new Date();
    this.storage.set(entity.id, entity);
  }
}

```

### 1.2 Access Modifier Restrictions with `implements`

When a class implements an interface member:

1. The implemented property or method **MUST be `public`**.

2. You cannot mark an interface-defined property as `protected` or `private` inside the implementing class.

```
interface Authenticatable {
  authenticate(token: string): boolean;
}

class AuthService implements Authenticatable {
  // ❌ ERROR: Class property 'authenticate' cannot be private/protected
  // because interface 'Authenticatable' requires it to be public.
  // protected authenticate(token: string): boolean { ... }

  // ✅ CORRECT: Implemented interface method must be public
  public authenticate(token: string): boolean {
    return token.length > 0;
  }
}

```

### 1.3 Property Initialization & Parameter Properties

Implementing an interface does not automatically infer property types or supply default values inside the class constructor; you must explicitly declare parameter types and initialize fields in the class.

```
interface PaymentConfig {
  apiKey: string;
  timeoutMs: number;
}

class StripeConfig implements PaymentConfig {
  // Shorthand parameter properties automatically assign and type public members
  constructor(
    public apiKey: string,
    public timeoutMs: number = 5000
  ) {}
}

```

## 2. Interface vs. Class: Comprehensive Comparison

Understanding the structural and operational differences between an `interface` and a `class` is critical for designing maintainable TypeScript architectures.

### 2.1 Feature Comparison Matrix

| Aspect / Feature | `interface` | `class` |
| ----- | ----- | ----- |
| **Primary Purpose** | Defines pure type contracts and object structures. | Blueprint for instantiating concrete objects with logic and state. |
| **Runtime Existence** | ❌ **Erased entirely** during transpilation. | ✅ **Emitted as a JavaScript function/class**. |
| **Instantiation (`new`)** | ❌ Cannot be instantiated using `new`. | ✅ Can be instantiated (`new MyClass()`). |
| **Method Implementation** | ❌ Contains no execution logic (signatures only). | ✅ Contains concrete execution code and logic. |
| **Fields / Properties** | ❌ Declares property names and types only. | ✅ Can declare, initialize, and assign state variables. |
| **Access Modifiers** | ❌ All declared members are implicitly public. | ✅ Supports `public`, `protected`, `private`, and `#field`. |
| **Inheritance Support** | Extends multiple interfaces (`extends A, B`). | Extends a single parent class (`extends Parent`). |
| **Type Checking** | Pure compile-time shape verification. | Provides both a compile-time type AND a runtime constructor. |

### 2.2 Compilation & Runtime Behavior

#### TypeScript Source Code

```
// Interface definition
interface PersonInterface {
  name: string;
  greet(): string;
}

// Class definition
class PersonClass implements PersonInterface {
  constructor(public name: string) {}

  greet(): string {
    return `Hello, I am ${this.name}`;
  }
}

```

#### Generated JavaScript (Transpiled Output)

```
"use strict";
// Notice: PersonInterface has completely vanished!

class PersonClass {
    constructor(name) {
        this.name = name;
    }
    greet() {
        return `Hello, I am ${this.name}`;
    }
}

```

## 3. When to Use Which: Architectural Guidelines

### 3.1 Use Cases for Interfaces

Use an **`interface`** when:

1. **Defining Pure Contracts / DTOs:** Modeling payload shapes for APIs, JSON files, or database queries where no logic or initialization is required.

2. **Decoupling System Components:** Defining abstract boundary definitions (e.g., repository signatures, service boundaries) that can be swapped or mocked in unit tests.

3. **Defining Public APIs or Libraries:** Creating shared contracts that external consumers can extend via declaration merging.

4. **Zero-Overhead Typing:** Enforcing structural type checks without increasing JavaScript bundle sizes.

```
// Excellent Interface Use Case: Data Transfer Object (DTO)
export interface CreateOrderDTO {
  readonly customerId: string;
  readonly items: Array<{ productId: string; quantity: number }>;
  readonly shippingAddress: string;
}

```

### 3.2 Use Cases for Classes

Use a **`class`** when:

1. **State & Behavior Encapsulation:** Creating objects that maintain internal state and expose methods to operate on that state.

2. **Instantiating Objects dynamically:** Needing constructors to initialize properties, run validation on creation, or construct instances using `new`.

3. **Runtime Reflection / Validation:** Needing `instanceof` checks at runtime (e.g., domain errors, custom error handling).

4. **Code Reuse via Inheritance:** Sharing concrete execution logic and parent class helper methods across subclasses.

```
// Excellent Class Use Case: Domain Entity with encapsulation and methods
export class BankAccount {
  private balance: number = 0;

  constructor(public readonly accountId: string, initialDeposit: number) {
    if (initialDeposit < 0) throw new Error("Initial deposit cannot be negative.");
    this.balance = initialDeposit;
  }

  public deposit(amount: number): void {
    if (amount <= 0) throw new Error("Deposit amount must be positive.");
    this.balance += amount;
  }

  public getBalance(): number {
    return this.balance;
  }
}

```

### 3.3 Combining Interfaces and Classes (Best Practices)

In enterprise TypeScript applications, the most robust architecture combines both: use **Interfaces** to define abstract system contracts and **Classes** to provide concrete implementations.

```
// 1. Contract Definition
export interface CacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T, ttlSeconds?: number): Promise<void>;
}

// 2. Concrete Redis Implementation
export class RedisCacheService implements CacheService {
  public async get<T>(key: string): Promise<T | null> {
    // Concrete Redis fetch logic
    return null;
  }

  public async set<T>(key: string, value: T, ttlSeconds?: number): Promise<void> {
    // Concrete Redis write logic
  }
}

// 3. Concrete In-Memory Implementation for Testing
export class InMemoryCacheService implements CacheService {
  private store = new Map<string, any>();

  public async get<T>(key: string): Promise<T | null> {
    return this.store.get(key) ?? null;
  }

  public async set<T>(key: string, value: T): Promise<void> {
    this.store.set(key, value);
  }
}

```

## 4. Decision Flowchart

```
                            Do you need runtime code emission
                           or instance creation using `new`?
                                      /        \
                                    YES         NO
                                    /            \
          Do you need to initialize state      Do you need to express
          or encapsulate runtime logic?        unions, tuples, or primitives?
                      /         \                      /         \
                    YES          NO                  YES          NO
                    /             \                  /             \
             [ Use CLASS ]   [ Abstract Class ]  [ Use TYPE ]  [ Use INTERFACE ]

```

## 5. Summary Checklist

* \[ \] **`implements` Keyword:** Used by classes to fulfill an interface contract without generating runtime inheritance code.

* \[ \] **Visibility Rule:** Members specified by an interface must be declared as `public` in the implementing class.

* \[ \] **Type Erasure:** Interfaces vanish during compilation; classes persist into JS runtime constructors.

* \[ \] **Architecture Rule:** Use **Interfaces** for data shapes, boundary contracts, and API definitions; use **Classes** for domain state, behavior encapsulation, and runtime object creation.