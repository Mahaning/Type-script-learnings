# TypeScript Simple Arrays: Operations, Looping & Patterns

A practical guide covering essential array operations, common looping techniques, and algorithmic patterns in TypeScript, along with problem statements and solution examples.

---

## Table of Contents
1. [Basic Array Operations](#1-basic-array-operations)
2. [Looping Techniques](#2-looping-techniques)
3. [Common Array Patterns](#3-common-array-patterns)
4. [Practice Problem Statements & Solutions](#4-practice-problem-statements--solutions)

---

## 1. Basic Array Operations

TypeScript provides static typing over JavaScript's built-in `Array` methods.

### Creation and Typing
```typescript
// Explicit typing
const numbers: number[] = [1, 2, 3, 4, 5];
const names: Array<string> = ["Alice", "Bob", "Charlie"];

// Readonly arrays (immutable)
const immutableScores: readonly number[] = [90, 85, 88];
```

### Mutating Operations (Adding / Removing)
```typescript
const fruits: string[] = ["apple", "banana"];

// Push: Add to end
fruits.push("cherry"); // ["apple", "banana", "cherry"]

// Pop: Remove from end
const last = fruits.pop(); // returns "cherry", fruits: ["apple", "banana"]

// Unshift: Add to beginning
fruits.unshift("mango"); // ["mango", "apple", "banana"]

// Shift: Remove from beginning
const first = fruits.shift(); // returns "mango", fruits: ["apple", "banana"]

// Splice: Insert / Remove at specific index
// splice(startIndex, deleteCount, ...itemsToAdd)
fruits.splice(1, 1, "grape"); // removes 1 item at index 1, inserts "grape"
```

### Non-Mutating Operations
```typescript
const nums = [10, 20, 30, 40, 50];

// Slice: Extract subset (start, end Excluded)
const sub = nums.slice(1, 4); // [20, 30, 40]

// Concat: Combine arrays
const combined = nums.concat([60, 70]); // [10, 20, 30, 40, 50, 60, 70]

// Includes & IndexOf
const hasTwenty = nums.includes(20); // true
const index30 = nums.indexOf(30);   // 2
```

---

## 2. Looping Techniques

### Traditional `for` Loop
Best when you need index manipulation or early termination (`break`/`continue`).

```typescript
const items: string[] = ["a", "b", "c"];

for (let i = 0; i < items.length; i++) {
  console.log(`Index ${i}: ${items[i]}`);
}
```

### `for...of` Loop
Best for straightforward iteration over values without needing indices.

```typescript
const scores: number[] = [88, 92, 79];

for (const score of scores) {
  console.log(score);
}
```

### Functional Array Iterators (`forEach`, `map`, `filter`, `reduce`)

```typescript
const values: number[] = [1, 2, 3, 4, 5];

// 1. forEach: Execute side effects
values.forEach((val, index) => {
  console.log(`Item at ${index} is ${val}`);
});

// 2. map: Transform elements into a new array
const doubled: number[] = values.map((val) => val * 2); // [2, 4, 6, 8, 10]

// 3. filter: Select elements matching a condition
const evens: number[] = values.filter((val) => val % 2 === 0); // [2, 4]

// 4. reduce: Accumulate elements to a single value
const sum: number = values.reduce((acc, curr) => acc + curr, 0); // 15
```

---

## 3. Common Array Patterns

### Pattern A: Linear Search / Condition Checking
```typescript
// Check if any element meets condition
const hasNegative = numbers.some((n) => n < 0);

// Check if ALL elements meet condition
const allPositive = numbers.every((n) => n > 0);

// Find first matching element or undefined
const firstEven = numbers.find((n) => n % 2 === 0);
```

### Pattern B: Accumulator / Frequency Counting
Used to aggregate data into an object or map.

```typescript
const letters: string[] = ["a", "b", "a", "c", "b", "a"];

const frequency = letters.reduce<Record<string, number>>((acc, char) => {
  acc[char] = (acc[char] || 0) + 1;
  return acc;
}, {});

// Result: { a: 3, b: 2, c: 1 }
```

### Pattern C: Two-Pointer Technique
Efficiently search pairs or reverse arrays in $O(N)$ time.

```typescript
function isPalindrome<T>(arr: T[]): boolean {
  let left = 0;
  let right = arr.length - 1;

  while (left < right) {
    if (arr[left] !== arr[right]) {
      return false;
    }
    left++;
    right--;
  }
  return true;
}
```

---

## 4. Practice Problem Statements & Solutions

### Problem 1: Maximum Subarray Sum Target (Finding Max Element)
**Problem Statement:**  
Given an array of numbers, write a function `findMax` that returns the largest number in the array. Return `null` if the array is empty.

**Solution:**
```typescript
function findMax(numbers: number[]): number | null {
  if (numbers.length === 0) return null;

  let maxVal = numbers[0];
  for (let i = 1; i < numbers.length; i++) {
    if (numbers[i] > maxVal) {
      maxVal = numbers[i];
    }
  }
  return maxVal;
}

// Example usage:
console.log(findMax([3, 7, 2, 9, 5])); // Output: 9
```

---

### Problem 2: Remove Duplicates from Array
**Problem Statement:**  
Write a function `removeDuplicates<T>` that takes an array of primitive values and returns a new array with all duplicate items removed, preserving the original order.

**Solution:**
```typescript
// Approach 1: Using Set
function removeDuplicates<T>(arr: T[]): T[] {
  return Array.from(new Set(arr));
}

// Approach 2: Using filter & indexOf
function removeDuplicatesFilter<T>(arr: T[]): T[] {
  return arr.filter((item, index) => arr.indexOf(item) === index);
}

// Example usage:
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 1])); // Output: [1, 2, 3, 4]
```

---

### Problem 3: Two Sum Problem
**Problem Statement:**  
Given an array of integers `nums` and an integer `target`, return the indices of the two numbers such that they add up to `target`. Assume exactly one solution exists.

**Solution:**
```typescript
function twoSum(nums: number[], target: number): [number, number] | null {
  const map = new Map<number, number>();

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    if (map.has(complement)) {
      return [map.get(complement)!, i];
    }

    map.set(nums[i], i);
  }

  return null;
}

// Example usage:
console.log(twoSum([2, 7, 11, 15], 9)); // Output: [0, 1]
```

---

### Problem 4: Chunk Array
**Problem Statement:**  
Write a function `chunkArray<T>` that accepts an array `arr` and a chunk size `size`, returning an array of arrays where each subarray has a maximum length equal to `size`.

**Solution:**
```typescript
function chunkArray<T>(arr: T[], size: number): T[][] {
  if (size <= 0) throw new Error("Chunk size must be greater than 0");

  const chunks: T[][] = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }

  return chunks;
}

// Example usage:
console.log(chunkArray([1, 2, 3, 4, 5, 6, 7], 3));
// Output: [[1, 2, 3], [4, 5, 6], [7]]
```