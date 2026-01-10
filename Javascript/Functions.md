# Functions (JavaScript vs TypeScript)

Functions are the core abstraction in both JavaScript and TypeScript.

TypeScript adds:

- parameter and return types
- overload signatures
- stricter checks around optional parameters and `this`

In both languages, functions are **first-class values**:

- You can store them in variables
- Pass them as arguments
- Return them from other functions
- Attach them to objects (methods)

The practical difference is:

- JavaScript gives you flexibility at runtime.
- TypeScript gives you safety at compile time (stronger contracts at the boundaries).

## Function declarations vs expressions

### Why there are multiple forms

- **Function declarations** are hoisted (you can call them before they appear in the file).
- **Function expressions** behave like normal values (useful when you want to pass/assign a function).
- **Arrow functions** have different `this` behavior (lexical `this`) and are very common in callbacks.

{% tabs %}

{% tab title="JavaScript" %}

```js
// Function declaration (hoisted)
function add(a, b) {
  return a + b;
}

// Function expression
const sub = function (a, b) {
  return a - b;
};

// Arrow function
const mul = (a, b) => a * b;
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function add(a: number, b: number): number {
  return a + b;
}

const sub = function (a: number, b: number): number {
  return a - b;
};

const mul = (a: number, b: number): number => a * b;
```

{% endtab %}

{% endtabs %}

## Spread arguments (the “unpacking” equivalent)

JavaScript uses `...` both for:

- **rest parameters** (collect arguments)
- **spread** (expand an array/object)

{% tabs %}

{% tab title="JavaScript" %}

```js
function add3(a, b, c) {
  return a + b + c;
}

const values = [1, 2, 3];
console.log(add3(...values));
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function add3(a: number, b: number, c: number): number {
  return a + b + c;
}

const values: number[] = [1, 2, 3];
console.log(add3(...values));
```

{% endtab %}

{% endtabs %}

<details>
<summary>Show hoisting differences (declaration vs expression)</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
// Works (declaration is hoisted)
console.log(add(1, 2));
function add(a, b) {
  return a + b;
}

// Fails (expression is not callable before assignment)
// console.log(sub(2, 1)); // TypeError: sub is not a function
const sub = function (a, b) {
  return a - b;
};
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
// Same runtime behavior as JS.
console.log(add(1, 2));
function add(a: number, b: number): number {
  return a + b;
}

// With strict TS, you often get earlier feedback when you use variables
// before initialization.
const sub = (a: number, b: number): number => a - b;
```

{% endtab %}

{% endtabs %}

</details>

## Optional parameters and defaults

- In JS, you can call a function with fewer arguments; missing parameters become `undefined`.
- In TS, optional parameters must be marked with `?` (and then become `T | undefined`).

{% tabs %}

{% tab title="JavaScript" %}

```js
function greet(name, title = "") {
  if (!name) return "Hello";
  return `Hello ${title} ${name}`.trim();
}

greet();
greet("Alice");
greet("Alice", "Dr.");
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function greet(name?: string, title: string = ""): string {
  if (!name) return "Hello";
  return `Hello ${title} ${name}`.trim();
}
```

{% endtab %}

{% endtabs %}

### Common pitfall: “optional” doesn’t mean “safe”

In JavaScript, missing parameters become `undefined`. That means you must decide whether “missing” is valid input.

{% tabs %}

{% tab title="JavaScript" %}

```js
function repeat(text, times) {
  // If times is undefined, this loop does nothing and silently returns ""
  let out = "";
  for (let i = 0; i < times; i++) out += text;
  return out;
}

repeat("a");
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function repeat(text: string, times: number): string {
  let out = "";
  for (let i = 0; i < times; i++) out += text;
  return out;
}

// repeat("a"); // compile-time error
```

{% endtab %}

{% endtabs %}

## Rest parameters

{% tabs %}

{% tab title="JavaScript" %}

```js
function sum(...nums) {
  return nums.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function sum(...nums: number[]): number {
  return nums.reduce((acc, n) => acc + n, 0);
}
```

{% endtab %}

{% endtabs %}

## Callbacks and higher-order functions

TypeScript shines when you pass functions around: it can type-check the callback signature.

{% tabs %}

{% tab title="JavaScript" %}

```js
function map(arr, fn) {
  const out = [];
  for (const item of arr) out.push(fn(item));
  return out;
}

map([1, 2, 3], (n) => n * 2);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function map<T, R>(arr: T[], fn: (item: T) => R): R[] {
  const out: R[] = [];
  for (const item of arr) out.push(fn(item));
  return out;
}

map([1, 2, 3], (n) => n * 2);
map(["a", "b"], (s) => s.toUpperCase());
```

{% endtab %}

{% endtabs %}

## Function types (TypeScript)

In TS, you can name function signatures to keep APIs readable.

```ts
type Mapper<T, R> = (item: T) => R;

function map<T, R>(arr: T[], fn: Mapper<T, R>): R[] {
  const out: R[] = [];
  for (const item of arr) out.push(fn(item));
  return out;
}
```

## Return types and inference

- TS can often infer the return type.
- Explicit return types are helpful for public APIs.

```ts
function parseId(value: string) {
  return value.trim();
}
// inferred: (value: string) => string
```

## Function overloads (TypeScript)

JavaScript commonly implements “overloads” by checking types at runtime.
TypeScript supports overload signatures to make those patterns safer.

<details>
<summary>Show runtime overloading (JS) vs overload signatures (TS)</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
function format(value) {
  if (typeof value === "number") return value.toFixed(2);
  if (typeof value === "string") return value.trim();
  throw new Error("Unsupported type");
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function format(value: number): string;
function format(value: string): string;
function format(value: number | string): string {
  if (typeof value === "number") return value.toFixed(2);
  return value.trim();
}
```

{% endtab %}

{% endtabs %}

</details>

## The `this` parameter (TypeScript)

In JS, `this` is a runtime value and can be confusing.
TypeScript allows a `this` parameter annotation for functions.

<details>
<summary>Show typing `this` in TS</summary>

```ts
type User = { name: string };

function greet(this: User) {
  return `Hello ${this.name}`;
}

const user: User = { name: "Alice" };
const msg = greet.call(user);
```

</details>

## `this` binding rules (why it’s confusing in JS)

The value of `this` in JavaScript depends on *how the function is called*, not where it’s defined.

- `obj.method()` sets `this` to `obj`
- `fn()` sets `this` to `undefined` in strict mode (or the global object in sloppy mode)
- `fn.call(x)` sets `this` to `x`
- Arrow functions capture `this` from the surrounding scope

{% tabs %}

{% tab title="JavaScript" %}

```js
const obj = {
  name: "Alice",
  regular() {
    return this.name;
  },
  arrow: () => {
    // `this` is not obj here; it's captured from outer scope
    return this?.name;
  },
};

console.log(obj.regular());
console.log(obj.arrow());
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
const obj = {
  name: "Alice",
  regular() {
    return this.name;
  },
  arrow: () => {
    return (this as any)?.name;
  },
};
```

TypeScript can help you type `this` in standalone functions, but it can’t change the runtime rule.
{% endtab %}

{% endtabs %}

## Closures

A **closure** is a function that “remembers” variables from its outer scope. This is the foundation of:

- factories
- memoization
- module patterns
- private state

{% tabs %}

{% tab title="JavaScript" %}

```js
function makeCounter() {
  let value = 0;
  return function inc() {
    value += 1;
    return value;
  };
}

const c = makeCounter();
console.log(c());
console.log(c());
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function makeCounter(): () => number {
  let value = 0;
  return function inc(): number {
    value += 1;
    return value;
  };
}
```

{% endtab %}

{% endtabs %}

## Recursion

Recursion exists in JS just like Python, but the practical constraints differ:

- JavaScript engines generally have smaller recursion limits than Python.
- Tail-call optimization is not reliably available across environments.

{% tabs %}

{% tab title="JavaScript" %}

```js
function sumRecursive(arr) {
  if (arr.length === 0) return 0;
  return arr[0] + sumRecursive(arr.slice(1));
}

function sumIterative(arr) {
  let total = 0;
  for (const n of arr) total += n;
  return total;
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function sumRecursive(arr: number[]): number {
  if (arr.length === 0) return 0;
  return arr[0] + sumRecursive(arr.slice(1));
}

function sumIterative(arr: number[]): number {
  let total = 0;
  for (const n of arr) total += n;
  return total;
}
```

{% endtab %}

{% endtabs %}

## Async functions

`async` functions always return a Promise at runtime.

{% tabs %}

{% tab title="JavaScript" %}

```js
async function getName() {
  return "Alice";
}

getName().then(console.log);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
async function getName(): Promise<string> {
  return "Alice";
}
```

{% endtab %}

{% endtabs %}

## Practical patterns

<details>
<summary>Show debounce (UI / event-heavy code)</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
function debounce(fn, delayMs) {
  let id;
  return (...args) => {
    clearTimeout(id);
    id = setTimeout(() => fn(...args), delayMs);
  };
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function debounce<TArgs extends unknown[]>(
  fn: (...args: TArgs) => void,
  delayMs: number
): (...args: TArgs) => void {
  let id: ReturnType<typeof setTimeout> | undefined;
  return (...args: TArgs) => {
    if (id) clearTimeout(id);
    id = setTimeout(() => fn(...args), delayMs);
  };
}
```

{% endtab %}

{% endtabs %}

</details>

<details>
<summary>Show memoization (CPU-heavy pure functions)</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
function memoize(fn) {
  const cache = new Map();
  return (arg) => {
    if (cache.has(arg)) return cache.get(arg);
    const value = fn(arg);
    cache.set(arg, value);
    return value;
  };
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function memoize<TArg, TResult>(fn: (arg: TArg) => TResult): (arg: TArg) => TResult {
  const cache = new Map<TArg, TResult>();
  return (arg: TArg) => {
    const cached = cache.get(arg);
    if (cached !== undefined) return cached;
    const value = fn(arg);
    cache.set(arg, value);
    return value;
  };
}
```

Note: for `undefined`-valid results you’d want a different cache sentinel.
{% endtab %}

{% endtabs %}

</details>

## Composition (pipeline-style)

In JS/TS you often compose small functions to build a data-processing pipeline.

{% tabs %}

{% tab title="JavaScript" %}

```js
function pipeline(value, ...fns) {
  return fns.reduce((acc, fn) => fn(acc), value);
}

const cleaned = pipeline(
  "   HELLO WORLD   ",
  (s) => s.trim(),
  (s) => s.toLowerCase(),
  (s) => s[0].toUpperCase() + s.slice(1)
);

console.log(cleaned);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function pipeline<T>(value: T, ...fns: Array<(v: any) => any>) {
  return fns.reduce((acc, fn) => fn(acc), value);
}
```

For full type safety, you typically write overloads or use a dedicated functional library.
{% endtab %}

{% endtabs %}

## Summary

- JavaScript functions are flexible at runtime.
- TypeScript makes function boundaries explicit: inputs/outputs and overload behavior.
- Prefer TS generics for reusable utilities (like `map`, `first`, `groupBy`).

## Important Keywords

### **Hoisting**

Behavior where declarations are processed before code execution (function declarations are hoisted; function expressions are not callable before assignment).

### **Closure**

Function that captures variables from an outer scope and can keep them alive.

### **Higher-order function**

Function that takes a function as input and/or returns a function.

### **Callback**

Function passed to another function to be called later (often in async workflows).

### **Arrow function**

Function syntax with lexical `this` and concise expressions: `(a) => a + 1`.

### **Overload (TypeScript)**

Multiple call signatures for the same function implementation, used to model runtime branching.

### **Rest parameters / Spread**

`...args` collects arguments; `fn(...arr)` spreads an array into arguments.

### **Type inference**

TypeScript’s ability to deduce types from code without explicit annotations.

### **Async function**

Function declared with `async` that returns a Promise at runtime.
