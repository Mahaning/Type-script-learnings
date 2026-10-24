# TypeScript Functions (Part 10): Simple Overloading Examples

Welcome to **Part 10** of our TypeScript Functions series. Following our deep dive into theory, advanced concepts, and method overriding in Part 9, this guide provides **simple, clear, and easy-to-understand programming examples** of function overloading in TypeScript.

## Table of Contents

1. [Example 1: Flexible Addition / Concatenation Utility](#example-1-flexible-addition--concatenation-utility)
2. [Example 2: Flexible Logger Utility](#example-2-flexible-logger-utility)
3. [Example 3: Unit Converter (Distance & Temperature)](#example-3-unit-converter-distance--temperature)
4. [Example 4: Array Element Finder](#example-4-array-element-finder)
5. [Example 5: Simple User Greeting Utility](#example-5-simple-user-greeting-utility)
6. [Key Rules to Remember](#key-rules-to-remember)

---

## Example 1: Flexible Addition / Concatenation Utility

A simple utility function that adds two numbers together or concatenates two strings together.

```typescript
// 1. OVERLOAD SIGNATURES
function add(a: number, b: number): number;
function add(a: string, b: string): string;

// 2. IMPLEMENTATION SIGNATURE
function add(a: any, b: any): any {
  return a + b;
}

// Usage
const sum = add(10, 20);          // Return type inferred as number (Output: 30)
const text = add("Hello, ", "World!"); // Return type inferred as string (Output: "Hello, World!")

// ❌ Error at compile time: Mixed types are not supported by the overload signatures!
// const invalid = add("Age: ", 25);
```

---

## Example 2: Flexible Logger Utility

A logger function that accepts either a plain message string or an object containing error details.

```typescript
interface ErrorDetail {
  code: number;
  message: string;
}

// 1. OVERLOAD SIGNATURES
function logMessage(message: string): void;
function logMessage(error: ErrorDetail): void;

// 2. IMPLEMENTATION SIGNATURE
function logMessage(input: string | ErrorDetail): void {
  if (typeof input === "string") {
    console.log(`[INFO]: ${input}`);
  } else {
    console.error(`[ERROR ${input.code}]: ${input.message}`);
  }
}

// Usage
logMessage("Application started successfully."); 
// Output: [INFO]: Application started successfully.

logMessage({ code: 404, message: "Resource not found." }); 
// Output: [ERROR 404]: Resource not found.
```

---

## Example 3: Unit Converter (Distance & Temperature)

An example demonstrating overloading based on differing parameter counts and string flags.

```typescript
// 1. OVERLOAD SIGNATURES: Converts meters to feet OR km to miles
function convert(value: number, unit: "mToFt"): number;
function convert(value: number, unit: "kmToMiles"): number;

// 2. IMPLEMENTATION SIGNATURE
function convert(value: number, unit: "mToFt" | "kmToMiles"): number {
  if (unit === "mToFt") {
    return value * 3.28084;
  }
  return value * 0.621371;
}

// Usage
const feet = convert(10, "mToFt");        // 32.8084
const miles = convert(100, "kmToMiles");  // 62.1371
```

---

## Example 4: Array Element Finder

A function that returns a single item if a number index is passed, or an array of matching items if a predicate filter function is passed.

```typescript
// 1. OVERLOAD SIGNATURES
function findItem<T>(list: T[], index: number): T | undefined;
function findItem<T>(list: T[], predicate: (item: T) => boolean): T[];

// 2. IMPLEMENTATION SIGNATURE
function findItem<T>(
  list: T[],
  arg: number | ((item: T) => boolean)
): T | T[] | undefined {
  if (typeof arg === "number") {
    return list[arg];
  }
  return list.filter(arg);
}

// Usage
const numbers = [10, 20, 30, 40, 50];

// Call Overload 1 (Index lookup)
const singleNumber = findItem(numbers, 2); 
console.log(singleNumber); // 30 (Type: number | undefined)

// Call Overload 2 (Filter predicate)
const evenNumbers = findItem(numbers, (num) => num > 25);
console.log(evenNumbers); // [30, 40, 50] (Type: number[])
```

---

## Example 5: Simple User Greeting Utility

A simple function that generates a greeting using a single full name, or a separate first and last name.

```typescript
// 1. OVERLOAD SIGNATURES
function greet(fullName: string): string;
function greet(firstName: string, lastName: string): string;

// 2. IMPLEMENTATION SIGNATURE
function greet(firstOrFull: string, lastName?: string): string {
  if (lastName) {
    return `Hello, ${firstOrFull} ${lastName}!`;
  }
  return `Hello, ${firstOrFull}!`;
}

// Usage
console.log(greet("Alice"));          // "Hello, Alice!"
console.log(greet("John", "Doe"));     // "Hello, John Doe!"
```

---

## Key Rules to Remember

1. **Overload signatures come first**: Declare all public overload signatures before writing the function body.
2. **Implementation signature is internal**: The function implementation signature MUST accept any combination of parameters defined in the overload signatures, but it cannot be called directly unless it matches one of the public overloads.
3. **No runtime overhead**: Overload signatures are completely stripped out when compiled to JavaScript.