# TypeScript Objects — Part 2: Methods, Accessing, and Modifying Data

Welcome to Part 2 of the TypeScript Objects guide! In this section, we build upon the basics to explore how to add methods (functions) to objects, access object properties using different notations, and safely modify object values within TypeScript's type system.

## 1. Adding Methods to Objects

An object property whose value is a function is called a **method**. Methods define behavior or actions that an object can perform, often interacting with the object's own data using the `this` keyword.

### Syntax and Implementation

You can define methods in an object literal using either traditional function expressions, method shorthand, or arrow functions.

```
// 1. Defining method types explicitly in an object interface/type
type BankAccount = {
    accountNumber: string;
    balance: number;
    // Method type declarations
    deposit(amount: number): void;
    withdraw: (amount: number) => boolean; 
    getSummary(): string;
};

// 2. Creating the object implementing the methods
const myAccount: BankAccount = {
    accountNumber: "ACC-98765",
    balance: 500,

    // Shorthand method syntax
    deposit(amount: number) {
        this.balance += amount;
        console.log(`Deposited: $${amount}. New Balance: $${this.balance}`);
    },

    // Arrow function syntax (Note: arrow functions do NOT bind their own 'this')
    withdraw: function (amount: number): boolean {
        if (amount <= this.balance) {
            this.balance -= amount;
            return true;
        }
        return false;
    },

    getSummary() {
        return `Account ${this.accountNumber} has a balance of $${this.balance}.`;
    }
};

// Executing object methods
myAccount.deposit(150); // Deposited: $150. New Balance: $650
console.log(myAccount.getSummary()); // Account ACC-98765 has a balance of $650.

```

> **Note on `this`:** Be cautious when using arrow functions as object methods, as they capture the `this` value of the enclosing lexical context rather than the object itself.

## 2. Accessing Object Data

TypeScript provides two primary ways to access object properties: **Dot Notation** and **Bracket Notation**.

### Dot Notation (`object.property`)

Dot notation is the standard, most common way to read values from an object when the property name is known ahead of time and is a valid identifier.

```
const product = {
    id: "prod-01",
    title: "Mechanical Keyboard",
    price: 89.99
};

// Accessing properties via dot notation
console.log(product.title); // Output: Mechanical Keyboard
console.log(product.price); // Output: 89.99

```

## 3. Bracket Notation (`object["property"]`)

Bracket notation allows you to access properties using a string expression. This is particularly useful in specific scenarios where dot notation is insufficient.

### When to Use Bracket Notation:

1. **Dynamic Property Names:** When the key is stored in a variable or determined at runtime.

2. **Keys with Special Characters or Spaces:** When property keys contain spaces, hyphens, or numbers at the beginning.

```
const userSettings = {
    "theme-color": "dark",
    "enable notifications": true,
    fontScale: 1.2
};

// 1. Accessing keys with special characters/spaces
console.log(userSettings["theme-color"]); // "dark"
console.log(userSettings["enable notifications"]); // true

// 2. Accessing properties dynamically using a variable
const dynamicKey = "fontScale";
console.log(userSettings[dynamicKey]); // 1.2

```

### Type Safety with Dynamic Keys (`keyof`)

To safely access dynamic properties in TypeScript without implicit `any` errors, use the `keyof` operator:

```
type Product = {
    name: string;
    price: number;
    inStock: boolean;
};

const item: Product = {
    name: "Headphones",
    price: 150,
    inStock: true
};

function getProperty(obj: Product, key: keyof Product) {
    return obj[key]; // Type-safe access!
}

const price = getProperty(item, "price"); // Returns 150

```

## 4. Modifying Object Values

Once an object is instantiated, its properties can be updated, extended, or protected against modification depending on how it was typed and declared.

### Updating Existing Values

You can mutate property values using either dot or bracket notation. TypeScript will ensure the new value matches the declared property type.

```
const car = {
    brand: "Tesla",
    speed: 0
};

// Updating values
car.speed = 60; // ✅ Valid
car["speed"] = 80; // ✅ Valid

// ❌ TypeScript Error: Type 'string' is not assignable to type 'number'
// car.speed = "80 mph"; 

```

### Readonly Properties

If you want to prevent modification of certain object values after creation, use the `readonly` modifier in your type definition.

```
type Configuration = {
    readonly apiKey: string; // Cannot be reassigned
    environment: string;
};

const config: Configuration = {
    apiKey: "SECRET_KEY_123",
    environment: "production"
};

config.environment = "development"; // ✅ Allowed

// ❌ TypeScript Error: Cannot assign to 'apiKey' because it is a read-only property.
// config.apiKey = "NEW_KEY"; 

```

### Modifying Objects Declared with `const`

A common source of confusion is declaring objects with `const`. Declaring an object with `const` prevents **reassigning the object variable itself**, but it **does not freeze the interior properties**.

```
const profile = {
    username: "coder123"
};

// ✅ Modifying properties inside a const object is permitted
profile.username = "dev_guru"; 

// ❌ Reassigning the entire object variable is forbidden
// profile = { username: "new_user" }; // Error: Assignment to constant variable.

```

To completely prevent property modifications at runtime, combine TypeScript types with `Object.freeze()`:

```
const immutableUser = Object.freeze({
    id: 1,
    role: "admin"
} as const);

// ❌ Error at runtime & compile time
// immutableUser.role = "editor"; 

```

## Summary Checklist

| Action | Syntax | Example | 
| ----- | ----- | ----- | 
| **Method Definition** | `key: () => ReturnType` | `greet(): void { console.log("Hi"); }` | 
| **Dot Notation** | `obj.key` | `user.name` | 
| **Bracket Notation** | `obj["key"]` | `user["theme-color"]` | 
| **Dynamic Access** | `obj[variableKey]` | `user[keyName]` | 
| **Prevent Mutations** | `readonly` / `Object.freeze` | `readonly id: number;` | 
