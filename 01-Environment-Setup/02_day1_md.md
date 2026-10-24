# Day 1: Playwright Environment Setup Guide (Windows)

Welcome to Day 1 of your Playwright automation journey. This document covers the full step-by-step setup process for setting up Playwright on a Windows machine, including prerequisites, environment path configuration, VS Code setup, and running your first test.

---

## 1. Prerequisites Installation

Before installing Playwright, your Windows machine needs Node.js and Git installed.

### Step 1: Install Node.js
1. Go to the official website: [Node.js Official Downloads](https://nodejs.org/)
2. Download the **LTS (Long Term Support)** installer for Windows (`.msi`).
3. Run the setup wizard.
4. **Crucial:** During installation, ensure the checkbox **"Add to PATH"** is selected.
5. Finish the installation wizard.

### Step 2: Install Visual Studio Code
1. Download VS Code: [VS Code Official Downloads](https://code.visualstudio.com/)
2. Run the installer and complete the setup.

---

## 2. Environment PATH Configuration & Verification

If Node.js and npm commands are not recognized after installation, you need to manually add them to your Windows System Environment Variables.

### Setting Up Environment Variables:
1. Press `Win + R` to open the Run dialog.
2. Type `sysdm.cpl` and hit **Enter** (this opens System Properties).
3. Navigate to the **Advanced** tab and click on **Environment Variables**.
4. In the **System variables** section, find the `Path` variable, select it, and click **Edit**.
5. Click **New** and add the path to your Node.js installation (default directory):
   ```text
   C:\Program Files\nodejs\
   ```
6. Click **OK** on all windows to save the changes.

### Verifying Installation:
Open a **new** PowerShell or Command Prompt window and execute the following commands to check versions:

```powershell
node -v
npm -v
```

> **Note:** If valid version numbers are returned (e.g., `v20.x.x` and `10.x.x`), your environment variables are configured correctly.

---

## 3. Visual Studio Code Configuration

Enhance your development and debugging workflow by configuring VS Code for Playwright.

1. Open **VS Code**.
2. Go to the Extensions sidebar (`Ctrl + Shift + X`).
3. Search for **Playwright Test for VSCode** (Published by **Microsoft**).
4. Click **Install**.

---

## 4. Initializing a New Playwright Project

Follow these steps to create your first Playwright testing project.

1. Open PowerShell or Command Prompt.
2. Create a folder for your automation suite and enter it:
   ```powershell
   mkdir playwright-automation
   cd playwright-automation
   ```
3. Run the Playwright setup command:
   ```powershell
   npm init playwright@latest
   ```
4. Complete the interactive setup prompts:
   - **Do you want to use TypeScript or JavaScript?** → `TypeScript` (Recommended)
   - **Where to put your end-to-end tests?** → `tests`
   - **Add a GitHub Actions workflow?** → `false`
   - **Install Playwright browsers?** → `true`

---

## 5. Running Your First Test

Playwright creates a default test file located at `tests/example.spec.ts`.

### Running Tests via Terminal:
- **Headless Mode** (Runs in the background):
  ```powershell
  npx playwright test
  ```
- **Headed Mode** (Opens browser windows while running):
  ```powershell
  npx playwright test --headed
  ```
- **View Interactive Test Report**:
  ```powershell
  npx playwright show-report
  ```

### Running Tests via VS Code Interface:
1. Open the project folder in VS Code (`File > Open Folder...`).
2. Open the **Testing** tab on the left panel (beaker icon).
3. Expand `tests/example.spec.ts`.
4. Click the green **Play** button next to any test to run it directly inside VS Code.

---

## 6. Common Windows Issues & Fixes

### Issue 1: PowerShell Script Execution Policy Error
If PowerShell gives an error stating `running scripts is disabled on this system`, run PowerShell as an **Administrator** and execute:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Issue 2: Missing Browser Binaries or Dependencies
If browsers fail to launch, reinstall required browser binaries by running:

```powershell
npx playwright install
```