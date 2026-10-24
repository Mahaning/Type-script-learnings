# TypeScript Objects — Part 3: Advanced Types, Classes & Object Independence

Welcome to Part 3 of the TypeScript Objects guide! In this module, we dive into structural type modeling, building objects with Classes and Constructors, and understanding object memory models (reference types vs. object independence).

---

## 1. Inline Object Types

An **inline object type** allows you to define an object's structural shape directly inside a variable declaration, function parameter, or return type signature without explicitly naming the type beforehand.

### Usage in Variable Declarations and Functions

```typescript
// Inline type in variable declaration
let serverConfig: { host: string; port: number; ssl: boolean } = {
  host: "localhost",
  port: 8080,
  ssl: true
};

// Inline type in function parameter
function startServer(config: { host: string; port: number; ssl: boolean }): void {
  console.log(`Server starting at ${config.ssl ? "https" : "http"}://${config.host}:${config.port}`);
}

startServer(serverConfig);
```

> **When to use:** Inline types are best for one-off structures or quick functions. If a shape is reused across multiple places, prefer using Type Aliases or Interfaces.

---

## 2. Using Type Aliases

A **Type Alias** (`type`) allows you to define a reusable name for any type, including object shapes. It makes code cleaner, reduces duplication, and makes refactoring simpler.

### Syntax & Example

```typescript
// Defining a Type Alias
type Product = {
  id: string;
  name: string;
  price: number;
  tags?: string[]; // Optional property
};

// Using the Type Alias
const laptop: Product = {
  id: "p-100",
  name: "Ultrabook 15",
  price: 1299.99
};

function displayProduct(item: Product): string {
  return `${item.name} - $${item.price}`;
}

console.log(displayProduct(laptop));
```

---

## 3. Intersection Types (`&`)

Intersection types allow you to combine multiple object types into a single type. An object of an intersection type will contain all properties from every combined type.

### Combining Object Types

```typescript
type Timestamps = {
  createdAt: Date;
  updatedAt: Date;
};

type Identifiable = {
  id: string;
};

type UserProfile = {
  username: string;
  email: string;
};

// Combining multiple types into a single object shape using &
type UserRecord = Identifiable & UserProfile & Timestamps;

const activeUser: UserRecord = {
  id: "usr-001",
  username: "johndoe",
  email: "john@example.com",
  createdAt: new Date("2024-01-01"),
  updatedAt: new Date()
};
```

---

## 4. Using Classes and Class Constructors

In TypeScript, **Classes** act as blueprints for creating objects with shared structure and functionality. They support Object-Oriented Programming (OOP) principles like encapsulation, inheritance, and type checks.

### Class Constructors

A `constructor` is a special method called automatically when creating an instance of a class. It initializes object properties.

```typescript
class BankAccount {
  // Public & private property definitions
  readonly accountNumber: string;
  private balance: number;
  public ownerName: string;

  // Constructor function
  constructor(accountNumber: string, ownerName: string, initialBalance: number) {
    this.accountNumber = accountNumber;
    this.ownerName = ownerName;
    this.balance = initialBalance;
  }

  // Class Methods
  public getBalance(): number {
    return this.balance;
  }

  public deposit(amount: number): void {
    if (amount > 0) {
      this.balance += amount;
    }
  }
}
```

### Parameter Properties (Shorthand Syntax)

TypeScript offers a shortcut to automatically declare and assign class properties directly from constructor parameters:

```typescript
class Vehicle {
  // Shorthand initialization
  constructor(
    public readonly vin: string,
    public make: string,
    public model: string,
    private mileage: number = 0
  ) {}

  public drive(miles: number): void {
    this.mileage += miles;
  }
}
```

---

## 5. Creating Class Instances

To instantiate an object from a class blueprint, use the `new` keyword. This calls the class's constructor and returns a new object instance.

```typescript
// Creating instances using 'new'
const account1 = new BankAccount("ACC-001", "Alice", 1000);
const account2 = new BankAccount("ACC-002", "Bob", 500);

account1.deposit(250);
console.log(account1.getBalance()); // Output: 1250
console.log(account2.getBalance()); // Output: 500
```

Classes in TypeScript double as both value constructors and **type identifiers**:

```typescript
// BankAccount can be used as a Type annotation
function logAccountOwner(account: BankAccount): void {
  console.log(`Owner: ${account.ownerName}`);
}

logAccountOwner(account1);
```

---

## 6. Object Independence (Reference vs. Value)

In JavaScript and TypeScript, primitives (`number`, `string`, `boolean`) are copied **by value**, whereas objects (including arrays and class instances) are handled **by reference**.

### Reference Behavior (Shared Mutation)

Assigning an existing object to a new variable does **not** create a new independent object—it creates a new reference pointing to the exact same memory location.

```typescript
const original = { name: "Alpha", value: 10 };

// Copying reference, NOT the actual object
const referenceCopy = original;

referenceCopy.value = 99;

// Both variables reflect the change!
console.log(original.value);     // 99
console.log(referenceCopy.value); // 99
```

### Achieving Object Independence (Shallow vs. Deep Copies)

To make objects independent from one another, you must explicitly copy their data into a new object structure.

#### 1. Shallow Copy (`Object.assign` or Spread Operator `{...}`)
Copies top-level properties. Nested objects still share references!

```typescript
const originalUser = {
  name: "Charlie",
  settings: { theme: "dark" }
};

// Shallow clone
const shallowClone = { ...originalUser };

shallowClone.name = "David"; // Independent top-level property
shallowClone.settings.theme = "light"; // ⚠️ Still mutates originalUser.settings!

console.log(originalUser.name);           // "Charlie" (Independent)
console.log(originalUser.settings.theme); // "light" (Shared reference!)
```

#### 2. Deep Copy (`structuredClone`)
Creates a fully independent copy, including all nested objects and structures.

```typescript
const originalSettings = {
  id: 1,
  preferences: { display: "grid", sound: true }
};

// Deep copy using modern built-in structuredClone
const deepClone = structuredClone(originalSettings);

deepClone.preferences.display = "list";

console.log(originalSettings.preferences.display); // "grid" (Fully independent!)
console.log(deepClone.preferences.display);        // "list"
```

---

## Summary Reference

| Concept | Key Syntax / Keyword | Primary Benefit |
| :--- | :--- | :--- |
| **Inline Type** | `{ key: type }` | Quick, single-use parameter or variable typing. |
| **Type Alias** | `type Name = { ... }` | Reusable, clean object structure definition. |
| **Intersection** | `TypeA & TypeB` | Merges multiple types into a single combined type. |
| **Class Instance** | `new ClassName(...)` | Creates an object instance bound to a class blueprint. |
| **Reference Assignment** | `const b = a` | Shared memory location; changes affect both variables. |
| **Deep Copy** | `structuredClone(obj)` | Full object independence across all nested properties. |