# TypeScript Arrays (Part 3): Multidimensional Arrays & Dynamic Dimensions

Welcome to **Part 3** of our TypeScript Arrays series. After exploring iteration, function integration, and internal V8 memory mechanics in Parts 1 and 2, this guide focuses on **Multidimensional Arrays**, **Tabular Data Processing**, **Dynamic Dimension Allocation**, and performing **CRUD Operations** on matrix structures.

## Table of Contents

1. [Introduction to Multidimensional Arrays](#1-introduction-to-multidimensional-arrays)

2. [Types & Declarations for N-Dimensional Arrays](#2-types--declarations-for-n-dimensional-arrays)

3. [Accessing Elements & Indexing Rules](#3-accessing-elements--indexing-rules)

4. [CRUD Operations on 2D Matrices](#4-crud-operations-on-2d-matrices)

   * [Create (Initialization)](#a-create-initialization)

   * [Read (Access & Traversal)](#b-read-access--traversal)

   * [Update (In-Place & Immutability)](#c-update-in-place--immutability)

   * [Delete (Row & Column Removal)](#d-delete-row--column-removal)

5. [Tabular Formatted Data (Tables & Grids)](#5-tabular-formatted-data-tables--grids)

6. [Dynamic Dimensional Arrays (N-Dimensions)](#6-dynamic-dimensional-arrays-n-dimensions)

7. [Summary](#7-summary)

---

## 1. Introduction to Multidimensional Arrays

A **Multidimensional Array** in TypeScript is an array whose elements are themselves arrays. They are commonly used to represent grids, matrices, tabular datasets, game boards, or spatial coordinate systems.

```
       Visual Representation of a 2D Array (3x3 Matrix)

            Col 0      Col 1      Col 2
          +----------+----------+----------+
  Row 0   |  [0][0]  |  [0][1]  |  [0][2]  |
          +----------+----------+----------+
  Row 1   |  [1][0]  |  [1][1]  |  [1][2]  |
          +----------+----------+----------+
  Row 2   |  [2][0]  |  [2][1]  |  [2][2]  |
          +----------+----------+----------+
```

---

## 2. Types & Declarations for N-Dimensional Arrays

TypeScript provides clean syntax for declaring multi-layered array types using standard bracket notation or generic `Array<T>` wrappers.

### A. 2D Arrays (Matrices)

```typescript
// Bracket notation syntax
let matrix2D: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// Generic syntax equivalent
let matrix2DGeneric: Array<Array<number>> = [
  [1, 2, 3],
  [4, 5, 6]
];
```

### B. 3D Arrays (Cubes / Volumetric Data)

```typescript
// 3D Array Type (Array of 2D Arrays)
let cube3D: number[][][] = [
  [
    [1, 2], [3, 4]
  ],
  [
    [5, 6], [7, 8]
  ]
];
```

### C. N-Dimensional Arrays via Recursive Types

If you need an array of arbitrary depth where dimensions are dynamic:

```typescript
type RecursiveArray<T> = Array<T | RecursiveArray<T>>;

const dynamicDepthArray: RecursiveArray<number> = [
  1,
  [2, 3],
  [4, [5, 6, [7]]]
];
```

---

## 3. Accessing Elements & Indexing Rules

Elements in multidimensional arrays are accessed by chaining square brackets `[row][column][depth]`.

```typescript
const grid: string[][] = [
  ["A0", "B0", "C0"],
  ["A1", "B1", "C1"],
  ["A2", "B2", "C2"]
];

// Accessing Row 1, Column 2
const value = grid[1][2]; // Output: "C1"

// Safe access with optional chaining
const safeValue = grid[5]?.[0]; // Output: undefined (no runtime error)
```

> ⚠️ **Out-of-Bounds Precaution**: Attempting `grid[5][0]` directly when row 5 does not exist will throw a runtime `TypeError: Cannot read properties of undefined (reading '0')`. Always validate dimensions or use optional chaining `.?[]`.

---

## 4. CRUD Operations on 2D Matrices

### A. Create (Initialization)

#### 1. Literal Creation
```typescript
const board: (string | null)[][] = [
  [null, "X", "O"],
  ["O", "X", null],
  [null, null, "X"]
];
```

#### 2. Dynamic Matrix Allocation using `Array.from`
Avoid using `.fill([])` directly, as it assigns the **same memory reference** to every row!

```typescript
// ❌ WRONG (Mutating one row mutates all rows!):
// const badGrid = new Array(3).fill(new Array(3).fill(0));

// ✅ CORRECT: Unique memory allocation for each row
function createMatrix<T>(rows: number, cols: number, initialValue: T): T[][] {
  return Array.from({ length: rows }, () => Array(cols).fill(initialValue));
}

const grid3x3 = createMatrix<number>(3, 3, 0);
// Result:
// [ [0, 0, 0],
//   [0, 0, 0],
//   [0, 0, 0] ]
```

---

### B. Read (Access & Traversal)

#### 1. Full Traversal via Nested Loops
```typescript
function printMatrix<T>(matrix: T[][]): void {
  for (let r = 0; r < matrix.length; r++) {
    let rowString = "";
    for (let c = 0; c < matrix[r].length; c++) {
      rowString += `${matrix[r][c]}\t`;
    }
    console.log(rowString);
  }
}
```

#### 2. Functional Traversal (`flatMap` / `forEach`)
```typescript
const matrix = [
  [1, 2],
  [3, 4]
];

// Flatten and process all elements
const allElements = matrix.flatMap(row => row); // [1, 2, 3, 4]
```

---

### C. Update (In-Place & Immutability)

#### 1. Direct Mutation (In-Place)
```typescript
function updateCell<T>(matrix: T[][], row: number, col: number, newValue: T): boolean {
  if (matrix[row] && matrix[row][col] !== undefined) {
    matrix[row][col] = newValue;
    return true;
  }
  return false;
}
```

#### 2. Immutable Update (Functional / React Pattern)
```typescript
function updateCellImmutable<T>(matrix: T[][], row: number, col: number, newValue: T): T[][] {
  return matrix.map((r, rowIndex) =>
    rowIndex === row
      ? r.map((cell, colIndex) => (colIndex === col ? newValue : cell))
      : r
  );
}
```

---

### D. Delete (Row & Column Removal)

#### 1. Delete an Entire Row
```typescript
function removeRow<T>(matrix: T[][], rowIndex: number): T[][] {
  return matrix.filter((_, index) => index !== rowIndex);
}
```

#### 2. Delete an Entire Column
```typescript
function removeColumn<T>(matrix: T[][], colIndex: number): T[][] {
  return matrix.map(row => row.filter((_, index) => index !== colIndex));
}
```

---

## 5. Tabular Formatted Data (Tables & Grids)

Multidimensional arrays are frequently used to manage tabular data (rows and columns with header records).

```typescript
type TableData = (string | number)[][];

const userTable: TableData = [
  ["ID", "Name", "Age", "Role"],       // Row 0: Headers
  [101, "Alice", 28, "Administrator"], // Row 1: Data
  [102, "Bob", 34, "Developer"],      // Row 2: Data
  [103, "Charlie", 22, "Designer"]     // Row 3: Data
];

// Utility function to convert Tabular Array into an Array of Objects
function tableToObjects<T extends Record<string, any>>(table: TableData): T[] {
  const [headers, ...rows] = table;
  
  return rows.map((row) => {
    const obj: any = {};
    headers.forEach((header, index) => {
      obj[String(header).toLowerCase()] = row[index];
    });
    return obj as T;
  });
}

interface UserRecord {
  id: number;
  name: string;
  age: number;
  role: string;
}

const formattedUsers = tableToObjects<UserRecord>(userTable);
console.log(formattedUsers);
/*
Output:
[
  { id: 101, name: "Alice", age: 28, role: "Administrator" },
  { id: 102, name: "Bob", age: 34, role: "Developer" },
  { id: 103, name: "Charlie", age: 22, role: "Designer" }
]
*/
```

---

## 6. Dynamic Dimensional Arrays (N-Dimensions)

When creating N-Dimensional arrays dynamically at runtime where the number of dimensions is passed as a variable (e.g., creating a $3 \times 3 \times 3 \times 3$ hypercube):

```typescript
/**
 * Recursively creates an N-Dimensional array initialized with a default value.
 * @param dimensions Array specifying length for each dimension e.g. [3, 3, 3]
 * @param initialValue Value to populate the deepest elements
 */
function createNDimensionalArray(dimensions: number[], initialValue: any = 0): any {
  if (dimensions.length === 0) {
    return initialValue;
  }

  const [currentDim, ...remainingDims] = dimensions;
  const arr = new Array(currentDim);

  for (let i = 0; i < currentDim; i++) {
    arr[i] = createNDimensionalArray(remainingDims, initialValue);
  }

  return arr;
}

// Example 1: Create 2D Array (2x4)
const dynamic2D = createNDimensionalArray([2, 4], "Empty");
console.log(dynamic2D);
// [ ["Empty", "Empty", "Empty", "Empty"], ["Empty", "Empty", "Empty", "Empty"] ]

// Example 2: Create 4D Hypercube (2x2x2x2)
const hyperCube = createNDimensionalArray([2, 2, 2, 2], 1);
console.log(hyperCube[0][1][0][1]); // Output: 1
```

---

## 7. Summary

* **Type Signatures**: Match dimensions using brackets (`T[][]` for 2D, `T[][][]` for 3D) or recursive generic wrappers.

* **Safe Creation**: Always use `Array.from()` or recursive functions to construct multidimensional arrays to prevent reference sharing issues.

* **CRUD Operations**: Use `.map()` and `.filter()` for immutable matrix state updates (essential for modern React/Redux workflows) or index-based assignments for in-place performance updates.

* **Tabular Data**: Structured 2D arrays easily convert into arrays of typed objects for API communication and reporting tools.