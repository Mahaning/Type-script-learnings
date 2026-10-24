# TypeScript Interfaces - Deep Dive (Part 2)

## Table of Contents

1. [Deep-Dive: Creating Interfaces](#1-deep-dive-creating-interfaces)
2. [Optional Properties (`?`) in Depth](#2-optional-properties--in-depth)
3. [Readonly Modifiers (`readonly`)](#3-readonly-modifiers-readonly)
4. [Defining Methods in Interfaces](#4-defining-methods-in-interfaces)
5. [Nested Interfaces & Complex Types](#5-nested-interfaces--complex-types)
6. [Index Signatures in Detail](#6-index-signatures-in-detail)
7. [Summary Checklist](#7-summary-checklist)

---

## 1. Deep-Dive: Creating Interfaces

Declaring an interface establishes a compile-time structural contract. TypeScript validates object shapes against this contract without emitting any runtime JavaScript code.

```typescript
interface DatabaseConfig {
  host: string;
  port: number;
  databaseName: string;
}

// Strictly enforces all required properties
const dbConfig: DatabaseConfig = {
  host: "localhost",
  port: 5432,
  databaseName: "production_db",
};
```

---

## 2. Optional Properties (`?`) in Depth

Optional properties mark fields that may or may not be present on an object. They are defined by appending a `?` after the property name.

### Syntax & Behavior

```typescript
interface HTTPRequest {
  url: string;
  method: "GET" | "POST" | "PUT" | "DELETE";
  headers?: Record<string, string>; // Optional: string | undefined
  timeout?: number;                  // Optional: number | undefined
}

const req1: HTTPRequest = {
  url: "/api/users",
  method: "GET",
};

const req2: HTTPRequest = {
  url: "/api/users",
  method: "POST",
  headers: { "Content-Type": "application/json" },
  timeout: 5000,
};
```

### Type Checking & `exactOptionalPropertyTypes`

By default in TypeScript, an optional property `timeout?: number` accepts `number`, `undefined`, or being omitted entirely. 

When `exactOptionalPropertyTypes: true` is enabled in `tsconfig.json`:
- Omitting the key is valid (`{ url: "/api", method: "GET" }`).
- Explicitly setting `undefined` requires explicit union typing (`timeout: number | undefined`).

```typescript
// Handling optional properties safely at runtime
function executeRequest(config: HTTPRequest): void {
  // Use Nullish Coalescing (??) to set fallback values
  const requestTimeout = config.timeout ?? 3000;
  console.log(`Executing request with timeout: ${requestTimeout}ms`);
}
```

---

## 3. Readonly Modifiers (`readonly`)

The `readonly` modifier locks a property so it cannot be reassigned after the object is created.

```typescript
interface UserSession {
  readonly sessionId: string;
  readonly createdAt: Date;
  activeStatus: boolean;
}

const session: UserSession = {
  sessionId: "sess_abc123456",
  createdAt: new Date(),
  activeStatus: true,
};

// Valid property mutation
session.activeStatus = false;

// ❌ Compiler Error: Cannot assign to 'sessionId' because it is a read-only property.
// session.sessionId = "sess_xyz987654";
```

### `readonly` vs `const`

| Feature | `const` | `readonly` |
| :--- | :--- | :--- |
| **Applies To** | Variable bindings (`const x = ...`) | Object properties within interfaces/types |
| **Enforcement** | Prevents variable re-binding | Prevents property re-assignment |
| **Mutability** | Object properties can still be mutated | Property value cannot be re-assigned |

```typescript
// `const` prevents reassigning the variable 'appConfig', but properties can change unless marked `readonly`
const appConfig: DatabaseConfig = {
  host: "127.0.0.1",
  port: 3000,
  databaseName: "dev_db",
};

appConfig.port = 8080; // ✅ Allowed by `const`! (Use `readonly` in interface to prevent this)
```

---

## 4. Defining Methods in Interfaces

There are two primary syntaxes to declare methods inside an interface: **Method Syntax** and **Property Function Syntax**.

```typescript
interface Calculator {
  // 1. Method Syntax
  add(a: number, b: number): number;

  // 2. Property / Function Type Syntax
  subtract: (a: number, b: number) => number;
}

const myCalc: Calculator = {
  add(a, b) {
    return a + b;
  },
  subtract: (a, b) => a - b,
};
```

### Method Syntax vs. Property Function Syntax

Under strict flag options like `--strictFunctionTypes`:
- **Method Syntax (`add(a: number): number`)** is checked *bivariantly* for compatibility.
- **Property Function Syntax (`subtract: (a: number) => number`)** is checked *contravariantly*, enforcing stricter type checking for parameters.

> **Best Practice:** Use property function syntax (`name: (args) => ReturnType`) when strict function parameter safety is required across subtypes.

---

## 5. Nested Interfaces & Complex Types

Interfaces can reference other interfaces as property types to build complex domain models.

```typescript
interface Dimensions {
  width: number;
  height: number;
  depth: number;
}

interface LocationCoordinates {
  latitude: number;
  longitude: number;
}

interface WarehouseItem {
  readonly sku: string;
  name: string;
  dimensions: Dimensions;            // Nested Interface
  location?: LocationCoordinates;    // Optional Nested Interface
}

const item: WarehouseItem = {
  sku: "SKU-88219",
  name: "Mechanical Keyboard",
  dimensions: {
    width: 45,
    height: 15,
    depth: 4,
  },
};
```

---

## 6. Index Signatures in Detail

Index signatures are used when you don't know the property names ahead of time, but know the key and value types.

```typescript
interface Dictionary {
  [key: string]: string;
}

interface DynamicMetrics {
  readonly id: string;               // Known required property
  [metricName: string]: number | string; // Index signature (must accommodate known properties)
}

const serverStats: DynamicMetrics = {
  id: "srv_us_east_1",
  cpuUsage: 84.5,
  memoryUsage: 62.1,
  status: "healthy",
};
```

> **Rule for Index Signatures:** All explicitly declared properties in an interface must conform to the return type of the index signature.

---

## 7. Summary Checklist

- [ ] **Optionality (`?`):** Appending `?` allows properties to be omitted or evaluate to `undefined`.
- [ ] **Immutability (`readonly`):** Prevents mutation of object keys after instantiation.
- [ ] **Method Signatures:** Prefer property function syntax (`fn: (arg: T) => R`) under strict function checking.
- [ ] **Modularity:** Nest interfaces cleanly to build complex enterprise object shapes.