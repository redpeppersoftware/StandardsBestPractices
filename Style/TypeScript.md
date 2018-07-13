# Red Pepper TypeScript Styleguide
This document outlines the general guidelines for working with TypeScript at Red Pepper.


## Single Responsibility Principle
Single Responsibility (SRP) refers to the practice of restricting any single file/module to have a clear, distinct purpose. This makes the project easier to manage and work with, and prevents tight coupling in files.

## File and Function Size
Individual code files should generally not exceed 700 lines of code. This is a _code smell_ that indicates your code may be tightly coupled and be violating SRP.

Functions should generally not exceed 75 lines of code. If your functions exceed this, it is an indicator that your function may be violating SRP. Very large functions are also very difficult to understand, debug, and change.

## Column Length Limit
Columns should not exceed 80 characters in length. This allows for flexible editing regardless of indivudal developer Editor setup.

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

## Lambda Functions
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

## Lambda Function Parameters
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

## Avoid Programming to Interfaces
This goes against conventional language paradigms adopted by languages such as Java and C#, where you are encouraged to _always_ program to interfaces.

Interfaces make sense in TypeScript in the same way that inheritence makes sense; it encapsulates common typings and/or functionality to be shared between **two or more** classes.

Because programming to interfaces as a rule of thumb results in more code fragmentation and unnecessary imports, it's favorable to avoid interfaces until there is a shared need between **two or more** classes.

_Bad_
```typescript
/* Animal.ts */
export default interface Animal{
  say: ()=>string;
}

/* Dog.ts */
import Animal from './Animal';

export default class Dog implements Animal{
  constructor(){}
  say(){
    return 'Bark!';
  }
}
```

_Good (single class)_
```typescript
/* Dog.ts */
export default class Dog{
  constructor(){}
  say(){
    return 'Bark!';
  }
}
```

_Good (multiple classes)_
```typescript
/* Animal.ts */
export default interface Animal{
  say: ()=>string;
}

/* Dog.ts */
import Animal from './Animal';

export default class Dog implements Animal{
  constructor(){}
  say(){
    return 'Bark!';
  }
}

/* Cat.ts */
import Animal from './Animal';

export default class Cat implements Animal{
  constructor(){}
  say(){
    return 'Meow!';
  }
}
```

