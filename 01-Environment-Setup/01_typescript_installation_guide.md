# Day 1: Full TypeScript Environment Setup Guide

This guide covers everything required to set up TypeScript from scratch, including prerequisites, global PATH configuration, project-level initialization, and VS Code integration.

---

## 1. Prerequisites Installation

Before using TypeScript, you need **Node.js** and **npm** (Node Package Manager) installed on your machine.

### Download & Install
1. Download the **LTS (Long Term Support)** version of Node.js from [nodejs.org](https://nodejs.org/).
2. Run the installer and keep the default options (npm comes bundled automatically).

### Verification
Open your terminal (Command Prompt, PowerShell, or macOS/Linux Terminal) and execute:

```bash
node -v
npm -v
```

*Expected Output:* Version numbers for both Node.js and npm (e.g., `v20.x.x` and `10.x.x`).

---

## 2. Global Setup vs. Project Setup

| Approach | Description | Use Case |
| :--- | :--- | :--- |
| **Global Setup** | Installs `tsc` globally in system PATH | Quick CLI experiments or system-wide access |
| **Project Setup** | Installs `typescript` inside project `node_modules` | Production apps, team collaboration, version locking |

---

## 3. Global PATH Setup

Installing globally exposes the TypeScript compiler (`tsc`) as a command-line tool across your operating system.

### Installation
Run the following command in your terminal:

```bash
npm install -g typescript
```

### Verification
Check if the global PATH accurately locates `tsc`:

```bash
tsc -v
```

### Common PATH & Execution Issues
* **Windows (PowerShell Script Execution Error):**  
  If you get an error stating scripts are disabled on your system, open PowerShell as **Administrator** and run:
  ```powershell
  Set-ExecutionPolicy RemoteSigned
  ```
* **macOS / Linux (`EACCES` or Permission Denied):**  
  Avoid using `sudo npm install -g`. Instead, update your npm default directory to a user-owned path:
  ```bash
  mkdir ~/.npm-global
  npm config set prefix '~/.npm-global'
  export PATH=~/.npm-global/bin:$PATH
  ```
  *(Add the `export` line to your `~/.zshrc` or `~/.bashrc` file to make it permanent).*

---

## 4. Local Project Setup (Recommended Standard)

For standard development, always prefer project-local setups to ensure team members use identical compiler versions.

### Step-by-Step Initialization

1. **Create and open project folder:**
   ```bash
   mkdir ts-day1
   cd ts-day1
   ```

2. **Initialize Node package manifest:**
   ```bash
   npm init -y
   ```

3. **Install TypeScript and execution tools as dev dependencies:**
   ```bash
   npm install --save-dev typescript tsx @types/node
   ```
   * `typescript`: The compiler engine.
   * `tsx`: Node.js wrapper to execute TypeScript files directly in memory without manually compiling first.
   * `@types/node`: TypeScript type definitions for standard Node.js modules.

4. **Initialize TypeScript configuration:**
   ```bash
   npx tsc --init
   ```
   This generates a default `tsconfig.json` in your project root.

---

## 5. Visual Studio Code Setup

### Recommended Extensions
1. **Pretty TypeScript Errors** (`yoavbls.pretty-ts-errors`) – Formats nested TypeScript error messages cleanly.
2. **ESLint** (`dbaeumer.vscode-eslint`) – Handles code quality and syntax rules.

### Configuring `tsconfig.json`
Replace or update your generated `tsconfig.json` with these foundational settings:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"]
}
```

### VS Code Build Task (Auto-Compile on Save)
1. In VS Code, press `Ctrl+Shift+B` (Windows/Linux) or `Cmd+Shift+B` (macOS).
2. Select **`tsc: watch - tsconfig.json`**.
3. VS Code will now watch your `./src` folder and automatically compile `.ts` files to `./dist` whenever you save.

---

## 6. Project Directory Structure & First Code Execution

### Folder Layout
```text
ts-day1/
├── dist/             # Compiled JavaScript output
├── node_modules/
├── src/
│   └── index.ts      # Source TypeScript code
├── package.json
└── tsconfig.json
```

### Create Source File
Create `src/index.ts`:

```typescript
interface User {
  id: number;
  name: string;
  role: 'Admin' | 'User';
}

const getWelcomeMessage = (user: User): string => {
  return `Welcome back, ${user.name} (${user.role})!`;
};

const currentUser: User = {
  id: 1,
  name: "Alex",
  role: "Admin"
};

console.log(getWelcomeMessage(currentUser));
```

### Configure Scripts in `package.json`
Update the `"scripts"` block in `package.json`:

```json
"scripts": {
  "dev": "tsx --watch src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js"
}
```

### Execution Commands
* **Development (Hot Reloading):**
  ```bash
  npm run dev
  ```
* **Build (Generate JS Files):**
  ```bash
  npm run build
  ```
* **Production Execution:**
  ```bash
  npm start
  ```