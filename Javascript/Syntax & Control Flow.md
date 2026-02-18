# 3. Syntax & Control Flow

(JavaScript and TypeScript, Side-by-Side)

TypeScript uses the same runtime syntax as JavaScript. Everything in this chapter runs the same way at runtime in both.

The difference is:
* **JavaScript** → Runtime behavior only.
* **TypeScript** → Adds compile-time type checking.

We will always compare both.

## 1. Variables — var, let, const

Modern development uses three variable declarations.

### `var` (Legacy)
```js
var x = 1;
```
* **Properties**: Function-scoped (not block-scoped), hoisted, can be re-declared and reassigned.
* **The Surprising Part**:
  ```js
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 (visible outside the block)
  ```
* **TS Influence**: TypeScript does not change `var` behavior; it only adds type checking. 
* **Verdict**: **Avoid `var`**.

### `let`
```js
let count = 0;
count += 1;
```
* **Properties**: Block-scoped, cannot be re-declared in the same scope, can be reassigned.
{% tabs %}
{% tab title="JavaScript" %}
```js
let count = 0;
count += 1;
```
Runtime behavior is the core.
{% endtab %}
{% tab title="TypeScript" %}
```ts
let count: number = 0;
count += 1;
```
TS adds `: number`. Runtime behavior is identical.
{% endtab %}
{% endtabs %}

### `const`
```js
const user = { name: "Alice" };
user.name = "Bob"; // Allowed!
```
* **CRITICAL**: `const` means the **binding** is constant, not the **value**. You cannot reassign the variable: `user = {}` would fail. But you can mutate object properties.
{% tabs %}
{% tab title="JavaScript" %}
```js
const user = { name: "Alice" };
user.name = "Bob";
```
Execution allows mutation.
{% endtab %}
{% tab title="TypeScript" %}
```ts
const user: { name: string } = { name: "Alice" };
user.name = "Bob";
```
TS checks the *shape* of the object.
{% endtab %}
{% endtabs %}

## 2. Block Scope & Temporal Dead Zone (TDZ)

`let` and `const` are **block-scoped**:
```js
{
  let x = 1;
}
console.log(x); // ReferenceError
```

### The Temporal Dead Zone (TDZ)
Even though a variable exists in a block, it cannot be accessed before it is declared.
```js
{
  // console.log(x); // ReferenceError
  const x = 1;
  console.log(x);
}
```
This period between the start of the block and the declaration is the **TDZ**. TypeScript may warn you earlier, but the runtime rules are identical.

## 3. Equality — == vs ===

### `==` (Loose Equality)
Performs **type coercion** (automatically converts types to match).
```js
null == undefined; // true
"1" == 1;          // true
```
This leads to many bugs and surprises.

### `===` (Strict Equality)
No coercion. Values must be the same type and same value.
```js
null === undefined; // false
"1" === 1;           // false
```
**ALWAYS use `===` unless you have a specific reason to use coercion.**

### TypeScript’s Role in Equality
In `strict` mode, TypeScript prevents you from comparing unrelated types.
```ts
const n: number = 0;
const s: string = "0";

// n === s; // Compile-time error in TS
```
In JavaScript, that comparison simply returns `false` at runtime.

### Edge Case: NaN
```js
NaN === NaN; // false
```
To check for `NaN`, use:
* `Number.isNaN(NaN); // true`
* `Object.is(NaN, NaN); // true`

## 4. Conditionals

### Basic Structure
```js
if (condition) {
  // block
} else if (other) {
  // ...
} else {
  // ...
}
```

### Guard Clauses (Reading Top-to-Bottom)
Avoid deeply nested logic by returning early.
**Hard to read:**
```js
function handle(user) {
  if (user) {
    if (user.isActive) {
      return "ok";
    }
  }
  return "skip";
}
```
**Better (Guard Clauses):**
```js
function handle(user) {
  if (!user) return "skip";
  if (!user.isActive) return "skip";
  return "ok";
}
```

### TypeScript Type Narrowing
TS uses control flow to "narrow" types. After a null check, it knows the variable is defined.
```ts
function handle(user?: { isActive: boolean }) {
  if (!user) return "skip";
  // TS now knows user is DEFINED here
  return user.isActive ? "ok" : "skip";
}
```

## 5. Ternary, Optional Chaining, and Nullish Coalescing

### Ternary Operator
```js
const label = isAdmin ? "Admin" : "User";
```
Structure: `condition ? valueIfTrue : valueIfFalse`. Only use for simple expressions.

### Optional Chaining (`?.`)
Prevents `Cannot read properties of undefined` crashes.
```js
const email = user?.profile?.email;
```

### Nullish Coalescing (`??`) vs `||`
* `||` treats **all falsy values** (`0`, `""`, `false`) as missing.
* `??` only treats **`null`** and **`undefined`** as missing.

```js
0 || 10;   // 10
0 ?? 10;   // 0 (Use this when 0 is a valid value!)
```

## 6. Loops and Iteration

### The Trio of Loops
* **`for`**: The traditional counter loop.
* **`for...of`**: Iterates over **values** (best for Arrays).
* **`for...in`**: Iterates over **keys/property names** (best for Objects).

```js
const arr = ["a", "b", "c"];

for (const val of arr) { console.log(val); } // "a", "b", "c"
for (const key in arr) { console.log(key); } // "0", "1", "2"
```

### Array Methods (Clean & Functional)
* **`map`**: Transforms each element (`[1, 2].map(n => n * 2)` -> `[2, 4]`).
* **`filter`**: Keeps matching elements (`[1, 2].filter(n => n > 1)` -> `[2]`).
* **`reduce`**: Accumulates values (`[1, 2].reduce((acc, n) => acc + n, 0)` -> `3`).

### The `forEach` + `await` Pitfall
**DO NOT use `forEach` with `async/await`**. It does not wait sequentially.
```js
// This runs all at once, ignoring order!
[1, 2, 3].forEach(async (n) => { await doWork(n); });

// DO THIS INSTEAD:
for (const n of [1, 2, 3]) {
  await doWork(n);
}
```

## 7. Advanced Syntax

### Destructuring
Extract values quickly from objects and arrays.
```js
const user = { id: 1, name: "Alice" };
const { id, name } = user; // id = 1, name = "Alice"

const [x, y] = [10, 20];   // x = 10, y = 20
```

### Exhaustive `switch` (TypeScript Strength)
TS can use the `never` type to ensure you've handled every possible case in a `switch`.
```ts
type Shape = { kind: "circle" } | { kind: "square" };

function handleShape(s: Shape) {
  switch (s.kind) {
    case "circle": return "...";
    case "square": return "...";
    default:
      const _exhaustive: never = s; // Error if a new shape is added but not handled!
      return _exhaustive;
  }
}
```

## 8. Truthiness Pitfalls

JavaScript's **Falsy Values**:
* `false`, `0`, `""`, `null`, `undefined`, `NaN`.
**Everything else is Truthy**, including `{}` and `[]`.

## Final Mental Model

1. **JavaScript** defines the runtime rules (how code executes).
2. **TypeScript** adds a safety layer that catches errors *before* they run.
3. **Control Flow** works identically in both runtimes, but TypeScript narrows types based on your checks.
