# Best Practices (JavaScript vs TypeScript)

This chapter focuses on patterns that keep JS/TS code predictable and maintainable.

## Prefer `const`, then `let`

`const` prevents accidental reassignment.

```js
const baseUrl = "https://example.com";
let retries = 0;
```

<details>
<summary>Remember: const does not make values immutable</summary>

```js
const user = { name: "Alice" };
user.name = "Bob"; // ok

// user = { name: "Carol" }; // not ok
```

</details>

## Prefer explicit boundaries (validate external inputs)

In JS/TS, the biggest bugs come from boundary crossings:

- HTTP JSON
- localStorage
- environment variables
- user input

TypeScript does not validate these at runtime.
Validate boundary inputs and convert to safe internal types.

## Avoid implicit `any` (TypeScript)

Use `strict` mode so you catch missing types.

```ts
// Bad in large codebases
// function f(x) { return x; }

function f(x: unknown): unknown {
  return x;
}
```

### Prefer `unknown` + narrowing over `any`

`unknown` forces you to check before using a value.
This reduces “type-safety leakage”.

## Don’t overuse type assertions

Type assertions (`as Something`) bypass safety checks.
Use them at well-validated boundaries, not as a way to “make errors go away”.

<details>
<summary>Show assertion vs narrowing</summary>

{% tabs %}

{% tab title="Bad" %}

```ts
const raw: unknown = JSON.parse("{}");
const user = raw as { name: string };
console.log(user.name.toUpperCase());
```

{% endtab %}

{% tab title="Better" %}

```ts
const raw: unknown = JSON.parse("{}");
if (raw && typeof raw === "object" && typeof (raw as any).name === "string") {
  console.log((raw as any).name.toUpperCase());
}
```

{% endtab %}

{% endtabs %}

</details>

## Use strict equality

```js
if (value === 0) {
  // ...
}
```

## Prefer pure functions for core logic

- easier to test
- fewer hidden side effects

## Prefer small, explicit modules

- Keep modules focused and single-purpose.
- Use explicit exports to keep dependency graphs understandable.
- Avoid “barrel files” (`index.ts` exporting everything) if they hide heavy imports or create cycles.

## Model data boundaries

The safest pattern is to:

1. Treat inputs as `unknown`
2. Validate
3. Convert to internal types

<details>
<summary>Show boundary validation pattern</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
function parsePort(value) {
  const n = Number(value);
  if (!Number.isInteger(n) || n <= 0) throw new Error("Invalid port");
  return n;
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
function parsePort(value: string): number {
  const n = Number(value);
  if (!Number.isInteger(n) || n <= 0) throw new Error("Invalid port");
  return n;
}
```

{% endtab %}

{% endtabs %}

</details>

## Avoid `null` when possible (TS)

Many TS teams prefer `undefined` + optional properties and keep `null` for interop.

## Don’t swallow errors

Handle errors at the right layer:

- log with context
- either recover or fail fast

<details>
<summary>Show adding context and rethrowing</summary>

```js
async function loadConfig() {
  try {
    return JSON.parse(await readFile("config.json", "utf8"));
  } catch (err) {
    throw new Error("loadConfig failed", { cause: err });
  }
}
```

</details>

## Async discipline

- Don’t forget `await` (errors can escape your `try/catch`)
- Prefer `Promise.all` when tasks are independent
- Use timeouts/cancellation where appropriate

## Prefer small modules with explicit exports

- clear dependency graph
- easier refactors

## Naming and consistency

- consistent naming for files and exports
- avoid default exports for large codebases

## Summary

- JS: discipline is mostly convention + tests.
- TS: discipline can be encoded in types (but still needs runtime validation).
- Put safety at boundaries: validate inputs, narrow types, avoid `any`.
- Prefer small modules, explicit exports, and predictable error handling.
- The best DX comes from combining types, linting, formatting, and tests.

## Important Keywords

### **Boundary**

Where untrusted data enters your system (HTTP, env vars, user input).

### **Narrowing**

Refining a broader type (like `unknown`) into a specific safe type.

### **`any`**

TypeScript escape hatch that removes type safety downstream.

### **Type assertion**

Telling TS to trust a type (`as T`) without runtime validation.

### **Pure function**

Function with no side effects and predictable outputs for the same inputs.

### **Module boundary**

Public surface area of a module (exports) that callers depend on.
