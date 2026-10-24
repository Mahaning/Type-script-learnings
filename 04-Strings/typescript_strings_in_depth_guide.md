# TypeScript Strings: Complete Reference & Internals

A comprehensive guide covering TypeScript strings, including types, declarations, built-in methods, string validation, immutability, memory allocation, and V8 engine internal implementation.

## Table of Contents

 1. [Introduction to TypeScript Strings](#1-introduction-to-typescript-strings)

 2. [Types, Declarations, and Syntax](#2-types-declarations-and-syntax)

 3. [Immutability Concepts](#3-immutability-concepts)

 4. [Memory Allocation & V8 Internal Implementation](#4-memory-allocation--v8-internal-implementation)

 5. [Finding Characters & Indexing](#5-finding-characters--indexing)

 6. [Substring Extraction](#6-substring-extraction)

 7. [String Validations](#7-string-validations)

 8. [Replacing String Content](#8-replacing-string-content)

 9. [Splitting, Trimming, and Concatenating](#9-splitting-trimming-and-concatenating)

10. [Multi-line Strings & Template Literals](#10-multi-line-strings--template-literals)

11. [Basic & Advanced Methods Summary](#11-basic--advanced-methods-summary)

## 1. Introduction to TypeScript Strings

In TypeScript, a string represents textual data as a sequence of 16-bit UTF-16 code units. TypeScript builds upon JavaScript's primitive `string` type while adding static typing, literal types, template literal types, and compile-time verification.

## 2. Types, Declarations, and Syntax

TypeScript offers several ways to type and declare strings.

### Declarations

```
// Single and Double Quotes (Primitive string)
const firstName: string = 'Alan';
const lastName: string = "Turing";

// Template Literals (ES6+)
const greeting: string = `Hello, ${firstName}!`;

// String Object Wrapper (Avoid in practice)
const objectString: String = new String("Hello"); // Creates an object wrapper, NOT primitive string

```

### Advanced Type Annotations

```
// 1. String Literal Types
type Direction = "NORTH" | "SOUTH" | "EAST" | "WEST";
let move: Direction = "NORTH"; // Valid
// move = "UP"; // ❌ Type Error: Type '"UP"' is not assignable to type 'Direction'.

// 2. Template Literal Types (TypeScript 4.1+)
type EventName = "click" | "hover";
type EventHandler = `on${Capitalize}`; // "onClick" | "onHover"

type HexColor = `#${string}`;
const primaryColor: HexColor = "#FF0000"; // Valid

```

## 3. Immutability Concepts

In JavaScript and TypeScript, **strings are primitive values and are completely immutable**.

Once a string is created, its sequence of characters cannot be changed in place. Any operation that appears to modify a string actually creates and returns a **new string in memory**.

```
let str = "hello";

// Attempting to mutate a character directly fails silently (or throws in strict mode)
str[0] = "H"; 
console.log(str); // Output: "hello" (Unchanged)

// Methods return NEW strings; they do not alter the original
const upperStr = str.toUpperCase();
console.log(str);      // Output: "hello"
console.log(upperStr); // Output: "HELLO"

```

## 4. Memory Allocation & V8 Internal Implementation

Understanding how strings work at the JavaScript runtime level (such as Google Chrome's V8 engine) helps in writing memory-efficient and high-performance code.

### 1. UTF-16 Encoding & Representation

* Standard strings are stored in UTF-16 format (2 bytes per code unit).

* For Latin-1 or ASCII characters, modern engines like V8 optimize storage using **One-Byte Strings** (1 byte per char) to reduce memory usage by 50%.

* Supplementary Unicode characters (like emojis `😀`) take 2 UTF-16 code units (**Surrogate Pairs**), making `.length` return `2` for a single visible emoji character.

```
console.log("A".length);   // 1 (Code Point: U+0041)
console.log("😀".length);  // 2 (Surrogate Pair: U+D83D U+DE00)

```

### 2. V8 String Representations Under the Hood

V8 represents strings internally using distinct class hierarchies to optimize operations:

* **SeqString (Sequential String):** A contiguous block of memory holding all string characters.

* **ConsString (Cons String):** Created when concatenating two large strings (`strA + strB`). Instead of copying characters into a new buffer immediately, V8 creates a binary tree referencing both strings. This makes concatenation $O(1)$ initially.

* **SlicedString:** Created when calling `.slice()` or `.substring()` on a large string. Instead of copying memory, V8 creates a reference pointing to a slice offset of the original parent string.

* **ThinString:** When a `ConsString` or `SlicedString` is interned or evaluated, V8 updates it to a `ThinString` that points directly to the flattened `SeqString`.

### 3. String Interning (The String Table)

* V8 maintains an internal hash table called the **String Table**.

* Small strings, property names, and literal keys are "interned"—meaning identical string literals point to the exact same pointer address in memory, preventing duplicate memory consumption and allowing fast pointer comparisons ($O(1)$ quality check).

## 5. Finding Characters & Indexing

```
const text = "TypeScript Programming";

// Accessing by Index
console.log(text[0]);          // "T"
console.log(text.charAt(4));    // "S"
console.log(text.charCodeAt(0)); // 84 (UTF-16 code unit at index 0)
console.log(text.codePointAt(0)); // 84 (Full Unicode code point value)

// Locating Substrings
console.log(text.indexOf("Script"));     // 4
console.log(text.lastIndexOf("r"));     // 18
console.log(text.indexOf("Python"));     // -1 (Not found)

```

## 6. Substring Extraction

TypeScript/JavaScript provides three main methods for extracting portions of a string:

| Method | Signature | Handling Negative Indices | Swapping Behavior | 
 | ----- | ----- | ----- | ----- | 
| `slice()` | `(start, end)` | Counts backwards from end of string | Returns `""` if `start > end` | 
| `substring()` | `(start, end)` | Treats negative numbers as `0` | Swaps arguments if `start > end` | 
| `substr()` *(Deprecated)* | `(start, length)` | Counts backwards from end | Deprecated, do not use | 

```
const str = "JavaScript";

// slice(start, end) -> end index is EXCLUDED
console.log(str.slice(0, 4));   // "Java"
console.log(str.slice(-6));     // "Script" (Negative counts from end)

// substring(start, end)
console.log(str.substring(0, 4));  // "Java"
console.log(str.substring(4, 0));  // "Java" (Swaps 4 and 0 automatically)
console.log(str.substring(-3));    // "JavaScript" (Treats -3 as 0)

```

## 7. String Validations

Check string structure and contents using boolean inspection methods:

```
const filename = "report-2026.pdf";

// Content Matching
console.log(filename.includes("2026"));      // true
console.log(filename.startsWith("report"));  // true
console.log(filename.endsWith(".pdf"));      // true

// Regex Validation
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const isValidEmail = emailPattern.test("user@example.com"); // true

```

## 8. Replacing String Content

```
const phrase = "The quick brown fox jumps over the lazy dog";

// Replace First Match
const simpleReplace = phrase.replace("fox", "cat"); 

// Replace All Occurrences (RegEx with global 'g' flag)
const regexReplace = phrase.replace(/the/gi, "a");

// replaceAll Method (ES2021+)
const globalReplace = phrase.replaceAll("the", "a");

// Using Replacement Function
const formatted = "item1, item2, item3".replace(/\d+/g, (match) => {
  return (parseInt(match) * 10).toString();
});
// "item10, item20, item30"

```

## 9. Splitting, Trimming, and Concatenating

```
// 1. Splitting into Arrays
const csv = "Apple,Banana,Cherry";
const list: string[] = csv.split(","); // ["Apple", "Banana", "Cherry"]

// Splitting Unicode Correctly (Spread Operator avoids breaking surrogate pairs)
const emojiStr = "A😀B";
console.log(emojiStr.split("")); // ["A", "\uD83D", "\uDE00", "B"] (Broken pair)
console.log([...emojiStr]);      // ["A", "😀", "B"] (Correct character boundary)

// 2. Trimming Whitespace
const dirtyInput = "   hello world   \n\t";
console.log(dirtyInput.trim());      // "hello world"
console.log(dirtyInput.trimStart()); // "hello world   \n\t"
console.log(dirtyInput.trimEnd());   // "   hello world"

// 3. Concatenating
const a = "Hello";
const b = "World";

const concated = a.concat(" ", b); // "Hello World"
const combined = a + " " + b;      // Preferred for performance and readability

```

## 10. Multi-line Strings & Template Literals

Template literals allow multi-line strings without string concatenation (`+`) or newline escape characters (`\n`).

```
// Multi-line String
const htmlTemplate: string = `

```

\`;

// Tagged Template Literals function sanitize(strings: TemplateStringsArray, ...values: unknown\[\]): string { return strings.reduce((acc, str, i) => { const val = values\[i - 1\]; const safeVal = typeof val === "string" ? val.replace(/</g, "<") : String(val ?? ""); return acc + safeVal + str; }); }