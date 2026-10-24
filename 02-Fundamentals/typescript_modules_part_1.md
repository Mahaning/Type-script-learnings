# TypeScript Modules - Deep Dive (Part 1)

## Table of Contents

1. [Introduction to Modules](#1-introduction-to-modules)

   * [What is a Module in TypeScript?](#what-is-a-module-in-typescript)

   * [Modules vs. Scripts / Global Scope](#modules-vs-scripts--global-scope)

2. [Types of Module Systems](#2-types-of-module-systems)

   * [ES Modules (ESM)](#1-es-modules-esm)

   * [CommonJS (CJS)](#2-commonjs-cjs)

   * [Asynchronous Module Definition (AMD) & SystemJS](#3-asynchronous-module-definition-amd--systemjs)

   * [Universal Module Definition (UMD)](#4-universal-module-definition-umd)

3. [Module Syntax & Usage in TypeScript](#3-module-syntax--usage-in-typescript)

   * [Named Exports & Imports](#named-exports--imports)

   * [Default Exports & Imports](#default-exports--imports)

   * [Re-exporting & Barrel Export Patterns](#re-exporting--barrel-export-patterns)

   * [Type-Only Imports and Exports](#type-only-imports-and-exports)

4. [Internal Working & Module Resolution Strategies](#4-internal-working--module-resolution-strategies)

   * [How TypeScript Resolves Imports](#how-typescript-resolves-imports)

   * [Classic vs. Node Resolution Strategies](#classic-vs-node-resolution-strategies)

   * [Module Transpilation & Emission Mechanics](#module-transpilation--emission-mechanics)

5. [Summary Checklist](#5-summary-checklist)

---

## 1. Introduction to Modules

In modern software development, modularity is essential for organizing code, enforcing encapsulation, and managing dependencies across large codebases.

### What is a Module in TypeScript?

In TypeScript (as in modern JavaScript), **a module is any file that contains a top-level `import` or `export` statement**. 

* Every module has its own **local scope**. Variables, functions, interfaces, and classes declared within a module are **not** accessible in the global scope unless explicitly exported.
* Any file that **does not** contain top-level `import` or `export` keywords is executed in the **global scope**.

```
+--------------------------------------------------------------------------+
|                        GLOBAL SCOPE VS MODULE SCOPE                      |
+--------------------------------------------------------------------------+

   [ Global Script: utils.ts ]             [ ES Module: mathService.ts ]
   +---------------------------+           +-------------------------------+
   | const secret = "123";     |           | const secret = "123";         |
   | // Leaks to global scope! |           | // Isolated to this file      |
   +---------------------------+           | export function add() {}      |
                                           +-------------------------------+
```

---

## 2. Types of Module Systems

Over time, JavaScript has evolved several module formats to handle dependency loading across different runtimes (browsers, Node.js, legacy loaders). TypeScript supports all major target formats through configuration settings in `tsconfig.json`.

```
       +-----------------------------------------------------------+
       |                  JAVASCRIPT MODULE SYSTEMS                |
       +-----------------------------------------------------------+
       |                                                           |
       |  1. ES Modules (ESM)         2. CommonJS (CJS)            |
       |     `import / export`           `require() / module.exports`|
       |     (Modern Browsers, Node)     (Legacy Node.js)          |
       |                                                           |
       |  3. AMD / SystemJS           4. UMD                       |
       |     `define(['dep'], factory)`  Universal wrapper for CJS |
       |     (Browser async loading)     + AMD + Global window     |
       +-----------------------------------------------------------+
```

### 1. ES Modules (ESM)
ESM is the official ECMAScript standard syntax (`import` / `export`). It supports static analysis, asynchronous loading, and aggressive tree-shaking by modern bundlers (Vite, Webpack, Esbuild).

```typescript
// math.ts (ESM)
export const add = (a: number, b: number): number => a + b;

// main.ts (ESM)
import { add } from "./math.js";
console.log(add(2, 3));
```

### 2. CommonJS (CJS)
CommonJS was popularized by Node.js. It uses synchronous `require()` calls and `module.exports`.

```typescript
// math.ts (CJS)
function add(a: number, b: number): number { return a + b; }
module.exports = { add };

// main.ts (CJS)
const { add } = require("./math");
console.log(add(2, 3));
```

### 3. Asynchronous Module Definition (AMD) & SystemJS
AMD (e.g., RequireJS) was designed for browser environments to load dependencies asynchronously before executing the module body.

```javascript
// AMD Output
define(["require", "exports", "./math"], function (require, exports, math_1) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    console.log(math_1.add(2, 3));
});
```

### 4. Universal Module Definition (UMD)
UMD acts as a multi-target wrapper, detecting at runtime whether the environment supports CommonJS, AMD, or plain global browser script tags (`window`).

---

## 3. Module Syntax & Usage in TypeScript

### Named Exports & Imports
Named exports allow exporting multiple items from a single file by name.

```typescript
// services/user.ts
export interface User {
  id: string;
  name: string;
}

export function getUser(id: string): User {
  return { id, name: "Alice" };
}

export const USER_LIMIT = 100;
```

```typescript
// consumer.ts
import { User, getUser, USER_LIMIT as MAX_USERS } from "./services/user";
```

### Default Exports & Imports
A module can have at most **one** default export.

```typescript
// Logger.ts
export default class Logger {
  log(message: string): void {
    console.log(`[LOG]: ${message}`);
  }
}

// app.ts
import CustomLogger from "./Logger"; // Name can be customized during import
```

### Re-exporting & Barrel Export Patterns
A **Barrel File** (`index.ts`) aggregates exports from multiple sub-modules to simplify consumers' import statements.

```typescript
// barrel/index.ts
export * from "./userService";
export * from "./authService";
export { default as SystemLogger } from "./loggerService";
```

```typescript
// Single import statement cleanly accessing sub-modules
import { userService, authService, SystemLogger } from "./barrel";
```

### Type-Only Imports and Exports
TypeScript provides explicit type-import syntax (`import type`) to guarantee that type declarations are completely erased during compilation, preventing runtime circular dependency glitches or unused import side-effects.

```typescript
import type { UserProfile } from "./types";
import { fetchUserData } from "./api";

// Inline type-only modifier
import { type APIResponse, processResponse } from "./api";
```

---

## 4. Internal Working & Module Resolution Strategies

### How TypeScript Resolves Imports

When TypeScript encounters an import statement like `import { add } from "./math"`, it determines the corresponding file path in two steps:

1. **Non-Relative Imports** (e.g., `import _ from "lodash"`): Resolved by searching `node_modules` folders up the directory tree or looking up path aliases defined in `tsconfig.json`.
2. **Relative Imports** (e.g., `import { add } from "./math"`): Resolved relative to the importing file location.

### Classic vs. Node Resolution Strategies

Configured via `"moduleResolution"` in `tsconfig.json`:

* **`node` / `node10` (Legacy Node.js):** Replicates standard CommonJS resolution. Lookups check `math.ts`, `math.tsx`, `math.d.ts`, or `math/index.ts`.
* **`node16` / `nodenext`:** Supports modern ES Module resolution rules in Node.js, enforcing file extension specifications (`.js`, `.mjs`) and respecting `package.json` `"type": "module"` flags.
* **`bundler` (Modern Front-end):** Designed for modern toolchains (Vite, Webpack, Esbuild) that handle module resolution natively.

```
                  [ TypeScript Import Resolution Flow ]
                                    |
                        Is import path relative?
                                 /     \
                          YES   /       \   NO
                               v         v
             Resolve relative path     Search `node_modules` /
               from current file       Check `tsconfig.json` paths
                               |         |
                               v         v
                      Locate matching .ts / .tsx / .d.ts file
```

### Module Transpilation & Emission Mechanics

TypeScript's output format depends directly on the `"module"` option in `tsconfig.json` (e.g., `"module": "CommonJS"` vs. `"module": "ESNext"`).

#### Source Code (`auth.ts`)
```typescript
export class AuthManager {
  public isAuthenticated(): boolean {
    return true;
  }
}
```

#### Transpiled Output with `"module": "CommonJS"`
```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.AuthManager = void 0;

var AuthManager = /** @class */ (function () {
    function AuthManager() {
    }
    AuthManager.prototype.isAuthenticated = function () {
        return true;
    };
    return AuthManager;
})();
exports.AuthManager = AuthManager;
```

#### Transpiled Output with `"module": "ESNext"`
```javascript
export class AuthManager {
    isAuthenticated() {
        return true;
    }
}
```

---

## 5. Summary Checklist

* \[ \] **Module vs Script:** Any file with `import`/`export` is a module with local scope; otherwise, it executes in global scope.
* \[ \] **Module Format Target:** Use `"module": "NodeNext"` for modern Node environments and `"module": "ESNext"` for modern bundlers.
* \[ \] **Type-Only Imports:** Use `import type` to prevent unused imports from lingering in runtime bundles.
* \[ \] **Barrels:** Use `index.ts` files to aggregate re-exports, but avoid deep barrel chains that slow compiler lookup times.