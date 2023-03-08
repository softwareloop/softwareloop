# Javascript idioms for Java developers

(by Paolo Predonzani, originally posted on February 9, 2015)

Being proficient in a programming language usually means being able to write
good code in that language, but I think it is just as important to be able to
read code written by others, especially by the experts in that language.

In this post I've collected a number of JS idioms, recurrent "phrases" and
patterns that I've found while peeking in the source code of the several
JavaScript libraries that I use. Some idioms derive directly from the language,
others are best practices that have become widespread.

I have a Java background so this list reflects what I feel as interesting or
unusual about JavaScript code.

## Functions are values

In JS, functions are values and can be assigned to variables, just like an
integer or a string can be assigned to a variable. The variable that holds the
function can be passed around and be invoked somewhere else in the program.

```javascript
var f = function(x, y) {
    return x+y;
};
f(2, 3); // returns 5
```

The interesting thing is that when we invoke `f(2, 3)`, "f" could be any
function that takes two parameters as an argument. Consider this other example:

```javascript
var md5 = function(text) {
    /* computes and returns an md5 hash of text */
};

var sha1 = function(text) {
    /* computes and returns an sha1 hash of text */
};

var f;
if (/* some condition*/) {
    f = md5;
} else {
    f = sha1;
}

var result = f("some text");
```
    

In the last line, when "f" is invoked, we don't know if the variable points to
"md5" or to "sha1" - it depends on the condition in the "if" statement. All we
know is that the function takes an argument and returns a value (presumably of
compatible types). This resembles implementation substitution and polymorphism
in Java.

## Inner functions

JavaScript has function scope, meaning that variables declared inside a function
are not visible outside the function.

```javascript
function outer() {
    var x = 1;
    var inner = function() {
        console.log(x); // Ok: inner can see outer's variables
    }
    console.log(x); // Ok
    inner(); // Ok
}
outer(); // Ok
console.log(x); // Error: x is not defined
inner(); // Error: inner is not defined
```
    

This provides a way to hide data and functions inside a container function,
effectively implementing "private" members which otherwise would not be
possible.

## Closures

What if, in the previous example, "outer" decides to expose "inner" by returning
it as its return value?

```javascript
function outer() {
    var x = 1;
    var inner = function() {
        console.log(x);
    };
    return inner;
}
var f = outer(); // Now f point to inner
f(); // Prints 1, the value of inner variable x
```

The surprising thing here is that when we call `f()` to invoke `inner`,
`outer()` has already returned and completed its execution, but `inner` is still
able to access a private variable of `outer`.

When an inner function is passed around as a value, the function retains access
to the variables of its outer function, even if the outer function has already
returned. This kind of variable visibility is called `closure` and has many
useful applications, especially in ajax callbacks:

```javascript
function f() {
    var x = 2;
    $.ajax({
        url: "test.html"
    }).done(function() {
        /* access x as closure */
    });
}
```
    

## Object literals

In Java we think of objects as instances of a certain class, so classes come
first and objects come second. But in JavaScript objects are basically key-value
maps and can be defined without the need for a class.

```javascript
var account = {
    name: "John Doe",
    balance: 0,
    currency: "USD"
}
```

Since functions are values, properties can be functions too:

```javascript
var account = {
    name: "John Doe",
    balance: 0,
    currency: "USD",
    deposit: function(amount) {
        this.balance = this.balance + amount;
    }
}
```

Simple objects with properties and methods, but without a class!

## document or window.document?

You'll see both used to refer to the document's DOM, but is there a difference?
Nope, they refer to exactly the same thing and this is not specific to
"document" only. It applies to all global variables. Let's try with "a":

```javascript
var a = "hello";
console.log(a); // prints "hello"
console.log(window.a); // prints "hello", too!
```

We discover that global variables are accessible as properties of a global
object which, in case of a browser environment, is the window object.

## Namespaces

If global variables and global functions are used a lot, the global object
becomes a crowded place and the possibility of name clashes arises.

To mitigate this risk, library developers are encouraged to define only one
global variable with a unique name and then define functions and variables as
properties of the global variable. JQuery, with its "$" or "jQuery" global
variable, is a clear example.

In other code you'll find:

```javascript
var mylib = {};
mylib.mypackage = {};
mylib.mypackage.f = function() {/*code*/};
```

This defines:

*   a "mylib" namespace as a global variable
*   a "mypackage" package inside "mylib"
*   an "f" function inside "mypackage"

See this YUI function for example:

```javascript
YAHOO.lang.isString(value)
```
    

So in JavaScript namespaces/packages are not native language features, but can
be achieved using plain objects.

More frequently, especially if the library is made of several files, the
namespace will be defined using the following idiom:

```javascript
var mylib = mylib || {};
mylib.mypackage = mylib.mypackage || {};
```
    

... which checks if the namespace/package already exists before creating it.

## Inline function calls

Instead of writing:

```javascript
var a = 0;
// other statements that use "a"
```
    

we could write:

```javascript
(function() {
    var a = 0;
    // other statements that use "a"
})();
```

We've wrapped the statements in a function and we've immediately invoked that
function. This turns "a" from a global variable to a local one and uses an
anonymous function as a wrapper. This technique, called "inline function call",
contributes to reducing the pollution of the global object.

## Modularisation

To take inline function calls one step further, we sometimes find that code
like this:

```javascript
$.ajax(/* details */);
```

is turned into this:

```javascript
(function($) {
    $.ajax(/* details */);
})($);
```
    

There is a subtle difference. While in the first fragment "$" is accessed as
global variable, in the second fragment the body of the function uses "$" as a
function argument.

But why on earth is this useful? The answer is that the body of the function is
made independent of global variables. In fact, if jQuery was available not as
"$" but as "jQuery" we could write:

```javascript
(function($) {
    $.ajax(/* details */);
})(jQuery);
```

...without changing the body of the function.

Effectively the body of the function becomes an isolated module:

*   it does not **read** global variables (relies on arguments instead)
*   it does not **write** global variables (writes to local variables only)

Modularisation is a big topic in modern JavaScript. In the early days, the
global namespace was handy for small programs but, as programs became larger,
the global namespace turned into a burden and the need arose to break large code
in small modules, as well as to handle dependencies between modules.

For further details see how [RequireJS](http://www.requirejs.org/docs/whyamd.html) and [CommonsJS](http://wiki.commonjs.org/wiki/CommonJS) deal with the subject.

## Constructors and prototype inheritance

In JavaScript it turns out that a function - any function - can be used as a
constructor to create new objects:

```javascript
function Account() {this.balance = 0};
var obj = new Account(); // creates a new object
```

The new object is bound to the function that created it via the "constructor"
property:

```javascript
obj.constructor == Account; // true
```
    

Notice that the capital first letter in the function name is a convention, not
a requirement.

Also, every function has a "prototype" property. It's an empty object that is
automatically attached to the function when the function is defined:

```javascript
Account.prototype; // An empty object {}
```
    

Instances created using the function point to the prototype via the
`__proto__` property:

```javascript
obj.__proto__ == Account.prototype; // true
obj.__proto__ == obj.constructor.prototype; // same as above
```    

Notice that if the constructor is invoked many times, all the created objects
will share the same prototype:

```javascript
var obj2 = new Account();
obj2.__proto__ == obj.__proto__; // true
```
    

An instance object and its prototype work together in a mechanism called
prototype inheritance which, for many practical purposes, is similar to the
relationship between an object and its class in Java. I won't go into the
details of the mechanism. I'll just point you to an [excellent book](https://www.packtpub.com/web-development/object-oriented-javascript-second-edition) by
Stoyan Stefanov and Kumar Chetan Sharma that explains it.

What matters is that a common idiom in JavaScript code is to enhance the prototype:

```javascript
Account.prototype.deposit = function(amount) {
    this.balance += amount;
}
```
    

Working with constructors, prototypes and complex prototype hierarchies (I
didn't mention them but they're possible) is tedious, so many JS frameworks help
to deal with the lower-level details with a simpler, higher-level API. However,
occasionally you'll find code that works at the lower level - the following
section shows an example.

## Enhancing built-ins

JS has a number of built-in data types such as Number, String, Array and Object.
Each type has built-in utility methods. For example String has "indexOf":

```javascript
"The quick brown fox".indexOf("quick"); // 4
```
    
In Java, basic data types cannot be modified or extended (many classes are
declared final), but in JavaScript built-in types can be enhanced with
additional methods.

Often this is used to "equalise" the JS implementation of different browsers
to a common set of features. The following example adds the "trim" method on
browsers that don't implement it natively:

```javascript
if (!String.prototype.trim) {
    String.prototype.trim = function() {
        return this.replace(/^\s+|\s+$/g, '');
    };
 }
```
    

## This and that

If you have:

```javascript
obj.aFunction()
```
    

then "obj" will be available in "aFunction" as variable "this", much like in any
object-oriented language.

There's a catch though - "this" does not work as expected in closures. A common
workaround is to assign "this" to a variable with a different name, e.g. "that":

```javascript
var obj = {
    message: "Hello!",
    outer: function() {
        console.log(this.message); // ok
        var that = this;
        console.log(this === that); // true
        function inner() {
            console.log(this === that); // false
            console.log(this === window); // true
            console.log(this.message); // undefined
            console.log(that.message); // "Hello!"
        }
        inner();
    }
};
obj.outer();
```

JS frameworks offer various approaches to ensure "this" is propagated in
closures. For example, Dojo has [hitch](http://dojotoolkit.org/reference-guide/1.10/dojo/_base/lang.html#dojo-base-lang-hitch).

## Reflection

Since JS objects are hash maps, reflection is a simple matter of enumerating
their properties. JS has the for..in statement just for this purpose. Inside the
loop, a test is usually added to exclude properties inherited from the object's
prototype:

```javascript
var obj = {
    name: "John",
    surname: "Doe"
};
for (var propertyName in obj) {
    if (obj.hasOwnProperty(propertyName)) {
        var propertyValue = obj[propertyName];
        console.log(propertyName, "=", propertyValue);
    } 
}
```

## Mixins

There are no classes in JavaScript so objects are free to have any 
properties/methods they like. Suppose you have some interesting
properties/methods on "obj1" and you'd like to make them available on "obj2"
too. The easiest approach is to copy the properties/methods from "obj1" to
"obj2". In this way "obj1" is used as a mixin for "obj2".

JS frameworks usually provide support for mixins but if you want to roll your
own, it's not difficult:

```javascript
function mixin(source, destination) {
    for (var propertyName in source) {
        if (source.hasOwnProperty(propertyName)) {
            destination[propertyName] = source[propertyName];
        } 
    }
}
var obj1 = {
    name: "John",
    surname: "Doe",
    sayHello: function() {console.log("Hello!")}
};
var obj2 = {};

mixin(obj1, obj2)
obj2.name; // "John"
obj2.surname; // "Doe"
obj2.sayHello(); // "Hello!"
```

Notice that mixins can be applied multiple times:

```javascript
mixin(obj1, destination);
mixin(obj2, destination);
mixin(obj3, destination);
```
    

Mixins, for many practical purposes, resemble multiple inheritance, aspects
and traits.

## Constructor args vs. mixins

Those with a Java background tend to write constructors with many arguments to
initialise an object's properties, e.g.:

```javascript
function Square(x, y, width, height) {
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
}
```
    
But in JavaScript the following pattern is common:

*   constructor with one argument: a mixin to apply 
*   default property values in the prototype.

For example:

```javascript
function Square(obj) {
    mixin(obj, this);
}

Square.prototype.x = 0;
Square.prototype.y = 0;
Square.prototype.width = 0;
Square.prototype.height = 0;

var obj = new Square({width: 10, height: 3});
obj.x; // 0
obj.y; // 0
obj.width; // 10
obj.height; // 3
```
    

This is an efficient way of handling optional arguments and default values in
the constructor.

