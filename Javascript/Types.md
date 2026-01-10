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

{% tab title="JavaScript" %}

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

{% endtab %}

{% tab title="TypeScript" %}

At runtime, TypeScript runs as JavaScript, so `typeof` works the same.

```ts
const value: unknown = "foo";

if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

{% endtab %}

{% endtabs %}

## Type conversions (runtime)

Most of the time, operators convert values automatically. You can also convert
explicitly.

### String conversion

{% tabs %}

{% tab title="JavaScript" %}

```js
String(false); // "false"
String(null); // "null"
```

{% endtab %}

{% tab title="TypeScript" %}

Same runtime behavior. TypeScript can help you keep track of the result
type.

```ts
const x = String(false);
// x is string
```

{% endtab %}

{% endtabs %}

### Numeric conversion

`Number(value)` converts to a number. Unary `+value` is a shorter form.

{% tabs %}

{% tab title="JavaScript" %}

```js
Number("123"); // 123
Number("123z"); // NaN

+"2" + +"3"; // 5
"2" + "3"; // "23" (string concatenation)
```

{% endtab %}

{% tab title="TypeScript" %}

Same runtime behavior. TypeScript helps you avoid mixing string/number by
accident.

```ts
const a = "2";
const b = "3";
const sum = Number(a) + Number(b);
```

{% endtab %}

{% endtabs %}

### Boolean conversion

`Boolean(value)` converts to true/false.

{% tabs %}

{% tab title="JavaScript" %}

```js
Boolean(0); // false
Boolean("0"); // true (non-empty string)
Boolean(""); // false
```

{% endtab %}

{% tab title="TypeScript" %}

Same runtime behavior. TypeScript helps you make boolean intent explicit.

```ts
const enabled = Boolean("0");
```

{% endtab %}

{% endtabs %}

## any vs unknown

{% tabs %}

{% tab title="JavaScript" %}
In JS everything is effectively “any” at compile time.

```js
function parse(value) {
  return value.toUpperCase();
}

parse(123); // runtime error
```

{% endtab %}

{% tab title="TypeScript" %}
`unknown` forces checks before use.

```ts
function parse(value: unknown): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  throw new Error("Expected a string");
}
```

{% endtab %}

{% endtabs %}

## Unions and narrowing

A union expresses “one of these types”.

In JavaScript, you model this with runtime checks.
In TypeScript, you can also describe it at compile time.

{% tabs %}

{% tab title="JavaScript" %}

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

{% endtab %}

{% tab title="TypeScript" %}

```ts
type Id = string | number;

function normalizeId(id: Id): string {
  if (typeof id === "number") {
    return String(id);
  }
  return id;
}
```

{% endtab %}

{% endtabs %}

## Optionality: undefined vs optional properties

In JS, missing properties produce `undefined`.
In TS, you can model this precisely.

{% tabs %}

{% tab title="JavaScript" %}

```js
const user = { name: "Alice" };
console.log(user.email); // undefined
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string; email?: string };

const user: User = { name: "Alice" };

// user.email is string | undefined
if (user.email) {
  console.log(user.email.toLowerCase());
}
```

{% endtab %}

{% endtabs %}

## `null` and `undefined`

In JavaScript:

- `undefined` usually means “missing”.
- `null` often means “explicitly empty”.

TypeScript can force you to handle these cases at compile time.

{% tabs %}

{% tab title="JavaScript" %}

```js
function getEmail(user) {
  return user.email.toLowerCase();
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { email?: string };

function getEmail(user: User): string {
  if (!user.email) {
    return "";
  }
  return user.email.toLowerCase();
}
```

{% endtab %}

{% endtabs %}

## Objects and arrays

Objects are the “container” type in JavaScript. Arrays are also objects.

{% tabs %}

{% tab title="JavaScript" %}

```js
typeof {}; // "object"
typeof []; // "object"

Array.isArray([]); // true
Array.isArray({}); // false
```

{% endtab %}

{% tab title="TypeScript" %}

Same runtime behavior. TypeScript can also describe the shapes.

```ts
const xs: number[] = [1, 2, 3];
const obj: { id: string } = { id: "1" };

console.log(Array.isArray(xs));
console.log(typeof obj);
```

{% endtab %}

{% endtabs %}

## Narrowing values with checks

When you have “one of many possible types”, you narrow it using checks like
`typeof`, `Array.isArray`, `instanceof`, and `"prop" in obj`.

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

  console.log(value.toISOString());
}
```

{% endtab %}

{% endtabs %}

## Practical approach: validate at the boundary

TypeScript does not validate runtime data for you. If you read JSON from a
server, you must validate it at runtime.

{% tabs %}

{% tab title="JavaScript" %}

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

{% endtab %}

{% tab title="TypeScript" %}

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

{% endtab %}

{% endtabs %}

## Summary

- JavaScript types are real runtime values.
- TypeScript adds compile-time checks and then emits JavaScript.
- Use runtime checks (`typeof`, `Array.isArray`, `instanceof`) to narrow.
- Validate untrusted input (like JSON) at runtime.

## Tasks

{% tabs %}

{% tab title="JavaScript" %}

- Predict the results:
  - `typeof null`
  - `typeof []`
  - `Boolean("0")`
  - `"" + 1 + 0`
- Fix this so it prints `5` (not `"23"`):

```js
console.log("2" + "3");
```

{% endtab %}

{% tab title="TypeScript" %}

- Take the JavaScript `parseUser` and rewrite it in TypeScript using
  `unknown`.
- Create a `type User = { id: string; name: string }` and ensure your
  `parseUser` returns `User`.

{% endtab %}

{% endtabs %}
