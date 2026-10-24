# Object-Oriented Programming (OOP) in TypeScript: A Comprehensive Guide

---

## 1. Introduction: What is OOP vs. OOP in TypeScript?

### What is OOP?
**Object-Oriented Programming (OOP)** is a programming paradigm based on the concept of "objects," which contain data (in the form of properties/fields) and code (in the form of methods/functions). OOP helps structure software applications so that properties and behaviors are bundled into individual objects.

### What is OOP in TypeScript (TS)?
JavaScript (JS) is natively a prototype-based language. While ES6 introduced the `class` keyword, JavaScript's class structure is largely syntactical sugar over prototypical inheritance.

**TypeScript** elevates OOP in JavaScript by adding **compile-time type checking, explicit access modifiers, abstract classes, interfaces, and generic constraints**. 

#### Key Differences: JS vs. TS OOP
| Feature | JavaScript (ES6+) | TypeScript |
| :--- | :--- | :--- |
| **Access Modifiers** | Private fields `#field` (Runtime) | `public`, `private`, `protected`, `readonly` (Compile-time) |
| **Interfaces** | Not available | Native support for contract definition |
| **Abstract Classes** | Not supported directly | Fully supported (`abstract class` / `abstract method`) |
| **Type Safety** | Dynamic / Weak | Static / Strong |

---

## 2. Core Characteristics (Pillars) of OOP

OOP is defined by four fundamental pillars:

```
                  +-----------------------------------+
                  |   4 Pillars of OOP in TypeScript  |
                  +-----------------------------------+
                                    |
     +-----------------+------------+------------+-----------------+
     |                 |                         |                 |
     v                 v                         v                 v
+------------+  +--------------+          +--------------+  +---------------+
|  Encapsu-  |  | Abstraction  |          | Inheritance  |  | Polymorphism  |
|  lation    |  |              |          |              |  |               |
+------------+  +--------------+          +--------------+  +---------------+
| Bundling   |  | Hiding core  |          | Reusing code |  | Multiple forms|
| data &     |  | complexity   |          | from parent  |  | for a single  |
| restricting|  | behind clean |          | classes via  |  | interface or  |
| direct     |  | interfaces.  |          | `extends`.   |  | method name.  |
| access.    |  |              |          |              |  |               |
+------------+  +--------------+          +--------------+  +---------------+
```

### 1. Encapsulation
Bundling data and methods that operate on that data into a single unit (class) while restricting direct access to internal state using access modifiers (`private`, `protected`, `readonly`).

### 2. Abstraction
Hiding complex implementation details and showing only essential features of an object. Achieved using **Interfaces** and **Abstract Classes**.

### 3. Inheritance
Allowing a class (child) to acquire properties and methods of another class (parent) using the `extends` keyword, promoting code reuse.

### 4. Polymorphism
The ability of different classes to respond to the same method call in their own specific way (e.g., via Method Overriding or Interface implementation).

---

## 3. Real-Time Theoretical Scenarios & When to Use OOP

### When to Use OOP
* **Complex Enterprise Domains:** When modeling real-world entities with state, identity, and lifetime (e.g., User, Payment, Order, ShoppingCart).
* **Framework / Library Design:** When building reusable UI components, base testing frameworks, or custom data structures.
* **Service Layer Integration:** Managing API clients, WebSocket connection state, or persistent database interfaces.
* **Test Automation Frameworks:** Building Page Object Models (POM) and base test runners.

### When NOT to Use OOP (Prefer Functional Programming / FP)
* Pure data transformation pipelines (`map`, `filter`, `reduce`).
* Simple React UI views without complex local state logic.
* Serverless/Lambda functions where stateless execution is preferred.

---

## 4. Real-World Implementations in TypeScript

---

### Scenario A: Web Development with React (Stateful Services & Custom Hooks)

#### Context: Managing Authentication & API State
Using OOP to encapsulate authentication state and API execution, separate from React UI components.

```typescript
// 1. Interface (Abstraction)
export interface IUser {
  id: string;
  email: string;
  token: string;
}

export interface IAuthService {
  login(email: string, pass: string): Promise<IUser>;
  logout(): void;
  getUser(): IUser | null;
}

// 2. Concrete Service Class (Encapsulation + Polymorphism)
export class AuthService implements IAuthService {
  private currentUser: IUser | null = null;

  constructor(private readonly apiUrl: string) {}

  async login(email: string, pass: string): Promise<IUser> {
    // Simulated API call
    const response = await fetch(`${this.apiUrl}/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, pass }),
    });

    if (!response.ok) throw new Error('Authentication failed');

    const user: IUser = await response.json();
    this.currentUser = user;
    localStorage.setItem('auth_token', user.token);
    return user;
  }

  logout(): void {
    this.currentUser = null;
    localStorage.removeItem('auth_token');
  }

  getUser(): IUser | null {
    return this.currentUser;
  }
}

// 3. Usage inside React Component / Custom Hook
import React, { useState } from 'react';

const authService = new AuthService('https://api.example.com');

export const LoginButton: React.FC = () => {
  const [user, setUser] = useState<IUser | null>(authService.getUser());

  const handleLogin = async () => {
    try {
      const loggedInUser = await authService.login('user@test.com', 'secret');
      setUser(loggedInUser);
    } catch (err) {
      console.error(err);
    }
  };

  return (
    <div>
      {user ? <p>Welcome, {user.email}</p> : <button onClick={handleLogin}>Log In</button>}
    </div>
  );
};
```

---

### Scenario B: Automation Testing with Playwright (Page Object Model)

#### Context: Page Object Model (POM) Design Pattern
Demonstrating **Inheritance**, **Encapsulation**, and **Abstraction** for browser automation.

```typescript
import { Page, Locator, expect } from '@playwright/test';

// 1. Base Class (Abstraction & Inheritance)
export abstract class BasePage {
  protected readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  abstract navigate(): Promise<void>;

  async getTitle(): Promise<string> {
    return await this.page.title();
  }
}

// 2. Child Class - Derived Page Object
export class LoginPage extends BasePage {
  // Encapsulated locators
  private readonly usernameInput: Locator;
  private readonly passwordInput: Locator;
  private readonly submitButton: Locator;

  constructor(page: Page) {
    super(page); // Call parent constructor
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.submitButton = page.locator('button[type="submit"]');
  }

  async navigate(): Promise<void> {
    await this.page.goto('/login');
  }

  async login(user: string, pass: string): Promise<void> {
    await this.usernameInput.fill(user);
    await this.passwordInput.fill(pass);
    await this.submitButton.click();
  }
}

// 3. Playwright Test Implementation
import { test } from '@playwright/test';

test('User can log in successfully', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.navigate();
  await loginPage.login('standard_user', 'secret_sauce');
  await expect(page).toHaveURL('/dashboard');
});
```

---

### Scenario C: Backend APIs & Services (Repository & Service Pattern)

#### Context: Payment Processing System
Demonstrating **Polymorphism** and **Dependency Injection** using TypeScript Interfaces and Abstract Classes.

```typescript
// 1. Domain Interfaces
export interface IPaymentRequest {
  amount: number;
  currency: string;
}

export interface IPaymentResult {
  transactionId: string;
  status: 'SUCCESS' | 'FAILED';
}

// 2. Abstraction Interface for Strategy/Polymorphism
export interface IPaymentProcessor {
  processPayment(request: IPaymentRequest): Promise<IPaymentResult>;
}

// 3. Concrete Processor A (Stripe)
export class StripeProcessor implements IPaymentProcessor {
  async processPayment(req: IPaymentRequest): Promise<IPaymentResult> {
    console.log(`Processing $${req.amount} via Stripe...`);
    return { transactionId: `stripe_${Date.now()}`, status: 'SUCCESS' };
  }
}

// 4. Concrete Processor B (PayPal)
export class PayPalProcessor implements IPaymentProcessor {
  async processPayment(req: IPaymentRequest): Promise<IPaymentResult> {
    console.log(`Processing $${req.amount} via PayPal...`);
    return { transactionId: `paypal_${Date.now()}`, status: 'SUCCESS' };
  }
}

// 5. Payment Service using Dependency Injection
export class PaymentService {
  // Encapsulated dependency injection
  constructor(private processor: IPaymentProcessor) {}

  // Allows switching payment strategy at runtime
  setProcessor(processor: IPaymentProcessor) {
    this.processor = processor;
  }

  async checkout(amount: number): Promise<IPaymentResult> {
    return await this.processor.processPayment({ amount, currency: 'USD' });
  }
}

// Execution
async function run() {
  const stripe = new StripeProcessor();
  const paypal = new PayPalProcessor();

  const service = new PaymentService(stripe);
  await service.checkout(100); // Uses Stripe

  service.setProcessor(paypal);
  await service.checkout(200); // Uses PayPal
}
```

---

### Scenario D: Unit Testing Strategy (Mocking OOP Dependencies)

#### Context: Testing the Service Layer with Mock Objects

```typescript
// Unit Test using Jest / Vitest
import { describe, it, expect, vi } from 'vitest';

describe('PaymentService Unit Test', () => {
  it('should process payment using mocked processor', async () => {
    // Creating a mock implementation fulfilling the interface (Polymorphism)
    const mockProcessor: IPaymentProcessor = {
      processPayment: vi.fn().mockResolvedValue({
        transactionId: 'mock_tx_123',
        status: 'SUCCESS',
      }),
    };

    const paymentService = new PaymentService(mockProcessor);
    const result = await paymentService.checkout(500);

    expect(result.status).toBe('SUCCESS');
    expect(result.transactionId).toBe('mock_tx_123');
    expect(mockProcessor.processPayment).toHaveBeenCalledWith({
      amount: 500,
      currency: 'USD',
    });
  });
});
```

---

## 5. Summary Cheat Sheet

| OOP Pillar | TypeScript Key Syntax | Main Benefit | Real-World Use Case |
| :--- | :--- | :--- | :--- |
| **Encapsulation** | `private`, `protected`, `readonly`, getters/setters | Data protection & controlled access | Securing API keys, state inside Services or Page Objects |
| **Abstraction** | `interface`, `abstract class` | Reduces complexity & hides implementation | Defining API contracts, generic test pages |
| **Inheritance** | `class Child extends Parent` | Code reusability | Sharing setup logic across multiple Playwright Page Objects |
| **Polymorphism** | `implements Interface`, Method Overriding | Interchangeable components | Switching Payment Gateways or Database Connectors |