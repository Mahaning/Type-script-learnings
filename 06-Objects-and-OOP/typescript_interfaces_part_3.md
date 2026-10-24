# TypeScript Interfaces - Deep Dive (Part 3)

## Table of Contents

1. [Interface Inheritance (`extends`)](#1-interface-inheritance-extends)

   * [Single & Multiple Interface Inheritance](#11-single--multiple-interface-inheritance)

   * [Overriding & Property Narrowing in Derived Interfaces](#12-overriding--property-narrowing-in-derived-interfaces)

2. [Combining Interfaces with Type Aliases](#2-combining-interfaces-with-type-aliases)

   * [Interfaces Extending Types](#21-interfaces-extending-types)

   * [Types Extending Interfaces (Intersections `&`)](#22-types-extending-interfaces-intersections-)

3. [Declaration Merging (Interface Re-Opening)](#3-declaration-merging-interface-re-opening)

   * [How Declaration Merging Works](#31-how-declaration-merging-works)

   * [Module Augmentation Use Cases](#32-module-augmentation-use-cases)

4. [Interfaces vs. Type Aliases: Exhaustive Comparison](#4-interfaces-vs-type-aliases-exhaustive-comparison)

5. [Summary Checklist](#5-summary-checklist)

## 1. Interface Inheritance (`extends`)

Interfaces can inherit structural properties from other interfaces using the `extends` keyword. This allows developers to build modular, non-duplicative type contracts.

```
       +-----------------------------------------------------------+
       |               INTERFACE INHERITANCE MODEL                 |
       +-----------------------------------------------------------+
       |                                                           |
       |  1. Single Extension        2. Multiple Extension         |
       |    interface Base                interface A  interface B |
       |         ^                             ^            ^      |
       |         |                             \            /      |
       |    interface Derived                interface Child       |
       |                                                           |
       +-----------------------------------------------------------+

```

### 1.1 Single & Multiple Interface Inheritance

Unlike classes (which only support single class inheritance), an interface can extend **multiple interfaces simultaneously**, creating a unified structural contract.

```
// Base Interfaces
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

// Derived Interface extending multiple base interfaces
interface DatabaseRecord extends Identifiable, Timestamped {
  version: number;
  isArchived: boolean;
}

const userRecord: DatabaseRecord = {
  id: "rec_9921",
  createdAt: new Date(),
  updatedAt: new Date(),
  version: 1,
  isArchived: false,
};

```

### 1.2 Overriding & Property Narrowing in Derived Interfaces

A derived interface can re-declare a property from a base interface **only if the new type is a subtype/narrower type** of the original property type.

```
interface BaseResponse {
  code: number | string;
  payload: unknown;
}

// ✅ VALID: Narrowing `code` to `number` and `payload` to a specific object shape
interface SuccessResponse extends BaseResponse {
  code: number; // Subtype of number | string
  payload: {
    data: string[];
  };
}

// ❌ INVALID: Property type expansion/incompatible override
/*
interface InvalidResponse extends BaseResponse {
  code: boolean; // Error: Type 'boolean' is not assignable to type 'number | string'.
}
*/

```

## 2. Combining Interfaces with Type Aliases

TypeScript allows interfaces and type aliases to interact flexibly. An interface can extend a type alias, and a type alias can intersect with an interface.

### 2.1 Interfaces Extending Types

An interface can extend a type alias if the type alias evaluates to an **object shape or intersection of object shapes**.

```
// Type Alias defining a point shape
type Point2D = {
  x: number;
  y: number;
};

// Interface extending the Type Alias
interface Point3D extends Point2D {
  z: number;
}

const point: Point3D = { x: 10, y: 20, z: 30 };

```

> ⚠️ **Limitation:** An interface **cannot** extend a type alias that represents a Union Type (`type A = B | C`).

```
type Animal = { name: string } | { age: number };

// ❌ Compiler Error: An interface can only extend an object type or intersection of object types with statically known members.
// interface Dog extends Animal { breed: string; }

```

### 2.2 Types Extending Interfaces (Intersections `&`)

A type alias uses the intersection operator (`&`) to combine interfaces or other type aliases.

```
interface UserBase {
  id: string;
  name: string;
}

interface UserPermissions {
  roles: string[];
}

// Type alias extending multiple interfaces via intersection (&)
type AdminUser = UserBase & UserPermissions & {
  superAdminKey: string;
};

const admin: AdminUser = {
  id: "usr_001",
  name: "Alice",
  roles: ["admin", "editor"],
  superAdminKey: "key_xyz123",
};

```

## 3. Declaration Merging (Interface Re-Opening)

One of the unique capabilities of TypeScript interfaces is **Declaration Merging**. Multiple interface declarations with the same name in the same scope merge into a single combined type definition.

### 3.1 How Declaration Merging Works

```
// Initial Declaration
interface WindowSettings {
  title: string;
}

// Re-opening the interface in another file or scope
interface WindowSettings {
  width: number;
  height: number;
}

// Resulting merged shape requires ALL properties
const config: WindowSettings = {
  title: "Main Dashboard",
  width: 1920,
  height: 1080,
};

```

### 3.2 Module Augmentation Use Cases

Declaration merging is widely used in enterprise applications and library definitions to augment third-party packages or express global scope additions (e.g., adding properties to Express `Request` or `window`).

```
// Augmenting Express Request object in Node.js
declare global {
  namespace Express {
    interface Request {
      currentUser?: {
        id: string;
        role: string;
      };
    }
  }
}

// In API Route Handler:
import { Request, Response } from "express";

function routeHandler(req: Request, res: Response) {
  // `currentUser` is recognized on Express Request thanks to declaration merging!
  console.log(req.currentUser?.id);
}

```

## 4. Interfaces vs. Type Aliases: Exhaustive Comparison

Choosing between `interface` and `type` is a key decision in TypeScript codebases.

| Feature | `interface` | `type` Alias | 
| ----- | ----- | ----- | 
| **Primary Declaration** | Object shapes, contracts, function signatures | Any valid TypeScript type (Primitives, Unions, Tuples, Objects) | 
| **Declaration Merging** | ✅ Supported (Re-openable) | ❌ Not supported (Throws Duplicate Identifier Error) | 
| **Extensibility Syntax** | `extends` keyword | Intersection operator (`&`) | 
| **Union Types (`\|`)** | ❌ Cannot directly define unions (`interface X = A \| B`) | ✅ Supported (`type X = A \| B`) | 
| **Tuple Types** | ❌ Cannot directly express tuple types | ✅ Supported (`type Point = [number, number]`) | 
| **Primitives (`string`, `number`)** | ❌ Cannot alias primitive values | ✅ Supported (`type UserId = string`) | 
| **Performance in Compiler** | ⚡ Slightly faster type checking (cached by shape name) | ⚡ Resolved recursively (can slow compiler in deep intersections) | 

### Recommendations & Guidelines

* **Use `interface` for:**
  * Defining public APIs, SDKs, or library structures (enables users to extend contracts via declaration merging).
  * Class contracts (`implements MyInterface`).
  * Standard object-oriented domain models.

* **Use `type` for:**
  * Union or Intersection types (`type Status = "pending" | "fulfilled" | "rejected"`).
  * Primitives, Tuples, and mapped/conditional type utilities.
  * Complex functional programming types.

## 5. Summary Checklist

* \[ \] **Multiple Extensions:** Interfaces can extend multiple interfaces simultaneously (`extends A, B`).

* \[ \] **Narrowing Rules:** Overridden properties in child interfaces must be compatible subtypes of parent properties.

* \[ \] **Declaration Merging:** Multiple `interface` declarations with the same name automatically combine.

* \[ \] **Type vs Interface:** Prefer `interface` for object contracts and class implementations; use `type` for unions, primitives, and complex utility types.