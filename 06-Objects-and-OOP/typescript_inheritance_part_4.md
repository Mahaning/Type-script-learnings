# TypeScript Inheritance - Deep Dive (Part 4)

## Table of Contents

1. [Object Creation & Memory Access Mechanics](#1-object-creation--memory-access-mechanics)
   * [Instantiation Sequence](#11-instantiation-sequence)
   * [Reference Variables vs Object Instances](#12-reference-variables-vs-object-instances)
2. [Parent Class Variable Types & Polymorphism](#2-parent-class-variable-types--polymorphism)
   * [Polymorphic Reference Assignments](#21-polymorphic-reference-assignments)
   * [Accessing Parent vs Child Members via Parent Reference](#22-accessing-parent-vs-child-members-via-parent-reference)
   * [Type Narrowing & Downcasting](#23-type-narrowing--downcasting)
3. [The `super` Keyword: Exhaustive Usage Rules](#3-the-super-keyword-exhaustive-usage-rules)
   * [Constructor Calls (`super(...)`)](#31-constructor-calls-super)
   * [Method Calls (`super.methodName()`)](#32-method-calls-supermethodname)
   * [Property Access Limitations](#33-property-access-limitations)
4. [Access Modifiers in Inheritance & Property Shadowing](#4-access-modifiers-in-inheritance--property-shadowing)
   * [Detailed Accessibility Matrix](#41-detailed-accessibility-matrix)
   * [Access Modifier Widening Rules on Overriding](#42-access-modifier-widening-rules-on-overriding)
   * [Property Shadowing vs Method Overriding](#43-property-shadowing-vs-method-overriding)
5. [Summary Checklist](#5-summary-checklist)

---

## 1. Object Creation & Memory Access Mechanics

### 1.1 Instantiation Sequence

When creating an instance of a child class (`new SubClass()`), TypeScript/JavaScript executes initialization in a strict 5-step sequence:

```
+-----------------------------------------------------------------------+
|                       INSTANTIATION SEQUENCE                          |
+-----------------------------------------------------------------------+

 1. Memory Allocation: Allocate object on the Heap.
       |
       v
 2. Parent Field Initialization: Parent instance properties initialized.
       |
       v
 3. Parent Constructor Execution: Parent `constructor()` runs via `super()`.
       |
       v
 4. Child Field Initialization: Child instance properties initialized.
       |
       v
 5. Child Constructor Execution: Remaining child `constructor()` body runs.
```

### Code Example: Execution Order Verification

```typescript
class BaseComponent {
  public baseProp = (() => {
    console.log("1. Base property initialized");
    return "Base Property";
  })();

  constructor() {
    console.log("2. Base constructor executed");
  }
}

class DerivedComponent extends BaseComponent {
  public derivedProp = (() => {
    console.log("3. Derived property initialized");
    return "Derived Property";
  })();

  constructor() {
    super(); // Must be called first!
    console.log("4. Derived constructor executed");
  }
}

// Instantiation
const comp = new DerivedComponent();
```

**Console Output:**
```text
1. Base property initialized
2. Base constructor executed
3. Derived property initialized
4. Derived constructor executed
```

---

### 1.2 Reference Variables vs Object Instances

In TypeScript, a variable's **type declaration** determines what properties are accessible at compile time, while the **object instance** determines what behavior runs at runtime.

```typescript
class Vehicle {
  public brand: string = "Generic Vehicle";

  public start(): void {
    console.log("Vehicle engine starting...");
  }
}

class ElectricCar extends Vehicle {
  public override brand: string = "Tesla";
  public batteryCapacity: number = 100;

  public override start(): void {
    console.log("Silent electric motor starting...");
  }

  public charge(): void {
    console.log("Charging battery...");
  }
}

// Reference Type: Vehicle | Object Type: ElectricCar
const myVehicle: Vehicle = new ElectricCar();

// Compile-Time check: Constrained by Reference Type `Vehicle`
console.log(myVehicle.brand); // "Tesla" (Property value from instance)
myVehicle.start();            // "Silent electric motor starting..." (Runtime polymorphic method)

// COMPILER ERROR: Property 'charge' does not exist on type 'Vehicle'.
// myVehicle.charge(); 
```

---

## 2. Parent Class Variable Types & Polymorphism

### 2.1 Polymorphic Reference Assignments

Because a child class **IS-A** parent class, a variable typed as the parent class can hold references to any derived class instance.

```typescript
class Shape {
  public draw(): void {
    console.log("Drawing a generic shape.");
  }
}

class Circle extends Shape {
  public override draw(): void {
    console.log("Drawing a Circle: ◯");
  }
}

class Square extends Shape {
  public override draw(): void {
    console.log("Drawing a Square: ▢");
  }
}

// Array of Parent References holding different Child Instances
const shapes: Shape[] = [new Shape(), new Circle(), new Square()];

shapes.forEach((shape: Shape) => {
  // Polymorphic Execution: Dispatches to exact instance method at runtime
  shape.draw();
});
```

---

### 2.2 Accessing Parent vs Child Members via Parent Reference

When an instance is assigned to a parent reference variable:

1. **Declared Parent Members:** Fully accessible.
2. **Overridden Parent Members:** Executed using child's implementation at runtime.
3. **Child-Exclusive Members:** Inaccessible unless type downcasting or type narrowing is applied.

```typescript
class Account {
  constructor(public accountNumber: string, public balance: number) {}

  public getDetails(): string {
    return `Account #${this.accountNumber} - Balance: $${this.balance}`;
  }
}

class SavingsAccount extends Account {
  constructor(accountNumber: string, balance: number, public interestRate: number) {
    super(accountNumber, balance);
  }

  public addInterest(): void {
    this.balance += this.balance * (this.interestRate / 100);
  }
}

const account: Account = new SavingsAccount("ACC-1001", 5000, 2.5);

console.log(account.getDetails()); // Accessible (Defined in Parent)

// account.addInterest(); 
// ❌ Error: 'addInterest' does not exist on type 'Account'.
```

---

### 2.3 Type Narrowing & Downcasting

To call child-exclusive methods on a parent reference variable, narrow the type using `instanceof` or dynamic casting (`as`).

```typescript
// Technique 1: Safe Type Narrowing using `instanceof`
if (account instanceof SavingsAccount) {
  // TypeScript automatically narrows `account` to `SavingsAccount` inside this block
  account.addInterest(); // ✅ Allowed!
  console.log(`Updated Interest Rate: ${account.interestRate}%`);
}

// Technique 2: Explicit Downcasting using `as` Keyword (Use with caution)
const explicitSavings = account as SavingsAccount;
explicitSavings.addInterest(); // ✅ Allowed!
```

---

## 3. The `super` Keyword: Exhaustive Usage Rules

The `super` keyword serves two primary purposes in derived classes:
1. Calling the parent constructor: `super(...)`
2. Accessing parent class methods: `super.methodName(...)`

```
+-------------------------------------------------------------------------+
|                         SUPER KEYWORD RULES                             |
+-------------------------------------------------------------------------+
|  Context                 | Usage Syntax       | Direct Restrictions    |
+--------------------------+--------------------+-------------------------+
|  Child Constructor       | super(...args)     | MUST be called before   |
|                          |                    | accessing `this`        |
+--------------------------+--------------------+-------------------------+
|  Child Instance Method   | super.methodName() | Calls parent version    |
|                          |                    | of method               |
+--------------------------+--------------------+-------------------------+
|  Child Property Access   | super.propertyName | ❌ Returns `undefined`  |
|                          |                    | for instance properties |
+--------------------------+--------------------+-------------------------+
```

### 3.1 Constructor Calls (`super(...)`)

* If a subclass has a `constructor`, calling `super(...)` is **mandatory**.
* `super()` initializes parent fields and prototype connections.

```typescript
class Animal {
  constructor(public name: string, public age: number) {}
}

class Cat extends Animal {
  public livesLeft: number;

  constructor(name: string, age: number, livesLeft: number = 9) {
    // ❌ Error if accessed before super(): console.log(this.livesLeft);
    
    super(name, age); // Delegate initialization to Animal constructor
    
    this.livesLeft = livesLeft; // ✅ Allowed after super()
  }
}
```

---

### 3.2 Method Calls (`super.methodName()`)

Subclasses can invoke overridden methods on parent classes using `super.methodName()`.

```typescript
class BaseService {
  public connect(): void {
    console.log("Establishing secure TLS database socket connection...");
  }
}

class CacheService extends BaseService {
  public override connect(): void {
    // 1. Execute parent class implementation
    super.connect(); 

    // 2. Execute additional derived class logic
    console.log("Warming up Redis in-memory cache layer...");
  }
}

const service = new CacheService();
service.connect();
```

---

### 3.3 Property Access Limitations

> ⚠️ **CRITICAL GOTCHA:** In JavaScript and TypeScript, `super.propertyName` **cannot** be used to access instance variables declared on the parent class. It resolves properties on the **prototype object**, where instance variables do not reside.

```typescript
class Parent {
  public instanceVar: string = "Parent Data";
}

class Child extends Parent {
  public printData(): void {
    console.log(this.instanceVar);  // ✅ Prints: "Parent Data"
    console.log(super.instanceVar); // ❌ Prints: undefined (instance variables live on `this`, not `prototype`)
  }
}

new Child().printData();
```

---

## 4. Access Modifiers in Inheritance & Property Shadowing

### 4.1 Detailed Accessibility Matrix

TypeScript supports three access modifiers (`public`, `protected`, `private`) alongside ECMAScript runtime private fields (`#field`).

| Access Modifier | Declaring Class | Derived Subclass | Instance Level (External) | Runtime Visibility in JS |
| :--- | :---: | :---: | :---: | :--- |
| `public` | ✅ | ✅ | ✅ | Plain Object Key |
| `protected` | ✅ | ✅ | ❌ | Plain Object Key |
| `private` | ✅ | ❌ | ❌ | Plain Object Key |
| `#field` (ES Private) | ✅ | ❌ | ❌ | Hard Runtime Isolation |

```typescript
class BaseAccess {
  public pubVar: string = "Public: Accessible everywhere";
  protected protVar: string = "Protected: Accessible in child classes";
  private privVar: string = "Private: Accessible ONLY in BaseAccess";
  #esPrivVar: string = "Hard Private: Runtime protected";

  public printBase(): void {
    console.log(this.privVar);   // ✅ Allowed
    console.log(this.#esPrivVar); // ✅ Allowed
  }
}

class SubAccess extends BaseAccess {
  public printSub(): void {
    console.log(this.pubVar);   // ✅ Allowed
    console.log(this.protVar);  // ✅ Allowed
    
    // console.log(this.privVar); 
    // ❌ Error: Property 'privVar' is private and only accessible within class 'BaseAccess'.

    // console.log(this.#esPrivVar); 
    // ❌ Error: Property '#esPrivVar' is not accessible outside class 'BaseAccess'.
  }
}
```

---

### 4.2 Access Modifier Widening Rules on Overriding

When overriding a parent method in a child class, you can **widen** visibility (e.g., `protected` to `public`), but you **cannot narrow** visibility (e.g., `public` to `private`).

```typescript
class ParentPermissions {
  protected calculateHash(): string {
    return "hash_v1_xyz";
  }

  public render(): void {
    console.log("Rendering...");
  }
}

class ChildPermissions extends ParentPermissions {
  // ✅ VALID: Widening visibility from `protected` to `public`
  public override calculateHash(): string {
    return "hash_v2_abc";
  }

  // ❌ INVALID: Cannot narrow visibility from `public` to `private` or `protected`
  // protected override render(): void { ... }
  // Error: Class 'ChildPermissions' incorrectly extends base class 'ParentPermissions'.
  // Property 'render' is protected in type 'ChildPermissions' but public in type 'ParentPermissions'.
}
```

---

### 4.3 Property Shadowing vs Method Overriding

* **Method Overriding:** Uses prototype chain resolution. A child class method replaces parent class method behavior polymorphically.
* **Property Shadowing:** Occurs when a child class redeclares an instance property with the same name as a parent class instance property. The child's instance value directly overwrites/shadows the parent's value on the instance object.

```typescript
class ParentConfig {
  public apiEndpoint: string = "https://api.v1.domain.com";

  public printEndpoint(): void {
    console.log(`Endpoint: ${this.apiEndpoint}`);
  }
}

class ChildConfig extends ParentConfig {
  // Shadowing the property 'apiEndpoint'
  public override apiEndpoint: string = "https://api.v2.domain.com";
}

const config = new ChildConfig();
config.printEndpoint(); // Output: "Endpoint: https://api.v2.domain.com"
```

---

## 5. Summary Checklist

- [ ] **Instantiation Order:** Base instance properties -> Base constructor -> Derived instance properties -> Derived constructor.
- [ ] **Type vs Instance:** Reference types constrain compile-time member availability; instance types govern runtime polymorphic behavior.
- [ ] **Downcasting Safety:** Always use `instanceof` narrowing before calling child-exclusive members on parent reference variables.
- [ ] **`super()` Mechanics:** Must call `super()` first in child constructors before accessing `this`. Use `super.method()` to access parent methods, but avoid using `super` for instance variables.
- [ ] **Visibility Rules:** Subclass overrides can widen visibility (`protected` → `public`), but can never narrow it (`public` → `private`).