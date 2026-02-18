# 3. Syntax & Control Flow

TypeScript uses the same runtime syntax as JavaScript, with a few additions for types.

## Variables: var, let, const

* `var`: function-scoped, hoisted (legacy)
* `let`: block-scoped
* `const`: block-scoped binding (value can still be mutable)

const user = { name: "Alice" }; user.name = "Bob"; // ok: object is mutable

// Avoid var var x = 1; `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts let count: number = 0; count += 1;

const user: { name: string } = { name: "Alice" }; user.name = "Bob";

````</div></div>###

`let` and `const` are block-scoped, but they also have a *temporal dead zone* (they exist in the scope but are not usable until their declaration is evaluated).

<details>
<summary>Show TDZ behavior</summary>

```js
{
  // console.log(x); // ReferenceError (TDZ)
  const x = 1;
  console.log(x);
}
````

### `const` means “binding is constant” (not “value is immutable”)

`const` prevents reassigning the variable, but it does not freeze objects.

<details>

<summary>Show const + mutation</summary>

```js
const settings = { theme: "dark" };
settings.theme = "light"; // ok

// settings = { theme: "dark" }; // not ok
```

</details>

## Equality: == vs ===

* `==` does coercion (often surprising)
* `===` is strict equality (preferred)

null == undefined; // true null === undefined; // false

````</div><div
It can, however, help you avoid comparisons between unrelated types.

```ts
const n: number = 0;
const s: string = "0";

// n === s; // compile-time error in strict mode
```</div></div>### Equality edge cases (practical)

- Prefer `===` for almost everything.
- Know the two common exceptions:
  - comparing to `null`/`undefined` (sometimes `x == null` is used to mean “null or undefined”)
  - checking `NaN` (because `NaN !== NaN`)

<details>
<summary>Show NaN and Object.is</summary>

```js
Number.isNaN(NaN); // true
NaN === NaN; // false

Object.is(NaN, NaN); // true
Object.is(0, -0); // false (=== treats them as equal)
````

## Conditionals

```js
if (condition) {
  // ...
} else if (other) {
  // ...
} else {
  // ...
}
```

### Guard clauses (readability)

Reference-style code often prefers early returns over deeply nested `if` blocks.

<details>

<summary>Show nested vs guard clauses</summary>

\`\`\`js function handle(user) { if (user) { if (user.isActive) { return "ok"; } } return "skip"; } \`\`\`\`\`\`js function handle(user) { if (!user) return "skip"; if (!user.isActive) return "skip"; return "ok"; } \`\`\`

</details>

### Ternary operator

Use it for simple expressions, not multi-step logic.

```js
const label = isAdmin ? "Admin" : "User";
```

## Optional chaining and nullish coalescing

These operators help avoid `Cannot read properties of undefined` bugs.

{% tabs %}
{% tab title="JavaScript" %}
\`\`\`js const email = user?.profile?.email; const displayName = user?.name ?? "Anonymous";

````

`??` uses **nullish** rules (only `null`/`undefined` count as missing).</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>```ts
type User = { name?: string; profile?: { email?: string } };

function getEmail(u: User): string {
  return u.profile?.email ?? "";
}
````

TS helps ensure you handle “possibly undefined” paths.
{% endtab %}
{% endtabs %}

\### `||` vs `??` (important)

`||` treats many values as falsy (including `0` and `""`). `??` only treats `null`/`undefined` as missing.

<details>

<summary>Show || vs ??</summary>

```js
0 || 10; // 10 (maybe not what you want)
0 ?? 10; // 0

"" || "default"; // "default"
"" ?? "default"; // ""
```

</details>

## Loops

* `for` / `while`: traditional loops
* `for...of`: values of an iterable (arrays, strings)
* `for...in`: keys of an object (including inherited enumerable keys)

<details>

<summary>Show for...of vs for...in</summary>

\`\`\`js const arr = \["a", "b", "c"];for (const value of arr) { console.log(value); // a b c }for (const key in arr) { console.log(key); // "0" "1" "2" } \</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>ts const arr: string\[] = \["a", "b", "c"];for (const value of arr) { console.log(value); }for (const key in arr) { // key is a string console.log(key); }### Iteration methods (map/filter/reduce)Arrays have higher-level methods that are often more readable than manual loops.\<details>\<summary>Show map/filter/reduce patterns\</summary>\<div data-gb-custom-block data-tag="tabs">\<div data-gb-custom-block data-tag="tab" data-title='JavaScript'>\`\`\`jsconst nums = \[1, 2, 3, 4];const squares = nums.map((n) => n \* n);const evens = nums.filter((n) => n % 2 === 0);const sum = nums.reduce((acc, n) => acc + n, 0);\`\`\`\</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>\`\`\`tsconst nums: number\[] = \[1, 2, 3, 4];const squares = nums.map((n) => n \* n);const evens = nums.filter((n) => n % 2 === 0);const sum = nums.reduce((acc, n) => acc + n, 0);TS usually infers types here, which makes these patterns safer.

</details>

### `forEach` pitfalls

`forEach` cannot be `break`/`continue`-d, and it does not work well with `await`.

<details>

<summary>Show why forEach + await is a foot-gun</summary>

```js
// This does not await each callback the way many people expect:
[1, 2, 3].forEach(async (n) => {
  await doWork(n);
});

// Prefer:
for (const n of [1, 2, 3]) {
  await doWork(n);
}
```

</details>

## Destructuring

Destructuring is a common JS/TS idiom for extracting values.

{% tabs %}
{% tab title="JavaScript" %}
\`\`\`js const user = { id: 1, name: "Alice" }; const { id, name } = user;

const arr = \[10, 20]; const \[x, y] = arr; `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts type User = { id: number; name: string }; const user: User = { id: 1, name: "Alice" };

const { id, name } = user;

````</div></div>

## switch and exhaustive handling

In JS, `switch` is runtime only.
In TS, `switch` can become safer with discriminated unions.

<details>
<summary>Show discriminated unions + switch exhaustiveness</summary>

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "square":
      return s.side ** 2;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}
````

### Truthiness pitfalls (important)

JavaScript has truthy/falsy values:

* falsy: `false`, `0`, `""`, `null`, `undefined`, `NaN`
* truthy: everything else

TypeScript can help by preventing some “possibly undefined” cases.

### Summary

* JS and TS share the same control-flow runtime behavior.
* Prefer `let`/`const` over `var`; know block scope and TDZ.
* Prefer `===` over `==`; learn `NaN` and `Object.is` edge cases.
* Use optional chaining (`?.`) and nullish coalescing (`??`) to avoid undefined access.
* Prefer `for...of` for readable iteration and correct `await` behavior.

### Important Keywords

#### **Block scope**

Variables are visible only within the nearest `{ ... }` block.

#### **Hoisting**

JS behavior where declarations are processed before execution (var/function declarations), often causing surprises.

#### **Temporal Dead Zone (TDZ)**

Period where `let`/`const` exist but cannot be accessed before initialization.

#### **Coercion**

Implicit conversion between types (often involved in `==`).

#### **Truthiness**

JS rule where some values count as false in conditionals (`0`, `""`, `null`, `undefined`, `NaN`).

#### **Optional chaining (`?.`)**

Safely access nested properties without throwing if a link is nullish.

#### **Nullish coalescing (`??`)**

Fallback operator that treats only `null`/`undefined` as missing.
{% endtab %}
{% endtabs %}
