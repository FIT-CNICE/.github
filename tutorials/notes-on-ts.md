# Typescript: Fundementals + Intermediate

_Typescript is a statically typed fake language._<!--more-->

<!-- mtoc-start -->

* [1. Type vs. Interface](#1-type-vs-interface)
  * [Open Interfaces: augmenting existing built-in or library types](#open-interfaces-augmenting-existing-built-in-or-library-types)
  * [Choose btw `type` or `interface`](#choose-btw-type-or-interface)
  * [Top types: `any`, `unknown`, `object`, and `{}`](#top-types-any-unknown-object-and-)
  * [Bottom type: `never`](#bottom-type-never)
  * [Nullish values: `null`, `void`, and `undefined`](#nullish-values-null-void-and-undefined)
  * [Non-null assertion: cast away the possibility of a `null`/`undefined`](#non-null-assertion-cast-away-the-possibility-of-a-nullundefined)
  * [Definite assignment assertion: assert smt was initialized](#definite-assignment-assertion-assert-smt-was-initialized)
  * [Optional chaining `?.`](#optional-chaining-)
  * [Nullish coalescing `??`](#nullish-coalescing-)
  * [`extends` and `implements` for interfaces](#extends-and-implements-for-interfaces)
* [2. Conditional & Mapped Types](#2-conditional--mapped-types)
  * [Conditional types](#conditional-types)
  * [Extract and Exclude](#extract-and-exclude)
* [3. Recursive types](#3-recursive-types)
* [4. Type queries](#4-type-queries)
  * [`keyof`](#keyof)
  * [`typeof`](#typeof)
  * [Indexed access types](#indexed-access-types)
* [5. Function and Constructor Types](#5-function-and-constructor-types)
  * [Callable types](#callable-types)
  * [`void`](#void)
  * [Function overloads](#function-overloads)
  * [`this` types](#this-types)
  * [Explicitly define return types](#explicitly-define-return-types)
* [6. Classes](#6-classes)
  * [`public`, `private`, and `protected`](#public-private-and-protected)
  * [JS private `#fields`](#js-private-fields)
  * [Param properties](#param-properties)
  * [Overrides](#overrides)
* [7. Type Guards and Narrowing](#7-type-guards-and-narrowing)
  * [Built-in type guards](#built-in-type-guards)
  * [User-defined type guard](#user-defined-type-guard)
  * [Narrowing with `switch(true)`](#narrowing-with-switchtrue)
  * [`satisfies` keyword](#satisfies-keyword)
* [8. Generics](#8-generics)
  * [Defining type parameter for function](#defining-type-parameter-for-function)
  * [Defining type parameter for interface](#defining-type-parameter-for-interface)
  * [An exercise: map, filter, and reduce for dictionary](#an-exercise-map-filter-and-reduce-for-dictionary)
  * [Generic constraints: “minimum requirement” for a type param](#generic-constraints-minimum-requirement-for-a-type-param)
  * [TypeParams best practice](#typeparams-best-practice)
  * [`infer`: access sub-parts of a large type](#infer-access-sub-parts-of-a-large-type)
  * [`Record`: create object types with specific key-value pairs](#record-create-object-types-with-specific-key-value-pairs)
  * [`Pick`: make new type by picking properties from old type](#pick-make-new-type-by-picking-properties-from-old-type)
  * [Template literal types: create types based on string literal](#template-literal-types-create-types-based-on-string-literal)
  * [Type registry: one handles many return types](#type-registry-one-handles-many-return-types)
* [9. Modules & CJS interop](#9-modules--cjs-interop)
  * [ES Module imports and exports](#es-module-imports-and-exports)
  * [Importing non-TS things](#importing-non-ts-things)

<!-- mtoc-end -->

## 1. Type vs. Interface

Simple **type** definition looks like the following

```javascript
type Amount = {
  currency: string,
  value: number
}
```

You can create type aliases that combine existing types with new behavior by using Intersection (&) types.

```typescript
type SpecialDate = Date & { getDescription(): string };

const newYearsEve: SpecialDate = Object.assign(new Date(), {
  getDescription: () => "Last day of the year",
});
//              ^| newYearsEve.getDescription is now available
```

An **interface** is a way of defining an object type (not to be confused with a type called object, which we’ll discuss later). In this context think of an object type is anything that looks like this.

```typescript
{
  field: value;
}
```

### Open Interfaces: augmenting existing built-in or library types

TypeScript interfaces are “open”, meaning that unlike in type aliases, you can have multiple declarations in the same scope:

```typescript
// @noImplicitAny: false
interface AnimalLike {
  // From before
  eat(food): void;
}
function feed(animal: AnimalLike) {
  animal.eat;
  animal.isAlive;
}

// SECOND DECLARATION OF THE SAME NAME
interface AnimalLike {
  isAlive(): boolean;
}
```

These declarations are merged together to create a result identical to what you would see if both the isAlive and eat methods were on a single interface declaration. This is useful in a situation where you want to add a global property to a predefined object whose type is not easy to find out. Consider the following example:

```typescript
window.document; // an existing property
window.exampleProperty = 42;
// tells TS that `exampleProperty` exists
interface Window {
  exampleProperty: number;
}
```

### Choose btw `type` or `interface`

In many situations, either a type alias or an interface would be perfectly fine, however…

- If you need to define something other than an object type (e.g., use of the `|` union type operator), you must use a type alias
- If you need to define a type to use with the implements heritage term on a class, use an interface
- If you need to allow consumers of your types to augment them, you must use an interface.

### Top types: `any`, `unknown`, `object`, and `{}`

Typescript provides two top types: `any` and `unknown`. You can think of values with an `any` type as “playing by the usual JavaScript rules”. Like any, `unknown` can accept any value that is possible to create in JavaScript.

However, **values with an `unknown` type cannot be used without applying a type guard!** See the following as an example:

```typescript
function doSomethingRisky() {
  if (Math.random() > 0.5) return "ok";
  else if (Math.random() > 0.5) throw new Error("Bad luck!");
  else throw "Really bad luck";
}

try {
  doSomethingRisky();
} catch (e: unknown) {
  if (e instanceof Error) {
    e;
  } else if (typeof e === "string") {
    e;
  } else {
    // Last resort
    console.error(e);
  }
}
```

There’s a compiler flag `useUnknownInCatchVariables` that helps enforce this across a project, without requiring any explicit type annotation.

The `object` type represents the set `{ all possible values except for primitives }`. Primitive value types in JavaScript are `{ string, number, boolean, Symbol, null, undefined, BigInt }`. **DO NOT confuse `object` with the concept of `object type` used to desctibe shapes of `interface`**. The following shows that you can not assign primitives to `object` typed variable:

```typescript
// @errors: 2322
let val: object = { status: "ok" };
val = "foo"; // ERROR
val = null; // ERROR
val = () => "ok"; // OK

// The type of this value cannot be modeled by an interface
let response:
  | { success: string; data: unknown }
  | { error: string; code: number } = { success: "ok", data: [] };

val = response; // OK
```

The empty object type {} represents the set **{ all possible values, except for null and undefined }**. Some examples below:

```typescript
const myObj: {
  a?: number;
  b: string;
} = { b: "foo" };
let nullableString: string | null = null;

let val2: {} = 4; // OK
val2 = "abc"; // OK
val2 = new Date(); // OK
val2 = nullableString; // NOT OK
val2 = myObj.a; // NOT OK
```

You can use the type `{}` in combination with the intersection type operator & to remove nullability from another type:

```typescript
type NullableStringOrNumber = string | number | null | undefined;
type StringOrNumber = NullableStringOrNumber & {};
```

### Bottom type: `never`

Consider the following scenario:

```typescript
class Car {
  drive() {
    console.log("vroom");
  }
}
class Truck {
  tow() {
    console.log("dragging something");
  }
}
type Vehicle = Truck | Car;

let myVehicle: Vehicle = obtainRandomVehicle();

// The exhaustive conditional
if (myVehicle instanceof Truck) {
  myVehicle.tow(); // Truck
} else if (myVehicle instanceof Car) {
  myVehicle.drive(); // Car
} else {
  // This will only work if the type of myVehicle is never.
  const neverValue: never = myVehicle;
}
```

Now, if we change the `Vehicle` type to be `Truck | Car | Boat` then, we will get the error in the `else` bracket:

```typescript
// The exhaustive conditional
if (myVehicle instanceof Truck) {
  myVehicle.tow(); // Truck
} else if (myVehicle instanceof Car) {
  myVehicle.drive(); // Car
} else {
  const neverValue: never = myVehicle;
  // ^Type 'Boat' is not assignable to type 'never'.
}
```

Thus, using `never` to exhaust conditions will help us catch upstream code changes and unexpected values.

### Nullish values: `null`, `void`, and `undefined`

- `null`: there is a value, and that value is nothing.
- `undefined`: the value isn't available
- `void`: should exclusively be used to describe that a function’s return value should be ignored

### Non-null assertion: cast away the possibility of a `null`/`undefined`

```typescript
// suppress warning of 'obj.fruits' is possible 'undefined'
obj.fruits!.push("banana");
```

Non-null assertion is recommended to be used in test suite

### Definite assignment assertion: assert smt was initialized

The definite assignment `!:` assertion is used to suppress TypeScript’s objections about a class field being used, when it can’t be proven1 that it was initialized. See an example below:

```typescript
class ThingWithAsyncSetup {
  setupPromise: Promise<any>; // ignore the <any> for now
  isSetup!: boolean;

  constructor() {
    this.setupPromise = new Promise((resolve) => {
      this.isSetup = false;
      return this.doSetup();
    }).then(() => {
      this.isSetup = true;
    });
  }

  private async doSetup() {}
}
```

### Optional chaining `?.`

Now let’s say we want to render information on a dashboard, for the customer’s most recent payment on any invoice (or leave blank if they haven’t made any payments).

All this, just to sort of drill down and find something if it’s there. Optional chaining gives us a more concise way to do this

```typescript
function getLastPayment(data: ResponseData): number | undefined {
  return data?.customer?.lastInvoice?.lastPayment?.amount;
}
```

### Nullish coalescing `??`

```typescript
type PlayerConfig = {
  volume?: 0 | 25 | 50 | 75 | 100;
};

let smt: PlayerConfig = getConfig();
// vol==50 when smt.volume is null or undefined
const vol = smt.volume ?? 50;
```

### `extends` and `implements` for interfaces

- Just as in in JavaScript, a subclass extends from a base class.
- Additionally a “sub-interface” extends from a base interface, as shown in the example below

```typescript
interface Animal {
  isAlive(): boolean;
}
interface Mammal extends Animal {
  getFurOrHairColor(): string;
}
interface Hamster extends Mammal {
  squeak(): string;
}
function careForHamster(h: Hamster) {
  h.getFurOrHairColor();
  h.squeak();
}
```

A better way to understand `x extends y` is to translate it into "x is subset/subtype of y" or "x is assignable to y". The translation makes the assignment below clearer:

```typescript
type a1 = 64 extends number ? true : false;
// a1=true, 64 is a numeric literal type, which is a specific subtype of number
type a2 = string[] extends any ? true : false;
// true, any is the top type, meaning that any type is assignable to `any`
type a3 = Date extends { new (...args: any[]): any } ? true : false;
// false, Date is JS build-in instance type, not constructor type, while
// {new (..args: any[]): any} is a constructor type
// to get a constructor type, we can use (typeof Date)
type a4 = typeof Date extends { new (...args: any[]): any } ? true : false;
// true, see above for the reason
```

TypeScript adds a second heritage clause that can be used to state that a given class should produce instances that confirm to a given interface: implements.

```typescript
// @noImplicitAny: false
function consumeFood(arg) {}
interface AnimalLike {
  eat(food): void;
}

class Dog implements AnimalLike {
  bark() {
    return "woof";
  }
  eat(food) {
    consumeFood(food);
  }
}
```

## 2. Conditional & Mapped Types

### Conditional types

Conditional types allow for types to be expressed using a very similar (basically, the same) syntax

```typescript
class Grill {
  startGas() {}
  stopGas() {}
}
class Oven {
  setTemperature(degrees: number) {}
}

type CookingDevice<T> = T extends "grill" ? Grill : Oven;

let device1: CookingDevice<"grill">;
//   ^let device1: Grill
let device2: CookingDevice<"oven">;
//   ^let device2: Oven
```

`extends` is a check of a subset relationship: `A extends B?` means "Is A a subset of B?" To verify this thought, see the following:

```typescript
type IsLowNumber<T> = T extends 1 | 2 ? true : false;
type TestOne = IsLowNumber<1>;
//     ^type TestOne=true
type TestTwo = IsLowNumber<2>;
//     ^type TestTwo=true
type TestTen = IsLowNumber<10>;
//     ^type TestTen= false
type TestTenWithTwo = IsLowNumber<10 | 2>;
//     ^type TestTenWithTwo = boolean
```

In the last case,

```typescript
  T = 2 —> { 2 } extends { 1, 2 } —> true
  T = 10 —> { 10 } extends { 1, 2 } —> false
  true | false —> boolean
```

### Extract and Exclude

```typescript
type FavoriteColors =
  | "dark sienna"
  | "van dyke brown"
  | "yellow ochre"
  | "sap green"
  | "titanium white"
  | "phthalo green"
  | "prussian blue"
  | "cadium yellow"
  | [number, number, number]
  | { red: number; green: number; blue: number };

type StringColors = Extract<FavoriteColors, string>;
// type StringColors = "dark sienna" | "van dyke brown" | "yellow ochre" | "sap green" | "titanium white" | "phthalo green" | "prussian blue" | "cadium yellow"

type ObjectColors = Extract<FavoriteColors, { red: number }>;
// type ObjectColors = {
//     red: number;
//     green: number;
//     blue: number;
// }

type TupleColors = Extract<FavoriteColors, [number, number, number]>;
// type TupleColors = [number, number, number]
```

```typescript
// a set of four specific things
type FavoriteColors =
  | "dark sienna"
  | "van dyke brown"
  | "yellow ochre"
  | "sap green"
  | "titanium white"
  | "phthalo green"
  | "prussian blue"
  | "cadium yellow"
  | [number, number, number]
  | { red: number; green: number; blue: number };

type NonStringColors = Exclude<FavoriteColors, string>;
// type NonStringColors = [number, number, number] | {
//     red: number;
//     green: number;
//     blue: number;
// }
```

## 3. Recursive types

Recursive types, are self-referential, and are often used to describe infinitely nestable types. For example, consider infinitely nestable arrays of numbers

```typescript
type NestedNumbers = number | NestedNumbers[];
const val: NestedNumbers = [3, 4, [5, 6, [7], 59], 221];
```

as a more complex example, we use recursive types to define possible values in JSON type:

```typescript
type JSONValue =
  | number
  | string
  | boolean
  | null
  | JSONValue[]
  | { [k: string]: JSONValue };

//! DO NOT EDIT ANY CODE BELOW THIS LINE
function isJSON(arg: JSONValue) {}

//✔️ POSITIVE test cases (must pass)
isJSON("hello"); //✔️ Strings
isJSON([4, 8, 15, 16, 23, 42]); //✔️ Arrays of numbers
isJSON({ greeting: "hello" }); //✔️ Objects
isJSON(false); //✔️ Boolean values
isJSON(true);
isJSON(null); //✔️ null values
isJSON({ a: { b: [2, 3, "foo", null, false] } }); //✔️ A complex object

//! NEGATIVE test cases (must fail)
// @ts-expect-error
isJSON(() => ""); //! Functions are not valid JSON
// @ts-expect-error
isJSON(class {}); //! Classes are not valid JSON
// @ts-expect-error
isJSON(undefined); //! undefined is not valid JSON
// @ts-expect-error
isJSON(BigInt(143)); //! BigInts are not valid JSON
```

## 4. Type queries

### `keyof`

The keyof type query allows us to obtain type representing all property keys on a given interface. Because not all keys are strings, so we can separate out those keys that are symbols and those that are strings using the intersection operator (&). You can think of & in this case as the intersection operator. we’re left only with the sub-part of the keyof Date that also is included by string or symbol, respectively.

```typescript
type DatePropertyNames = keyof Date;

type DateStringPropertyNames = DatePropertyNames & string;
type DateSymbolPropertyNames = DatePropertyNames & symbol;
```

### `typeof`

The `typeof` type query allows you to extract a type from a value. A common use of `typeof` is to obtain a type representing the “static site” of a class (meaning: constructor, static properties, and other things not present on an instance of the class)

### Indexed access types

At the simplest level, these kinds of types are all about accessing some part of another type, via an index:

```typescript
interface Car {
  make: string;
  model: string;
  year: number;
  color: {
    red: string;
    green: string;
    blue: string;
  };
}

let carColor: Car["color"];
let carProperty: Car["color" | "year"];
```

## 5. Function and Constructor Types

### Callable types

Both type alias and interfaces offer the capability to describe **call signatures**:

```typescript
interface TwoNumberCalculation {
  (x: number, y: number): number;
}

type TwoNumberCalc = (x: number, y: number) => number;

const add: TwoNumberCalculation = (a, b) => a + b;
const subtract: TwoNumberCalc = (x, y) => x - y;
```

Let’s pause for a minute to note:

- The return type for an interface is `:number`, and for the type alias it’s `=> number`
- Because we provide types for the functions add and subtract, we don’t need to provide type annotations for each individual function’s argument list or return type

### `void`

The return value of a `void`-returning function is intended to be ignored. We could type functions as returning `undefined`, but there are some interesting differences that highlight the reason for void’s existence:

```typescript
function invokeInFourSeconds(callback: () => undefined) {
  setTimeout(callback, 4000);
}
function invokeInFiveSeconds(callback: () => void) {
  setTimeout(callback, 5000);
}
const values: number[] = [];
// invalid, gives error: Type 'number' is not assignable to type 'undefined'.
invokeInFourSeconds(() => values.push(4));
// It happens that Array.prototype.push returns a number, so ()=> void is better here
invokeInFiveSeconds(() => values.push(4));
```

### Function overloads

In the example below, the third paragraph implements function `handleMainEvent`, and the first two paragraphs are overloads for binding `Form` with `Submit` and `Frame` with `Message`. Note that **the type union is required for the implementation**, you can get `function typeincompatible` if you don't have the unions.

```ts
function handleMainEvent(elem: Form, handler: Submit);
function handleMainEvent(elem: Frame, handler: Message);
// implementation function signature must be general enough to include
// everything that’s possible through the exposed first and second function heads.
function handleMainEvent(elem: Form | Frame, handler: Submit | Message) {}
```

### `this` types

When dealing with callback functions tied to DOM elements, it's a better practice to include `this` as the first argument of the callback. Take the snippet below as an example:

```ts
function myClickHandler(this: HTMLButtonElement, event: Event) {
  this.disabled = true;
}

const myButton = document.getElementsByTagName("button")[0];
const boundHandler = myClickHandler.bind(myButton);
boundHandler(new Event("click")); // bound version: ok
myClickHandler.call(myButton, new Event("click")); // also ok
```

### Explicitly define return types

Consider the following two implementations of the `getData` function:

```ts
// ts is happy with this
export async function getData(url: string) {
  const resp = await fetch(url);
  const data = (await resp.json()) as {
    properties: string[];
  };
  return data;
}

// ts is unhappy with this
async function getData(url: string) {
  const resp = await fetch(url);
  if (resp.ok) {
    const data = (await resp.json()) as {
      properties: string[];
    };
    return data;
  }
}
```

TS will not be happy with the second implementation because the return type could be `undefined`. If we use the same example, but define a return type explicitly, the error message is surfaced at the declaration site:

```ts
async function getData(
  url: string,
  // (2366): Function lacks ending return statement and
  // return type does not include 'undefined'.
): Promise<{ properties: string[] }> {
  const resp = await fetch(url);
  if (resp.ok) {
    const data = (await resp.json()) as {
      properties: string[];
    };
    return data;
  }
}
```

## 6. Classes

### `public`, `private`, and `protected`

| keyword   | who can access (instance field/method)                          |
| :-------- | :-------------------------------------------------------------- |
| public    | Anyone who has access to the scope in which the instance exists |
| protected | the instance itself, and subclasses                             |
| private   | only the instance itself                                        |

| keyword   | who can access (static field/method)                         |
| :-------- | :----------------------------------------------------------- |
| public    | Anyone who has access to the scope in which the class exists |
| protected | static and instance scopes of the class and its subclasses   |
| private   | static scope instance scopes of the class only               |

Note that `private` in TS does not provide any security benefits. A checkpoint in browser can still expose its value.

### JS private `#fields`

TS(>=3.8) supports use of ECMAScript private class fields. The class below has the JS private field: `#serialNumber`:

```ts
class Car {
  private static nextSerialNumber: number;
  private static generateSerialNumber() {
    return this.nextSerialNumber++;
  }

  make: string;
  model: string;
  year: number;
  #serialNumber = Car.generateSerialNumber();

  constructor(make: string, model: string, year: number) {
    this.make = make;
    this.model = model;
    this.year = year;
  }
}
```

Unlike TypeScript’s private keyword, these are truly private fields, which cannot be easily accessed at runtime. It’s important to remember, particularly if you’re writing client side code, that there are still ways of accessing private field data through things like the Chrome Dev Tools protocol. Use this as an encapsulation tool, not as a security construct.

### Param properties

Repeatitive definition of `make`, `model`, and `year` in the `Car` class can be optimized through the use of **param properties**:

```ts
class Car {
  constructor(
    public make: string,
    public model: string,
    public year: number,
  ) {}
}
```

### Overrides

A common mistake, that has historically been difficult for TypeScript to assist with is typos when overriding a class method, see the following example:

```ts
class Car {
  honk() {
    console.log("beep");
  }
}

class Truck extends Car {
  hoonk() {
    // OOPS! you meant to override, but endup adding a new method
    console.log("BEEP");
  }
}

const t = new Truck();
t.honk(); // "beep"
```

TypeScript 5 includes an override keyword that makes this easier to spot:

```ts
// @errors: 4117
class Car {
  honk() {
    console.log("beep");
  }
}

class Truck extends Car {
  override hoonk() {
    // Error: This member cannot have an 'override' modifier because it is not declared in the base class 'Car'. Did you mean 'honk'?
    console.log("BEEP");
  }
}

const t = new Truck();
t.honk(); // "beep"
```

## 7. Type Guards and Narrowing

### Built-in type guards

Below is an illustrative example of a wide variety of built-in types:

```ts
let value:
  | Date
  | null
  | undefined
  | "pineapple"
  | [number]
  | { dateRange: [Date, Date] };

// instanceof
if (value instanceof Date) {
  value;
}
// typeof
else if (typeof value === "string") {
  value;
}
// Specific value check
else if (value === null) {
  value;
}
// Truthy/falsy check
else if (!value) {
  value;
  // let value: undefined
}
// Some built-in functions
else if (Array.isArray(value)) {
  value;
}
// Property presence check
else if ("dateRange" in value) {
  value;
} else {
  value;
  // let value: never
}
```

### User-defined type guard

- Use `is` for type guarding:

```ts
interface CarLike {
  make: string;
  model: string;
}

let maybeCar: any;

// the guard
function isCarLike(valueToTest: any): valueToTest is CarLike {
  return (
    valueToTest &&
    typeof valueToTest === "object" &&
    "make" in valueToTest &&
    typeof valueToTest["make"] === "string" &&
    "model" in valueToTest &&
    typeof valueToTest["model"] === "string"
  );
}

// using the guard
if (isCarLike(maybeCar)) {
  maybeCar;
  // let maybeCar: CarLike
}
```

- Use `assert value is Class`:

```ts
interface CarLike {
  make: string;
  model: string;
}

let maybeCar: any;

// the guard
function assertsIsCarLike(valueToTest: any): asserts valueToTest is CarLike {
  if (
    !(
      valueToTest &&
      typeof valueToTest === "object" &&
      "make" in valueToTest &&
      typeof valueToTest["make"] === "string" &&
      "model" in valueToTest &&
      typeof valueToTest["model"] === "string"
    )
  )
    throw new Error(`Value does not appear to be a CarLike${valueToTest}`);
}

assertIsCarLike(maybeCar);
maybeCar;
// let maybeCar: CarLike
```

- Use with private `#field` presence checks

```ts
class Car {
  static #nextSerialNumber: number = 100;
  static #generateSerialNumber() {
    return this.#nextSerialNumber++;
  }

  #serialNumber = Car.#generateSerialNumber();

  static isCar(other: any): other is Car {
    if (
      other && // is it truthy
      typeof other === "object" && // and an object
      #serialNumber in other
    ) {
      // and we can find a private field that we can access from here
      // then it *must* be a car
      other;
      // ^? (parameter) other: Car
      return true;
    }
    return false;
  }
}

let val: any;
if (Car.isCar(val)) {
  val;
  // ^? let val: Car
}
```

### Narrowing with `switch(true)`

TypeScript 5.3 introduced the ability to use switch(true) for narrowing:

```ts
class Fish {
  swim(): void {}
}
class Bird {
  fly(): void {}
}

let val = {} as any;
switch (true) {
  case val instanceof Bird:
    val.fly();
    break;
  case val instanceof Fish:
    val.swim();
    break;
}
```

### `satisfies` keyword

How can we make sure that `usHolidays` conforms to the type `Holidays`? We could use a type annotation:

```ts
type DateLike = Date | number | string;

type Holidays = {
  [k: string]: DateLike;
};

const usHolidays: Holidays = {
  independenceDay: "July 4, 2024",
  memorialDay: new Date("May 27, 2024"),
  laborDay: 1725260400000, // September 2, 2024
};
```

\*\*but now, we have to treat `memorialDay`, for example, as `Date | number | string`. To prevent this, we can use `satisfies`:

```ts
const usHolidays = {
  independenceDay: "July 4, 2024",
  memorialDay: new Date("May 27, 2024"),
  laborDay: 1725260400000, // September 2, 2024
};
```

It’s important to remember that we’re **not** actually executing a type guard here — the satisfies operator is exclusively using type information, based on what’s been inferred by the declaration of usHolidays and what’s been declared for the Holidays type.

## 8. Generics

Generics are dummy names for type parameters.

### Defining type parameter for function

simple example can be found below:

```ts
function wrapInArray<T>(arg: T): [T] {
  return [arg];
}
wrapInArray(new Date());
// ^? wrapInArray<number>(arg: number):[number]
wrapInArray(new Date());
// ^? wrapInArray<Date>(arg: Date):[Date]
```

### Defining type parameter for interface

simple example can be found below:

```ts
interface Dict<T> {
  [k: string]: T;
}
```

### An exercise: map, filter, and reduce for dictionary

```ts
///// SAMPLE DATA FOR YOUR EXPERIMENTATION PLEASURE (do not modify)
const fruits = {
  apple: { color: "red", mass: 100 },
  grape: { color: "red", mass: 5 },
  banana: { color: "yellow", mass: 183 },
  lemon: { color: "yellow", mass: 80 },
  pear: { color: "green", mass: 178 },
  orange: { color: "orange", mass: 262 },
  raspberry: { color: "red", mass: 4 },
  cherry: { color: "red", mass: 5 },
};

interface Dict<T> {
  [k: string]: T;
}

// Array.prototype.map, but for Dict
function mapDict(...args: any[]): any {}
// Array.prototype.filter, but for Dict
function filterDict(...args: any[]): any {}
// Array.prototype.reduce, but for Dict
function reduceDict(...args: any[]): any {}
```

<details><summary markdown="span">try the challenge before see the solution here!</summary>

```typescript
// Array.prototype.map, but for Dict
function mapDict<T, S>(
  inputDict: Dict<T>,
  mapFunction: (original: T, key: string) => S
): Dict<S> {
  const outDict: Dict<S> = {}
  for (let k of Object.keys(inputDict)) {
    const thisVal = inputDict[k]
    outDict[k] = mapFunction(thisVal, k)
  }
  return outDict
}
// Array.prototype.filter, but for Dict
function filterDict<T>(
  inputDict: Dict<T>,
  filterFunction: (value: T, key: string) => boolean
): Dict<T> {
  const outDict: Dict<T> = {}
  for (let k of Object.keys(inputDict)) {
    const thisVal = inputDict[k]
    if (filterFunction(thisVal, k)) outDict[k] = thisVal
  }
  return outDict
}
// Array.prototype.reduce, but for Dict
function reduceDict<T, S>(
  inputDict: Dict<T>,
  reducerFunction: (
    currentVal: S,
    dictItem: T,
    key: string
  ) => S,
  initialValue: S
): S {
  let value = initialValue
  for (let k of Object.keys(inputDict)) {
    const thisVal = inputDict[k]
    value = reducerFunction(value, thisVal, k)
  }
  return value
}
```
</details>
<br/>

### Generic constraints: “minimum requirement” for a type param

The way we define constraints on generics is by using the `extends` keyword. Assume we have the following function parametrized by generics:

```typescript
interface Dict<T> {
  [k: string]: T;
}

function foo<T>(list: T[]): Dict<T> {
  // do smt
}
```

Let's say, you want `T` to describe some objects that have `id` field. We can impose the constraint by first define an extra interface like the following:

```typescript
interface HasId {
  id: string
}

function<T extends HasId>(list: T[]): Dict<T>{}
```

### TypeParams best practice

The snippet below gives two similar functions(`example1` and `example2`) that produce the same results(i.e. `result1` and `result2` are the same).

```typescript
interface HasId {
  id: string;
}
interface Dict<T> {
  [k: string]: T;
}

function example1<T extends HasId[]>(list: T) {
  return list.pop();
}
function example2<T extends HasId>(list: T[]) {
  return list.pop();
}

class Payment implements HasId {
  static #next_id_counter = 1;
  id = `pmnt_${Payment.#next_id_counter++}`;
}
class Invoice implements HasId {
  static #next_id_counter = 1;
  id = `invc_${Invoice.#next_id_counter++}`;
}

const result1 = example1([
  //   ^ const result1: HasId | undefined
  new Payment(),
  new Invoice(),
  new Payment(),
]);

const result2 = example2([
  //   ^ const result2: Payment | Invoice | undefined
  new Payment(),
  new Invoice(),
  new Payment(),
]);
```

Compare the types of `result1` and `result2`, we’re effectively losing type information because of the way we define our type parameter in `example1`. Thus, the best practice is to define type parameters as simply as possible.

### `infer`: access sub-parts of a large type

`infer` can only be used in conditional type expression. In the following example, the `ReturnType` type takes a function type T and checks if T is a function that returns some type R. If it is, it infers the return type of the function (R) and returns it. Otherwise, it returns never.

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function add(a: number, b: number): number {
  return a + b;
}
function greet(name: string): string {
  return `Hello, ${name}!`;
}

type AddReturnType = ReturnType<typeof add>; // number
type GreetReturnType = ReturnType<typeof greet>; // string
```

### `Record`: create object types with specific key-value pairs

`Record` is useful when you want to **enforce** a certain type for all the values in an object, but allows keys to be multiple types.

```typescript
type User = {
  id: number;
  name: string;
};

// A record that maps user IDs to user objects
// there must be two keys, "ddo" and "ccc", in the object
type UsersById = Record<number | "ddo" | "ccc", User>;

const users: UsersById = {
  2: { id: 2, name: "Bob" },
  ddo: { id: 3, name: "Pob" },
  ccc: { id: 4, name: "Jobs" },
};
```

### `Pick`: make new type by picking properties from old type

```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

// Create a new type that only includes the 'id' and 'name' properties
type UserSummary = Pick<User, "id" | "name">;
```

### Template literal types: create types based on string literal

Here is a basic example:

```typescript
type ArtFeatures = "cabin" | "tree" | "sunset";
type Colors = "darkSienna" | "sapGreen" | "titaniumWhite" | "prussianBlue";

type ArtMethodNames = `paint_${Colors}_${ArtFeatures}`;
```

TS provides some utility types you can use in template literal types: `UpperCase`/`LowerCase`/`Capitalize`/`Uncapitalize`. For example,

```typescript
type ArtMethodNames = `paint${Capitalize<Colors>}${Capitalize<ArtFeatures>}`;
```

With the template, we can map the keys of an existing object type into something else. In the following example, we add `set` in front of keys of `type DataSDK`:

```typescript
interface DataState {
  digits: number[];
  names: string[];
  flags: Record<"darkMode" | "mobile", boolean>;
}

type DataSDK = {
  // The mapped type, notice the use of "as" here
  [K in keyof DataState as `set${Capitalize<K>}`]: (arg: DataState[K]) => void;
};

function load(dataSDK: DataSDK) {
  dataSDK.setDigits([14]);
  dataSDK.setFlags({ darkMode: true, mobile: false });
}
```

TS5 allows `infer` to be used in template literal type, which makes it easy to extract portions of string literal:

```typescript
const courseWebsite = "Frontend Masters";
type ExtractMasterName<S> = S extends `${infer T} Masters` ? T : never;
let fe: ExtractMasterName<typeof courseWebsite> = "Backend";
//  typeof fe is "Frontend", so the assignment above causes error.
```

### Type registry: one handles many return types

Imagine we need a function(let's call it `multiFunc`) to return data of different types/classes, depending on various conditions. Of course you can define it using union type as the following:

```typescript
function multiFunc(
  typeName: "class1" | "class2",
  identifier: string,
): Class1 | Class2 {
  if (typeName == "class1") {
    return new Class1(identifier);
  } else {
    return new Class2(identifier);
  }
}
```

There is one big disadvantage of doing so: **poor scalabilit**: as you add more types, the union(`Class1 | Class2 | Class3...`) becomes unwieldy and complicates your function definition.

We can use type registry to avoid over-complicating your function definitions. Here is how we do it in the example above:

```typescript 
// define an interface for the registry 
interface FuncTypeRegistry {
    "class1": Class1,
    "class2": Class2
}
// now we can use generics to define our "multiFunc"
function multiFunc<K extends keyof FuncTypeRegistry>(
    typeName: K,
    identifier: string,
   // notice how union type now disappears!!
): FuncTypeRegistry[K] {
// logic is unchanged.
}
```

We can take a step further by specifying the format of `identifier` using [literal template](#template-literal-types-create-types-based-on-string-literal):

```typescript 
function multiFunc<K extends keyof FuncTypeRegistry>(
    typeName: K,
    // assume we want to set identifier for Class1 instance 
    // as "class1_xxxx"
    // and Class2 instance as "class2_xxx"
    identifier: `${K}_${string}`,
): FuncTypeRegistry[K] {
// logic is unchanged.
}
```

Based on this, we can even define another function that takes a array of "identifier" and return an array of `Class1` or `Class2` as the following:

```typescript 
function multiFuncArray<K extends keyof FuncTypeRegistry>(
    typeName: K,
    // an array of "class1_xxx" or array of "class2_xxx"
    identifiers: `${K}_${string}`[]
   // notice how extra bracket is added here
): FuncTypeRegistry[K][]{
    // new logic for arrays
}
```

## 9. Modules & CJS interop

### ES Module imports and exports

Basic examples:

```typescript
// named imports
import { Blueberry, Raspberry } from "./berries";
import Kiwi from "./kiwi"; // default import
export function makeFruitSalad() {} // named export
export default class FruitBasket {} // default export
export { lemon, lime } from "./citrus"; // re-export
export * as berries from "./berries"; // re-export entire module as a single namespace
```

Although fairly uncommon in the JS world, it’s possible to import an entire module as a namespace. TypeScript supports this as well

```typescript
import * as allBerries from "./berries"; // namespace import
allBerries.Strawberry; // using the namespace
allBerries.Blueberry;
allBerries.Raspberry;
export * from "./berries"; // namespace re-export
```

TypeScript provides an unambiguous way of importing _only_ types.

```typescript
import type { Strawberry } from "./berries/strawberry";
```

### Importing non-TS things

Importing images can be accomplished through a module declaration as shown below:

```typescript
// @filename: global.d.ts
declare module "*.png" {
  const imgUrl: string;
  export default imgUrl;
}
// @filename: component.ts
import img from "./file.png";
```
