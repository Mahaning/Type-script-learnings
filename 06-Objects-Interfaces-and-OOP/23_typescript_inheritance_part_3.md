# TypeScript Inheritance - Deep Dive (Part 3)

## Table of Contents

1. [Internal Working & Runtime Engine Mechanics](#1-internal-working--runtime-engine-mechanics)
   * [Prototypal Delegation & Memory Layout](#11-prototypal-delegation--memory-layout)
   * [Method Resolution Order (MRO)](#12-method-resolution-order-mro)
   * [Property Lookup Flowchart](#13-property-lookup-flowchart)
2. [Practical Enterprise Usage Patterns](#2-practical-enterprise-usage-patterns)
   * [Pattern 1: Abstract Base Service with Template Method Pattern](#pattern-1-abstract-base-service-with-template-method-pattern)
   * [Pattern 2: Polymorphic Command Pipeline](#pattern-2-polymorphic-command-pipeline)
   * [Pattern 3: Custom Error Hierarchy for Domain-Driven Design](#pattern-3-custom-error-hierarchy-for-domain-driven-design)
3. [Inheritance vs. Composition: Real-World Refactoring](#3-inheritance-vs-composition-real-world-refactoring)
   * [When to Prefer Inheritance](#when-to-prefer-inheritance)
   * [When to Prefer Composition](#when-to-prefer-composition)
   * [Refactoring Example](#refactoring-example)
4. [Summary & Complete Series Recap](#4-summary--complete-series-recap)

---

## 1. Internal Working & Runtime Engine Mechanics

While TypeScript provides classical `class` syntax, the JavaScript runtime engine (V8, JavaScriptCore, SpiderMonkey) executes inheritance using **prototypal delegation** and **prototype chain traversal**.

### 1.1 Prototypal Delegation & Memory Layout

When an object instance is created in TypeScript via `new ChildClass()`, the engine allocates distinct memory references for instance fields versus prototype methods.

```
+-------------------------------------------------------------------------+
|                         MEMORY LAYOUT DIAGRAM                           |
+-------------------------------------------------------------------------+

  [ Instance: myDog ]
  +-----------------------+
  | name: "Rover"         |  (Instance Property - Heap)
  | breed: "Labrador"     |  (Instance Property - Heap)
  | [[Prototype]] --------+---> [ Dog.prototype ]
  +-----------------------+     +-----------------------+
                                | bark(): function      |
                                | [[Prototype]] --------+---> [ Animal.prototype ]
                                +-----------------------+     +-----------------------+
                                                              | eat(): function       |
                                                              | [[Prototype]] --------+---> [ Object.prototype ]
                                                              +-----------------------+     +-----------------------+
                                                                                            | toString()            |
                                                                                            | [[Prototype]]: null   |
                                                                                            +-----------------------+
```

* **Instance Properties (`name`, `breed`):** Directly owned by the `myDog` object instance in memory.
* **Prototype Methods (`bark`, `eat`):** Shared across all instances to conserve memory space. They reside on `Dog.prototype` and `Animal.prototype`.

---

### 1.2 Method Resolution Order (MRO)

When calling `myDog.eat()`, the engine resolves the property through the prototype chain sequentially:

```
+-------------------------------------------------------------------------+
|                  METHOD RESOLUTION SEQUENCE (MRO)                        |
+-------------------------------------------------------------------------+

  myDog.eat()
     |
     v
1. Check `myDog` instance properties -------------> [Not Found]
     |
     v
2. Follow `[[Prototype]]` to `Dog.prototype` ------> [Not Found]
     |
     v
3. Follow `[[Prototype]]` to `Animal.prototype` ---> [FOUND & EXECUTED]
```

If the method is not found anywhere on the chain up to `Object.prototype`, the runtime throws a `TypeError: myDog.eat is not a function`.

---

### 1.3 Property Lookup Flowchart

```
                          [ Property / Method Access ]
                                       |
                                       v
                        Does instance own property?
                                     /   \
                              YES   /     \   NO
                                   v       v
                          [ Return Value ]  Does [[Prototype]] exist?
                                                  /   \
                                           YES   /     \   NO
                                                v       v
                                    Traverse to parent  [ Return undefined /
                                       prototype         Throw TypeError ]
```

---

## 2. Practical Enterprise Usage Patterns

### Pattern 1: Abstract Base Service with Template Method Pattern

The **Template Method Pattern** defines the skeleton of an algorithm in an abstract base class, deferring specific steps to subclasses without altering the structure.

```typescript
/**
 * Abstract Base Processor for Async Job Queues
 */
export abstract class BaseJobProcessor<TData, TResult> {
  protected abstract jobName: string;

  /**
   * Template Method controlling execution workflow.
   */
  public async executeJob(data: TData): Promise<TResult> {
    this.logInfo(`Starting job execution...`);
    
    try {
      this.validate(data);
      const result = await this.process(data);
      await this.onSuccess(result);
      return result;
    } catch (error) {
      this.onError(error as Error);
      throw error;
    } finally {
      this.onComplete();
    }
  }

  // Mandatory abstract step for subclasses
  protected abstract process(data: TData): Promise<TResult>;

  // Optional hook methods (Default implementations)
  protected validate(data: TData): void {
    if (!data) throw new Error("Job payload cannot be null or undefined.");
  }

  protected async onSuccess(result: TResult): Promise<void> {
    this.logInfo(`Job completed successfully.`);
  }

  protected onError(error: Error): void {
    console.error(`[${this.jobName}] Job failed: ${error.message}`);
  }

  protected onComplete(): void {
    this.logInfo(`Job cleanup complete.`);
  }

  private logInfo(message: string): void {
    console.log(`[${new Date().toISOString()}] [${this.jobName}]: ${message}`);
  }
}

/**
 * Concrete Subclass Implementation
 */
export interface EmailPayload {
  to: string;
  body: string;
}

export class EmailJobProcessor extends BaseJobProcessor<EmailPayload, boolean> {
  protected jobName = "EmailNotificationJob";

  protected async process(payload: EmailPayload): Promise<boolean> {
    // Concrete email sending logic
    console.log(`Sending email to ${payload.to}`);
    return true;
  }

  protected override validate(payload: EmailPayload): void {
    super.validate(payload);
    if (!payload.to.includes("@")) {
      throw new Error("Invalid email recipient.");
    }
  }
}
```

---

### Pattern 2: Polymorphic Command Pipeline

Inheritance enables **Polymorphism**, allowing homogenous handling of diverse objects through a shared parent type.

```typescript
export abstract class SystemCommand {
  constructor(public readonly executionId: string) {}

  public abstract execute(): Promise<void>;
  public abstract rollback(): Promise<void>;
}

export class CreateUserCommand extends SystemCommand {
  constructor(executionId: string, public readonly userId: string) {
    super(executionId);
  }

  public async execute(): Promise<void> {
    console.log(`Creating user record: ${this.userId}`);
  }

  public async rollback(): Promise<void> {
    console.log(`Deleting user record: ${this.userId}`);
  }
}

export class SendWelcomeEmailCommand extends SystemCommand {
  constructor(executionId: string, public readonly email: string) {
    super(executionId);
  }

  public async execute(): Promise<void> {
    console.log(`Sending welcome email to: ${this.email}`);
  }

  public async rollback(): Promise<void> {
    console.log(`Canceling queued welcome email for: ${this.email}`);
  }
}

/**
 * Transaction Executor operating polymorphically on SystemCommand instances
 */
export class CommandPipeline {
  private executedCommands: SystemCommand[] = [];

  public async runPipeline(commands: SystemCommand[]): Promise<void> {
    for (const command of commands) {
      try {
        await command.execute();
        this.executedCommands.push(command);
      } catch (error) {
        console.error(`Command failed on step ${command.executionId}. Rolling back...`);
        await this.rollbackAll();
        break;
      }
    }
  }

  private async rollbackAll(): Promise<void> {
    while (this.executedCommands.length > 0) {
      const command = this.executedCommands.pop();
      await command?.rollback();
    }
  }
}
```

---

### Pattern 3: Custom Error Hierarchy for Domain-Driven Design

Inheriting from JavaScript's built-in `Error` class enables strongly-typed domain error handling in web APIs and services.

```typescript
export abstract class BaseDomainError extends Error {
  public abstract readonly statusCode: number;

  constructor(message: string, public readonly details?: Record<string, any>) {
    super(message);
    // Restore correct prototype chain in transpiled JS targets
    Object.setPrototypeOf(this, new.target.prototype);
    this.name = this.constructor.name;
    Error.captureStackTrace?.(this, this.constructor);
  }
}

export class NotFoundError extends BaseDomainError {
  public readonly statusCode = 404;
}

export class UnauthorizedError extends BaseDomainError {
  public readonly statusCode = 401;
}

export class ValidationError extends BaseDomainError {
  public readonly statusCode = 400;
}

// Global Error Handler usage
export function handleApiError(error: Error) {
  if (error instanceof BaseDomainError) {
    return {
      status: error.statusCode,
      body: {
        error: error.name,
        message: error.message,
        details: error.details ?? null,
      },
    };
  }

  return {
    status: 500,
    body: { error: "InternalServerError", message: "An unexpected error occurred." },
  };
}
```

---

## 3. Inheritance vs. Composition: Real-World Refactoring

### When to Prefer Inheritance
* Strong **"IS-A"** relationship (e.g., `NotFoundError` **IS-A** `BaseDomainError`).
* High degree of shared implementation logic across related classes.
* Enforcing invariant workflows via Abstract Base Classes (Template Method Pattern).

### When to Prefer Composition
* **"HAS-A"** relationship (e.g., `Car` **HAS-A** `Engine`, not `Car` **IS-A** `Engine`).
* Dynamic behavioral switching at runtime.
* Avoiding deep, rigid hierarchies ("Fragile Base Class" problem).

---

### Refactoring Example

#### ❌ Rigid Inheritance Approach (Problematic)

```typescript
class Logger {
  log(msg: string) { console.log(msg); }
}

class DatabaseService extends Logger {
  saveData(data: string) {
    this.log("Saving data..."); // DatabaseService IS NOT A Logger!
  }
}
```

#### ✅ Clean Composition Approach (Preferred)

```typescript
interface ILogger {
  log(message: string): void;
}

class ConsoleLogger implements ILogger {
  log(message: string): void {
    console.log(`[LOG]: ${message}`);
  }
}

class DatabaseService {
  // DatabaseService HAS-A Logger injected via dependency injection
  constructor(private logger: ILogger) {}

  public saveData(data: string): void {
    this.logger.log("Saving data...");
    // Database save operations...
  }
}
```

---

## 4. Summary & Complete Series Recap

| Concept | Key Takeaway | Primary Use Case |
| --- | --- | --- |
| **Prototypal Chain** | Method resolution moves upward from instance to `Object.prototype`. | Memory allocation & execution model. |
| **`super` Mechanism** | Invokes base constructors and methods. | Initializing parent state in derived classes. |
| **Template Method** | Base class controls structure; subclass provides steps. | Reusable workflows and frameworks. |
| **Polymorphism** | Treat derived objects through their base interface. | Command processing, plugins, handlers. |
| **Composition over Inheritance** | Combine small, focused objects rather than extending classes. | Flexible, decoupled application architecture. |