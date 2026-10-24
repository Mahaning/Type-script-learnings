# TypeScript Inheritance - Deep Dive (Part 1)

## Table of Contents

1. [Introduction to Inheritance & Theoretical Foundations](#1-introduction-to-inheritance--theoretical-foundations)
2. [Core Terminology & Structural Definitions](#2-core-terminology--structural-definitions)
3. [In-Depth Comparison: Java vs. TypeScript vs. JavaScript](#3-in-depth-comparison-java-vs-typescript-vs-javascript)
4. [Access Modifiers & Encapsulation Rules](#4-access-modifiers--encapsulation-rules)
5. [Implementing a Parent Class (Superclass)](#5-implementing-a-parent-class-superclass)
6. [Creating a Child Class (Subclass) & Mechanics of `super`](#6-creating-a-child-class-subclass--mechanics-of-super)
7. [Under the Hood: Transpilation & Prototypal Chain](#7-under-the-hood-transpilation--prototypal-chain)
8. [Practical Design Pattern: Base Repository Pattern](#8-practical-design-pattern-base-repository-pattern)
9. [Summary & Comprehensive Checklist](#9-summary--comprehensive-checklist)

---

## 1. Introduction to Inheritance & Theoretical Foundations

Inheritance is one of the four foundational pillars of Object-Oriented Programming (OOP), alongside **Encapsulation**, **Abstraction**, and **Polymorphism**.

In Object-Oriented Design, inheritance represents an **"IS-A"** relationship between objects:
* A `Dog` **IS-A** `Animal`.
* A `CheckingAccount` **IS-A** `BankAccount`.
* A `Student` **IS-A** `Person`.

### Why Use Inheritance?

1. **Code Reusability & DRY Principle:** Common fields and behaviors are consolidated into a single parent class, eliminating redundancy across specialized classes.
2. **Polymorphic Behavior:** Functions can accept instances of a parent class and seamlessly operate on any child class instance without needing custom logic for each derivative type.
3. **Hierarchical Modeling:** Provides a natural structure for domain modeling that matches real-world hierarchies.
4. **Centralized Maintenance:** Updates to shared behavior happen in one place (the superclass), propagating down to all subclasses.

---

## 2. Core Terminology & Structural Definitions

| Term | Technical Definition |
| :--- | :--- |
| **Superclass / Parent Class / Base Class** | The class whose members (properties and methods) are inherited by another class. |
| **Subclass / Child Class / Derived Class** | The class that inherits from the superclass and can extend or override inherited behavior. |
| **`extends`** | The keyword establishing an explicit prototype-based connection between a derived class and its parent. |
| **`super`** | A keyword used within derived classes to call superclass constructors (`super(...)`) or access parent methods (`super.methodName()`). |
| **Method Overriding** | The mechanism by which a child class redefines a method implementation that it inherited from a parent class. |
| **Parameter Properties** | A TypeScript feature allowing shorthand property declaration and assignment directly inside constructor signatures. |

---

## 3. In-Depth Comparison: Java vs. TypeScript vs. JavaScript

Understanding how TypeScript handles classes requires contrasting its approach with traditional class-based languages (like Java) and its underlying runtime execution context (JavaScript).

| Feature | Java | TypeScript (TS) | JavaScript (ES6+) |
| :--- | :--- | :--- | :--- |
| **Type System** | Nominally typed at runtime and compile time. | Structurally typed at compile time; erased at runtime. | Dynamic and weak typing at runtime. |
| **Underlying Engine** | Class-based execution engine on the JVM. | Transpiles to JavaScript prototypal inheritance chains. | Prototypal delegation (`Object.create`, `prototype`). |
| **Access Control** | Enforced strictly at runtime by JVM (`public`, `private`, `protected`, package-private). | Enforced **only at compile time** by TS compiler (`public`, `private`, `protected`). | Private fields starting with `#` are private at runtime; no `protected`. |
| **Multiple Inheritance** | Not supported for classes; interface implementation allowed. | Not supported for classes; interface implementation and mixins allowed. | Not supported natively; achieved via mixins. |
| **Method Overloading** | Multiple method declarations with different parameters in runtime bytecode. | Multiple type signatures with a single implementation block. | Single function implementation inspecting `arguments`. |
| **Constructor Execution** | `super()` must be invoked before executing child constructor code. | `super()` **must** be called before accessing `this` in child constructor. | `super()` **must** be called before accessing `this` in child constructor. |

---

## 4. Access Modifiers & Encapsulation Rules

TypeScript adds visibility semantics to class members that control where property or method references are legal.

### Access Modifier Visibility Matrix

| Access Modifier | Inside Own Class | Inside Child Class | Outside Class (Instance Level) | Emitted to JS Output? |
| :--- | :---: | :---: | :---: | :---: |
| **`public`** (Default) | ✅ Yes | ✅ Yes | ✅ Yes | Plain property |
| **`protected`** | ✅ Yes | ✅ Yes | ❌ No | Plain property |
| **`private`** (TS keyword) | ✅ Yes | ❌ No | ❌ No | Plain property |
| **`#` (ES Private Field)** | ✅ Yes | ❌ No | ❌ No | Enforced at runtime `#field` |

> **Crucial Difference (`private` vs `#`):**
> * TS `private` is a **compile-time check only**. Once transpiled, the property is a standard JavaScript object key accessible at runtime.
> * JavaScript `#field` uses ECMAScript hard privacy, enforcing true runtime isolation.

---

## 5. Implementing a Parent Class (Superclass)

When designing a base class, declare fields with the appropriate visibility (`protected` or `public`) and provide explicit types and default behavior.

```typescript
/**
 * Represents a generic user account in the system.
 */
export class UserAccount {
  // Public property: accessible everywhere
  public username: string;

  // Protected property: accessible within UserAccount and derived classes
  protected email: string;

  // Private property: accessible only within UserAccount
  private failedLoginAttempts: number = 0;

  /**
   * Shorthand Parameter Property constructor syntax can also be used.
   */
  constructor(username: string, email: string) {
    this.username = username;
    this.email = email;
  }

  /**
   * Public method available to all callers.
   */
  public getAccountInfo(): string {
    return `User: ${this.username} | Email: ${this.maskEmail()}`;
  }

  /**
   * Protected method available to subclasses for customization.
   */
  protected maskEmail(): string {
    const parts = this.email.split("@");
    if (parts.length !== 2) return "******";
    const [local, domain] = parts;
    return `${local.charAt(0)}***@${domain}`;
  }

  /**
   * Private internal utility.
   */
  private recordFailedAttempt(): void {
    this.failedLoginAttempts++;
    console.warn(`Account ${this.username} recorded failed login attempt: ${this.failedLoginAttempts}`);
  }
}
```

---

## 6. Creating a Child Class (Subclass) & Mechanics of `super`

To derive a class from a base class, use the `extends` keyword.

### Important Rules for Subclasses:
1. If the subclass defines a `constructor`, it **MUST call `super(...)`** before accessing `this` or returning from the constructor.
2. The argument types passed to `super(...)` must match the parent class constructor signature.
3. Subclasses can override parent methods using the `override` keyword (enabled via TS `noImplicitOverride` compiler flag).

```typescript
import { UserAccount } from "./UserAccount";

/**
 * Derived class representing an Admin user with permission levels.
 */
export class AdminUser extends UserAccount {
  public permissions: string[];
  private superAdmin: boolean;

  constructor(
    username: string,
    email: string,
    permissions: string[],
    isSuperAdmin: boolean = false
  ) {
    // 1. MUST invoke super before accessing 'this'
    super(username, email);

    // 2. Subclass property initializations
    this.permissions = permissions;
    this.superAdmin = isSuperAdmin;
  }

  /**
   * Overriding the parent display method to include admin metadata.
   */
  public override getAccountInfo(): string {
    // Using super to call the parent class implementation
    const baseInfo = super.getAccountInfo();
    return `[ADMIN] ${baseInfo} | Permissions Count: ${this.permissions.length}`;
  }

  /**
   * Specialized subclass method accessing protected property 'email' from parent class.
   */
  public sendAdminAlert(message: string): void {
    // Accessing protected parent property 'this.email'
    console.log(`Alert dispatched to ${this.email}: "${message}"`);
  }
}
```

---

## 7. Under the Hood: Transpilation & Prototypal Chain

TypeScript classes do not create classical classes at runtime; they compile down to standard JavaScript prototypal inheritance mechanisms.

### TypeScript Source
```typescript
class Animal {
  constructor(public name: string) {}
  speak(): void { console.log(`${this.name} makes a noise.`); }
}

class Dog extends Animal {
  speak(): void { console.log(`${this.name} barks.`); }
}
```

### Transpiled ES5 Output (Simplified Concept)
When targeted to older JS standards (e.g., ES5), TypeScript emits helper code to wire up prototype objects manually:

```javascript
var __extends = (this && this.__extends) || (function () {
    var extendStatics = function (d, b) {
        Object.setPrototypeOf(d, b);
    };
    return function (d, b) {
        extendStatics(d, b);
        function __() { this.constructor = d; }
        d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
    };
})();

var Animal = /** @class */ (function () {
    function Animal(name) {
        this.name = name;
    }
    Animal.prototype.speak = function () {
        console.log(this.name + " makes a noise.");
    };
    return Animal;
})();

var Dog = /** @class */ (function (_super) {
    __extends(Dog, _super);
    function Dog(name) {
        return _super.call(this, name) || this;
    }
    Dog.prototype.speak = function () {
        console.log(this.name + " barks.");
    };
    return Dog;
})(Animal);
```

### Prototype Chain Hierarchy
When calling `dog.speak()`:
1. JavaScript checks if `speak` exists directly on the `dog` instance object.
2. If not found, it checks `Dog.prototype`.
3. If overridden, it executes `Dog.prototype.speak`.
4. If not overridden, it traverses up `Object.getPrototypeOf(Dog.prototype)` to find `Animal.prototype.speak`.

---

## 8. Practical Design Pattern: Base Repository Pattern

Inheritance shines when creating foundation abstractions that enforce consistent operational contracts across application layers, such as database repositories.

```typescript
export interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

/**
 * Generic Base Class for CRUD Operations
 */
export abstract class BaseRepository<T extends BaseEntity> {
  protected items: Map<string, T> = new Map();

  public save(item: T): T {
    item.updatedAt = new Date();
    this.items.set(item.id, item);
    return item;
  }

  public findById(id: string): T | undefined {
    return this.items.get(id);
  }

  public delete(id: string): boolean {
    return this.items.delete(id);
  }

  // Abstract declaration forces subclasses to implement validation logic
  public abstract validate(item: T): boolean;
}

/**
 * Concrete Subclass Implementation
 */
export interface Product extends BaseEntity {
  name: string;
  price: number;
}

export class ProductRepository extends BaseRepository<Product> {
  public validate(item: Product): boolean {
    return item.name.length > 0 && item.price > 0;
  }

  // Specialized query method exclusive to ProductRepository
  public findByPriceRange(min: number, max: number): Product[] {
    return Array.from(this.items.values()).filter(
      p => p.price >= min && p.price <= max
    );
  }
}
```

---

## 9. Summary & Comprehensive Checklist

When working with TypeScript Inheritance Part 1, keep this mental checklist handy:

- [ ] **Inheritance Keyword:** Use `extends` to declare a subclass relationship.
- [ ] **Constructor Requirement:** Invoke `super(...args)` in the child constructor **prior** to referencing `this`.
- [ ] **Encapsulation:** Use `protected` for members meant to be shared only down the inheritance tree, and `private` / `#` for members exclusive to the declaring class.
- [ ] **Type Erasure:** Remember that TS visibility modifiers exist purely at compile time; runtime JavaScript respects standard object property behaviors unless using `#` private identifiers.
- [ ] **Method Overriding:** Use the `override` keyword on child methods to protect against accidental signature drifts when parent classes change.