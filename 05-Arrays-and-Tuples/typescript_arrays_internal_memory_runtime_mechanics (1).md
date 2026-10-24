# TypeScript & JavaScript Arrays: Internal Working, Memory Allocation & Mechanics

Understanding how arrays work under the hood in JavaScript engines (like Google Chrome's **V8**, Node.js, or Firefox's **SpiderMonkey**) is critical for writing high-performance TypeScript applications.

Because TypeScript compiles down to JavaScript, TypeScript arrays inherit the exact memory models and internal runtime mechanisms of the underlying JavaScript engine.

---

## Table of Contents

1. [Low-Level Memory Allocation (C++ vs V8 Engine)](#1-low-level-memory-allocation-c-vs-v8-engine)
2. [V8 Elements Kinds & Memory Optimization](#2-v8-elements-kinds--memory-optimization)
3. [Memory Layout & Indexing Mechanics ($O(1)$)](#3-memory-layout--indexing-mechanics-o1)
4. [Internal Mechanics of Array Operations](#4-internal-mechanics-of-array-operations)
   - [Accessing & Updating ($O(1)$)](#a-accessing--updating-o1)
   - [Adding Elements (Push vs Unshift & Dynamic Capacity Growth)](#b-adding-elements-push-vs-unshift--dynamic-capacity-growth)
   - [Deleting Elements (Pop vs Shift vs Splice)](#c-deleting-elements-pop-vs-shift-vs-splice)
   - [Replacing Elements](#d-replacing-elements)
   - [Copying Arrays (Shallow vs Deep Copy Mechanics)](#e-copying-arrays-shallow-vs-deep-copy-mechanics)
   - [Sorting Mechanics (Timsort Algorithm)](#f-sorting-mechanics-timsort-algorithm)
5. [Summary Time & Space Complexity Reference](#5-summary-time--space-complexity-reference)

---

## 1. Low-Level Memory Allocation (C++ vs V8 Engine)

### Classical Static Arrays (C++ / Rust style)
In lower-level languages, an array is a **contiguous block of memory** allocated on the heap or stack with a fixed size.

```
Address:    0x1000       0x1004       0x1008       0x100C
Memory:  [  1001   |    1002    |    1003    |    1004    ]
Index:       [0]          [1]          [2]          [3]
Offset:  Base + 0*4   Base + 1*4   Base + 2*4   Base + 3*4
```

### JavaScript / V8 Engine Dynamic Arrays
JavaScript arrays are **objects with specialized array properties** managed by internal C++ classes in the engine (e.g., `v8::internal::JSArray`).

```
+-------------------------------------------------------------+
|                     JSArray Header Object                   |
+-------------------------------------------------------------+
| - Map (Hidden Class / Shape pointer)                        |
| - Prototype Pointer -> Array.prototype                      |
| - Length Property: 4                                        |
| - Elements Pointer ----------------------------------+      |
+------------------------------------------------------|------+
                                                       |
                                                       v
+-------------------------------------------------------------+
|              FixedArray Backing Store (Heap)                |
+-------------------------------------------------------------+
| Capacity: 8  | Occupied: 4                                  |
| [0]: 10  | [1]: 20  | [2]: 30  | [3]: 40  | [4..7]: Hole   |
+-------------------------------------------------------------+
```

A JS Array consists of two memory allocations:
1. **JSArray Object Header**: Contains pointer metadata, class shapes, and the `length` property.
2. **Backing Store (`FixedArray`)**: Contiguous buffer holding the actual element data or pointers to heap values.

---

## 2. V8 Elements Kinds & Memory Optimization

V8 dynamically tracks the type of elements stored inside an array to optimize memory layout and execution speed.

```
                              [ Array Creation ]
                                      |
                     +----------------+----------------+
                     |                |                |
                PACKED_SMI      PACKED_DOUBLE    PACKED_ELEMENTS
                     |                |                |
             (Add undefined/hole)     |                |
                     v                v                v
                 HOLEY_SMI      HOLEY_DOUBLE     HOLEY_ELEMENTS
```

### 1. Packed vs. Holey
- **PACKED (Dense)**: All indices from `0` to `length - 1` contain valid data. Allows fast $O(1)$ memory offset lookups.
- **HOLEY (Sparse)**: Array contains missing elements/holes (e.g., `let arr = []; arr[100] = 5;`). Requires checking prototypes when accessing missing indices, slowing down operations.

### 2. Element Value Kinds
- **SMI (Small Integers)**: 31-bit signed integers stored directly inside pointer slots (no heap allocation overhead).
- **DOUBLE**: Unboxed 64-bit IEEE 754 floating-point numbers allocated in a contiguous float block.
- **ELEMENTS**: Mixed types, strings, or objects stored as pointers to managed Heap Objects.

> **Performance Rule:** Once an array transitions down the matrix (e.g., from `PACKED_SMI` to `PACKED_ELEMENTS` by pushing a string), it **never transitions back** to a faster kind!

---

## 3. Memory Layout & Indexing Mechanics ($O(1)$)

### Address Calculation Formula

For a **PACKED** backing store, index access does not use hash tables or loops. It calculates memory addresses directly using pointer arithmetic in **$O(1)$ time**:

$$\text{Memory Address} = \text{Base Address} + (\text{Index} \times \text{Element Byte Size})$$

```
Base Address: 0x2000
Element Size: 8 bytes (64-bit pointer/double)

Index [0] => 0x2000 + (0 * 8) = 0x2000
Index [1] => 0x2000 + (1 * 8) = 0x2008
Index [2] => 0x2000 + (2 * 8) = 0x2010
Index [3] => 0x2000 + (3 * 8) = 0x2018
```

```
[Access Request: arr[2]]
        |
        v
Check: Index < Length? (2 < 4) ---> YES
        |
        v
Calculate: Base + (2 * 8) = 0x2010
        |
        v
Read Memory at 0x2010 directly in 1 CPU cycle -> Return Value
```

---

## 4. Internal Mechanics of Array Operations

### A. Accessing & Updating ($O(1)$)

Accessing or replacing a value at a known index is an **$O(1)$ constant-time** operation.

```typescript
let arr: number[] = [10, 20, 30];

// Update index 1
arr[1] = 99;
```

```
Before Update:
Backing Store: [0x2000: 10] [0x2008: 20] [0x2010: 30]

Step 1: Compute offset for Index 1 -> 0x2008
Step 2: Overwrite value at 0x2008 with 99

After Update:
Backing Store: [0x2000: 10] [0x2008: 99] [0x2010: 30]
```

---

### B. Adding Elements (Push vs Unshift & Dynamic Capacity Growth)

#### 1. `push()` — Append to End ($O(1)$ Amortized)
When the backing `FixedArray` fills up, V8 allocates a **new larger buffer** (typically $1.5\times \text{to} \times 2\times$ the previous size), copies the existing elements over, and frees the old buffer.

```typescript
let list = [1, 2]; // Initial capacity: 2
list.push(3);      // Requires re-allocation!
```

```
Step 1: Existing Buffer Full
Capacity: 2 | Occupied: 2 | Memory: [ 1 | 2 ]

Step 2: Re-allocation Triggered (Growth Factor ~1.5x - 2x)
New Capacity: 4
Allocate New Memory Buffer: [ _ | _ | _ | _ ]

Step 3: Copy Old Data
Memory Buffer: [ 1 | 2 | _ | _ ]

Step 4: Append New Element & Update Length
Memory Buffer: [ 1 | 2 | 3 | _ ]
Occupied: 3 | Capacity: 4
```

#### 2. `unshift()` — Prepend to Start ($O(n)$ Time)
Adding elements to the beginning forces **every existing element to shift right by 1 position** to free up index `0`.

```typescript
let arr = [2, 3, 4];
arr.unshift(1); // O(n) operation
```

```
Initial:      [ 2 ] [ 3 ] [ 4 ] [ _ ]
               [0]   [1]   [2]   [3]

Step 1: Shift elements right (Index i -> i+1)
Move [2] -> [3]: [ 2 ] [ 3 ] [ 4 ] [ 4 ]
Move [1] -> [2]: [ 2 ] [ 3 ] [ 3 ] [ 4 ]
Move [0] -> [1]: [ 2 ] [ 2 ] [ 3 ] [ 4 ]

Step 2: Insert new item at index 0
Write [1] -> [0]: [ 1 ] [ 2 ] [ 3 ] [ 4 ]
```

---

### C. Deleting Elements (Pop vs Shift vs Splice)

#### 1. `pop()` — Remove from End ($O(1)$)
Decrements the internal `length` property and marks the memory slot as available. No element shifting required.

```
Initial:  [ 10 | 20 | 30 | 40 ]  Length: 4
Operation: arr.pop()
Result:   [ 10 | 20 | 30 | HOLE ] Length: 3
```

#### 2. `shift()` — Remove from Start ($O(n)$)
Removes the element at index `0` and **shifts all remaining elements left by 1 position**.

```
Initial:      [ 10 ] [ 20 ] [ 30 ] [ 40 ]
               [0]    [1]    [2]    [3]

Step 1: Remove [0]
               [ -- ] [ 20 ] [ 30 ] [ 40 ]

Step 2: Shift all items Left by 1
Move [1] -> [0]: [ 20 ] [ 20 ] [ 30 ] [ 40 ]
Move [2] -> [1]: [ 20 ] [ 30 ] [ 30 ] [ 40 ]
Move [3] -> [2]: [ 20 ] [ 30 ] [ 40 ] [ -- ]

Step 3: Update Length to 3
```

#### 3. `splice(index, deleteCount, ...items)` ($O(n)$)
Requires shifting trailing elements left (if deleting) or right (if inserting).

---

### D. Replacing Elements

Direct replacing via `arr[i] = newValue` takes $O(1)$ time. Using `splice(index, 1, newItem)` takes $O(n)$ in the worst case if indices shift, but $O(1)$ if the number of deleted items equals the number of inserted items.

```typescript
let arr = ["A", "B", "C"];
arr[1] = "Z"; // O(1) direct address replacement
```

---

### E. Copying Arrays (Shallow vs Deep Copy Mechanics)

#### 1. Reference Assignment (No Copy)
Assigning an array variable copies only the **memory pointer** pointing to the object header. Both variables mutate the exact same array in memory.

```typescript
let original = [10, 20];
let alias = original; // Copies memory address pointer
```

```
original Variable Pointer ---\
                              +---> [ JSArray Object Header ] ---> [ Backing Store ]
alias Variable Pointer ------/
```

#### 2. Shallow Copy (`[...arr]`, `arr.slice()`, `Array.from()`)
Allocates a **new `JSArray` header** and **new `FixedArray` backing store**, copying all primitive values or object reference pointers directly.

```typescript
let obj = { role: "admin" };
let original = [1, obj];
let shallowCopy = [...original];
```

```
[ Original Array ] ----> Backing Store: [ 1 | Pointer 0x9000 ]
                                                   |
[ Shallow Copy   ] ----> Backing Store: [ 1 | Pointer 0x9000 ]
                                                   |
                                                   v
                                          Heap Object { role: "admin" }
```

#### 3. Deep Copy (`structuredClone(arr)`)
Recursively duplicates all nested objects, arrays, and primitive fields into brand new unique heap allocations.

---

### F. Sorting Mechanics (Timsort Algorithm)

Since V8 v7.0+, JavaScript uses **Timsort** (a hybrid sorting algorithm derived from Merge Sort and Insertion Sort).

```
                            Input Array
                                 |
           +---------------------+---------------------+
           v                                           v
Small Arrays (Length < 64)                  Large Arrays (Length >= 64)
           |                                           |
    Insertion Sort                               Split into "Runs"
           |                                           |
           |                               Merge Runs via Stack Rule
           |                                           |
           +---------------------+---------------------+
                                 |
                                 v
                             Sorted Array
```

1. **Minrun Division**: Small subarrays (lengths < 64) are sorted using **Insertion Sort** ($O(n^2)$ worst, but extremely fast for tiny contiguous memory sets).
2. **Merging Runs**: Larger arrays are divided into natural ordered blocks called **Runs**, which are merged using a modified **Merge Sort** with $O(n \log n)$ time complexity and guaranteed stability.

---

## 5. Summary Time & Space Complexity Reference

| Operation | Method / Syntax | Time Complexity | Space Complexity | Internal Mechanism |
| :--- | :--- | :--- | :--- | :--- |
| **Index Access** | `arr[i]` | $O(1)$ | $O(1)$ | Direct offset memory math |
| **Index Update** | `arr[i] = val` | $O(1)$ | $O(1)$ | Overwrites address location |
| **Append End** | `push(val)` | $O(1)$ amortized | $O(1)$ | Capacity check + offset write |
| **Remove End** | `pop()` | $O(1)$ | $O(1)$ | Decrements `length` counter |
| **Prepend Start** | `unshift(val)` | $O(n)$ | $O(1)$ | Shifts $n$ elements right |
| **Remove Start** | `shift()` | $O(n)$ | $O(1)$ | Shifts $n$ elements left |
| **Insert / Delete** | `splice()` | $O(n)$ | $O(1)$ | Memory block relocation |
| **Slice / Copy** | `[...arr]`, `slice()` | $O(n)$ | $O(n)$ | Allocates new backing store |
| **Sorting** | `sort()` | $O(n \log n)$ | $O(n)$ | Timsort algorithm |