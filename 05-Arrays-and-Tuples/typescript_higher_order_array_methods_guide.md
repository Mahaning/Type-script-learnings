# TypeScript Higher-Order Array Methods: Deep Dive

A comprehensive guide covering `forEach`, `map`, `filter`, `reduce`, `sum` (custom implementation & utility patterns), and `every` in TypeScript. Each section includes full type declarations, performance & optimization considerations, decision criteria ("when to use" / "when NOT to use"), and practical TypeScript examples.

---

## Table of Contents
1. [forEach()](#1-foreach)
2. [map()](#2-map)
3. [filter()](#3-filter)
4. [reduce()](#4-reduce)
5. [sum() (Custom Extension / Utility)](#5-sum-custom-extension--utility)
6. [every()](#6-every)

---

## 1. forEach()

### Declaration
```typescript
Array<T>.forEach(
  callbackfn: (value: T, index: number, array: T[]) => void,
  thisArg?: any
): void
```

### Optimization
* **V8 Inlining:** Modern JS engines optimize `forEach`, but standard imperative `for` or `for...of` loops are still slightly faster because `forEach` incurs function call overhead per iteration.
* **No Short-Circuiting:** `forEach` executes for every index in the array (skipping sparse empty slots). You cannot use `break` or `continue`. If you need to exit early, use `for...of` or `some()` / `every()`.

### When to Use
* When performing side effects for each element (e.g., logging, triggering events, writing to a database, updating DOM).
* When you do **not** need to transform or return a new array.

### When NOT to Use
* **Do NOT use to transform data:** If you are building a new array by pushing inside `forEach`, use `map()` or `filter()` instead.
* **Do NOT use with async/await expecting sequential execution:** `forEach` does not await promises. `await` inside a `forEach` callback executes concurrently and won't halt the main function thread.

### Examples

#### Basic Usage (Side Effects)
```typescript
interface LogMessage {
  id: string;
  text: string;
}

const logs: LogMessage[] = [
  { id: "1", text: "System Boot" },
  { id: "2", text: "Network Connected" }
];

logs.forEach((log, index) => {
  console.log(`[${index}] Log ID ${log.id}: ${log.text}`);
});
```

#### Async Anti-Pattern vs. Proper Solution
```typescript
// ❌ WRONG: forEach does NOT wait for promises
async function sendNotificationsWrong(users: string[]): Promise<void> {
  users.forEach(async (user) => {
    await fetch(`/api/notify/${user}`); // Executes floating promises!
  });
  console.log("All done!"); // Prints immediately before promises resolve
}

// ✅ CORRECT: Use for...of or Promise.all + map
async function sendNotificationsCorrect(users: string[]): Promise<void> {
  for (const user of users) {
    await fetch(`/api/notify/${user}`);
  }
  console.log("All done sequentially!");
}
```

---

## 2. map()

### Declaration
```typescript
Array<T>.map<U>(
  callbackfn: (value: T, index: number, array: T[]) => U,
  thisArg?: any
): U[]
```

### Optimization
* **1:1 Projection:** Always creates a new array of the exact same length as the source array.
* **Memory Allocation:** Pre-allocates memory for the output array. Avoid chaining multiple `.map()` calls sequentially if you can combine transformations into a single pass to minimize garbage collection.

### When to Use
* When transforming every element in an array into a new representation (e.g., extracting fields, remapping structures, applying mathematical transformations).
* Immutability patterns (e.g., React state updates).

### When NOT to Use
* **Do NOT use if you don't use the returned array:** If you aren't storing or returning the output array, use `forEach()`.
* **Do NOT use to filter and map simultaneously:** Doing `.filter().map()` creates an intermediate array. Use `reduce()` or `flatMap()` if memory/performance is critical.

### Examples

#### Model Transformation
```typescript
interface UserDto {
  id: number;
  first_name: string;
  last_name: string;
}

interface UserVM {
  id: number;
  fullName: string;
}

const rawUsers: UserDto[] = [
  { id: 1, first_name: "Ada", last_name: "Lovelace" },
  { id: 2, first_name: "Alan", last_name: "Turing" }
];

const viewModels: UserVM[] = rawUsers.map((user) => ({
  id: user.id,
  fullName: `${user.first_name} ${user.last_name}`
}));
```

---

## 3. filter()

### Declaration
```typescript
// Standard Filter Signature
Array<T>.filter(
  predicate: (value: T, index: number, array: T[]) => unknown,
  thisArg?: any
): T[]

// Type Guard Filter Signature (TypeScript Specific)
Array<T>.filter<S extends T>(
  predicate: (value: T, index: number, array: T[]) => value is S,
  thisArg?: any
): S[]
```

### Optimization
* Creates a new array containing only elements where the predicate returns a truthy value.
* Performs full linear scan ($O(N)$ runtime).

### When to Use
* When subsetting an array based on conditions.
* When filtering out `null` or `undefined` values while narrowing TypeScript types via **Custom Type Guards**.

### When NOT to Use
* **Do NOT use to find a single element:** Use `find()` or `findIndex()` instead. `filter()` will inspect every element even after finding a match.

### Examples

#### Subsetting
```typescript
interface Task {
  title: string;
  completed: boolean;
}

const tasks: Task[] = [
  { title: "Task 1", completed: true },
  { title: "Task 2", completed: false },
  { title: "Task 3", completed: true }
];

const pendingTasks = tasks.filter((task) => !task.completed);
```

#### Type-Safe Null/Undefined Removal (Custom Type Guard)
```typescript
const mixedValues: (string | null | undefined)[] = ["Alpha", null, "Beta", undefined, "Gamma"];

// Type Guard predicate narrows (string | null | undefined)[] -> string[]
function isNotNull<T>(val: T | null | undefined): val is T {
  return val !== null && val !== undefined;
}

const stringsOnly: string[] = mixedValues.filter(isNotNull);
// Result type is string[] (no longer includes null or undefined)
```

---

## 4. reduce()

### Declaration
```typescript
// With initialValue
Array<T>.reduce<U>(
  callbackfn: (previousValue: U, currentValue: T, currentIndex: number, array: T[]) => U,
  initialValue: U
): U

// Without initialValue (Uses first element as initial accumulator)
Array<T>.reduce(
  callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: T[]) => T
): T
```

### Optimization
* Extremely versatile: Can emulate `map`, `filter`, `find`, and aggregation in a single loop pass ($O(N)$ execution time).
* **Object Mutation in Accumulator:** Mutating the accumulator object inside `reduce` (e.g., `acc[key] = val; return acc;`) is substantially faster than spreading object instances (`return { ...acc, [key]: val }`), which allocates object instances on every step ($O(N^2)$ memory allocation).

### When to Use
* Aggregating data into a single output value (e.g., total sum, grouping items into dynamic lookup maps, building complex data trees).
* Combining multiple operations (e.g., filtering and mapping in a single pass).

### When NOT to Use
* Simple operations where `map()`, `filter()`, or `flat()` are clearer. Readability can suffer if `reduce` logic becomes overly complex.

### Examples

#### Building a Hash Map Index
```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

const products: Product[] = [
  { id: "p1", name: "Laptop", price: 1200 },
  { id: "p2", name: "Mouse", price: 25 }
];

type ProductMap = Record<string, Product>;

// Optimized reduce mutating the accumulator container
const productLookup = products.reduce<ProductMap>((acc, product) => {
  acc[product.id] = product;
  return acc;
}, {});
```

---

## 5. sum() (Custom Extension / Utility)

> **Note:** Unlike JavaScript, there is no native `Array.prototype.sum()` method built into ECMAScript standard library. Summation is typically implemented via `reduce()` or helper utilities.

### Declarations & Implementations

#### Implementation A: Utility Function
```typescript
function sum<T>(
  items: T[],
  selector?: (item: T) => number
): number {
  if (!selector) {
    return (items as unknown as number[]).reduce((acc, curr) => acc + curr, 0);
  }
  return items.reduce((acc, curr) => acc + selector(curr), 0);
}
```

#### Implementation B: Global Prototype Extension (Module Declaration)
```typescript
export {};

declare global {
  interface Array<T> {
    sum(this: number[]): number;
    sumBy(this: T[], selector: (item: T) => number): number;
  }
}

Array.prototype.sum = function (this: number[]): number {
  return this.reduce((acc, curr) => acc + curr, 0);
};

Array.prototype.sumBy = function <T>(this: T[], selector: (item: T) => number): number {
  return this.reduce((acc, curr) => acc + selector(curr), 0);
};
```

### Optimization
* **Numerical Precision:** Floating point arithmetic inaccuracies (e.g., `0.1 + 0.2 === 0.30000000000000004`) apply. For high-precision financial/scientific operations, consider scaled integer arithmetic or dedicated libraries.
* **Loop Efficiency:** Standard `for` loop iteration inside a dedicated `sum` helper avoids function-closure allocation per element.

### When to Use
* Calculating totals for numeric series, order amounts, dynamic tallies, or invoice lines.

### When NOT to Use
* Do not attempt calling standard `.sum()` directly without adding prototype extensions or helper libraries (like Lodash `_.sum`), as TypeScript will throw a compilation error.

### Examples

#### Using Utility Function & Prototype Extensions
```typescript
// Example 1: Direct Number Array Sum
const prices = [19.99, 5.00, 10.50];
const total1 = sum(prices); // 35.49

// Example 2: Selector Object Sum
interface OrderLine {
  item: string;
  cost: number;
  qty: number;
}

const cart: OrderLine[] = [
  { item: "Book", cost: 15, qty: 2 },
  { item: "Pen", cost: 2, qty: 5 }
];

const totalCost = sum(cart, (line) => line.cost * line.qty); // 40

// Example 3: Prototype Extensions (if configured)
const numericSeries = [10, 20, 30];
console.log(numericSeries.sum()); // 60
console.log(cart.sumBy(c => c.cost)); // 17
```

---

## 6. every()

### Declaration
```typescript
// Standard Signature
Array<T>.every(
  predicate: (value: T, index: number, array: T[]) => unknown,
  thisArg?: any
): boolean

// Type Guard Signature
Array<T>.every<S extends T>(
  predicate: (value: T, index: number, array: T[]) => value is S,
  thisArg?: any
): this is S[]
```

### Optimization
* **Short-Circuiting Evaluation:** `every()` stops execution and immediately returns `false` as soon as it encounters the **first element** that fails the condition.
* **Vacuous Truth:** Calling `.every()` on an empty array (`[]`) returns `true` for any condition.

### When to Use
* Validating that **all** elements in a dataset adhere to strict business rules or constraints (e.g., checking form validation status, authorization permissions).

### When NOT to Use
* **Do NOT use if you only need at least one element to match:** Use `some()` instead.
* **Avoid relying on full iteration side effects:** Do not place state mutations inside `every()` callbacks because execution stops early on failure.

### Examples

#### Validation Check
```typescript
interface FormField {
  name: string;
  isValid: boolean;
}

const formFields: FormField[] = [
  { name: "email", isValid: true },
  { name: "password", isValid: true },
  { name: "age", isValid: false }
];

const isFormValid: boolean = formFields.every((field) => field.isValid);
// Output: false (Short-circuits at index 2)
```

#### Guarding Empty Arrays
```typescript
const emptyList: number[] = [];

// Vacuously true behavior
const allPositive = emptyList.every((n) => n > 0); 
console.log(allPositive); // true
```

---

## Summary Comparison Matrix

| Method | Returns | Mutates Original? | Short-Circuits? | Typical Primary Purpose |
| :--- | :--- | :--- | :--- | :--- |
| `forEach()` | `void` | No | No | Execute side effects |
| `map()` | `U[]` | No | No | Transform elements 1:1 |
| `filter()` | `T[]` / `S[]` | No | No | Select matching elements |
| `reduce()` | `U` | No | No | Aggregate array into single result |
| `sum()` *(Custom)* | `number` | No | No | Compute total numeric value |
| `every()` | `boolean` | No | Yes (on `false`) | Test if **all** elements pass condition |