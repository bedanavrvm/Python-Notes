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

JavaScript is a general-purpose language that can run in multiple
environments.

- **In the browser**, it can interact with web pages.
- **On servers (Node.js)**, it can read files, handle HTTP requests, etc.

JavaScript itself is the language. The environment decides which extra APIs
you can use.

## Where JavaScript runs (environments)

JavaScript code runs inside a **runtime** (a JavaScript engine). Different
runtimes give you different APIs.

{% tabs %}

{% tab title="Browser" %}

- **DOM APIs** (`document`, `window`) to read/update the page
- **Web APIs** (`fetch`, `localStorage`, `WebSocket`)
- **Security model** (sandboxing, CORS)
{% endtab %}

{% tab title="Node.js" %}

- **Node APIs** (`fs`, `path`, `http`)
- **Different globals** (`process`, `Buffer`)
- **Different module systems** (ESM/CommonJS)
{% endtab %}

{% endtabs %}

If something works in one environment but not the other, it’s usually because
the **API** is missing (not because “JavaScript is different”).

TypeScript does not change which APIs exist at runtime. It adds type
definitions so your editor and compiler can warn you earlier.

## Your first JavaScript program

The easiest first program is printing a message.

{% tabs %}

{% tab title="JavaScript" %}

```js
console.log("Hello, world!");
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
const message: string = "Hello, world!";
console.log(message);
```

{% endtab %}

{% endtabs %}

### Run it in the browser

{% tabs %}

{% tab title="JavaScript" %}

- Open any website.
- Open DevTools (usually `F12`).
- Go to the **Console** tab.
- Paste:

```js
console.log("Hello from the browser!");
```

{% endtab %}

{% tab title="TypeScript" %}

Browsers run JavaScript, not TypeScript.

- Write TypeScript in a `.ts` file.
- Compile it to `.js`.
- Load the emitted `.js` in the browser.

Example TypeScript:

```ts
const message: string = "Hello from TypeScript!";
console.log(message);
```

After compilation, the browser runs JavaScript.

{% endtab %}

{% endtabs %}

### Run it with Node.js

{% tabs %}

{% tab title="JavaScript" %}

- Create a file named `hello.js`.
- Put this inside:

```js
console.log("Hello from Node.js!");
```

- Run:

```bash
node hello.js
```

{% endtab %}

{% tab title="TypeScript" %}

- Create a file named `hello.ts`.
- Put this inside:

```ts
const message: string = "Hello from Node.js!";
console.log(message);
```

- Compile TypeScript to JavaScript:

```bash
tsc hello.ts
```

- Run the emitted JavaScript:

```bash
node hello.js
```

{% endtab %}

{% endtabs %}

## A tiny preview of JS code structure

You don’t need to memorize rules yet. Just recognize these patterns:

- **Statements** usually go on their own line.
- **Semicolons** are optional in most cases, but many teams still use them
  consistently.
- **Comments** are for humans:

{% tabs %}

{% tab title="JavaScript" %}

```js
// single-line comment
let count = 1;
count = count + 1;
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
// single-line comment
let count: number = 1;
count = count + 1;
```

{% endtab %}

{% endtabs %}

### About strict mode

“Strict” means two different things in JS and TS.

{% tabs %}

{% tab title="JavaScript" %}

Modern JavaScript avoids many old pitfalls by using **strict mode**.

- In **ES modules**, strict mode is enabled automatically.
- In **classic scripts**, you can enable it with:

```js
"use strict";
```

{% endtab %}

{% tab title="TypeScript" %}

TypeScript has a compiler option called **`strict`**, which enables stronger
type-checking rules.

- This is compile-time only (no runtime behavior change by itself).
- It helps you catch issues like `undefined` access earlier.

{% endtab %}

{% endtabs %}

## What is TypeScript (and why it exists)?

TypeScript (TS) is **JavaScript + types**.

- You write `.ts` / `.tsx`.
- TypeScript checks types at **compile-time**.
- Then it emits JavaScript, and the emitted `.js` is what actually runs.

### What TypeScript does not change

- **The runtime is still JavaScript** (V8, SpiderMonkey, JavaScriptCore).
- **Types do not exist at runtime**.
- You can still get runtime errors if your data is wrong.

Here is the key similarity/difference:

{% tabs %}

{% tab title="JavaScript" %}

```js
const id = "abc";
console.log(id);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type Id = string;
const id: Id = "abc";
console.log(id);
```

{% endtab %}

{% endtabs %}

Both run the same way at runtime. TypeScript just adds checks before you run.

### How TypeScript runs: the compilation pipeline

Typical pipeline:

1. Write TypeScript
2. Type-check (compile-time)
3. Emit JavaScript
4. Run the emitted JavaScript (browser/Node)

Compare the execution model:

{% tabs %}

{% tab title="JavaScript" %}

Write `.js` and run it directly.

{% endtab %}

{% tab title="TypeScript" %}

Write `.ts`, type-check, emit `.js`, then run the emitted `.js`.

{% endtab %}

{% endtabs %}

<details>
<summary>Show what “type-check” vs “emit” means</summary>

- **Type-check**: TypeScript validates types. This can fail the build.
- **Emit**: TypeScript outputs JavaScript. Some setups allow emitting even if
  type-check fails.

</details>

## JavaScript vs TypeScript: first comparison

The most common TypeScript benefit is catching “wrong shape” mistakes before
running.

{% tabs %}

{% tab title="JavaScript" %}

```js
function greet(user) {
  return "Hello " + user.name.toUpperCase();
}

console.log(greet({ name: "Alice" }));
console.log(greet({})); // runtime error
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string };

function greet(user: User): string {
  return "Hello " + user.name.toUpperCase();
}

console.log(greet({ name: "Alice" }));
// console.log(greet({})); // compile-time error
```

{% endtab %}

{% endtabs %}

## When to use JavaScript vs TypeScript

## When to choose JavaScript

- Small scripts or quick prototypes
- Learning the language fundamentals and runtime behavior
- Environments where you can’t (or don’t want to) add a build step

## When to choose TypeScript

- Medium/large applications
- Team projects and long-lived codebases
- Refactoring-heavy projects where safety matters
- Libraries/SDKs where the public API benefits from types

## Common terminology (you’ll see these everywhere)

- **Runtime**: the JS engine executing your code.
- **Compile-time**: TypeScript checking types and producing JavaScript.
- **Transpilation**: converting source code to another form (TS -> JS).
- **Type erasure**: TypeScript removes types when emitting JavaScript.

## Common pitfalls (JS and TS)

### 1. TypeScript can’t validate data automatically

TypeScript checks the code you wrote, not the real data you receive.

<details>
<summary>Show the JSON boundary pitfall</summary>

```ts
type User = { id: string };

const raw: unknown = JSON.parse('{"id": 123}');
const user = raw as User;

// Compiles, but can still fail later because id is not a string.
```

</details>

## Tasks

{% tabs %}

{% tab title="JavaScript" %}

- **Task 1 (Browser)**: open DevTools Console and run `console.log("hi")`.
- **Task 2 (Node.js)**: create `hello.js` and run it using `node hello.js`.
- **Task 3 (Environment check)**:
  - try `document.title` in Node (it should fail)
  - try `process.version` in the browser (it should fail)

{% endtab %}

{% tab title="TypeScript" %}

- **Task 1 (Type-check vs runtime)**:
  - write a `hello.ts` with `const x: number = 1`
  - compile to JS and run it
  - confirm the runtime output is normal JS behavior
- **Task 2 (TS mindset)**: look at the `greet({})` call above and explain:
  - why JS fails at runtime
  - why TS warns earlier

{% endtab %}

{% endtabs %}

## Summary

- JavaScript is the language; environments (browser/Node) provide different
  APIs.
- You can start learning JS immediately by running code in the browser console
  or Node.
- TypeScript is optional: it adds compile-time checks and emits JavaScript.
