# TypeScript Classes — Part 6 (College & Exam Guide)

> **Core Topic:** Testing Overloaded Constructors, Implementing Method Overloading & Internal Overload Logic

## 💡 Quick Cheat Sheet (Exam Revision)

| Concept | Key Definition | Important Rules |
| :--- | :--- | :--- |
| **Constructor Overloads** | Multiple constructor signatures providing different ways to create an object. | Only **one** implementation constructor is allowed. |
| **Method Overloads** | Multiple method signatures with different parameters for a single method name. | Must have a **compatible implementation signature** that covers all overloads. |
| **Type Narrowing / Dispatch Logic** | The internal `if-else` or `typeof` checks used to determine which overload was called. | Handled inside the single implementation block using `typeof` or `instanceof`. |

---

## 1. Testing Overloaded Constructors

When testing overloaded constructors, the objective is to verify that the class instantiates correctly with different argument types and numbers without throwing runtime errors or type errors.

### College Example: Creating Student Profiles

Suppose a college system allows creating a `StudentProfile` using either:
1. **USN and Name** (Basic admission entry)
2. **USN, Name, and Marks Array** (Detailed academic entry)
3. **An existing JSON object** (Bulk database import)

```typescript
class StudentProfile {
  public usn: string;
  public name: string;
  public marks: number[];

  // 1. Overload Signatures (Public facing contract)
  constructor(usn: string, name: string);
  constructor(usn: string, name: string, marks: number[]);
  constructor(rawConfig: { usn: string; name: string; marks?: number[] });

  // 2. Implementation Constructor (Handles all cases internally)
  constructor(
    param1: string | { usn: string; name: string; marks?: number[] },
    param2?: string,
    param3?: number[]
  ) {
    if (typeof param1 === "object") {
      // Handled object payload
      this.usn = param1.usn;
      this.name = param1.name;
      this.marks = param1.marks ?? [];
    } else {
      // Handled positional parameters
      this.usn = param1;
      this.name = param2 ?? "Unknown";
      this.marks = param3 ?? [];
    }
  }

  public getSummary(): string {
    return `[${this.usn}] ${this.name} - Total Marks: ${this.marks.length}`;
  }
}

// ==========================================
// TESTING THE OVERLOADED CONSTRUCTOR
// ==========================================

// Test Case 1: Two arguments (USN & Name)
const student1 = new StudentProfile("1NT21CS001", "Aarav");
console.log(student1.getSummary()); 
// Output: [1NT21CS001] Aarav - Total Marks: 0

// Test Case 2: Three arguments (USN, Name & Marks)
const student2 = new StudentProfile("1NT21CS002", "Ananya", [85, 92, 78]);
console.log(student2.getSummary()); 
// Output: [1NT21CS002] Ananya - Total Marks: 3

// Test Case 3: Config Object
const student3 = new StudentProfile({
  usn: "1NT21CS003",
  name: "Rohan",
  marks: [90, 88]
});
console.log(student3.getSummary()); 
// Output: [1NT21CS003] Rohan - Total Marks: 2
```

---

## 2. Step-by-Step Method Overloading Implementation

Implementing method overloading in TypeScript follows a strict 3-step process:

1. **Declare Overload Signatures**: Define all valid parameter lists and return types.
2. **Provide Implementation Signature**: Define a broader single signature that encompasses all overloads.
3. **Implement Internal Logic**: Write the function body with conditional type checks (`typeof`, `Array.isArray`, etc.).

```
┌────────────────────────────────────────────────────────┐
│ 1. Overload Signature 1: search(id: number)           │
├────────────────────────────────────────────────────────┤
│ 2. Overload Signature 2: search(name: string)          │
├────────────────────────────────────────────────────────┤
│ 3. Implementation Signature: search(query: string|num) │
├────────────────────────────────────────────────────────┤
│ 4. Dispatch Logic: if (typeof query === 'number') ...  │
└────────────────────────────────────────────────────────┘
```

### Complete Implementation Example: Grade Search System

```typescript
class StudentDatabase {
  private records = [
    { id: 101, name: "Aarav", grade: "A" },
    { id: 102, name: "Priya", grade: "S" },
    { id: 103, name: "Rohan", grade: "B" }
  ];

  // STEP 1: Overload Signatures (Type Definitions)
  findStudent(id: number): string;
  findStudent(name: string): string;
  findStudent(id: number, returnGradeOnly: boolean): string;

  // STEP 2: Implementation Signature (Must cover all signatures above)
  findStudent(query: string | number, returnGradeOnly?: boolean): string {
    // STEP 3: Dispatch Logic (Runtime Type Narrowing)

    // Handling Number ID Query
    if (typeof query === "number") {
      const result = this.records.find((r) => r.id === query);
      if (!result) return "Student ID Not Found";
      return returnGradeOnly ? result.grade : `Found: ${result.name} (${result.grade})`;
    }

    // Handling String Name Query
    if (typeof query === "string") {
      const result = this.records.find(
        (r) => r.name.toLowerCase() === query.toLowerCase()
      );
      if (!result) return "Student Name Not Found";
      return `Found: ${result.name} [ID: ${result.id}]`;
    }

    return "Invalid Query";
  }
}

// Testing the Method Overloads
const db = new StudentDatabase();

console.log(db.findStudent(101));                  // Output: Found: Aarav (A)
console.log(db.findStudent(102, true));            // Output: S
console.log(db.findStudent("Priya"));              // Output: Found: Priya [ID: 102]
```

---

## 3. Writing Internal Overload Logic (Dispatching)

The internal logic inside the implementation block is called **Dispatch Logic**. Because JavaScript doesn't natively support function overloading, TypeScript compiles overloads into a single function. You must manually inspect the arguments using **Type Guards**.

### Common Type Guards for Overload Logic:

| Guard Technique | Used For | Example Code |
| :--- | :--- | :--- |
| `typeof param === "string"` | Primitives (`string`, `number`, `boolean`) | `if (typeof val === "number")` |
| `Array.isArray(param)` | Arrays vs single elements | `if (Array.isArray(val))` |
| `param !== undefined` | Optional argument presence | `if (param2 !== undefined)` |
| `param instanceof ClassName` | Objects instantiated from classes | `if (val instanceof Student)` |

### Example: Internal Dispatch for Attendance Marking

```typescript
class AttendanceManager {
  // Overload Signatures
  markAttendance(studentId: number): void;
  markAttendance(studentIds: number[]): void;
  markAttendance(studentId: number, status: "PRESENT" | "ABSENT"): void;

  // Single Implementation Signature
  markAttendance(
    target: number | number[],
    status: "PRESENT" | "ABSENT" = "PRESENT"
  ): void {
    // 1. Dispatch Logic using Array.isArray
    if (Array.isArray(target)) {
      console.log(`Marking ${target.length} students as ${status}.`);
      target.forEach((id) => console.log(` - Student ${id}: ${status}`));
      return;
    }

    // 2. Dispatch Logic using Single Number Target
    if (typeof target === "number") {
      console.log(`Student ${target} marked as ${status}.`);
      return;
    }
  }
}

// Testing Dispatch Logic
const manager = new AttendanceManager();

manager.markAttendance(101);                     // Student 101 marked as PRESENT.
manager.markAttendance([102, 103, 104]);         // Marking 3 students as PRESENT.
manager.markAttendance(105, "ABSENT");           // Student 105 marked as ABSENT.
```

---

## ⚠️ Exam Pitfalls & Viva Questions

1. **Q: Why can't we write separate function bodies for each overload in TypeScript?**
   * *A: TypeScript compiles to standard JavaScript, which does not support having multiple functions with the exact same name. Therefore, TypeScript allows multiple signatures, but requires a single function body that handles all types.*

2. **Q: Is the implementation signature visible to consumers calling the method?**
   * *A: No! Only the overload signatures are visible. Consumers cannot call the method using parameter combinations that match only the implementation signature.*

3. **Q: What happens if an explicit parameter check is missing in dispatch logic?**
   * *A: It can cause unpredictable runtime bugs or return `undefined`, because the TypeScript compiler relies on you, the developer, to correctly check argument types inside the single implementation block.*

---

## 🎯 Quick Self-Check Questions

1. **What keyword/check do you use to distinguish an Array overload parameter from a single primitive?**
   * *Answer:* `Array.isArray(param)`

2. **How many constructors with implementation bodies can a TypeScript class have?**
   * *Answer:* Exactly **one**.