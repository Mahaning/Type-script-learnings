# TypeScript Arrays & Tuples: Built-in Methods and Operations

A complete reference guide covering every major built-in method, operator, and type utility available for **Arrays** and **Tuples** in TypeScript.

---

## Part 1: Array Built-in Methods & Operations

Because TypeScript compiles to JavaScript, all standard JavaScript Array methods are fully supported with strict typing added.

### 1. Adding & Removing Elements (Mutating)

| Method | Syntax / Signature | Description |
| :--- | :--- | :--- |
| `push()` | `arr.push(...items: T[]): number` | Appends elements to the end; returns new array length. |
| `pop()` | `arr.pop(): T \| undefined` | Removes and returns the last element. |
| `unshift()`| `arr.unshift(...items: T[]): number` | Inserts elements at the beginning; returns new array length. |
| `shift()` | `arr.shift(): T \| undefined` | Removes and returns the first element. |
| `splice()` | `arr.splice(start: number, deleteCount?: number, ...items: T[]): T[]` | Modifies contents by removing or replacing existing elements and/or adding new ones in place. |

```typescript
const numbers: number[] = [1, 2, 3];

numbers.push(4);               // [1, 2, 3, 4]
const last = numbers.pop();    // returns 4, array is [1, 2, 3]
numbers.unshift(0);            // [0, 1, 2, 3]
const first = numbers.shift(); // returns 0, array is [1, 2, 3]
numbers.splice(1, 1, 99);      // removes 1 element at index 1, inserts 99 -> [1, 99, 3]
```

---

### 2. Search & Inspection Methods

| Method | Syntax / Signature | Description |
| :--- | :--- | :--- |
| `indexOf()` | `arr.indexOf(item: T, fromIndex?: number): number` | Returns the first index of `item`, or `-1` if not found. |
| `lastIndexOf()` | `arr.lastIndexOf(item: T, fromIndex?: number): number` | Returns the last index of `item`, or `-1`. |
| `includes()` | `arr.includes(item: T, fromIndex?: number): boolean` | Returns `true` if array contains `item`. |
| `find()` | `arr.find(predicate: (val: T, idx: number) => boolean): T \| undefined` | Returns the **first element** matching condition, or `undefined`. |
| `findIndex()` | `arr.findIndex(predicate: (val: T, idx: number) => boolean): number` | Returns the index of the first element matching condition, or `-1`. |
| `findLast()` | `arr.findLast(predicate: (val: T, idx: number) => boolean): T \| undefined` | Returns the **last element** matching condition. |
| `findLastIndex()` | `arr.findLastIndex(predicate: (val: T, idx: number) => boolean): number` | Returns the index of the last element matching condition. |
| `some()` | `arr.some(predicate: (val: T) => boolean): boolean` | Returns `true` if **at least one** element passes condition. |
| `every()` | `arr.every(predicate: (val: T) => boolean): boolean` | Returns `true` if **all** elements pass condition. |

```typescript
const items: string[] = ["apple", "banana", "cherry", "banana"];

items.indexOf("banana");                       // 1
items.lastIndexOf("banana");                   // 3
items.includes("cherry");                      // true
items.find((item) => item.startsWith("b"));    // "banana"
items.findIndex((item) => item.length > 5);   // 1 ("banana")
items.some((item) => item.startsWith("a"));    // true
items.every((item) => item.length >= 5);       // true
```

---

### 3. Transformation & Extraction (Non-Mutating)

| Method | Syntax / Signature | Description |
| :--- | :--- | :--- |
| `map()` | `arr.map<U>(callback: (val: T, idx: number) => U): U[]` | Creates a new array with results of calling callback on every element. |
| `filter()` | `arr.filter(predicate: (val: T, idx: number) => boolean): T[]` | Creates a new array with elements that pass the predicate test. |
| `slice()` | `arr.slice(start?: number, end?: number): T[]` | Returns a shallow copy of a portion of the array (end non-inclusive). |
| `concat()` | `arr.concat(...items: (T \| T[])[]): T[]` | Combines arrays into a new array. |
| `flat()` | `arr.flat<D>(depth?: D): FlatArray<T, D>[]` | Flattens nested array structures up to specified depth. |
| `flatMap()` | `arr.flatMap<U>(callback: (val: T) => U \| U[]): U[]` | Maps each element using mapping function, then flattens result by depth 1. |

```typescript
const nums: number[] = [1, 2, 3, 4];

const doubled = nums.map((n) => n * 2);             // [2, 4, 6, 8]
const evens = nums.filter((n) => n % 2 === 0);      // [2, 4]
const sub = nums.slice(1, 3);                       // [2, 3]
const merged = nums.concat([5, 6]);                 // [1, 2, 3, 4, 5, 6]

const nested = [1, [2, [3]]];
const flat1 = nested.flat(1);                       // [1, 2, [3]]
const flat2 = nested.flat(2);                       // [1, 2, 3]
```

---

### 4. Reduction & Aggregation

| Method | Syntax / Signature | Description |
| :--- | :--- | :--- |
| `reduce()` | `arr.reduce(callback: (acc: U, curr: T) => U, initialValue: U): U` | Accumulates array elements into a single value (left-to-right). |
| `reduceRight()` | `arr.reduceRight(callback: (acc: U, curr: T) => U, initialValue: U): U` | Accumulates array elements into a single value (right-to-left). |

```typescript
const values: number[] = [10, 20, 30];

const total = values.reduce((acc, curr) => acc + curr, 0); // 60
```

---

### 5. Sorting, Reversing & Immutability Alternatives

| Method | Type | Description |
| :--- | :--- | :--- |
| `sort()` | Mutating | Sorts array elements in place. |
| `reverse()` | Mutating | Reverses array elements in place. |
| `toSorted()` | Non-mutating | Returns a new sorted array without modifying original. |
| `toReversed()` | Non-mutating | Returns a new reversed array without modifying original. |
| `toSpliced()` | Non-mutating | Returns a new array with elements removed/inserted without modifying original. |
| `with()` | Non-mutating | `arr.with(index, value)` returns a new array with value updated at index. |

```typescript
const scores: number[] = [30, 10, 20];

// Non-mutating ES2023 methods
const sorted = scores.toSorted((a, b) => a - b); // [10, 20, 30] (scores is unchanged)
const reversed = scores.toReversed();            // [20, 10, 30]
const updated = scores.with(0, 99);              // [99, 10, 20]
```

---

### 6. String & Conversion Methods

| Method | Syntax / Signature | Description |
| :--- | :--- | :--- |
| `join()` | `arr.join(separator?: string): string` | Joins all elements into a single string. |
| `toString()` | `arr.toString(): string` | Returns string representation of array. |
| `entries()` | `arr.entries(): IterableIterator<[number, T]>` | Returns iterator with key/value pairs. |
| `keys()` | `arr.keys(): IterableIterator<number>` | Returns iterator with array indices. |
| `values()` | `arr.values(): IterableIterator<T>` | Returns iterator with array values. |

---

## Part 2: Tuple Operations & Built-in Interoperability

Tuples inherit standard `Array` methods at runtime, but TypeScript imposes **type system restrictions** on top of them.

### 1. Structural Access & Type Extraction

```typescript
type UserTuple = [id: number, name: string, active: boolean];

const user: UserTuple = [101, "Alice", true];

// Indexed Access
const userId: number = user[0];
const userName: string = user[1];

// Extracting Length as Literal Type
type TupleLength = UserTuple["length"]; // 3

// Extracting Union Type of Elements
type ElementUnion = UserTuple[number]; // number | string | boolean
```

---

### 2. Built-in Methods on Tuples & Type Safety Behaviors

| Method / Operation | Runtime Behavior | TypeScript Type Compiler Behavior |
| :--- | :--- | :--- |
| `destructuring` | Unpacks array values | Retains exact positional type for each variable. |
| `push()` / `pop()` | Mutates array | Allowed on mutable tuples, but **type unsafe** (violates tuple length). Use `readonly` to block. |
| `map()` | Creates transformed array | Transforms tuple to standard array `U[]` unless typed with assertions. |
| `concat()` | Merges tuples/arrays | Can retain tuple structure using `const` assertions or explicit tuple types. |

```typescript
// 1. Destructuring
const coordinates: [number, number, number] = [12.5, 45.0, 100.2];
const [x, y, z] = coordinates; // x: number, y: number, z: number

// 2. Readonly Tuples (Blocks mutating built-ins)
type Point = readonly [number, number];
const pt: Point = [10, 20];

// pt.push(30); ❌ Error: Property 'push' does not exist on type 'readonly [number, number]'
// pt[0] = 5;   ❌ Error: Cannot assign to '0' because it is a read-only property.

// 3. Spreading Tuples
type Header = [string, string];
type Body = [number, number];

type Combined = [...Header, ...Body]; // [string, string, number, number]
const payload: Combined = ["auth", "token", 200, 404];
```

---

### 3. Utility Types for Tuples

TypeScript type system operations designed specifically for manipulating tuple types at compile time:

```typescript
// Tuple Type Manipulations

type Person = [name: string, age: number, isMember: boolean];

// 1. First element
type First<T extends any[]> = T extends [infer Head, ...any[]] ? Head : never;
type FirstType = First<Person>; // string

// 2. Tail elements (all except first)
type Tail<T extends any[]> = T extends [any, ...infer Rest] ? Rest : never;
type TailTypes = Tail<Person>; // [number, boolean]

// 3. Prepend Element
type Prepend<T extends any[], Element> = [Element, ...T];
type ExtendedPerson = Prepend<Person, id: number>; // [number, string, number, boolean]
```

---

## Part 3: Quick Comparison Matrix

| Operation / Feature | Array | Tuple |
| :--- | :--- | :--- |
| **Type Notation** | `T[]` or `Array<T>` | `[Type1, Type2, ...]` |
| **Fixed Length at Type Level** | No | Yes |
| **Heterogeneous Types** | Union array `(string \| number)[]` | Ordered positional `[string, number]` |
| **`readonly` support** | `readonly T[]` | `readonly [T1, T2]` |
| **`length` Type** | `number` | Exact literal number (e.g., `2`) |
| **Spread Operator (`...`)** | Merges into broader array | Merges into exact positional tuple |