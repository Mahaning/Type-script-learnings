# TypeScript Objects — Part 1: Fundamentals & Type Annotations

Welcome to Part 1 of the TypeScript Objects guide! In this module, we will explore the core concepts of objects in JavaScript/TypeScript, how key-value pairs structure data, real-world examples, different ways to instantiate objects, and how to define and apply object type annotations to ensure strong type safety in your applications.

---

## 1. Introduction to Objects

In JavaScript and TypeScript, an **object** is a fundamental data structure used to store collections of related data and functionality. Unlike primitive types (`string`, `number`, `boolean`, etc.) which store a single value, an object allows you to group multiple values together as named properties.

Objects form the backbone of almost all complex applications—representing entities such as Users, Products, Shopping Carts, Configuration settings, and API responses.

---

## 2. Object Key-Value Pairs

An object is made up of **properties**. Each property consists of a **key** (also called a property name) and a **value**.

- **Key**: A string or symbol identifier that names the property.
- **Value**: Any valid TypeScript type, including numbers, strings, booleans, arrays, functions (methods), or even other objects (nested objects).

### Syntax Structure:

```typescript
const objectName = {
    key1: value1,
    key2: value2,
    key3: value3
};
```

---

## 3. Real-World Example

Imagine building an e-commerce application. Instead of managing separate variable items like `userFirstName`, `userLastName`, `userAge`, and `userIsPremium`, you can encapsulate all this information into a single `user` object.

```typescript
const user = {
    id: 101,
    firstName: "Alex",
    lastName: "Morgan",
    email: "alex.morgan@example.com",
    isPremiumAccount: true,
    address: {
        city: "Seattle",
        zipCode: 98101
    }
};

// Accessing properties using dot notation
console.log(user.firstName); // Output: Alex
console.log(user.address.city); // Output: Seattle
```

---

## 4. Ways to Create Objects

In TypeScript, there are several ways to create objects depending on your use case:

### 1. Object Literal Syntax (Most Common)
Directly defining the object structure using curly braces `{}`.

```typescript
const laptop = {
    brand: "Dell",
    ramInGB: 16,
    isSSD: true
};
```

### 2. Using the `new Object()` Constructor
Using the built-in JavaScript `Object` constructor (less common in modern codebases).

```typescript
const book = new Object();
book.title = "TypeScript Deep Dive";
book.pages = 350;
```

### 3. Using `Object.create()`
Creates a new object using an existing object as the prototype.

```typescript
const personPrototype = {
    greet() {
        console.log("Hello!");
    }
};

const john = Object.create(personPrototype);
john.name = "John";
john.greet(); // Output: Hello!
```

### 4. Constructor Functions or ES6 Classes
Useful when creating multiple instances of the same object blueprint.

```typescript
class Car {
    make: string;
    model: string;

    constructor(make: string, model: string) {
        this.make = make;
        this.model = model;
    }
}

const myCar = new Car("Toyota", "Corolla");
```

---

## 5. Object Types in TypeScript

TypeScript allows you to specify the **shape** of an object using explicit type annotations. This ensures that an object must have specific properties with specified types.

### Syntax for Object Type Annotation:

```typescript
let variableName: { key1: Type1; key2: Type2 };
```

### Implicit (Inferred) Object Types
When you create an object literal, TypeScript automatically infers its shape:

```typescript
const product = {
    id: 1,
    name: "Wireless Mouse",
    price: 29.99
};

// TypeScript inferred product as: { id: number; name: string; price: number; }
```

### Explicit Object Type Annotation

You can explicitly declare what properties an object must contain:

```typescript
const employee: {
    id: number;
    name: string;
    department: string;
    isFullTime: boolean;
} = {
    id: 502,
    name: "Sarah Jenkins",
    department: "Engineering",
    isFullTime: true
};
```

---

## 6. Using Object Types

Explicitly specifying object types helps prevent runtime errors, catches missing properties, and avoids typos during compile time.

### Preventing Invalid Assignments

If you try to assign an incorrect data type or miss a required property, TypeScript raises a compile-time error:

```typescript
let player: {
    username: string;
    score: number;
};

// ❌ ERROR: Type 'string' is not assignable to type 'number'.
player = {
    username: "GamerOne",
    score: "one hundred" 
};

// ❌ ERROR: Property 'score' is missing in type '{ username: string; }'
player = {
    username: "GamerOne"
};

// ✅ CORRECT
player = {
    username: "GamerOne",
    score: 100
};
```

### Passing Object Types to Functions

You can also annotate object types in function parameter lists to enforce incoming argument structures:

```typescript
function printUserProfile(profile: { name: string; age: number; email: string }): void {
    console.log(`Name: ${profile.name}, Age: ${profile.age}, Email: ${profile.email}`);
}

const newUser = {
    name: "Elena",
    age: 28,
    email: "elena@example.com"
};

printUserProfile(newUser);
```

---

## Key Takeaways

1. **Objects** group related values using key-value pairs.
2. **Object Literal Syntax** (`{}`) is the standard and most readable way to create objects.
3. **TypeScript Object Types** define the mandatory structure (shape) of an object.
4. Using explicit type definitions provides **autocomplete**, **early error detection**, and **code maintainability**.

---

*Stay tuned for Part 2: Optional Properties, Readonly Properties, Type Aliases, Interfaces, and Index Signatures!*