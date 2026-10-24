# TypeScript Classes - Part 2: Constructors, Methods & Undefined Data

Part 2 explores how constructors initialize instance properties, how to build and call instance methods, implementing display methods, and handling `undefined` or missing values cleanly.

---

## 1. Class Constructors

The `constructor` is a special method automatically called when a new instance of a class is created using the `new` operator. Its primary role is to set up the initial state of an object.

- TypeScript allows only **one** constructor per class.
- Constructor parameters can have types, default values, or be optional.

```typescript
class Book {
  title: string;
  pages: number;

  // Constructor receiving initialization data
  constructor(title: string, pages: number = 100) {
    this.title = title;
    this.pages = pages;
  }
}

const book1 = new Book("TypeScript Essentials", 250);
const book2 = new Book("Quick Starter"); // Uses default pages = 100
```

---

## 2. Assigning Constructor Data

When instantiating a class, data passed into the constructor parameters must be explicitly assigned to instance properties using the `this` keyword.

```typescript
class Student {
  studentId: number;
  fullName: string;
  gradeLevel: string;

  constructor(id: number, name: string, grade: string) {
    // Assigning incoming parameter data to class fields
    this.studentId = id;
    this.fullName = name;
    this.gradeLevel = grade;
  }
}

const student = new Student(1001, "Jordan Lee", "10th Grade");
```

### Constructor Parameter Properties (Shorthand)

TypeScript provides a shortcut syntax to declare and assign fields automatically by prefixing constructor parameters with access modifiers (`public`, `private`, `protected`, or `readonly`):

```typescript
class CompactStudent {
  // Automatically creates properties and assigns values passed to constructor
  constructor(
    public studentId: number,
    public fullName: string,
    public gradeLevel: string
  ) {}
}
```

---

## 3. Instance Methods

Instance methods define behaviors for objects created from the class. They have access to internal instance properties via `this`.

```typescript
class Account {
  balance: number;

  constructor(initialBalance: number) {
    this.balance = initialBalance;
  }

  // Instance Method: Deposit funds
  deposit(amount: number): number {
    this.balance += amount;
    return this.balance;
  }

  // Instance Method: Withdraw funds
  withdraw(amount: number): boolean {
    if (amount <= this.balance) {
      this.balance -= amount;
      return true;
    }
    return false;
  }
}

const myAccount = new Account(500);
myAccount.deposit(200); // balance is now 700
```

---

## 4. Implementing Display Methods

Display methods (often named `display()`, `toString()`, or `printSummary()`) format internal object state into human-readable strings or logs.

```typescript
class ProductItem {
  id: number;
  title: string;
  price: number;

  constructor(id: number, title: string, price: number) {
    this.id = id;
    this.title = title;
    this.price = price;
  }

  // Display Method
  display(): string {
    return `[ID: ${this.id}] ${this.title} - $${this.price.toFixed(2)}`;
  }

  // Console output helper method
  printDetails(): void {
    console.log(this.display());
  }
}

const product = new ProductItem(42, "Wireless Mouse", 29.99);
console.log(product.display()); // Output: [ID: 42] Wireless Mouse - $29.99
```

---

## 5. Handling Undefined Properties

In TypeScript, optional parameters or fields marked with `?` can hold `undefined`. It is essential to safely check for `undefined` inside methods before accessing or calling methods on those properties.

### Techniques for Safe Handling:

1. **If-Statement Checks**: Basic conditional validation.
2. **Nullish Coalescing Operator (`??`)**: Provides fallback defaults for `undefined` or `null`.
3. **Optional Chaining (`?.`)**: Safely calls methods or accesses nested fields.

```typescript
class UserProfile {
  username: string;
  bio?: string;          // Optional (string | undefined)
  websiteUrl?: string;   // Optional (string | undefined)

  constructor(username: string, bio?: string, websiteUrl?: string) {
    this.username = username;
    this.bio = bio;
    this.websiteUrl = websiteUrl;
  }

  // Display method safely handling optional/undefined values
  displayProfile(): void {
    console.log(`User: ${this.username}`);

    // 1. Handling undefined using Nullish Coalescing (??)
    const bioText = this.bio ?? "No bio provided.";
    console.log(`Bio: ${bioText}`);

    // 2. Handling undefined using Optional Chaining (?.) and If Checks
    if (this.websiteUrl) {
      const cleanUrl = this.websiteUrl.toLowerCase().trim();
      console.log(`Website: ${cleanUrl}`);
    } else {
      console.log("Website: N/A");
    }
  }
}

const userWithBio = new UserProfile("dev_jane", "Full-stack enthusiast", "https://jane.dev");
const sparseUser = new UserProfile("guest_user");

sparseUser.displayProfile();
// Output:
// User: guest_user
// Bio: No bio provided.
// Website: N/A
```

---

## Summary Checklist (Part 2)

- [x] **Constructor & Data Assignment**: Use constructors to initialize object fields on instantiation.
- [x] **Instance Methods**: Create functions inside classes that access properties using `this`.
- [x] **Display Methods**: Build dedicated methods to format object data cleanly for output.
- [x] **Handling Undefined**: Safely guard optional properties using `??`, `?.`, and conditional checks.