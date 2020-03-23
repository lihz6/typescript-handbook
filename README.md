# TypeScript Handbook Refresher

## Basic Types

- `undefined`: `undefined`
- `boolean`: `true` `false`
- `number`: `0` `0b0` `0o0` `0x0` `0.0`
- `string`: `''` `""` <code>\`\${expr}`</code>
- `symbol`: `Symbol()`
- `null`: `null`
- `object`: not `undefined`, `boolean`, `number`, `string`, `symbol` or `null`

* `array`: `ReadonlyArray<T>` `readonly Array<T>` `readonly T[]` `Array<T>` `T[]`
* `tuple`: `readonly [T1, T2, ...]` `[T1, T2, ...]`

- `any`: `T | any` `unknown`
- `void`: `undefined` `void | T`
- `never`: `throw Error()` `while (true) {}` `...`

### Type Assertions

- `<Type>value`
- `value as Type`
- `value!`

### Type Variables

```ts
let isDone: boolean;
isDone = true;

let isDone: boolean = true;

let isDone = true;
```

```ts
const isDone: boolean = true;

const isDone = true;
```

```ts
const person: { name: string; age: number } = { name: 'John', age: 21 };

const person = { name: 'John', age: 21 };
```

```ts
const person: { readonly name: string; readonly age: number } = {
  name: 'John',
  age: 21
};

const person = { name: 'John', age: 21 } as const;
```

### Type Functions

```ts
function add(a: number, b: number): number {
  return a + b;
}

const add: (a: number, b: number) => number = (a, b) => a + b;

function add(a: number, b: number) {
  return a + b;
}

const add = (a: number, b: number) => a + b;
```

```ts
function add(a: number, b: number | undefined = undefined) {
  if (typeof b === 'undefined') {
    return a + 1;
  }
  return a + b;
}

function add(a: number, b: void | number) {
  if (typeof b === 'undefined') {
    return a + 1;
  }
  return a + b;
}

function add(a: number, b?: number) {
  if (typeof b === 'undefined') {
    return a + 1;
  }
  return a + b;
}

function add(a: number, b: number = 1) {
  return a + b;
}
```

```ts
function sum(a: number, ...nums: number[]) {
  return nums.reduce((a, b) => a + b, a);
}
```

### Type Classes

```js
// This is JavaScript code.
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}
```

```ts
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

```ts
class Person {
  // One of "[public | protected | private] [readonly]"
  constructor(public name: string, public age: number) {}
}
```

```ts
class Component {
  // "!" indicates this field is set by external code
  context!: string;

  printContext() {
    console.log(this.context);
  }
}
```

## Interfaces

### Duck Typing

```ts
interface LabeledValue {
  label: string;
}

function printLabel(labeledObj: LabeledValue) {
  console.log(labeledObj.label);
}

const myObj = { size: 10, label: 'Size 10 Object' };
printLabel(myObj);
```

### Optional Properties

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  const newSquare = { color: 'white', area: 100 };
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

const mySquare = createSquare({ color: 'black' });
```

> Note: Both `{}` and `{ prop: undefined }` are assignable to `{ prop?: T }`, but `{}` not to `{ prop: T | undefined }`.

### Excess Property Checks

Object literals get special treatment and undergo _excess property checking_ when **assigning them to other variables**, or **passing them as arguments**.

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  // ...
}

// Error: There’s probably a bug in this code.
const mySquare = createSquare({ colour: 'red', width: 100 });
```

### Indexable Types

```ts
interface Indexable<T, V extends T> {
  [key: string]: T;
  [key: number]: V;
}
```

> Note: There are two types of supported index signatures: `string` and `number`. Because JavaScript runtime treats `indexable[100]` as `indexable['100']`, `Indexable[number]` must be a subtype of `Indexable[string]`.

### Readonly properties

Some properties should only be modifiable when an object is first created.

```ts
interface Point {
  readonly x: number;
  readonly y: number;
}
```

> Note: `readonly` prevents from modifying while `const` prevents from re-assigning.

### Function Types

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = (src, sub) => {
  const result = src.search(sub);
  return result > -1;
};
```

### Implementing an Interface

```ts
interface ClockInterface {
  // Can't be `private` or `protected`.
  // See "Interfaces Extending Classes"
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  setTime(d: Date) {
    this.currentTime = d;
  }
}
```

### Interfaces for Contructors

```ts
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
  tick(): void;
}

function createClock(
  ctor: ClockConstructor,
  hour: number,
  minute: number
): ClockInterface {
  return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log('beep beep');
  }
}
class AnalogClock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log('tick tock');
  }
}

const digital = createClock(DigitalClock, 7, 32);
const analog = createClock(AnalogClock, 12, 17);
```

### Extending Interfaces

```ts
interface Shape<T> {
  shadowed: T;
}

interface Square<T, V extends T> extends Shape<T> {
  shadowed: V;
}
```

```ts
interface Shape {
  color: string;
}

interface PenStroke {
  penWidth: number;
}

interface Square extends Shape, PenStroke {
  sideLength: number;
}
```

### Hybrid Types

```ts
interface Counter {
  (start: number): void;
  interval: number;
  reset(): void;
}

function getCounter(): Counter {
  const counter = (function() {} as unknown) as Counter;
  counter.interval = 123;
  counter.reset = function() {};
  return counter;
}
```

### Interfaces Extending Classes

When an interface type extends a class type it inherits the members of the class. If an interface extends a class with private or protected members, that interface type can only be implemented by that class or a subclass of it.

```ts
class Control {
  private state: any;
}

interface SelectableControl extends Control {
  select(): void;
}

class Button extends Control implements SelectableControl {
  select() {}
}

class TextBox extends Control {
  select() {}
}

// Error: Types have separate declarations of a private property 'state'.
class Image implements SelectableControl {
  private state: any;
  select() {}
}
```

## Classes

### `this` and Overload

```ts
class Overload {
  prop: string;

  constructor();
  constructor(prop: string);
  constructor(prop?: string) {
    // SyntaxError: `this(prop);`
    this.prop = prop ?? 'Unknown';
  }

  method(a: number): string;
  method(a: string): number;
  method(a: number | string): string | number {
    if (typeof a === 'number') {
      return `${this.prop}: ${a}`;
    }
    return parseInt(a);
  }
}
```

### `super` and Inheritance

```ts
class Animal<T, V> {
  constructor(public name: string) {}
  method(_arg: T): V {
    return null as any;
  }
}

class Monkey<T, V, T1 extends T, V1 extends V> extends Animal<T1, V> {
  constructor(name: string) {
    super(name);
  }
  method(_arg: T): V1 {
    super.method(_arg as T1);
    return null as any;
  }
}
```

### Readonly modifier

```ts
class Octopus {
  readonly numberOfLegs: number = 8;
  constructor(readonly name: string) {}
}
```

### Accessors

```ts
class Employee {
  private _fullName!: string;

  get fullName(): string {
    return this._fullName;
  }

  set fullName(newName: string) {
    this._fullName = newName;
  }
}
```

> Note: Accessors with a `get` and no `set` are automatically inferred to be `readonly`.

### Static Properties

```ts
class Grid {
  static origin = { x: 0, y: 0 };

  constructor(public scale: number) {}

  calculateDistanceFromOrigin(point: { x: number; y: number }) {
    const xDist = point.x - Grid.origin.x;
    const yDist = point.y - Grid.origin.y;
    return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
  }
}
```

### `public`, `private`, and `protected` modifiers

For two types to be considered compatible, if one of them has a `private` member, then the other must have a `private` member that originated in the same declaration. The same applies to `protected` members.

```ts
// `public` by default and freely access
class Animal {
  public name: string;
  public constructor(theName: string) {
    this.name = theName;
  }
  public move(distanceInMeters: number) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}
```

```ts
// `private` restricts in containing class
class Animal {
  private name: string;
  constructor(theName: string) {
    this.name = theName;
  }
}
```

```ts
// `protected` reaches to deriving classes
class Person {
  protected name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Employee extends Person {
  private department: string;

  constructor(name: string, department: string) {
    super(name);
    this.department = department;
  }

  public getElevatorPitch() {
    // Note: `super.name` is `undefined`.
    return `Hello, my name is ${this.name} and I work in ${this.department}.`;
  }
}
```

### Abstract Classes

```ts
abstract class Department {
  constructor(public name: string) {}

  // must be implemented in derived classes
  // can't mark as `private`
  abstract getFullname(): string;

  printName(): void {
    console.log('Department name: ' + this.getFullname());
  }
}
```

### Constructor functions

```ts
class Greeter {
  static standardGreeting = 'Hello, there';
  greeting!: string;
  greet() {
    if (this.greeting) {
      return 'Hello, ' + this.greeting;
    } else {
      return Greeter.standardGreeting;
    }
  }
}

class MyGeeter extends Greeter {}

function greet(greeterMaker: typeof Greeter) {
  greeterMaker.standardGreeting = 'Hey there!';
  const greeter: Greeter = new greeterMaker();
  console.log(greeter.greet());
}

greet(MyGeeter);
greet(Greeter);
```

## Functions

### `this` parameters

`this` parameters are fake parameters that come first in the parameter list of a function:

```ts
function f(this: void) {
  // make sure `this` is unusable in this standalone function
}
```

```ts
interface Card {
  suit: string;
  card: number;
}
interface Deck {
  suits: string[];
  cards: number[];
  createCardPicker(this: Deck): () => Card;
}
const deck: Deck = {
  suits: ['hearts', 'spades', 'clubs', 'diamonds'],
  cards: Array(52),
  createCardPicker(this: Deck) {
    return () => {
      const pickedCard = Math.floor(Math.random() * 52);
      const pickedSuit = Math.floor(pickedCard / 13);

      return { suit: this.suits[pickedSuit], card: pickedCard % 13 };
    };
  }
};

const cardPicker = deck.createCardPicker();
const { suit, card } = cardPicker();
```

```ts
declare const addClickListener: (
  onclick: (this: void, e: Event) => void
) => void;
class Handler {
  type!: string;
  onClickGood = (e: Event) => {
    this.type = e.type;
  };
}

addClickListener(new Handler().onClickGood);
```

> Note: An arrow function cannot have a `this` parameter. Read [Understanding JavaScript Function Invocation and “this”](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/).

### Overloads

```ts
function pickCard(x: { suit: string; card: number }[]): number;
function pickCard(x: number): { suit: string; card: number };
function pickCard(x: any): any {
  const suits = ['hearts', 'spades', 'clubs', 'diamonds'];

  // Check to see if we're working with an object/array
  // if so, they gave us the deck and we'll pick the card
  if (typeof x == 'object') {
    const pickedCard = Math.floor(Math.random() * x.length);
    return pickedCard;
  }
  // Otherwise just let them pick the card
  else if (typeof x == 'number') {
    const pickedSuit = Math.floor(x / 13);
    return { suit: suits[pickedSuit], card: x % 13 };
  }
}
```

> Note:
>
> 1. Order overloads from most specific to least specific.
> 2. The last piece is not part of the overload list.

## Enums

- Each enum member can be numeric or string value.
- Only numeric members can be reverse mapping.
- Each enum member can be constant or computed.
  > A constant is an expression that can be fully evaluated at compile time.
- When each member is constant:
  - Enum members also become types as well.
  - The enum becomes a union type of all members.

```ts
enum FileAccess {
  // constant members
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
  // computed member
  G = '123'.length
}
```

```ts
enum Enum {
  A
}
Enum[Enum.A]; // "A"
```

```ts
enum ShapeKind {
  Circle,
  Square
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}
```

### `const` enums

- `const` enums members are expanded to values.
- `const` enums cannot have computed members.
- `const` enums cannot be used as values.

```ts
const enum Reply {
  YES,
  NO
}
// expanded: [0 /* YES */, 1 /* NO */];
const replies = [Reply.YES, Reply.NO];
```

### Enums at compile time

```ts
enum KeyOfShape {
  COLOR = 'color',
  WIDTH = 'width'
}

interface Shape {
  [KeyOfShape.COLOR]: string;
  [KeyOfShape.WIDTH]: number;
}

type KeyOfShapeNames = keyof typeof KeyOfShape;

function valueOf<T, K extends keyof T>(t: T, k: K): T[K] {
  return t[k];
}

function keyOf<T>(t: T, v: T[keyof T]): keyof T {
  for (const k in t) {
    if (t[k] === v) {
      return k;
    }
  }
  throw Error();
}
```

## Generics

Generics is about _relationships_ of types. The power of a system comes more from the relationships among types than from the types themselves.

### The Identity Function.

The identity function is a function that will return back whatever is passed in.

```ts
// limited to accept a specific type
function identity(arg: number): number {
  return arg;
}

// losing type information when returns
function identity(arg: any): any {
  return arg;
}
```

```ts
// the generic way, use a type variable for
// capturing type information when provided.
function identity<T>(arg: T): T {
  return arg;
}

// explicitly pass in the type arguments
identify<string>('myString');

// rely on type argument inference
identify('myString');
```

### Generic Functions and Methods

```ts
function identity<T>(arg: T): T {
  return arg;
}

const identity: <T>(arg: T) => T = arg => arg;

const identityObj = {
  identity<T>(arg: T): T {
    return arg;
  }
};

class IdentityCls {
  identity<T>(arg: T): T {
    return arg;
  }
}
```

### Generic Interfaces

```ts
interface GenericIdentityFn {
  <T>(arg: T): T;
}

interface GenericIdentityFn<T> {
  (arg: T): T;
}

interface Addable<T> {
  readonly zeroValue: T;
  value: T;
  add(other: Addable<T>): Addable<T>;
}

class MyNumber implements Addable<number> {
  readonly zeroValue = 0;
  constructor(public value: number) {}
  add(other: MyNumber): MyNumber {
    return new MyNumber(this.value + other.value);
  }
}
```

### Generic Classes

```ts
class Container<T> {
  constructor(private item: T) {}
  store(item: T) {
    this.item = item;
  }
  fetch(): T {
    return this.item;
  }
}
```

> Note: Static members can not use the class’s type parameter.

### Generic Constraints

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
```

### Using Class Types in Generics

```ts
function create<T>(c: { new (): T }): T {
  return new c();
}
```

## Literal Types

- There are two sets of literal types available in TypeScript today, strings and numbers.
- Using `const` to declare a variable triggers _literal narrowing_, while `var` or `let` don't.
- Literal types combine nicely with _union types_, _type guards_ and _type aliases_.

```ts
// type "Hello World"
const helloWorld = 'Hello World';

// type string
let hiWorld = 'Hi World';
```

```ts
type Easing = 'ease-in' | 'ease-out' | 'ease-in-out';
class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === 'ease-in') {
      // ...
    } else if (easing === 'ease-out') {
      // ...
    } else if (easing === 'ease-in-out') {
      // ...
    } else {
      // Error: No more values here.
    }
  }
}

const button = new UIElement();
button.animate(0, 0, 'ease-in');
// Error: "uneasy" is not allowed here.
button.animate(0, 0, 'uneasy');
```

```ts
function rollDice(): 1 | 2 | 3 | 4 | 5 | 6 {
  // Error: Type '0' is not assignable to type '1 | 2 | 3 | 4 | 5 | 6'
  return 0;
}

function foo(x: number) {
  if (x !== 1 || x !== 2) {
    //           ~~~~~~~
    // Error: The types '1' and '2' have no overlap.
  }
}
```

## Advanced Types

### Intersection Types

An intersection type combines multiple types into one, for example, `Person & Serializable & Loggable`. That means an object of this type will have all members of all three types.

```ts
function extend<F, S>(first: F, second: S): F & S {
  const result: Partial<F & S> = {};
  for (const prop in first) {
    if (Object.prototype.hasOwnProperty.call(first, prop)) {
      (result as F)[prop] = first[prop];
    }
  }
  for (const prop in second) {
    if (Object.prototype.hasOwnProperty.call(first, prop)) {
      (result as S)[prop] = second[prop];
    }
  }
  return result as F & S;
}
```

### Union Types

```ts
function padLeft(value: string, padding: number | string) {
  if (typeof padding === 'number') {
    return Array(padding + 1).join(' ') + value;
  }
  if (typeof padding === 'string') {
    return padding + value;
  }
}
```

### Type Guards

```ts
interface Bird {
  fly();
  layEggs();
}

interface Fish {
  swim();
  layEggs();
}

// Type predicate: A predicate takes the form `parameterName is Type`.
function isFish(pet: Fish | Bird): pet is Fish {
  return 'swim' in pet;
}

function playSmallPet(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim();
  } else {
    pet.fly();
  }
}
```

#### Using the `in` operator

```ts
function move(pet: Fish | Bird) {
  if ('swim' in pet) {
    return pet.swim();
  }
  return pet.fly();
}
```

#### `typeof` type guards

The `typeof` type guards are recognized in two different forms:

- `typeof v === <typename>`
- `typeof v !== <typename>`

where `<typename>` must be:

- `'boolean'`
- `'string'`
- `'number'`
- `'symbol'`

> Note: There are three more possible values of `typeof v`:
>
> - `'undefined'`
> - `'function'`
> - `'object'`

#### `instanceof` type guards

```ts
interface Padder {}

class SpaceRepeatingPadder implements Padder {}

class StringPadder implements Padder {}

function applyRandomPadder(padder: SpaceRepeatingPadder | StringPadder) {
  if (padder instanceof SpaceRepeatingPadder) {
    padder; // type narrowed to 'SpaceRepeatingPadder'
  }
  if (padder instanceof StringPadder) {
    padder; // type narrowed to 'StringPadder'
  }
}
```

#### Assertion Functions

```ts
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new Error(msg);
  }
}

function yell(str: any) {
  assert(typeof str === 'string');

  return str.toUppercase();
  //         ~~~~~~~~~~~
  // Error: Property 'toUppercase' does not exist on type 'string'.
  //        Did you mean 'toUpperCase'?
}
```

```ts
function assertIsString(val: any): asserts val is string {
  if (typeof val !== 'string') {
    throw new Error('Not a string!');
  }
}
```

### Nullable types

By default, `null` and `undefined` assignable to anything, but this is a [“billion dollar mistake”](https://en.wikipedia.org/wiki/Null_pointer#History). Fortunately, the `--strictNullChecks` flag fixes this:

```ts
let s = 'foo';
s = null; // Error: 'null' is not assignable to 'string'
```

```ts
let s: string | null = 'bar';
s = null; // OK

s = undefined; // Error: 'undefined' is not assignable to 'string | null'
```

#### Automatically Adding <code>&nbsp;| void</code>

Optional Parameters and Properties automatically add <code>&nbsp;| void</code>:

```ts
function f(x: number, y?: number) {
  // ...
}

f(1);
f(1, 2);
f(1, undefined);

f(1, null); // Error: 'null' is not assignable to 'number | void'
```

```ts
class C {
  constructor(public a: number, public b?: number) {}
}

new C(1);
new C(1, 2);
new C(1, undefined);

new C(1, null); // Error: 'null' is not assignable to 'number | void'
```

#### Optional Chaining with `?.`

Use _optional chaining_ to simplify working with nullable types:

```ts
const x = foo === null || foo === undefined ? undefined : foo.bar.baz();

// optional property access
const x = foo?.bar.baz();
```

```ts
function f(foo?: number[] | null) {
  // optional element access
  foo?.[0];
}

function f(foo?: (() => void) | null) {
  // optional call
  foo?.();
}
```

#### Nullish Coalescing with `??`

```ts
const x = foo !== null && foo !== undefined ? foo : bar();

const x = foo ?? bar();
```

> Note: Comparing with `val || alt`, `val ?? alt` avoids some unintended behavior from `0`, `NaN`, `false` and `''` being treated as falsy values.

#### Eliminate Nullable with `!`

```ts
function f(arg?: number | null): number {
  return arg!;
}
```

### Type Aliases

Type aliases create a new name for a type, doesn’t actually create a new type.

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
  if (typeof n === 'string') {
    return n;
  } else {
    return n();
  }
}
```

```ts
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
};
```

```ts
// This is how type aliases can be extended.
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
  name: string;
}
```

> Note: For [being open to extension](https://en.wikipedia.org/wiki/Open/closed_principle), use an interface over a type alias if possible.

### Discriminated Unions

There are three ingredients:

1. Types that have a common, **singleton type** property — the _discriminant_.
   1. String literal types.
   2. Numeric literal types.
   3. Enum member types.
2. A type alias that takes the union of those types — the _union_.
3. Type guards on the common property.

```ts
// Step 1
interface Square {
  kind: 'square';
  size: number;
}
interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}
interface Circle {
  kind: 'circle';
  radius: number;
}

// Step 2
type Shape = Square | Rectangle | Circle;

// Step 3
function area(s: Shape) {
  switch (s.kind) {
    case 'square':
      return s.size * s.size;
    case 'rectangle':
      return s.height * s.width;
    case 'circle':
      return Math.PI * s.radius ** 2;
  }
}
```

#### Exhaustiveness checking

```ts
function area(s: Shape) {
  switch (s.kind) {
    case 'square':
      return s.size * s.size;
    case 'rectangle':
      return s.height * s.width;
  }
  // should error here - case 'circle' not handled
}
```

There are three ways to tell the compiler to cover all variants of the discriminated union.

1. Turn on `--noImplicitReturns` to Ensure that all codepaths return in a function.
2. Turn on `--strictNullChecks` and specify a return type.
3. Use the `never` type to check for exhaustiveness.

```ts
function assertNever(x: never): never {
  throw new Error('Unexpected object: ' + x);
}

function area(s: Shape) {
  switch (s.kind) {
    case 'square':
      return s.size * s.size;
    case 'rectangle':
      return s.height * s.width;
    default:
      // error here if there are missing cases
      return assertNever(s);
  }
}
```

### Polymorphic `this` types

```ts
class BasicCalculator {
  constructor(protected value: number) {}
  currentValue(): number {
    return this.value;
  }
  // Change `this` to `BasicCalculator` to see what happens.
  multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
}

class ScientificCalculator extends BasicCalculator {
  constructor(value = 0) {
    super(value);
  }
  sin() {
    this.value = Math.sin(this.value);
    return this;
  }
  // ... other operations go here ...
}

const v = new ScientificCalculator(2)
  .multiply(5)
  .sin()
  .currentValue();
```

### Index types

- `keyof T`, the **index type query** operator.
- `T[keyof T]`, the **indexed access** operator.

```ts
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
  // o[propertyName] is of type T[K]
  return o[propertyName];
}

const taxi = {
  manufacturer: 'Toyota',
  model: 'Camry',
  year: 2014
};

const name: string = getProperty(taxi, 'manufacturer');
const year: number = getProperty(taxi, 'year');
```

#### Index types and index signatures

```ts
interface Dictionary<T> {
  [key: string]: T;
}
let keys: keyof Dictionary<number>; // string | number
let val1: Dictionary<number>[string]; // number;
let val2: Dictionary<number>[number]; // number;
let val3: Dictionary<number>['foo']; // number
let val4: Dictionary<number>[123]; // number
```

```ts
interface Dictionary<T> {
  [key: number]: T;
}
let keys: keyof Dictionary<number>; // number
let val1: Dictionary<number>[number]; // number;
let val2: Dictionary<number>[42]; // number
```

### Mapped types

The syntax resembles the syntax for index signatures with a `for .. in` inside.

```ts
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
```

```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

```ts
// use an intersection type to add new members
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean };
```

```ts
type Proxy<T> = {
  get(): T;
  set(value: T): void;
};

type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>;
};

function proxify<T>(o: T): Proxify<T> {
  const result = {} as Proxify<T>;
  for (const k in o) {
    result[k] = {
      get() {
        return o[k];
      },
      set(value) {
        o[k] = value;
      }
    };
  }
  return result;
}

function unproxify<T>(t: Proxify<T>): T {
  let result = {} as T;
  for (const k in t) {
    result[k] = t[k].get();
  }
  return result;
}
```

### Conditional Types

A conditional type `T extends U ? X : Y`, when `T` is assignable to `U` the type is `X`, otherwise the type is `Y`.

```ts
type TypeName<T> = T extends string
  ? 'string'
  : T extends number
  ? 'number'
  : T extends boolean
  ? 'boolean'
  : T extends undefined
  ? 'undefined'
  : T extends Function
  ? 'function'
  : 'object';

type T0 = TypeName<string>; // 'string'
type T1 = TypeName<'a'>; // 'string'
type T2 = TypeName<true>; // 'boolean'
type T3 = TypeName<() => void>; // 'function'
type T4 = TypeName<string[]>; // 'object'
```

`T extends U ? X : Y` is either **resolved** to `X` or `Y`, or **deferred** - where they stick around instead of picking a branch.

```ts
declare function f<T extends boolean>(x: T): T extends true ? string : number;

// Type is 'string | number'
const x = f(Math.random() < 0.5);
```

```ts
interface Foo {
  propA: boolean;
  propB: boolean;
}

declare function f<T>(x: T): T extends Foo ? string : number;

function foo<U>(x: U) {
  // `U extends Foo ? string : number` is assignable to `string | number`
  const b: string | number = a;
}
```

#### Distributive conditional types

`T extends U ? X : Y` in which `T` is a naked type parameter are called distributive conditional types. Distributive conditional types are automatically distributed over union types during instantiation. For example, an instantiation of `T extends U ? X : Y` with the type argument `A | B | C` for `T` is resolved as `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`.

```ts
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T20 = Boxed<string>; // BoxedValue<string>;
type T21 = Boxed<number[]>; // BoxedArray<number>;
type T22 = Boxed<string | number[]>; // BoxedValue<string> | BoxedArray<number>;
```

```ts
type Diff<T, U> = T extends U ? never : T; // Remove types from T that are assignable to U
type Filter<T, U> = T extends U ? T : never; // Remove types from T that are not assignable to U

type T30 = Diff<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>; // 'b' | 'd'
type T31 = Filter<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>; // 'a' | 'c'
type T32 = Diff<string | number | (() => void), Function>; // string | number
type T33 = Filter<string | number | (() => void), Function>; // () => void

type NonNullable<T> = Diff<T, null | undefined>; // Remove null and undefined from T

type T34 = NonNullable<string | number | undefined>; // string | number
type T35 = NonNullable<string | string[] | null | undefined>; // string | string[]
```

> Note: `T | never` = `T`, `T & never` = `never`.

```ts
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;
```

#### Type inference in conditional types

Within the `extends` clause of a conditional type, it is now possible to have `infer` declarations that introduce a type variable to be inferred. Such inferred type variables may be referenced in the true branch of the conditional type. It is possible to have multiple `infer` locations for the same type variable.

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

```ts
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
  ? U
  : T extends Promise<infer U>
  ? U
  : T;

type T0 = Unpacked<string>; // string
type T1 = Unpacked<string[]>; // string
type T2 = Unpacked<() => string>; // string
type T3 = Unpacked<Promise<string>>; // string
type T4 = Unpacked<Promise<string>[]>; // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>; // string
```

```ts
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;
type T11 = Foo<{ a: string; b: number }>; // string | number

type Bar<T> = T extends { a: (x: infer U) => void; b: (x: infer U) => void }
  ? U
  : never;
type T21 = Bar<{ a: (x: string) => void; b: (x: number) => void }>; // string & number
```

```ts
// Error, not supported
type ReturnType<T extends (...args: any[]) => infer R> = R;

type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R
  ? R
  : any;
```

## Type Inference

### Infer from Initializers

- initializing variables and members
- setting parameter default values
- determining function return types

```ts
let x = 3; // number
```

### Best Narrowed Type

```ts
let s = 'a'; // string

const s = 'a'; // 'a'

const flags = [1, 2, 3] as const; // readonly [1, 2, 3]
```

### Best Common Type

```ts
let x = [0, 1, null]; // (number | null)[]
```

```ts
const zoo = [new Elephant(), new Snake()]; // (Elephant | Snake)[]

const zoo = [new Animal(), new Elephant(), new Snake()]; // Animal[]
```

### Contextual Typing

```ts
window.onmousedown = function(mouseEvent) {
  console.log(mouseEvent.button); // OK
  console.log(mouseEvent.kangaroo); // Error!
};
```
