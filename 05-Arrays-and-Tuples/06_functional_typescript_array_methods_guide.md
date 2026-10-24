# Purely Functional TypeScript Array Methods Guide

A reference covering `forEach`, `map`, `filter`, `reduce`, `sum` (pure function utility), and `every` in TypeScript. This guide strictly avoids Object-Oriented Programming (OOP) concepts such as classes, `this`, prototypes, and constructors, focusing exclusively on pure functions, type guards, and immutable data flow.

---

## 1. `forEach()`

### Declaration
```typescript
forEach(
  callback: (value: T, index: number, array: readonly T[]) => void
): void
```

### Optimization
* **Function Call Overhead:** Performs $O(N)$ iterations, invoking a callback for each element. Standard `for...of` loops are slightly faster in performance-critical paths because they avoid function execution overhead.
* **No Short-Circuiting:** Cannot exit early using `break` or `continue`. It evaluates all items in the array.

### When to Use
* To execute side effects for each element (e.g., logging output, calling external web APIs, updating global state stores).
* When you do **not** need to return a value or produce a transformed array.

### When NOT to Use
* **Do NOT use to transform data:** If you need a modified array output, use `map()` or `filter()`.
* **Do NOT use with async/await expecting sequential execution:** `forEach()` does not await promises inside its callback.

### Examples

#### Basic Side-Effect Logging
```typescript
type UserLog = {
  readonly id: string;
  readonly message: string;
};

const logs: readonly UserLog[] = [
  { id: "L1", message: "User logged in" },
  { id: "L2", message: "Data synchronized" }
];

logs.forEach((log, index) => {
  console.log(`[Entry ${index + 1}] ${log.id}: ${log.message}`);
});
```

---

## 2. `map()`

### Declaration
```typescript
map<T, U>(
  callback: (value: T, index: number, array: readonly T[]) => U
): U[]
```

### Optimization
* **1:1 Mapping:** Produces a new array with the exact same length as the original array.
* **Avoid Chaining:** Chaining multiple `.map()` calls creates intermediate arrays. Combine transformations into a single pure function step when processing large datasets.

### When to Use
* Transforming a list of elements from one structure into another (e.g., extracting values, changing shapes, applying mathematical transformations).
* Maintaining immutability when working with state.

### When NOT to Use
* Do not use `map()` if you do not use the returned array.
* Do not use `map()` to filter elements (use `filter()` or `flatMap()`).

### Examples

#### Data DTO Transformation
```typescript
type RawUser = {
  readonly id: number;
  readonly first_name: string;
  readonly last_name: string;
};

type UserProfile = {
  readonly id: number;
  readonly fullName: string;
};

const rawUsers: readonly RawUser[] = [
  { id: 101, first_name: "Alan", last_name: "Turing" },
  { id: 102, first_name: "Grace", last_name: "Hopper" }
];

const profiles: UserProfile[] = rawUsers.map((user) => ({
  id: user.id,
  fullName: `${user.first_name} ${user.last_name}`
}));
```

---

## 3. `filter()`

### Declaration
```typescript
// Standard Filter
filter<T>(
  predicate: (value: T, index: number, array: readonly T[]) => boolean
): T[]

// Type-Guard Filter (TypeScript Type Narrowing)
filter<T, S extends T>(
  predicate: (value: T, index: number, array: readonly T[]) => value is S
): S[]
```

### Optimization
* Performs a single full pass ($O(N)$ time complexity) and constructs a new array containing only matching elements.

### When to Use
* Subsetting an array based on logical conditions.
* Stripping `null` or `undefined` values while narrowing types safely via custom **Type Guards**.

### When NOT to Use
* **Do NOT use to locate a single element:** Use `find()` instead. `filter()` will scan the entire array even after finding the target item.

### Examples

#### Functional Type Guard Subsetting
```typescript
type SearchResult = string | null | undefined;

const searchResults: readonly SearchResult[] = ["Apple", null, "Banana", undefined, "Cherry"];

// Custom functional type guard
const isDefined = <T>(val: T | null | undefined): val is T => {
  return val !== null && val !== undefined;
};

const validResults: string[] = searchResults.filter(isDefined);
```

---

## 4. `reduce()`

### Declaration
```typescript
reduce<T, U>(
  callback: (accumulator: U, currentValue: T, index: number, array: readonly T[]) => U,
  initialValue: U
): U
```

### Optimization
* Highly efficient for combining multiple array processing steps (such as `filter` and `map`) into a single $O(N)$ pass.
* Avoid spread copies (`...acc`) inside the iteration when building objects or arrays to prevent $O(N^2)$ unnecessary memory allocations.

### When to Use
* Aggregating data into a single summary output (e.g., total counts, grouped record maps, computing complex data trees).

### When NOT to Use
* Simple mappings or filtering where `map()` or `filter()` is significantly easier to read and maintain.

### Examples

#### Grouping Data into a Dictionary Map
```typescript
type Item = {
  readonly category: string;
  readonly name: string;
};

const items: readonly Item[] = [
  { category: "fruit", name: "Apple" },
  { category: "vegetable", name: "Carrot" },
  { category: "fruit", name: "Banana" }
];

type GroupedItems = Record<string, string[]>;

const grouped = items.reduce<GroupedItems>((acc, item) => {
  const currentList = acc[item.category] ?? [];
  return {
    ...acc,
    [item.category]: [...currentList, item.name]
  };
}, {});
```

---

## 5. `sum()` (Pure Function Utility)

*(Note: JavaScript does not have a native `Array.prototype.sum()`. We define a pure, standalone helper function without mutating prototypes or using classes).*

### Declaration
```typescript
function sum(numbers: readonly number[]): number;

function sum<T>(
  items: readonly T[],
  selector: (item: T) => number
): number;
```

### Optimization
* A pure functional implementation using a `for` loop or `reduce()` executes in linear $O(N)$ time with minimal allocation overhead.

### When to Use
* Summing numerical arrays or computing totals from structured items (e.g., cart totals, metrics aggregation).

### When NOT to Use
* Avoid relying on standard array syntax like `arr.sum()` directly, as it does not exist natively in JS/TS. Always use a standalone helper function.

### Examples

#### Standalone Functional Implementation
```typescript
function sum<T>(
  items: readonly T[],
  selector?: (item: T) => number
): number {
  if (!selector) {
    return (items as readonly unknown[] as number[]).reduce((acc, val) => acc + val, 0);
  }
  return items.reduce((acc, item) => acc + selector(item), 0);
}

// Basic usage
const values = [10, 20, 30, 40];
const totalValue = sum(values); // 100

// Selector usage
type OrderItem = { readonly title: string; readonly price: number };
const order: readonly OrderItem[] = [
  { title: "Shirt", price: 25 },
  { title: "Pants", price: 50 }
];

const totalCost = sum(order, (item) => item.price); // 75
```

---

## 6. `every()`

### Declaration
```typescript
every<T>(
  predicate: (value: T, index: number, array: readonly T[]) => boolean
): boolean
```

### Optimization
* **Short-Circuit Evaluation:** Stops execution immediately and returns `false` as soon as the first non-matching element is evaluated.
* **Empty Array Check:** An empty array `[]` returns `true` regardless of the condition (vacuous truth).

### When to Use
* Checking if **all** items satisfy a validation rule (e.g., form field validity, permission checks).

### When NOT to Use
* Do not use if you only need to verify that **at least one** element passes a condition (use `some()` instead).

### Examples

#### Form Validation Rule
```typescript
type FieldValidation = {
  readonly fieldName: string;
  readonly isValid: boolean;
};

const validations: readonly FieldValidation[] = [
  { fieldName: "username", isValid: true },
  { fieldName: "email", isValid: true },
  { fieldName: "password", isValid: false }
];

const isFormValid: boolean = validations.every((v) => v.isValid); // false
```

---

## Functional Summary Matrix

| Method | Output | Mutates Source? | Short-circuits? | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **`forEach()`** | `void` | No | No | Execute side effects per element |
| **`map()`** | `U[]` | No | No | Transform elements 1:1 |
| **`filter()`** | `T[]` / `S[]` | No | No | Keep elements matching criteria |
| **`reduce()`** | `U` | No | No | Aggregate elements into a single value |
| **`sum()`** | `number` | No | No | Compute numerical total (Pure helper) |
| **`every()`** | `boolean` | No | Yes (on `false`) | Check if all elements satisfy criteria |