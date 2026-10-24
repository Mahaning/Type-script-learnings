# TypeScript String Practice Problems & Solutions

A collection of practice problems covering string manipulation, validation, searching, and algorithmic patterns in TypeScript, complete with problem statements, explanations, and code solutions.

## Table of Contents

1. [Problem 1: Valid Palindrome (Ignoring Alphanumeric Characters)](#problem-1-valid-palindrome)
2. [Problem 2: Anagram Checker](#problem-2-anagram-checker)
3. [Problem 3: Title Case Converter](#problem-3-title-case-converter)
4. [Problem 4: Run-Length Encoding (String Compression)](#problem-4-run-length-encoding)
5. [Problem 5: Longest Common Prefix](#problem-5-longest-common-prefix)
6. [Problem 6: First Non-Repeating Character](#problem-6-first-non-repeating-character)

---

## Problem 1: Valid Palindrome

### Problem Statement
Write a function `isPalindrome(s: string): boolean` that takes a string `s` and determines whether it is a palindrome. A string is a palindrome if, after converting all uppercase letters to lowercase and removing all non-alphanumeric characters, it reads the same forward and backward.

### Examples
* `isPalindrome("A man, a plan, a canal: Panama")` $\rightarrow$ `true`
* `isPalindrome("race a car")` $\rightarrow$ `false`

### Solution

```typescript
function isPalindrome(s: string): boolean {
  // Normalize string: convert to lowercase and remove non-alphanumeric characters
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, "");

  // Two-pointer approach
  let left = 0;
  let right = cleaned.length - 1;

  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }

  return true;
}

// Example Usage
console.log(isPalindrome("A man, a plan, a canal: Panama")); // true
console.log(isPalindrome("race a car")); // false
```

---

## Problem 2: Anagram Checker

### Problem Statement
Write a function `isAnagram(str1: string, str2: string): boolean` that checks whether two strings are valid anagrams of each other. An anagram is a word formed by rearranging the letters of another word, using all original letters exactly once. Ignore casing and whitespace.

### Examples
* `isAnagram("listen", "silent")` $\rightarrow$ `true`
* `isAnagram("hello", "billion")` $\rightarrow$ `false`

### Solution

```typescript
function isAnagram(str1: string, str2: string): boolean {
  const cleanStr1 = str1.toLowerCase().replace(/\s+/g, "");
  const cleanStr2 = str2.toLowerCase().replace(/\s+/g, "");

  if (cleanStr1.length !== cleanStr2.length) {
    return false;
  }

  // Count character frequencies using a Map
  const charCounts = new Map<string, number>();

  for (const char of cleanStr1) {
    charCounts.set(char, (charCounts.get(char) || 0) + 1);
  }

  for (const char of cleanStr2) {
    if (!charCounts.has(char) || charCounts.get(char) === 0) {
      return false;
    }
    charCounts.set(char, charCounts.get(char)! - 1);
  }

  return true;
}

// Example Usage
console.log(isAnagram("listen", "silent")); // true
console.log(isAnagram("hello", "billion")); // false
```

---

## Problem 3: Title Case Converter

### Problem Statement
Write a function `toTitleCase(text: string): string` that converts a sentence into title case, where the first letter of every word is capitalized and the remaining letters are in lowercase.

### Examples
* `toTitleCase("TYPESCRIPT is AWESOME")` $\rightarrow$ `"Typescript Is Awesome"`
* `toTitleCase("hello WORLD from ts")` $\rightarrow$ `"Hello World From Ts"`

### Solution

```typescript
function toTitleCase(text: string): string {
  if (!text) return "";

  return text
    .split(/\s+/)
    .map((word) => {
      if (word.length === 0) return "";
      return word[0].toUpperCase() + word.slice(1).toLowerCase();
    })
    .join(" ");
}

// Example Usage
console.log(toTitleCase("TYPESCRIPT is AWESOME")); // "Typescript Is Awesome"
console.log(toTitleCase("hello WORLD from ts")); // "Hello World From Ts"
```

---

## Problem 4: Run-Length Encoding (String Compression)

### Problem Statement
Write a function `compressString(str: string): string` that performs basic string compression using the counts of repeated characters. If the compressed string is not shorter than the original string, return the original string.

### Examples
* `compressString("aabcccccaaa")` $\rightarrow$ `"a2b1c5a3"`
* `compressString("abcd")` $\rightarrow$ `"abcd"` (Since `"a1b1c1d1"` is longer)

### Solution

```typescript
function compressString(str: string): string {
  if (str.length <= 1) return str;

  let compressed = "";
  let count = 1;

  for (let i = 0; i < str.length; i++) {
    if (i + 1 < str.length && str[i] === str[i + 1]) {
      count++;
    } else {
      compressed += str[i] + count;
      count = 1;
    }
  }

  return compressed.length < str.length ? compressed : str;
}

// Example Usage
console.log(compressString("aabcccccaaa")); // "a2b1c5a3"
console.log(compressString("abcd")); // "abcd"
```

---

## Problem 5: Longest Common Prefix

### Problem Statement
Write a function `longestCommonPrefix(strs: string[]): string` to find the longest common prefix string amongst an array of strings. If there is no common prefix, return an empty string `""`.

### Examples
* `longestCommonPrefix(["flower", "flow", "flight"])` $\rightarrow$ `"fl"`
* `longestCommonPrefix(["dog", "racecar", "car"])` $\rightarrow$ `""`

### Solution

```typescript
function longestCommonPrefix(strs: string[]): string {
  if (strs.length === 0) return "";

  // Start with the first string as the prefix candidate
  let prefix = strs[0];

  for (let i = 1; i < strs.length; i++) {
    while (!strs[i].startsWith(prefix)) {
      // Reduce the prefix length by 1 character
      prefix = prefix.substring(0, prefix.length - 1);

      if (prefix === "") return "";
    }
  }

  return prefix;
}

// Example Usage
console.log(longestCommonPrefix(["flower", "flow", "flight"])); // "fl"
console.log(longestCommonPrefix(["dog", "racecar", "car"])); // ""
```

---

## Problem 6: First Non-Repeating Character

### Problem Statement
Write a function `firstUniqChar(s: string): number` that finds the first non-repeating character in a string and returns its 0-based index. If no non-repeating character exists, return `-1`.

### Examples
* `firstUniqChar("leetcode")` $\rightarrow$ `0` (Character `'l'`)
* `firstUniqChar("loveleetcode")` $\rightarrow$ `2` (Character `'v'`)
* `firstUniqChar("aabb")` $\rightarrow$ `-1`

### Solution

```typescript
function firstUniqChar(s: string): number {
  const frequencyMap = new Map<string, number>();

  // Pass 1: Count occurrences of each character
  for (const char of s) {
    frequencyMap.set(char, (frequencyMap.get(char) || 0) + 1);
  }

  // Pass 2: Find the first character with a count of 1
  for (let i = 0; i < s.length; i++) {
    if (frequencyMap.get(s[i]) === 1) {
      return i;
    }
  }

  return -1;
}

// Example Usage
console.log(firstUniqChar("leetcode")); // 0
console.log(firstUniqChar("loveleetcode")); // 2
console.log(firstUniqChar("aabb")); // -1
```