# TypeScript vs Flow

Both TypeScript and Flow are very similar products and they share most of their syntax with some important differences.
In this document I've tried to compile the list of differences and similarities between Flowtype and TypeScript -- specifically the syntax, usage and usability.

## Disclaimer

*This document might be incomplete and/or contain mistakes and was last updated to describe **TypeScript 3.0** and **Flow 0.65**.*

*I'm maintaining it in my spare time, so if you find mistakes, or learn about latest additions to either project, please help keep this repo up-to-date by contributing and [editing this page](https://github.com/niieani/typescript-vs-flowtype/edit/master/README.md).*

*Thanks!*

# Differences in usage and usability

|   | TypeScript            | Flow |
|---|------------------|--------|
| Leading Design Goal / North Star | identify errors in programs through [a balance between correctness and productivity](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals) | enforce type soundness / safety |
| IDE integrations | top-notch | sketchy, must save file to run type-check; some IDEs have workarounds to run real-time |
| type-checking speed (w/o transpilation, *subjective, need benchmarks!*) | speed does not degrade much as the project grows | [speed degrades with each additional file <img width="439" title="numberOfFiles · O( (LoCperFile + SizeOfTypesOfExports) ^ k )" alt="numberOfFiles · O( (LoCperFile + SizeOfTypesOfExports) ^ k )" src="https://user-images.githubusercontent.com/563469/31196357-c02ca3e4-a94d-11e7-9292-f92e10091ac7.png"> ](https://youtu.be/WgvAPzOmiP0?t=616) |
| autocomplete | <ul><li>both during declaration and usage</li><li>feels instantaneous</li><li>feels reliable</li></ul> | <ul><li>[only for usage](https://github.com/facebook/flow/issues/3074)</li><li>feels sluggish (often a second or more of delay)</li><li>feels unreliable (sometimes does not show up at all)</li></ul> |
| expressiveness | great (since TS @ 2.1) | great |
| type safety | very good (7 / 10) | great (8 / 10) |
| specifying generic parameters during call-time | yes [e.g.](http://www.typescriptlang.org/play/#src=function%20someFactory%3CT%3E()%20%7B%0D%0A%20%20return%20class%20%7B%0D%0A%20%20%20%20method(someParam%20%3A%20T)%20%7B%0D%0A%20%20%20%20%20%20%0D%0A%20%20%20%20%7D%0D%0A%20%20%7D%0D%0A%7D%0D%0A%0D%0A%2F%2F%20how%20to%20invoke%20this%20factory%20with%20a%20defined%20%3CT%3E%3F%0D%0A%0D%0Aconst%20SomeClass%20%3D%20someFactory%3C%7B%20whatever%3A%20string%20%7D%3E()%0D%0Aconst%20someInstance%20%3D%20new%20SomeClass()%0D%0AsomeInstance.method('will-error-here')%0D%0A) | no |
| specifying generic parameters for type definitions | yes | yes |
| typings for public libraries | plenty of well maintained typings | a handful of mostly incomplete typings |
| unique features | <ul><li>autocomplete for object construction</li><li>declarable `this` in functions (typing `someFunction.bind()`)</li><li>large library of typings</li><li>more flexible [type mapping via iteration](https://github.com/Microsoft/TypeScript/pull/12114)</li><li>namespacing</li></ul> | <ul><li>variance</li><li>existential types `*`</li><li>testing potential code-paths when types not declared for maximum inference</li><li>`$Diff<A, B>` type</li></ul> |
| type spread operator | [work in progress](https://github.com/Microsoft/TypeScript/pull/13470) | [shipped](https://github.com/facebook/flow/commit/ad443dc92879ae21705d4c61b942ba2f8ad61e4d) >=0.42 |
| userland plugins | [basic](https://github.com/Microsoft/TypeScript/issues/6508), not effecting emitting yet (planned) | no |
| programmatic hooking | architecture prepared, work in progress | work in progress |
| documentation and resources | <ul><li>very good docs</li><li>many books</li><li>videos</li><li>e-learning resources</li></ul> | <ul><li>incomplete, often vague docs</li><ul> |
| error quality | good | good in some, vague in other cases |
| transparency | meeting notes, leadership reasoning and roadmap happens mostly publicly | low transparency, roadmap developed behind closed doors |
| commercial support | no | no |

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

https://flow.org/blog/2015/03/12/Bounded-Polymorphism/

## maybe & nullable type

### Flow

```js
let a: ?string

// equivalent to:

let a: string | null | void
```

### TypeScript

```ts
let a: string | null | undefined
```

Optional parameters [implicitly add `undefined`](https://github.com/Microsoft/TypeScript/issues/13195):

```ts
function f(x?: number) { }
// is semantically the same as:
function f(x: number | undefined) { }
// and also same as (the `| undefined` is redundant):
function f(x?: number | undefined) { }
```

Optional properties implicitly add `undefined`

```ts
class A {
  foo?: string;
}
```

## type casting

### Flow

```js
(1 + 1 : number);
```

### TypeScript

```ts
(1 + 1) as number;

// OR (old version, not recommended):

<number> (1 + 1);
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
- https://flow.org/en/docs/types/modules/

## Exact/Partial Object Types

By default objects in Flow are not exact (can contain more properties than declared), whereas in TypeScript they are always exact (must contain only declared properties).

### Flow

When using flow, `{ name: string }` only means “an object with **at least** a name property”.

```js
type ExactUser = {| name: string, age: number |};
type User = { name: string, age: number };
type OptionalUser = $Shape<User>; // all properties become optional
```

### TypeScript

TypeScript is more strict here, in that if you want to use a property which is not declared, you must explicitly say so by defining the indexed property. It is possible to use [dotted syntax](https://github.com/Microsoft/TypeScript/pull/12671) to access indexed properties since TypeScript 2.2. This is mostly a design decision as it forces you to write the typings upfront.

```js
type ExactUser = { name: string, age: number };
type User = { name: string, age: number, [otherProperty: string]: any };
type OptionalUser = Partial<ExactUser>; // all properties become optional
```

### Reference

- https://flow.org/en/docs/types/objects/
- https://github.com/Microsoft/TypeScript/issues/2710

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

Works the same in both cases, however Flow has an additional syntax to directly import a `typeof`:

### Flow

```js
import typeof {jimiguitar as GuitarT} from "./User";

// OR (below also works in TypeScript)

import {jimiguitar} from "./User.js";
type GuitarT = typeof jimiguitar;
```

### TypeScript

```ts
import {jimiguitar} from "./User";
type GuitarT = typeof jimiguitar;
```

## Restrictive type

When you don't know a type, commonly you would use `any` type. A restrictive type accepts anything, like `any` but in order to use that variable you must ensure values type by refining it.

### Flow

`mixed`

```js
function stringifyNum(num: number) {
  // Do stuff
}

function stringify(value: mixed) {
  if (typeof value === 'string') {
    return '' + value; // Works!
  }
  if (typeof value === 'number') {
    return stringifyNum(value); // Works!
  }
  return '';
}
```

Reference: https://flow.org/en/docs/types/mixed/

### Typescript

`unknown`

```ts
function stringifyNum(num: number) {
  // Do stuff
}

function stringify(value: unknown) {
  if (typeof value === 'string') {
    return '' + value; // Works!
  }
  if (typeof value === 'number') {
    return stringifyNum(value); // Works!
  }
  return '';
}
```

Reference: https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#new-unknown-top-type

## Accessing the type of a Class

Classes are typed, so you don't need to define an explicit type for them.
If you want to reference the type, you can do it the following way:

### Flow

```js
class Test {};
type TestType = Class<Test>;
// This should be equivalent to (if you can confirm, please send a PR):
type TestType = typeof Test;
```

### TypeScript

```ts
class Test {};
type TestType = typeof Test;
```

## Keys/Props Of Type

### Flow

```js
var props = {
  foo: 1,
  bar: 'two',
  baz: 'three',
}

type PropsType = typeof props;
type KeysOfProps = $Enum<PropsType>;

function getProp<T>(key: KeysOfProps): T {
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
type KeysOfProps = keyof PropsType;

function getProp<T>(key: KeysOfProps): T {
  return props[key]
}
```

## Records

### Flow

```js
type $Record<T, U> = {[key: $Enum<T>]: U}
type SomeRecord = $Record<{ a: number }, string>
```

### TypeScript

```ts
type SomeRecord = Record<{ a: number }, string>
```

## Lookup Types

### Flow

```js
type A = {
  thing: string
}

// when the property is a string constant use $PropertyType (i.e. you know it when typing)
type lookedUpThing = $PropertyType<A, 'thing'>

// when you want the property to be dynamic use $ElementType (since Flow 0.49)
function getProperty<T : Object, Key : string>(obj: T, key: Key): $ElementType<T, Key> {
    return obj[key];
}
```

Reference:
- https://github.com/facebook/flow/pull/2952#issuecomment-308248000
- https://github.com/facebook/flow/commit/968210c5887b5bdd47d17167300033d1e1077d1a
- https://github.com/facebook/flow/issues/2464#issuecomment-308536540
- [flow/try](https://flow.org/try/#0MYGwhgzhAEBKCmwD2AnAJgHgLLwLYCN4UIAuaAeXwCtEAXAPmgG8AoaaNMWsMnAoiAG4WbaMgB2EWigCuwWqgAUnbrzyFiASmaj2tABYBLCADoVYaAF4OXMMPYBfUaIDm8WhgDS8AJ5kpKIbiLvSKANZk3j6aZAAkAKIgePDitAAqPgAO8NjqAgA00FGMrOxl0CjuMiji0AbGZrYA2mEAuvbQTk4itFnw0GlGwVY67DzQ4jL8KPmi+P7SQS6zThJSYkiS0nIKKABiYIYg1fBkCMjoGINLjNbi8ADucIioaIpM4wCMDtoA9L-QIgoVDQFhrWgVM4vS7XYK3CaPZ4XN4fMifQrzaAAcgAZkgkFifsIWIoUCY3LRFFiwFiYtAAktNIJoP9AShgSgSWSKVSaXTJtMmVzye4qfhaWQBRomSyAUDUMKeVjxXSGcEZSIcTJxPJDJtoBSAArA7IoXpXMiUGjyQpRBaBOGKJDUMhpQoRIq+OkJJK4FLpPpXW2+Eq6CpVGrQZ1UFrtFjdRRGk1EXrvTanaDfQpY9MSiZTaVAA)

### TypeScript

Arguably, it's a bit easier to type both cases in TS, since they follow the same pattern.

```ts
type A = {
  thing: string
}

type lookedUpThing = A['thing']

// and...

function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];  // Inferred type is T[K]
}

function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]) {
    obj[key] = value;
}
```

Reference:
- https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
- https://github.com/facebook/flow/issues/2310

## Type-narrowing functions

### Flow

Note: undocumented syntax, may change:

```js
function isNil(value: mixed): boolean %checks {
  return value == null;
}

const thing = null;

if (!isNil(thing)) {
  const another = thing.something;
}
```

Reference:
- [flow/try](https://flow.org/try/#0PQKgBAAgZgNg9gdzCYAoVxhngEwIYDOAFsAJYEBypMYUArgHYDGALqXA6vc2x2OVRgAKAG54YdAKYAuMAFtSAD0k4AlLIBGcODEl4GYAKRMikpgGsCYAN6owYAE6SWdBwbETJYALzewDOhgYAG5UAF90Jg4CFjAWIlIGAHMff0CQ9FIoMCEAQgFqIXjEpNVVGzswKIYYsH04eMkHVOLkgDoCODlnBOTQiKA)

### TypeScript

Type-narrowing functions are called type guard functions in TypeScript.

```ts
function isNil<T>(value: T | null): value is null {
  return value == null;
}

const thing: any = null;

if (!isNil(thing)) {
  const another = thing.something;
}
```

## Getting the type of a function call return value

### Flow

`$Call` utility type:

```js
type Fn1 = <T>(T) => T;
type E = $Call<Fn1, number>;

declare var e: E; // E is number
(42: E); // OK
```

Reference: https://github.com/facebook/flow/commit/ac7d9ac68acc555973d495f0a3f1f97758eeedb4

### TypeScript

`ReturnType` utility type:

```ts
type fn1<T> = (a: T) => T;

type E = ReturnType<fn1<number>>;

var e: E; // E is number
```

## Mapped Types / Foreach Property

### Flow

```js
type InputType = { hello: string };
type MappedType = $ObjMap<InputType, ()=>number>;
```

Reference:

- https://gist.github.com/gabro/bb83ed574690645053b815da2082b937
- https://twitter.com/andreypopp/status/782192355206135808

### TypeScript

A bit more flexibility here, as you have access to each individual key name and can combine with Lookup types and even do simple transformations.

```ts
type InputType = { hello: string };
type MappedType = {
  [P in keyof InputType]: number;
};
```

## Function and method overloading

### Flow

It is possible to declare multiple signatures for the same method (also called: overloading). This feature is undocumented, and only available in type declarations (`.js.flow` files or module statements), not inline/alongside your code.

```js
declare function add(x: string, y: string): string;
declare function add(x: number, y: number): number;

declare class Adder {
  add(x: string, y: string): string;
  add(x: number, y: number): number;
}
```

However, it's possible to create function overloads inline for functions outside of classes, by using additional declarations.

```js
declare function add(x: string, y: string): string;
declare function add(x: number, y: number): number;
function add(x, y) {
  return x + y;
}

add(1, 1); // Ok
add("1", "1"); // Ok
add(1, "1"); // Error
```

### TypeScript

TypeScript supports both function and method overloading, in both: type definitions (`.d.ts`) and inline alongside code.

```ts
class Adder {
  add(x: string, y: string): string;
  add(x: number, y: number): number;
  add(x, y) {
    return x + y;
  }
}


function add(x: string, y: string): string;
function add(x: number, y: number): number;
function add(x, y) {
  return x + y;
}
```

## Read-only Types

### Flow

```js
type A = {
  +b: string
}

let a: A = { b: 'something' }
a.b = 'something-else'; // ERROR
```

### TypeScript

```ts
type A = {
  readonly b: string
}

let a: A = { b: 'something' }
a.b = 'something-else'; // ERROR
```

One caveat that makes TypeScript's `readonly` less safe is that the same `non-readonly` property in a type is compatible with a `readonly` property. This essentially means that you can pass an object with `readonly` properties to a function which expects non-readonly properties and TypeScript will *not* throw errors: [example](https://www.typescriptlang.org/play/index.html#src=%0D%0Afunction%20test(x%3A%20%7B%20foo%3A%20string%20%7D)%20%7B%20%0D%0A%20%20%20%20x.foo%20%3D%20'bar'%3B%0D%0A%7D%0D%0A%0D%0Aconst%20x%3A%20%7B%20readonly%20foo%3A%20string%20%7D%20%3D%20%7B%20foo%3A%20'baz'%20%7D%3B%0D%0A%0D%0Atest(x)%3B).

## "Impossible flow" type

### Flow

`empty`

```js
function returnsImpossible() {
  throw new Error();
}

// type of returnsImpossible() is 'empty'
```

### TypeScript

`never`

```ts
function returnsImpossible() {
  throw new Error();
}

// type of returnsImpossible() is 'never'
```

## Difference types

### Flow

```js
type C = $Diff<{ a: string, b: number }, { a: string }>
// C is { b: number}
```

Note however that $Diff is not an official feature. 

> It only works properly as lower bound, i.e. you can assign something to it, but can't use it after that.

([source](https://github.com/facebook/flow/issues/3541#issuecomment-289291932))

### Typescript

You can define your own filter type, but it does not have a helper type for that.

```ts
class A {
  a: string;
  b: number;
}

class B {
  a: string;
  c: boolean;
}

type Omit<T, U> = Pick<T, Exclude<keyof T, keyof U>>;
//  

type C = Omit<A, B>;
// C is { b: number }
```

However, Flow implementation is stricter in this case, as B have a property that A does not have, it would rise an error. In Typescript, however, they would be ignored. 

# Same syntax

Most of the syntax of Flow and TypeScript is the same. TypeScript is more expressive for certain use-cases (advanced mapped types with keysof, readonly properties), and Flow is more expressive for others (e.g. `$Diff`).

## optional parameters

### Flow and TypeScript

```js
function(a?: string) {}
```

# TypeScript-only concepts

## call-time generic parameters

In TypeScript, you can create more complex behaviors, like this:

```ts
function makeTgenerator<T>() {
  return function(next: () => T) {
    const something = next();
    return something;
  }
}

const usage = makeTgenerator<string>()
// 'usage' is of type: (next: () => string) => string
```

#### Flow

In Flow it is possible to define generic functions similarly to the above example, but only if one of the parameters or its return type is inferrable to the desired generic type, i.e. you cannot call any method/constructor using a custom `T`.

## Declarable arbitrary `this` in functions (outside of objects)

```ts
function something(this: { hello: string }, firstArg: string) {
  return this.hello + firstArg;
}
```

## Private and Public properties in classes

```ts
class SomeClass {
  constructor(public prop: string, private prop2: string) {
    // transpiles to:
    // this.prop = prop;
    // this.prop2 = prop2;
  }
  private prop3: string;
}
```

## [Non-null assertion operator](https://github.com/Microsoft/TypeScript/pull/7140)

Add `!` to signify we know an object is non-null.

```ts
// Compiled with --strictNullChecks
function validateEntity(e?: Entity) {
  // Throw exception if e is null or invalid entity
}

function processEntity(e?: Entity) {
  validateEntity(e);
  let s = e!.name;  // Assert that e is non-null and access name
}
```

## Conditional Typing

```ts
type XorY<T, U> = T extends U ? X : Y;
```

This alone, introduces new helper types, or types aliases.

```ts
type Exclude<T, U> = T extends U ? never : T;

/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;

/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;

/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any[]) => any> =
    T extends (...args: any[]) => infer R ? R : any;
```

## Mapped Type Modifiers

You can use `+` and `-` operators to modify mapped types.

```ts
type Mutable<T> = {
  -readonly [P in keyof T]: T[P]
}

interface Foo {
  readonly abc: number;
}

// 'abc' is no longer read-only.
type TotallyMutableFoo = Mutable<Foo>
```

### Helper type modifiers

`Required` is a type mapper to make all properties of an object to be required.

`Partial` is a type mapper to make all properties of an object to be optional.

`Readonly` is a type mapper to make all properties of an object to be readonly.

# Flow-only concepts

## Inferred existential types

`*` as a type or a generic parameter signifies to the type-checker to infer the type if possible

```js
Array<*>
```

TypeScript has a proposal for an equivalent (needs link).

## Variance

https://flow.org/en/docs/lang/variance/

```js
function getLength(o: {+p: ?string}): number {
  return o.p ? o.p.length : 0;
}
```

[TypeScript proposal](https://github.com/Microsoft/TypeScript/issues/10717)

Bivariance is among [the design decisions](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-function-parameters-bivariant) driving TypeScript.

## Useful References

* https://github.com/Microsoft/TypeScript/issues/1265
* Undocumented Flow modifiers https://github.com/facebook/flow/issues/2464
* http://sitr.us/2015/05/31/advanced-features-in-flow.html
