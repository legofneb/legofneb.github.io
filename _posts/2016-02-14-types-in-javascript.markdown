---
layout: post
title:  "Types in Javascript"
date:   2016-02-14
categories: javascript
---

Javascript comes with a small set of standard types. Here are the common types:

~~~ javascript
// Boolean
let aBoolean = true;
    aBoolean = false;

// Numbers
let aNumber = 42;
    aNumber = 1.234;

// Strings
let aString  = 'this is a string';
    aString  = "This is also a string";

// Arrays
const animals = ['dog', 'cat', 'fish'];
const aMixOfTypes = [true, 42, 'some string'];

// Objects
const anObject = new Object();
const myDog   = { name: 'Luke', weight: 25 };
const Person = { name: 'Sam', runs: true };

// undefined
Person.runs = undefined;
console.log(Person.runs); // outputs: undefined

// null
Person.name = null;
console.log(Person.null); // outputs: null

~~~

You may be wondering why undefined and null exist. Undefined specifies that the property does not exist, while null specifies that the value has no valid value. While they may sound similar and redundant, they are used in different ways.
