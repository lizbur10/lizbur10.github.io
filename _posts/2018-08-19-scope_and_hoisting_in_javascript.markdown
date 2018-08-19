---
layout: post
title:      "Scope & Hoisting in JavaScript"
date:       2018-08-19 22:15:35 +0000
permalink:  scope_and_hoisting_in_javascript
---


## Scope
Scope refers to the context within which any binding (i.e., variable, function or class) is accessible. With variables, the simplest example might be:

```
var x = 'some value';

function foo() {
     var y = 'some other value'
}

console.log(x); // 'some value'
console.log(y); // Uncaught ReferenceError: y is not defined
```

Functions in Javascript create their own scope (with one exception which I'll cover later in this post.) Therefore, because `y` is declared inside function `foo`, it cannot be accessed from outside `foo`. However, the program can go up the scope chain to find a variable declaration. When we move the `console.log`s *inside* `foo`, both values are logged: 

```
var x = 'some value';

function foo() {
    var y = 'some other value'
    console.log(x); // 'some value'
    console.log(y); // 'some other value'
}

```

This is because the program checks for `x` inside `foo` then, when it doesn't find it, looks in the enclosing scope (the global scope in this example) and finds it there. Variable `x` is globally scoped -- it can be accessed anywhere in the program -- while variable `y` is local to function `foo`. 

Forgetting to use the var keyword when you declare a variable changes things. Consider the variable `moo`: 

```
function moocow() {

    (function()
    {
        var cow = 1;
        moo = 3; 
    }())
    console.log(moo);  // 3
    console.log(cow); // ReferenceError: cow is not defined
}
```

One might think that both `console.logs` would return a reference error since both variables are inside the anonymous function, but that's not what happens if the variable declaration keyword is null. Instead, when the program gets to the line `moo = 3;`, it first looks for the variable declaration inside the current scope, then goes up the scope chain. If it still doesn't find it, it automatically declares `moo` as a global variable. This can sometimes create unexpected behavior, which is one of the reasons that ES6 provides some other options for declaring variables (discussed in the next section).

In summary, when using pre-ES6 JavaScript, a variable's scope is determined by two factors:
1. Where the variable is declared (inside a function or not)
2. The keyword used to declare it (`var` or `null`).

With ES6, the points above still hold, but the options for both factors have been expanded.


### Scope in ES6

In ES6, two more keywords were added that can be used to declare variables: `let` and `const`. Within the context of this discussion of scope, the main difference between the ES6 keywords and `var` is that the ES6 keywords are block-scoped rather than function-scoped. (I will discuss another of the differences in the hoisting section below.) To illustrate what this means:

```
function foo() {
   if (true) {
	    var x = "foo";
   }
   console.log(x); // "foo"
}
```

```
function foo() {
   if (true) {
	    const /* let */ x = "foo"; 
   }
   console.log(x); // ReferenceError: x is not defined
}
```

In the first version, when `foo()` is run, "foo" is logged. In the second version, where the variable is declared using the `const` keyword, `ReferenceError: x is not defined` is returned. The variables declared using the `var` keyword are not scoped to the `if` block, only to the enclosing `foo` function, so they are accessible to the `console.log`.  With variables declared using the ES6 keywords, on the other hand, anytime a variable is declared inside a block (i.e., inside curly brackets), the variable is local to that block and cannot be accessed outside it.

To summarize:

![](http://burtonux.com/flatiron_blog/scope_table.jpg)

## Hoisting
In Javascript, *all declarations* are hoisted, whether a variable, function, or class is being declared. When a program is run, the engine does an initial compilation pass before executing the code. In that compilation phase, it finds and "remembers" all the declarations. 

Consider the following: 

```
foo = 'bingo';
var foo;
console.log(foo); // 'bingo'
```

In this code, the 1st and 3rd lines are ignored in the compilation phase but the engine sees that `foo` has been declared in line 2 (i.e., it exists). Then, in the execution phase, the variable's value is assigned in the first line and then successfully logged. (Alert readers may recall, however, that the result would be exactly the same if the declaration in the second line wasn't there since JavaScript would declare it automatically.) 

By contrast: 

```
console.log(foo); // undefined
var foo = "bingo";
```

In this case, the variable declaration still happens in the compilation phase, but in the execution phase the `console.log` occurs before the assignment of the value, so the variable exists but its value is not yet defined. 

A somewhat more complex example:

```
var foo = 'global value'; 
  
(function() { 
  console.log(foo); // undefined 
  var foo = 'local value'; 
})();
```

Here, even though `foo` is declared and assigned in the global scope, the function creates its own scope so the `foo` variable inside the function is a completely separate variable. From there, the behavior is the same as in the previous example; it just happens inside the function `foo`. Specifically, the declaration of the local `foo` variable is hoisted to the top of the function but 'undefined' is logged because the variable assignment happens after the `console.log`. The implication is that a variable can be invoked before it is declared, but if its value isn't  assigned at the time it's declared there can be unexpected results.

### Hoisting in ES6

As with variable scope, some changes were made in ES6 to reduce the likelihood of variables behaving unexpectedly. While variables declared using `let` or `const` are hoisted just as those declared using `var` are (remember, all declarations are hoisted in Javascript), the difference is that variables declared using `var` are automatically initialized to a value of `undefined` and those declared using `let` or `const` are not. For all practical purposes, however, one can proceed as if `let` and `const` are *not* hoisted because the result is the same: in ES6, if a variable is invoked before it's declared (or if it isn't declared at all), a reference error is thrown and program execution stops.

## Scope and Hoisting with Functions
As mentioned earlier, scope and hoisting apply to declared functions just as they do to declared variables. The difference is there is no distinction between declaration and assignment as there is with variables. Therefore, a function can be called either before or after its declaration. However, this only applies to function *declarations*:

```
function foo() {
   <do something>
}
```

Function *expressions* are not hoisted. This is true for named function expressions:

```
const foo = function bar() {
  <do something>
}
```

anonymous function expressions:

```
const foo = function() {
  <do something>
}
```

Immediately-Invoked Function Expressions (IIFE):

```
(function() {
  <do something>
}())
```

and arrow functions (which can only be defined as expressions):

```
const foo = () =>  <do something>;
```

With all of these methods of defining functions, the function isn't loaded until the code is reached during the execution phase (i.e., they are not hoisted), so they cannot be invoked before they are defined. 

## this

Finally, a few words about `this`. As mentioned earlier, functions define their own scope. Sometimes this can cause problems. The following example (taken from the [MDN arrow function documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)) illustrates the issue:

```
function Person() {
  this.age = 0;

  setInterval(function growUp() {
    this.age++;
  }, 1000);
}

var p = new Person();
```

Here we have a constructor that sets an age property for a Person object and then uses `setInterval` to execute a callback function (`growUp`) to update the age property once every second. Unfortunately, the context of the `growUp` function is global so `this` no longer refers to the `Person` object and `this.age` is undefined.

There are a number of ways to fix the issue. One common approach is to create an additional variable and then increment that (`that`) instead:

```
function Person() {
   var that = this;
   that.age = 0;

   setInterval(function growUp() {
      that.age++;
   }, 1000);
}
```

This solution works, but it is not particularly elegant, and it makes the code a bit more opaque. 

Another option is to bind `this` to the `growUp` function:

```
function Person() {
  this.age = 0;

  setInterval(function growUp() {
    this.age++;
  }.bind(this), 1000);
}

var p = new Person();
```

Again, this works, but ES6 provides an even cleaner solution: arrow functions. Unlike the other ways of defining functions, arrow functions do not define their own scope. Instead, the enclosing `Person` object is the execution context, so `this` refers to the new `Person` object `p`. As a result, in the code below, the age of `p` is correctly incremented:

```
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; 
  }, 1000);
}

var p = new Person();
```

The solution is not only cleaner and more concise, but also more logical: it just seems natural in this context that `this` would be pointing to the object and not to the global context. Nice.


### Sources used for this post:

[Eloquent JavaScript, Chapter 3: Functions](https://eloquentjavascript.net/03_functions.html)

[Stack Overflow: Are variables declared with let or const not hoisted in ES6?](https://stackoverflow.com/questions/31219420/are-variables-declared-with-let-or-const-not-hoisted-in-es6#31222689)

[Stack Overflow: What is the difference between a function expression vs declaration in JavaScript? ](https://stackoverflow.com/questions/1013385/what-is-the-difference-between-a-function-expression-vs-declaration-in-javascrip#3344397)

[Ben Alman: Immediately-Invoked Function Expression](http://benalman.com/news/2010/11/immediately-invoked-function-expression/#iife)

[Arrow functions - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

[Jacob Worrel: ES6 Arrow Functions | What Not To Do](https://medium.com/@jacobworrel/es6-arrow-functions-what-not-to-do-c28c96b4f396)

