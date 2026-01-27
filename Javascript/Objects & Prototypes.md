# Objects, Classes & Prototypes (JavaScript vs TypeScript)

JavaScript is prototype-based.
TypeScript can *type* both class-based and prototype-based patterns.

## Objects and property access

{% tabs %}

{% tab title="JavaScript" %}

```js
const user = { name: "Alice", age: 25 };

console.log(user.name);
console.log(user["age"]);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string; age: number };

const user: User = { name: "Alice", age: 25 };

console.log(user.name);
```

{% endtab %}

{% endtabs %}

### Dot vs bracket access (when to use)

- Dot access (`obj.key`) is for known, valid identifier names.
- Bracket access (`obj[key]`) is for dynamic keys.

<details>
<summary>Show dynamic keys and why they matter</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
const key = "age";
const user = { name: "Alice", age: 25 };
console.log(user[key]);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string; age: number };
const key: keyof User = "age";

const user: User = { name: "Alice", age: 25 };
console.log(user[key]);
```

{% endtab %}

{% endtabs %}

</details>

## Destructuring

{% tabs %}

{% tab title="JavaScript" %}

```js
const user = { name: "Alice", age: 25 };
const { name, age } = user;
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string; age: number };

const user: User = { name: "Alice", age: 25 };
const { name, age } = user;
```

{% endtab %}

{% endtabs %}

## Prototypes (what actually happens)

Every object has an internal prototype chain used for property lookup.

### Property lookup rules (high level)

When you read `obj.prop`, JavaScript does:

1. Check if `obj` itself has `prop`
2. If not, check `Object.getPrototypeOf(obj)` and repeat
3. Stop at `null` (end of chain)

This is why inheriting “works” without copying methods.

### Own properties vs inherited properties

`for...in` iterates enumerable properties, including inherited ones.
For “just my keys”, prefer `Object.keys()` (own enumerable only).

<details>
<summary>Show hasOwn / hasOwnProperty</summary>

```js
const obj = Object.create({ inherited: 1 });
obj.own = 2;

Object.keys(obj); // ["own"]
"inherited" in obj; // true

Object.hasOwn(obj, "own"); // true
Object.hasOwn(obj, "inherited"); // false
```

</details>

<details>
<summary>Show Object.create(null) dictionary gotcha</summary>

```js
const dict = Object.create(null);
dict["a"] = 1;

// dict.hasOwnProperty("a"); // TypeError: hasOwnProperty is not a function
Object.hasOwn(dict, "a"); // true
```

</details>

<details>
<summary>Show prototype inheritance in JS (and how TS types it)</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
const animal = {
  speak() {
    return "...";
  },
};

const dog = Object.create(animal);
dog.speak = function () {
  return "woof";
};

console.log(dog.speak());
console.log(Object.getPrototypeOf(dog) === animal);
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type Animal = { speak: () => string };

const animal: Animal = {
  speak() {
    return "...";
  },
};

const dog = Object.create(animal) as Animal;
dog.speak = () => "woof";

console.log(dog.speak());
```

{% endtab %}

{% endtabs %}

</details>

## Classes (syntax sugar over prototypes)

Classes are mostly syntax over prototypes.

{% tabs %}

{% tab title="JavaScript" %}

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hi ${this.name}`;
  }
}

const p = new Person("Alice");
console.log(p.greet());
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  greet(): string {
    return `Hi ${this.name}`;
  }
}
```

{% endtab %}

{% endtabs %}

### Inheritance (extends) vs composition

`class` supports `extends`, but in many codebases composition is preferred:

- Inheritance couples subclasses to base class details.
- Composition makes dependencies explicit.

<details>
<summary>Show inheritance vs composition (conceptual)</summary>

{% tabs %}

{% tab title="Inheritance" %}

```js
class BaseLogger {
  log(msg) {
    console.log(msg);
  }
}

class AppLogger extends BaseLogger {
  error(msg) {
    this.log("ERROR: " + msg);
  }
}
```

{% endtab %}

{% tab title="Composition" %}

```js
function createLogger(output) {
  return {
    log(msg) {
      output(msg);
    },
    error(msg) {
      output("ERROR: " + msg);
    },
  };
}
```

{% endtab %}

{% endtabs %}

</details>

### `this` in methods (common pitfall)

In JavaScript, `this` depends on how a function is called.
When you pass a method as a callback, you can lose the binding.

<details>
<summary>Show method extraction pitfall</summary>

```js
const user = {
  name: "Alice",
  greet() {
    return "hi " + this.name;
  },
};

const greet = user.greet;
// greet(); // often fails because this is undefined in strict mode
```

</details>

## Public/private fields

JS now has true private fields with `#`.
TS also has `private` which is usually enforced at compile time.

<details>
<summary>Show JS #private vs TS private</summary>

{% tabs %}

{% tab title="JavaScript" %}

```js
class Counter {
  #value = 0;

  inc() {
    this.#value += 1;
    return this.#value;
  }
}

const c = new Counter();
c.inc();
// c.#value; // SyntaxError
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
class Counter {
  private value = 0;

  inc(): number {
    this.value += 1;
    return this.value;
  }
}

const c = new Counter();
c.inc();
// c.value; // compile-time error
```

{% endtab %}

{% endtabs %}

</details>

## Interfaces and structural typing

TypeScript interfaces describe shapes.

```ts
interface Logger {
  log(message: string): void;
}

function run(logger: Logger) {
  logger.log("running");
}

run({ log: (m) => console.log(m) });
```

## Property descriptors, getters, and setters

Properties are not just “fields”. They can be computed via getters/setters.

<details>
<summary>Show getter/setter</summary>

```js
const user = {
  first: "Alice",
  last: "Smith",
  get fullName() {
    return this.first + " " + this.last;
  },
};

console.log(user.fullName);
```

</details>

<details>
<summary>Show defineProperty (descriptor)</summary>

```js
const obj = {};
Object.defineProperty(obj, "id", {
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false,
});

obj.id = 456; // silently fails or throws in strict mode
```

</details>

## Index signatures and records

When your object is a dictionary/map:

```ts
const counts: Record<string, number> = {};
counts["a"] = 1;
```

## Mutability and immutability patterns

JavaScript objects are mutable by default.

Common patterns:

- Treat objects as immutable in application code (create copies)
- Use `Object.freeze` for shallow freezing

<details>
<summary>Show shallow copy update</summary>

```js
const user = { name: "Alice", age: 25 };
const updated = { ...user, age: 26 };
```

</details>

<details>
<summary>Show Object.freeze caveat (shallow)</summary>

```js
const state = Object.freeze({ nested: { count: 0 } });

// state.nested = { count: 1 }; // blocked
state.nested.count = 1; // still possible because nested object is not frozen
```

</details>

## Summary

- JS objects inherit via prototypes.
- Classes are common, but they still use prototype lookup.
- TS lets you model object shapes and catch missing/incorrect properties before runtime.

## Important Keywords

### **Prototype**

Internal link used for property lookup (`Object.getPrototypeOf(obj)`).

### **Prototype chain**

Sequence of prototypes searched during property access.

### **Own property**

Property stored directly on the object (not inherited).

### **Structural typing**

Type compatibility based on shape (TypeScript).

### **Descriptor**

Metadata controlling property behavior (writable/enumerable/configurable, getters/setters).

### **Composition**

Building behavior by combining objects/functions rather than inheriting from base classes.
