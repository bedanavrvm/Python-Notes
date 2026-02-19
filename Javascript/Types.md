# Types (JavaScript & TypeScript)

A value in JavaScript is always of a certain type (for example, a number or a
string).

JavaScript and TypeScript both work with the same runtime, but they talk about
types in different ways.

### JavaScript (Runtime)

- Types exist at runtime.
- Variables can hold any kind of value.
- Mistakes show up when the code runs.

### TypeScript (Static Analysis)

- The runtime is still JavaScript.
- TypeScript adds compile-time checks, then emits JavaScript.
- Many mistakes are caught before running.

## Values, variables, and dynamic typing

JavaScript is dynamically typed: the same variable can hold different types at
different times.

### JavaScript

```js
let message = "hello";
message = 123456;
```

### TypeScript

```ts
let message = "hello";
// message = 123456; // compile-time error
```

## The 8 basic JavaScript types

There are 8 basic types in JavaScript.

- **Primitive types**: `number`, `bigint`, `string`, `boolean`, `null`,
  `undefined`, `symbol`
- **Non-primitive type**: `object`

TypeScript uses many of the same names as annotations (for example `string` and
`number`), but those annotations do not exist at runtime.

## Primitives vs. Objects (Value vs. Reference)

This is the single most important mental model in JavaScript.

*   **Primitives** (`string`, `number`, `boolean`, etc.) are passed by **value**.
*   **Objects** (`{...}`, `[...]`, `function`) are passed by **reference**.

### JavaScript (Runtime)

**Primitives are immutable and distinct.**

```js
let a = 10;
let b = a; // Copy the value '10'
a = 20;
console.log(b); // 10 (untouched)
```

**Objects share the same reference.**

```js
const x = { val: 10 };
const y = x; // Copy the *address* of the object
x.val = 20;
console.log(y.val); // 20 (they point to the same house)
```

**const** prevents reassignment, not mutation.

```js
const user = { name: "Alice" };
user.name = "Bob"; // Allowed! You didn't change the reference, just the inside.
// user = {}; // Error! You cannot change the reference.
```

### TypeScript (Static Analysis)

TypeScript cannot change how memory works, but it can force immutability checks.

```ts
// "readonly" stops mutation at compile time
interface Config {
  readonly apiUrl: string;
}

const config: Config = { apiUrl: "https://api.com" };

// config.apiUrl = "oops"; // Error: Cannot assign to 'apiUrl' because it is a read-only property.
```

## Deep Dive: The "Other" Primitives

### BigInt (Integers larger than $2^{53}$)

JavaScript numbers are floating point (IEEE 754). They become unsafe above
`9,007,199,254,740,991`.

#### JavaScript

Use the `n` suffix.

```js
const huge = 9007199254740995n; // BigInt
const num = 10;

// You cannot mix BigInt and Number
// console.log(huge + num); // TypeError: Cannot mix BigInt and other types
console.log(huge + BigInt(num)); // Works
```

#### TypeScript

TS ensures you don't mix them by accident.

```ts
const huge: bigint = 9007199254740995n;
const num: number = 10;

// const sum = huge + num; // TS Error
```

Use `target: es2020` or higher in `tsconfig.json` to use BigInt.

### Symbol (Unique Identifiers)

Symbols are guaranteed to be unique. They are often used for "hidden" properties
or library internals.

#### JavaScript

```js
const id1 = Symbol("id");
const id2 = Symbol("id");

console.log(id1 === id2); // false (always!)

const user = {
  [id1]: "Hidden Value",
  name: "Alice",
};

console.log(Object.keys(user)); // ["name"] (Symbol is hidden from iteration)
```

#### TypeScript

TS supports `unique symbol` for strict checking, though rarely needed in app
code.

```ts
const key: unique symbol = Symbol();
```

## Checking runtime types: `typeof` and `Array.isArray`

`typeof` is a JavaScript operator that returns a string describing the runtime
type.

### JavaScript

```js
typeof undefined; // "undefined"
typeof 0; // "number"
typeof 10n; // "bigint"
typeof true; // "boolean"
typeof "foo"; // "string"
typeof Symbol("id"); // "symbol"
typeof Math; // "object"
typeof null; // "object" (language quirk)
typeof alert; // "function"

Array.isArray([]); // true
```

### TypeScript

At runtime, TypeScript runs as JavaScript, so `typeof` works the same.

```ts
const value: unknown = "foo";

if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

## Type conversions (runtime)

Most of the time, operators convert values automatically. You can also convert
explicitly.

### String conversion

#### JavaScript

```js
String(false); // "false"
String(null); // "null"
```

#### TypeScript

Same runtime behavior. TypeScript can help you keep track of the result
type.

```ts
const x = String(false);
// x is string
```

### Numeric conversion

`Number(value)` converts to a number. Unary `+value` is a shorter form.

#### JavaScript

```js
Number("123"); // 123
Number("123z"); // NaN

+"2" + +"3"; // 5
"2" + "3"; // "23" (string concatenation)
```

#### TypeScript

Same runtime behavior. TypeScript helps you avoid mixing string/number by
accident.

```ts
const a = "2";
const b = "3";
const sum = Number(a) + Number(b);
```

### Boolean conversion

`Boolean(value)` converts to true/false.

#### JavaScript

```js
Boolean(0); // false
Boolean("0"); // true (non-empty string)
Boolean(""); // false
```

#### TypeScript

Same runtime behavior. TypeScript helps you make boolean intent explicit.

```ts
const enabled = Boolean("0");
```

## any vs unknown

### JavaScript

In JS everything is effectively “any” at compile time.

```js
function parse(value) {
  return value.toUpperCase();
}

parse(123); // runtime error
```

### TypeScript

`unknown` forces checks before use.

```ts
function parse(value: unknown): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  throw new Error("Expected a string");
}
```

## Unions and narrowing

A union expresses “one of these types”.

In TypeScript, you can also describe it at compile time.

### JavaScript

```js
function normalizeId(id) {
  if (typeof id === "number") {
    return String(id);
  }

  if (typeof id === "string") {
    return id;
  }

  throw new Error("Expected id to be a string or number");
}
```

### TypeScript

```ts
type Id = string | number;

function normalizeId(id: Id): string {
  if (typeof id === "number") {
    return String(id);
  }
  return id;
}
```

## Literal Types and Inference

Why does `const` sometimes have a different type than `let`?

### JavaScript

Variables hold values. Access behavior doesn't change based on how they were declared, only reassignment rules apply.

```js
const method = "GET";
let status = "pending";
```

### TypeScript

TS infers more specific types for constants.

```ts
const method = "GET"; // Type is "GET" (Literal), not just string
let status = "pending"; // Type is string (because it can change)

// Use 'as const' to freeze objects into literals
const req = { url: "https://api.com", method: "GET" } as const;
// req.method is typed as "GET", not string
```

## Type Aliases (`type`)

You can give a name to any type.

### JavaScript

Does not exist. You just write objects and functions. JSDoc is the closest equivalent.

```js
/**
 * @typedef {Object} Point
 * @property {number} x
 * @property {number} y
 */
```

### TypeScript

```ts
type Point = { x: number; y: number };
type ID = string | number;
type Callback = (data: string) => void;

const pt: Point = { x: 10, y: 20 };
```

## Optionality: undefined vs optional properties

In JS, missing properties produce `undefined`.
In TS, you can model this precisely.

### JavaScript

```js
const user = { name: "Alice" };
console.log(user.email); // undefined
```

### TypeScript

```ts
type User = { name: string; email?: string };

const user: User = { name: "Alice" };

// user.email is string | undefined
if (user.email) {
  console.log(user.email.toLowerCase());
}
```

## `null` and `undefined`

In JavaScript:

- `undefined` usually means “missing”.
- `null` often means “explicitly empty”.

TypeScript can force you to handle these cases at compile time.

### JavaScript

```js
function getEmail(user) {
  return user.email.toLowerCase();
}
```

### TypeScript

```ts
type User = { email?: string };

function getEmail(user: User): string {
  if (!user.email) {
    return "";
  }
  return user.email.toLowerCase();
}
```

## Objects and arrays

Objects are the “container” type in JavaScript. Arrays are also objects.

### JavaScript

```js
typeof {}; // "object"
typeof []; // "object"

Array.isArray([]); // true
Array.isArray({}); // false
```

### TypeScript

Same runtime behavior. TypeScript can also describe the shapes.

```ts
const xs: number[] = [1, 2, 3];
const obj: { id: string } = { id: "1" };

console.log(Array.isArray(xs));
console.log(typeof obj);
```

## Tuples

A tuple is an array with **fixed size** and **known types** at specific positions.

### JavaScript

It’s just an array. There is no runtime enforcement of "first item must be a string".

```js
const role = ["admin", 1]; // Just (string | number)[]
role[0] = 100; // Allowed
role.push("oops"); // Allowed
```

### TypeScript

TS enforces the structure.

```ts
let role: [string, number] = ["admin", 1];

// role[0] = 100; // Error: Type 'number' is not assignable to type 'string'.
// role[2] = "oops"; // Error: Index '2' is out of bounds.

// Useful for CSV rows, React hooks, or key-value pairs
function useState(initial: number): [number, (v: number) => void] {
  return [initial, (v) => {}];
}
```

## Narrowing values with checks

When you have “one of many possible types”, you narrow it using checks like
`typeof`, `Array.isArray`, `instanceof`, and `"prop" in obj`.

### JavaScript

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

### TypeScript

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

  console.log(value.toISOString());
}
```

## Practical approach: validate at the boundary

TypeScript does not validate runtime data for you. If you read JSON from a
server, you must validate it at runtime.

### JavaScript

```js
function parseUser(value) {
  if (!value || typeof value !== "object") {
    throw new Error("Expected object");
  }

  if (typeof value.id !== "string") {
    throw new Error("Expected id: string");
  }

  if (typeof value.name !== "string") {
    throw new Error("Expected name: string");
  }

  return { id: value.id, name: value.name };
}
```

### TypeScript

```ts
type User = { id: string; name: string };

type AnyRecord = Record<string, unknown>;

function isRecord(value: unknown): value is AnyRecord {
  return typeof value === "object" && value !== null;
}

function parseUser(value: unknown): User {
  if (!isRecord(value)) {
    throw new Error("Expected object");
  }

  const id = value["id"];
  const name = value["name"];

  if (typeof id !== "string") {
    throw new Error("Expected id: string");
  }

  if (typeof name !== "string") {
    throw new Error("Expected name: string");
  }

  return { id, name };
}
```

## Summary

- JavaScript types are real runtime values.
- TypeScript adds compile-time checks and then emits JavaScript.
- Use runtime checks (`typeof`, `Array.isArray`, `instanceof`) to narrow.
- Validate untrusted input (like JSON) at runtime.

## Tasks

### JavaScript

- Predict the results:
  - `typeof null`
  - `typeof []`
  - `Boolean("0")`
  - `"" + 1 + 0`
- Fix this so it prints `5` (not `"23"`):

```js
console.log("2" + "3");
```

### TypeScript

- Take the JavaScript `parseUser` and rewrite it in TypeScript using
  `unknown`.
- Create a `type User = { id: string; name: string }` and ensure your
  `parseUser` returns `User`.
