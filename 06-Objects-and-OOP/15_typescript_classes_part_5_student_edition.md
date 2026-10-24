# TypeScript Classes — Part 5 (College & Exam Guide)

> **Core Topic:** Shared Properties, Static vs. Instance Members, Method Overloading & Constructor Overloading

---

## 💡 Quick Cheat Sheet (Exam Revision)

| Feature | Key Concept | Quick Code Pattern |
| :--- | :--- | :--- |
| **Shared Properties** | A single property in memory modified by multiple instances. | `ClassName.sharedVar` |
| **Static vs. Regular** | Static = Class level (`ClassName.x`). Regular = Object level (`this.x`). | Static saves memory for global data. |
| **Method Overloading** | Same method name, multiple parameter signatures, single implementation. | `fn(x: string): void;`<br>`fn(x: number): void;` |
| **Constructor Overloading** | Creating objects with different constructor signatures. | `constructor(x: string);`<br>`constructor(x: number);` |
| **Optional Params** | Parameters marked with `?` or default values in constructors. | `constructor(name: string, age?: number)` |

---

## 1. Shared Properties & Demonstration

A **shared property** is a `static` field that acts as a central data point across all instantiated objects of a class. When one instance updates this shared property (or a method alters it), the change is reflected globally across the entire system.

### Real-World Analogy:
Think of a classroom library. The books belong to the **Classroom** (shared), while each student's backpack belongs to the **Individual Student** (regular property).

### Demonstration Code:

```typescript
class CollegeCampus {
  // Shared property across ALL instances
  static totalStudentsOnCampus: number = 0;

  // Regular instance property (Unique per student)
  public studentName: string;

  constructor(name: string) {
    this.studentName = name;
    // Increment shared counter whenever a student arrives
    CollegeCampus.totalStudentsOnCampus++;
  }

  // Method to display campus state
  displayInfo(): void {
    console.log(`Student: ${this.studentName}`);
    console.log(`Current Total Campus Strength: ${CollegeCampus.totalStudentsOnCampus}`);
  }
}

// Instantiating 3 distinct students
const s1 = new CollegeCampus("Ananya");
const s2 = new CollegeCampus("Rahul");
const s3 = new CollegeCampus("Priya");

s1.displayInfo();
// Output: 
// Student: Ananya
// Current Total Campus Strength: 3

console.log(CollegeCampus.totalStudentsOnCampus); // Direct access via class name: 3
```

---

## 2. Static vs. Regular (Instance) Members

Understanding the core differences between static and instance members is one of the most frequently asked questions in computer science lab viva examinations.

| Comparison Factor | Regular (Instance) Member | Static Class Member |
| :--- | :--- | :--- |
| **Memory Allocation** | Created separately for **every** instance created with `new`. | Created **once** when the class definition is loaded. |
| **Access Syntax** | `instanceName.propertyName` | `ClassName.propertyName` |
| **Keyword Context** | Uses `this` inside methods. | Uses `ClassName` inside methods. |
| **Lifecycle** | Destroyed when object is garbage collected. | Exists for the entire lifespan of the application. |
| **Best Used For** | Unique object data (e.g., student name, GPA, account number). | Global configuration, shared counters, constants, utility methods. |

---

## 3. Method Overloading in TypeScript

In languages like C++ or Java, method overloading is done by defining multiple methods with the exact same name but different parameters.

In TypeScript, you write **overload signatures** (type declarations) followed by **one single implementation function** that handles all signatures.

### Steps to Write Method Overloading:
1. Define Overload Signatures (No body `{}`).
2. Define the Implementation Signature (Sufficiently generic to handle all cases).
3. Implement the internal logic using `typeof` checks.

```typescript
class ResultCalculator {
  // 1. Overload Signatures (Type definitions for caller)
  calculateGrade(score: number): string;
  calculateGrade(scores: number[]): string;

  // 2. Implementation Signature (Must accept all combinations)
  calculateGrade(scoreOrScores: number | number[]): string {
    if (typeof scoreOrScores === "number") {
      return scoreOrScores >= 50 ? "PASS" : "FAIL";
    } 
    
    if (Array.isArray(scoreOrScores)) {
      const total = scoreOrScores.reduce((sum, s) => sum + s, 0);
      const avg = total / scoreOrScores.length;
      return avg >= 50 ? "PASS" : "FAIL";
    }

    return "INVALID_INPUT";
  }
}

const calc = new ResultCalculator();

// Single score evaluation
console.log(calc.calculateGrade(75)); // Output: PASS

// Array of subject marks evaluation
console.log(calc.calculateGrade([45, 60, 30, 80])); // Output: PASS
```

---

## 4. Constructor Overloading

Similar to method overloading, TypeScript allows a class to have **multiple constructor signatures** so that objects can be initialized in different ways.

> ⚠️ **Exam Note:** A class can only have **ONE** constructor implementation block! You achieve overloading using overload signatures above the actual constructor definition.

```typescript
class StudentProfile {
  public name: string;
  public age: number;
  public course: string;

  // 1. Constructor Overload Signatures
  constructor(name: string);                       // Pattern 1: Name only
  constructor(name: string, age: number);          // Pattern 2: Name + Age
  constructor(name: string, age: number, course: string); // Pattern 3: Full details

  // 2. Single Implementation Constructor
  constructor(name: string, age?: number, course?: string) {
    this.name = name;
    this.age = age ?? 18; // Default to 18 if missing
    this.course = course ?? "General Sciences"; // Default course
  }

  showProfile(): void {
    console.log(`Name: ${this.name} | Age: ${this.age} | Course: ${this.course}`);
  }
}

// Creating objects using different constructor signatures:
const studentA = new StudentProfile("Kiran");
const studentB = new StudentProfile("Meera", 20);
const studentC = new StudentProfile("Suresh", 21, "Computer Science");

studentA.showProfile(); // Output: Name: Kiran | Age: 18 | Course: General Sciences
studentB.showProfile(); // Output: Name: Meera | Age: 20 | Course: General Sciences
studentC.showProfile(); // Output: Name: Suresh | Age: 21 | Course: Computer Science
```

---

## 5. Optional Constructor Parameters

Instead of writing full constructor overloads, you can often make parameters flexible by using **optional parameters** (`?`) or **default parameter values**.

### A. Optional Parameters (`?`)
Optional parameters must always be placed **after** mandatory parameters in the constructor.

```typescript
class CourseEnrollment {
  public studentName: string;
  public electiveCourse?: string; // Optional property (string | undefined)

  constructor(studentName: string, electiveCourse?: string) {
    this.studentName = studentName;
    this.electiveCourse = electiveCourse;
  }

  getEnrollmentSummary(): string {
    const elective = this.electiveCourse ? this.electiveCourse : "None Selected";
    return `Student: ${this.studentName} | Elective: ${elective}`;
  }
}

const e1 = new CourseEnrollment("Arjun");
const e2 = new CourseEnrollment("Bhavna", "Artificial Intelligence");

console.log(e1.getEnrollmentSummary()); // Output: Student: Arjun | Elective: None Selected
console.log(e2.getEnrollmentSummary()); // Output: Student: Bhavna | Elective: Artificial Intelligence
```

### B. Default Parameters (Cleaner Alternative)
Instead of handling `undefined` manually, you can assign default values directly inside constructor parameters.

```typescript
class LabSession {
  // Using default values in constructor parameters
  constructor(
    public labName: string,
    public durationHours: number = 2,
    public requiresSystem: boolean = true
  ) {}

  getDetails(): void {
    console.log(`Lab: ${this.labName}, Duration: ${this.durationHours}h, System Required: ${this.requiresSystem}`);
  }
}

const lab1 = new LabSession("Physics Lab"); // Uses defaults (2 hrs, System Required)
const lab2 = new LabSession("Workshop", 3, false); // Overrides defaults

lab1.getDetails(); // Output: Lab: Physics Lab, Duration: 2h, System Required: true
lab2.getDetails(); // Output: Lab: Workshop, Duration: 3h, System Required: false
```

---

## 🎯 Viva & Exam Questions Checklist

1. **Q: Can a class have multiple `constructor` code blocks in TypeScript?**
   * *A: No. TypeScript only allows ONE implementation block for a constructor, though you can declare multiple overload signatures above it.*

2. **Q: Where must optional parameters (`?`) be placed in a function or constructor signature?**
   * *A: Optional parameters MUST always be declared AFTER all required parameters.*

3. **Q: How does TypeScript distinguish between method overloads during compilation?**
   * *A: It checks argument types and counts against the overload signatures in top-to-bottom order and picks the first matching signature.*