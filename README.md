# Typescript vs Flow

Differences between Flowtype and TypeScript 2.1 -- syntax, usage and usability

## Disclamer

This might be incomplete and/or contain mistakes.
I'm open to contributions and comments.

# Differences in usage and usability

|   | TypeScript            | Flow |
|---|------------------|--------|
| IDE integrations | top-notch | sketchy, must save file to run type-check; some IDEs do hacky workarounds to run real-time |
| speed | real-time                  | good       |
| autocomplete | both during declaration and usage | only for usage |
| expressiveness | great (since TS @ 2.1) | great |
| type safety | great (TS @ 2.0) | great |
| typings for public libraries | plenty of well mainained typings | a handful of mostly incomplete typings |
| unique features | autocomplete for object construction, huge library of mainained Typings, `readonly` properties, `never` type, namespacing | variance, existential types `*`, testing all potential code-paths when typings not declared for maximum inference, `$Diff<A, B>` type |
| ecosystem flexibility | no extensions as of yet | no extensions as of yet |

# Differences in syntax

## bounded polymorphism

### Flow

```js
function fooGood<T: { x: number }>(obj: T): T {
  console.log(Math.abs(obj.x));
  return obj;
}
```

### TypeScript

```ts
function fooGood<T extends { x: number }>(obj: T): T {
  console.log(Math.abs(obj.x));
  return obj;
}
```

### Reference

https://flowtype.org/blog/2015/03/12/Bounded-Polymorphism.html

## nullable type

### Flow

```js
let a: ?string

// or:

let a: string | null | void
```

### TypeScript

```ts
let a: string?

// or:

let a: string | null | undefined
```

## mapping dynamic module names

### Flow

`.flowconfig`

```ini
[options]
module.name_mapper='^\(.*\)\.css$' -> '<PROJECT_ROOT>/CSSModule.js.flow'
```

`CSSModule.js.flow`

```js
// @flow

// CSS modules have a `className` export which is a string
declare export var className: string;
```

### TypeScript

```ts
declare module "*.css" {
  export const className: string;
}
```

### Reference

- https://www.typescriptlang.org/docs/handbook/modules.html
- https://flowtype.org/docs/modules.html

## Exact/Partial Object Types

By default objects in flow are not strict, whereas in TypeScript they are always strict, unless set as partial.

### Flow

When using flow, `{ name: string }` only means “an object with **at least** a name property”.

```js
type StrictUser = {| name: string, age: number |};
type User = { name: string, age: number };
type PartialUser = $Shape<User>;
```

### TypeScript

```js
type StrictUser = { name: string, age: number };
type PartialUser = Partial<{ name: string, age: number }>;
```

### Reference

- https://flowtype.org/docs/objects.html

## Importing types

### Flow

```js
import type {UserID, User} from "./User.js";
```

### TypeScript

TypeScript does not treat Types in any special way when importing.

```ts
import {UserID, User} from "./User.js";
```

## typeof

Works the same with the addition in flow of directly importing a typeof:

### Flow

```js
import typeof {jimiguitar as GuitarT} from "./User.js";

// OR (below also works in TypeScript)

import {jimiguitar} from "./User.js";
type GuitarT = typeof jimiguitar;
```

### TypeScript

```ts
import {jimiguitar} from "./User.js";
type GuitarT = typeof jimiguitar;
```

## Keys/Props Of Type

### Flow

```js
var props = {
  foo: 1,
  bar: 'two',
  baz: 'three',
}

function getProp<T>(key: $Enum<typeof props>): T {
  return props[key]
}
```

### TypeScript

```ts
var props = {
  foo: 1,
  bar: 'two',
  baz: 'three',
}

type PropsType = typeof props

function getProp<K extends keyof PropsType>(key: K): PropsType[K] {
  return props[key]
}
```

## Mapped Types

TODO: use an example which does the same

### Flow

```js
type ExtractCodomain = <V>(v: () => V) => V;
function f<O>(o: O): $ObjMap<O, ExtractCodomain>;
```

### TypeScript

```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

# Same syntax

Most of the syntax of Flow and TypeScript is the same. TypeScript is more expressive for certain use-cases (advanced mapped types with keysof, readonly properties), and Flow is more expressive for others (e.g. `$Diff`).

## optional parameters

### Flow and TypeScript

```js
function(a?: string) {}
```

## Flow's "mixed" type

Flow's `mixed` type is simply a union of all the basic types (string, number, boolean) without their Object versions.

The TypeScript equivalent should be:

```ts
type mixed = string | number | boolean
```

Reference: https://flowtype.org/docs/quick-reference.html#mixed

# TypeScript-only concepts

## [Non-null assertion operator](https://github.com/Microsoft/TypeScript/pull/7140)

Add `!` to signify we know an object is non-null.

```ts
// Compiled with --strictNullChecks
function validateEntity(e: Entity?) {
    // Throw exception if e is null or invalid entity
}

function processEntity(e: Entity?) {
    validateEntity(e);
    let s = e!.name;  // Assert that e is non-null and access name
}
```

## Lookup Types

```ts
type A = {
  thing: string
}

type lookedUpThing = typeof A['thing']
```

## Read-only Types

```ts
type A = {
  readonly b: string
}

let a: A = { b: 'something' }
a.b = 'something-else'; // ERROR
```

# Flow-only concepts

## Difference types

```js
type C = $Diff<{ a: string, b: number }, { a: string }>
// C is { b: number}
```

TypeScript has a proposal for this (needs link).

## Inferred generics / existential types

`*` as a generic parameter tries to infer the type if possible

```js
Array<*>
```

TypeScript has a proposal for this (needs link).

## Variance

https://flowtype.org/docs/variance.html

```js
function getLength(o: {+p: ?string}): number {
  return o.p ? o.p.length : 0;
}
```

(TypeScript proposal)[https://github.com/Microsoft/TypeScript/issues/10717]

## Useful References

* https://github.com/Microsoft/TypeScript/issues/1265
* Undocumented Flow modifiers https://github.com/facebook/flow/issues/2464
* http://sitr.us/2015/05/31/advanced-features-in-flow.html
