# Declarative arguments-resolver

0. [Overview](#overview)
    0. [Install](#install)
    0. [Example](#example)
0. [How to use](#how-to-use)
    0. [Declaration structure](#declaration-structure)
    0. [Errors](#errors)
    0. [Built-in types](#built-in-types)
    0. [Custom types](#custom-types)

## Overview

Decree is a declarative arguments-resolver. It saves you time and code when you
need to do arguments validation and disambiguation in your APIs.

Simply declare the conditions your arguments should hold, such as their types,
whether they are optional and their default values. Decree will take care of the
rest, and provide you with clean and disambiguated arguments.

If the user provided an illegal combination of arguments, Decree will throw an
exception and tell you where was the problem.

### Install

`npm install decree`

### Example

Let's say you have a function which takes 4 arguments, some are optional. Inside
the function you need to:

0. Verify the types of the arguments.
0. Detect which of the arguments were provided and which were omitted.
0. Disambiguate and assign default values to omitted arguments.

**Without Decree:**

```Javascript
/**
 * Make a cup of coffe.
 * @param {number} [sugars=1] - number of sugars. non-negative decimal number. defaults to 1.
 * @param {string} [flavor='bitter'] - flavor of the coffee. defaults to 'bitter'.
 * @param {string|integer} [size='large'] - size of cup. 'small', 'medium', 'large' or a positive integer.
 * @param {function} callback - called when coffee is ready.
 */
function makeCoffee(sugars, flavor, size, callback){
    // verify arguments:
    // was sugars provided? if not, flavor = sugars? size = flavor? callback = size?
    // but what if flavor was not provided...?
    // what about the callback? maybe callback = size? callback = flavor?
    // is 'callback' a function? if not, throw an exception?
    // ...
    // ...
    // ...
    // finally:
    if (/* arguments are valid */){
        // make coffee...
        callback('Coffee is ready!');
    } else {
        throw Error('Invalid arguments!');
    }
}
```

 **With Decree:**

Simply decalare the properties of your arguments:

```Javascript
var decs = [{
    name: 'sugars',
    type: 'nn-number', // non-negative
    optional: true,
    default: 1
}, {
    name: 'flavor',
    type: 'string',
    optional: true,
    default: 'bitter'
}, {
    name: 'size',
    types: ['string', 'p-int'], // string or positive integer
    optional: true,
    default: 'large'
}, {
    name: 'callback',
    type: 'function'
}];
```

Let Decree do the rest:

```Javascript
/**
 * Make a cup of coffe.
 * @param {number} [sugars=1] - number of sugars. non-negative decimal number. defaults to 1.
 * @param {string} [flavor='bitter'] - flavor of the coffee. defaults to 'bitter'.
 * @param {string|integer} [size='large'] - size of cup. 'small', 'medium', 'large' or a positive integer.
 * @param {function} callback - called when coffee is ready.
 */
function makeCoffee() {
    decree(decs)(arguments, function(sugars, flavor, size, callback) {
        // arguments are disambiguated and ready to be used.
        // make coffee...
        callback(size + ' coffee is ready! ' + flavor + ' with ' + sugars + ' sugars.');
    });
};
```

Now use your function as usual:

```Javascript
makeCoffe(1.5, function(msg){
    console.log(msg); // 'large coffee is ready! bitter with 1.5 sugars.'
});
```

On problems Decree will throw an exception:

```Javascript
try {
    makeCoffe(-1.5, function(msg){});
} catch (e) {
    // 'sugars' can't be negative...
}
```

## How to use

Decree needs to know what you expect. Simply build an array to describe your
arguments expectations.

```Javascript
// declarations:
var decs = [arg1, arg2, arg3, ...];
```

Each item in the array is an object which describes an argument. See
[declaration structure](#declaration-structure).

 When finished declaring your expectations, use Decree to resolve your
 function's arguments:

```Javascript
var decree = require('decree');
var decs = [ /* ... */ ];

function foo() {
    // pass your function's arguments directly to decree:
    decree(decs)(arguments, function(arg1, arg2, arg3, ...) {
        // here you can be sure your arguments are of
        // the correct types and values
    });
}

// use foo as normal:
foo( ... );
// if there is a problem with the provided arguments, an exception
// is thrown.
```

### Declaration structure

When declaring an argument, tell Decree:

0. `name {String}`: **Optional**. Will be used to identify the argument in error
   messages.
0. `type {String}` / `types{Array[String]}`: **Required**.
   See [built-in types](#built-in-types).
0. `optional {Boolean}`: **Optional**. Is this argument optional?
   Defaults to `false`.
0. `default`: **Optional**. If the argument is optional, this default value will
   be assigned if no value is provided.

```Javascript
{
    name: ...,
    types: [ ... ],
    optional: ...,
    default: ...
}
```

### Errors

When there is a problem with the arguments Decree can provide a detailed
explanation of what went wrong. By default, an error object will be thrown,
unless you provide a second callback which is called with the error.

```Javascript
var decree = require('decree');

// with an exception:
function foo() {
    try {
        decree(/* decs */)(arguments, function(/* args */) {
            // ...
        });
    } catch (err) {
        // if here, there was a problem with the arguments the user passed.
        // 'err' contains the information you need
    }
}

// or, with an error handling callback:
function foo() {
    decree(/* decs */)(arguments, function(/* args */) {
        // ...
    }, function(err) {
        // if here, there was a problem with the arguments the user passed.
        // 'err' contains the information you need
    });
}
```

### Built-in types

Decree supports several argument types:

- `*`: Argument matches any type.
- `array`
- `function`
- `hash`: Argument is a simple key-value object.
- `string`
- `number`
- `n-number`: Argument is a negative number
- `p-number`: Argument is a positive number
- `nn-number`: Argument is a non-negative number
- `np-number`: Argument is a non-positive number
- `int`: Argument is an integer
- `n-int`: Argument is a negative integer
- `p-int`: Argument is a positive integer
- `nn-int`: Argument is a non-negative integer
- `np-int`: Argument is a non-positive integer

### Custom types

Coming soon
