# 9. Errors & Testing

JavaScript error handling is runtime-based. TypeScript helps you reduce certain classes of errors before runtime, but does not replace runtime handling.

## Throwing and catching\`\`\`js

function parseJson(text) { try { return JSON.parse(text); } catch (err) { throw new Error("Invalid JSON"); } } `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts function parseJson(text: string): unknown { try { return JSON.parse(text); } catch { throw new Error("Invalid JSON"); } } \`\`\`### `catch` values can be anything

In JavaScript, you can throw any value (`throw "oops"`, `throw 123`). In practice, teams standardize on throwing `Error` instances.

TypeScript (in modern configs) treats `catch (err)` as `unknown`, forcing you to narrow before using it.

<details>

<summary>Show narrowing in a catch block (TS)</summary>

```ts
try {
  doWork();
} catch (err: unknown) {
  if (err instanceof Error) {
    console.error(err.message);
  } else {
    console.error("Unknown error", err);
  }
}
```

</details>

### Error causes (chaining)

When wrapping an error, keep the original error as context.

<details>

<summary>Show error cause (when supported)</summary>

```js
try {
  JSON.parse("{");
} catch (err) {
  throw new Error("Invalid JSON", { cause: err });
}
```

</details>

## Error types and custom errors\`\`\`js

class HttpError extends Error { constructor(status, message) { super(message); this.status = status; } }

throw new HttpError(404, "Not found"); `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts class HttpError extends Error { status: number;

constructor(status: number, message: string) { super(message); this.status = status; } }

````</div></div>##

TypeScript does not enforce what a function can throw.
This is why many TS codebases use “Result” types for expected failures.

<details>
<summary>Show Result-based error handling</summary>

```ts
type Ok<T> = { ok: true; value: T };
type Err<E> = { ok: false; error: E };
type Result<T, E> = Ok<T> | Err<E>;

function parseIntSafe(text: string): Result<number, "not_a_number"> {
  const n = Number(text);
  if (Number.isNaN(n)) return { ok: false, error: "not_a_number" };
  return { ok: true, value: n };
}
````

### When to prefer Result types

Use Result-style values for **expected** failures where callers should handle the outcome:

* parsing
* validation
* business rules (“not allowed”)

Use exceptions for **unexpected** failures:

* programming bugs
* I/O failures you cannot recover from locally

## Runtime validation (JS and TS)

If you receive external data (JSON), you must validate at runtime.

{% tabs %}
{% tab title="JavaScript" %}
`js function isUser(value) { return value && typeof value.name === "string"; }`
{% endtab %}

{% tab title="TypeScript" %}
\`\`\`ts type User = { name: string };

function isUser(value: unknown): value is User { return !!value && typeof (value as any).name === "string"; }

````</div></div>##

Async failures often show up as rejected Promises. If you forget to handle them, you can get unhandled rejection warnings/crashes.

<details>
<summary>Show Promise rejection handling</summary>

```js
fetch("/api")
  .then((r) => r.json())
  .catch((err) => {
    console.error("request failed", err);
  });
````

<details>

<summary>Show try/catch with async/await</summary>

```js
async function load() {
  try {
    const r = await fetch("/api");
    return await r.json();
  } catch (err) {
    console.error(err);
    return null;
  }
}
```

</details>

### Don’t swallow errors (layering)

Good error handling is often about _where_ you handle errors:

* Lower layers should attach context and rethrow/return a Result.
* Higher layers decide user-facing behavior (retry, fallback, show message).

<details>

<summary>Show adding context then rethrowing</summary>

```js
async function fetchUser(id) {
  try {
    const res = await fetch(`/users/${id}`);
    if (!res.ok) throw new Error("bad status: " + res.status);
    return await res.json();
  } catch (err) {
    throw new Error(`fetchUser failed for id=${id}`, { cause: err });
  }
}
```

</details>

### Retry patterns (when appropriate)

Retries are useful for transient failures (network hiccups), but dangerous for permanent failures.

<details>

<summary>Show a simple retry helper</summary>

```js
async function retry(fn, attempts) {
  let lastErr;
  for (let i = 0; i < attempts; i += 1) {
    try {
      return await fn();
    } catch (err) {
      lastErr = err;
    }
  }
  throw lastErr;
}
```

</details>

### Testing basics

At minimum, tests should verify:

* pure logic functions
* boundary conditions
* error cases

<details>

<summary>Show a tiny test style (framework-agnostic)</summary>

\`\`\`js function add(a, b) { return a + b; }function assertEqual(actual, expected) { if (actual !== expected) { throw new Error(Expected ${expected} but got ${actual}); } }assertEqual(add(2, 3), 5); \</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>ts function add(a: number, b: number): number { return a + b; }function assertEqual(actual: T, expected: T): void { if (actual !== expected) { throw new Error(Expected ${expected} but got ${actual}); } }assertEqual(add(2, 3), 5);\</details>## Testing layers (reference)- Unit tests: fast, pure logic- Integration tests: boundaries (DB, filesystem, HTTP)- End-to-end tests: real user flows## Testing async codeAsync code introduces common mistakes:- tests that forget to \`await\`- flaky timing-based assertions\<details>\<summary>Show an async test pitfall (conceptual)\</summary>\`\`\`js// Bad: test may finish before the promise resolvesdoAsyncWork();// Good: await the work (or return the promise)await doAsyncWork();

</details>

### Mocking vs real dependencies

Mocking is useful, but over-mocking can make tests meaningless.

Practical guideline:

* Mock at the edges (network/time)
* Prefer real logic for the core

### Summary

* JS errors are runtime; you must handle and test them.
* TS reduces type-related errors but does not remove the need for runtime validation.
* Consider Result-style error handling for expected failures.

### Important Keywords

#### **Exception**

A runtime error represented by a thrown value (ideally an `Error`).

#### **Stack trace**

The call history captured when an error occurs.

#### **Unhandled rejection**

A Promise rejection that is never handled with `catch`/`try-catch`.

#### **Type guard**

Runtime check that narrows types (useful in `catch` blocks).

#### **Result type**

Value-based error handling pattern (e.g., `{ ok: true } | { ok: false }`).
{% endtab %}
{% endtabs %}
