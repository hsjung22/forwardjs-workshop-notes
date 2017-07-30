# Forwardjs workshop notes

*Notes from "Object Oriented Programming with Javascript" by Elie Schoppik*

## Table of Contents
  1. [Intro](#intro)
  1. [Closure](#closure)
  1. [Implicit binding](#implicit-binding)
  1. [Explicit binding](#explicit-binding)
  1. [Call](#call)
  1. [Apply](#apply)
  1. [Bind](#bind)
  1. [New](#new)


## Intro

  When a function is invoked, two variables get created automatically `this` and `arguments`.

  ```javascript
  function first() {
    return this
  }

  first()
  // ‘this‘, 'arguments' are set when a function is invoked
  // ‘this’  refers to Global object.
  // In browser, this === window
  ```

  ```javascript
  function first() {
    this.job = "Instructor";
    this.firstName = "Elie";
  }
  first()
  // => undefined
  // 'this' inside the function refers to the global object
  // resulting in unintentionally setting global variables.


  "use strict"
  function first() {
    // because we are in 'strict' mode, when inside a function, 'this' is undefined
    this.isHilarious = true
  }
  first()
  // => TypeError. 'this' is undefined.
  ```

## Closure

Closure is when a function has access to the scope of its outer function that has already returned.

  - Example 1

    ```javascript
    function outer() {
      var instructor = "Elie"
      return function inner() {
        return "Hello " + instructor;
      }
    }
    outer()
    // => returns the inner function
    outer()() // invokes inner()
    // => "Hello Elie"
    ```

  - Example 2

    Only variables USED in inner function is remembered in memory even after outer is done executed. Therefore only the `instructor` variable is stored.

    ```javascript
    function outer() {
      var instructor = "Elie"
      var funFact = "Play Cello!"
      return function inner() {
        return "Hello " + instructor;
      }
    }
    ```

  - Example 3

    ```javascript
    var count = 0
    function increment() {
      return ++count;
    }
    increment()
    ```

    The problem with this is that the count is a global variable. We need a way to make the count value more private.

    ```javascript
    function counter() {
      var count = 0;
      return function inner() {
        return ++count;
      }
    }    
    ```

  - Example 4

    ```javascript
    function classRoom() {
      var instructors = ["Elie"]
      return {
        getInstructors() {
          return instructors;
        },
        addInstructor(person) {
          instructors.push(person);
          return instructors;
        }
      }
    }

    var forward = classRoom()
    forward.addInstructor('Matt')
    // => ["Elie", "Matt"]
    ```


## Implicit binding

`this` gets assigned to the closest 'parent' object.

  - Example 1

    ```javascript
    var obj = {
      firstName: "Elie",
      whoami: function() {
        return this;
      }
    }
    obj.whoami() === obj
    // => true
    ```

  - Example 2

    ```javascript
    var obj = {
      firstName: "Elie",
      whoami: this
    }
    obj.whoami === window
    // => true
    ```

  - Example 3

    ```javascript
    var obj = {
      firstName: "Elie",
      moreInfo: {
        homeState: "NJ",
        displayInfo: function() {
          return this.firstName + " is from " + this.homeState
        }
      }
    }
    obj.moreInfo.displayInfo()
    // => "undefined is from NJ"
    // 'this' inside the displayInfo function refers to the 'moreInfo' object
    // this.firstName does not exists and so returns undefined
    ```

  - Example 4

    ```javascript
    var obj = {
      firstName: "Elie",
      homeState: "NJ",
      displayInfo: function() {
          return this.firstName + " is from " + this.homeState;
      }
    }
    obj.moreInfo.displayInfo()
    // => "Elie is from NJ"
    ```

## Explicit binding

```javascript
(function(){}).call
(function(){}).apply
(function(){}).bind
// call apply bind has to be applied to a function
```

### call

Immediately invokes the function it is attached to.
`thisArg` is used to explicitly set the value of keyword `this`.
The `arg1, arg2, ...` parameters of call are the parameters of the `function` that is immediately invoked

Syntax:
```javascript
function.call(thisArg, arg1, arg2, ...)
```

  - Example 1

    ```javascript
    var instructor = {
      firstName: "Elie",
      sayHi: function() {
        return "Hello " + this.firstName;
      }
    }

    var instructor2 = {
      firstName: "Matt"
    }
    ```

    ```javascript
    instructor.sayHi().call(instructor2)
    // => TypeError
    // sayHi is already invoked and returns a string.
    // "".call is not a function
    ```

    ```javascript
    instructor.sayHi.call(instructor2)
    // => "Hello Matt"
    // 'this' inside sayHi function now refers to 'instructor2'
    ```

    We can separate out the `sayHi` function from instructor variable.
    This makes `sayHi` function available to any object that has a firstName.

    ```javascript
    var instructor = {
      firstName: "Elie"
    }

    var instructor2 = {
      firstName: "Matt"
    }

    function sayHi() {
      return "Hello " + this.firstName;
    }
    ```

    ```javascript
    sayHi()
    // => "Hello undefined"
    sayHi.call(instructor)
    // => "Hello Ellie"
    sayHi.call(instructor2)
    // => "Hello Matt"
    ```

  - Example 2

    ```javascript
    function doMath(a,b,c) {
      return `${this.firstName} adds ${a+b+c}`
    }

    var instructor = {
      firstName: "Elie"
    }
    ```

    ```javascript
    doMath(1,2,3)
    // => "undefined adds 6"

    doMath.call(instructor)
    // => "Elie adds NaN"

    // 'call' accepts infinite parameters
    doMath.call(instructor, 1, 2, 3)
    // => "Elie adds 6"  
    ```

### apply

Same with `call`, it immediately invokes the function it is attached to. Difference being it only accepts 2 arguments. The first being `thisArg`, used to explicitly set the value of keyword `this`. The second argument being an array-like object that specifies the arguments with which function should be called.

Syntax:
```javascript
function.apply(thisArg, [argsArray])
```

  - Example 1

    ```javascript
    function doMath(a,b,c) {
      return `${this.firstName} adds ${a+b+c}`
    }

    var instructor = {
      firstName: "Elie"
    }
    ```

    ```javascript
    doMath.apply(instructor, [1,2,3])
    // => "Elie adds 6"
    ```

  - Example 2

    ```javascript
    var numbers = [4,1,6,10,9,3,2]
    Math.max(numbers)
    // => NaN;
    // Math.max() doesn't accept array, only comma separated
    Math.max.apply(this, numbers)
    // => 10
    // we don't care what 'this' is, then pass in numbers (array)
    // 'apply' turns array into comma separated
    ```

  - Example 3

    ```javascript
    function addNumbers(a,b,c) {
      return a+b+c
    }
    ```

    ```javascript
    var nums = [5,10,15]
    addNumbers(nums)
    // => "5,10,15undefinedundefined"
    addNumbers.apply(this, nums)
    // => 30

    // es6 spread notation
    addNumbers(...nums)
    // => 30
    ```

### bind

Same with `call`, `thisArg` is used to explicitly set the value of keyword `this`. The `arg1, arg2, ...` are arguments to prepend to arguments given to the bound `function`. The big difference is that bind returns a function. Which means it is not immediately invoked like `call` or `apply`.

Syntax:
```javascript
function.bind(thisArg, arg1, arg2, ...)
```

  - Example 1

    ```javascript
    function doMath(a,b,c) {
      return `${this.firstName} adds ${a+b+c}`
    }
    var instructor = {
      firstName: "Elie"
    }
    var partial = doMath.bind(instructor, 10)
    partial(5,10,15)
    // => "Elie adds 25"
    // a = 10, b = 5, c = 10; 15 gets ignored
    ```

  - Example 2

    ```javascript
    var obj = {
      firstName: "Elie",
      lastName: "Schoppik",
      sayHi: function() {
        setTimeout(function() {
          console.log(`Hi ${this.firstName} ${this.lastName}`)
        }.bind(this), 1000)
      }
    }
    obj.sayHi()
    ```

    callback to SetTimeout, the keyword this is global (window). Thats why we need to explicitly set `this` with `bind`

    Es6 solution:

    ```javascript
    var obj = {
      firstName: "Elie",
      lastName: "Schoppik",
      sayHi() {
        setTimeout(() => {
          console.log(`Hi ${this.firstName} ${this.lastName}`)
        }, 1000)
      }
    }
    ```
    With arrow functions, the value of `this` is based on the function's surrounding context

  - Building our own `bind`

    ```javascript
    function add(a,b,c) {
      return `${this.firstName} adds ${a + b + c}`
    }
    var instructor = { firstName: "Elie" }

    myBind(add, instructor, 1)(2, 3) // Elie adds 6
    myBind(add, instructor)(1, 2, 3) // Elie adds 6
    myBind(add, instructor, 1, 2, 3)() // Elie adds 6
    myBind(add, instructor, 1, 2, 3)(4, 5, 6) // Elie adds 6
    ```

    Solution:
    ```javascript
    function myBind(fn, thisArg) {
      var outerArgs = [].slice.call(arguments,2)
      return function inner() {
        // turning array-like object into an array
        var innerArguments = [].slice.call(arguments)
        var allArguments = outerArgs.concat(innerArguments)
        // through closure, this inner function has in memory fn and thisArgs
        return fn.apply(thisArg, allArguments)
      }
    }
    ```

    Es6 solution:
    ```javascript
    function myBind(fn, thisArg, ...outerArgs) {
      return function inner(...innerArgs) {
        return fn.apply(thisArg, [...outerArgs, ...innerArgs])
      }
    }
    ```

### new

  - Constructor function

    Person function is a constructor function that will be used to create new Person objects. It acts as a "blueprint." Constructor functions are the closest thing to classes in JavaScript. The constructor function controls the setting of data on the objects that will be created. By convention, a constructor function starts with a capital letter.

    ```javascript
    function Person(firstName) {
      this.firstName = firstName;
    }
    ```

    The constructor function is called with the `new` keyword.

    1. creates a new empty object
    1. sets `this` to be that new object just created
    1. implicitly returns `this`
    1. internal link gets created between the object and the prototype property on the constructor function. This link is accessible through `__proto__`

    ```javascript
    var elie = new Person("Elie")
    ```

  - Prototype chain

    Every function has a property called 'prototype', and every 'prototype' has a 'constructor' property that points back to the original function. `__proto__` is a link that connects the object created to its prototype property on the constructor

    ```javascript
    Person.prototype.constructor === Person
    // => true
    elie.__proto__ === Person
    // => false
    elie.__proto__ === Person.prototype
    // => true
    ```

    ```javascript
    "awesome".includes('a')
    // => true
    "awesome".__proto__ === String.prototype
    // includes actually exists in String.prototype
    "awesome".__proto__.includes
    "awesome".includes
    // It does it by prototype chain.
    ```

  - Example 1

    Create a constructor function for a Person

    ```javascript
    function Person(firstName, lastName) {
      this.firstName = firstName
      this.lastName = lastName
      this.fullName = function() {
        return `${this.firstName} ${this.lastName}`
      }
    }
    ```

    ```javascript
    var elie = new Person('Elie', 'Schoppik')
    elie.fullName()

    var tim = new Person('Tim', 'Garcia')
    tim.fullName()

    var matt = new Person('Matt', 'Lane')
    matt.fullName()
    ```

    The problem is that `fullName` function has to be re-created each time a new Person object is created. If there are multiple Person objects created, each of them will have the same copy of `fullName` function. Since the function does not need to be unique per Person instance, it can be optimized by using prototype. Any function or property added to the prototype is "inherited" among all instances linked to that prototype.

    Prototype:

    ```javascript
    function Person(firstName, lastName) {
      this.firstName = firstName
      this.lastName = lastName
    }

    Person.prototype.fullName = function() {
      return `${this.firstName} ${this.lastName}`
    }
    ```
    Now the `fullName` function is defined on a prototype and all Person objects "inherit" that prototype so as to gain all those functions at very little runtime cost.

  - Inheritance

    Parent:
    ```javascript
    function Person(firstName, lastName) {
      this.firstName = firstName
      this.lastName = lastName
    }

    Person.prototype.fullName = function() {
      return `${this.firstName} ${this.lastName}`
    }
    ```

    Child:
    ```javascript
    function Student() {
      // "inherit" properties from the parent
      Person.apply(this, arguments)
    }
    // inherit functions
    Student.prototype = Object.create(Person.prototype)
    // reset the constructor
    Student.prototype.constructor = Student

    var s = new Student('Elie', 'Schoppik')
    ```

  - Es6 Syntax

    ```javascript
    class Person {
      constructor(firstName, lastName) {
        this.firstName = firstName
        this.lastName = lastName
      }
      fullName() {
        return `${this.firstName} ${this.lastName}`
      }
      static isPerson(obj) {
        return obj.constructor === Person
      }
    }
    ```

    ```javascript
    var elie = new Person('Elie', 'Schoppik')
    elie.fullName()
    // => "Elie Schoppik"
    elie.__proto__ === Person.prototype
    // => true
    Person.prototype.fullName
    // function fullName
    Person.isPerson(elie)
    // => true
    ```

  - Es6 Inheritance

    ```javascript
    class Person {
      constructor(firstName, lastName) {
        this.firstName = firstName
        this.lastName = lastName
      }
      fullName() {
        return `${this.firstName} ${this.lastName}`
      }
    }

    class Student extends Person {
      constructor() {
        super(...arguments)
        this.isInSchool = true
      }
    }
    var s = new Student('Elie', 'Schoppik')
    ```
