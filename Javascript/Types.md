# Types (JavaScript & TypeScript)

A value in JavaScript is always of a certain type (for example, a number or a
string).

JavaScript and TypeScript both work with the same runtime, but they talk about
types in different ways.

{% tabs %}

{% tab title="JavaScript" %}

- Types exist at runtime.
- Variables can hold any kind of value.
- Mistakes show up when the code runs.

{% endtab %}

{% tab title="TypeScript" %}

- The runtime is still JavaScript.
- TypeScript adds compile-time checks, then emits JavaScript.
- Many mistakes are caught before running.

{% endtab %}

{% endtabs %}

## Values, variables, and dynamic typing

JavaScript is dynamically typed: the same variable can hold different types at
different times.

{% tabs %}

{% tab title="JavaScript" %}

```js
let message = "hello";
message = 123456;
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
let message = "hello";
// message = 123456; // compile-time error
```

{% endtab %}

{% endtabs %}

## The 8 basic JavaScript types

There are 8 basic types in JavaScript.

- **Primitive types**: `number`, `bigint`, `string`, `boolean`, `null`,
  `undefined`, `symbol`
- **Non-primitive type**: `object`

TypeScript uses many of the same names as annotations (for example `string` and
`number`), but those annotations do not exist at runtime.

## Checking runtime types: `typeof` and `Array.isArray`

`typeof` is a JavaScript operator that returns a string describing the runtime
type.

{% tabs %}
# 4. Types (JavaScript & TypeScript)

A value in JavaScript is always of a certain type at runtime. When your program runs, every piece of data already *is* something: a number, a string, an object, etc.

TypeScript does not change this runtime reality. Instead:
* **JavaScript** → types exist at runtime.
* **TypeScript** → types exist at compile time (before runtime).

This is the single most important idea in this chapter. JavaScript and TypeScript run on the same engine; TypeScript just adds a layer of static analysis before your code becomes JavaScript.

## 1. Two Different Conversations About Types

### In JavaScript
Types exist automatically at runtime, are attached to **values** (not variables), and are flexible (**dynamic typing**). Errors only show up when the code actually runs.
```js
let message = "hello";
message = 123456; // Perfectly valid in JS
```

### In TypeScript
The runtime is still JavaScript, but TypeScript adds a compile step.
```ts
let message = "hello";
message = 123456; // ❌ Compile-time error
```
TypeScript **inferred** `let message: string` and enforces that contract. After compilation, these annotations disappear, and the resulting JS behaves exactly like JS.

## 2. The 8 Basic JavaScript Types

JavaScript has 8 fundamental types at runtime. TypeScript uses these same names in annotations, but deletes them during compilation.

### Primitives (Stored by Value)
1. **number**: Includes `10`, `3.14`, `NaN`, and `Infinity`.
2. **bigint**: For very large integers (`10n`).
3. **string**: Text data.
4. **boolean**: `true` or `false`.
5. **null**: Represents an explicit "empty" value.
6. **undefined**: Represents a "missing" value.
7. **symbol**: Unique and immutable identifiers.

### Non-Primitives (Stored by Reference)
8. **object**: Includes objects `{}`, arrays `[]`, and functions.

## 3. Checking Runtime Types: `typeof`

`typeof` is a JavaScript runtime tool. It behaves the same in both languages because the runtime *is* JavaScript.

{% tabs %}
{% tab title="JavaScript" %}
```js
typeof undefined;     // "undefined"
typeof 0;             // "number"
typeof 10n;           // "bigint"
typeof true;          // "boolean"
typeof "foo";         // "string"
typeof Symbol("id");  // "symbol"
typeof Math;          // "object"
typeof null;          // "object" (Historical Bug!)
typeof alert;         // "function"

Array.isArray([]);    // true
```
{% endtab %}
{% tab title="TypeScript" %}
```ts
const value: unknown = "foo";

if (typeof value === "string") {
  // In this block, TS "narrows" value to a string
  console.log(value.toUpperCase());
}
```
**Key Difference**: In TS, `typeof` checks actually "narrows" the type at compile-time.
{% endtab %}
{% endtabs %}

## 4. Type Conversions (The Pitfalls)

All conversions are JavaScript runtime behavior. TypeScript helps you reason about them safely.

### Numeric Conversion
JavaScript converts types automatically, which can be dangerous.
```js
Number("123z"); // NaN
+"2" + +"3";   // 5
"2" + "3";     // "23" (Common Pitfall!)
```

### Boolean Conversion (Truthiness)
Boolean conversion follows specific rules:
```js
Boolean(0);   // false
Boolean("0"); // true (Even the string "0" is TRUTHY!)
Boolean("");  // false
```

## 5. `any` vs `unknown` (TypeScript Strength)

### `any`
Disables type checking. It tells TypeScript: "I don't care, treat this like raw JavaScript."
```ts
function parse(value: any) {
  return value.toUpperCase(); // No error here, but might crash at runtime!
}
```

### `unknown`
Forces you to **prove** what something is before using it.
```ts
function parse(value: unknown): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // Safe narrowing
  }
  throw new Error("Expected a string");
}
```
**Rule of thumb**: `any` = "I don't care." `unknown` = "I don't know yet." **Always prefer `unknown`**.

## 6. Optionality: `undefined` vs Optional Properties

{% tabs %}
{% tab title="JavaScript" %}
```js
const user = { name: "Alice" };
console.log(user.email); // undefined
```
Missing properties simply return `undefined` without a crash.
{% endtab %}
{% tab title="TypeScript" %}
```ts
type User = { name: string; email?: string };
const user: User = { name: "Alice" };


## 7. Narrowing with Checks

When you have a variable that could be "one of many types" (a union), you use runtime checks to narrow down which one it is.

{% tabs %}
{% tab title="JavaScript" %}
```js
function printValue(value) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
    return;
  }
  if (typeof value === "number") {
    console.log(value.toFixed(2));
    return;
  }
  if (value instanceof Date) {
    console.log(value.toISOString());
    return;
  }
  console.log("Unknown value");
}
```
{% endtab %}
{% tab title="TypeScript" %}
```ts
function printValue(value: string | number | Date) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
    return;
  }
  if (typeof value === "number") {
    console.log(value.toFixed(2));
    return;
  }
  // TS knows if it's not string/number, it MUST be Date
  console.log(value.toISOString());
}
```
{% endtab %}
{% endtabs %}

## 8. Validate at the Boundary

TypeScript does **not** validate runtime data for you. When you fetch JSON from a server, it arrives as `unknown`. You must validate it manually.

{% tabs %}
{% tab title="The Validation Pattern (TS)" %}
```ts
type User = { id: string; name: string };

function parseUser(value: unknown): User {
  if (typeof value !== "object" || value === null) {
    throw new Error("Expected an object");
  }

  const record = value as Record<string, unknown>;

  if (typeof record.id !== "string" || typeof record.name !== "string") {
    throw new Error("Bad data shape: Expected id and name as strings");
  }

  return { id: record.id, name: record.name };
}
```
* **Runtime checks** protect from bad data.
* **TypeScript** ensures the rest of your app can trust the result is a `User`.
{% endtab %}
{% endtabs %}

## Final Summary
* **JavaScript** = Behavior (types move and change during execution).
* **TypeScript** = Contracts (types are agreed upon before execution).
* **Narrowing** is how we satisfy the compiler using runtime logic.

***

## Tasks

### JavaScript
1. **Predict Results**:
   * `typeof null`
   * `typeof []`
   * `Boolean("0")`
   * `"" + 1 + 0`
2. **Fix this** so it prints `5` (not `"23"`):
   ```js
   console.log("2" + "3");
   ```

### TypeScript
1. Rewrite the JavaScript `parseUser` logic using `unknown`.
2. Ensure the function correctly validates that `id` and `name` exist as strings.
3. **Requirement:** Do not use `any`.
