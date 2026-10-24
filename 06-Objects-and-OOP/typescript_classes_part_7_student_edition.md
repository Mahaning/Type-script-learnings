# TypeScript Classes — Part 7 (College & Exam Guide)

> **Core Topic:** Final Class Demonstration & In-Depth Internal Mechanics (Class, Object & Property Memory Layouts with Visual Block Diagrams)

---

## 💡 Quick Cheat Sheet (Exam & Viva Memory Map)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 V8 ENGINE MEMORY ARCHITECTURE                         │
├───────────────────────────────┬──────────────────────────────┬─────────────────────────┤
│ Concept                       │ Memory Location              │ Shared or Unique?       │
├───────────────────────────────┼──────────────────────────────┼─────────────────────────┤
│ Instance Properties           │ Object Instance Allocation   │ Unique per Instance     │
│ Class Prototype Methods       │ Class.prototype Object       │ Shared across Instances │
│ Static Properties & Methods   │ Constructor Function Object  │ Single Global Copy      │
│ Access Modifiers & Readonly   │ Pure Compile-Time Checks     │ Erased at Runtime       │
└───────────────────────────────┴──────────────────────────────┴─────────────────────────┘
```

---

## 1. Final Comprehensive Class Demonstration

Below is a complete class integrating properties, constructors, instance methods, static members, getters/setters, overloads, and optional values.

```typescript
interface StudentConfig {
  id: number;
  name: string;
  department?: string;
}

class AcademicRecord {
  // 1. Static Members
  private static totalStudentsEnrolled: number = 0;
  public static readonly UNIVERSITY_CODE: string = "UNI-2026";

  // 2. Instance Fields
  public readonly id: number;
  public name: string;
  private _gpa: number = 0.0;
  public department?: string;

  // 3. Overloaded Constructors
  constructor(id: number, name: string);
  constructor(config: StudentConfig);
  constructor(param1: number | StudentConfig, param2?: string) {
    if (typeof param1 === "object") {
      this.id = param1.id;
      this.name = param1.name;
      this.department = param1.department ?? "General Sciences";
    } else {
      this.id = param1;
      this.name = param2 ?? "Unnamed Student";
      this.department = "General Sciences";
    }
    AcademicRecord.totalStudentsEnrolled++;
  }

  // 4. Getter & Setter
  public get gpa(): number {
    return this._gpa;
  }

  public set gpa(value: number) {
    if (value < 0.0 || value > 4.0) {
      throw new Error("Invalid GPA range (0.0 - 4.0)");
    }
    this._gpa = value;
  }

  // 5. Method Overloads & Instance Methods
  public updateRecord(name: string): void;
  public updateRecord(name: string, gpa: number): void;
  public updateRecord(name: string, gpa?: number): void {
    this.name = name;
    if (gpa !== undefined) {
      this.gpa = gpa;
    }
  }

  public getSummary(): string {
    return `[${this.id}] ${this.name} (${this.department}) - GPA: ${this._gpa.toFixed(2)}`;
  }

  // 6. Static Method
  public static getTotalEnrollments(): number {
    return AcademicRecord.totalStudentsEnrolled;
  }
}

// Usage
const student1 = new AcademicRecord(101, "Rahul Sharma");
student1.gpa = 3.8;

const student2 = new AcademicRecord({ id: 102, name: "Priya Patel", department: "Computer Science" });
student2.updateRecord("Priya Patel", 3.95);

console.log(student1.getSummary()); // Output: [101] Rahul Sharma (General Sciences) - GPA: 3.80
console.log(student2.getSummary()); // Output: [102] Priya Patel (Computer Science) - GPA: 3.95
console.log(`Total Students: ${AcademicRecord.getTotalEnrollments()}`); // Output: Total Students: 2
```

### 📊 Structural Block Diagram of the Class Component Layout

```
┌────────────────────────────────────────────────────────────────────────┐
│                        AcademicRecord CLASS MAP                        │
├────────────────────────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ STATIC MEMBERS (Attached to Class / Constructor Function)          │ │
│ │  ├── totalStudentsEnrolled: number = 2                             │ │
│ │  ├── UNIVERSITY_CODE: string = "UNI-2026"                          │ │
│ │  └── getTotalEnrollments(): number                                │ │
│ └────────────────────────────────────────────────────────────────────┘ │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ INSTANCE PROTOTYPE METHODS (Shared on AcademicRecord.prototype)    │ │
│ │  ├── getSummary(): string                                          │ │
│ │  ├── updateRecord(name, gpa?): void                                │ │
│ │  ├── get gpa(): number                                             │ │
│ │  └── set gpa(value: number): void                                  │ │
│ └────────────────────────────────────────────────────────────────────┘ │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ INSTANCE PROPERTIES (Unique per heap object instantiation)         │ │
│ │  ├── id: number (readonly)                                         │ │
│ │  ├── name: string                                                  │ │
│ │  ├── _gpa: number                                                  │ │
│ │  └── department?: string                                           │ │
│ └────────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Internal Working of a Class in JavaScript/V8 Engine

At the JavaScript engine level (such as Chrome's V8), **classes do not exist natively as distinct data types**. They are compiled down to **constructor functions** and **prototype objects**.

### 📊 TypeScript Code vs. Under-The-Hood Engine Transformation

```
  TYPESCRIPT AUTHORING                     COMPILED JAVASCRIPT / ENGINE LAYOUT
┌──────────────────────────┐             ┌─────────────────────────────────────────┐
│ class Student {          │             │ function Student(name) {                │
│   name: string;          │             │   this.name = name; // Instance field   │
│   constructor(name) {    │  TRANSFORM  │ }                                       │
│     this.name = name;    │ ──────────> │                                         │
│   }                      │             │ Student.prototype.study = function() {  │
│   study() { ... }        │             │   console.log(this.name + " studies");  │
│ }                        │             │ };                                      │
└──────────────────────────┘             └─────────────────────────────────────────┘
```

### 📊 Memory Relationship Block Diagram

```
STACK MEMORY                             HEAP MEMORY
┌─────────────────┐                      ┌────────────────────────────────────────┐
│  Variable:      │                      │ Constructor Function Object            │
│  Student        │ ────────────────────>│ ┌────────────────────────────────────┐ │
│  (Reference)    │                      │ │ name: "Student"                    │ │
│                 │                      │ │ prototype: [Pointer to Prototype]──┼─┐
│ └─────────────────┘                      │ └────────────────────────────────────┘ │
                                         └──────────────────┬─────────────────────┘
                                                            │
                                                            ▼
                                         ┌────────────────────────────────────────┐
                                         │ Student.prototype Object               │
                                         │ ┌────────────────────────────────────┐ │
                                         │ │ constructor: [Pointer to Student]  │ │
                                         │ │ study: function() { ... }          │ │
                                         │ └────────────────────────────────────┘ │
                                         └────────────────────────────────────────┘
```

---

## 3. Internal Working of Class Objects in Memory

When instantiating an object using `const s1 = new Student("Ananya")`, the V8 engine allocates memory and hooks up internal pointers through four explicit phases:

### 📊 4-Phase Instantiation Execution Lifecycle

```
  PHASE 1: Heap Allocation     PHASE 2: Prototype Link      PHASE 3: Constructor Exec      PHASE 4: Stack Assignment
┌──────────────────────────┐  ┌─────────────────────────┐  ┌──────────────────────────┐  ┌─────────────────────────┐
│ Allocate fresh memory    │  │ Set hidden internal     │  │ Bind 'this' to new      │  │ Assign heap memory      │
│ block inside Heap for    │─>│ __proto__ reference to  │─>│ object; populate         │─>│ address to variable 's1'│
│ object instance.         │  │ Student.prototype.      │  │ instance properties.     │  │ in Call Stack.          │
└──────────────────────────┘  └─────────────────────────┘  └──────────────────────────┘  └─────────────────────────┘
```

### 📊 Heap Memory Diagram: Multiple Object Instances Sharing Prototypes

```
CALL STACK                               HEAP MEMORY
┌───────────────┐                        ┌────────────────────────────────────────┐
│ s1 (AddressA) │───────────────────────>│ Object Instance 1 (AddressA)           │
└───────────────┘                        │ ┌────────────────────────────────────┐ │
                                         │ │ name: "Rahul"                      │ │
                                         │ │ _gpa: 3.80                         │ │
                                         │ │ __proto__: ──────────────────────┐ │ │
                                         │ └──────────────────────────────────┼─┘ │
                                         └────────────────────────────────────┼───┘
                                                                              │
┌───────────────┐                        ┌────────────────────────────────────┼───┐
│ s2 (AddressB) │───────────────────────>│ Object Instance 2 (AddressB)       │   │
└───────────────┘                        │ ┌────────────────────────────────┐ │   │
                                         │ │ name: "Priya"                  │ │   │
                                         │ │ _gpa: 3.95                     │ │   │
                                         │ │ __proto__: ──────────────────┐ │ │   │
                                         │ └──────────────────────────────┼─┘ │   │
                                         └────────────────────────────────┼───┘   │
                                                                          │       │
                                                                          ▼       ▼
                                         ┌────────────────────────────────────────┐
                                         │ Shared AcademicRecord.prototype        │
                                         │ ┌────────────────────────────────────┐ │
                                         │ │ constructor: f AcademicRecord()    │ │
                                         │ │ getSummary: f ()                   │ │
                                         │ │ updateRecord: f ()                 │ │
                                         │ └────────────────────────────────────┘ │
                                         └────────────────────────────────────────┘
```

---

## 4. Internal Working of All Property Types

### 📊 Property Types Internal Architecture Overview

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              PROPERTY TYPES IMPLEMENTATION                             │
├────────────────────┬─────────────────────────────┬─────────────────────────────────────┤
│ Property Type      │ Compile-Time (TypeScript)   │ Runtime Behavior (JavaScript/V8)    │
├────────────────────┼─────────────────────────────┼─────────────────────────────────────┤
│ Instance Field     │ Type checking               │ Attached to `this` inside instance  │
│ Readonly           │ Mutation error checks       │ Erased (standard mutable property)  │
│ Private / Protected│ Access validation checks    │ Erased (standard visible property)  │
│ Static             │ Access via Class Name only  │ Attached to Constructor Function    │
│ Getter / Setter    │ Property syntax validation  │ Object.defineProperty() accessor    │
│ Optional (?)       │ Guard checks for undefined  │ Assigned `undefined` if omitted     │
└────────────────────┴─────────────────────────────┴─────────────────────────────────────┘
```

---

### A. Instance Properties (`public`, `private`, `protected`)

`private` and `protected` modifiers exist **only during compile-time verification**. After compilation to JavaScript, they are converted into normal instance properties.

#### 📊 Block Diagram: Transpilation of Private Properties

```
TYPESCRIPT (Compile Time)                    JAVASCRIPT ENGINE (Runtime)
┌────────────────────────────────┐           ┌────────────────────────────────┐
│ class Student {                │           │ function Student() {           │
│   private _gpa: number = 3.8;  │ TRANSFORM │   // Modifiers are stripped!   │
│ }                              │ ────────> │   this._gpa = 3.8;             │
│                                │           │ }                              │
│ const s = new Student();       │           │ const s = new Student();       │
│ s._gpa; // ❌ TS Compiler Error│           │ s._gpa; // ⚠️ Works in raw JS! │
└────────────────────────────────┘           └────────────────────────────────┘
```

---

### B. Readonly Properties (`readonly`)

The `readonly` modifier prevents reassignment after constructor initialization. This protection is completely handled by the TypeScript compiler and is removed in runtime JavaScript.

#### 📊 Block Diagram: Readonly Compilation Lifecycle

```
┌──────────────────────────────────┐
│  TypeScript Authoring            │
│  readonly id: number = 101;      │
└────────────────┬─────────────────┘
                 │
                 ▼
┌──────────────────────────────────┐
│  TypeScript Compiler (tsc)       │
│  Checks for illegal assignments: │
│  s1.id = 202; ❌ REJECTED        │
└────────────────┬─────────────────┘
                 │ (Strips type assertions)
                 ▼
┌──────────────────────────────────┐
│  JavaScript Runtime Execution    │
│  this.id = 101;                  │
│  (Becomes a regular JS property) │
└──────────────────────────────────┘
```

---

### C. Static Properties (`static`)

Static members are not stored on instance objects or on `prototype`. They are attached directly to the **constructor function object itself**.

#### 📊 Block Diagram: Memory Isolation of Static Members

```
┌────────────────────────────────────────────────────────────────────────┐
│ CONSTRUCTOR FUNCTION OBJECT: AcademicRecord                            │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ totalStudentsEnrolled: 2                                           │ │
│ │ UNIVERSITY_CODE: "UNI-2026"                                        │ │
│ │ getTotalEnrollments: f ()                                          │ │
│ └────────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
          │ (Instances DO NOT inherit these properties directly)
          ├──────────────────────────────┐
          ▼                              ▼
┌───────────────────┐          ┌───────────────────┐
│ Instance student1 │          │ Instance student2 │
│ (No static fields)│          │ (No static fields)│
└───────────────────┘          └───────────────────┘
```

---

### D. Getters and Setters (`get` / `set`)

Getters and setters transform a property access into an internal function execution via `Object.defineProperty`.

#### 📊 Block Diagram: Intercepting Reads and Writes via Getters/Setters

```
READING PROPERTY (student1.gpa)
┌──────────┐      Property Read       ┌────────────────────────────────┐      Return Value
│ Consumer │ ───────────────────────> │ get gpa()                      │ ─────────────────────> 3.80
└──────────┘                          │   return this._gpa;            │
                                      └────────────────────────────────┘

WRITING PROPERTY (student1.gpa = 3.9)
┌──────────┐      Assign Value        ┌────────────────────────────────┐      Validation Passed
│ Consumer │ ───────────────────────> │ set gpa(value)                 │ ─────────────────────> Updates
└──────────┘                          │   if (value < 0 || value > 4)  │                      this._gpa
                                      │     throw Error;               │                      Memory Slot
                                      │   this._gpa = value;           │
                                      └────────────────────────────────┘
```

---

### E. Optional Properties (`?`)

Optional properties (`department?: string`) allow a property to be omitted during object creation.

#### 📊 Block Diagram: Optional Property Memory State

```
Option A: Value Supplied                              Option B: Value Omitted
new AcademicRecord({ ..., department: "CS" })         new AcademicRecord({ ... })

┌───────────────────────────────────────┐            ┌───────────────────────────────────────┐
│ Object Instance Memory                │            │ Object Instance Memory                │
│ ┌───────────────────────────────────┐ │            │ ┌───────────────────────────────────┐ │
│ │ id: 102                           │ │            │ │ id: 101                           │ │
│ │ name: "Priya"                     │ │            │ │ name: "Rahul"                     │ │
│ │ department: "Computer Science"    │ │            │ │ department: undefined             │ │
│ └───────────────────────────────────┘ │            │ └───────────────────────────────────┘ │
└───────────────────────────────────────┘            └───────────────────────────────────────┘
```

---

## 🎯 Viva & Exam Questions (Part 7)

1. **Q: How does a class method differ from an instance property in terms of memory location?**
   * *A:* Instance properties are allocated inside each distinct object on the Heap. Class methods are stored once on `ClassName.prototype` and shared across all instances via `__proto__` pointers.

2. **Q: Are TypeScript `private` and `readonly` modifiers preserved in JavaScript output?**
   * *A:* No. Both modifiers are checked at compile time by TypeScript and stripped out during compilation, yielding standard JavaScript properties at runtime.

3. **Q: What is the internal memory location of static class properties?**
   * *A:* Static properties are stored directly on the constructor function object itself, completely separate from object instances and `prototype` objects.