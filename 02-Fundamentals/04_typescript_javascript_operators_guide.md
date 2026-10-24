# ⚡ The Ultimate Guide to Operators in JavaScript & TypeScript

Operators are special symbols or keywords used to perform operations on one, two, or three operands (values or variables) and produce a result.

---

## 📑 Table of Contents

1. [Definition & Core Concepts](#1-definition--core-concepts)
2. [Internal Working & Engine Execution](#2-internal-working--engine-execution)
3. [Type Operators](#3-type-operators)
4. [Arithmetic Operators](#4-arithmetic-operators)
5. [Increment and Decrement Operators](#5-increment-and-decrement-operators)
6. [Assignment & Shorthand Assignment Operators](#6-assignment--shorthand-assignment-operators)
7. [Relational & Comparison Operators](#7-relational--comparison-operators)
8. [Equality (`==`) vs. Strict Equality (`===`)](#8-equality--vs-strict-equality-)
9. [Logical Operators](#9-logical-operators)
10. [Conditional & Ternary Operators](#10-conditional--ternary-operators)
11. [Combination & Short-Circuit Assignment Operators](#11-combination--short-circuit-assignment-operators)
12. [Summary Operator Cheat Sheet](#12-summary-operator-cheat-sheet)

---

## 1. Definition & Core Concepts

An **operator** defines a transformation or evaluation rule applied to data operands. Operators are categorized by arity (the number of operands they accept):

* **Unary Operators:** Accept **1** operand (e.g., `typeof x`, `++a`, `!flag`).
* **Binary Operators:** Accept **2** operands (e.g., `a + b`, `x === y`).
* **Ternary Operator:** Accepts **3** operands (`condition ? expr1 : expr2`).

---

## 2. Internal Working & Engine Execution

When the V8 engine (or SpiderMonkey) parses an operator, it executes three low-level steps:

```
┌────────────────────────────────────────────────────────────────────────┐
│ 1. Value Extraction (Getvalue)                                         │
│    Reads values from Stack registers or heap references.               │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Implicit Type Coercion (ToPrimitive / ToNumber / ToString)          │
│    If types differ, abstract algorithms coerce operands.              │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Evaluation & Result Emission                                        │
│    Executes operation on the CPU / bytecode engine and pushes result.  │
└────────────────────────────────────────────────────────────────────────┘
```

### Key ECMAScript Internal Abstract Operations
* **`ToPrimitive(input [, PreferredType])`**: Converts objects to scalar primitive values by calling `Symbol.toPrimitive`, `valueOf()`, or `toString()`.
* **`ToNumber(input)`**: Converts primitives (like strings or booleans) into double-precision 64-bit IEEE 754 numbers (`true` $\rightarrow 1$, `null` $\rightarrow 0$, `"42"` $\rightarrow 42$, `undefined` $\rightarrow \text{NaN}$).
* **`ToBoolean(input)`**: Evaluates truthiness (`0`, `""`, `null`, `undefined`, `NaN`, `false` are falsy; everything else is truthy).

---

## 3. Type Operators

Type operators evaluate or manipulate data type metadata at runtime (JS) or compile time (TS).

### Runtime JavaScript Type Operators

#### A. `typeof`
Returns a string indicating the primitive data type of the operand.

```typescript
console.log(typeof 42);          // "number"
console.log(typeof "hello");     // "string"
console.log(typeof true);        // "boolean"
console.log(typeof undefined);   // "undefined"
console.log(typeof Symbol());    // "symbol"
console.log(typeof 10n);         // "bigint"
console.log(typeof {});         // "object"
console.log(typeof null);       // "object" (Historical JS V8 quirk: type tag 0)
console.log(typeof (() => {}));  // "function"
```

#### B. `instanceof`
Checks if an object's prototype chain contains the `.prototype` property of a constructor function.

```typescript
class Animal {}
class Dog extends Animal {}

const pet = new Dog();

console.log(pet instanceof Dog);    // true
console.log(pet instanceof Animal); // true
console.log(pet instanceof Object); // true
console.log([] instanceof Array);   // true
```

### TypeScript-Only Type Operators

#### C. `keyof`
Extracts the union of keys from an object type.

```typescript
interface User {
  id: number;
  name: string;
}

type UserKeys = keyof User; // "id" | "name"
```

#### D. `as` & `is` (Type Assertions & Predicates)

```typescript
// Type Assertion
const input = document.getElementById("user-input") as HTMLInputElement;

// Type Predicate Function
function isString(val: unknown): val is string {
  return typeof val === "string";
}
```

---

## 4. Arithmetic Operators

Arithmetic operators perform standard mathematical operations on numeric operands.

| Operator | Name | Example | Result (`a=10, b=3`) |
| :--- | :--- | :--- | :--- |
| `+` | Addition | `a + b` | `13` |
| `-` | Subtraction | `a - b` | `7` |
| `*` | Multiplication | `a * b` | `30` |
| `/` | Division | `a / b` | `3.333...` |
| `%` | Modulus (Remainder)| `a % b` | `1` |
| `**` | Exponentiation | `a ** b` | `1000` |
| `+` (Unary) | Unary Plus | `+"42"` | `42` (Coerces string to number) |
| `-` (Unary) | Unary Negation | `-"42"` | `-42` |

### Internal Working: The Addition `+` Dual Behavior

The `+` operator has two distinct behaviors: numeric addition and string concatenation.

```typescript
// Rule: If either operand is a string (or coerces to one), concatenation occurs.
console.log(5 + "5");     // "55"  (Number 5 coerced to "5")
console.log("5" + true);  // "5true"
console.log(5 + true);   // 6      (Boolean true coerced to 1)
console.log([] + []);     // ""     (Objects coerced to primitives: "" + "" = "")
console.log([] + {});     // "[object Object]"
```

---

## 5. Increment and Decrement Operators

Increment (`++`) and decrement (`--`) modify a variable by adding or subtracting `1`.

### Pre-Increment / Pre-Decrement vs. Post-Increment / Post-Decrement

* **Pre (`++x`, `--x`):** Modifies the variable **first**, then returns the **updated value**.
* **Post (`x++`, `x--`):** Returns the **original value first**, then modifies the variable.

```typescript
// --- Pre-Increment (++a) ---
let a = 5;
let b = ++a; // Step 1: a becomes 6. Step 2: b is assigned 6.
console.log(`a: ${a}, b: ${b}`); // a: 6, b: 6

// --- Post-Increment (c++) ---
let c = 5;
let d = c++; // Step 1: d is assigned 5. Step 2: c becomes 6.
console.log(`c: ${c}, d: ${d}`); // c: 6, d: 5

// --- Complex Expression Internal Evaluation ---
let x = 10;
let result = x++ + ++x;
// Step 1: x++ evaluates to 10 (x becomes 11 in background)
// Step 2: ++x increments x from 11 to 12, then evaluates to 12
// Step 3: 10 + 12 = 22
console.log(result); // 22
console.log(x);      // 12
```

---

## 6. Assignment & Shorthand Assignment Operators

Assignment operators assign values from the right-hand side operand to the left-hand variable.

| Operator | Syntax | Equivalent Expression |
| :--- | :--- | :--- |
| `=` | `x = y` | `x = y` |
| `+=` | `x += y` | `x = x + y` |
| `-=` | `x -= y` | `x = x - y` |
| `*=` | `x *= y` | `x = x * y` |
| `/=` | `x /= y` | `x = x / y` |
| `%=` | `x %= y` | `x = x % y` |
| `**=` | `x **= y` | `x = x ** y` |

```typescript
let balance = 100;
balance += 50;  // balance = 150
balance *= 2;   // balance = 300
balance -= 75;  // balance = 225
```

---

## 7. Relational & Comparison Operators

Relational operators compare two operands and evaluate to a boolean value (`true` or `false`).

| Operator | Name | Example (`a=5, b=10`) | Result |
| :--- | :--- | :--- | :--- |
| `>` | Greater Than | `a > b` | `false` |
| `<` | Less Than | `a < b` | `true` |
| `>=` | Greater Than or Equal | `a >= 5` | `true` |
| `<=` | Less Than or Equal | `a <= b` | `true` |
| `in` | Property In Object | `"name" in obj` | `true` / `false` |

### Internal Working: String Comparison Mechanics

When comparing strings, JavaScript uses Unicode character code order lexicographically:

```typescript
console.log("apple" < "banana"); // true ('a' code point 97 < 'b' code point 98)
console.log("20" > "3");         // false ('2' code point 50 < '3' code point 51)
console.log("20" > 3);           // true ("20" coerced to number 20, 20 > 3)
```

---

## 8. Equality (`==`) vs. Strict Equality (`===`)

| Feature | Loose Equality (`==`) | Strict Equality (`===`) |
| :--- | :--- | :--- |
| **Type Coercion** | **Yes** (converts operands to matching types) | **No** (returns `false` if types differ) |
| **Performance** | Slightly slower (requires coercion steps) | Faster (direct byte & type check) |
| **Best Practice** | Avoid (except when checking `null`/`undefined`) | **Always Use** |

### Internal Algorithm Comparison

#### Abstract Equality (`==`) Rules
1. If types match, apply strict equality `===`.
2. If `null == undefined`, return `true`.
3. If comparing `number == string`, convert string to number.
4. If comparing `boolean == any`, convert boolean to number (`true` $\rightarrow 1$, `false` $\rightarrow 0$).
5. If comparing `object == primitive`, convert object using `ToPrimitive()`.

```typescript
// --- Abstract Equality (==) Unexpected Behavior ---
console.log(0 == false);        // true  (false -> 0)
console.log("" == 0);           // true  ("" -> 0)
console.log("" == false);       // true  ("" -> 0, false -> 0)
console.log(null == undefined); // true  (Special rule)
console.log(null == 0);         // false (null does NOT coerce to 0 for ==)
console.log([1, 2] == "1,2");   // true  ([1,2].toString() -> "1,2")

// --- Strict Equality (===) ---
console.log(0 === false);       // false (different types)
console.log("" === 0);          // false
console.log(null === undefined);// false
console.log("5" === "5");       // true
```

---

## 9. Logical Operators

Logical operators combine boolean expressions and implement short-circuit evaluation.

| Operator | Name | Short-Circuit Rule |
| :--- | :--- | :--- |
| `&&` | Logical AND | Returns first **falsy** value, or last value if all are truthy. |
| `||` | Logical OR | Returns first **truthy** value, or last value if all are falsy. |
| `!` | Logical NOT | Inverts boolean truthiness value (`!truthy` $\rightarrow `false`). |
| `??` | Nullish Coalescing | Returns RHS only if LHS is `null` or `undefined`. |

### Short-Circuit Execution Mechanics

```typescript
// --- Logical AND (&&) ---
// Stops evaluating as soon as a falsy value is encountered.
const user = { authenticated: true, name: "Alice" };
const displayName = user.authenticated && user.name; 
console.log(displayName); // "Alice"

// --- Logical OR (||) ---
// Stops evaluating as soon as a truthy value is encountered.
const count = 0;
const defaultCount = count || 10; 
console.log(defaultCount); // 10 (0 is falsy, so it picks 10!)

// --- Nullish Coalescing (??) ---
// Treats 0, "", and false as VALID values; only triggers for null or undefined.
const exactCount = count ?? 10;
console.log(exactCount); // 0 (0 is not null/undefined!)
```

---

## 10. Conditional & Ternary Operators

### The Ternary Operator (`? :`)
The only operator in JS/TS taking three operands. Serves as a inline shorthand for an `if-else` statement.

```typescript
// Syntax: condition ? expressionIfTrue : expressionIfFalse

const age = 20;
const canVote = age >= 18 ? "Eligible" : "Ineligible";
console.log(canVote); // "Eligible"

// Nested Ternary (Use sparingly for readability)
const score = 85;
const grade = score >= 90 ? "A" 
            : score >= 80 ? "B" 
            : score >= 70 ? "C" 
            : "F";
console.log(grade); // "B"
```

---

## 11. Combination & Short-Circuit Assignment Operators

ES2021 introduced short-circuit logical assignment operators that combine logical checks with assignment.

| Operator | Syntax | Equivalent Expression |
| :--- | :--- | :--- |
| `&&=` | `x &&= y` | `x && (x = y)` (Assigns `y` only if `x` is truthy) |
| `||=` | `x ||= y` | `x || (x = y)` (Assigns `y` only if `x` is falsy) |
| `??=` | `x ??= y` | `x ?? (x = y)` (Assigns `y` only if `x` is nullish) |

```typescript
let config = { timeout: 0, title: "" };

// Assign default title if missing
config.title ||= "Default Application";
console.log(config.title); // "Default Application"

// Timeout 0 is falsy, so ||= overwrites it unexpectedly:
// config.timeout ||= 5000; -> config.timeout becomes 5000

// Solution: Use ??= to safely preserve 0 or false
config.timeout ??= 5000;
console.log(config.timeout); // 0 (Preserved!)
```

---

## 12. Summary Operator Cheat Sheet

```typescript
// Real-world application combining multiple operators

interface Order {
  id: string;
  total?: number;
  isPaid: boolean;
}

function processOrder(order: Order) {
  // 1. Nullish coalescing + Shorthand assignment
  order.total ??= 100;

  // 2. Unary + Arithmetic
  const tax = order.total * 0.10;
  const finalPrice = order.total + tax;

  // 3. Logical + Ternary
  const status = order.isPaid && finalPrice > 0 
    ? "PROCESSING_SHIPMENT" 
    : "AWAITING_PAYMENT";

  // 4. Type check operator
  if (typeof status === "string") {
    console.log(`Order ${order.id}: ${status} ($${finalPrice.toFixed(2)})`);
  }
}

processOrder({ id: "ORD-9912", isPaid: true });
// Output: Order ORD-9912: PROCESSING_SHIPMENT ($110.00)
```