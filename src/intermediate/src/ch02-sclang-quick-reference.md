# Chapter 2: sclang quick reference

sclang is not a hard language to pick up if you're familiar with the likes of
JavaScript, Python, Ruby, or Lua. Its object model and syntax are derived from
Smalltalk.

We're just going to breeze through sclang basics here. This isn't meant as a
complete language spec, but just enough information so you can get on
comfortable footing for writing programs in SC.

Since I'm showing you around a whole gaddang programming language in this
chapter, it's going to get very long and very dull. As such, this probably
isn't the most exciting place to start — the beginner's guide is much better if
you haven't yet written a line of code. But once you start writing more complex
programs and need a quick reference on how to do basic programming tasks, this
is the chapter for you.

## Running code in sclang

For beginning users who come to SC from other languages, the standard method of
running code in SC from the IDE can be pretty weird and offputting. As a
refresher, you write SuperCollider code in `.scd` files, which have a bunch of
code blocks hanging out in space. Rather than running the whole thing at once,
individual lines are run using Shift+Enter, and blocks of code delimited by
parentheses are run using Ctrl+Enter (Cmd+Enter on macOS).

Weird? Yes. But it's less weird once you realize that the **IDE is a glorified
REPL**, just like the Python interactive interpreter. It's just that REPL is
the most common way of interacting with SuperCollider.

It's actually possible — and may be preferable — to run sclang like an ordinary
program from the command line. This can be done by running

    sclang my_file.scd

and it will work fine. The IDE equivalent is to fire up the interpreter, open
up `my_file.scd`, and select "Language → Evaluate File."

Your file needs to be set up in a special way, however. The parentheses used to
delimit code blocks for interactive execution will not work as expected, and
will usually cause syntax errors. Think of the entire file as a single
parenthetical code block. [also discuss asynchronous operations]

Why use REPL in the IDE if sclang can mostly be run as a normal programming
language? Although the workflow is unusual compared to programming
environments, it's not without precedent when viewed as artistic audio
software. In your average modular synth software or hardware, the "work
interface" and the "performance interface" are the same. Same for SC: you write
your code and you run your code in the same place.

One obvious reason to use the IDE is if you incorporate live coding into your
work — it's a fully capable live coding interface, and can even be augmented
with custom behavior specifically for that practice. But even if you don't, the
IDE provides several user interface benefits, such as showing you a CPU meter
for the server (very important to ensure code is suitable for live performance)
and giving you an easy interface for volume control and recording to disk.

If the whole REPL thing is weird for you, I understand, but I assure you that
you will get used to it. It's all part of SC being live performance software.

## Comments

```supercollider
// two slashes,

/*
or multiline style.
*/

/* Multiline comments /* DO */ nest. */
```

## Blocks & variables

When you execute code in between parentheses, execute code from a single line,
or execute an entire file, that's a block. The final expression in a block is
its return value. When you execute a block in the IDE, the post window prints
out the block's return value, preceded by a `->`.

```supercollider
(
1;
2;
3;
)
// -> 3
```

You can declare local variables using `var`. They're lexically scoped, so they
behave similarly to a lot of your favorite languages. 

```supercollider
(
var foo = 3, bar;
var baz = 50, quux = foo + baz;
quux;
)
// -> 53
```

All `var` statements have to go at the top of the block, and can't be
interspersed with other code. (I know, it sucks.) An unassigned variable is
given a value of `nil`.

A variable name always starts with a *lowercase* Latin letter, then any number
of characters in `a-z A-Z 0-9 _`. The reserved keywords are quite few:

    var arg classvar const pi true false nil inf

These identifiers are valid variable names, but I wouldn't recommend them,
because then you won't have access to the actual builtins in the local scope:

    this super thisProcess thisFunction thisFunctionDef thisMethod thisThread
    currentEnvironment topEnvironment

Assigning to variables outside of `var` statements is done in the regular C
way, like `foo = 3;`. Operators like `+=` are *not* supported in the spirit of
minimalist Smalltalk syntax.

Variables are scoped to the block they are in, and they cease to exist outside
of the block. How do we share information between blocks then?

## Environment variables and interpreter globals

sclang doesn't have global variables. Instead, sclang has a syntactic feature
known as "environment variables" (not to be confused with the OS-level
environment variables) that provides a substitute for global scope. These are
notated with the `~` prefix.

```supercollider
~foo = 3;
~foo // -> 3
```

These variables persist across blocks. However, what they really supply is
*dynamic scoping*, and under some conditions they don't behave exactly like
globals. Just know that these tilde variables are close enough to globals that
they are useful for passing stuff between blocks.

I lied — sclang does actually have global variables, but it has exactly 26 of
them: the letters `a` through `z`. I wish I were kidding.

```supercollider
x = 3;
x // -> 3
```

I strongly advise against using these in any production code. They're okay when
you're doing nonserious tests, but please don't use them in real compositions.

## Method calls

In sclang, pretty much every useful operation is performed by doing method
calls on objects. The object we're calling on is referred to as the *receiver*,
and it can take any number of arguments. The following are the most common
syntaxes for method calls:

```supercollider
receiver.doSomething(1, 2, 3);
doSomething(receiver, 1, 2, 3);
```

The second one looks like a function call, but it's not. `doSomething` is
actually a method name, and not referring to any variable in the local scope.

Method calls may take keyword arguments:

```supercollider
receiver.doSomething(foo: 1, bar: 2, baz: 3);
```

If no arguments are provided, the parentheses may be omitted (as they
conventionally are):

```supercollider
"Hello World".postln();
"Hello World".postln;
```

## Mathematics

I'm sure you can guess what the `nil`, `true`, and `false` keywords are.
Integers (32-bit signed) and floats (64-bit) are notated in pretty obvious
ways:

```supercollider
0 -1 1443 0x07 3.0 4.0 1e8 1e-5
```

If a decimal point is provided, there must be at least one digit on each side.
Expressions like `.5` and `3.` are syntax errors. This avoids some ambiguous
parses.

Your favorite arithmetic operators work as expected. But unlike many other
languages, **infix operator precedence is always left to right**. The order of
operations is not respected. These are identical:

```supercollider
3 + 5 * 4
(3 + 5) * 4
```

You're probably groaning right now, but it's internally consistent and not
unheard of in language design. One good reason for it is that sclang supports
operator overloading, allowing you to define your own arbitrary operators like
`@|@<>@|@`. How would we define the order of operations for those?

I expect you'll find these operators useful (they are syntactic sugar for
method calls):

```supercollider
+ - / * ** == != < > <= >=
```

`/` always graduates to a float even if both of its arguments are integers.
Same for the exponentiation operator `**`.

**There is no unary `-`**. When you write `-1`, it's actually a single token.
To do unary negation on variables and other things that aren't numeric
literals, use the `neg` method.

These are good math functions to have under your belt — you can figure them out
easy:

```supercollider
asInteger asFloat floor ceil round abs sgn
sin cos tan tanh
```

## Functions

Functions are denoted with curly braces. They are first-class objects. The
return value is simply the final line, and the function is called using the
`.value` method:

```supercollider
var myFunction = {
    someOtherObject.doSomething;
    3
};
myFunction.value
// -> 3
```

`myFunction.value` can be shortened to `myFunction.()`, but not `myFunction()`.

Function arguments are denoted using this funky pipe-delimited section:

```supercollider
var myFunction = { |foo, bar, baz = 5|
    foo + bar + baz;
};
myFunction.(3, 4);
// -> 12
```

There is an older way of writing this: `{ arg foo, bar, baz = 5; ... }`, but
it's going out of fashion.

Like in JavaScript, functions not strict about variable arguments. Unspecified
arguments with no explicit default value are set to `nil`. If too many
arguments are provided, they are simply ignored.

## Conditionals

Boolean operators in sclang are a little different from other languages.

```supercollider
(0 == 1).not  // -> false

// Equivalent:
true.and(false)
true and: false

// The second is known as "key binary operator" syntax, and can be used in the
// special case where only one argument is passed to the method.

// 'and' and 'or' require their second argument to be wrapped in a function
// for short circuiting to occur:
doThisThing.() or: { doThatThing.() }
```

In keeping with sclang's minimal syntax, control structures are just method
calls involving functions.

```supercollider
(
if(2 + 2 == 4, {
    "Nice!".postln;
}, {
    "What?".postln;
});
)

// Shortcut syntax that may be used because all arguments are functions.

(
if(2 + 2 == 4) {
    "Nice!".postln;
} {
    "What?".postln;
};
)

// 'if' produces a return value from the function/block that was run, so it
// actually doubles as a ternary operator.
if(2 + 2 == 4, { "Nice" }, { "What" }).postln;

// 'case' is a quick way to write a bunch of nested 'if's. The first condition
// that's true is evaluated.
case
    { x < 0 } { "It's negative".postln; }
    { x > 1000 } { "It's nonnegative, but too big".postln; }
    { x % 2 == 0 } { "It's an even number".postln; };

// 'switch' is very similar to ordinary switch statements. Like 'if' and
// 'case', it passes along the return value of the function that was actually
// evaluated.
switch(thing,
    \option1, { "Option 1" },
    \option2, { "Option 2" }
).postln;

// You can wrap the values in functions — saves a few commas.
switch(thing)
    { \option1 } { "Option 1".postln }
    { \option2 } { "Option 2".postln };
```

sclang is strict about only Booleans being placed in `if`, `not`, `and`, and
`or`. If you try something like `if(0, ...)`, you will receive an error that
says "Non Boolean in test."

`if`, `switch`, and `case` are *not* keywords. They are methods.

## Arrays

Creation, getting, and setting are the usual. Zero-indexed.

```supercollider
(
var myArray = [1, 3, 4, 4, 3];
myArray[0] = myArray[3];
myArray[1] = "hey";
myArray;
)
// -> [ 4, hey, 4, 4, 3 ]

// NOTE: getting and setting indices are actually syntactic sugar for the ".at"
// and ".put" methods.
```

Retrieving an out-of-bounds index returns `nil`. Setting an out-of-bounds index
produces an error.

Here are a few useful ways to create arrays:

```supercollider
(0..4)    // -> [ 0, 1, 2, 3, 4 ], inclusive of both endpoints
(1,3..10) // -> [ 1, 3, 5, 7, 9 ]
4.dup(8)  // -> [ 4, 4, 4, 4, 4, 4, 4, 4 ]
{ |i| i ** 2 }.dup(8)   // -> [ 0, 1, 4, 9, 16, 25, 36, 49 ]
```

sclang supports multichannel expansion. Binary and unary operators, when
applied to arrays, are broadcast to the array's elements:

```supercolliders
[1, 2, 3, 4] + 4   // -> [ 5, 6, 7, 8 ]
[3, 4, 5] * [1, 6, 8]  // -> [ 3, 24, 40 ]
```

This sounds small, but it becomes unbelievably convenient when working with
multichannel sound design.

Appending to arrays is done using the `.add` method. However, for efficiency
**`.add` may or may not be in place**. To avoid unexpected behavior, always
assign the return value of `.add` back to the slot for the original array:

```supercollider
(
var myArray = [1, 2, 3];
myArray = myArray.add(4);
)
// -> [1, 2, 3, 4]
```

Slices are denoted with `..` and are inclusive of both indices:

```supercollider
(
var myArray = [1, 5, 2, 0, 4, 4];
myArray[2..4]
)
// -> [ 2, 0, 4 ]
```

The following methods are also useful — I am certain you can figure out how
they work from their names and very brief examples, and if not, you can consult
the documentation:

```supercollider
["con", "cat"] ++ ["en", "a", "tion"]
newArray = array.copy
array.indexOf(thing)
array.includes(thing)
["heads", "tails"].choose
array.scramble
```

## Iteration

Imperative `for` and `while` loops are relatively rare in idiomatic sclang
code. Instead, take advantage of sclang's rich variety of methods for iterating
and filtering arrays. Here is a very terse summary of them.

```supercollider
// Prints numbers 0 through 7 inclusive.
8.do { |i| "i".postln; };

// Do something for each element. index is optional.
array.do { |elem| ... };
array.do { |elem, index| ... };

// Do something for each element and make a new array from the return values.
// Often called "map" in other environments
var newArray = array.collect { |elem| ... };

// Filter the array and keep only elements for which the given function returns
// true. The function should only return Booleans.
var newArray = array.select { |elem| elem % 2 == 0 };
```

## Strings and symbols

```supercollider
"Strings are delimited with double quotes."
"Strings can be
multiline."
"Escape characters: \t \f \v \n \r \\"
```

sclang will tolerate non-ASCII bytes in strings, but they are just bytes, and
Unicode isn't really understood. The SCIDE displays UTF-8, so if you're
careful, non-ASCII characters will come out unscathed.

Symbols are similar to strings, but are efficient when reused, and not so
efficient for textual operations. A symbol is therefore ideal for identifiers
that are only meaningful within your program, while a string is intended for
actual text input, manipulation, and output.

```supercollider
'Symbols are delimited by single quotes'

// But if the symbol is a valid identifier, there's a much simpler syntax:
\validIdentifier2000
```

Here are some useful methods — any ambiguities can be cleared up by consulting
the docs:

```supercollider
"string".asSymbol
\symbol.asString
"string".size
"concaten" ++ "ation"
"with" + "space" + "in" + "between"
"You can format %.".format("strings")
```
