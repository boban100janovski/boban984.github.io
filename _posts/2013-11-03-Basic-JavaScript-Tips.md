---
tags: javascript
title: Basic JavaScript Tips and Tricks
---
{% include toc %}

This blog post will be about some basic JavaScript you should know when beginning to write JS (it will save you a lot of time and headaches),
also i will provide some key differences between C# (same applies for similar languages like java) and JavaScript.

We will cover defining variables, dynamic typing, falsie values, default values, guards, determining type, and type coercion.
<!--more-->
My first professional encounter with programming was using .net platform, so my favorite language used to be C# and I still think it is one of the best programming languages out there.

After some time I started to get familiar with JavaScript mainly by using jQuery.

I found the language confusing and irritating at first, but as I got to know more and more about JavaScript it quickly become a pleasure and great fun to work with, now I can say that it\'s my new favorite programming language.

JavaScript is a lightweight, interpreted, object-oriented language with first-class functions, most known as the scripting language for Web pages, but used in non-browser environments as server-side programming (node.js) or Apache CouchDB, game development and the creation of desktop applications.

What this means that you can do the same thing in lots of different ways, the language offers freedom but you have to know what you are doing, so learning some basics at the beginning will save you a lot of pain later on.

JS is a prototype-based, multi-paradigm scripting language that is dynamic, is type safe, and supports object-oriented, imperative, and functional programming styles.

Let us see what is different between C# and JavaScript, these are not all differences just they key ones.

| C#  | JavaScript |
|---|---|
| Strongly-Typed | Loosely-Typed |
| Static | Dynamic |
| Classical Inheritance	| Prototypal |
| Classes | Functions |
| Constructors | Functions |
| Methods | Functions |

Note: C# does support dynamic in .net 4.5 by using the dynamic key word,
but most of the code is still written statically.

# Defining variables

## C\#

Car ferrari = new Car();

Here Car has a static structure with predefined properties, methods and events.
The variable someCar will always be of type Car so we can\'t reassign that variable to some other type.

`ferrari = new Bike() // Compilation Error`

## JavaScript

Types are now dynamic defined by their structure, checking is done at runtime.
If you would try to access some property that doesn\'t exist you would get a message like property not defined or similar.

``` javascript
var number = 1;
number = new Date();
```

Here you see that number can change the type.

# Dynamic Typing

``` javascript
var person = {
    name : "Bobi",
    age : "32"
}
```

Even though person is already defined as an object that has 2 properties name and age,
we can add new properties at any time.

``` javascript
person.phone = "070 333 444";

person.sayHi = function () {
    alert('Hi i am '+ this.name);
}
```

This is a powerful aspect of the language that allows you to grow your object.

This was strange for me at first, coming from C#.

# Falsie values in JavaScript

In JavaScript the following values are evaluated to false:

* false
* null
* undefined
* 0
* \"\" // Empty string
* Nan

Look at the following examples

``` javascript
//For example, these all evaluate to false.
console.log( null      ? true : false ); // false
console.log( 0         ? true : false ); // false
console.log( ""        ? true : false ); // false
console.log( false     ? true : false ); // false
console.log( NaN       ? true : false ); // false
console.log( undefined ? true : false ); // false

//using the not operator ! on these will evaluate true
console.log( !null      ); // true
console.log( !0         ); // true
console.log( !""        ); // true
console.log( !false     ); // true
console.log( !NaN       ); // true
console.log( !undefined ); // true

//But all the values are not equal to false
console.log( null == false      ); // false
console.log( 0 == false         ); // true
console.log( "" == false        ); // true
console.log( false == false     ); // true
console.log( NaN == false       ); // false
console.log( undefined == false ); // false

//For null == false , problem here is type coercion ,typeof null is object and typeof false is boolean.
//There is no coercion under which objects of the null and boolean types can be equal, which is why this is false.
//For NaN == and == return false always even NaN == NaN is false, to check if a value is NaN use isNaN(value)
//For undefined it evaluates to false in boolean operations. Always use === when comparing to undefined.

//If we use === the results are more what you would expect , cause there is no type coercion
console.log( null === false      ); // false
console.log( 0 === false         ); // false
console.log( "" === false        ); // false
console.log( false === false     ); // true
console.log( NaN === false       ); // false
console.log( undefined === false ); // false
```

[code on jsfiddle](http://jsfiddle.net/boban984/E7trA/)

Lets say we have a variable `name = "Bobi"`, that is of type string

In C# you would test a string value like this: `if (name != null && name.length > 0) or name != ""`

C# has a simple method for checking all these conditions `!string.IsNullOrEmpty(name)`, you shouldn\'t test like this is JavaScript.

In JavaScript you can check for undefined, null, empty string and the other false values easily like : `if(name) { ... }`

# Default Values and Guards

Using the false value checking we can easily set default values.

The `&& and ||` operators are called short\-circuit operators.

They will return the value of the second operand based on the value of the first operand, handy for guards and defaults.

The && operator is useful for checking for null objects before accessing their attributes.

Bad

``` javascript
if (name == null) {
    name = "default value";
}
```

Still Bad

``` javascript
name = name ? name : "default value";
```

Good

``` javascript
name = name || "default value";
```


Guard against **null** object using &&, this way we avoid trying to access the property

``` javascript
var name = obj && obj.name
```

# Determining type

JavaScript has 4 data types:

* boolean
* number
* string
* object

Special values

* null
* undefined
* false

We can determine the type using `typeof` and `instanceof` commands:

``` javascript
typeof x or typeof(x) //The first one is most commonly used

x instanceof constructor
```

example:

``` javascript
typeof undefined           // "undefined"
typeof 0                    // "number"
typeof true                 // "boolean"
typeof "foo"                // "string"
typeof new String("foo")    // "object"
typeof {}                   // "object"
typeof []                   // "object"
typeof new Date             // "object"

typeof null                 // "object"
typeof function(){}         // "function"
typeof NaN                  // "number"
```

The last three checks are quirks in the language null is not a real object, you cant do:

``` javascript
var foo = null
foo.one = 1 // error, can't assign a property to primitive
```

Functions are objects in JavaScript despite of their special treatment.

The `typeof NaN == 'number'` is a bit strange because `NaN` stands for \'Not-A-Number\'

The typeof operator works well for primitive types but it can\'t differentiate objects.

``` javascript
// Common sample usage of typeof
function f(x) {
 if (typeof x == 'function') {
   ... // when x is a function
 } else {
   ... // in other cases
 }
}

//Bad usage
if (typeof($) !== 'undefined')

//The typeof is used here because if we check the variable directly we will get an error
// if($) {...} will throw an error
```

A global variable can be accessed as a property of built-in window object. Let\'s check it:

`if (window.jQuery !== undefined) { ... }`

No error here cause i am looking for the object property.

We know that jQuery is not falsie value so the check can be written as:

`if(window.jQuery) {...}`

typeof shouldn\'t be used to check for variable or property existence

How do we check custom objects?

We can do that using the instanceof operator, like so:

``` javascript
function Person(name){
  this.name = name
}

var guy = new Person('Bobi');
guy instanceof Person //returns true

// The following code uses instanceof to demonstrate that
// String and Date objects are also of type Object (they are derived from Object).

var myString = new String();
var myDate = new Date();

myString instanceof String; // returns true
myString instanceof Object; // returns true
myString instanceof Date;   // returns false

myDate instanceof Date;     // returns true
myDate instanceof Object;   // returns true
myDate instanceof String;   // returns false
```

So in a few words, `typeof` use it for primitives and functions (wrong about null),
instanceof good for custom objects and native ones too.

# Type Coercion

``` javascript
if ([0]) {
    [0] == true; //false
    !![0]; //true
}

if ("someString") {
    "someString" == false; //false
    "someString" == true; //false
}
```

Type coercion in JavaScript can be confusing, but it\'s just a feature of the language, we need to understand it not be afraid of it.

Coercion happens when we use the `==` sign.

**The comparison `x == y`, where x and y are values, produces true or false.**

The construct `if ( Expression ) Statement` will coerce the result of evaluating the Expression to a boolean using the abstract method ToBoolean

lets take a look at the way ECMA defines how `==` works


Type(x) | Type(y) | Result
--------|---------|-------
x and y are the same type | | same as === , no coercion
null | undefined | true
Number | String | x == toNumber(y)
Boolean | (any) | toNumber(x) == y
String or Number | Object | x == toPrimitive(y)
Other cases | | False

**ToNumber**

Argument Type | Result
--------------|-------
Undefined | false
Null | false
Boolean | The result equals the input argument (no conversion).
Number | The result is false if the argument is +0, −0, or NaN; otherwise the result is true.
String | The result is false if the argument is the empty String (its length is zero); otherwise the result is true.
Object | true.

**ToPrimitive**

Argument Type | Result
--------------|-------
Object | (in the case of equality operator coercion) if valueOf returns a primitive, return it. Otherwise if toString returns a primitive return it. Otherwise throw an error
otherwise… | The result equals the input argument (no conversion).

Examples:

``` javascript
[0] == true;

//what's going on
//convert boolean using toNumber
[0] == 1;
//convert object using toPrimitive
//[0].valueOf() is not a primitive so use...
//[0].toString() -> "0"
"0" == 1;
//convert string using toNumber
0 == 1; //false!
```

``` javascript
"someString" == true;

// what's going on

// convert boolean using toNumber
"someString" == 1;
// convert string using toNumber
NaN == 1; //false!
```

``` javascript
"someString" == false;

// what's going on 

// convert boolean using toNumber
"someString" == 0;

// convert string using toNumber
NaN == 0; //false!
```

``` javascript
numObj = new Number(1);
numObj == 1;

// what's going on convert object using toPrimitive
// valueOf returns a primitive so use it
1 == 1; //true!
```

Nice tool to see how coercion works: [http://jscoercion.qfox.nl/](http://jscoercion.qfox.nl/)

Good blog post if u want to know more : 
[http://webreflection.blogspot.com/2010/10/javascript-coercion-demystified.html](http://webreflection.blogspot.com/2010/10/javascript-coercion-demystified.html)

That's it for now guys, hope you liked my blog post, I would appreciate any feedback, write some comments in the form bellow.