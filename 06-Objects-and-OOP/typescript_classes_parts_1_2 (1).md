# TypeScript Classes Guide

---

# Part 1: Basics, Properties & Modifiers

Welcome to Part 1 of the TypeScript Classes guide. This reference covers foundational concepts, syntax, and essential property modifiers with practical code examples.

## 1. Introduction to Classes

In Object-Oriented Programming (OOP), a **class** serves as a blueprint for creating objects. It encapsulates data (properties) and behavior (methods) into a reusable structure.

While JavaScript introduced `class` syntax in ES6 (ES2015), TypeScript enhances JavaScript classes by adding **static type annotations**, **access control**, **abstract classes**, and strict property initialization rules.

### Key Benefits of Classes in TypeScript:

* **Type Safety**: Enforces property types and method return types at compile time.
* **Encapsulation**: Restricts access to internal state using access modifiers.
* **Code Reusability**: Enables inheritance and polymorphic behavior.

## 2. Types of Classes in TypeScript

TypeScript supports several variations of classes based on how they are used and instantiated:

| Class Type | Description | Instantiable? | 
| ----- | ----- | ----- | 
| **Standard Class** | A typical class with properties and concrete methods. | Yes | 
| **Abstract Class** | A base class that cannot be instantiated directly and may contain abstract methods that derived classes must implement. | No | 
| **Generic Class** | A class that operates with type parameters to support reusable, type-safe structures. | Yes | 
| **Expression Class** | Anonymous or named classes assigned directly to variables. | Yes | 

## 3. Class Creation & Syntax

To create a basic class in TypeScript, use the `class` keyword followed by the class name, fields, constructor, and methods.

```typescript
class User {
  // 1. Property Declarations (Fields)
  id: number;
  username: string;

  // 2. Constructor: Executes when instantiating the class
  constructor(id: number, username: string) {
    this.id = id;
    this.username = username;
  }

  // 3. Method Definition
  getProfile(): string {
    return `User #${this.id}: ${this.username}`;
  }
}

// Instantiating the class using the 'new' keyword
const user1 = new User(101, "alex_dev");
console.log(user1.getProfile()); // Output: User #101: alex_dev
```

### Strict Property Initialization

When `strictPropertyInitialization` is enabled in `tsconfig.json`, TypeScript ensures every class property is initialized in the constructor or assigned a default value.

```typescript
class Product {
  name: string; // Error: Property 'name' has no initializer and is not definitely assigned in the constructor.
  
  constructor() {
    // Forgot to initialize this.name
  }
}
```

## 4. Class Properties

Properties represent the state of an object. TypeScript allows explicit typing for properties and supports default values.

### Default Property Values

Properties can be assigned default values directly at declaration:

```typescript
class Vehicle {
  make: string;
  speed: number = 0; // Default value assigned

  constructor(make: string) {
    this.make = make;
  }

  accelerate(amount: number): void {
    this.speed += amount;
  }
}

const car = new Vehicle("Toyota");
console.log(car.speed); // Output: 0
car.accelerate(50);
console.log(car.speed); // Output: 50
```

## 5. Readonly Properties

The `readonly` modifier prevents fields from being reassigned outside of the constructor. Once a `readonly` property is initialized (at declaration or in the constructor), it becomes immutable.

```typescript
class Config {
  readonly apiKey: string;
  readonly environment: string = "production"; // Readonly with default value

  constructor(key: string) {
    this.apiKey = key; // Allowed inside the constructor
  }

  updateKey(newKey: string) {
    // Error: Cannot assign to 'apiKey' because it is a read-only property.
    // this.apiKey = newKey; 
  }
}

const appConfig = new Config("secret-12345");
console.log(appConfig.apiKey); // Output: secret-12345

// Error: Cannot assign to 'apiKey' because it is a read-only property.
// appConfig.apiKey = "new-secret";
```

## 6. Optional Properties

Properties can be marked as optional using the `?` postfix operator. Optional properties are typed as `Type | undefined`.

```typescript
class CustomerProfile {
  id: number;
  email: string;
  phoneNumber?: string; // Optional property (string | undefined)

  constructor(id: number, email: string, phoneNumber?: string) {
    this.id = id;
    this.email = email;
    this.phoneNumber = phoneNumber;
  }

  getContactInfo(): string {
    if (this.phoneNumber) {
      return `Email: ${this.email}, Phone: ${this.phoneNumber}`;
    }
    return `Email: ${this.email}`;
  }
}

const customerA = new CustomerProfile(1, "alice@example.com");
const customerB = new CustomerProfile(2, "bob@example.com", "+1-555-0199");

console.log(customerA.getContactInfo()); // Output: Email: alice@example.com
console.log(customerB.getContactInfo()); // Output: Email: bob@example.com, Phone: +1-555-0199
```

---

# Part 2: Access Modifiers, Parameter Properties & Accessors

## 1. Access Modifiers

Access modifiers control visibility and accessibility of class members (properties and methods) from outside the class or in subclasses.

| Modifier | Access within Class | Access in Subclasses | Access from Instances |
| ----- | ----- | ----- | ----- |
| `public` *(default)* | ✅ Yes | ✅ Yes | ✅ Yes |
| `protected` | ✅ Yes | ✅ Yes | ❌ No |
| `private` | ✅ Yes | ❌ No | ❌ No |

### Example: Member Visibility

```typescript
class BankAccount {
  public accountHolder: string;
  protected balance: number;
  private pin: number;

  constructor(holder: string, initialBalance: number, pin: number) {
    this.accountHolder = holder;
    this.balance = initialBalance;
    this.pin = pin;
  }

  public getDetails(): string {
    return `${this.accountHolder} - Balance: $${this.balance}`;
  }

  private validatePin(inputPin: number): boolean {
    return this.pin === inputPin;
  }
}

class SavingsAccount extends BankAccount {
  public addInterest(rate: number): void {
    // Allowed: 'balance' is protected and accessible in derived classes
    this.balance += this.balance * rate; 
    
    // Error: 'pin' is private and accessible only within BankAccount
    // console.log(this.pin); 
  }
}

const account = new BankAccount("Sarah", 1000, 1234);
console.log(account.accountHolder); // Allowed (public)
// console.log(account.balance);    // Error: 'balance' is protected
// console.log(account.pin);        // Error: 'pin' is private
```

---

## 2. Parameter Properties

Parameter properties allow you to declare and initialize class fields directly within the constructor arguments, drastically reducing boilerplate code.

### Without Parameter Properties (Verbose):
```typescript
class Employee {
  public id: number;
  private salary: number;

  constructor(id: number, salary: number) {
    this.id = id;
    this.salary = salary;
  }
}
```

### With Parameter Properties (Shorthand):
```typescript
class Employee {
  // Prefixing arguments with access or readonly modifiers automatically creates and assigns class properties
  constructor(
    public id: number,
    private salary: number,
    readonly department: string = "Engineering"
  ) {}
}

const emp = new Employee(1, 85000);
console.log(emp.id);          // Output: 1
console.log(emp.department);  // Output: Engineering
```

---

## 3. Getters and Setters (Accessors)

Getters (`get`) and Setters (`set`) intercept property access and mutation, allowing validation, computed values, or side effects when reading or writing a field.

```typescript
class TemperatureSensor {
  private _celsius: number = 0;

  // Getter for Celsius
  get celsius(): number {
    return this._celsius;
  }

  // Setter for Celsius with validation
  set celsius(value: number) {
    if (value < -273.15) {
      throw new Error("Temperature below absolute zero is impossible!");
    }
    this._celsius = value;
  }

  // Computed property: Getter for Fahrenheit
  get fahrenheit(): number {
    return (this._celsius * 9) / 5 + 32;
  }
}

const sensor = new TemperatureSensor();
sensor.celsius = 25;              // Invokes setter
console.log(sensor.celsius);      // Output: 25 (Invokes getter)
console.log(sensor.fahrenheit);   // Output: 77 (Invokes computed getter)

// sensor.celsius = -300;         // Throws Error: Temperature below absolute zero is impossible!
```

---

## Summary Checklist (Part 1 & 2)

* \[x\] **Classes**: Core blueprints for objects in OOP.
* \[x\] **`readonly` & Optional (`?`)**: Ensure immutability or declare optional properties.
* \[x\] **Access Modifiers**: `public`, `protected`, and `private` enforce data visibility rules.
* \[x\] **Parameter Properties**: Clean shortcut to declare and assign fields inside `constructor(...)`.
* \[x\] **Getters & Setters**: Control and validate reads/writes to object properties.