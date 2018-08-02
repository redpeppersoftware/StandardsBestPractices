
# Red Pepper TypeScript Styleguide
This document outlines the general guidelines for working with TypeScript at Red Pepper.

## Contents

- [Single Responsibility Principle](#single-responsibility-principle)
- [File and Function Size](#file-and-function-size)
- [Column Length Limit](#column-length-limit)
- [Tabs vs Spaces](#tabs-vs-spaces)
- [Naming](#naming)
  * [Classes](#classes)
  * [Files](#files)
- [Rule of One](#rule-of-one)
- [Const vs Let vs Var](#const-vs-let-vs-var)
  * [When to Use Const](#when-to-use-const)
- [Anonymous Functions](#anonymous-functions)
- [Arrow Function Parameters](#arrow-function-parameters)
- [Ternary Operator](#ternary-operator)
- [Loops](#loops)
- [Programming to Interfaces](#programming-to-interfaces)
  * [Using Interfaces](#using-interfaces)

## Single Responsibility Principle
Single Responsibility (SRP) refers to the practice of restricting any single file/module to have a clear, distinct purpose. This makes the project easier to manage and work with, and prevents tight coupling in files.

## File and Function Size
Individual code files should generally not exceed 700 lines of code. This is a _code smell_ that indicates your code may be tightly coupled and be violating SRP.

Functions should generally not exceed 75 lines of code. If your functions exceed this, it is an indicator that your function may be violating SRP. Very large functions are also very difficult to understand, debug, and change.

## Column Length Limit
Columns should not exceed 120 characters in length. This allows for flexible editing regardless of indivudal developer Editor setup.

## Tabs vs Spaces
Prefer 2 spaces in place of tabs.

## Naming

### Classes
Classes should be named using `PascalCase`.
### Files
File names should be named using `snake-case` when consecutive words appear adjacent to each other. 

When appropriate, join file names with the type of file contained using dots of the form `feature.type.ts`. For example, you may have the following file:

_Good_
```typescript
 /* user-profile.ts */
 export default class UserProfile {
   constructor(){}
 }
```

When writing tests, you can often use this naming convention so that your test framework automatically pics up your test file by naming your test file `user-profile.spec.ts` or `user-profile.test.ts`.

**Why?**
This naming convention helps maintain SRP and makes it easy to see at-a-glance what a file's purpose is by looking at the name. It also has functional applications (EG: usage with test frameworks).

## Rule of One
The Rule of One states that any given file/module should have only one Class, Module, Component, etc definition.

**Why?**

This makes code cleaner, clearer, and easier to refactor. In some environments it also facilitates lazy loading and more efficient module resolution by leveraging `export default` functionality.

_Bad_
```typescript
/* MyClass.ts */
export class MyClass {
  id: number;
  constructor(){
    this.id = getId();
  }
}

const idCount = 0;
export function getId(): number{
  return idCount++;
}
```

_Good_
```typescript
/* MyClass.ts */

import getId from './getId';

export default class MyClass {
  id: number;
  constructor(){
    this.id = getId();
  }
}

/* getId.ts */
const idCount = 0;
export default function getId(): number{
  return idCount++;
}
```

## Const vs Let vs Var

Always use either `const` or `let`, _never_ `var`. For a more detailed description of different declaration behavior, consult the [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/variable-declarations.html)

### When to Use Const

In general, using `const` for variable declarations is the _most explicit_ way of describing a variable that shouldn't change during the lifetime of you application. For example, the following pattern is used in almost every programming language:

```typescript
export const animals = ['Cat', 'Dog', 'Snake'];

animals = ['Carrot']; // Error: animals is read-only
```

Const should also follow practices observed in other languages. This includes situations where static lists of primitives or application-specific objects should be used. Consider the following list, where we're defining placeholder labels **That will never change** during runtime:

_Bad_
```typescript
let entityKeys = {
  user: 'user',
  role: 'role',
  task: 'task',
};

type item = {
  type: string;
  value: any | null; // null indicates it is a palceholder and not an option
  label: string;
};

let placeholderItems: item[] = [
  { type: entityKeys.user, value: null, label: 'Select a User' },
  { type: entityKeys.role, value: null, label: 'Select a role' }
];

let user = placeholderItems[0];

entityKeys = Object.assign(entityKeys, {user: 'contact'}); // OK, compiles

console.log(user.type === entityKeys.user); // False, not what we want
```

_Good_
```typescript
//List of server-side entity types
const entityKeys = {
  user: 'user',
  role: 'role',
  task: 'task',
};

type item = {
  type: string;
  value: any | null; // null indicates it is a palceholder and not an option
  label: string;
};

let placeholderItems: item[] = [
  { type: entityKeys.user, value: null, label: 'Select a User' },
  { type: entityKeys.role, value: null, label: 'Select a role' }
];

let user = placeholderItems[0];

entityKeys = Object.assign(entityKeys, {user: 'contact'}); // Compiler Error!!
```

### When to Use Let

`let` is the preferred declarator for most situations.

Many StyleGuides and linters prescribe the use of `const` always. This StyleGuide does _not_ encourage this pattern. Using `const` instead of `let` is left up to the discretion of the programmer. In most cases, however, `let` is recommended. 

For futher reading, refer to [this article](https://madhatted.com/2016/1/25/let-it-be).

Consider the following example. The `animalFactory` function uses `const` within it's scope to assign two different class instances. Because `const` variables cannot be reassigned, we need to make the decision using the "ternary" operator. In this case, it is simple:

```typescript
abstract class Animal {
  constructor( public fur:boolean,  private say:string ){}
  speak():void{
    console.log(this.say);
  }
};

class Dog extends Animal{
  constructor(){
    super(true, 'bark');
  }
}

class Snake extends Animal{
  constructor(){
    super(false, 'hiss');
  }
}

class Fish extends Animal{
  constructor(){
    super(false, 'glug');
  }
}

function animalFactory(isFurry: boolean):Animal{
  const result = isFurry ? new Dog() : new Snake();
  result.speak();
  return result;
}

let dog = animalFactory(true); // 'bark'
let snake = animalFactory(false); // 'hiss'

```

We have an unused class `Fish` that we want to add to our `animalFactory` function. Because `const` variables cannot be reassigned, a developer might introduce another ternary operator like so:

_Bad_
```typescript
function animalFactory(isFurry: boolean, hasLungs:boolean = true):Animal{
  const result = hasLungs ? isFurry ? new Dog() : new Snake() : new Fish();
  result.speak();
  return result;
}

let dog = animalFactory(true); // 'bark'
let snake = animalFactory(false); // 'hiss'
let fish = animalFactory(false, false) // 'glug';
```

Nesting the ternary operator like this is bad practice, and can quickly produce expressions that are difficult to understand and debug. If your assignment expression becomes complex when using `const`, always *prefer changing the expression to `let`*. Keeping your code readable is more important that cramming complex logic into single expressions in order to use `const`:

_Good_
```typescript
function animalFactory(isFurry: boolean, hasLungs:boolean = true):Animal{
  let result;
  if(hasLungs){
    result = isFurry ? new Dog() : new Snake();
  }else{
    result = new Fish();
  }
  result.speak();
  return result;
}

let dog = animalFactory(true); // 'bark'
let snake = animalFactory(false); // 'hiss'
let fish = animalFactory(false, false) // 'glug';
```

For this reason, `let` is generally prefferred _unless a variable is truly constant for the entire runtime of the application_.



## Anonymous Functions
Prefer arrow-function syntax over `function` syntax.

**Why?**

Arrow functions are shorter to define and _explicitly bind_ `this` to the current context. Consider the following snippet:

```typescript
class ContextObj{
  x: number = -1;
  constructor(){}
  fn(cb:(...args)=>any){
    cb.bind(this)();
  }
}

let obj = new ContextObj();

(function explicitContext(){

  obj.fn(()=>{
    console.log(this.x);
  }); // 5

  obj.fn(function(){
    console.log(this.x);
  }); // -1

}).bind({ // set the 'this' context explicitly for explicitContext
  x: 5
})();
```

In this snippet there are two `this` contexts, one inside of a `ContextObj` instance and another that we set manually in the `explicitContext` IIFE. Both calls to `ContextObj.fn` receive an anonymous function. The first is called using arrow function syntax, the second using `function` syntax.

Inside of `ContextObj.fn` the `ContextObj` instance re-binds the `this` context of the passed callback. As you can see, this only has an effect on the `function` anonymous function. The arrow-function callback inherits the `this` context of _the place where it was first defined_.

The reason why we prefer arrow-functions is because libraries, both internal and external, often will attempt to rebind `this`, which can cause unexpected behavior if you explicitly reference `this` in your callback. 

## Arrow Function Parameters
A lambda function with one parameter should not include parentheses. Similarly, a lambda with a single expression should not use block scoping.

_Bad_
```typescript
  let arr = [1, 2, 3];
  arr.forEach((x)=>{
    console.log(x);
  });
  arr.map((x)=>{
    return x+1;
  });
```

_Good_
```typescript
  let arr = [1, 2, 3];
  arr.forEach(x => console.log(x) );
  arr.map(x => x+1);
```

## Ternary Operator
Use of the ternary operator is useful in simple use cases, however it can be confusing to work with.

We should **never nest ternary expressions**. This makes code very hard to maintain.

Furthermore, ternary expressions should be **simple**. When in doubt, prefer `if...else` blocks or `switch` statements.

_Bad_
```typescript
function getProperty(x: number, y: number, obj: MyClass){
  return (x < 0) ? (y < 5 ? obj.a : obj.b) : (y < 5 ? obj.c : obj.d) ;
}

function getNum(x: number, override: number){
  return (x < 0) ? 5 : override || 100; // Using the || fall-through behavior here to set a default value is over complex when inside of a ternary already
}
```

_Good_
```typescript
function getProperty(x: number, y: number, obj: MyClass){
  if(x < 0){
    return y < 5 ? obj.a : obj.b;
  }else{
    return y < 5 ? obj.c : obj.d;
  }
}

function getNum(x: number, override: number){
  override = override || 100; // More explicit way of using || operator for default initialization
  return (x < 0) ? 5 : override;
}
```

## Loops
Unless there is a very strong use-case, avoid using `for`, `while`, and `for..in`.

Instead use language constructs like `forEach`, `map`, `some`, `reduce`, etc.

**Why?**

Functional array/iterator methods provided by the language are more declarative and explicit in what is being done with the iterable. They are also more succinct and cleaner than `for/while` loops.

_Bad_
```typescript
let arr = [1,2,3,4];
let sum = 0;

for(var i=0; i<arr.length; ++i){
  sum += arr[i];
}
for(var i=0; i<arr.length; ++i){
  console.log(arr[i]);
}
```

_Good_
```typescript
let arr = [1,2,3,4];
let sum = arr.reduce((acc,curr)=>(acc+curr), 0);
arr.forEach(val=>console.log(val));
```

## Programming to Interfaces

In TypeScript the convention is to generally _avoid_ programming to interfaces. Interfaces don't provide runtime benefits like they do in other programming languages (Like C# and Java), and it results in unnecessary impots/exports.

Since classes retain typing information it's encouraged to import classes directly instead of importing interfaces. For examples, see [the Interfaces section of the Angular styleguide](https://angular.io/guide/styleguide#interfaces).

### Using Interfaces

Interfaces are useful for joining disparate classes that share common properties. For example:

```typescript
class Dog{
  constructor(public id: number){}
  bark(){
    console.log('bark');
  }
}

class Cat{
  constructor(public id: number){}
}

class Car {
  constructor(public id: number){}
  numWheels():number{
    return 4;
  }
}

interface HasID{
  id:number;
}

function logID(val: HasID){
  console.log(val.id);
}

logID(new Dog(1)); // 1
logID(new Cat(2)); // 2
logID(new Car(3)); // 3
```

This allows for defining code that expects specific object signatures without needing to _explicitly extend interfaces in the actual classes_. TypeScript knows whether or not a given class instance will match the expected interface signature automatically.

For this purpose, you can alternatively declare a `type` that produces the same behavior:

```typescript
type HasID = {
  id:number;
}
```
