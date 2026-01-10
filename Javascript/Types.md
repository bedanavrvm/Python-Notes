# Types & the Type System (JavaScript vs TypeScript)

JavaScript has *runtime* types.
TypeScript adds a *compile-time* type system.

## Runtime types in JavaScript

In JS, the engine decides types at runtime.

```js
typeof 123;        // "number"
typeof "hello";    // "string"
typeof true;       // "boolean"
typeof undefined;  // "undefined"
typeof null;       // "object" (legacy quirk)
Array.isArray([]); // true
```

## Static types in TypeScript

In TS, you describe allowed values.

```ts
let id: number = 123;
// id = "abc"; // compile-time error
```

## Core TS types (and how they map to JS)

- `string`, `number`, `boolean`: match JS primitives
- `null`, `undefined`: exist at runtime; TS can force you to handle them
- `any`: opt-out (dangerous)
- `unknown`: safer “I don’t know yet”
- `never`: impossible code paths

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

```ts
type Id = string | number;

function normalizeId(id: Id): string {
  if (typeof id === "number") {
    return String(id);
  }
  return id;
}
```

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

## Structural typing (why TS feels “JS-like”)

TypeScript is structurally typed: if it *looks like* the right shape, it’s allowed.

```ts
type Point = { x: number; y: number };

function printPoint(p: Point) {
  console.log(p.x, p.y);
}

printPoint({ x: 1, y: 2, z: 3 }); // ok (extra props allowed in variables)
```

<details>
<summary>Show excess property checks gotcha</summary>

```ts
type Point = { x: number; y: number };

printPoint({ x: 1, y: 2, z: 3 });
// In many setups this is an error because object literals are checked more strictly.
```

</details>

## Generics (type parameters)

Generics let you write reusable typed code.

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}

const n = first([1, 2, 3]);
const s = first(["a", "b"]);
```

## Important: TS does not validate runtime data

If you read JSON from a server, TypeScript does not magically validate it.
You still need runtime validation (manual checks or schema validation).

## `null` and `undefined` (and why TS cares)

In JavaScript, `undefined` usually means “missing” and `null` often means “explicitly empty”.

In TypeScript (especially with `strictNullChecks`), `null` and `undefined` force you to handle absence.

{% tabs %}

{% tab title="JavaScript" %}

```js
function getEmail(user) {
  // If user.email is missing, this becomes undefined and calling toLowerCase crashes.
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

## `never` (useful for exhaustiveness)

`never` describes code paths that should be impossible. It is most useful for exhaustive checks.

```ts
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "error"; message: string };

function render(s: State): string {
  switch (s.status) {
    case "idle":
      return "Idle";
    case "loading":
      return "Loading";
    case "error":
      return s.message;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}
```

## Narrowing patterns (how TS refines unions)

Common ways to narrow in TypeScript:

- `typeof x === "string"` (primitives)
- `x instanceof Date` (classes)
- `"prop" in obj` (property existence)
- discriminated unions (tag field like `kind`)

<details>
<summary>Show narrowing patterns</summary>

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

  // value is Date here
  console.log(value.toISOString());
}
```

</details>

## Literal types, `as const`, and `satisfies`

These tools help you create precise types from values.

{% tabs %}

{% tab title="JavaScript" %}

```js
const status = "idle";
// JS knows it's a string at runtime, but there's no compile-time literal typing.
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
const status = "idle" as const;
// type of status is "idle", not string

const config = {
  retries: 3,
  mode: "safe",
} satisfies { retries: number; mode: "safe" | "fast" };
```

{% endtab %}

{% endtabs %}

## Objects, arrays, tuples

Tuples are useful when you want fixed-length, position-based data.

```ts
const point: [number, number] = [10, 20];
const headers: ReadonlyArray<string> = ["Accept", "Content-Type"];
```

<details>
<summary>Show why tuples help</summary>

```ts
function move([x, y]: [number, number]): [number, number] {
  return [x + 1, y + 1];
}

// move(["x", "y"]); // compile-time error
```

</details>

## Intersection types (`&`)

Intersections combine multiple object shapes.

```ts
type Timestamped = { createdAt: Date };
type Identified = { id: string };

type Entity = Timestamped & Identified;

const e: Entity = { id: "1", createdAt: new Date() };
```

## `keyof` and indexed access types

These power a lot of “type-safe object utilities”.

```ts
type User2 = { id: string; name: string; age: number };

type UserKey = keyof User2; // "id" | "name" | "age"

function pluck<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const u: User2 = { id: "1", name: "Alice", age: 25 };
const name = pluck(u, "name");
```

## Utility types (standard library)

TypeScript ships with many reusable helpers.

<details>
<summary>Show common utility types</summary>

```ts
type User3 = { id: string; name: string; age: number };

type UserPreview = Pick<User3, "id" | "name">;
type UserWithoutAge = Omit<User3, "age">;
type PartialUser = Partial<User3>;
type RequiredUser = Required<User3>;
type UserMap = Record<string, User3>;

type Fn = (x: number) => string;
type FnReturn = ReturnType<Fn>; // string
type FnParams = Parameters<Fn>; // [number]
```

</details>

## Type guards and assertions

Type guards validate at runtime and narrow at compile time.

{% tabs %}

{% tab title="JavaScript" %}

```js
function isUser(value) {
  return value && typeof value.name === "string";
}
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string };

function isUser(value: unknown): value is User {
  return !!value && typeof (value as any).name === "string";
}
```

{% endtab %}

{% endtabs %}

### Type assertions (`as`)

An assertion tells the compiler “trust me” without adding runtime checks.

```ts
const raw: unknown = JSON.parse('{"name":"Alice"}');
const user = raw as { name: string }; // unsafe if raw is not correct
```

## Enums vs unions

Enums exist at runtime in a way that union types do not.
In modern TS codebases, unions are often preferred unless you need the runtime object.

{% tabs %}

{% tab title="Union type" %}

```ts
type Status = "idle" | "loading" | "error";

function setStatus(s: Status) {}
```

{% endtab %}

{% tab title="Enum" %}

```ts
enum Status {
  Idle = "idle",
  Loading = "loading",
  Error = "error",
}

function setStatus(s: Status) {}
```

{% endtab %}

{% endtabs %}

## Practical approach: validate at the boundary

- Parse unknown input (`unknown`)
- Validate required fields
- Convert into safe internal types

<details>
<summary>Show a minimal runtime validation pattern (no external libraries)</summary>

```ts
type User4 = { id: string; name: string };

function parseUser(value: unknown): User4 {
  if (!value || typeof value !== "object") {
    throw new Error("Expected object");
  }

  const v = value as any;
  if (typeof v.id !== "string") throw new Error("Expected id: string");
  if (typeof v.name !== "string") throw new Error("Expected name: string");

  return { id: v.id, name: v.name };
}
```

</details>

## Summary

- JavaScript has runtime types; TypeScript adds compile-time types.
- `unknown` + narrowing is safer than `any`.
- Unions + narrowing model real-world branching logic.
- Utility types (`Pick`, `Omit`, `Partial`, `Record`, etc.) reduce repetitive typing.
- TypeScript cannot validate runtime data: validate at boundaries.

## Important Keywords

### **Static typing**

Type checking performed at compile time (TypeScript) rather than runtime.

### **Runtime type**

The actual type/value a JS engine sees while executing.

### **Union type**

Type that can be one of several options (`string | number`).

### **Narrowing**

Refining a union type using checks (`typeof`, `in`, `instanceof`).

### **Type guard**

Function that validates at runtime and narrows at compile time (`value is User`).

### **Type assertion**

Telling TS to trust a type without runtime validation (`value as User`).

### **Literal type**

A specific value as a type (`"idle"` instead of `string`).

### **`keyof`**

Operator that produces the union of object keys.

### **Utility types**

Built-in helper types like `Pick`, `Omit`, `Partial`, and `Record`.
