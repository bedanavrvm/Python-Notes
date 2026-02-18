# Chapter 2 — Environment & Tooling

(Learning JavaScript and TypeScript Together)

This chapter answers one practical question: **How does my code actually run, and what tools sit between me and the runtime?**

- **JavaScript** runs directly in a runtime.
- **TypeScript** requires a build step before reaching that runtime.

## 1. Where JavaScript Runs

JavaScript always runs inside a runtime environment. While runtimes provide the "body" for the code (as seen in Chapter 1), here we focus on the practical API differences.

### JavaScript in the Browser
The browser runtime provides APIs for UI and network interaction.

```js
setTimeout(() => {
  console.log("Timer done");
}, 1000);
```
- **setTimeout**: Provided by the browser runtime (Web API).
- **Function**: The arrow function `() => { ... }` runs after 1000ms.
- **Constraint**: This works in browsers because they provide the timer system.

### JavaScript in Node.js
Node.js provides APIs for server-side tasks like file handling.

```js
const fs = require("fs");

fs.readFile("data.txt", "utf8", (err, data) => {
  console.log(data);
});
```
- **require("fs")**: Imports Node’s file system module.
- **readFile**: Reads a file asynchronously.
- **Callback**: `(err, data) => {}` handles the result; `err` for errors, `data` for content.
- **Constraint**: This will **NOT** work in the browser.

## 2. How TypeScript Fits Into This

JavaScript runs directly. TypeScript adds a mandatory build step: **Type-check → Compile → Execute**.

### Minimal Setup: Running a File



{% tabs %}
{% tab title="JavaScript" %}
- **File**: `index.js`
  ```js
  console.log("Hello JS");
  ```
- **Execution**: `node index.js`
- **Fact**: Node executes the file directly. No build step required.
{% endtab %}

{% tab title="TypeScript" %}
- **File**: `src/index.ts`
  ```ts
  console.log("Hello TS");
  ```
- **Execution**:
  1. **Compile**: `npx tsc` (reads `tsconfig.json`, outputs JS to `dist/`).
  2. **Run**: `node dist/index.js`
- **Fact**: The runtime cannot execute `.ts`. It only runs the emitted JavaScript.
{% endtab %}
{% endtabs %}

## 3. Node Version Management

"It works on my machine" often means "your version of Node is different than mine." Node versions update frequently, and older code might break on newer versions (or vice versa).

### Managing Versions with Tools
Developers use **Version Managers** to quickly switch between versions:
- **nvm (Node Version Manager)**: The classic choice for macOS/Linux.
- **nvm-windows**: The standard for Windows users.
- **fnm / volta**: Modern, fast alternatives written in newer languages like Rust.

### Pining the Version
Projects use configuration files to tell these tools exactly which version to use:
- **`.nvmrc`**: A simple file containing just the version (e.g., `18.17.0`).
- **`.node-version`**: A more modern, tool-agnostic version of the same idea.

When you enter a project folder, your version manager sees these files and says: *"Hey, you should be using Node 18 here,"* and switches for you.

## 4. The node_modules Directory

When you install a package, it gets downloaded into a folder called `node_modules`.

- **Where your code lives**: `src/`
- **Where other people's code lives**: `node_modules/`

### Important Rules
1. **Never Edit node_modules**: If you change code inside this folder, your changes will be deleted the next time you (or a teammate) run `npm install`.
2. **Never Commit to Git**: This folder can contain thousands of files and take up GBs of space. We use a `.gitignore` file to tell Git to ignore it. 
3. **The "Black Hole"**: Because it contains every dependency of your dependencies (and their dependencies too!), it can grow massive very quickly.

### How to Restore
If you delete your `node_modules`, don't panic! Just run `npm install`. The package manager will look at your **lockfile**, download everything again, and rebuild the folder perfectly.

## 5. Package Managers and Lockfiles

Every modern project revolves around `package.json` and a **lockfile**.

- **package.json**: Defines project name, dependencies, and `scripts`. This is the file you **manually edit** to add or update libraries.
- **Lockfiles**: These are **automatically generated** by your package manager and should **never be edited manually**.
  - **npm** generates `package-lock.json`
  - **Yarn** generates `yarn.lock`
  - **pnpm** generates `pnpm-lock.yaml`
  
  **Why they matter**: While `package.json` uses ranges (like `^1.2.3`), the lockfile "freezes" the exact version used at the moment of installation. This ensures that every developer on the team has the exact same code in their `node_modules`.

### Identifying Your Manager
You can tell which package manager a project uses just by looking at the lockfile in the root directory. If you see a `yarn.lock`, use `yarn install`. If you see `package-lock.json`, use `npm install`. Mixing them can lead to broken dependencies!

### Dependency Versions (Semver)
Versions follow `MAJOR.MINOR.PATCH` (e.g., `1.2.3`).
- **^1.2.3**: (~1.x.x) Allows Minor and Patch updates.
- **~1.2.3**: (~1.2.x) Allows only Patch updates.
- **1.2.3**: Exact version required.

## 6. Recommended Project Structure

A common layout separates source from output:

```text
my_project/
  package.json
  src/
    index.ts   <-- Your source code
  dist/
    index.js   <-- Compiled output (for TS)
```

### Scripts: JS vs TS Comparisons



{% tabs %}
{% tab title="JavaScript Project" %}
```json
{
  "name": "my-js-project",
  "type": "module",
  "scripts": {
    "start": "node src/index.js"
  }
}
```
- No build step; `start` runs the source directly.
{% endtab %}

{% tab title="TypeScript Project" %}
```json
{
  "name": "my-ts-project",
  "type": "module",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```
- `build` runs the compiler; `start` runs the emitted JS.
{% endtab %}
{% endtabs %}

## 7. Development Tools (TypeScript)

Rebuilding manually is slow. Developers use tools like `tsx` for faster feedback:
- **`tsx src/index.ts`**: Runs TS directly in development (internal compilation happens automatically).
- **`tsc --noEmit`**: Type-check only. Useful for CI/CD pipelines where you don't need JS output yet.

## 8. tsconfig.json (The Brain of TS)

This file tells the TypeScript compiler how to behave. Without it, the compiler doesn't know where your files are or what kind of JavaScript you want to create.

### Code Example: tsconfig.json

```json
{
  "compilerOptions": {
    /* What kind of JavaScript should we create? */
    "target": "ES2020",         // Output modern JS syntax
    "module": "NodeNext",       // Use modern Node.js module system
    
    /* Where go the files? */
    "outDir": "dist",           // Put compiled JS files here
    "rootDir": "src",           // Look for TS files here
    
    /* How strict should we be? */
    "strict": true,             // Enable ALL the safety checks
    "noImplicitAny": true,      // Error if a variable's type is "mystery" (any)
    
    /* Extra Housekeeping */
    "skipLibCheck": true,      // Don't check library types (faster builds)
    "sourceMap": true          // Create maps for easy debugging
  },
  "include": ["src/**/*"]       // Only compile things inside the src folder
}
```

### Key Settings Explained
- **target**: If your code needs to run on very old browsers, you might set this to `ES5`. If it's for a modern server, `ES2020` is great.
- **lib**: Tells TS which "Global" objects exist. For a browser project, you'd add `"DOM"`.
- **strict**: This is the "Professional Switch." Turning this on makes the compiler harder to satisfy, but catches significantly more bugs.

## 9. Development Guards

### Linting vs Type-Checking
- **TypeScript** checks if your **types** are correct (e.g., "is this a number?").
- **Linting (ESLint)** checks for **code quality** (e.g., "did you forget an `await`?", "is this variable unused?").
- **Formatting (Prettier)** ensures everyone's code looks identical.

### Testing
Types do not guarantee runtime correctness. Logic bugs (e.g., `1 + 1 = 3`) will pass type-checking but fail tests. **Type safety ≠ logic correctness.**

### Source Maps
When debugging, source maps allow you to see your original `.ts` or source code in the debugger/stack trace, even though the runtime is executing compiled `.js`.

---

## Important things to know

### Keywords & Extensions

- **Semver**: Semantic Versioning (Major.Minor.Patch).
- **Lockfile**: A file that "freezes" exact dependency versions to ensure consistency for all developers.
- **Sandbox**: The secure environment in a browser that prevents JS from touching your files.
- **Callback**: A function passed to another function to be executed later (found in Node APIs).
- **ASI**: Automatic Semicolon Insertion; the JS engine's attempt to guess where semicolons belong.
- **Development Dependency**: A tool (like `typescript` or `eslint`) needed to build/check code, but not needed to run the final app.

### Concepts for Later

- **CI (Continuous Integration)**: Automated pipelines that run your `typecheck`, `lint`, and `test` scripts whenever you change code.
- **Bundlers**: Tools (like Vite or Webpack) that combine many small files into one large file for the browser.
- **Transpilation**: The specific word for converting one high-level language (TS) into another (JS).

## Tasks



{% tabs %}
{% tab title="General" %}
1. **Check Node Version**: Run `node -v` in your terminal.
2. **Examine package.json**: Open a project and identify the `scripts` and `dependencies`.
3. **Semver Check**: Look at your version ranges. Are you using `^` or `~`?
{% endtab %}

{% tab title="TypeScript" %}
1. **Clean Start**: Delete your `dist/` folder and run `npx tsc` to see it regenerate.
2. **Typecheck vs Build**: Run `npx tsc --noEmit` and confirm no JS files were created.
3. **tsconfig Toggle**: Set `"strict": false` in a `tsconfig.json` and see if any previously flagged errors disappear (then turn it back on!).
{% endtab %}
{% endtabs %}
