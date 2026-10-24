# TypeScript Inheritance - Deep Dive (Part 5)

## Table of Contents

1. [Real-World Enterprise Examples](#1-real-world-enterprise-examples)
   * [Example 1: Payment Gateway Integration (Single Inheritance)](#example-1-payment-gateway-integration-single-inheritance)
   * [Example 2: Multi-Tiered User Authorization (Multilevel Inheritance)](#example-2-multi-tiered-user-authorization-multilevel-inheritance)
   * [Example 3: UI Component Framework (Hierarchical Inheritance)](#example-3-ui-component-framework-hierarchical-inheritance)
   * [Example 4: Logging & Auditing Capabilities (Mixins / Hybrid Inheritance)](#example-4-logging--auditing-capabilities-mixins--hybrid-inheritance)
2. [Which Inheritance Pattern to Use: Decision Framework](#2-which-inheritance-pattern-to-use-decision-framework)
   * [Pattern Selection Matrix](#pattern-selection-matrix)
   * [Architectural Flowchart](#architectural-flowchart)
3. [When NOT to Use Inheritance](#3-when-not-to-use-inheritance)
4. [Summary & Complete Series Conclusion](#4-summary--complete-series-conclusion)

---

## 1. Real-World Enterprise Examples

### Example 1: Payment Gateway Integration (Single Inheritance)

Single inheritance works best when subclasses share a strict contract and core processing steps, but vary in external API integrations.

```typescript
export interface PaymentResult {
  success: boolean;
  transactionId: string;
  errorMessage?: string;
}

// Parent Class: Shared foundation for all payment providers
export abstract class PaymentProvider {
  constructor(protected apiKey: string, protected merchantId: string) {}

  // Shared validation step across all providers
  protected validateAmount(amount: number): void {
    if (amount <= 0) {
      throw new Error("Payment amount must be greater than zero.");
    }
  }

  // Abstract declaration forcing concrete implementations
  public abstract processPayment(amount: number, currency: string): Promise<PaymentResult>;
}

// Child Class: Specific Stripe implementation
export class StripeProvider extends PaymentProvider {
  constructor(apiKey: string, merchantId: string, private stripeAccountId: string) {
    super(apiKey, merchantId);
  }

  public async processPayment(amount: number, currency: string): Promise<PaymentResult> {
    this.validateAmount(amount);
    
    // Simulate Stripe API call
    console.log(`Connecting to Stripe Account ${this.stripeAccountId}...`);
    return {
      success: true,
      transactionId: `ch_stripe_${Math.random().toString(36).substring(7)}`,
    };
  }
}
```

---

### Example 2: Multi-Tiered User Authorization (Multilevel Inheritance)

Multilevel inheritance (`BaseUser` → `StaffMember` → `SystemAdmin`) is ideal when permission levels accumulate hierarchically.

```typescript
export class BaseUser {
  constructor(public userId: string, public email: string) {}

  public getPermissions(): string[] {
    return ["READ_PROFILE", "UPDATE_OWN_PROFILE"];
  }
}

export class StaffMember extends BaseUser {
  constructor(
    userId: string,
    email: string,
    public department: string
  ) {
    super(userId, email);
  }

  public override getPermissions(): string[] {
    // Accumulate parent permissions and add staff-specific capabilities
    return [...super.getPermissions(), "READ_INTERNAL_DOCS", "SUBMIT_TICKETS"];
  }
}

export class SystemAdmin extends StaffMember {
  constructor(
    userId: string,
    email: string,
    department: string,
    public superAdminToken: string
  ) {
    super(userId, email, department);
  }

  public override getPermissions(): string[] {
    // Accumulate staff permissions and add full admin capabilities
    return [...super.getPermissions(), "MANAGE_USERS", "SYSTEM_SHUTDOWN"];
  }
}
```

---

### Example 3: UI Component Framework (Hierarchical Inheritance)

Hierarchical inheritance is well-suited for UI libraries or DOM abstractions where multiple distinct components share base layout, rendering, and event-handling traits.

```typescript
export abstract class UIComponent {
  constructor(public id: string, public isVisible: boolean = true) {}

  public show(): void {
    this.isVisible = true;
    this.render();
  }

  public hide(): void {
    this.isVisible = false;
    this.render();
  }

  public abstract render(): void;
}

export class ButtonComponent extends UIComponent {
  constructor(id: string, public label: string, public onClick: () => void) {
    super(id);
  }

  public render(): void {
    if (!this.isVisible) return;
    console.log(`[UI Component] <button id="${this.id}">${this.label}</button>`);
  }
}

export class ModalComponent extends UIComponent {
  constructor(id: string, public title: string, public content: string) {
    super(id, false); // Modals start hidden by default
  }

  public render(): void {
    if (!this.isVisible) return;
    console.log(`[UI Component] <dialog id="${this.id}"><h3>${this.title}</h3><p>${this.content}</p></dialog>`);
  }
}
```

---

### Example 4: Logging & Auditing Capabilities (Mixins / Hybrid Inheritance)

When you need to add pluggable behaviors across unrelated class hierarchies without forcing deep class trees, use **Mixins**.

```typescript
type Constructor<T = {}> = new (...args: any[]) => T;

// Mixin 1: Adds Auditing capabilities
export function WithAuditing<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    public createdAt: Date = new Date();
    public lastModifiedAt: Date = new Date();

    public markUpdated(): void {
      this.lastModifiedAt = new Date();
    }
  };
}

// Mixin 2: Adds Soft-Delete capabilities
export function WithSoftDelete<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    public isDeleted: boolean = false;
    public deletedAt?: Date;

    public softDelete(): void {
      this.isDeleted = true;
      this.deletedAt = new Date();
    }
  };
}

// Unrelated Domain Entities acquiring shared capabilities
class OrderEntity {
  constructor(public orderId: string, public amount: number) {}
}

class UserEntity {
  constructor(public username: string) {}
}

// Composed Classes
export const AuditedOrder = WithSoftDelete(WithAuditing(OrderEntity));
export const AuditedUser = WithAuditing(UserEntity);

// Usage
const order = new AuditedOrder("ORD-9912", 250.0);
order.markUpdated();
order.softDelete();
console.log(`Order Deleted At: ${order.deletedAt}`);
```

---

## 2. Which Inheritance Pattern to Use: Decision Framework

### Pattern Selection Matrix

| Scenario / Use Case | Recommended Pattern | Primary Reason |
| :--- | :--- | :--- |
| **API Clients & Payment Gateways** | Single Inheritance | Shared base configuration and standard execution lifecycle. |
| **Multi-level Access Controls / Roles** | Multilevel Inheritance | Gradual layer-by-layer accumulation of traits and capabilities. |
| **UI Design Systems / Component Trees** | Hierarchical Inheritance | Common lifecycle (`render`, `hide`) with specialized view elements. |
| **Cross-Cutting Concerns (Logging, Caching)** | Mixins / Interfaces | Avoids rigid base class pollution across unrelated classes. |
| **Stateful Workflow Pipelines** | Abstract Class + Template Method | Guarantees strict ordering of steps while delegating implementation. |

---

### Architectural Flowchart

```
                 Do classes share an "IS-A" relationship?
                               /         \
                             YES          NO  --> [ Use Composition / Dependency Injection ]
                             /
        Is behavior cross-cutting across unrelated classes?
                           /     \
                         YES      NO
                         /         \
       [ Use Mixins / Interfaces ]  How many parent classes are needed?
                                            /          \
                                       SINGLE          MULTIPLE
                                         /                \
                    Does depth increase linearly?      [ Combine Interfaces & Abstract Classes ]
                              /          \
                            YES           NO
                            /              \
            [ Multilevel Inheritance ]   [ Single or Hierarchical Inheritance ]
```

---

## 3. When NOT to Use Inheritance

While inheritance is powerful, overusing it leads to fragile codebases. Avoid inheritance in the following situations:

1. **For Code Sharing Alone:** Do not inherit from a class just to borrow a couple of utility methods. Use composition or utility functions instead.
2. **Deep Class Hierarchies (Depth > 3):** Inheritance hierarchies deeper than 3 levels make debugging state and overriding logic difficult.
3. **Dynamic Behavioral Changes:** If an object needs to change its behavior dynamically at runtime (e.g., changing from `FastShipping` to `ExpressShipping`), use the **Strategy Pattern** via composition instead of extending classes.

---

## 4. Summary & Complete Series Conclusion

Throughout this 5-part series, we covered:
* **Part 1:** Core definitions, access modifier fundamentals, and Java vs. JS vs. TS differences.
* **Part 2:** Inheritance variants, `super()` execution order, and method overloading vs. overriding.
* **Part 3:** V8 engine internals, prototypal memory chain execution, and enterprise design patterns.
* **Part 4:** Memory instantiation phases, variable reference typing, polymorphic dispatch, and property shadowing.
* **Part 5:** Concrete enterprise implementations, pattern selection frameworks, and composition vs. inheritance boundaries.