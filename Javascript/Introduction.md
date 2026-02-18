# Introduction (JavaScript & TypeScript)

JavaScript (JS) is the programming language of the web.

- **JavaScript** answers: *what does the program do when it runs?*
- **TypeScript** answers: *can we catch some mistakes before it runs?*

Think of them like this:

{% tabs %}

{% tab title="JavaScript" %}

- Runs directly in the runtime (browser/Node).
- No build step required.
- Errors show up at runtime.

{% endtab %}

{% tab title="TypeScript" %}

- You still run JavaScript in the runtime.
- TypeScript adds compile-time checks and then emits JavaScript.
- Many mistakes can be caught before running.

{% endtab %}

{% endtabs %}

This chapter gives you a practical starting point:

- **What JavaScript is**
- **Where it runs**
- **How to run your first code**
- **Where TypeScript fits (optional)**

## What is JavaScript?

JavaScript is a general-purpose language that can run in multiple environments.

## What is JavaScript?

When you write JavaScript, you’re not just writing a language — you’re writing code that runs inside an environment. That environment is called a **runtime**, and it shapes what your JavaScript can and cannot do.

To understand this clearly, think of JavaScript as a **brain**. The brain can think (logic, variables, loops, functions), but it needs a body and senses to interact with the world. The runtime is that body.

## 1. The JavaScript Engine: The Brain

At the core of every runtime is a **JavaScript engine**. This engine reads your code, understands it, and executes it.

- **Google Chrome** uses the **V8** engine.
- **Firefox** uses **SpiderMonkey**.
- **Node.js** also uses **V8**.

The engine only understands the JavaScript language itself:
- Variables
- Functions
- Objects
- Arrays
- Promises
- Basic language syntax

```js
let x = 5;
let y = 10;
console.log(x + y);
```

The engine understands how to add numbers and run functions. But here’s something important: **The engine does NOT know what a webpage is, what `document` is, or what `fetch` is.** Those things come from the runtime environment.

## 2. The Runtime: The Environment Around the Engine

A runtime wraps around the JavaScript engine and provides extra capabilities. Different runtimes give you different “powers”. Two major runtimes are the **Browser** and **Node.js**.

### JavaScript in the Browser

When JavaScript runs in a browser like Chrome or Firefox, it gets access to **Browser APIs**. These are not part of JavaScript the language; they are provided by the browser runtime.

#### The DOM APIs (`document`, `window`)
When you open a webpage, the browser loads HTML and converts it into a structure called the **DOM (Document Object Model)** — a tree representation of your page. The browser gives JavaScript access to this via objects like `document`.

```js
document.getElementById("title").textContent = "Hello!";
```

- **How it works**: The JS engine executes the code, but `document` is provided by the browser runtime.
- **Node.js Check**: If you run this in Node.js, you get `ReferenceError: document is not defined` because Node.js does not provide DOM APIs.

#### Web APIs (`fetch`, `localStorage`, `WebSocket`)
Browsers provide tools to interact with things beyond just the page structure:

- **fetch**: Allows you to make network requests. (`fetch("https://api...")`). JavaScript itself does not know how to make HTTP requests; the runtime provides this ability.
- **localStorage**: Allows you to store data in the user’s browser.
- **WebSocket**: Allows real-time, two-way communication with servers.

#### The Security Model (Sandboxing and CORS)
Browsers are designed with security in mind because they execute code from unknown websites. 

**Sandboxing**
Browser JavaScript runs inside a **sandbox**. Think of it as a controlled box: your code can play inside, but it cannot escape to harm the system. 
- It cannot read files from your computer.
- It cannot access your operating system.
- It cannot inspect other browser tabs.

**CORS (Cross-Origin Resource Sharing)**
If your webpage is loaded from `https://example.com`, your JavaScript cannot freely request data from `https://another-site.com` unless that server explicitly allows it. This prevents malicious websites from stealing your data.

### JavaScript in Node.js

Node.js is designed for server environments where system access is necessary. It does not provide `document` or `window`, but it gives you:

- **File system access (`fs`)**
- **Operating system access**
- **Process control**
- **Server creation (`http` module)**

```js
const fs = require("fs");
fs.readFile("file.txt", "utf8", (err, data) => {
  console.log(data);
});
```

Browsers would never allow this — it would be a massive security risk.

## The Big Idea

JavaScript by itself is just a programming language. But it becomes powerful because of the runtime it lives inside. The runtime:
- Provides APIs (like DOM, fetch, fs)
- Controls security rules
- Determines what your code can interact with

This explains why some code works in the browser but not in Node, and vice-versa. The engine executes the logic, but the runtime gives it the body to interact with the world.

## Your First Program: JavaScript vs TypeScript

The traditional first program prints a message.

{% tabs %}

{% tab title="JavaScript" %}

```js
console.log("Hello, world!");
```

- **Execution**: Runs directly in the runtime (Browser/Node).
- **Difference**: Plain JavaScript logic without extra annotations.

{% endtab %}

{% tab title="TypeScript" %}

```ts
const message: string = "Hello, world!";
console.log(message);
```

- **Execution**: After compilation, it becomes plain JavaScript.
- **Difference**: Adds type information (`: string`). TypeScript exists only during development.

{% endtab %}

{% endtabs %}

### Run it in the Browser

{% tabs %}

{% tab title="JavaScript" %}

- Open any website.
- Press **F12** (DevTools) and go to the **Console**.
- Run: `console.log("Hello from the browser!");`
- **Result**: It works immediately.

{% endtab %}

{% tab title="TypeScript" %}

- **Browsers do not understand TypeScript.**
- **Workflow**:
  1. Write `.ts` code.
  2. Compile it to `.js`.
  3. Load the emitted `.js` in the browser.

After compilation, the browser runs only the resulting JavaScript.

{% endtab %}

{% endtabs %}

### Run it with Node.js

{% tabs %}

{% tab title="JavaScript" %}

- Create `hello.js`: `console.log("Hello from Node.js!");`
- Run: `node hello.js`
- **Result**: Node runs the JavaScript directly.

{% endtab %}

{% tab title="TypeScript" %}

- Create `hello.ts`:
  ```ts
  const message: string = "Hello from Node.js!";
  console.log(message);
  ```
- Compile: `tsc hello.ts`
- Run the resulting JS: `node hello.js`
- **Result**: Node runs the emitted JavaScript; TypeScript disappears after compilation.

{% endtab %}

{% endtabs %}

## Code Structure: JS vs TS Side-by-Side

### Statements
{% tabs %}

{% tab title="JavaScript" %}

```js
let count = 1;
count = count + 1;
```
- No built-in type annotations.

{% endtab %}

{% tab title="TypeScript" %}

```ts
let count: number = 1;
count = count + 1;
```
- Declares the variable type (`: number`).
- **Runtime behavior**: Identical to JS.

{% endtab %}

{% endtabs %}

### Semicolons: JavaScript vs TypeScript

Semicolons are technically optional in both, but the practical usage differs.

| Feature | JavaScript | TypeScript |
| :--- | :--- | :--- |
| **Required?** | No (**ASI** handles it) | No (follows JS syntax rules) |
| **Common Practice** | Often used for safety | Very often enforced by convention |

- **In JavaScript**: Automatic Semicolon Insertion (ASI) handles most cases, but it has edge cases. Many teams use them for consistency.
- **In TypeScript**: Follows the same rules, but real-world TS projects often use strict formatting tools (**ESLint**, **Prettier**) that require semicolons.

### Comments

Same syntax in both; TypeScript does not change comment behavior.

```js
// single-line comment
/* multi-line 
   comment */
```

## Strict Mode: The Most Confusing “Strict”

The word "strict" means something completely different in JS and TS.

{% tabs %}

{% tab title="JavaScript (Runtime)" %}

Enabled with `"use strict";` (automatic in ES modules). It affects **runtime behavior**:
- Prevents accidental globals.
- Throws errors for unsafe actions.
- Makes the engine execution more predictable.

{% endtab %}

{% tab title="TypeScript (Compiler)" %}

Enabled via a setting in `tsconfig.json`: `{"strict": true}`. It affects **compile-time safety**:
- Enables stronger type checking.
- catches common mistakes (like `undefined` access) before running.

{% endtab %}

{% endtabs %}

**The mental model**: JavaScript strict affects how the code **runs**. TypeScript strict affects how the code is **checked** during development.

---

## The Core Mental Model

Whenever you learn something new in the JS/TS ecosystem, ask:

1. **Is this runtime behavior?** → That’s JavaScript.
2. **Is this type checking or development tooling?** → That’s TypeScript.
3. **Does this code still exist after compilation?** → If yes, it’s JavaScript.
4. **Does it disappear after compilation?** → It’s TypeScript-only.

## Final Perspective

JavaScript is the engine’s language. TypeScript is a **safety layer** on top of it. They share the same runtime, the same execution model, and the same syntax foundation. TypeScript just adds a static type system to catch bugs early.

## Tasks

{% tabs %}

{% tab title="JavaScript" %}

- **Task 1 (Browser)**: Open DevTools and run a `console.log` message.
- **Task 2 (Node.js)**: Create `hello.js` and run it using `node hello.js`.
- **Task 3 (ASI)**: Try writing code without semicolons and see if it runs in the browser console.

{% endtab %}

{% tab title="TypeScript" %}

- **Task 1 (Workflow)**: Write a `.ts` file, compile it using `tsc`, and run the resulting `.js` file.
- **Task 2 (Type Safety)**: In a TS file, declare a `number` and try to assign a `string`. Observe the compiler error.

{% endtab %}

{% endtabs %}

## Important things to know

This section consolidates key terms and concepts introduced in this chapter.

### Core Concepts & Keywords

- **JavaScript Engine**: The "brain" inside a runtime (like V8) that reads and executes code logic. It does not natively know about webpages or files.
- **Runtime**: The "body" that wraps an engine and provides **APIs** (powers) to interact with the world (e.g., Browser for web pages, Node.js for servers).
- **API (Application Programming Interface)**: The set of tools and objects provided by the runtime (like `fetch`, `document`, or `fs`) that let you use its powers.
- **DOM (Document Object Model)**: A tree-like representation of your HTML that the browser provides so JavaScript can change the page.
- **Sandboxing**: A security system that keeps your code in a "sandbox" so it cannot harm your computer or access private files.
- **CORS (Cross-Origin Resource Sharing)**: A browser security rule that prevents websites from making requests to other sites unless explicitly permitted.
- **ASI (Automatic Semicolon Insertion)**: A JavaScript feature where the engine tries to add semicolons for you if you leave them out.

### Looking Ahead

You encountered several terms that were not fully explored yet. You don’t need to master these now, but you will see them again:

- **ES Modules**: A modern way of organizing code that enables "Strict Mode" automatically.
- **Static Type System**: What TypeScript adds to JavaScript to catch bugs before they ever run.
- **Promises**: Special JavaScript objects used to handle tasks that take time (like fetching data).
- **Development Tooling**: Tools like **ESLint** and **Prettier** which are used in professional projects to enforce rules (like semicolons) and maintain code quality.
