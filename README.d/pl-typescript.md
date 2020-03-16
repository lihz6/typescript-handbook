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

```typescript
let isDone: boolean;
isDone = true;

let isDone: boolean = true;

let isDone = true;
```

```typescript
const isDone: boolean = true;

const isDone = true;
```

### Type Functions

```typescript
function add(a: number, b: number): number {
  return a + b;
}

const add: (a: number, b: number) => number = (a, b) => a + b;

function add(a: number, b: number) {
  return a + b;
}

const add = (a: number, b: number) => a + b;
```

```typescript
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

```typescript
function sum(a: number, ...nums: number[]) {
  return nums.reduce((a, b) => a + b, a);
}
```

### Type Classes

```javascript
// This is JavaScript code.
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}
```

```typescript
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

```typescript
class Person {
  // One of "[public | protected | private] [readonly]"
  constructor(public name: string, public age: number) {}
}
```

```typescript
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

```typescript
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

```typescript
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

```typescript
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

```typescript
interface Indexable<T, V extends T> {
  [key: string]: T;
  [key: number]: V;
}
```

> Note: There are two types of supported index signatures: `string` and `number`. Because JavaScript runtime treats `indexable[100]` as `indexable['100']`, `Indexable[number]` must be a subtype of `Indexable[string]`.

### Readonly properties

Some properties should only be modifiable when an object is first created.

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}
```

> Note: `readonly` prevents from modifying while `const` prevents from re-assigning.

### Function Types

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = (src, sub) => {
  const result = src.search(sub);
  return result > -1;
};
```

### Implementing an Interface

```typescript
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

```typescript
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

```typescript
interface Shape<T> {
  shadowed: T;
}

interface Square<T, V extends T> extends Shape<T> {
  shadowed: V;
}
```

```typescript
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

```typescript
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

```typescript
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

```typescript
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

```typescript
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

```typescript
class Octopus {
  readonly numberOfLegs: number = 8;
  constructor(readonly name: string) {}
}
```

### Accessors

```typescript
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

```typescript
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

```typescript
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

```typescript
// `private` restricts in containing class
class Animal {
  private name: string;
  constructor(theName: string) {
    this.name = theName;
  }
}
```

```typescript
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

```typescript
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

```typescript
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

```typescript
function f(this: void) {
  // make sure `this` is unusable in this standalone function
}
```

```typescript
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
  },
};

const cardPicker = deck.createCardPicker();
const { suit, card } = cardPicker();
```

```typescript
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

```typescript
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
