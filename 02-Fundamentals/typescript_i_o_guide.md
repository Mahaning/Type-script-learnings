# 📥 The Complete TypeScript I/O Operations Guide

Input and Output (I/O) operations in TypeScript allow your application to interact with external systems—such as file systems, terminal interfaces, databases, and network endpoints.

Since TypeScript compiles down to JavaScript, I/O capabilities depend on the host runtime: **Node.js, Bun, or Deno** for backend/CLI applications, and **Web APIs** for browser environments.

---

## 🏗️ 1. Core Architecture of TypeScript I/O

TypeScript uses non-blocking, event-driven I/O models powered by the underlying JavaScript runtime.

```
                   ┌──────────────────────────┐
                   │  TypeScript Code         │
                   └────────────┬─────────────┘
                                │ Compile
                                ▼
                   ┌──────────────────────────┐
                   │  JavaScript Execution    │
                   └────────────┬─────────────┘
                                │
        ┌───────────────────────┴───────────────────────┐
        ▼                                               ▼
┌───────────────────────────────┐               ┌───────────────────────────────┐
│       Node.js Runtime         │               │        Browser Runtime        │
├───────────────────────────────┤               ├───────────────────────────────┤
│ • File System (`fs/promises`) │               │ • Web Fetch API               │
│ • Terminal Streams (`stdin`)  │               │ • DOM Events                  │
│ • Network Sockets (`net`, `http`)             │ • WebSockets & LocalStorage   │
└───────────────────────────────┘               └───────────────────────────────┘
```

---

## 📁 2. File System Operations (Node.js / Bun)

### A. Asynchronous File I/O (`fs/promises`)

Always prefer asynchronous file operations in production to prevent blocking the event loop.

```typescript
import { readFile, writeFile, appendFile, unlink } from 'fs/promises';
import { join } from 'path';

interface LogEntry {
  timestamp: string;
  message: string;
  level: 'INFO' | 'WARN' | 'ERROR';
}

async function handleFileOperations(): Promise<void> {
  const filePath = join(__dirname, 'app.log');

  const logData: LogEntry = {
    timestamp: new Date().toISOString(),
    message: 'Application started successfully.',
    level: 'INFO',
  };

  try {
    // 1. Write to file
    await writeFile(filePath, JSON.stringify(logData, null, 2), 'utf-8');
    console.log('✅ File written successfully');

    // 2. Read from file
    const fileContent = await readFile(filePath, 'utf-8');
    const parsedLog: LogEntry = JSON.parse(fileContent);
    console.log('📖 Read Log Message:', parsedLog.message);

    // 3. Append to file
    await appendFile(filePath, '\n// End of Log', 'utf-8');

  } catch (error) {
    console.error('❌ File System Error:', error);
  }
}

handleFileOperations();
```

---

### B. High-Performance Streaming for Large Files

For processing large files (hundreds of MBs or GBs) without consuming excessive memory, use Streams.

```typescript
import { createReadStream, createWriteStream } from 'fs';
import { pipeline } from 'stream/promises';
import { Transform } from 'stream';

// Transform stream to uppercase all text chunk-by-chunk
const uppercaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  },
});

async function processLargeFile(source: string, destination: string): Promise<void> {
  try {
    await pipeline(
      createReadStream(source),
      uppercaseTransform,
      createWriteStream(destination)
    );
    console.log('🚀 Large file processed and streamed successfully.');
  } catch (err) {
    console.error(' Stream processing failed:', err);
  }
}
```

---

## 💻 3. Command Line Interface (CLI) Input & Output

### Interactive Terminal Prompts

Read input directly from the user's terminal session using `readline/promises`.

```typescript
import * as readline from 'readline/promises';
import { stdin as input, stdout as output } from 'process';

interface UserProfile {
  username: string;
  age: number;
}

async function collectUserInput(): Promise<UserProfile> {
  const rl = readline.createInterface({ input, output });

  try {
    const username = await rl.question('Enter your username: ');
    const ageString = await rl.question('Enter your age: ');

    const profile: UserProfile = {
      username: username.trim(),
      age: parseInt(ageString, 10),
    };

    console.log(`\n✨ Profile Created: Hello ${profile.username}, age ${profile.age}`);
    return profile;
  } finally {
    rl.close(); // Ensure interface is closed
  }
}

collectUserInput();
```

---

## 🌐 4. Network I/O (Fetch & HTTP Requests)

Network operations are asynchronous by nature. TypeScript interfaces ensure strong type safety when consuming remote APIs.

```typescript
interface Post {
  userId: number;
  id: number;
  title: string;
  body: string;
}

// Strongly typed GET request
async function fetchPost(postId: number): Promise<Post> {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`);

  if (!response.ok) {
    throw new Error(`Failed to fetch post: ${response.statusText}`);
  }

  const data: Post = await response.json();
  return data;
}

// Strongly typed POST request
async function createPost(newPost: Omit<Post, 'id'>): Promise<Post> {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newPost),
  });

  return await response.json();
}
```

---

## 📊 5. Summary & Best Practices Matrix

| Category | Recommended Tool / API | Memory Efficiency | Primary Use Case |
| :--- | :--- | :--- | :--- |
| **Small File Reading** | `fs/promises.readFile()` | Low (Loads full file to RAM) | Config files, JSON storage |
| **Large File Reading** | `fs.createReadStream()` | High (Processes chunks in memory) | Video files, CSV processing, big logs |
| **User Inputs** | `readline/promises` | High | Interactive CLI applications |
| **HTTP Requests** | `fetch()` API | High | Fetching/Sending API payloads |
| **Standard Output** | `console.log()` / `process.stdout` | High | Debugging & CLI output messages |

---

## ⚠️ Key Engineering Rules

1. **Never use `readFileSync` in servers:** Synchronous file calls pause execution for all connected users while disk operations complete.
2. **Always Type API Responses:** Avoid using `any` when parsing JSON from I/O calls. Use strong `interface` definitions or libraries like `zod` for validation.
3. **Handle Errors Safely:** Always wrap asynchronous I/O calls in `try...catch` blocks to prevent unhandled promise rejections.