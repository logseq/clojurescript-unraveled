# About this book

This book covers the ClojureScript programming language, serves as a
detailed guide of its tooling for development, and presents a series of
articles about topics that are applicable to day-to-day programming in
ClojureScript.

It is not an introductory book to programming in that it assumes the
reader has experience programming in at least one language. However, it
doesn’t assume experience with *Clojure* or functional programming.
We’ll try to include links to reference material when talking about
the theoretical underpinnings of ClojureScript that may not be familiar
to everyone.

Since the ClojureScript documentation is good but sparse, we wanted to
write a compendium of reference information and extensive examples to
serve as a ClojureScript primer as well as a series of practical
how-to’s. This document will evolve with the ClojureScript language,
both as a reference of the language features and as a sort of cookbook
with practical programming recipes.

You’ll get the most out of this book if you:

  - are curious about ClojureScript or functional programming and have
    some programming experience;

  - write JavaScript or any other language that compiles to it and want
    to know what ClojureScript has to offer;

  - already know some Clojure and want to learn how ClojureScript
    differs from it, plus practical topics like how to target both
    languages with the same code base.

Don’t be turned off if you don’t see yourself in any of the above
groups. We encourage you to give this book a try and to give us feedback
on how we can make it more accessible. Our goal is to make ClojureScript
more friendly to newcomers and spread the ideas about programming that
Clojure has helped popularize, as we see a lot of value in them.

This is a list of translations of the book:

  - Ukrainian: <https://lambdabooks.github.io/clojurescript-unraveled/>

# Introduction

> Why are we doing this? Because Clojure *rocks*, and JavaScript
> *reaches*.
> 
> —  Rich Hickey 

*ClojureScript* is an implementation of the Clojure programming language
that targets JavaScript. Because of this, it can run in many different
execution environments including web browsers, Node.js, io.js and
Nashorn.

Unlike other languages that intend to *compile* to JavaScript (like
TypeScript, FunScript, or CoffeeScript), ClojureScript is designed to
use JavaScript like bytecode. It embraces functional programming and has
very safe and consistent defaults. Its semantics differ significantly
from those of JavaScript.

Another big difference (and in our opinion an advantage) over other
languages is that Clojure is designed to be a guest. It is a language
without its own virtual machine that can be easily adapted to the
nuances of its execution environment. This has the benefit that Clojure
(and hence ClojureScript) has access to all the existing libraries
written for the host language.

Before we jump in, let us summarize some of the core ideas that
ClojureScript brings to the table. Don’t worry if you don’t understand
all of them right now, they’ll become clear throughout the book.

  - ClojureScript enforces the functional programming paradigm with its
    design decisions and idioms. Although being strongly opinionated
    about functional programming it’s a pragmatic language rather than
    pursuing theoretical purity.

  - Encourages programming with immutable data, offering highly
    performant and state of the art immutable collection
    implementations.

  - It makes a clear distinction of identity and its state, with
    explicit constructs for managing change as a series of immutable
    values over time.

  - It has type-based and value-based polymorphism, elegantly solving
    the expression problem.

  - It is a Lisp dialect so programs are written in the programming
    language’s own data structures, a property known as *homoiconicity*
    that makes metaprogramming (programs that write programs) as simple
    as it can be.

These ideas together have a great influence in the way you design and
implement software, even if you are not using ClojureScript. Functional
programming, decoupling of data (which is immutable) from the operations
to transform it, explicit idioms for managing change over time and
polymorphic constructs for programming to abstractions greatly simplify
the systems we write.

> We can make the same exact software we are making today with
> dramatically simpler stuff — dramatically simpler languages, tools,
> techniques, approaches.
> 
> —  Rich Hickey 

We hope you enjoy the book and ClojureScript brings the same joy and
inspiration that has brought to us.

# Language (the basics)

This chapter will be a little introduction to ClojureScript without
assumptions about previous knowledge of the Clojure language, providing
a quick tour over all the things you will need to know about
ClojureScript and understand the rest of this book.

You can run the code snippets in the online interactive repl:
<http://www.clojurescript.io/>

## First steps with Lisp syntax

Invented by John McCarthy in 1958, Lisp is one of the oldest programming
languages that is still around. It has evolved into many derivatives
called dialects, ClojureScript being one of them. It is a programming
language written in its own data structures — originally lists enclosed
in parentheses — but Clojure(Script) has evolved the Lisp syntax with
more data structures, making it more pleasant to write and read.

A list with a function in the first position is used for calling a
function in ClojureScript. In the example below, we apply the addition
function to three arguments. Note that unlike in other languages, `+` is
not an operator but a function. Lisp has no operators; it only has
functions.

``` clojure
(+ 1 2 3)
;; => 6
```

In the example above, we’re applying the addition function `+` to the
arguments `1`, `2` and `3`. ClojureScript allows many unusual characters
like `?` or `-` in symbol names, which makes it easier to read:

``` clojure
(zero? 0)
;; => true
```

To distinguish function calls from lists of data items, we can quote
lists to keep them from being evaluated. The quoted lists will be
treated as data instead of as a function call:

``` clojure
'(+ 1 2 3)
;; => (+ 1 2 3)
```

ClojureScript uses more than lists for its syntax. The full details will
be covered later, but here is an example of the usage of a vector
(enclosed in brackets) for defining local bindings:

``` clojure
(let [x 1
      y 2
      z 3]
  (+ x y z))
;; => 6
```

This is practically all the syntax we need to know for using not only
ClojureScript, but any Lisp. Being written in its own data structures
(often referred to as *homoiconicity*) is a great property since the
syntax is uniform and simple; also, code generation via
[macros](#macros-section) is easier than in any other language, giving
us plenty of power to extend the language to suit our needs.

## The base data types

The ClojureScript language has a rich set of data types like most
programming languages. It provides scalar data types that will be very
familiar to you, such as numbers, strings, and floats. Beyond these, it
also provides a great number of others that might be less familiar, such
as symbols, keywords, regexes (regular expressions), vars, atoms, and
volatiles.

*ClojureScript* embraces the host language, and where possible, uses the
host’s provided types. For example: numbers and strings are used as is
and behave in the same way as in JavaScript.

### Numbers

In *ClojureScript*, numbers include both integers and floating points.
Keeping in mind that *ClojureScript* is a guest language that compiles
to JavaScript, integers are actually JavaScript’s native floating points
under the hood.

As in any other language, numbers in *ClojureScript* are represented in
the following ways:

``` clojure
23
+23
-100
1.7
-2
33e8
12e-14
3.2e-4
```

### Keywords

Keywords in *ClojureScript* are objects that always evaluate to
themselves. They are usually used in [map data
structures](#maps-section) to efficiently represent the keys.

``` clojure
:foobar
:2
:?
```

As you can see, the keywords are all prefixed with `:`, but this
character is only part of the literal syntax and is not part of the name
of the object.

You can also create a keyword by calling the `keyword` function. Don’t
worry if you don’t understand or are unclear about anything in the
following example; [functions](#function-section) are discussed in a
later section.

``` clojure
(keyword "foo")
;; => :foo
```

#### Namespaced keywords

When prefixing keywords with a double colon `::`, the keyword will be
prepended by the name of the current namespace. Note that namespacing
keywords affects equality comparisons.

``` clojure
---
::foo
;; => :cljs.user/foo
```

(= ::foo :foo) ;; ⇒ false ---

Another alternative is to include the namespace in the keyword literal,
this is useful when creating namespaced keywords for other namespaces:

``` clojure
---
:cljs.unraveled/foo
;; => :cljs.unraveled/foo
---
```

The `keyword` function has an arity-2 variant where we can specify the
namespace as the first parameter:

``` clojure
---
(keyword "cljs.unraveled" "foo")
;; => :cljs.unraveled/foo
---
```

### Symbols

Symbols in *ClojureScript* are very, very similar to **keywords** (which
you now know about). But instead of evaluating to themselves, symbols
are evaluated to something that they refer to, which can be functions,
variables, etc.

Symbols start with a non numeric character and can contain alphanumeric
characters as well as \*, +, \!, -, \_, ', and ? such as :

``` clojure
sample-symbol
othersymbol
f1
my-special-swap!
```

Don’t worry if you don’t understand right away; symbols are used in
almost all of our examples, which will give you the opportunity to learn
more as we go on.

### Strings

There is almost nothing new we can explain about strings that you don’t
already know. In *ClojureScript*, they work the same as in any other
language. One point of interest, however, is that they are immutable.

In this case they are the same as in JavaScript:

``` clojure
"An example of a string"
```

One peculiar aspect of strings in *ClojureScript* is due to the
language’s Lisp syntax: single and multiline strings have the same
syntax:

``` clojure
"This is a multiline
      string in ClojureScript."
```

### Characters

*ClojureScript* also lets you write single characters using Clojure’s
character literal syntax.

``` clojure
\a        ; The lowercase a character
\newline  ; The newline character
```

Since the host language doesn’t contain character literals,
*ClojureScript* characters are transformed behind the scenes into single
character JavaScript strings.

### Collections

Another big step in explaining a language is to explain its collections
and collection abstractions. *ClojureScript* is not an exception to this
rule.

*ClojureScript* comes with many types of collections. The main
difference between *ClojureScript* collections and collections in other
languages is that they are persistent and immutable.

Before moving on to these (possibly) unknown concepts, we’ll present a
high-level overview of existing collection types in *ClojureScript*.

#### Lists

This is a classic collection type in languages based on Lisp. Lists are
the simplest type of collection in *ClojureScript*. Lists can contain
items of any type, including other collections.

Lists in *ClojureScript* are represented by items enclosed between
parentheses:

``` clojure
'(1 2 3 4 5)
'(:foo :bar 2)
```

As you can see, all list examples are prefixed with the `'` char. This
is because lists in Lisp-like languages are often used to express things
like function or macro calls. In that case, the first item should be a
symbol that will evaluate to something callable, and the rest of the
list elements will be function arguments. However, in the preceding
examples, we don’t want the first item as a symbol; we just want a list
of items.

The following example shows the difference between a list without and
with the preceding single quote mark:

``` clojure
(inc 1)
;; => 2

'(inc 1)
;; => (inc 1)
```

As you can see, if you evaluate `(inc 1)` without prefixing it with `'`,
it will resolve the `inc` symbol to the **inc** function and will
execute it with `1` as the first argument, returning the value `2`.

You can also explicitly create a list with the `list` function:

``` clojure
(list 1 2 3 4 5)
;; => (1 2 3 4 5)

(list :foo :bar 2)
;; => (:foo :bar 2)
```

Lists have the peculiarity that they are very efficient if you access
them sequentially or access their first elements, but a list is not a
very good option if you need random (index) access to its elements.

#### Vectors

Like lists, **vectors** store a series of values, but in this case, with
very efficient index access to their elements, as opposed to lists,
which are evaluated in order. Don’t worry; in the following sections
we’ll go in depth with details, but at this moment, this simple
explanation is more than enough.

Vectors use square brackets for the literal syntax; let’s see some
examples:

``` clojure
[:foo :bar]
[3 4 5 nil]
```

Like lists, vectors can contain objects of any type, as you can observe
in the preceding example.

You can also explicitly create a vector with the `vector` function, but
this is not commonly used in ClojureScript programs:

``` clojure
(vector 1 2 3)
;; => [1 2 3]

(vector "blah" 3.5 nil)
;; => ["blah" 3.5 nil]
```

#### Maps

Maps are a collection abstraction that allow you to store key/value
pairs. In other languages, this type of structure is commonly known as a
hash-map or dict (dictionary). Map literals in *ClojureScript* are
written with the pairs between curly braces.

``` clojure
{:foo "bar", :baz 2}
{:alphabet [:a :b :c]}
```

<div class="note">

Commas are frequently used to separate a key-value pair, but they are
completely optional. In *ClojureScript* syntax, commas are treated like
spaces.

</div>

Like vectors, every item in a map literal is evaluated before the result
is stored in a map, but the order of evaluation is not guaranteed.

#### Sets

And finally, **sets**.

Sets store zero or more unique items of any type and are unordered. Like
maps, they use curly braces for their literal syntax, with the
difference being that they use a `#` as the leading character. You can
also use the `set` function to convert a collection to a set:

``` clojure
#{1 2 3 :foo :bar}
;; => #{1 :bar 3 :foo 2}
(set [1 2 1 3 1 4 1 5])
;; => #{1 2 3 4 5}
```

In subsequent sections, we’ll go in depth about sets and the other
collection types you’ve seen in this section.

## Vars

*ClojureScript* is a mostly functional language that focuses on
immutability. Because of that, it does not have the concept of variables
as you know them in most other programming languages. The closest
analogy to variables are the variables you define in algebra; when you
say `x = 6` in mathematics, you are saying that you want the symbol `x`
to stand for the number six.

In *ClojureScript*, vars are represented by symbols and store a single
value together with metadata.

You can define a var using the `def` special form:

``` clojure
(def x 22)
(def y [1 2 3])
```

Vars are always top level in the namespace ([which we will explain
later](#namespace-section)). If you use `def` in a function call, the
var will be defined at the namespace level, but we do not recommend this
- instead, you should use `let` to define variables within a function.

## Functions

### The first contact

It’s time to make things happen. *ClojureScript* has what are known as
first class functions. They behave like any other type; you can pass
them as parameters and you can return them as values, always respecting
the lexical scope. *ClojureScript* also has some features of dynamic
scoping, but this will be discussed in another section.

If you want to know more about scopes, this [Wikipedia
article](http://en.wikipedia.org/wiki/Scope_\(computer_science\)) is
very extensive and explains different types of scoping.

As *ClojureScript* is a Lisp dialect, it uses the prefix notation for
calling a function:

``` clojure
(inc 1)
;; => 2
```

In the example above, `inc` is a function and is part of the
*ClojureScript* runtime, and `1` is the first argument for the `inc`
function.

``` clojure
(+ 1 2 3)
;; => 6
```

The `+` symbol represents an `add` function. It allows multiple
parameters, whereas in ALGOL-type languages, `+` is an operator and only
allows two parameters.

The prefix notation has huge advantages, some of them not always
obvious. *ClojureScript* does not make a distinction between a function
and an operator; everything is a function. The immediate advantage is
that the prefix notation allows an arbitrary number of arguments per
"operator". It also completely eliminates the problem of operator
precedence.

### Defining your own functions

You can define an unnamed (anonymous) function with the `fn` special
form. This is one type of function definition; in the following example,
the function takes two parameters and returns their average.

``` clojure
(fn [param1 param2]
  (/ (+ param1 param2) 2.0))
```

You can define a function and call it at the same time (in a single
expression):

``` clojure
((fn [x] (* x x)) 5)
;; => 25
```

Let’s start creating named functions. But what does a *named function*
really mean? It is very simple; in *ClojureScript*, functions are
first-class and behave like any other value, so naming a function is
done by simply binding the function to a symbol:

``` clojure
(def square (fn [x] (* x x)))

(square 12)
;; => 144
```

*ClojureScript* also offers the `defn` macro as a little syntactic sugar
for making function definition more idiomatic:

``` clojure
(defn square
  "Return the square of a given number."
  [x]
  (* x x))
```

The string that comes between the function name and the parameter vector
is called a *docstring* (documentation string); programs that
automatically create web documentation from your source files will use
these docstrings.

### Functions with multiple arities

*ClojureScript* also comes with the ability to define functions with an
arbitrary number of arguments. (The term *arity* means the number of
arguments that a function takes.) The syntax is almost the same as for
defining an ordinary function, with the difference that it has more than
one body.

Let’s see an example, which will explain it better:

``` clojure
(defn myinc
  "Self defined version of parameterized `inc`."
  ([x] (myinc x 1))
  ([x increment]
   (+ x increment)))
```

This line: `([x] (myinc x 1))` says that if there is only one argument,
call the function `myinc` with that argument and the number `1` as the
second argument. The other function body `([x increment] (+ x
increment))` says that if there are two arguments, return the result of
adding them.

Here are some examples using the previously defined multi-arity
function. Observe that if you call a function with the wrong number of
arguments, the compiler will emit an error message.

``` clojure
(myinc 1)
;; => 2

(myinc 1 3)
;; => 4

(myinc 1 3 3)
;; Compiler error
```

<div class="note">

Explaining the concept of "arity" is out of the scope of this book,
however you can read about that in this [Wikipedia
article](http://en.wikipedia.org/wiki/Arity).

</div>

### Variadic functions

Another way to accept multiple parameters is defining variadic
functions. Variadic functions are functions that accept an arbitrary
number of arguments:

``` clojure
(defn my-variadic-set
  [& params]
  (set params))

(my-variadic-set 1 2 3 1)
;; => #{1 2 3}
```

The way to denote a variadic function is using the `&` symbol prefix on
its arguments vector.

### Short syntax for anonymous functions

*ClojureScript* provides a shorter syntax for defining anonymous
functions using the `#()` reader macro (usually leads to one-liners).
Reader macros are "special" expressions that will be transformed to the
appropriate language form at compile time; in this case, to some
expression that uses the `fn` special form.

``` clojure
(def average #(/ (+ %1 %2) 2))

(average 3 4)
;; => 3.5
```

The preceding definition is shorthand for:

``` clojure
(def average-longer (fn [a b] (/ (+ a b) 2)))

(average-longer 7 8)
;; => 7.5
```

The `%1`, `%2`…​ `%N` are simple markers for parameter positions that
are implicitly declared when the reader macro will be interpreted and
converted to a `fn` expression.

If a function only accepts one argument, you can omit the number after
the `%` symbol, e.g., a function that squares a number: `#(* %1 %1))`
can be written `#(* % %))`.

Additionally, this syntax also supports the variadic form with the `%&`
symbol:

``` clojure
(def my-variadic-set #(set %&))

(my-variadic-set 1 2 2)
;; => #{1 2}
```

## Flow control

*ClojureScript* has a very different approach to flow control than
languages like JavaScript, C, etc.

### Branching with `if`

Let’s start with a basic one: `if`. In *ClojureScript*, the `if` is an
expression and not a statement, and it has three parameters: the first
one is the condition expression, the second one is an expression that
will be evaluated if the condition expression evaluates to logical true,
and the third expression will be evaluated otherwise.

``` clojure
(defn discount
  "You get 5% discount for ordering 100 or more items"
  [quantity]
  (if (>= quantity 100)
    0.05
    0))

(discount 30)
;; => 0

(discount 130)
;; => 0.05
```

The block expression `do` can be used to have multiple expressions in an
`if` branch. [`do` is explained in the next section](#block-section).

### Branching with `cond`

Sometimes, the `if` expression can be slightly limiting because it does
not have the "else if" part to add more than one condition. The `cond`
macro comes to the rescue.

With the `cond` expression, you can define multiple conditions:

``` clojure
(defn mypos?
  [x]
  (cond
    (> x 0) "positive"
    (< x 0) "negative"
    :else "zero"))

(mypos? 0)
;; => "zero"

(mypos? -2)
;; => "negative"
```

Also, `cond` has another form, called `condp`, that works very similarly
to the simple `cond` but looks cleaner when the condition (also called a
predicate) is the same for all conditions:

``` clojure
(defn translate-lang-code
  [code]
  (condp = (keyword code)
    :es "Spanish"
    :en "English"
    "Unknown"))

(translate-lang-code "en")
;; => "English"

(translate-lang-code "fr")
;; => "Unknown"
```

The line `condp = (keyword code)` means that, in each of the following
lines, *ClojureScript* will apply the `=` function to the result of
evaluating `(keyword
code)`.

### Branching with `case`

The `case` branching expression has a similar use as our previous
example with `condp`. The main differences are that `case` always uses
the `=` predicate/function and its branching values are evaluated at
compile time. This results in a more performant form than `cond` or
`condp` but has the disadvantage that the condition value must be
static.

Here is the previous example rewritten to use `case`:

``` clojure
(defn translate-lang-code
  [code]
  (case code
    "es" "Spanish"
    "en" "English"
    "Unknown"))

(translate-lang-code "en")
;; => "English"

(translate-lang-code "fr")
;; => "Unknown"
```

## Truthiness

This is the aspect where each language has its own semantics (mostly
wrongly). The majority of languages consider empty collections, the
integer 0, and other things like this to be false. In *ClojureScript*,
unlike in other languages, only two values are considered as false:
`nil` and `false`. Everything else is treated as logical `true`.

Jointly with the ability to implement the callable protocol (the `IFn`,
explained more in detail later), data structures like sets can be used
just as predicates, without need of additional wrapping them in a
function:

``` clojure
(def valid? #{1 2 3})

(filter valid? (range 1 10))
;; => (1 2 3)
```

This works because a set returns either the value itself for all
contained elements or `nil`:

``` clojure
(valid? 1)
;; => 1

(valid? 4)
;; => nil
```

## Locals, Blocks, and Loops

### Locals

*ClojureScript* does not have the concept of variables as in ALGOL-like
languages, but it does have locals. Locals, as per usual, are immutable,
and if you try to mutate them, the compiler will throw an error.

Locals are defined with the `let` expression. The expression starts with
a vector as the first parameter followed by an arbitrary number of
expressions. The first parameter (the vector) should contain an
arbitrary number of pairs that give a *binding form* (usually a symbol)
followed by an expression whose value will be bound to this new local
for the remainder of the `let` expression.

``` clojure
(let [x (inc 1)
      y (+ x 1)]
  (println "Simple message from the body of a let")
  (* x y))
;; Simple message from the body of a let
;; => 6
```

In the preceding example, the symbol `x` is bound to the value
`(inc 1)`, which comes out to 2, and the symbol `y` is bound to the sum
of `x` and 1, which comes out to 3. Given those bindings, the
expressions `(println "Simple message from the body
of a let")` and `(* x y)` are evaluated.

### Blocks

In JavaScript, braces `{` and `}` delimit a block of code that “belongs
together”. Blocks in *ClojureScript* are created using the `do`
expression and are usually used for side effects, like printing
something to the console or writing a log in a logger.

A side effect is something that is not necessary for the return value.

The `do` expression accepts as its parameter an arbitrary number of
other expressions, but it returns the return value only from the last
one:

``` clojure
(do
  (println "hello world")
  (println "hola mundo")
  (* 3 5) ;; this value will not be returned; it is thrown away
  (+ 1 2))

;; hello world
;; hola mundo
;; => 3
```

The body of the `let` expression, explained in the previous section, is
very similar to the `do` expression in that it allows multiple
expressions. In fact, the `let` has an implicit `do`.

### Loops

The functional approach of *ClojureScript* means that it does not have
standard, well-known, statement-based loops such as `for` in JavaScript.
The loops in *ClojureScript* are handled using recursion. Recursion
sometimes requires additional thinking about how to model your problem
in a slightly different way than imperative languages.

Many of the common patterns for which `for` is used in other languages
are achieved through higher-order functions - functions that accept
other functions as parameters.

#### Looping with loop/recur

Let’s take a look at how to express loops using recursion with the
`loop` and `recur` forms. `loop` defines a possibly empty list of
bindings (notice the symmetry with `let`) and `recur` jumps execution
back to the looping point with new values for those bindings.

Let’s see an example:

``` clojure
(loop [x 0]
  (println "Looping with " x)
  (if (= x 2)
    (println "Done looping!")
    (recur (inc x))))
;; Looping with 0
;; Looping with 1
;; Looping with 2
;; Done looping!
;; => nil
```

In the above snippet, we bind the name `x` to the value `0` and execute
the body. Since the condition is not met the first time, it’s rerun with
`recur`, incrementing the binding value with the `inc` function. We do
this once more until the condition is met and, since there aren’t any
more `recur` calls, exit the loop.

Note that `loop` isn’t the only point we can `recur` to; using `recur`
inside a function executes the body of the function recursively with the
new bindings:

``` clojure
(defn recursive-function
  [x]
  (println "Looping with" x)
  (if (= x 2)
    (println "Done looping!")
    (recur (inc x))))

(recursive-function 0)
;; Looping with 0
;; Looping with 1
;; Looping with 2
;; Done looping!
;; => nil
```

#### Replacing for loops with higher-order functions

In imperative programming languages it is common to use `for` loops to
iterate over data and transform it, usually with the intent being one of
the following:

  - Transform every value in the iterable yielding another iterable

  - Filter the elements of the iterable by certain criteria

  - Convert the iterable to a value where each iteration depends on the
    result from the previous one

  - Run a computation for every value in the iterable

The above actions are encoded in higher-order functions and syntactic
constructs in ClojureScript; let’s see an example of the first three.

For transforming every value in an iterable data structure we use the
`map` function, which takes a function and a sequence and applies the
function to every element:

``` clojure
(map inc [0 1 2])
;; => (1 2 3)
```

The first parameter for `map` can be *any* function that takes one
argument and returns a value. For example, if you had a graphing
application and you wanted to graph the equation `y = 3x + 5` for a set
of *x* values, you could get the *y* values like this:

``` clojure
(defn y-value [x] (+ (* 3 x) 5))

(map y-value [1 2 3 4 5])
;; => (8 11 14 17 20)
```

If your function is short, you can use an anonymous function instead,
either the normal or short syntax:

``` clojure
(map (fn [x] (+ (* 3 x) 5)) [1 2 3 4 5])
;; => (8 11 14 17 20)

(map #(+ (* 3 %) 5) [1 2 3 4 5])
;; => (8 11 14 17 20)
```

For filtering the values of a data structure we use the `filter`
function, which takes a predicate and a sequence and gives a new
sequence with only the elements that returned `true` for the given
predicate:

``` clojure
(filter odd? [1 2 3 4])
;; => (1 3)
```

Again, you can use any function that returns `true` or `false` as the
first argument to `filter`. Here is an example that keeps only words
less than five characters long. (The `count` function returns the length
of its argument.)

``` clojure
(filter (fn [word] (< (count word) 5)) ["ant" "baboon" "crab" "duck" "echidna" "fox"])
;; => ("ant" "crab" "duck" "fox")
```

Converting an iterable to a single value, accumulating the intermediate
result at every step of the iteration can be achieved with `reduce`,
which takes a function for accumulating values, an optional initial
value and a collection:

``` clojure
(reduce + 0 [1 2 3 4])
;; => 10
```

Yet again, you can provide your own function as the first argument to
`reduce`, but your function must have *two* parameters. The first one is
the "accumulated value" and the second parameter is the collection item
being processed. The function returns a value that becomes the
accumulator for the next item in the list. For example, here is how you
would find the sum of squares of a set of numbers (this is an important
calculation in statistics). Using a separate function:

``` clojure
(defn sum-squares
  [accumulator item]
  (+ accumulator (* item item)))

(reduce sum-squares 0 [3 4 5])
;; => 50
```

…​and with an anonymous function:

``` clojure
(reduce (fn [acc item] (+ acc (* item item))) 0 [3 4 5])
;; => 50
```

Here is a `reduce` that finds the total number of characters in a set of
words:

``` clojure
(reduce (fn [acc word] (+ acc (count word))) 0 ["ant" "bee" "crab" "duck"])
;; => 14
```

We have not used the short syntax here because, although it requires
less typing, it can be less readable, and when you are starting with a
new language, it’s important to be able to read what you wrote\! If you
are comfortable with the short syntax, feel free to use it.

Remember to choose your starting value for the accumulator carefully. If
you wanted to use `reduce` to find the product of a series of numbers,
you would have to start with one rather than zero, otherwise all the
numbers would be multiplied by zero\!

``` clojure
;; wrong starting value
(reduce * 0 [3 4 5])
;; => 0

;; correct starting accumulator
(reduce * 1 [3 4 5])
;; => 60
```

#### `for` sequence comprehensions

In ClojureScript, the `for` construct isn’t used for iteration but for
generating sequences, an operation also known as "sequence
comprehension". In this section we’ll learn how it works and use it to
declaratively build sequences.

`for` takes a vector of bindings and an expression and generates a
sequence of the result of evaluating the expression. Let’s take a look
at an example:

``` clojure
(for [x [1 2 3]]
  [x (* x x)])
;; => ([1 1] [2 4] [3 9])
```

In this example, `x` is bound to each of the items in the vector
`[1 2 3]` in turn, and returns a new sequence of two-item vectors with
the original item squared.

`for` supports multiple bindings, which will cause the collections to be
iterated in a nested fashion, much like nesting `for` loops in
imperative languages. The innermost binding iterates “fastest.”

``` clojure
(for [x [1 2 3]
      y [4 5]]
  [x y])

;; => ([1 4] [1 5] [2 4] [2 5] [3 4] [3 5])
```

We can also follow the bindings with three modifiers: `:let` for
creating local bindings, `:while` for breaking out of the sequence
generation, and `:when` for filtering out values.

Here’s an example of local bindings using the `:let` modifier; note that
the bindings defined with it will be available in the expression:

``` clojure
(for [x [1 2 3]
      y [4 5]
      :let [z (+ x y)]]
  z)
;; => (5 6 6 7 7 8)
```

We can use the `:while` modifier for expressing a condition that, when
it is no longer met, will stop the sequence generation. Here’s an
example:

``` clojure
(for [x [1 2 3]
      y [4 5]
      :while (= y 4)]
  [x y])

;; => ([1 4] [2 4] [3 4])
```

For filtering out generated values, use the `:when` modifier as in the
following example:

``` clojure
(for [x [1 2 3]
      y [4 5]
      :when (= (+ x y) 6)]
  [x y])

;; => ([1 5] [2 4])
```

We can combine the modifiers shown above for expressing complex sequence
generations or more clearly expressing the intent of our comprehension:

``` clojure
(for [x [1 2 3]
      y [4 5]
      :let [z (+ x y)]
      :when (= z 6)]
  [x y])

;; => ([1 5] [2 4])
```

When we outlined the most common usages of the `for` construct in
imperative programming languages, we mentioned that sometimes we want to
run a computation for every value in a sequence, not caring about the
result. Presumably we do this for achieving some sort of side-effect
with the values of the sequence.

ClojureScript provides the `doseq` construct, which is analogous to
`for` but executes the expression, discards the resulting values, and
returns `nil`.

``` clojure
(doseq [x [1 2 3]
        y [4 5]
       :let [z (+ x y)]]
  (println x "+" y "=" z))

;; 1 + 4 = 5
;; 1 + 5 = 6
;; 2 + 4 = 6
;; 2 + 5 = 7
;; 3 + 4 = 7
;; 3 + 5 = 8
;; => nil
```

If you want just iterate and apply some side effectfull operation (like
`println`) over each item in the collection, you can just use the
specialized function `run!` that internally uses fast reduction:

``` clojure
(run! println [1 2 3])
;; 1
;; 2
;; 3
;; => nil
```

This function explicitly returns `nil`.

## Collection types

### Immutable and persistent

We mentioned before that ClojureScript collections are persistent and
immutable, but we didn’t explain what that meant.

An immutable data structure, as its name suggests, is a data structure
that cannot be changed. In-place updates are not allowed in immutable
data structures.

Let’s illustrate that with an example: appending values to a vector
using the `conj` (conjoin) operation.

``` clojure
(let [xs [1 2 3]
      ys (conj xs 4)]
  (println "xs:" xs)
  (println "ys:" ys))

;; xs: [1 2 3]
;; ys: [1 2 3 4]
;; => nil
```

As you can see, we derived a new version of the `xs` vector appending an
element to it and got a new vector `ys` with the element added. However,
the `xs` vector remained unchanged because it is immutable.

A persistent data structure is a data structure that returns a new
version of itself when transforming it, leaving the original unmodified.
ClojureScript makes this memory and time efficient using an
implementation technique called *structural sharing*, where most of the
data shared between two versions of a value is not duplicated and
transformations of a value are implemented by copying the minimal amount
of data required.

If you want to see an example of how structural sharing works, read on.
If you’re not interested in more details you can skip over to the [next
section](#the-sequence-abstraction).

For illustrating the structural sharing of ClojureScript data
structures, let’s compare whether some parts of the old and new versions
of a data structure are actually the same object with the `identical?`
predicate. We’ll use the list data type for this purpose:

``` clojure
(let [xs (list 1 2 3)
      ys (cons 0 xs)]
  (println "xs:" xs)
  (println "ys:" ys)
  (println "(rest ys):" (rest ys))
  (identical? xs (rest ys)))

;; xs: (1 2 3)
;; ys: (0 1 2 3)
;; (rest ys): (1 2 3)
;; => true
```

As you can see in the example, we used `cons` (construct) to prepend a
value to the `xs` list and we got a new list `ys` with the element
added. The `rest` of the `ys` list (all the values but the first) are
the same object in memory as the `xs` list, thus `xs` and `ys` share
structure.

### The sequence abstraction

One of the central ClojureScript abstractions is the *sequence* which
can be thought of as a list and can be derived from any of the
collection types. It is persistent and immutable like all collection
types, and many of the core ClojureScript functions return sequences.

The types that can be used to generate a sequence are called "seqables";
we can call `seq` on them and get a sequence back. Sequences support two
basic operations: `first` and `rest`. They both call `seq` on the
argument we provide them:

``` clojure
(first [1 2 3])
;; => 1

(rest [1 2 3])
;; => (2 3)
```

Calling `seq` on a seqable can yield different results if the seqable is
empty or not. It will return `nil` when empty and a sequence otherwise:

``` clojure
(seq [])
;; => nil

(seq [1 2 3])
;; => (1 2 3)
```

`next` is a similar sequence operation to `rest`, but it differs from
the latter in that it yields a `nil` value when called with a sequence
with one or zero elements. Note that, when given one of the
aforementioned sequences, the empty sequence returned by `rest` will
evaluate as a boolean true whereas the `nil` value returned by `next`
will evaluate as false ([see the section on *truthiness* later in this
chapter](#truthiness-section)).

``` clojure
(rest [])
;; => ()

(next [])
;; => nil

(rest [1 2 3])
;; => (2 3)

(next [1 2 3])
;; => (2 3)
```

#### nil-punning

Since `seq` returns `nil` when the collection is empty, and `nil`
evaluates to false in boolean context, you can check to see if a
collection is empty by using the `seq` function. The technical term for
this is nil-punning.

``` clojure
(defn print-coll
  [coll]
  (when (seq coll)
    (println "Saw " (first coll))
    (recur (rest coll))))

(print-coll [1 2 3])
;; Saw 1
;; Saw 2
;; Saw 3
;; => nil

(print-coll #{1 2 3})
;; Saw 1
;; Saw 3
;; Saw 2
;; => nil
```

Though `nil` is neither a seqable nor a sequence, it is supported by all
the functions we saw so far:

``` clojure
(seq nil)
;; => nil

(first nil)
;; => nil

(rest nil)
;; => ()
```

#### Functions that work on sequences

The ClojureScript core functions for transforming collections make
sequences out of their arguments and are implemented in terms of the
generic sequence operations we learned about in the preceding section.
This makes them highly generic because we can use them on any data type
that is seqable. Let’s see how we can use `map` with a variety of
seqables:

``` clojure
(map inc [1 2 3])
;; => (2 3 4)

(map inc #{1 2 3})
;; => (2 4 3)

(map count {:a 41 :b 40})
;; => (2 2)

(map inc '(1 2 3))
;; => (2 3 4)
```

<div class="note">

When you use the `map` function on a map collection, your higher-order
function will receive a two-item vector containing a key and value from
the map. The following example uses
[destructuring](#destructuring-section) to access the key and value.

</div>

``` clojure
(map (fn [[key value]] (* value value))
     {:ten 10 :seven 7 :four 4})
;; => (100 49 16)
```

Obviously the same operation can be done in more idiomatic way only
obtaining a seq of values:

``` clojure
(map (fn [value] (* value value))
     (vals {:ten 10 :seven 7 :four 4}))
;; => (100 49 16)
```

As you may have noticed, functions that operate on sequences are safe to
use with empty collections or even `nil` values since they don’t need to
do anything but return an empty sequence when encountering such values.

``` clojure
(map inc [])
;; => ()

(map inc #{})
;; => ()

(map inc nil)
;; => ()
```

We already saw examples with the usual suspects like `map`, `filter`,
and `reduce`, but ClojureScript offers a plethora of generic sequence
operations in its core namespace. Note that many of the operations we’ll
learn about either work with seqables or are extensible to user-defined
types.

We can query a value to know whether it’s a collection type with the
`coll?` predicate:

``` clojure
(coll? nil)
;; => false

(coll? [1 2 3])
;; => true

(coll? {:language "ClojureScript" :file-extension "cljs"})
;; => true

(coll? "ClojureScript")
;; => false
```

Similar predicates exist for checking if a value is a sequence (with
`seq?`) or a seqable (with `seqable?`):

``` clojure
(seq? nil)
;; => false
(seqable? nil)
;; => false

(seq? [])
;; => false
(seqable? [])
;; => true

(seq? #{1 2 3})
;; => false
(seqable? #{1 2 3})
;; => true

(seq? "ClojureScript")
;; => false
(seqable? "ClojureScript")
;; => false
```

For collections that can be counted in constant time, we can use the
`count` operation. This operation also works on strings, even though, as
you have seen, they are not collections, sequences, or seqable.

``` clojure
(count nil)
;; => 0

(count [1 2 3])
;; => 3

(count {:language "ClojureScript" :file-extension "cljs"})
;; => 2

(count "ClojureScript")
;; => 13
```

We can also get an empty variant of a given collection with the `empty`
function:

``` clojure
(empty nil)
;; => nil

(empty [1 2 3])
;; => []

(empty #{1 2 3})
;; => #{}
```

The `empty?` predicate returns true if the given collection is empty:

``` clojure
(empty? nil)
;; => true

(empty? [])
;; => true

(empty? #{1 2 3})
;; => false
```

The `conj` operation adds elements to collections and may add them in
different "places" depending on the type of collection. It adds them
where it is most performant for the collection type, but note that not
every collection has a defined order.

We can pass as many elements as we want to add to `conj`; let’s see it
in action:

``` clojure
(conj nil 42)
;; => (42)

(conj [1 2] 3)
;; => [1 2 3]

(conj [1 2] 3 4 5)
;; => [1 2 3 4 5]

(conj '(1 2) 0)
;; => (0 1 2)

(conj #{1 2 3} 4)
;; => #{1 3 2 4}

(conj {:language "ClojureScript"} [:file-extension "cljs"])
;; => {:language "ClojureScript", :file-extension "cljs"}
```

#### Laziness

Most of ClojureScript’s sequence-returning functions generate lazy
sequences instead of eagerly creating a whole new sequence. Lazy
sequences generate their contents as they are requested, usually when
iterating over them. Laziness ensures that we don’t do more work than we
need to and gives us the possibility of treating potentially infinite
sequences as regular ones.

Consider the `range` function, which generates a range of integers:

``` clojure
(range 5)
;; => (0 1 2 3 4)
(range 1 10)
;; => (1 2 3 4 5 6 7 8 9)
(range 10 100 15)
;; (10 25 40 55 70 85)
```

If you just say `(range)`, you will get an infinite sequence of all the
integers. Do **not** try this in the REPL, unless you are prepared to
wait for a very, very long time, because the REPL wants to fully
evaluate the expression.

Here is a contrived example. Let’s say you are writing a graphing
program and you are graphing the equation *y*= 2 *x* <sup>2</sup> + 5,
and you want only those values of *x* for which the *y* value is less
than 100. You can generate all the numbers 0 through 100, which will
certainly be enough, and then `take-while` the condition holds:

``` clojure
(take-while (fn [x] (< (+ (* 2 x x) 5) 100))
            (range 0 100))
;; => (0 1 2 3 4 5 6)
```

### Collections in depth

Now that we’re acquainted with ClojureScript’s sequence abstraction and
some of the generic sequence manipulating functions, it’s time to dive
into the concrete collection types and the operations they support.

#### Lists

In ClojureScript, lists are mostly used as a data structure for grouping
symbols together into programs. Unlike in other Lisps, many of the
syntactic constructs of ClojureScript use data structures different from
the list (vectors and maps). This makes code less uniform, but the gains
in readability are well worth the price.

You can think of ClojureScript lists as singly linked lists, where each
node contains a value and a pointer to the rest of the list. This makes
it natural (and fast\!) to add items to the front of the list, since
adding to the end would require traversal of the entire list. The
prepend operation is performed using the `cons` function.

``` clojure
(cons 0 (cons 1 (cons 2 ())))
;; => (0 1 2)
```

We used the literal `()` to represent the empty list. Since it doesn’t
contain any symbols, it is not treated as a function call. However, when
using list literals that contain elements, we need to quote them to
prevent ClojureScript from evaluating them as a function call:

``` clojure
(cons 0 '(1 2))
;; => (0 1 2)
```

Since the head is the position that has constant time addition in the
list collection, the `conj` operation on lists naturally adds items to
the front:

``` clojure
(conj '(1 2) 0)
;; => (0 1 2)
```

Lists and other ClojureScript data structures can be used as stacks
using the `peek`, `pop`, and `conj` functions. Note that the top of the
stack will be the "place" where `conj` adds elements, making `conj`
equivalent to the stack’s push operation. In the case of lists, `conj`
adds elements to the front of the list, `peek` returns the first element
of the list, and `pop` returns a list with all the elements but the
first one.

Note that the two operations that return a stack (`conj` and `pop`)
don’t change the type of the collection used for the stack.

``` clojure
(def list-stack '(0 1 2))

(peek list-stack)
;; => 0

(pop list-stack)
;; => (1 2)

(type (pop list-stack))
;; => cljs.core/List

(conj list-stack -1)
;; => (-1 0 1 2)

(type (conj list-stack -1))
;; => cljs.core/List
```

One thing that lists are not particularly good at is random indexed
access. Since they are stored in a single linked list-like structure in
memory, random access to a given index requires a linear traversal in
order to either retrieve the requested item or throw an index out of
bounds error. Non-indexed ordered collections like lazy sequences also
suffer from this limitation.

#### Vectors

Vectors are one of the most common data structures in ClojureScript.
They are used as a syntactic construct in many places where more
traditional Lisps use lists, for example in function argument
declarations and `let` bindings.

ClojureScript vectors have enclosing brackets `[]` in their syntax
literals. They can be created with `vector` and from another collection
with `vec`:

``` clojure
(vector? [0 1 2])
;; => true

(vector 0 1 2)
;; => [0 1 2]

(vec '(0 1 2))
;; => [0 1 2]
```

Vectors are, like lists, ordered collections of heterogeneous values.
Unlike lists, vectors grow naturally from the tail, so the `conj`
operation appends items to the end of a vector. Insertion on the end of
a vector is effectively constant time:

``` clojure
(conj [0 1] 2)
;; => [0 1 2]
```

Another thing that differentiates lists and vectors is that vectors are
indexed collections and as such support efficient random index access
and non-destructive updates. We can use the `nth` function to retrieve
values given an index:

``` clojure
(nth [0 1 2] 0)
;; => 0
```

Since vectors associate sequential numeric keys (indexes) to values, we
can treat them as an associative data structure. ClojureScript provides
the `assoc` function that, given an associative data structure and a set
of key-value pairs, yields a new data structure with the values
corresponding to the keys modified. Indexes begin at zero for the first
element in a vector.

``` clojure
(assoc ["cero" "uno" "two"] 2 "dos")
;; => ["cero" "uno" "dos"]
```

Note that we can only `assoc` to a key that is either contained in the
vector already or if it is the last position in a vector:

``` clojure
(assoc ["cero" "uno" "dos"] 3 "tres")
;; => ["cero" "uno" "dos" "tres"]

(assoc ["cero" "uno" "dos"] 4 "cuatro")
;; Error: Index 4 out of bounds [0,3]
```

Perhaps surprisingly, associative data structures can also be used as
functions. They are functions of their keys to the values they are
associated with. In the case of vectors, if the given key is not present
an exception is thrown:

``` clojure
(["cero" "uno" "dos"] 0)
;; => "cero"

(["cero" "uno" "dos"] 2)
;; => "dos"

(["cero" "uno" "dos"] 3)
;; Error: Not item 3 in vector of length 3
```

As with lists, vectors can also be used as stacks with the `peek`,
`pop`, and `conj` functions. Note, however, that vectors grow from the
opposite end of the collection as lists:

``` clojure
(def vector-stack [0 1 2])

(peek vector-stack)
;; => 2

(pop vector-stack)
;; => [0 1]

(type (pop vector-stack))
;; => cljs.core/PersistentVector

(conj vector-stack 3)
;; => [0 1 2 3]

(type (conj vector-stack 3))
;; => cljs.core/PersistentVector
```

The `map` and `filter` operations return lazy sequences, but as it is
common to need a fully realized sequence after performing those
operations, vector-returning counterparts of such functions are
available as `mapv` and `filterv`. They have the advantages of being
faster than building a vector from a lazy sequence and making your
intent more explicit:

``` clojure
(map inc [0 1 2])
;; => (1 2 3)

(type (map inc [0 1 2]))
;; => cljs.core/LazySeq

(mapv inc [0 1 2])
;; => [1 2 3]

(type (mapv inc [0 1 2]))
;; => cljs.core/PersistentVector
```

#### Maps

Maps are ubiquitous in ClojureScript. Like vectors, they are also used
as a syntactic construct, particularly for attaching
[metadata](#metadata-section) to vars. Any ClojureScript data structure
can be used as a key in a map, although it’s common to use keywords
since they can also be called as functions.

ClojureScript maps are written literally as key-value pairs enclosed in
braces `{}`. Alternatively, they can be created with the `hash-map`
function:

``` clojure
(map? {:name "Cirilla"})
;; => true

(hash-map :name "Cirilla")
;; => {:name "Cirilla"}

(hash-map :name "Cirilla" :surname "Fiona")
;; => {:name "Cirilla" :surname "Fiona"}
```

Since regular maps don’t have a specific order, the `conj` operation
just adds one or more key-value pairs to a map. `conj` for maps expects
one or more sequences of key-value pairs as its last arguments:

``` clojure
(def ciri {:name "Cirilla"})

(conj ciri [:surname "Fiona"])
;; => {:name "Cirilla", :surname "Fiona"}

(conj ciri [:surname "Fiona"] [:occupation "Wizard"])
;; => {:name "Cirilla", :surname "Fiona", :occupation "Wizard"}
```

In the preceding example, it just so happens that the order was
preserved, but if you have many keys, you will see that the order is not
preserved.

Maps associate keys to values and, as such, are an associative data
structure. They support adding associations with `assoc` and, unlike
vectors, removing them with `dissoc`. `assoc` will also update the value
of an existing key. Let’s explore these functions:

``` clojure
(assoc {:name "Cirilla"} :surname "Fiona")
;; => {:name "Cirilla", :surname "Fiona"}
(assoc {:name "Cirilla"} :name "Alfonso")
;; => {:name "Alfonso"}
(dissoc {:name "Cirilla"} :name)
;; => {}
```

Maps are also functions of their keys, returning the values related to
the given keys. Unlike vectors, they return `nil` if we supply a key
that is not present in the map:

``` clojure
({:name "Cirilla"} :name)
;; => "Cirilla"

({:name "Cirilla"} :surname)
;; => nil
```

ClojureScript also offers sorted hash maps which behave like their
unsorted versions but preserve order when iterating over them. We can
create a sorted map with default ordering with `sorted-map`:

``` clojure
(def sm (sorted-map :c 2 :b 1 :a 0))
;; => {:a 0, :b 1, :c 2}

(keys sm)
;; => (:a :b :c)
```

If we need a custom ordering we can provide a comparator function to
`sorted-map-by`, let’s see an example inverting the value returned by
the built-in `compare` function. Comparator functions take two items to
compare and return -1 (if the first item is less than the second), 0 (if
they are equal), or 1 (if the first item is greater than the second).

``` clojure
(defn reverse-compare [a b] (compare b a))

(def sm (sorted-map-by reverse-compare :a 0 :b 1 :c 2))
;; => {:c 2, :b 1, :a 0}

(keys sm)
;; => (:c :b :a)
```

#### Sets

Sets in ClojureScript have literal syntax as values enclosed in `#{}`
and they can be created with the `set` constructor. They are unordered
collections of values without duplicates.

``` clojure
(set? #{\a \e \i \o \u})
;; => true

(set [1 1 2 3])
;; => #{1 2 3}
```

Set literals cannot contain duplicate values. If you accidentally write
a set literal with duplicates an error will be thrown:

``` clojure
#{1 1 2 3}
;; clojure.lang.ExceptionInfo: Duplicate key: 1
```

There are many operations that can be performed with sets, although they
are located in the `clojure.set` namespace and thus need to be imported.
You’ll learn [the details of namespacing](#namespace-section) later; for
now, you only need to know that we are loading a namespace called
`clojure.set` and binding it to the `s` symbol.

``` clojure
(require '[clojure.set :as s])

(def danish-vowels #{\a \e \i \o \u \æ \ø \å})
;; => #{"a" "e" "å" "æ" "i" "o" "u" "ø"}

(def spanish-vowels #{\a \e \i \o \u})
;; => #{"a" "e" "i" "o" "u"}

(s/difference danish-vowels spanish-vowels)
;; => #{"å" "æ" "ø"}

(s/union danish-vowels spanish-vowels)
;; => #{"a" "e" "å" "æ" "i" "o" "u" "ø"}

(s/intersection danish-vowels spanish-vowels)
;; => #{"a" "e" "i" "o" "u"}
```

A nice property of immutable sets is that they can be nested. Languages
that have mutable sets can end up containing duplicate values, but that
can’t happen in ClojureScript. In fact, all ClojureScript data
structures can be nested arbitrarily due to immutability.

Sets also support the generic `conj` operation just like every other
collection does.

``` clojure
(def spanish-vowels #{\a \e \i \o \u})
;; => #{"a" "e" "i" "o" "u"}

(def danish-vowels (conj spanish-vowels \æ \ø \å))
;; => #{"a" "e" "i" "o" "u" "æ" "ø" "å"}

(conj #{1 2 3} 1)
;; => #{1 3 2}
```

Sets act as read-only associative data that associates the values it
contains to themselves. Since every value except `nil` and `false` is
truthy in ClojureScript, we can use sets as predicate functions:

``` clojure
(def vowels #{\a \e \i \o \u})
;; => #{"a" "e" "i" "o" "u"}

(get vowels \b)
;; => nil

(contains? vowels \b)
;; => false

(vowels \a)
;; => "a"

(vowels \z)
;; => nil

(filter vowels "Hound dog")
;; => ("o" "u" "o")
```

Sets have a sorted counterpart like maps do that are created using the
functions `sorted-set` and `sorted-set-by` which are analogous to map’s
`sorted-map` and `sorted-map-by`.

``` clojure
(def unordered-set #{[0] [1] [2]})
;; => #{[0] [2] [1]}

(seq unordered-set)
;; => ([0] [2] [1])

(def ordered-set (sorted-set [0] [1] [2]))
;; =># {[0] [1] [2]}

(seq ordered-set)
;; => ([0] [1] [2])
```

#### Queues

ClojureScript also provides a persistent and immutable queue. Queues are
not used as pervasively as other collection types. They can be created
using the `#queue []` literal syntax, but there are no convenient
constructor functions for them.

``` clojure
(def pq #queue [1 2 3])
;; => #queue [1 2 3]
```

Using `conj` to add values to a queue adds items onto the rear:

``` clojure
(def pq #queue [1 2 3])
;; => #queue [1 2 3]

(conj pq 4 5)
;; => #queue [1 2 3 4 5]
```

A thing to bear in mind about queues is that the stack operations don’t
follow the usual stack semantics (pushing and popping from the same
end). `pop` takes values from the front position, and `conj` pushes
(appends) elements to the back.

``` clojure
(def pq #queue [1 2 3])
;; => #queue [1 2 3]

(peek pq)
;; => 1

(pop pq)
;; => #queue [2 3]

(conj pq 4)
;; => #queue [1 2 3 4]
```

Queues are not as frequently used as lists or vectors, but it is good to
know that they are available in ClojureScript, as they may occasionally
come in handy.

## Destructuring

Destructuring, as its name suggests, is a way of taking apart structured
data such as collections and focusing on individual parts of them.
ClojureScript offers a concise syntax for destructuring both indexed
sequences and associative data structures that can be used any place
where bindings are declared.

Let’s see an example of what destructuring is useful for that will help
us understand the previous statements better. Imagine that you have a
sequence but are only interested in the first and third item. You could
get a reference to them easily with the `nth` function:

``` clojure
(let [v [0 1 2]
      fst (nth v 0)
      thrd (nth v 2)]
  [thrd fst])
;; => [2 0]
```

However, the previous code is overly verbose. Destructuring lets us
extract values of indexed sequences more succintly using a vector on the
left-hand side of a binding:

``` clojure
(let [[fst _ thrd] [0 1 2]]
  [thrd fst])
;; => [2 0]
```

In the above example, `[fst _ thrd]` is a destructuring form. It is
represented as a vector and used for binding indexed values to the
symbols `fst` and `thrd`, corresponding to the index `0` and `2`,
respectively. The `_` symbol is used as a placeholder for indexes we are
not interested in — in this case `1`.

Note that destructuring is not limited to the `let` binding form; it
works in almost every place where we bind values to symbols such as in
the `for` and `doseq` special forms or in function arguments. We can
write a function that takes a pair and swaps its positions very
concisely using destructuring syntax in function arguments:

``` clojure
(defn swap-pair [[fst snd]]
  [snd fst])

(swap-pair [1 2])
;; => [2 1]

(swap-pair '(3 4))
;; => [4 3]
```

Positional destructuring with vectors is quite handy for taking indexed
values out of sequences, but sometimes we don’t want to discard the rest
of the elements in the sequence when destructuring. Similarly to how `&`
is used for accepting variadic function arguments, the ampersand can be
used inside a vector destructuring form for grouping together the rest
of a sequence:

``` clojure
(let [[fst snd & more] (range 10)]
  {:first fst
   :snd snd
   :rest more})
;; => {:first 0, :snd 1, :rest (2 3 4 5 6 7 8 9)}
```

Notice how the value in the `0` index got bound to `fst`, the value in
the `1` index got bound to `snd`, and the sequence of elements from `2`
onwards got bound to the `more` symbol.

We may still be interested in a data structure as a whole even when we
are destructuring it. This can be achieved with the `:as` keyword. If
used inside a destructuring form, the original data structure is bound
to the symbol following that keyword:

``` clojure
(let [[fst snd & more :as original] (range 10)]
  {:first fst
   :snd snd
   :rest more
   :original original})
;; => {:first 0, :snd 1, :rest (2 3 4 5 6 7 8 9), :original (0 1 2 3 4 5 6 7 8 9)}
```

Not only can indexed sequences be destructured, but associative data can
also be destructured. Its destructuring binding form is represented as a
map instead of a vector, where the keys are the symbols we want to bind
values to and the values are the keys that we want to look up in the
associative data structure. Let’s see an example:

``` clojure
(let [{language :language} {:language "ClojureScript"}]
  language)
;; => "ClojureScript"
```

In the above example, we are extracting the value associated with the
`:language` key and binding it to the `language` symbol. When looking up
keys that are not present, the symbol will get bound to `nil`:

``` clojure
(let [{name :name} {:language "ClojureScript"}]
  name)
;; => nil
```

Associative destructuring lets us give default values to bindings which
will be used if the key isn’t found in the data structure we are taking
apart. A map following the `:or` keyword is used for default values as
the following examples show:

``` clojure
(let [{name :name :or {name "Anonymous"}} {:language "ClojureScript"}]
  name)
;; => "Anonymous"

(let [{name :name :or {name "Anonymous"}} {:name "Cirilla"}]
  name)
;; => "Cirilla"
```

Associative destructuring also supports binding the original data
structure to a symbol placed after the `:as` keyword:

``` clojure
(let [{name :name :as person} {:name "Cirilla" :age 49}]
  [name person])
;; => ["Cirilla" {:name "Cirilla" :age 49}]
```

Keywords aren’t the only things that can be the keys of associative data
structures. Numbers, strings, symbols and many other data structures can
be used as keys, so we can destructure using those, too. Note that we
need to quote the symbols to prevent them from being resolved as a var
lookup:

``` clojure
(let [{one 1} {0 "zero" 1 "one"}]
  one)
;; => "one"

(let [{name "name"} {"name" "Cirilla"}]
  name)
;; => "Cirilla"

(let [{lang 'language} {'language "ClojureScript"}]
  lang)
;; => "ClojureScript"
```

Since the values corresponding to keys are usually bound to their
equivalent symbol representation (for example, when binding the value of
`:language` to the symbol `language`) and keys are usually keywords,
strings, or symbols, ClojureScript offers shorthand syntax for these
cases.

We’ll show examples of all of these, starting with destructuring
keywords using `:keys`:

``` clojure
(let [{:keys [name surname]} {:name "Cirilla" :surname "Fiona"}]
  [name surname])
;; => ["Cirilla" "Fiona"]
```

As you can see in the example, if we use the `:keys` keyword and
associate it with a vector of symbols in a binding form, the values
corresponding to the keywordized version of the symbols will be bound to
them. The `{:keys [name surname]}` destructuring is equivalent to `{name
:name surname :surname}`, only shorter.

The string and symbol shorthand syntax works exactly like `:keys`, but
using the `:strs` and `:syms` keywords respectively:

``` clojure
(let [{:strs [name surname]} {"name" "Cirilla" "surname" "Fiona"}]
  [name surname])
;; => ["Cirilla" "Fiona"]

(let [{:syms [name surname]} {'name "Cirilla" 'surname "Fiona"}]
  [name surname])
;; => ["Cirilla" "Fiona"]
```

An interesting property of destructuring is that we can nest
destructuring forms arbitrarily, which makes code that accesses nested
data on a collection very easy to understand, as it mimics the
collection’s structure:

``` clojure
(let [{[fst snd] :languages} {:languages ["ClojureScript" "Clojure"]}]
  [snd fst])
;; => ["Clojure" "ClojureScript"]
```

## Threading Macros

Threading macros, also known as arrow functions, enables one to write
more readable code when multiple nested function calls are performed.

Imagine you have `(f (g (h x)))` where a function `f` receives as its
first parameter the result of executing function `g`, repeated multiple
times. With the most basic `→` threading macro you can convert that into
`(-> x (h) (g) (f))` which is easier to read.

The result is syntactic sugar, because the arrow functions are defined
as macros and it does not imply any runtime performance. The `(-> x (h)
(g) (f))` is automatically converted to (f (g (h x))) at compile time.

Take note that the parenthesis on `h`, `g` and `f` are optional, and can
be ommited: `(f (g (h x)))` is the same as `(-> x h g f)`.

### The thread-first macro (`->`)

This is called **thread first** because it threads the first argument
throught the different expressions as first arguments.

Using a more concrete example, this is how the code looks without using
threading macros:

``` clojure
(def book {:name "Lady of the Lake"
           :readers 0})

(update (assoc book :age 1999) :readers inc)
;; => {:name "Lady of the lake" :age 1999 :readers 1}
```

We can rewrite that code to use the `->` threading macro:

``` clojure
(-> book
    (assoc :age 1999)
    (update :readers inc))
;; => {:name "Lady of the lake" :age 1999 :readers 1}
```

This threading macro is especially useful for transforming data
structures, because *ClojureScript* (and *Clojure*) functions for data
structures transformations consistently uses the first argument for
receive the data structure.

### The thread-last macro (`->>`)

The main difference between the thread-last and thread-first macros is
that instead of threading the first argument given as the first argument
on the following expresions, it threads it as the last argument.

Let’s look at an example:

``` clojure
(def numbers [1 2 3 4 5 6 7 8 9 0])

(take 2 (filter odd? (map inc numbers)))
;; => (3 5)
```

The same code written using `->>` threading macro:

``` clojure
(->> numbers
     (map inc)
     (filter odd?)
     (take 2))
;; => (3 5)
```

This threading macro is especially useful for transforming sequences or
collections of data because *ClojureScript* functions that work with
sequences and collections consistently use the last argument position to
receive them.

### The thread-as macro (`as->`)

Finally, there are cases where neither `->` nor `->>` are applicable. In
these cases, you’ll need to use `as->`, the more flexible alternative,
that allows you to thread into any argument position, not just the first
or last.

It expects two fixed arguments and an arbitrary number of expressions.
As with `->`, the first argument is a value to be threaded through the
following forms. The second argument is the name of a binding. In each
of the subsequent forms, the bound name can be used for the prior
expression’s result.

Let’s see an example:

``` clojure
(as-> numbers $
  (map inc $)
  (filter odd? $)
  (first $)
  (hash-map :result $ :id 1))
;; => {:result 3 :id 1}
```

### The thread-some macros (`some->` and `some->>`)

Two of the more specialized threading macros that *ClojureScript* comes
with. They work in the same way as their analagous `->` and `->>` macros
with the additional support for short-circuiting the expression if one
of the expresions evaluates to `nil`.

Let’s see another example:

``` clojure
(some-> (rand-nth [1 nil])
        (inc))
;; => 2

(some-> (rand-nth [1 nil])
        (inc))
;; => nil
```

This is an easy way avoid null pointer exceptions.

### The thread-cond macros (`cond->` and `cond->>`)

The `cond->` and `cond->>` macros are analogous to `->` and `->>` that
offers the ability to conditionally skip some steps from the pipeline.
Let see an example:

``` clojure
(defn describe-number
  [n]
  (cond-> []
    (odd? n) (conj "odd")
    (even? n) (conj "even")
    (zero? n) (conj "zero")
    (pos? n) (conj "positive")))

(describe-number 3)
;; => ["odd" "positive"]

(describe-number 4)
;; => ["even" "positive"]
```

The value threading only happens when the corresponding condition
evaluates to logical true.

### Additional Readings

  - <http://www.spacjer.com/blog/2015/11/09/lesser-known-clojure-variants-of-threading-macro/>

  - <http://clojure.org/guides/threading_macros>

## Reader Conditionals

This language feature allows different dialects of Clojure to share
common code that is mostly platform independent but need some platform
dependent code.

To use reader conditionals, all you need is to rename your source file
with `.cljs` extension to one with `.cljc`, because reader conditionals
only work if they are placed in files with `.cljc` extension.

### Standard (`#?`)

There are two types of reader conditionals, standard and splicing. The
standard reader conditional behaves similarly to a traditional cond and
the syntax looks like this:

``` clojure
(defn parse-int
  [v]
  #?(:clj  (Integer/parseInt v)
     :cljs (js/parseInt v)))
```

As you can observe, `#?` reading macro looks very similar to cond, the
difference is that the condition is just a keyword that identifies the
platform, where `:cljs` is for *ClojureScript* and `:clj` is for
*Clojure*. The advantage of this approach, is that it is evaluated at
compile time so no runtime performance overhead exists for using this.

### Splicing (`#?@`)

The splicing reader conditional works in the same way as the standard
and allows splice lists into the containing form. The `#?@` reader macro
is used for that and the code looks like this:

``` clojure
(defn make-list
  []
  (list #?@(:clj  [5 6 7 8]
            :cljs [1 2 3 4])))

;; On ClojureScript
(make-list)
;; => (1 2 3 4)
```

The *ClojureScript* compiler will read that code as this:

``` clojure
(defn make-list
  []
  (list 1 2 3 4))
```

The splicing reader conditional can’t be used to splice multiple top
level forms, so the following code is ilegal:

``` clojure
#?@(:cljs [(defn func-a [] :a)
           (defn func-b [] :b)])
;; => #error "Reader conditional splicing not allowed at the top level."
```

If you need so, you can use multiple forms or just use `do` block for
group multiple forms together:

``` clojure
#?(:cljs (defn func-a [] :a))
#?(:cljs (defn func-b [] :b))

;; Or

#?(:cljs
   (do
     (defn func-a [] :a)
     (defn func-b [] :b)))
```

### More readings

  - <http://clojure.org/guides/reader_conditionals>

  - <https://danielcompton.net/2015/06/10/clojure-reader-conditionals-by-example>

  - <https://github.com/funcool/cuerdas> (example small project that
    uses reader conditionals)

## Namespaces

### Defining a namespace

The *namespace* is ClojureScript’s fundamental unit of code modularity.
Namespaces are analogous to Java packages or Ruby and Python modules and
can be defined with the `ns` macro. If you have ever looked at a little
bit of ClojureScript source, you may have noticed something like this at
the beginning of the file:

``` clojure
(ns myapp.core
  "Some docstring for the namespace.")

(def x "hello")
```

Namespaces are dynamic, meaning you can create one at any time. However,
the convention is to have one namespace per file. Naturally, a namespace
definition is usually at the beginning of the file, followed by an
optional docstring.

Previously we have explained vars and symbols. Every var that you define
will be associated with its namespace. If you do not define a concrete
namespace, then the default one called "cljs.user" will be used:

``` clojure
(def x "hello")
;; => #'cljs.user/x
```

### Loading other namespaces

Defining a namespace and the vars in it is really easy, but it’s not
very useful if we can’t use symbols from other namespaces. For this
purpose, the `ns` macro offers a simple way to load other namespaces.

Observe the following:

``` clojure
(ns myapp.main
  (:require myapp.core
            clojure.string))

(clojure.string/upper-case myapp.core/x)
;; => "HELLO"
```

As you can observe, we are using fully qualified names (namespace + var
name) for access to vars and functions from different namespaces.

While this will let you access other namespaces, it’s also repetitive
and overly verbose. It will be especially uncomfortable if the name of a
namespace is very long. To solve that, you can use the `:as` directive
to create an additional (usually shorter) alias to the namespace. This
is how it can be done:

``` clojure
(ns myapp.main
  (:require [myapp.core :as core]
            [clojure.string :as str]))

(str/upper-case core/x)
;; => "HELLO"
```

Additionally, *ClojureScript* offers a simple way to refer to specific
vars or functions from a concrete namespace using the `:refer`
directive, followed by a sequence of symbols that will refer to vars in
the namespace. Effectively, it is as if those vars and functions are now
part of your namespace, and you do not need to qualify them at all.

``` clojure
(ns myapp.main
  (:require [clojure.string :refer [upper-case]]))
(upper-case x)
;; => "HELLO"
```

And finally, you should know that everything located in the `cljs.core`
namespace is automatically loaded and you should not require it
explicitly. Sometimes you may want to declare vars that will clash with
some others defined in the `cljs.core` namespace. To do this, the `ns`
macro offers another directive that allows you to exclude specific
symbols and prevent them from being automatically loaded.

Observe the following:

``` clojure
(ns myapp.main
  (:refer-clojure :exclude [min]))

(defn min
  [x y]
  (if (> x y)
    y
    x))
```

The `ns` macro also has other directives for loading host classes (with
`:import`) and macros (with `:refer-macros`), but these are explained in
other sections.

### Namespaces and File Names

When you have a namespace like `myapp.core`, the code must be in a file
named *core.cljs* inside the *myapp* directory. So, the preceding
examples with namespaces `myapp.core` and `myapp.main` would be found in
project with a file structure like this:

    myapp
    └── src
        └── myapp
            ├── core.cljs
            └── main.cljs

## Abstractions and Polymorphism

I’m sure that at more than one time you have found yourself in this
situation: you have defined a great abstraction (using interfaces or
something similar) for your "business logic", and you have found the
need to deal with another module over which you have absolutely no
control, and you probably were thinking of creating adapters, proxies,
and other approaches that imply a great amount of additional complexity.

Some dynamic languages allow "monkey-patching"; languages where the
classes are open and any method can be defined and redefined at any
time. Also, it is well known that this technique is a very bad practice.

We can not trust languages that allow you to silently overwrite methods
that you are using when you import third party libraries; you cannot
expect consistent behavior when this happens.

These symptoms are commonly called the "expression problem"; see
<http://en.wikipedia.org/wiki/Expression_problem> for more details

### Protocols

The *ClojureScript* primitive for defining "interfaces" is called a
protocol. A protocol consists of a name and set of functions. All the
functions have at least one argument corresponding to the `this` in
JavaScript or `self` in Python.

Protocols provide a type-based polymorphism, and the dispatch is always
done by the first argument (equivalent to JavaScript’s `this`, as
previously mentioned).

A protocol looks like this:

``` clojure
(ns myapp.testproto)

(defprotocol IProtocolName
  "A docstring describing the protocol."
  (sample-method [this] "A doc string associated with this function."))
```

<div class="note">

the "I" prefix is commonly used to designate the separation of protocols
and types. In the Clojure community, there are many different opinions
about how the "I" prefix should be used. In our opinion, it is an
acceptable solution to avoid name clashing and possible confusion. But
not using the prefix is not considered bad practice.

</div>

From the user perspective, protocol functions are simply plain functions
defined in the namespace where the protocol is defined. This enables an
easy and simple aproach for avoid conflicts between different protocols
implemented for the same type that have conflicting function names.

Here is an example. Let’s create a protocol called `IInvertible` for
data that can be "inverted". It will have a single method named
`invert`.

``` clojure
(defprotocol IInvertible
  "This is a protocol for data types that are 'invertible'"
  (invert [this] "Invert the given item."))
```

#### Extending existing types

One of the big strengths of protocols is the ability to extend existing
and maybe third party types. This operation can be done in different
ways.

The majority of time you will tend to use the **extend-protocol** or the
**extend-type** macros. This is how `extend-type` syntax looks:

``` clojure
(extend-type TypeA
  ProtocolA
  (function-from-protocol-a [this]
    ;; implementation here
    )

  ProtocolB
  (function-from-protocol-b-1 [this parameter1]
    ;; implementation here
    )
  (function-from-protocol-b-2 [this parameter1 parameter2]
    ;; implementation here
    ))
```

You can observe that with **extend-type** you are extending a single
type with different protocols in a single expression.

Let’s play with our `IInvertible` protocol defined previously:

``` clojure
(extend-type string
  IInvertible
  (invert [this] (apply str (reverse this))))

(extend-type cljs.core.List
  IInvertible
  (invert [this] (reverse this)))

(extend-type cljs.core.PersistentVector
  IInvertible
  (invert [this] (into [] (reverse this))))
```

You may note that a special symbol **string** is used instead of
`js/String` for extend the protol for string. This is because the
builtin javascript types have special treatment and if you replace the
`string` with `js/String` the compiler will emit a warning about that.

So if you want extend your protocol to javascript primitive types,
instead of using `js/Number`, `js/String`, `js/Object`, `js/Array`,
`js/Boolean` and `js/Function` you should use the respective special
symbols: `number`, `string`, `object`, `array`, `boolean` and
`function`.

Now, it’s time to try our protocol implementation:

``` clojure
(invert "abc")
;; => "cba"

(invert 0)
;; => 0

(invert '(1 2 3))
;; => (3 2 1)

(invert [1 2 3])
;; => [3 2 1]
```

In comparison, **extend-protocol** does the inverse; given a protocol,
it adds implementations for multiple types. This is how the syntax
looks:

``` clojure
(extend-protocol ProtocolA
  TypeA
  (function-from-protocol-a [this]
    ;; implementation here
    )

  TypeB
  (function-from-protocol-a [this]
    ;; implementation here
    ))
```

Thus, the previous example could have been written equally well with
this way:

``` clojure
(extend-protocol IInvertible
  string
  (invert [this] (apply str (reverse this)))

  cljs.core.List
  (invert [this] (reverse this))

  cljs.core.PersistentVector
  (invert [this] (into [] (reverse this))))
```

#### Participate in ClojureScript abstractions

ClojureScript itself is built up on abstractions defined as protocols.
Almost all behavior in the *ClojureScript* language itself can be
adapted to third party libraries. Let’s look at a real life example.

In previous sections, we have explained the different kinds of built-in
collections. For this example we will use a **set**. See this snippet of
code:

``` clojure
(def mynums #{1 2})

(filter mynums [1 2 4 5 1 3 4 5])
;; => (1 2 1)
```

What happened? In this case, the *set* type implements the
*ClojureScript* internal `IFn` protocol that represents an abstraction
for functions or anything callable. This way it can be used like a
callable predicate in filter.

OK, but what happens if we want to use a regular expression as a
predicate function for filtering a collection of strings:

``` clojure
(filter #"^foo" ["haha" "foobar" "baz" "foobaz"])
;; TypeError: Cannot call undefined
```

The exception is raised because the `RegExp` type does not implement the
`IFn` protocol so it cannot behave like a callable, but that can be
easily fixed:

``` clojure
(extend-type js/RegExp
  IFn
  (-invoke
   ([this a]
     (re-find this a))))
```

Let’s analyze this: we are extending the `js/RegExp` type so that it
implements the `invoke` function in the `IFn` protocol. To invoke a
regular expression `a` as if it were a function, call the `re-find`
function with the object of the function and the pattern.

Now, you will be able use the regex instances as predicates in a filter
operation:

``` clojure
(filter #"^foo" ["haha" "foobar" "baz" "foobaz"])
;; => ("foobar" "foobaz")
```

#### Introspection using Protocols

*ClojureScript* comes with a useful function that allows runtime
introspection: `satisfies?`. The purpose of this function is to
determine at runtime if some object (instance of some type) satisfies
the concrete protocol.

So, with the previous examples, if we check if a `set` instance
satisfies an **IFn** protocol, it should return `true`:

``` clojure
(satisfies? IFn #{1})
;; => true
```

### Multimethods

We have previously talked about protocols which solve a very common use
case of polymorphism: dispatch by type. But in some circumstances, the
protocol approach can be limiting. And here, **multimethods** come to
the rescue.

These **multimethods** are not limited to type dispatch only; instead,
they also offer dispatch by types of multiple arguments and by value.
They also allow ad-hoc hierarchies to be defined. Also, like protocols,
multimethods are an "Open System", so you or any third parties can
extend a multimethod for new types.

The basic constructions of **multimethods** are the `defmulti` and
`defmethod` forms. The `defmulti` form is used to create the multimethod
with an initial dispatch function. This is a model of what it looks
like:

``` clojure
(defmulti say-hello
  "A polymorphic function that return a greetings message
  depending on the language key with default lang as `:en`"
  (fn [param] (:locale param))
  :default :en)
```

The anonymous function defined within the `defmulti` form is a dispatch
function. It will be called in every call to the `say-hello` function
and should return some kind of marker object that will be used for
dispatch. In our example, it returns the contents of the `:locale` key
of the first argument.

And finally, you should add implementations. That is done with the
`defmethod` form:

``` clojure
(defmethod say-hello :en
  [person]
  (str "Hello " (:name person "Anonymous")))

(defmethod say-hello :es
  [person]
  (str "Hola " (:name person "Anónimo")))
```

So, if you execute that function over a hash map containing the
`:locale` and optionally the `:name` key, the multimethod will first
call the dispatch function to determine the dispatch value, then it will
search for an implementation for that value. If an implementation is
found, the dispatcher will execute it. Otherwise, the dispatch will
search for a default implementation (if one is specified) and execute
it.

``` clojure
(say-hello {:locale :es})
;; => "Hola Anónimo"

(say-hello {:locale :en :name "Ciri"})
;; => "Hello Ciri"

(say-hello {:locale :fr})
;; => "Hello Anonymous"
```

If the default implementation is not specified, an exception will be
raised notifying you that some value does not have an implementation for
that multimethod.

### Hierarchies

Hierarchies are *ClojureScript*’s way to let you build whatever
relations that your domain may require. Hierarchies are defined in term
of relations between named objects, such as symbols, keywords, or types.

Hierarchies can be defined globally or locally, depending on your needs.
Like multimethods, hierarchies are not limited to a single namespace.
You can extend a hierarchy from any namespace, not only from the one in
which it is defined.

The global namespace is more limited, for good reasons. Keywords or
symbols that are not namespaced can not be used in the global hierarchy.
That behavior helps prevent unexpected situations when two or more third
party libraries use the same symbol for different semantics.

#### Defining a hierarchy

The hierarchy relations should be established using the `derive`
function:

``` clojure
(derive ::circle ::shape)
(derive ::box ::shape)
```

We have just defined a set of relationships between namespaced keywords.
In this case the `::circle` is a child of `::shape`, and `::box` is also
a child of `::shape`.

<div class="tip">

The `::circle` keyword syntax is a shorthand for `:current.ns/circle`.
So if you are executing it in a REPL, `::circle` will be evaluated as
`:cljs.user/circle`.

</div>

#### Hierarchies and introspection

*ClojureScript* comes with a little toolset of functions that allows
runtime introspection of globally or locally defined hierarchies. This
toolset consists of three functions: `isa?`, `ancestors`, and
`descendants`.

Let’s see an example of how it can be used with the hierarchy defined in
the previous example:

``` clojure
(ancestors ::box)
;; => #{:cljs.user/shape}

(descendants ::shape)
;; => #{:cljs.user/circle :cljs.user/box}

(isa? ::box ::shape)
;; => true

(isa? ::rect ::shape)
;; => false
```

#### Locally defined hierarchies

As we mentioned previously, in *ClojureScript* you also can define local
hierarchies. This can be done with the `make-hierarchy` function. Here
is an example of how you can replicate the previous example using a
local hierarchy:

``` clojure
(def h (-> (make-hierarchy)
           (derive :box :shape)
           (derive :circle :shape)))
```

Now you can use the same introspection functions with that locally
defined hierarchy:

``` clojure
(isa? h :box :shape)
;; => true

(isa? :box :shape)
;; => false
```

As you can observe, in local hierarchies we can use normal (not
namespace qualified) keywords, and if we execute the `isa?` without
passing the local hierarchy parameter, it returns `false` as expected.

#### Hierarchies in multimethods

One of the big advantages of hierarchies is that they work very well
together with multimethods. This is because multimethods by default use
the `isa?` function for the last step of dispatching.

Let’s see an example to clearly understand what that means. First, we
define the multimethod with the `defmulti` form:

``` clojure
(defmulti stringify-shape
  "A function that prints a human readable representation
  of a shape keyword."
  identity
  :hierarchy #'h)
```

With the `:hierarchy` keyword parameter, we indicate to the multimethod
what hierarchy we want to use; if it is not specified, the global
hierarchy will be used.

Second, we define an implementation for our multimethod using the
`defmethod` form:

``` clojure
(defmethod stringify-shape :box
  [_]
  "A box shape")

(defmethod stringify-shape :shape
  [_]
  "A generic shape")

(defmethod stringify-shape :default
  [_]
  "Unexpected object")
```

Now, let’s see what happens if we execute that function with a box:

``` clojure
(stringify-shape :box)
;; => "A box shape"
```

Now everything works as expected; the multimethod executes the direct
matching implementation for the given parameter. Next, let’s see what
happens if we execute the same function but with the `:circle` keyword
as the parameter which does not have the direct matching dispatch value:

``` clojure
(stringify-shape :circle)
;; => "A generic shape"
```

The multimethod automatically resolves it using the provided hierarchy,
and since `:circle` is a descendant of `:shape`, the `:shape`
implementation is executed.

Finally, if you give a keyword that isn’t part of the hierarchy, you get
the `:default` implementation:

``` clojure
(stringify-shape :triangle)
;; => "Unexpected object"
```

## Data types

Until now, we have used maps, sets, lists, and vectors to represent our
data. And in most cases, this is a really great approach. But sometimes
we need to define our own types, and in this book we will call them
**data types**.

A data type provides the following:

  - A unique host-backed type, either named or anonymous.

  - The ability to implement protocols (inline).

  - Explicitly declared structure using fields or closures.

  - Map-like behavior (via records, see below).

### Deftype

The most low-level construction in *ClojureScript* for creating your own
types is the `deftype` macro. As a demonstration, we will define a type
called `User`:

``` clojure
(deftype User [firstname lastname])
```

Once the type has been defined, we can create an instance of our `User`.
In the following example, the `.` after `User` indicates that we are
calling a constructor.

``` clojure
(def person (User. "Triss" "Merigold"))
```

Its fields can be accessed using the prefix dot notation:

``` clojure
(.-firstname person)
;; => "Triss"
```

Types defined with `deftype` (and `defrecord`, which we will see later)
create a host-backed class-like object associated with the current
namespace. For convenience, *ClojureScript* also defines a constructor
function called `→User` that can be imported using the `:require`
directive.

We personally do not like this type of function, and we prefer to define
our own constructors with more idiomatic names:

``` clojure
(defn make-user
  [firstname lastname]
  (User. firstname lastname))
```

We use this in our code instead of `→User`.

### Defrecord

The record is a slightly higher-level abstraction for defining types in
*ClojureScript* and should be the preferred way to do it.

As we know, *ClojureScript* tends to use plain data types such as maps,
but in most cases we need a named type to represent the entities of our
application. Here come the records.

A record is a data type that implements the map protocol and therefore
can be used like any other map. And since records are also proper types,
they support type-based polymorphism through protocols.

In summary: with records, we have the best of both worlds, maps that can
play in different abstractions.

Let’s start defining the `User` type but using records:

``` clojure
(defrecord User [firstname lastname])
```

It looks really similar to the `deftype` syntax; in fact, it uses
`deftype` behind the scenes as a low-level primitive for defining types.

Now, look at the difference with raw types for access to its fields:

``` clojure
(def person (User. "Yennefer" "of Vengerberg"))

(:firstname person)
;; => "Yennefer"

(get person :firstname)
;; => "Yennefer"
```

As we mentioned previously, records are maps and act like them:

``` clojure
(map? person)
;; => true
```

And like maps, they support extra fields that are not initially defined:

``` clojure
(def person2 (assoc person :age 92))

(:age person2)
;; => 92
```

As we can see, the `assoc` function works as expected and returns a new
instance of the same type but with new key value pair. But take care
with `dissoc`\! Its behavior with records is slightly different than
with maps; it will return a new record if the field being dissociated is
an optional field, but it will return a plain map if you dissociate a
mandatory field.

Another difference with maps is that records do not act like functions:

``` clojure
(def plain-person {:firstname "Yennefer", :lastname "of Vengerberg"})

(plain-person :firstname)
;; => "Yennefer"

(person :firstname)
;; => person.User does not implement IFn protocol.
```

For convenience, the `defrecord` macro, like `deftype`, exposes a
`→User` function, as well as an additional `map→User` constructor
function. We have the same opinion about that constructor as with
`deftype` defined ones: we recommend defining your own instead of using
the other ones. But as they exist, let’s see how they can be used:

``` clojure
(def cirilla (->User "Cirilla" "Fiona"))
(def yen (map->User {:firstname "Yennefer"
                     :lastname "of Vengerberg"}))
```

### Implementing protocols

Both type definition primitives that we have seen so far allow inline
implementations for protocols (explained in a previous section). Let’s
define one for example purposes:

``` clojure
(defprotocol IUser
  "A common abstraction for working with user types."
  (full-name [_] "Get the full name of the user."))
```

Now, you can define a type with inline implementation for an
abstraction, in our case the `IUser`:

``` clojure
(defrecord User [firstname lastname]
  IUser
  (full-name [_]
    (str firstname " " lastname)))

;; Create an instance.
(def user (User. "Yennefer" "of Vengerberg"))

(full-name user)
;; => "Yennefer of Vengerberg"
```

### Reify

The `reify` macro is an *ad hoc constructor* you can use to create
objects without pre-defining a type. Protocol implementations are
supplied the same as `deftype` and `defrecord`, but in contrast, `reify`
does not have accessible fields.

This is how we can emulate an instance of the user type that plays well
with the `IUser` abstraction:

``` clojure
(defn user
  [firstname lastname]
  (reify
    IUser
    (full-name [_]
      (str firstname " " lastname))))

(def yen (user "Yennefer" "of Vengerberg"))
(full-name yen)
;; => "Yennefer of Vengerberg"
```

### Specify

`specify!` is an advanced alternative to `reify`, allowing you to add
protocol implementations to an existing JavaScript object. This can be
useful if you want to graft protocols onto a JavaScript library’s
components.

``` clojure
(def obj #js {})

(specify! obj
  IUser
  (full-name [_]
    "my full name"))

(full-name obj)
;; => "my full name"
```

`specify` is an immutable version of `specify!` that can be used on
immutable, copyable values implementing `ICloneable` (e.g. ClojureScript
collections).

``` clojure
(def a {})

(def b (specify a
         IUser
         (full-name [_]
           "my full name")))

(full-name a)
;; Error: No protocol method IUser.full-name defined for type cljs.core/PersistentArrayMap: {}

(full-name b)
;; => "my full name"
```

## Host interoperability

*ClojureScript*, in the same way as its brother Clojure, is designed to
be a "guest" language. This means that the design of the language works
well on top of an existing ecosystem such as JavaScript for
*ClojureScript* and the JVM for *Clojure*.

### The types

*ClojureScript*, unlike what you might expect, tries to take advantage
of every type that the platform provides. This is a (perhaps incomplete)
list of things that *ClojureScript* inherits and reuses from the
underlying platform:

  - *ClojureScript* strings are JavaScript **Strings**.

  - *ClojureScript* numbers are JavaScript **Numbers**.

  - *ClojureScript* `nil` is a JavaScript **null**.

  - *ClojureScript* regular expressions are JavaScript `RegExp`
    instances.

  - *ClojureScript* is not interpreted; it is always compiled down to
    JavaScript.

  - *ClojureScript* allows easy call to platform APIs with the same
    semantics.

  - *ClojureScript* data types internally compile to objects in
    JavaScript.

On top of it, *ClojureScript* builds its own abstractions and types that
do not exist in the platform, such as Vectors, Maps, Sets, and others
that are explained in preceding sections of this chapter.

### Interacting with platform types

*ClojureScript* comes with a little set of special forms that allows it
to interact with platform types such as calling object methods, creating
new instances, and accessing object properties.

#### Access to the platform

*ClojureScript* has a special syntax for access to the entire platform
environment through the `js/` special namespace. This is an example of
an expression to execute JavaScript’s built-in `parseInt` function:

``` clojure
(js/parseInt "222")
;; => 222
```

#### Creating new instances

*ClojureScript* has two ways to create instances:

Using the `new` special form

``` clojure
(new js/RegExp "^foo$")
```

Using the `.` special form

``` clojure
(js/RegExp. "^foo$")
```

The last one is the recommended way to create instances. We are not
aware of any real differences between the two forms, but in the
ClojureScript community, the last one is used most often.

#### Invoke instance methods

To invoke methods of some object instance, as opposed to how it is done
in JavaScript (e.g., `obj.method()`, the method name comes first like
any other standard function in Lisp languages but with a little
variation: the function name starts with special form `.`.

Let’s see how we can call the `.test()` method of a regexp instance:

``` clojure
(def re (js/RegExp "^Clojure"))

(.test re "ClojureScript")
;; => true
```

You can invoke instance methods on JavaScript objects. The first example
follows the pattern you have seen; the last one is a shortcut:

``` clojure
(.sqrt js/Math 2)
;; => 1.4142135623730951
(js/Math.sqrt 2)
;; => 1.4142135623730951
```

#### Access to object properties

Access to an object’s properties is really very similar to calling a
method. The difference is that instead of using the `.` you use `.-`.
Let’s see an example:

``` clojure
(.-multiline re)
;; => false
(.-PI js/Math)
;; => 3.141592653589793
```

#### Property access shorthand

Symbols with the `js/` prefix can contain dots to denote nested property
access. Both of the following expressions invoke the same function:

``` clojure
(.log js/console "Hello World")

(js/console.log "Hello World")
```

And both of the following expressions access the same property:

``` clojure
(.-PI js/Math)
;; => 3.141592653589793

js/Math.PI
;; => 3.141592653589793
```

#### JavaScript objects

*ClojureScript* has different ways to create plain JavaScript objects;
each one has its own purpose. The basic one is the `js-obj` function. It
accepts a variable number of pairs of keys and values and returns a
JavaScript object:

``` clojure
(js-obj "country" "FR")
;; => #js {:country "FR"}
```

The return value can be passed to some kind of third party library that
accepts a plain JavaScript object, but you can observe the real
representation of the return value of this function. It is really
another form for doing the same thing.

Using the reader macro `#js` consists of prepending it to a
ClojureScript map or vector, and the result will be transformed to plain
JavaScript:

``` clojure
(def myobj #js {:country "FR"})
```

The translation of that to plain JavaScript is similar to this:

``` javascript
var myobj = {country: "FR"};
```

As explained in the previous section, you can also access the plain
object properties using the `.-` syntax:

``` clojure
(.-country myobj)
;; => "FR"
```

And as JavaScript objects are mutable, you can set a new value for some
property using the `set!` function:

``` clojure
(set! (.-country myobj) "KR")
```

#### Conversions

The inconvenience of the previously explained forms is that they do not
make recursive transformations, so if you have nested objects, the
nested objects will not be converted. Consider this example that uses
Clojurescript maps, then a similar one with JavaScript objects:

``` clojure
(def clj-map {:country {:code "FR" :name "France"}})
;; => {:country {:code "FR", :name "France"}}
(:code (:country clj-map))
;; => "FR"

(def js-obj #js {:country {:code "FR" :name "France"}})
;; => #js {:country {:code "FR", :name "France"}
(.-country js-obj)
;; => {:code "FR", :name "France"}
(.-code (.-country js-obj)
;; => nil
```

To solve that use case, *ClojureScript* comes with the `clj→js` and
`js→clj` functions that transform Clojure collection types into
JavaScript and back. Note that the conversion to ClojureScript changes
the `:country` keyword to a string.

``` clojure
(clj->js {:foo {:bar "baz"}})
;; => #js {:foo #js {:bar "baz"}}
(js->clj #js {:country {:code "FR" :name "France"}}))
;; => {"country" {:code "FR", :name "France"}}
```

In the case of arrays, there is a specialized function `into-array` that
behaves as expected:

``` clojure
(into-array ["France" "Korea" "Peru"])
;; => #js ["France" "Korea" "Peru"]
```

#### Arrays

In the previous example, we saw how we can create an array from an
existing *ClojureScript* collection. But there is another function for
creating arrays: `make-array`.

**Creating a preallocated array with length 10.**

``` clojure
(def a (make-array 10))
;; => #js [nil nil nil nil nil nil nil nil nil nil]
```

In *ClojureScript*, arrays also play well with sequence abstractions, so
you can iterate over them or simply get the number of elements with the
`count` function:

``` clojure
(count a)
;; => 10
```

As arrays in the JavaScript platform are a mutable collection type, you
can access a concrete index and set the value at that position:

``` clojure
(aset a 0 2)
;; => 2
a
;; => #js [2 nil nil nil nil nil nil nil nil nil]
```

Or access in an indexed way to get its values:

``` clojure
(aget a 0)
;; => 2
```

In JavaScript, array index access is equivalent to object property
access, so you can use the same functions for interacting with plain
objects:

``` clojure
(def b #js {:hour 16})
;; => #js {:hour 16}

(aget b "hour")
;; => 16

(aset b "minute" 22)
;; => 22

b
;; => #js {:hour 16, :minute 22}
```

## State management

We’ve learned that one of ClojureScript’s fundamental ideas is
immutability. Both scalar values and collections are immutable in
ClojureScript, except those mutable types present in the JS host like
`Date`.

Immutability has many great properties but we are sometimes faced with
the need to model values that change over time. How can we achieve this
if we can’t change data structures in place?

### Vars

Vars can be redefined at will inside a namespace but there is no way to
know **when** they change. The inability to redefine vars from other
namespaces is a bit limiting; also, if we are modifying state, we’re
probably interested in knowing when it occurs.

### Atoms

ClojureScript gives us the `Atom` type, which is an object containing a
value that can be altered at will. Besides altering its value, it also
supports observation through watcher functions that can be attached and
detached from it and validation for ensuring that the value contained in
the atom is always valid.

If we were to model an identity corresponding to a person called Ciri,
we could wrap an immutable value containing Ciri’s data in an atom. Note
that we can get the atom’s value with the `deref` function or using its
shorthand `@` notation:

``` clojure
(def ciri (atom {:name "Cirilla" :lastname "Fiona" :age 20}))
;; #<Atom: {:name "Cirilla", :lastname "Fiona", :age 20}>

(deref ciri)
;; {:name "Cirilla", :lastname "Fiona", :age 20}

@ciri
;; {:name "Cirilla", :lastname "Fiona", :age 20}
```

We can use the `swap!` function on an atom to alter its value with a
function. Since Ciri’s birthday is today, let’s increment her age count:

``` clojure
(swap! ciri update :age inc)
;; {:name "Cirilla", :lastname "Fiona", :age 21}

@ciri
;; {:name "Cirilla", :lastname "Fiona", :age 21}
```

The `reset!` functions replaces the value contained in the atom with a
new one:

``` clojure
(reset! ciri {:name "Cirilla", :lastname "Fiona", :age 22})
;; {:name "Cirilla", :lastname "Fiona", :age 22}

@ciri
;; {:name "Cirilla", :lastname "Fiona", :age 22}
```

#### Observation

We can add and remove watcher functions for atoms. Whenever the atom’s
value is changed through a `swap!` or `reset!`, all the atom’s watcher
functions will be called. Watchers are added with the `add-watch`
function. Notice that each watcher has a key associated (`:logger` in
the example) to it which is later used to remove the watch from the
atom.

``` clojure
(def a (atom))

(add-watch a :logger (fn [key the-atom old-value new-value]
                       (println "Key:" key "Old:" old-value "New:" new-value)))

(reset! a 42)
;; Key: :logger Old: nil New: 42
;; => 42

(swap! a inc)
;; Key: :logger Old: 42 New: 43
;; => 43

(remove-watch a :logger)
```

### Volatiles

Volatiles, like atoms, are objects containing a value that can be
altered. However, they don’t provide the observation and validation
capabilities that atoms provide. This makes them slightly more
performant and a more suitable mutable container to use inside stateful
functions that don’t need observation nor validation.

Their API closely resembles that of atoms. They can be dereferenced to
grab the value they contain and support swapping and resetting with
`vswap!` and `vreset!` respectively:

``` clojure
(def ciri (volatile! {:name "Cirilla" :lastname "Fiona" :age 20}))
;; #<Volatile: {:name "Cirilla", :lastname "Fiona", :age 20}>

(volatile? ciri)
;; => true

(deref ciri)
;; {:name "Cirilla", :lastname "Fiona", :age 20}

(vswap! ciri update :age inc)
;; {:name "Cirilla", :lastname "Fiona", :age 21}

(vreset! ciri {:name "Cirilla", :lastname "Fiona", :age 22})
;; {:name "Cirilla", :lastname "Fiona", :age 22}
```

Note that another difference with atoms is that the constructor of
volatiles uses a bang at the end. You create volatiles with `volatile!`
and atoms with `atom`.

# Tooling & Compiler

This chapter will cover a little introduction to existing tooling for
making things easy when developing using ClojureScript. It will cover:

  - Using the REPL

  - Leiningen and cljsbuild

  - Google Closure Library

  - Modules

  - Unit testing

  - Library development

  - Browser based development

  - Server based development

Unlike the previous chapter, this chapter intends to tell different
stories each independent of the other.

## Getting Started with the Compiler

At this point, you are surely very bored with the constant theoretical
explanations about the language itself and will want to write and
execute some code. The goal of this section is to provide a little
practical introduction to the *ClojureScript* compiler.

The *ClojureScript* compiler takes the source code that has been split
over numerous directories and namespaces and compiles it down to
JavaScript. Today, JavaScript has a great number of different
environments where it can be executed - each with its own peculiarities.

This chapter intends to explain how to use *ClojureScript* without any
additional tooling. This will help you understand how the compiler works
and how you can use it when other tooling is not available (such as
[leiningen](http://leiningen.org/)  
[cljsbuild](https://github.com/emezeske/lein-cljsbuild) or
[boot](http://boot-clj.com/)).

### Execution environments

What is an execution environment? An execution environment is an engine
where JavaScript can be executed. For example, the most popular
execution environment is a browser (Chrome, Firefox, …​) followed by the
second most popular - [nodejs](https://nodejs.org/).

There are others, such as Rhino (JDK 6+), Nashorn (JDK 8), QtQuick
(QT),…​ but none of them have significant differences from the first
two. So, *ClojureScript* at the moment may compile code to run in the
browser or in nodejs-like environments out of the box.

### Download the compiler

Although the *ClojureScript* is self hosted, the best way to use it is
just using the JVM implementation. To use it, you should have jdk8
installed. *ClojureScript* itself only requires JDK 7, but the
standalone compiler that we are going to use in this chapter requires
JDK 8, which can be found at
<http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html>

You can download the latest *ClojureScript* compiler using `wget`:

``` bash
wget https://github.com/clojure/clojurescript/releases/download/r1.9.36/cljs.jar
```

The *ClojureScript* compiler is packaged in a standalone executable jar
file, so this is the only file (along with JDK 8) that you need to
compile your *ClojureScript* source code to JavaScript.

### Compile for Node.js

Let’s start with a practical example compiling code that will target
**Node.js** (hereafter simply "nodejs"). For this example, you should
have nodejs installed.

There are different ways to install nodejs, but the recommended way is
using nvm ("Node.js Version Manager"). You can read the instructions on
how to install and use nvm on its [home
page](https://github.com/creationix/nvm).

When you have installed nvm, follow installing the latest version of
nodejs:

``` shell
nvm install v6.2.0
nvm alias default v6.2.0
```

You can test if **nodejs** is installed in your system with this
command:

``` shell
$ node --version
v6.2.0
```

#### Create the example application

For the first step of our practical example, we will create our
application directory structure and populate it with example code.

Start by creating the directory tree structure for our “hello world”
application:

``` bash
mkdir -p myapp/src/myapp
touch myapp/src/myapp/core.cljs
```

Resulting in this directory tree:

``` text
myapp
└── src
    └── myapp
        └── core.cljs
```

Second, write the example code into the previously created
`myapp/src/myapp/core.cljs` file:

``` clojure
(ns myapp.core
  (:require [cljs.nodejs :as nodejs]))

(nodejs/enable-util-print!)

(defn -main
  [& args]
  (println "Hello world!"))

(set! *main-cli-fn* -main)
```

<div class="note">

It is very important that the declared namespace in the file exactly
matches the directory structure. This is the way *ClojureScript*
structures its source code.

</div>

#### Compile the example application

In order to compile that source code, we need a simple build script that
tells the *ClojureScript* compiler the source directory and the output
file. *ClojureScript* has a lot of other options, but at this moment we
can ignore that.

Let’s create the *myapp/build.clj* file with the following content:

``` clojure
(require '[cljs.build.api :as b])

(b/build "src"
 {:main 'myapp.core
  :output-to "main.js"
  :output-dir "out"
  :target :nodejs
  :verbose true})
```

This is a brief explanation of the compiler options used in this
example:

  - The `:output-to` parameter indicates to the compiler the destination
    of the compiled code, in this case to the "main.js" file.

  - The `:main` property indicates to the compiler the namespace that
    will act as the entry point of your application when it’s executed.

  - The `:target` property indicates the platform where you want to
    execute the compiled code. In this case, we are going to use
    **nodejs**. If you omit this parameter, the source will be compiled
    to run in the browser environment.

To run the compilation, just execute the following command:

``` bash
cd myapp
java -cp ../cljs.jar:src clojure.main build.clj
```

And when it finishes, execute the compiled file using **node**:

``` shell
$ node main.js
Hello world!
```

### Compile for the Browser

In this section we are going to create an application similar to the
"hello world" example from the previous section to run in the browser
environment. The minimal requirement for this application is just a
browser that can execute JavaScript.

The process is almost the same, and the directory structure is the same.
The only things that changes is the entry point of the application and
the build script. So, start re-creating the directory tree from previous
example in a different directory.

``` bash
mkdir -p mywebapp/src/mywebapp
touch mywebapp/src/mywebapp/core.cljs
```

Resulting in this directory tree:

``` text
mywebapp
└── src
    └── mywebapp
        └── core.cljs
```

Then, write new content to the `mywebapp/src/mywebapp/core.cljs` file:

``` clojure
(ns mywebapp.core)

(enable-console-print!)

(println "Hello world!")
```

In the browser environment we do not need a specific entry point for the
application, so the entry point is the entire namespace.

#### Compile the example application

In order to compile the source code to run properly in a browser,
overwrite the *mywebapp/build.clj* file with the following content:

``` clojure
(require '[cljs.build.api :as b])

(b/build "src"
 {:output-to "main.js"
  :output-dir "out/"
  :source-map true
  :main 'mywebapp.core
  :verbose true
  :optimizations :none})
```

This is a brief explanation of the compiler options we’re using:

  - The `:output-to` parameter indicates to the compiler the destination
    of the compiled code, in this case the "main.js" file.

  - The `:main` property indicates to the compiler the namespace that
    will act as the entry point of your application when it’s executed.

  - `:source-map` indicates the destination of the source map. (The
    source map connects the ClojureScript source to the generated
    JavaScript so that error messages can point you back to the original
    source.)

  - `:output-dir` indicates the destination directory for all file
    sources used in a compilation. It is just for making source maps
    work properly with the rest of the code, not only your source.

  - `:optimizations` indicates the compilation optimization. There are
    different values for this option, but that will be covered in
    subsequent sections in more detail.

To run the compilation, just execute the following command:

``` bash
cd mywebapp;
java -cp ../cljs.jar:src clojure.main build.clj
```

This process can take some time, so do not worry; wait a little bit. The
JVM bootstrap with the Clojure compiler is slightly slow. In the
following sections, we will explain how to start a watch process to
avoid constantly starting and stopping this slow process.

While waiting for the compilation, let’s create a dummy HTML file to
make it easy to execute our example app in the browser. Create the
*index.html* file with the following content; it goes in the main
*mywebapp* directory.

``` html
<!DOCTYPE html>
<html>
  <header>
    <meta charset="utf-8" />
    <title>Hello World from ClojureScript</title>
  </header>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

Now, when the compilation finishes and you have the basic HTML file you
can just open it with your favorite browser and take a look in the
development tools console. The "Hello world\!" message should appear
there.

### Watch process

You may have already noticed the slow startup time of the
*ClojureScript* compiler. To solve this, the *ClojureScript* standalone
compiler comes with a tool to watch for changes in your source code, and
re-compile modified files as soon as they are written to disk

Start by creating another build script, but this time name it
*watch.clj*:

``` clojure
(require '[cljs.build.api :as b])

(b/watch "src"
 {:output-to "main.js"
  :output-dir "out/"
  :source-map true
  :main 'mywebapp.core
  :optimizations :none})
```

Now, execute the script just like you have in previous sections:

``` bash
$ java -cp ../cljs.jar:src clojure.main watch.clj
Building ...
Reading analysis cache for jar:file:/home/niwi/cljsbook/playground/cljs.jar!/cljs/core.cljs
Compiling src/mywebapp/core.cljs
Compiling out/cljs/core.cljs
Using cached cljs.core out/cljs/core.cljs
... done. Elapsed 0.754487937 seconds
Watching paths: /home/niwi/cljsbook/playground/mywebapp/src
```

Go back to the `mywebapp.core` namespace, and change the print text to
`"Hello World,
Again!"`. You’ll see that the file `src/mywebapp/core.cljs` the file is
immediately recompiled, and if you reload `index.html` in your browser
the new text is displayed in the developer console. Another advantage of
this method is that it gives a little bit more output.

### Optimization levels

The *ClojureScript* compiler has different levels of optimization.
Behind the scenes, those compilation levels are coming from the Google
Closure Compiler.

A simplified overview of the compilation process is:

1.  The reader reads the code and does some analysis. This compiler may
    raise some warnings during this phase.

2.  Then, the *ClojureScript* compiler emits JavaScript code. The result
    is one JavaScript output file for each ClojureScript input file.

3.  The generated JavaScript files are passed through the Google Closure
    Compiler which, depending on the optimization level and other
    options (sourcemaps, output dir output to, …​), generates the final
    output file(s).

The final output format depends on the optimization level chosen:

#### none

This optimization level causes the generated JavaScript to be written
into separate output files for each namespace, without any additional
transformations to the code.

#### whitespace

This optimization level causes the generated JavaScript files to be
concatenated into a single output file, in dependency order. Line breaks
and other whitespace are removed.

This reduces compilation speed somewhat, resulting in a slower
compilations. However, it is not terribly slow and it is quite usable
for small-to-medium sized applications.

#### simple

The simple compilation level builds on the work from the `whitespace`
optimization level, and additionally performs optimizations within
expressions and functions, such as renaming local variables and function
parameters to have shorter names.

Compilation with the `:simple` optimization always preserves the
functionality of syntactically valid JavaScript, so it does not
interfere with the interaction between the compiled *ClojureScript* and
other JavaScript.

#### advanced

The advanced compilation level builds on the `simple` optimization
level, and additionally performs more aggressive optimizations and dead
code elimination. This results in a significantly smaller output file.

The `:advanced` optimizations only work for a strict subset of
JavaScript which follows the Google Closure Compiler rules.
*ClojureScript* generates valid JavaScript within this strict subset,
but if you are interacting with third party JavaScript code, some
additional work is required to make everything work as expected.

This interaction with third party javascript libraries will be explained
in later sections.

## Working with the REPL

### Introduction

Although you can create a source file and compile it every time you want
to try something out in ClojureScript, it’s easier to use the REPL. REPL
stands for:

  - Read - get input from the keyboard

  - Evaluate the input

  - Print the result

  - Loop back for more input

In other words, the REPL lets you try out ClojureScript concepts and get
immediate feedback.

*ClojureScript* comes with support for executing the REPL in different
execution environments, each of which has its own advantages and
disadvantages. For example, you can run a REPL in nodejs but in that
environment you don’t have any access to the DOM. Which REPL environment
is best for you depends on your specific needs and requirements.

### Nashorn REPL

The Nashorn REPL is the easiest and perhaps most painless REPL
environment because it does not require any special stuff, just the JVM
(JDK 8) that you have used in previous examples for running the
*ClojureScript* compiler.

Let’s start creating the *repl.clj* file with the following content:

``` clojure
(require '[cljs.repl]
         '[cljs.repl.nashorn])

(cljs.repl/repl
 (cljs.repl.nashorn/repl-env)
 :output-dir "out"
 :cache-analysis true)
```

Then, execute the following command to get the REPL up and running:

``` bash
$ java -cp cljs.jar:src clojure.main repl.clj
To quit, type: :cljs/quit
cljs.user=> (+ 1 2)
3
```

You may have noticed that the REPL does not have support for history and
other shell-like facilities. This is because the default REPL does not
comes with "readline" support. But this problem can be solved using a
simple tool named `rlwrap` which you should be able to find find with
the package manager of your operating system (e.g. for Ubuntu, type
`sudo apt install -y rlwrap` to install).

The `rlwrap` tool gives the REPL "readline" capability, and will allow
you to have command history, code navigation, and other shell-like
utilities that will make your REPL experience much more pleasant. To use
it, just prepend it to the previous command that we used to start the
REPL:

``` bash
$ rlwrap java -cp cljs.jar:src clojure.main repl.clj
To quit, type: :cljs/quit
cljs.user=> (+ 1 2)
3
```

### Node.js REPL

You must, of course, have nodejs installed on your system to use this
REPL.

You may be wondering why we might want a nodejs REPL, when we already
have the nashorn REPL available which doesn’t have any external
dependencies. The answer is very simple: nodejs is the most used
JavaScript execution environment on the backend, and it has a great
number of community packages built around it.

The good news is that starting a nodejs REPL is very easy once you have
it installed in your system. Start writing this content to a new
`repl.clj` file:

``` clojure
(require '[cljs.repl]
         '[cljs.repl.node])

(cljs.repl/repl
 (cljs.repl.node/repl-env)
 :output-dir "out"
 :cache-analysis true)
```

And start the REPL like you have done it previously with nashorn REPL:

``` bash
$ rlwrap java -cp cljs.jar:src clojure.main repl.clj
To quit, type: :cljs/quit
cljs.user=> (+ 1 2)
3
```

### Browser REPL

This REPL is the most laborious to get up and running. This is because
it uses a browser for its execution environment and it has additional
requirements.

Let’s start by creating a file named `brepl.clj` with the following
content:

``` clojure
(require
  '[cljs.build.api :as b]
  '[cljs.repl :as repl]
  '[cljs.repl.browser :as browser])

(b/build "src"
 {:output-to "main.js"
  :output-dir "out/"
  :source-map true
  :main 'myapp.core
  :verbose true
  :optimizations :none})

(repl/repl (browser/repl-env)
  :output-dir "out")
```

This script builds the source, just as we did earlier, and then starts
the REPL.

But the browser REPL also requires that some code be executed in the
browser before the REPL gets working. To do that, just re-create the
application structure very similar to the one that we have used in
previous sections:

``` bash
mkdir -p src/myapp
touch src/myapp/core.cljs
```

Then, write new content to the `src/myapp/core.cljs` file:

``` clojure
(ns myapp.core
 (:require [clojure.browser.repl :as repl]))

(defonce conn
  (repl/connect "http://localhost:9000/repl"))

(enable-console-print!)

(println "Hello, world!")
```

And finally, create the missing *index.html* file that is going to be
used as the entry point for running the browser side code of the REPL:

``` html
<!DOCTYPE html>
<html>
  <header>
    <meta charset="utf-8" />
    <title>Hello World from ClojureScript</title>
  </header>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

Well, that was a lot of setup\! But trust us, it’s all worth it when you
see it in action. To do that, just execute the `brepl.clj` in the same
way that we have done it in previous examples:

``` bash
$ rlwrap java -cp cljs.jar:src clojure.main brepl.clj
Compiling client js ...
Waiting for browser to connect ...
```

And finally, open your favourite browser and go to
<http://localhost:9000/>. Once the page is loaded (the page will be
blank), switch back to the console where you have run the REPL and you
will see that it is up and running:

``` bash
[...]
To quit, type: :cljs/quit
cljs.user=> (+ 14 28)
42
```

One of the big advantages of the browser REPL is that you have access to
everything in the browser environment. For example, type `(js/alert
"hello world")` in the REPL. This will cause the browser to display an
alert box. Nice\!

## The Closure Library

The Google Closure Library is a javascript library developed by Google.
It has a modular architecture, and provides cross-browser functions for
DOM manipulations and events, ajax and JSON, and other features.

The Google Closure Library is written specifically to take advantage of
the Closure Compiler (which is used internally by the *ClojureScript*
compiler).

*ClojureScript* is built on the Google Closure Compiler and Closure
Library. In fact, *ClojureScript* namespaces are Closure modules. This
means that you can interact with the Closure Library very easily:

``` clojure
(ns yourapp.core
  (:require [goog.dom :as dom]))

(def element (dom/getElement "body"))
```

This code snippet shows how you can import the **dom** module of the
Closure library and use a function declared in that module.

Additionally, the closure library exposes "special" modules that behave
like a class or object. To use these features, you must use the
`:import` directive in the `(ns
…​)` form:

``` clojure
(ns yourapp.core
  (:import goog.History))

(def instance (History.))
```

In a *Clojure* program, the `:import` directive is used for host (Java)
interop to import Java classes. If, however, you define types (classes)
in *ClojureScript*, you should use the standard `:require` directive and
not the `:import` directive.

## Dependency management

Until now, we have used the builtin *ClojureScript* toolchain to compile
our source files to JavaScript. This is the minimal setup required for
working with and understanding the compiler. For larger projects,
however, we often want to use a more powerful build tool that can manage
a project’s dependencies on other libraries.

For this reason, the remainder of this chapter will explain how to use
**Leiningen**, the de facto clojure build and dependency management
tool, for building *ClojureScript* projects. The **boot** build tool is
also growing in popularity, but for the purposes of this book we will
limit ourselves to Leiningen.

### Installing leiningen

The installation process of leiningen is quite simple; just follow these
steps:

``` bash
mkdir ~/bin
cd ~/bin
wget https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein
chmod a+x ./lein
export PATH=$PATH:~/bin
```

Make sure that the `~/bin` directory is always set on your path. To make
it permanent, add the line starting with `export` to your `~/.bashrc`
file (assuming you are using the bash shell).

Now, open another clean terminal and execute `lein version`. You should
see something like the following:

``` bash
$ lein version
Leiningen 2.5.1 on Java 1.8.0_45 OpenJDK 64-Bit Server VM
```

<div class="note">

We assume here that you are using a Unix-like system such as Linux or
BSD. If you are a Windows user, please check the instructions on the
[Leiningen homepage](http://leiningen.org/). You can also get the
Linux/Mac OS X/BSD version of the leiningen script at the web site.

</div>

### First project

The best way to show how a tool works is by creating a toy project with
it. In this case, we will create a small application that determines if
a year is a leap year or not. To start, we will use the **mies**
leiningen template.

<div class="note">

Templates are a facility in leiningen for creating an initial project
structure. The clojure community has a great many of them. In this case
we’ll use the **mies** template that was started by the clojurescript
core developer. Consult the leiningen docs to learn more about
templates.

</div>

Let’s start creating the project layout:

``` bash
$ lein new mies leapyears
$ cd leapyears # move into newly created project directory
```

The project has the following structure:

    leapyears
    ├── index.html
    ├── project.clj
    ├── README.md
    ├── scripts
    │   ├── build
    │   ├── release
    │   ├── watch
    │   ├── repl
    │   └── brepl
    └── src
        └── leapyears
            └── core.cljs

The `project.clj` file contains information that Leiningen uses to
download dependencies and build the project. For now, just trust that
everything in that file is exactly as it should be.

Open the `index.html` file and add the following content at the
beginning of body:

``` html
<section class="viewport">
  <div id="result">
    ----
  </div>
  <form action="" method="">
    <label for="year">Enter a year</label>
    <input id="year" name="year" />
  </form>
</section>
```

The next step is adding some code to make the form interactive. Put the
following code into the `src/leapyears/core.cljs`:

``` clojure
(ns leapyears.core
  (:require [goog.dom :as dom]
            [goog.events :as events]
            [cljs.reader :refer (read-string)]))

(enable-console-print!)

(def input (dom/getElement "year"))
(def result (dom/getElement "result"))

(defn leap?
  [year]
  (or (zero? (js-mod year 400))
      (and (pos? (js-mod year 100))
           (zero? (js-mod year 4)))))

(defn on-change
  [event]
  (let [target (.-target event)
        value (read-string (.-value target))]
    (if (leap? value)
      (set! (.-innerHTML result) "YES")
      (set! (.-innerHTML result) "NO"))))

(events/listen input "keyup" on-change)
```

Now, compile the clojurescript code with:

``` bash
$ ./scripts/watch
```

Behind the scenes, the `watch` script uses the `lein` build tool to
execute a command similar to the `java` build command from the previous
sections:

``` bash
rlwrap lein trampoline run -m clojure.main scripts/watch.clj
```

<div class="warning">

You must have `rlwrap` installed on your system.

</div>

Finally, open the `index.html` file in a browser. Typing a year in the
textbox should display an indication of its leap year status.

You may have noticed other files in the scripts directory, like `build`
and `release`. These are the same build scripts mentioned in the
previous section, but we will stick with `watch` here.

### Managing dependencies

The real purpose of using Leiningen for the ClojureScript compilation
process is to automate the retrieval of dependencies. This is
dramatically simpler than retrieving them manually.

The dependencies, among other parameters, are declared in the
`project.clj` file and have this form (from the **mies** template):

``` clojure
(defproject leapyears "0.1.0-SNAPSHOT"
  :description "FIXME: write this!"
  :url "http://example.com/FIXME"
  :dependencies [[org.clojure/clojure "1.8.0"]
                 [org.clojure/clojurescript "1.9.36"]
                 [org.clojure/data.json "0.2.6"]]
  :jvm-opts ^:replace ["-Xmx1g" "-server"]
  :node-dependencies [[source-map-support "0.3.2"]]
  :plugins [[lein-npm "0.5.0"]]
  :source-paths ["src" "target/classes"]
  :clean-targets ["out" "release"]
  :target-path "target")
```

And here is a brief explanation of the properties relevant for
ClojureScript:

  - `:dependencies`: a vector of dependencies that your project needs.

  - `:clean-targets`: a vector of paths that `lein clean` should delete.

The dependencies in ClojureScript are packaged using `jar` files. If you
are coming from Clojure or any JVM language, `jar` files will be very
familiar to you. But if you aren’t familiar with them, do not worry: a
.jar file is like a plain zip file that contains the `project.clj` for
the library, some metadata, and the ClojureScript sources. The packaging
will be explained in another section.

Clojure packages are often published on [Clojars](http://clojars.org).
You can also find many third party libraries on the [ClojureScript
Wiki](https://github.com/clojure/clojurescript/wiki#libraries).

## External dependencies

In some circumstances you may found yourself that you need some library
but that does not exists in *ClojureScript* but it is already
implemented in javascript and you want to use it on your project.

There are many ways that you can do it mainly depending on the library
that you want to include. Let see some ways.

### Closure Module compatible library

If you have a library that is just written to be compatible with google
closure module system and you want to include it on your project you
should just put it in the source (classpath) and access it like any
other clojure namespace.

This is the most simplest case, because google closure modules are
directly compatible and you can mix your clojure code with javascript
code written using google closure module system without any additional
steps.

Let play with it creating new project using **mies** template:

``` shell
lein new mies myextmods
cd myextmods
```

Create a simple google closure module for experiment:

**src/myextmods/myclosuremodule.js.**

``` javascript
goog.provide("myextmods.myclosuremodule");

goog.scope(function() {
  var module = myextmods.myclosuremodule;
  module.get_greetings = function() {
    return "Hello from google closure module.";
  };
});
```

Now, open the repl, require the namespace and try to use the exposed
function:

``` clojure
(require '[myextmods.myclosuremodule :as cm])
(cm/get_greetings)
;; => "Hello from google closure module."
```

<div class="note">

you can open the nodejs repl just executing `./scripts/repl` on the root
of the repository.

</div>

### CommonJS modules compatible libraries

Due to the Node.JS popularity the commonjs used in node is today the
most used module format for javascript libraries, independently if they
will be used in server side development using nodejs or using browser
side applications.

Let’s play with that. Start creating a simple file using commonjs module
format (pretty analogous to the previous example using google closure
modules):

**src/myextmods/mycommonjsmodule.js.**

``` js
function getGreetings() {
  return "Hello from commonjs module.";
}

exports.getGreetings = getGreetings;
```

Later, in order to use that simple pet library you should indicate to
the *ClojureScript* compiler the path to that file and the used module
type with `:foreign-libs` attribute.

Open `scripts/repl.clj` and modify it to somethig like this:

``` clojure
(require
  '[cljs.repl :as repl]
  '[cljs.repl.node :as node])

(repl/repl
 (node/repl-env)
 :language-in  :ecmascript5
 :language-out :ecmascript5
 :foreign-libs [{:file "myextmods/mycommonjsmodule.js"
                 :provides ["myextmods.mycommonjsmodule"]
                 :module-type :commonjs}]
 :output-dir "out"
 :cache-analysis false)
```

<div class="note">

Although the direct path is used to point to this pet library you can
specify a full URI to remote resource and it will be automatically
downloaded.

</div>

Now, let’s try to play with moment within the repl (executing the
`./scripts/repl` script that uses the previously modified
`./scripts/repl.clj` file):

``` clojure
(require '[myextmods.mycommonjsmodule :as cm])
(cm/getGreetings)
;; => "Hello from commonjs module."
```

### Legacy, module-less (global scope) libraries

Although today is very common have libraries packaged using some kind of
modules, there are also a great amount of libraries that just exposes a
global objects and does not uses any kind of modules; and you may want
to use them from *ClojureScript*.

In order to use a library that exposes a global object, you should
follow similar steps as with commojs modules with the exception that you
should omit the `:module-type` attribute.

This will create a *synthetic* namespace that you should require in
order to be able to access to the global object through the `js/`
namespace. The namespace is called *synthetic* because it does not
expose any object behind it, it just indicates to the compiler that you
want that dependency.

Let’s play with that. Start creating a simple file declaring just a
global function:

**src/myextmods/myglobalmodule.js.**

``` js
function getGreetings() {
  return "Hello from global scope.";
}
```

Open `scripts/repl.clj` and modify it to somethig like this:

``` clojure
(require
  '[cljs.repl :as repl]
  '[cljs.repl.node :as node])

(repl/repl
 (node/repl-env)
 :language-in  :ecmascript5
 :language-out :ecmascript5
 :foreign-libs [{:file "myextmods/mycommonjsmodule.js"
                 :provides ["myextmods.mycommonjsmodule"]
                 :module-type :commonjs}
                {:file "myextmods/myglobalmodule.js"
                 :provides ["myextmods.myglobalmodule"]}]
 :output-dir "out"
 :cache-analysis false)
```

And in the same way as in previous examples, let evaluate that in the
repl:

``` clojure
(require 'myextmods.myglobalmodule)
(js/getGreetings)
;; => "Hello from global scope."
```

## Unit testing

As you might expect, testing in *ClojureScript* consists of the same
concepts widely used by other language such as Clojure, Java, Python,
JavaScript, etc.

Regardless of the language, the main objective of unit testing is to run
some test cases, verifying that the code under test behaves as expected
and returns without raising unexpected exceptions.

The immutablity of *ClojureScript* data structures helps to make
programs less error prone, and facilitates testing a little bit. Another
advantage of *ClojureScript* is that it tends to use plain data instead
of complex objects. Building "mock" objects for testing is thus greatly
simplified.

### First steps

The "official" *ClojureScript* testing framework is in the "cljs.test"
namespace. It is a very simple library, but it should be more than
enough for our purposes.

There are other libraries that offer additional features or directly
different approaches to testing, such as
[test.check](https://github.com/clojure/test.check). However, we will
not cover them here.

Start creating a new project using the **mies** leiningen template for
experimenting with tests:

``` bash
$ lein new mies mytestingapp
$ cd mytestingapp
```

This project will contain the same layout as we have seen in the
**dependency management** subchapter, so we won’t explain it again.

The next step is a creating a directory tree for our tests:

``` bash
$ mkdir -p test/mytestingapp
$ touch test/mytestingapp/core_tests.cljs
```

Also, we should adapt the existing `watch.clj` script to work with this
newly created test directory:

``` clojure
(require '[cljs.build.api :as b])

(b/watch (b/inputs "test" "src")
  {:main 'mytestingapp.core_tests
   :target :nodejs
   :output-to "out/mytestingapp.js"
   :output-dir "out"
   :verbose true})
```

This new script will compile and watch both directories "src" and
"test", and it sets the new entry point to the `mytestingapp.core_tests`
namespace.

Next, put some test code in the `core_tests.cljs` file:

``` clojure
(ns mytestingapp.core-tests
  (:require [cljs.test :as t]))

(enable-console-print!)

(t/deftest my-first-test
  (t/is (= 1 2)))

(set! *main-cli-fn* #(t/run-tests))
```

The relevant part of that code snippet is:

``` clojure
(t/deftest my-first-test
  (t/is (= 1 2)))
```

The `deftest` macro is a basic primitive for defining our tests. It
takes a name as its first parameter, followed by one or multiple
assertions using the `is` macro. In this example, we try to assert that
`(= 1 2)` is true.

Let’s try to run this. First start the watch process:

``` bash
$ ./scripts/watch
Building ...
Copying jar:file:/home/niwi/.m2/repository/org/clojure/clojurescript/1.9.36/clojurescript-1.9.36.jar!/cljs/core.cljs to out/cljs/core.cljs
Reading analysis cache for jar:file:/home/niwi/.m2/repository/org/clojure/clojurescript/1.9.36/clojurescript-1.9.36.jar!/cljs/core.cljs
Compiling out/cljs/core.cljs
... done. Elapsed 3.862126827 seconds
Watching paths: /home/niwi/cljsbook/playground/mytestingapp/test, /home/niwi/cljsbook/playground/mytestingapp/src
```

When the compilation is finished, try to run the compiled file with
`nodejs`:

``` bash
$ node out/mytestingapp.js

Testing mytestingapp.core-tests

FAIL in (my-first-test) (cljs/test.js:374:14)
expected: (= 1 2)
  actual: (not (= 1 2))

Ran 1 tests containing 1 assertions.
1 failures, 0 errors.
```

You can see that the expected assert failure is successfully printed in
the console. To fix the test, just change the `=` with `not=` and run
the file again:

``` bash
$ node out/mytestingapp.js

Testing mytestingapp.core-tests

Ran 1 tests containing 1 assertions.
0 failures, 0 errors.
```

It is fine to test these kinds of assertions, but they are not very
useful. Let’s go to test some application code. For this, we will use a
function to check if a year is a leap year or not. Write the following
content to the `src/mytestingapp/core.clj` file:

``` clojure
(defn leap?
  [year]
  (and (zero? (js-mod year 4))
       (pos? (js-mod year 100))
       (pos? (js-mod year 400))))
```

Next, write a new test case to check that our new `leap?` function works
properly. Make the `core_tests.cljs` file look like:

``` clojure
(ns mytestingapp.core-tests
  (:require [cljs.test :as t]
            [mytestingapp.core :as core]))

(enable-console-print!)

(t/deftest my-first-test
  (t/is (not= 1 2)))

(t/deftest my-second-test
  (t/is (core/leap? 1980))
  (t/is (not (core/leap? 1981))))

(set! *main-cli-fn* #(t/run-tests))
```

Run the compiled file again to see that there are now two tests running.
The first test passes as before, and our two new leap year tests pass as
well.

### Async Testing

One of the peculiarities of *ClojureScript* is that it runs in an
asynchronous, single-threaded execution environment, which has its
challenges.

In an async execution environment, we should be able to test
asynchronous functions. To this end, the *ClojureScript* testing library
offers the `async` macro, allowing you to create tests that play well
with asynchronous code.

First, we need to write a function that works in an asynchronous way.
For this purpose, we will create the `async-leap?` predicate that will
do the same operation but asychronously return a result using a
callback:

``` clojure
(defn async-leap?
  [year callback]
  (js/setImmediate
   (fn []
     (let [result (or (zero? (js-mod year 400))
                      (and (pos? (js-mod year 100))
                           (zero? (js-mod year 4))))]
       (callback result)))))
```

The JavaScript function `setImmediate` is used to emulate an
asynchronous task, and the callback is executed with the result of that
predicate.

To test it, we should write a test case using the previously mentioned
`async` macro:

``` clojure
(t/deftest my-async-test
  (t/async done
    (core/async-leap? 1980 (fn [result]
                             (t/is (true? result))
                             (done)))))
```

The `done` function exposed by the `async` macro should be called after
the asynchronous operation is finished and all assertions have run.

It is very important to execute the `done` function only once. Omitting
it or executing it twice may cause strange behavior and should be
avoided.

### Fixtures

TBD

### Integrating with CI

Most continuous integration tools and services expect that test scripts
you provide return a standard exit code. But the ClojureScript test
framework cannot customize this exit code without some configuration,
because JavaScript lacks a universal exit code API for ClojureScript to
use.

To fix this, the *ClojureScript* test framework provides an avenue for
executing custom code after the tests are done. This is where you are
expected to set the environment-specific exit code depending on the
final test status: `0` for success, `1` for failure.

Insert this code at the end of `core_tests.cljs`:

``` clojure
(defmethod t/report [::t/default :end-run-tests]
  [m]
  (if (t/successful? m)
    (set! (.-exitCode js/process) 0)
    (set! (.-exitCode js/process) 1)))
```

Now, you may check the exit code of the test script after running:

``` bash
$ node out/mytestingapp.js
$ echo $?
```

This code snippet obviously assumes that you are running the tests using
**nodejs**. If you are running your script in another execution
environment, you should be aware of how you can set the exit code in
that environment and modify the previous snippet accordingly.

# Language (advanced topics)

This chapter intends to explain some advanced topics that are part of
the language and that does not fit in the first chapter. The good
candidates for this section are transducers, core protocols, transients,
metadata. In summary: topics that are not mandatory for understand the
language.

## Transducers

### Data transformation

ClojureScript offers a rich vocabulary for data transformation in terms
of the sequence abstraction, which makes such transformations highly
general and composable. Let’s see how we can combine several collection
processing functions to build new ones. We will be using a simple
problem throughout this section: splitting grape clusters, filtering out
the rotten ones, and cleaning them. We have a collection of grape
clusters like the following:

``` clojure
(def grape-clusters
  [{:grapes [{:rotten? false :clean? false}
             {:rotten? true :clean? false}]
    :color :green}
   {:grapes [{:rotten? true :clean? false}
             {:rotten? false :clean? false}]
    :color :black}])
```

We are interested in splitting the grape clusters into individual
grapes, discarding the rotten ones and cleaning the remaining grapes so
they are ready for eating. We are well-equipped in ClojureScript for
this data transformation task; we could implement it using the familiar
`map`, `filter` and `mapcat` functions:

``` clojure
(defn split-cluster
  [c]
  (:grapes c))

(defn not-rotten
  [g]
  (not (:rotten? g)))

(defn clean-grape
  [g]
  (assoc g :clean? true))

(->> grape-clusters
     (mapcat split-cluster)
     (filter not-rotten)
     (map clean-grape))
;; => ({rotten? false :clean? true} {:rotten? false :clean? true})
```

In the above example we succintly solved the problem of selecting and
cleaning the grapes, and we can even abstract such transformations by
combining the `mapcat`, `filter` and `map` operations using partial
application and function composition:

``` clojure
(def process-clusters
  (comp
    (partial map clean-grape)
    (partial filter not-rotten)
    (partial mapcat split-cluster)))

(process-clusters grape-clusters)
;; => ({rotten? false :clean? true} {:rotten? false :clean? true})
```

The code is very clean, but it has a few problems. For example, each
call to `map`, `filter` and `mapcat` consumes and produces a sequence
that, although lazy, generates intermediate results that will be
discarded. Each sequence is fed to the next step, which also returns a
sequence. Wouldn’t be great if we could do the transformation in a
single transversal of the `grape-cluster` collection?

Another problem is that even though our `process-clusters` function
works with any sequence, we can’t reuse it with anything that is not a
sequence. Imagine that instead of having the grape cluster collection
available in memory it is being pushed to us asynchronously in a stream.
In that situation we couldn’t reuse `process-clusters` since usually
`map`, `filter` and `mapcat` have concrete implementations depending on
the type.

### Generalizing to process transformations

The process of mapping, filtering or mapcatting isn’t necessarily tied
to a concrete type, but we keep reimplementing them for different types.
Let’s see how we can generalize such processes to be context
independent. We’ll start by implementing naive versions of `map` and
`filter` first to see how they work internally:

``` clojure
(defn my-map
  [f coll]
  (when-let [s (seq coll)]
    (cons (f (first s)) (my-map f (rest s)))))

(my-map inc [0 1 2])
;; => (1 2 3)

(defn my-filter
  [pred coll]
  (when-let [s (seq coll)]
    (let [f (first s)
          r (rest s)]
      (if (pred f)
        (cons f (my-filter pred r))
        (my-filter pred r)))))

(my-filter odd? [0 1 2])
;; => (1)
```

As we can see, they both assume that they receive a seqable and return a
sequence. Like many recursive functions they can be implemented in terms
of the already familiar `reduce` function. Note that functions that are
given to reduce receive an accumulator and an input and return the next
accumulator. We’ll call these types of functions reducing functions from
now on.

``` clojure
(defn my-mapr
  [f coll]
  (reduce (fn [acc input]         ;; reducing function
            (conj acc (f input)))
          []                      ;; initial value
          coll))                  ;; collection to reduce

(my-mapr inc [0 1 2])
;; => [1 2 3]

(defn my-filterr
  [pred coll]
  (reduce (fn [acc input]         ;; reducing function
            (if (pred input)
              (conj acc input)
              acc))
          []                      ;; initial value
          coll))                  ;; collection to reduce

(my-filterr odd? [0 1 2])
;; => [1]
```

We’ve made the previous versions more general since using `reduce` makes
our functions work on any thing that is reducible, not just sequences.
However you may have noticed that, even though `my-mapr` and
`my-filterr` don’t know anything about the source (`coll`) they are
still tied to the output they produce (a vector) both with the initial
value of the reduce (`[]`) and the hardcoded `conj` operation in the
body of the reducing function. We could have accumulated results in
another data structure, for example a lazy sequence, but we’d have to
rewrite the functions in order to do so.

How can we make these functions truly generic? They shouldn’t know about
either the source of inputs they are transforming nor the output that is
generated. Have you noticed that `conj` is just another reducing
function? It takes an accumulator and an input and returns another
accumulator. So, if we parameterise the reducing function that `my-mapr`
and `my-filterr` use, they won’t know anything about the type of the
result they are building. Let’s give it a shot:

``` clojure
(defn my-mapt
  [f]                         ;; function to map over inputs
  (fn [rfn]                   ;; parameterised reducing function
    (fn [acc input]           ;; transformed reducing function, now it maps `f`!
      (rfn acc (f input)))))

(def incer (my-mapt inc))

(reduce (incer conj) [] [0 1 2])
;; => [1 2 3]

(defn my-filtert
  [pred]                      ;; predicate to filter out inputs
  (fn [rfn]                   ;; parameterised reducing function
    (fn [acc input]           ;; transformed reducing function, now it discards values based on `pred`!
      (if (pred input)
        (rfn acc input)
        acc))))

(def only-odds (my-filtert odd?))

(reduce (only-odds conj) [] [0 1 2])
;; => [1]
```

That’s a lot of higher-order functions so let’s break it down for a
better understanding of what’s going on. We’ll examine how `my-mapt`
works step by step. The mechanics are similar for `my-filtert`, so we’ll
leave it out for now.

First of all, `my-mapt` takes a mapping function; in the example we are
giving it `inc` and getting another function back. Let’s replace `f`
with `inc` to see what we are building:

``` clojure
(def incer (my-mapt inc))
;; (fn [rfn]
;;   (fn [acc input]
;;     (rfn acc (inc input))))
;;               ^^^
```

The resulting function is still parameterised to receive a reducing
function to which it will delegate, let’s see what happens when we call
it with `conj`:

``` clojure
(incer conj)
;; (fn [acc input]
;;   (conj acc (inc input)))
;;    ^^^^
```

We get back a reducing function which uses `inc` to transform the inputs
and the `conj` reducing function to accumulate the results. In essence,
we have defined map as the transformation of a reducing function. The
functions that transform one reducing function into another are called
transducers in ClojureScript.

To ilustrate the generality of transducers, let’s use different sources
and destinations in our call to `reduce`:

``` clojure
(reduce (incer str) "" [0 1 2])
;; => "123"

(reduce (only-odds str) "" '(0 1 2))
;; => "1"
```

The transducer versions of `map` and `filter` transform a process that
carries inputs from a source to a destination but don’t know anything
about where the inputs come from and where they end up. In their
implementation they contain the *essence* of what they accomplish,
independent of context.

Now that we know more about transducers we can try to implement our own
version of `mapcat`. We already have a fundamental piece of it: the
`map` transducer. What `mapcat` does is map a function over an input and
flatten the resulting structure one level. Let’s try to implement the
catenation part as a transducer:

``` clojure
(defn my-cat
  [rfn]
  (fn [acc input]
    (reduce rfn acc input)))

(reduce (my-cat conj) [] [[0 1 2] [3 4 5]])
;; => [0 1 2 3 4 5]
```

The `my-cat` transducer returns a reducing function that catenates its
inputs into the accumulator. It does so reducing the `input` reducible
with the `rfn` reducing function and using the accumulator (`acc`) as
the initial value for such reduction. `mapcat` is simply the composition
of `map` and `cat`. The order in which transducers are composed may seem
backwards but it’ll become clear in a moment.

``` clojure
(defn my-mapcat
  [f]
  (comp (my-mapt f) my-cat))

(defn dupe
  [x]
  [x x])

(def duper (my-mapcat dupe))

(reduce (duper conj) [] [0 1 2])
;; => [0 0 1 1 2 2]
```

### Transducers in ClojureScript core

Some of the ClojureScript core functions like `map`, `filter` and
`mapcat` support an arity 1 version that returns a transducer. Let’s
revisit our definition of `process-cluster` and define it in terms of
transducers:

``` clojure
(def process-clusters
  (comp
    (mapcat split-cluster)
    (filter not-rotten)
    (map clean-grape)))
```

A few things changed since our previous definition `process-clusters`.
First of all, we are using the transducer-returning versions of
`mapcat`, `filter` and `map` instead of partially applying them for
working on sequences.

Also you may have noticed that the order in which they are composed is
reversed, they appear in the order they are executed. Note that all
`map`, `filter` and `mapcat` return a transducer. `filter` transforms
the reducing function returned by `map`, applying the filtering before
proceeding; `mapcat` transforms the reducing function returned by
`filter`, applying the mapping and catenation before proceeding.

One of the powerful properties of transducers is that they are combined
using regular function composition. What’s even more elegant is that the
composition of various transducers is itself a transducer\! This means
that our `process-cluster` is a transducer too, so we have defined a
composable and context-independent algorithmic transformation.

Many of the core ClojureScript functions accept a transducer, let’s look
at some examples with our newly created `process-cluster`:

``` clojure
(into [] process-clusters grape-clusters)
;; => [{:rotten? false, :clean? true} {:rotten? false, :clean? true}]

(sequence process-clusters grape-clusters)
;; => ({:rotten? false, :clean? true} {:rotten? false, :clean? true})

(reduce (process-clusters conj) [] grape-clusters)
;; => [{:rotten? false, :clean? true} {:rotten? false, :clean? true}]
```

Since using `reduce` with the reducing function returned from a
transducer is so common, there is a function for reducing with a
transformation called `transduce`. We can now rewrite the previous call
to `reduce` using `transduce`:

``` clojure
(transduce process-clusters conj [] grape-clusters)
;; => [{:rotten? false, :clean? true} {:rotten? false, :clean? true}]
```

### Initialisation

In the last example we provided an initial value to the `transduce`
function (`[]`) but we can omit it and get the same result:

``` clojure
(transduce process-clusters conj grape-clusters)
;; => [{:rotten? false, :clean? true} {:rotten? false, :clean? true}]
```

What’s going on here? How can `transduce` know what initial value use as
an accumulator when we haven’t specified it? Try calling `conj` without
any arguments and see what happens:

``` clojure
(conj)
;; => []
```

The `conj` function has a arity 0 version that returns an empty vector
but is not the only reducing function that supports arity 0. Let’s
explore some others:

``` clojure
(+)
;; => 0

(*)
;; => 1

(str)
;; => ""

(= identity (comp))
;; => true
```

The reducing function returned by transducers must support the arity 0
as well, which will typically delegate to the transformed reducing
function. There is no sensible implementation of the arity 0 for the
transducers we have implemented so far, so we’ll simply call the
reducing function without arguments. Here’s how our modified `my-mapt`
could look like:

``` clojure
(defn my-mapt
  [f]
  (fn [rfn]
    (fn
      ([] (rfn))                ;; arity 0 that delegates to the reducing fn
      ([acc input]
        (rfn acc (f input))))))
```

The call to the arity 0 of the reducing function returned by a
transducer will call the arity 0 version of every nested reducing
function, eventually calling the outermost reducing function. Let’s see
an example with our already defined `process-clusters` transducer:

``` clojure
((process-clusters conj))
;; => []
```

The call to the arity 0 flows through the transducer stack, eventually
calling `(conj)`.

### Stateful transducers

So far we’ve only seen purely functional transducer; they don’t have any
implicit state and are very predictable. However, there are many data
transformation functions that are inherently stateful, like `take`.
`take` receives a number `n` of elements to keep and a collection and
returns a collection with at most `n` elements.

``` clojure
(take 10 (range 100))
;; => (0 1 2 3 4 5 6 7 8 9)
```

Let’s step back for a bit and learn about the early termination of the
`reduce` function. We can wrap an accumulator in a type called `reduced`
for telling `reduce` that the reduction process should terminate
immediately. Let’s see an example of a reduction that aggregates the
inputs in a collection and finishes as soon as there are 10 elements in
the accumulator:

``` clojure
(reduce (fn [acc input]
          (if (= (count acc) 10)
            (reduced acc)
            (conj acc input)))
         []
         (range 100))
;; => [0 1 2 3 4 5 6 7 8 9]
```

Since transducers are modifications of reducing functions they also use
`reduced` for early termination. Note that stateful transducers may need
to do some cleanup before the process terminates, so they must support
an arity 1 as a "completion" step. Usually, like with arity 0, this
arity will simply delegate to the transformed reducing function’s arity
1.

Knowing this we are able to write stateful transducers like `take`.
We’ll be using mutable state internally for tracking the number of
inputs we have seen so far, and wrap the accumulator in a `reduced` as
soon as we’ve seen enough elements:

``` clojure
(defn my-take
  [n]
  (fn [rfn]
    (let [remaining (volatile! n)]
      (fn
        ([] (rfn))
        ([acc] (rfn acc))
        ([acc input]
          (let [rem @remaining
                nr (vswap! remaining dec)
                result (if (pos? rem)
                         (rfn acc input)   ;; we still have items to take
                         acc)]             ;; we're done, acc becomes the result
            (if (not (pos? nr))
              (ensure-reduced result)      ;; wrap result in reduced if not already
              result)))))))
```

This is a simplified version of the `take` function present in
ClojureScript core. There are a few things to note here so let’s break
it up in pieces to understand it better.

The first thing to notice is that we are creating a mutable value inside
the transducer. Note that we don’t create it until we receive a reducing
function to transform. If we created it before returning the transducer
we couldn’t use `my-take` more than once. Since the transducer is handed
a reducing function to transform each time it is used, we can use it
multiple times and the mutable variable will be created in every use.

``` clojure
(fn [rfn]
  (let [remaining (volatile! n)] ;; make sure to create mutable variables inside the transducer
    (fn
      ;; ...
)))

(def take-five (my-take 5))

(transduce take-five conj (range 100))
;; => [0 1 2 3 4]

(transduce take-five conj (range 100))
;; => [0 1 2 3 4]
```

Let’s now dig into the reducing function returned from `my-take`. First
of all we `deref` the volatile to get the number of elements that remain
to be taken and decrement it to get the next remaining value. If there
are still remaining items to take, we call `rfn` passing the accumulator
and input; if not, we already have the final result.

``` clojure
([acc input]
  (let [rem @remaining
        nr (vswap! remaining dec)
        result (if (pos? rem)
                 (rfn acc input)
                 acc)]
    ;; ...
))
```

The body of `my-take` should be obvious by now. We check whether there
are still items to be processed using the next remainder (`nr`) and, if
not, wrap the result in a `reduced` using the `ensure-reduced` function.
`ensure-reduced` will wrap the value in a `reduced` if it’s not reduced
already or simply return the value if it’s already reduced. In case we
are not done yet, we return the accumulated `result` for further
processing.

``` clojure
(if (not (pos? nr))
  (ensure-reduced result)
  result)
```

We’ve seen an example of a stateful transducer but it didn’t do anything
in its completion step. Let’s see an example of a transducer that uses
the completion step to flush an accumulated value. We’ll implement a
simplified version of `partition-all`, which given a `n` number of
elements converts the inputs in vectors of size `n`. For understanding
its purpose better let’s see what the arity 2 version gives us when
providing a number and a collection:

``` clojure
(partition-all 3 (range 10))
;; => ((0 1 2) (3 4 5) (6 7 8) (9))
```

The transducer returning function of `partition-all` will take a number
`n` and return a transducer that groups inputs in vectors of size `n`.
In the completion step it will check if there is an accumulated result
and, if so, add it to the result. Here’s a simplified version of
ClojureScript core `partition-all` function, where `array-list` is a
wrapper for a mutable JavaScript array:

``` clojure
(defn my-partition-all
  [n]
  (fn [rfn]
    (let [a (array-list)]
      (fn
        ([] (rfn))
        ([result]
          (let [result (if (.isEmpty a)                  ;; no inputs accumulated, don't have to modify result
                         result
                         (let [v (vec (.toArray a))]
                           (.clear a)                    ;; flush array contents for garbage collection
                           (unreduced (rfn result v))))] ;; pass to `rfn`, removing the reduced wrapper if present
            (rfn result)))
        ([acc input]
          (.add a input)
          (if (== n (.size a))                           ;; got enough results for a chunk
            (let [v (vec (.toArray a))]
              (.clear a)
              (rfn acc v))                               ;; the accumulated chunk becomes input to `rfn`
            acc))))))

(def triples (my-partition-all 3))

(transduce triples conj (range 10))
;; => [[0 1 2] [3 4 5] [6 7 8] [9]]
```

### Eductions

Eductions are a way to combine a collection and one or more
transformations that can be reduced and iterated over, and that apply
the transformations every time we do so. If we have a collection that we
want to process and a transformation over it that we want others to
extend, we can hand them a eduction, encapsulating the source collection
and our transformation. We can create an eduction with the `eduction`
function:

``` clojure
(def ed (eduction (filter odd?) (take 5) (range 100)))

(reduce + 0 ed)
;; => 25

(transduce (partition-all 2) conj ed)
;; => [[1 3] [5 7] [9]]
```

### More transducers in ClojureScript core

We learned about `map`, `filter`, `mapcat`, `take` and `partition-all`,
but there are a lot more transducers available in ClojureScript. Here is
an incomplete list of some other intersting ones:

  - `drop` is the dual of `take`, dropping up to `n` values before
    passing inputs to the reducing function

  - `distinct` only allows inputs to occur once

  - `dedupe` removes succesive duplicates in input values

We encourage you to explore ClojureScript core to see what other
transducers are out there.

### Defining our own transducers

There a few things to consider before writing our own transducers so in
this section we will learn how to properly implement one. First of all,
we’ve learned that the general structure of a transducer is the
following:

``` clojure
(fn [xf]
  (fn
    ([]          ;; init
      ...)
    ([r]         ;; completion
      ...)
    ([acc input] ;; step
      ...)))
```

Usually only the code represented with `…​` changes between transducers,
these are the invariants that must be preserved in each arity of the
resulting function:

  - arity 0 (init): must call the arity 0 of the nested transform `xf`

  - arity 1 (completion): used to produce a final value and potentially
    flush state, must call the arity 1 of the nested transform `xf`
    **exactly once**

  - arity 2 (step): the resulting reducing function which will call the
    arity 2 of the nested transform `xf` zero, one or more times

### Transducible processes

A transducible process is any process that is defined in terms of a
succession of steps ingesting input values. The source of input varies
from one process to another. Most of our examples dealt with inputs from
a collection or a lazy sequence, but it could be an asynchronous stream
of values or a `core.async` channel. The outputs produced by each step
are also different for every process; `into` creates a collection with
every output of the transducer, `sequence` yields a lazy sequence, and
asynchronous streams would probably push the outputs to their listeners.

In order to improve our understanding of transducible processes, we’re
going to implement an unbounded queue, since adding values to a queue
can be thought in terms of a succession of steps ingesting input. First
of all we’ll define a protocol and a data type that implements the
unbounded queue:

``` clojure
(defprotocol Queue
  (put! [q item] "put an item into the queue")
  (take! [q] "take an item from the queue")
  (shutdown! [q] "stop accepting puts in the queue"))

(deftype UnboundedQueue [^:mutable arr ^:mutable closed]
  Queue
  (put! [_ item]
    (assert (not closed))
    (assert (not (nil? item)))
    (.push arr item)
    item)
  (take! [_]
    (aget (.splice arr 0 1) 0))
  (shutdown! [_]
    (set! closed true)))
```

We defined the `Queue` protocol and as you may have noticed the
implementation of `UnboundedQueue` doesn’t know about transducers at
all. It has a `put!` operation as its step function and we’re going to
implement the transducible process on top of that interface:

``` clojure
(defn unbounded-queue
  ([]
   (unbounded-queue nil))
  ([xform]
   (let [put! (completing put!)
         xput! (if xform (xform put!) put!)
         q (UnboundedQueue. #js [] false)]
     (reify
       Queue
       (put! [_ item]
         (when-not (.-closed q)
           (let [val (xput! q item)]
             (if (reduced? val)
               (do
                 (xput! @val)  ;; call completion step
                 (shutdown! q) ;; respect reduced
                 @val)
               val))))
       (take! [_]
         (take! q))
       (shutdown! [_]
         (shutdown! q))))))
```

As you can see, the `unbounded-queue` constructor uses an
`UnboundedQueue` instance internally, proxying the `take!` and
`shutdown!` calls and implementing the transducible process logic in the
`put!` function. Let’s deconstruct it to understand what’s going on.

``` clojure
(let [put! (completing put!)
      xput! (if xform (xform put!) put!)
      q (UnboundedQueue. #js [] false)]
  ;; ...
)
```

First of all, we use `completing` for adding the arity 0 and arity 1 to
the `Queue` protocol’s `put!` function. This will make it play nicely
with transducers in case we give this reducing function to `xform` to
derive another. After that, if a transducer (`xform`) was provided, we
derive a reducing function applying the transducer to `put!`. If we’re
not handed a transducer we will just use `put!`. `q` is the internal
instance of `UnboundedQueue`.

``` clojure
(reify
  Queue
  (put! [_ item]
    (when-not (.-closed q)
      (let [val (xput! q item)]
        (if (reduced? val)
          (do
            (xput! @val)  ;; call completion step
            (shutdown! q) ;; respect reduced
            @val)
          val))))
  ;; ...
)
```

The exposed `put!` operation will only be performed if the queue hasn’t
been shut down. Notice that the `put!` implementation of
`UnboundedQueue` uses an assert to verify that we can still put values
to it and we don’t want to break that invariant. If the queue isn’t
closed we can put values into it, we use the possibly transformed
`xput!` for doing so.

If the put operation gives back a reduced value it’s telling us that we
should terminate the transducible process. In this case that means
shutting down the queue to not accept more values. If we didn’t get a
reduced value we can happily continue accepting puts.

Let’s see how our queue behaves without transducers:

``` clojure
(def q (unbounded-queue))
;; => #<[object Object]>

(put! q 1)
;; => 1
(put! q 2)
;; => 2

(take! q)
;; => 1
(take! q)
;; => 2
(take! q)
;; => nil
```

Pretty much what we expected, let’s now try with a stateless transducer:

``` clojure
(def incq (unbounded-queue (map inc)))
;; => #<[object Object]>

(put! incq 1)
;; => 2
(put! incq 2)
;; => 3

(take! incq)
;; => 2
(take! incq)
;; => 3
(take! incq)
;; => nil
```

To check that we’ve implemented the transducible process, let’s use a
stateful transducer. We’ll use a transducer that will accept values
while they aren’t equal to 4 and will partition inputs in chunks of 2
elements:

``` clojure
(def xq (unbounded-queue (comp
                           (take-while #(not= % 4))
                           (partition-all 2))))

(put! xq 1)
(put! xq 2)
;; => [1 2]
(put! xq 3)
(put! xq 4) ;; shouldn't accept more values from here on
(put! xq 5)
;; => nil

(take! xq)
;; => [1 2]
(take! xq) ;; seems like `partition-all` flushed correctly!
;; => [3]
(take! xq)
;; => nil
```

The example of the queue was heavily inspired by how `core.async`
channels use transducers in their internal step. We’ll discuss channels
and their usage with transducers in a later section.

Transducible processes must respect `reduced` as a way for signaling
early termination. For example, building a collection stops when
encountering a `reduced` and `core.async` channels with transducers are
closed. The `reduced` value must be unwrapped with `deref` and passed to
the completion step, which must be called exactly once.

Transducible processes shouldn’t expose the reducing function they
generate when calling the transducer with their own step function since
it may be stateful and unsafe to use from elsewhere.

## Transients

Although ClojureScript’s immutable and persistent data structures are
reasonably performant there are situations in which we are transforming
large data structures using multiple steps to only share the final
result. For example, the core `into` function takes a collection and
eagerly populates it with the contents of a sequence:

``` clojure
(into [] (range 100))
;; => [0 1 2 ... 98 99]
```

In the above example we are generating a vector of 100 elements
`conj`-ing one at a time. Every intermediate vector that is not the
final result won’t be seen by anybody except the `into` function and the
array copying required for persistence is an unnecesary overhead.

For these situations ClojureScript provides a special version of some of
its persistent data structures, which are called transients. Maps,
vectors and sets have a transient counterpart. Transients are always
derived from a persistent data structure using the `transient` function,
which creates a transient version in constant time:

``` clojure
(def tv (transient [1 2 3]))
;; => #<[object Object]>
```

Transients support the read API of their persistent counterparts:

``` clojure
(def tv (transient [1 2 3]))

(nth tv 0)
;; => 1

(get tv 2)
;; => 3

(def tm (transient {:language "ClojureScript"}))

(:language tm)
;; => "ClojureScript"

(def ts (transient #{:a :b :c}))

(contains? ts :a)
;; => true

(:a ts)
;; => :a
```

Since transients don’t have persistent and immutable semantics for
updates they can’t be transformed using the already familiar `conj` or
`assoc` functions. Instead, the transforming functions that work on
transients end with a bang. Let’s look at an example using `conj!` on a
transient:

``` clojure
(def tv (transient [1 2 3]))

(conj! tv 4)
;; => #<[object Object]>

(nth tv 3)
;; => 4
```

As you can see, the transient version of the vector is neither immutable
nor persistent. Instead, the vector is mutated in place. Although we
could transform `tv` repeatedly using `conj!` on it we shouldn’t abandon
the idioms used with the persistent data structures: when transforming a
transient, use the returned version of it for further modifications like
in the following example:

``` clojure
(-> [1 2 3]
  transient
  (conj! 4)
  (conj! 5))
;; => #<[object Object]>
```

We can convert a transient back to a persistent and immutable data
structure by calling `persistent!` on it. This operation, like deriving
a transient from a persistent data structure, is done in constant time.

``` clojure
(-> [1 2 3]
  transient
  (conj! 4)
  (conj! 5)
  persistent!)
;; => [1 2 3 4 5]
```

A peculiarity of transforming transients into persistent structures is
that the transient version is invalidated after being converted to a
persistent data structure and we can’t do further transformations to it.
This happens because the derived persistent data structure uses the
transient’s internal nodes and mutating them would break the
immutability and persistent guarantees:

``` clojure
(def tm (transient {}))
;; => #<[object Object]>

(assoc! tm :foo :bar)
;; => #<[object Object]>

(persistent! tm)
;; => {:foo :bar}

(assoc! tm :baz :frob)
;; Error: assoc! after persistent!
```

Going back to our initial example with `into`, here’s a very simplified
implementation of it that uses a transient for performance, returning a
persistent data structure and thus exposing a purely functional
interface although it uses mutation internally:

``` clojure
(defn my-into
  [to from]
  (persistent! (reduce conj! (transient to) from)))

(my-into [] (range 100))
;; => [0 1 2 ... 98 99]
```

## Metadata

ClojureScript symbols, vars and persistent collections support attaching
metadata to them. Metadata is a map with information about the entity
it’s attached to. The ClojureScript compiler uses metadata for several
purposes such as type hints, and the metadata system can be used by
tooling, library and application developers too.

There may not be many cases in day-to-day ClojureScript programming
where you need metadata, but it is a nice language feature to have and
know about; it may come in handy at some point. It makes things like
runtime code introspection and documentation generation very easy.
You’ll see why throughout this section.

### Vars

Let’s define a var and see what metadata is attached to it by default.
Note that this code is executed in a REPL, and thus the metadata of a
var defined in a source file may vary. We’ll use the `meta` function to
retrieve the metadata of the given value:

``` clojure
(def answer-to-everything 42)
;; => 42

#'answer-to-everything
;; => #'cljs.user/answer-to-everyhing

(meta #'answer-to-everything)
;; => {:ns cljs.user,
;;     :name answer-to-everything,
;;     :file "NO_SOURCE_FILE",
;;     :source "answer-to-everything",
;;     :column 6,
;;     :end-column 26,
;;     :line 1,
;;     :end-line 1,
;;     :arglists (),
;;     :doc nil,
;;     :test nil}
```

Few things to note here. First of all, `#'answer-to-everything` gives us
a reference to the `Var` that holds the value of the
`answer-to-everything` symbol. We see that it includes information about
the namespace (`:ns`) in which it was defined, its name, file (although,
since it was defined at a REPL doesn’t have a source file), source,
position in the file where it was defined, argument list (which only
makes sense for functions), documentation string and test function.

Let’s take a look at a function var’s metadata:

``` clojure
(defn add
  "A function that adds two numbers."
  [x y]
  (+ x y))

(meta #'add)
;; => {:ns cljs.user,
;;     :name add,
;;     :file "NO_SOURCE_FILE",
;;     :source "add",
;;     :column 7,
;;     :end-column 10,
;;     :line 1,
;;     :end-line 1,
;;     :arglists (quote ([x y])),
;;     :doc "A function that adds two numbers.",
;;     :test nil}
```

We see that the argument lists are stored in the `:arglists` field of
the var’s metadata and its documentation in the `:doc` field. We’ll now
define a test function to learn about what `:test` is used for:

``` clojure
(require '[cljs.test :as t])

(t/deftest i-pass
  (t/is true))

(meta #'i-pass)
;; => {:ns cljs.user,
;;     :name i-pass,
;;     :file "NO_SOURCE_FILE",
;;     :source "i-pass",
;;     :column 12,
;;     :end-column 18,
;;     :line 1,
;;     :end-line 1,
;;     :arglists (),
;;     :doc "A function that adds two numbers.",
;;     :test #<function (){ ... }>}
```

The `:test` attribute (truncated for brevity) in the `i-pass` var’s
metadata is a test function. This is used by the `cljs.test` library for
discovering and running tests in the namespaces you tell it to.

### Values

We learned that vars can have metadata and what kind of metadata is
added to them for consumption by the compiler and the `cljs.test`
testing library. Persistent collections can have metadata too, although
they don’t have any by default. We can use the `with-meta` function to
derive an object with the same value and type with the given metadata
attached. Let’s see how:

``` clojure
(def map-without-metadata {:language "ClojureScript"})
;; => {:language "ClojureScript"}

(meta map-without-metadata)
;; => nil

(def map-with-metadata (with-meta map-without-metadata
                                  {:answer-to-everything 42}))
;; => {:language "ClojureScript"}

(meta map-with-metadata)
;; => {:answer-to-everything 42}

(= map-with-metadata
   map-without-metadata)
;; => true

(identical? map-with-metadata
            map-without-metadata)
;; => false
```

It shouldn’t come as a surprise that metadata doesn’t affect equality
between two data structures since equality in ClojureScript is based on
value. Another interesting thing is that `with-meta` creates another
object of the same type and value as the given one and attaches the
given metadata to it.

Another open question is what happens with metadata when deriving new
values from a persistent data structure. Let’s find out:

``` clojure
(def derived-map (assoc map-with-metadata :language "Clojure"))
;; => {:language "Clojure"}

(meta derived-map)
;; => {:answer-to-everything 42}
```

As you can see in the example above, metadata is preserved in derived
versions of persistent data structures. There are some subtleties,
though. As long as the functions that derive new data structures return
collections with the same type, metadata will be preserved; this is not
true if the types change due to the transformation. To ilustrate this
point, let’s see what happens when we derive a seq or a subvector from a
vector:

``` clojure
(def v (with-meta [0 1 2 3] {:foo :bar}))
;; => [0 1 2 3]

(def sv (subvec v 0 2))
;; => [0 1]

(meta sv)
;; => nil

(meta (seq v))
;; => nil
```

### Syntax for metadata

The ClojureScript reader has syntactic support for metadata annotations,
which can be written in different ways. We can prepend var definitions
or collections with a caret (`^`) followed by a map for annotating it
with the given metadata map:

``` clojure
(def ^{:doc "The answer to Life, Universe and Everything."} answer-to-everything 42)
;; => 42

(meta #'answer-to-everything)
;; => {:ns cljs.user,
;;     :name answer-to-everything,
;;     :file "NO_SOURCE_FILE",
;;     :source "answer-to-everything",
;;     :column 6,
;;     :end-column 26,
;;     :line 1,
;;     :end-line 1,
;;     :arglists (),
;;     :doc "The answer to Life, Universe and Everything.",
;;     :test nil}

(def map-with-metadata ^{:answer-to-everything 42} {:language "ClojureScript"})
;; => {:language "ClojureScript"}

(meta map-with-metadata)
;; => {:answer-to-everything 42}
```

Notice how the metadata given in the `answer-to-everything` var
definition is merged with the var metadata.

A very common use of metadata is to set certain keys to a `true` value.
For example we may want to add to a var’s metadata that the variable is
dynamic or a constant. For such cases, we have a shorthand notation that
uses a caret followed by a keyword. Here are some examples:

``` clojure
(def ^:dynamic *foo* 42)
;; => 42

(:dynamic (meta #'*foo*))
;; => true

(def ^:foo ^:bar answer 42)
;; => 42

(select-keys (meta #'answer) [:foo :bar])
;; => {:foo true, :bar true}
```

There is another shorthand notation for attaching metadata. If we use a
caret followed by a symbol it will be added to the metadata map under
the `:tag` key. Using tags such as `^boolean` gives the ClojureScript
compiler hints about the type of expressions or function return types.

``` clojure
(defn ^boolean will-it-blend? [_] true)
;; => #<function ... >

(:tag (meta #'will-it-blend?))
;; => boolean

(not ^boolean (js/isNaN js/NaN))
;; => false
```

### Functions for working with metadata

We’ve learned about `meta` and `with-meta` so far but ClojureScript
offers a few functions for transforming metadata. There is `vary-meta`
which is similar to `with-meta` in that it derives a new object with the
same type and value as the original but it doesn’t take the metadata to
attach directly. Instead, it takes a function to apply to the metadata
of the given object to transform it for deriving new metadata. This is
how it works:

``` clojure
(def map-with-metadata ^{:foo 40} {:language "ClojureScript"})
;; => {:language "ClojureScript"}

(meta map-with-metadata)
;; => {:foo 40}

(def derived-map (vary-meta map-with-metadata update :foo + 2))
;; => {:language "ClojureScript"}

(meta derived-map)
;; => {:foo 42}
```

If instead we want to change the metadata of an existing var or value we
can use `alter-meta!` for changing it by applying a function or
`reset-meta!` for replacing it with another map:

``` clojure
(def map-with-metadata ^{:foo 40} {:language "ClojureScript"})
;; => {:language "ClojureScript"}

(meta map-with-metadata)
;; => {:foo 40}

(alter-meta! map-with-metadata update :foo + 2)
;; => {:foo 42}

(meta map-with-metadata)
;; => {:foo 42}

(reset-meta! map-with-metadata {:foo 40})
;; => {:foo 40}

(meta map-with-metadata)
;; => {:foo 40}
```

## Core protocols

One of the greatest qualities of the core ClojureScript functions is
that they are implemented around protocols. This makes them open to work
on any type that we extend with such protocols, be it defined by us or a
third party.

### Functions

As we’ve learned in previous chapters not only functions can be invoked.
Vectors are functions of their indexes, maps are functions of their keys
and sets are functions of their values.

We can extend types to be callable as functions implementing the `IFn`
protocol. A collection that doesn’t support calling it as a function is
the queue, let’s implement `IFn` for the `PersistentQueue` type so we’re
able to call queues as functions of their indexes:

``` clojure
(extend-type PersistentQueue
  IFn
  (-invoke
    ([this idx]
      (nth this idx))))

(def q #queue[:a :b :c])
;; => #queue [:a :b :c]

(q 0)
;; => :a

(q 1)
;; => :b

(q 2)
;; => :c
```

### Printing

For learning about some of the core protocols we’ll define a `Pair`
type, which will hold a pair of values.

``` clojure
(deftype Pair [fst snd])
```

If we want to customize how types are printed we can implement the
`IPrintWithWriter` protocol. It defines a function called `-pr-writer`
that receives the value to print, a writer object and options; this
function uses the writer object’s `-write` function to write the desired
`Pair` string representation:

``` clojure
(extend-type Pair
  IPrintWithWriter
  (-pr-writer [p writer _]
    (-write writer (str "#<Pair " (.-fst p) "," (.-snd p) ">"))))
```

### Sequences

In a [previous section](#the-sequence-abstraction) we learned about
sequences, one of ClojureScript’s main abstractions. Remember the
`first` and `rest` functions for working with sequences? They are
defined in the `ISeq` protocol, so we can extend types for responding to
such functions:

``` clojure
(extend-type Pair
  ISeq
  (-first [p]
    (.-fst p))

  (-rest [p]
    (list (.-snd p))))

(def p (Pair. 1 2))
;; => #<Pair 1,2>

(first p)
;; => 1

(rest p)
;; => (2)
```

Another handy function for working with sequences is `next`. Although
`next` works as long as the given argument is a sequence, we can
implement it explicitly with the `INext` protocol:

``` clojure
(def p (Pair. 1 2))

(next p)
;; => (2)

(extend-type Pair
  INext
  (-next [p]
    (println "Our next")
    (list (.-snd p))))

(next p)
;; Our next
;; => (2)
```

Finally, we can make our own types seqable implementing the `ISeqable`
protocol. This means we can pass them to `seq` for getting a sequence
back.

ISeqable

``` clojure
(def p (Pair. 1 2))

(extend-type Pair
  ISeqable
  (-seq [p]
    (list (.-fst p) (.-snd p))))

(seq p)
;; => (1 2)
```

Now our `Pair` type works with the plethora of ClojureScript functions
for working with sequences:

``` clojure
(def p (Pair. 1 2))
;; => #<Pair 1,2>

(map inc p)
;; => (2 3)

(filter odd? p)
;; => (1)

(reduce + p)
;; => 3
```

### Collections

Collection functions are also defined in terms of protocols. For this
section examples we will make the native JavaScript string act like a
collection.

The most important function for working with collection is `conj`,
defined in the `ICollection` protocol. Strings are the only type which
makes sense to `conj` to a string, so the `conj` operation for strings
will be simply a concatenation:

``` clojure
(extend-type string
  ICollection
  (-conj [this o]
    (str this o)))

(conj "foo" "bar")
;; => "foobar"

(conj "foo" "bar" "baz")
;; => "foobarbaz"
```

Another handy function for working with collections is `empty`, which is
part of the `IEmptyableCollection` protocol. Let’s implement it for the
string type:

``` clojure
(extend-type string
  IEmptyableCollection
  (-empty [_]
    ""))

(empty "foo")
;; => ""
```

We used the `string` special symbol for extending the native JavaScript
string. If you want to learn more about it check the [section about
extending JavaScript types](#extending-javascript-types).

#### Collection traits

There are some qualities that not all collections have, such as being
countable in constant time or being reversible. These traits are
splitted into different protocols since not all of them make sense for
every collection. For illustrating these protocols we’ll use the `Pair`
type we defined earlier.

For collections that can be counted in constant time using the `count`
function we can implement the `ICounted` protocol. It should be easy to
implement it for the `Pair` type:

``` clojure
(extend-type Pair
  ICounted
  (-count [_]
    2))

(def p (Pair. 1 2))

(count p)
;; => 2
```

Some collection types such as vectors and lists can be indexed by a
number using the `nth` function. If our types are indexed we can
implement the `IIndexed` protocol:

``` clojure
(extend-type Pair
  IIndexed
  (-nth
    ([p idx]
      (case idx
        0 (.-fst p)
        1 (.-snd p)
        (throw (js/Error. "Index out of bounds"))))
    ([p idx default]
      (case idx
        0 (.-fst p)
        1 (.-snd p)
        default))))

(nth p 0)
;; => 1

(nth p 1)
;; => 2

(nth p 2)
;; Error: Index out of bounds

(nth p 2 :default)
;; => :default
```

### Associative

There are many data structures that are associative: they map keys to
values. We’ve encountered a few of them already and we know many
functions for working with them like `get`, `assoc` or `dissoc`. Let’s
explore the protocols that these functions build upon.

First of all, we need a way to look up keys on an associative data
structure. The `ILookup` protocol defines a function for doing so, let’s
add the ability to look up keys in our `Pair` type since it is an
associative data structure that maps the indices 0 and 1 to values.

``` clojure
(extend-type Pair
  ILookup
  (-lookup
    ([p k]
      (-lookup p k nil))
    ([p k default]
      (case k
        0 (.-fst p)
        1 (.-snd p)
        default))))

(get p 0)
;; => 1

(get p 1)
;; => 2

(get p :foo)
;; => nil

(get p 2 :default)
;; => :default
```

For using `assoc` on a data structure it must implement the
`IAssociative` protocol. For our `Pair` type only 0 and 1 will be
allowed as keys for associating values. `IAssociative` also has a
function for asking whether a key is present or not.

``` clojure
(extend-type Pair
  IAssociative
  (-contains-key? [_ k]
    (contains? #{0 1} k))

  (-assoc [p k v]
    (case k
      0 (Pair. v (.-snd p))
      1 (Pair. (.-fst p) v)
      (throw (js/Error. "Can only assoc to 0 and 1 keys")))))

(def p (Pair. 1 2))
;; => #<Pair 1,2>

(assoc p 0 2)
;; => #<Pair 2,2>

(assoc p 1 1)
;; => #<Pair 1,1>

(assoc p 0 0 1 1)
;; => #<Pair 0,1>

(assoc p 2 3)
;; Error: Can only assoc to 0 and 1 keys
```

The complementary function for `assoc` is `dissoc` and it’s part of the
`IMap` protocol. It doesn’t make much sense for our `Pair` type but
we’ll implement it nonetheless. Dissociating 0 or 1 will mean putting
a `nil` in such position and invalid keys will be ignored.

``` clojure
(extend-type Pair
  IMap
  (-dissoc [p k]
    (case k
      0 (Pair. nil (.-snd p))
      1 (Pair. (.-fst p) nil)
      p)))

(def p (Pair. 1 2))
;; => #<Pair 1,2>

(dissoc p 0)
;; => #<Pair ,2>

(dissoc p 1)
;; => #<Pair 1,>

(dissoc p 2)
;; => #<Pair 1,2>

(dissoc p 0 1)
;; => #<Pair ,>
```

Associative data structures are made of key and value pairs we can call
entries. The `key` and `val` functions allow us to query the key and
value of such entries and they are built upon the `IMapEntry` protocol.
Let’s see a few examples of `key` and `val` and how map entries can be
used to build up maps:

``` clojure
(key [:foo :bar])
;; => :foo

(val [:foo :bar])
;; => :bar

(into {} [[:foo :bar] [:baz :xyz]])
;; => {:foo :bar, :baz :xyz}
```

Pairs can be map entries too, we treat their first element as the key
and the second as the value:

``` clojure
(extend-type Pair
  IMapEntry
  (-key [p]
    (.-fst p))

  (-val [p]
    (.-snd p)))

(def p (Pair. 1 2))
;; => #<Pair 1,2>

(key p)
;; => 1

(val p)
;; => 2

(into {} [p])
;; => {1 2}
```

### Comparison

For checking whether two or more values are equivalent with `=` we must
implement the `IEquiv` protocol. Let’s do it for our `Pair` type:

``` clojure
(def p  (Pair. 1 2))
(def p' (Pair. 1 2))
(def p'' (Pair. 1 2))

(= p p')
;; => false

(= p p' p'')
;; => false

(extend-type Pair
  IEquiv
  (-equiv [p other]
    (and (instance? Pair other)
         (= (.-fst p) (.-fst other))
         (= (.-snd p) (.-snd other)))))

(= p p')
;; => true

(= p p' p'')
;; => true
```

We can also make types comparable. The function `compare` takes two
values and returns a negative number if the first is less than the
second, 0 if both are equal and 1 if the first is greater than the
second. For making our types comparable we must implement the
`IComparable` protocol.

For pairs, comparison will mean checking if the two first values are
equal. If they are, the result will be the comparison of the second
values. If not, we will return the result of the first comparison:

``` clojure
(extend-type Pair
  IComparable
  (-compare [p other]
    (let [fc (compare (.-fst p) (.-fst other))]
      (if (zero? fc)
        (compare (.-snd p) (.-snd other))
        fc))))

(compare (Pair. 0 1) (Pair. 0 1))
;; => 0

(compare (Pair. 0 1) (Pair. 0 2))
;; => -1

(compare (Pair. 1 1) (Pair. 0 2))
;; => 1

(sort [(Pair. 1 1) (Pair. 0 2) (Pair. 0 1)])
;; => (#<Pair 0,1> #<Pair 0,2> #<Pair 1,1>)
```

### Metadata

The `meta` and `with-meta` functions are also based upon two protocols:
`IMeta` and `IWithMeta` respectively. We can make our own types capable
of carrying metadata adding an extra field for holding the metadata and
implementing both protocols.

Let’s implement a version of `Pair` that can have metadata:

``` clojure
(deftype Pair [fst snd meta]
  IMeta
  (-meta [p] meta)

  IWithMeta
  (-with-meta [p new-meta]
    (Pair. fst snd new-meta)))


(def p (Pair. 1 2 {:foo :bar}))
;; => #<Pair 1,2>

(meta p)
;; => {:foo :bar}

(def p' (with-meta p {:bar :baz}))
;; => #<Pair 1,2>

(meta p')
;; => {:bar :baz}
```

### Interoperability with JavaScript

Since ClojureScript is hosted in a JavaScript VM we often need to
convert ClojureScript data structures to JavaScript ones and viceversa.
We also may want to make native JS types participate in an abstraction
represented by a protocol.

#### Extending JavaScript types

When extending JavaScript objects instead of using JS globals like
`js/String`, `js/Date` and such, special symbols are used. This is done
for avoiding mutating global JS objects.

The symbols for extending JS types are: `object`, `array`, `number`,
`string`, `function`, `boolean` and `nil` is used for the null object.
The dispatch of the protocol to native objects uses Google Closure’s
[goog.typeOf](https://google.github.io/closure-library/api/namespace_goog.html#typeOf)
function. There’s a special `default` symbol that can be used for making
a default implementation of a protocol for every type.

For illustrating the extension of JS types we are going to define a
`MaybeMutable` protocol that’ll have a `mutable?` predicate as its only
function. Since in JavaScript mutability is the default we’ll extend the
default JS type returning true from `mutable?`:

``` clojure
(defprotocol MaybeMutable
  (mutable? [this] "Returns true if the value is mutable."))

(extend-type default
  MaybeMutable
  (mutable? [_] true))

;; object
(mutable? #js {})
;; => true

;; array
(mutable? #js [])
;; => true

;; string
(mutable? "")
;; => true

;; function
(mutable? (fn [x] x))
;; => true
```

Since fortunately not all JS object’s values are mutable we can refine
the implementation of `MaybeMutable` for returning `false` for strings
and functions.

``` clojure
(extend-protocol MaybeMutable
  string
  (mutable? [_] false)

  function
  (mutable? [_] false))


;; object
(mutable? #js {})
;; => true

;; array
(mutable? #js [])
;; => true

;; string
(mutable? "")
;; => false

;; function
(mutable? (fn [x] x))
;; => false
```

There is no special symbol for JavaScript dates so we have to extend
`js/Date` directly. The same applies to the rest of the types found in
the global `js` namespace.

#### Converting data

For converting values from ClojureScript types to JavaScript ones and
viceversa we use the `clj→js` and `js→clj` functions, which are based in
the `IEncodeJS` and `IEncodeClojure` protocols respectively.

For the examples we’ll use the Set type introduced in ES6. Note that is
not available in every JS runtime.

##### From ClojureScript to JS

First of all we’ll extend ClojureScript’s set type for being able to
convert it to JS. By default sets are converted to JavaScript arrays:

``` clojure
(clj->js #{1 2 3})
;; => #js [1 3 2]
```

Let’s fix it, `clj→js` is supposed to convert values recursively so
we’ll make sure to convert all the set contents to JS and creating the
set with the converted values:

``` clojure
(extend-type PersistentHashSet
  IEncodeJS
  (-clj->js [s]
    (js/Set. (into-array (map clj->js s)))))

(def s (clj->js #{1 2 3}))
(es6-iterator-seq (.values s))
;; => (1 3 2)

(instance? js/Set s)
;; => true

(.has s 1)
;; => true
(.has s 2)
;; => true
(.has s 3)
;; => true
(.has s 4)
;; => false
```

The `es6-iterator-seq` is an experimental function in ClojureScript core
for obtaining a seq from an ES6 iterable.

##### From JS to ClojureScript

Now it’s time to extend the JS set to convert to ClojureScript. As with
`clj→js`, `js→clj` recursively converts the value of the data structure:

``` clojure
(extend-type js/Set
  IEncodeClojure
  (-js->clj [s options]
    (into #{} (map js->clj (es6-iterator-seq (.values s))))))

(= #{1 2 3}
   (js->clj (clj->js #{1 2 3})))
;; => true

(= #{[1 2 3] [4 5] [6]}
   (js->clj (clj->js #{[1 2 3] [4 5] [6]})))
;; => true
```

Note that there is no one-to-one mapping between ClojureScript and
JavaScript values. For example, ClojureScript keywords are converted to
JavaScript strings when passed to `clj→js`.

### Reductions

The `reduce` function is based on the `IReduce` protocol, which enables
us to make our own or third-party types reducible. Apart from using them
with `reduce` they will automatically work with `transduce` too, which
will allow us to make a reduction with a transducer.

The JS array is already reducible in ClojureScript:

``` clojure
(reduce + #js [1 2 3])
;; => 6

(transduce (map inc) conj [] [1 2 3])
;; => [2 3 4]
```

However, the new ES6 Set type isn’t so let’s implement the `IReduce`
protocol. We’ll get an iterator using the Set’s `values` method and
convert it to a seq with the `es6-iterator-seq` function; after that
we’ll delegate to the original `reduce` function to reduce the
obtained sequence.

``` clojure
(extend-type js/Set
  IReduce
  (-reduce
   ([s f]
     (let [it (.values s)]
       (reduce f (es6-iterator-seq it))))
   ([s f init]
     (let [it (.values s)]
       (reduce f init (es6-iterator-seq it))))))

(reduce + (js/Set. #js [1 2 3]))
;; => 6

(transduce (map inc) conj [] (js/Set. #js [1 2 3]))
;; => [2 3 4]
```

Associative data structures can be reduced with the `reduce-kv`
function, which is based in the `IKVReduce` protocol. The main
difference between `reduce` and `reduce-kv` is that the latter uses a
three-argument function as a reducer, receiving the accumulator, key and
value for each item.

Let’s look at an example, we will reduce a map to a vector of pairs.
Note that, since vectors associate indexes to values, they can also be
reduced with `reduce-kv`.

``` clojure
(reduce-kv (fn [acc k v]
             (conj acc [k v]))
           []
           {:foo :bar
            :baz :xyz})
;; => [[:foo :bar] [:baz :xyz]]
```

We’ll extend the new ES6 map type to support `reduce-kv`, we’ll do this
by getting a sequence of key-value pairs and calling the reducing
function with the accumulator, key and value as positional arguments:

``` clojure
(extend-type js/Map
  IKVReduce
  (-kv-reduce [m f init]
   (let [it (.entries m)]
     (reduce (fn [acc [k v]]
               (f acc k v))
             init
             (es6-iterator-seq it)))))

(def m (js/Map.))
(.set m "foo" "bar")
(.set m "baz" "xyz")

(reduce-kv (fn [acc k v]
             (conj acc [k v]))
           []
           m)
;; => [["foo" "bar"] ["baz" "xyz"]]
```

In both examples we ended up delegating to the `reduce` function, which
is aware of reduced values and terminates when encountering one. Take
into account that if you don’t implement these protocols in terms of
`reduce` you will have to check for reduced values for early
termination.

### Asynchrony

There are some types that have the notion of asynchronous computation,
the value they represent may not be realized yet. We can ask whether a
value is realized using the `realized?` predicate.

Let’s ilustrate it with the `Delay` type, which takes a computation and
executes it when the result is needed. When we dereference a delay the
computation is run and the delay is realized:

``` clojure
(defn computation []
  (println "running!")
  42)

(def d (Delay. computation nil))

(realized? d)
;; => false

(deref d)
;; running!
;; => 42

(realized? d)
;; => true

@d
;; => 42
```

Both `realized?` and `deref` sit atop two protocols: `IPending` and
`IDeref`.

ES6 introduced a type that captures the notion of an asynchronous
computation that might fail: the Promise. A Promise represents an
eventual value and can be in one of three states:

  - pending: there is still no value available for this computation.

  - rejected: an error occurred and the promise contains a value that
    indicates the error.

  - resolved: the computation succesfully executed and the promise
    contains a value with the result.

Since the ES6 defined interface for promises doesn’t support querying
its state we’ll use Bluebird library’s Promise type for the examples.
You can use Bluebird’s promise type with the
[Promesa](https://github.com/funcool/promesa) library.

First of all we’ll add the ability to check if a promise is realized
(either resolved or rejected) using the `realized?` predicate. We just
have to implement the `IPending` protocol:

``` clojure
(require '[promesa.core :as p])

(extend-type js/Promise
  IPending
  (-realized? [p]
    (not (.isPending p))))


(p/promise (fn [resolve reject]))
;; => #<Promise {:status :pending}>

(realized? (p/promise (fn [resolve reject])))
;; => false

(p/resolved 42)
;; => #<Promise {:status :resolved, :value 42}>

(realized? (p/resolved 42))
;; => true

(p/rejected (js/Error. "OH NO"))
;; => #<Promise {:status :rejected, :error #object[Error Error: OH NO]}>

(realized? (p/rejected (js/Error. "OH NO")))
;; => true
```

Now we’ll extend promises to be derefable. When a promise that is still
pending is dereferenced we will return a special keyword:
`:promise/pending`. If it’s not we’ll just return the value it contains,
be it an error or a result:

``` clojure
(require '[promesa.core :as pro])

(extend-type js/Promise
  IDeref
  (-deref [p]
    (cond
      (.isPending p)
      :promise/pending

      (.isRejected p)
      (.reason p)

      :else
      (.value p))))

@(p/promise (fn [resolve reject]))
;; => :promise/pending

@(p/resolved 42)
;; => 42

@(p/rejected (js/Error. "OH NO"))
;; => #object[Error Error: OH NO]
```

### State

The ClojureScript state constructs such as the Atom and the Volatile
have different characteristics and semantics, and the operations on them
like `add-watch`, `reset!` or `swap!` are backed by protocols.

#### Atom

For ilustrating such protocols we will implement our own simplified
version of an `Atom`. It won’t support validators nor metadata, but we
will be able to:

  - `deref` the atom for getting its current value

  - `reset!` the value contained in the atom

  - `swap!` the atom with a function for transforming its state

`deref` is based on the `IDeref` protocol. `reset!` is based on the
`IReset` protocol and `swap!` on `ISwap`. We’ll start by defining a data
type and a constructor for our atom implementation:

``` clojure
(deftype MyAtom [^:mutable state ^:mutable watches]
  IPrintWithWriter
  (-pr-writer [p writer _]
    (-write writer (str "#<MyAtom " (pr-str state) ">"))))

(defn my-atom
  ([]
    (my-atom nil))
  ([init]
    (MyAtom. init {})))

(my-atom)
;; => #<MyAtom nil>

(my-atom 42)
;; => #<MyAtom 42>
```

Note that we’ve marked both the current state of the atom (`state`) and
the map of watchers (`watches`) with the `{:mutable true}` metadata.
We’ll be modifying them and we’re making this explicit with the
annotations.

Our `MyAtom` type is not very useful yet, we’ll start by implementing
the `IDeref` protocol so we can dereference its current value:

``` clojure
(extend-type MyAtom
  IDeref
  (-deref [a]
    (.-state a)))

(def a (my-atom 42))

@a
;; => 42
```

Now that we can dereference it we’ll implement the `IWatchable`
protocol, which will let us add and remove watches to our custom atom.
We’ll store the watches in the `watches` map of `MyAtom`, associating
keys to callbacks.

``` clojure
(extend-type MyAtom
  IWatchable
  (-add-watch [a key f]
    (let [ws (.-watches a)]
      (set! (.-watches a) (assoc ws key f))))

  (-remove-watch [a key]
    (let [ws (.-watches a)]
      (set! (.-watches a) (dissoc ws key))))

  (-notify-watches [a oldval newval]
    (doseq [[key f] (.-watches a)]
      (f key a oldval newval))))
```

We can now add watches to our atom but is not very useful since we still
can’t change it. For incorporating change we have to implement the
`IReset` protocol and make sure we notify the watches every time we
reset the atom’s value.

``` clojure
(extend-type MyAtom
  IReset
  (-reset! [a newval]
    (let [oldval (.-state a)]
      (set! (.-state a) newval)
      (-notify-watches a oldval newval)
      newval)))
```

Now let’s check that we got it right. We’ll add a watch, change the
atom’s value making sure the watch gets called and then remove it:

``` clojure
(def a (my-atom 41))
;; => #<MyAtom 41>

(add-watch a :log (fn [key a oldval newval]
                    (println {:key key
                              :old oldval
                              :new newval})))
;; => #<MyAtom 41>

(reset! a 42)
;; {:key :log, :old 41, :new 42}
;; => 42

(remove-watch a :log)
;; => #<MyAtom 42>

(reset! a 43)
;; => 43
```

Our atom is still missing the swapping functionality so we’ll add that
now, let’s implement the `ISwap` protocol. There are four arities for
the `-swap!` method of the protocol since the function passed to `swap!`
may take one, two, three or more arguments:

``` clojure
(extend-type MyAtom
  ISwap
  (-swap!
   ([a f]
    (let [oldval (.-state a)
          newval (f oldval)]
      (reset! a newval)))

   ([a f x]
     (let [oldval (.-state a)
           newval (f oldval x)]
       (reset! a newval)))

   ([a f x y]
     (let [oldval (.-state a)
           newval (f oldval x y)]
       (reset! a newval)))

   ([a f x y more]
     (let [oldval (.-state a)
           newval (apply f oldval x y more)]
       (reset! a newval)))))
```

We now have a custom implementation of the atom abstraction, let’s test
it in the REPL and see if it behaves like we expect:

``` clojure
(def a (my-atom 0))
;; => #<MyAtom 0>

(add-watch a :log (fn [key a oldval newval]
                    (println {:key key
                              :old oldval
                              :new newval})))
;; => #<MyAtom 0>

(swap! a inc)
;; {:key :log, :old 0, :new 1}
;; => 1

(swap! a + 2)
;; {:key :log, :old 1, :new 3}
;; => 3

(swap! a - 2)
;; {:key :log, :old 3, :new 1}
;; => 1

(swap! a + 2 3)
;; {:key :log, :old 1, :new 6}
;; => 6


(swap! a + 4 5 6)
;; {:key :log, :old 6, :new 21}
;; => 21

(swap! a * 2)
;; {:key :log, :old 21, :new 42}
;; => 42

(remove-watch a :log)
;; => #<MyAtom 42>
```

We did it\! We implemented a version of ClojureScript Atom without
support for metadata nor validators, extending it to support such
features is left as an exercise for the reader. Note that you’ll need to
modify the `MyAtom` type for being able to store metadata and a
validator.

#### Volatile

Volatiles are simpler than atoms in that they don’t support watching for
changes. All changes override the previous value much like the mutable
variables present in almost every programming language. Volatiles are
based on the `IVolatile` protocol that only defines a method for
`vreset!`, since `vswap!` is implemented as a macro.

Let’s start by creating our own volatile type and constructor:

``` clojure
(deftype MyVolatile [^:mutable state]
  IPrintWithWriter
  (-pr-writer [p writer _]
    (-write writer (str "#<MyVolatile " (pr-str state) ">"))))

(defn my-volatile
  ([]
    (my-volatile nil))
  ([v]
    (MyVolatile. v)))

(my-volatile)
;; => #<MyVolatile nil>

(my-volatile 42)
;; => #<MyVolatile 42>
```

Our `MyVolatile` still needs to support dereferencing and reseting it,
let’s implement `IDeref` and `IVolatile`, which will enable use to use
`deref`, `vreset!` and `vswap!` in our custom volatile:

``` clojure
(extend-type MyVolatile
  IDeref
  (-deref [v]
    (.-state v))

  IVolatile
  (-vreset! [v newval]
    (set! (.-state v) newval)
    newval))

(def v (my-volatile 0))
;; => #<MyVolatile 42>

(vreset! v 1)
;; => 1

@v
;; => 1

(vswap! v + 2 3)
;; => 6

@v
;; => 6
```

### Mutation

In the [section about transients](#transients) we learned about the
mutable counterparts of the immutable and persistent data structures
that ClojureScript provides. These data structures are mutable, and the
operations on them end with a bang (`!`) to make that explicit. As you
may have guessed every operation on transients is based on protocols.

#### From persistent to transient and viceversa

We’ve learned that we can transform a persistent data structure with the
`transient` function, which is based on the `IEditableCollection`
protocol; for transforming a transient data structure to a persistent
one we use `persistent!`, based on `ITransientCollection`.

Implementing immutable and persistent data structures and their
transient counterparts is out of the scope of this book but we recommend
taking a look at ClojureScript’s data structure implementation if you
are curious.

#### Case study: the hodgepodge library

[Hodgepodge](https://github.com/funcool/hodgepodge) is a ClojureScript
library for treating the browser’s local and session storage as if it
were a transient data structure. It allows you to insert, read and
delete ClojureScript data structures without worrying about encoding and
decoding them.

Browser’s storage is a simple key-value store that only supports
strings. Since all of ClojureScript data structures can be dumped into a
string and reified from a string using [the reader](#the-reader) we can
store arbitrary ClojureScript data in storage. We can also extend the
reader for being able to read custom data types so we’re able to put our
types in storage and `hodgepodge` will handle the encoding and decoding
for us.

We’ll start by wrapping the low-level storage methods with functions.
The following operations are supported by the storage:

  - getting the value corresponding to a key

  - setting a key to a certain value

  - removing a value given its key

  - counting the number of entries

  - clearing the storage

Let’s wrap them in a more idiomatic API for ClojureScript:

``` clojure
(defn contains-key?
  [^js/Storage storage ^string key]
  (let [ks (.keys js/Object storage)
        idx (.indexOf ks key)]
    (>= idx 0)))

(defn get-item
  ([^js/Storage storage ^string key]
     (get-item storage key nil))
  ([^js/Storage storage ^string key ^string default]
     (if (contains-key? storage key)
       (.getItem storage key)
       default)))

(defn set-item
  [^js/Storage storage ^string key ^string val]
  (.setItem storage key val)
  storage)

(defn remove-item
  [^js/Storage storage ^string key]
  (.removeItem storage key)
  storage)

(defn length
  [^js/Storage storage]
  (.-length storage))

(defn clear!
  [^js/Storage storage]
  (.clear storage))
```

Nothing too interesting going on there, we just wrapped the storage
methods in a nicer API. Now we will define a couple functions for
serializing and deserializing ClojureScript data structures to strings:

``` clojure
(require '[cljs.reader :as reader])

(defn serialize [v]
  (binding [*print-dup* true
            *print-readably* true]
    (pr-str v)))

(def deserialize
  (memoize reader/read-string))
```

The `serialize` function is used for converting a ClojureScript data
structure into a string using the `pr-str` function, configuring a
couple dynamic variables for obtaining the desired behaviour:

  - `print-dup` is set to `true` for a printed object to preserve its
    type when read later

  - `print-readably` is set to `true` for not converting
    non-alphanumeric characters to their escape sequences

The `deserialize` function simply invokes the reader’s function for
reading a string into a ClojureScript data structure: `read-string`.
It’s memoized for not having to call the reader each time we
deserialize a string since we can assume that a repeated string always
corresponds to the same data structure.

Now we can start extending the browser’s `Storage` type for acting like
a transient data structure. Take into account that the `Storage` type is
only available in browsers. We’ll start by implementing the `ICounted`
protocol for counting the items in the storage, we’ll simply delegate to
our previously defined `length` function:

``` clojure
(extend-type js/Storage
  ICounted
  (-count [^js/Storage s]
   (length s)))
```

We want to be able to use `assoc!` and `dissoc!` for inserting and
deleting key-value pairs in the storage, as well as the ability to read
from it. We’ll implement the `ITransientAssociative` protocol for
`assoc!`, `ITransientMap` for `dissoc!` and `ILookup` for reading
storage keys.

``` clojure
(extend-type js/Storage
  ITransientAssociative
  (-assoc! [^js/Storage s key val]
    (set-item s (serialize key) (serialize val))
    s)

  ITransientMap
  (-dissoc! [^js/Storage s key]
    (remove-item s (serialize key))
    s)

  ILookup
  (-lookup
    ([^js/Storage s key]
       (-lookup s key nil))
    ([^js/Storage s key not-found]
       (let [sk (serialize key)]
         (if (contains-key? s sk)
           (deserialize (get-item s sk))
           not-found)))))
```

Now we’re able to perform some operations on either session or local
storage, let’s give them a try:

``` clojure
(def local-storage js/localStorage)
(def session-storage js/sessionStorage)

(assoc! local-storage :foo :bar)

(:foo local-storage)
;; => :bar

(dissoc! local-storage :foo)

(get local-storage :foo)
;; => nil

(get local-storage :foo :default)
;; => :default
```

Finally, we want to be able to use `conj!` and `persistent!` on local
storage so we must implement the `ITransientCollection` protocol, let’s
give it a go:

``` clojure
(extend-type js/Storage
  ITransientCollection
  (-conj! [^js/Storage s ^IMapEntry kv]
    (assoc! s (key kv) (val kv))
    s)

  (-persistent! [^js/Storage s]
    (into {}
          (for [i (range (count s))
                :let [k (.key s i)
                      v (get-item s k)]]
            [(deserialize k) (deserialize v)]))))
```

`conj!` simply obtains the key and value from the map entry and
delegates to `assoc!`. `persistent!` deserializes every key-value pair
in the storage and returns an immutable snapshot of the storage as a
ClojureScript map. Let’s try it out:

``` clojure
(clear! local-storage)

(persistent! local-storage)
;; => {}

(conj! local-storage [:foo :bar])
(conj! local-storage [:baz :xyz])

(persistent! local-storage)
;; => {:foo :bar, :baz :xyz}
```

#### Transient vectors and sets

We’ve learned about most of the protocols for transient data structures
but we’re missing two: `ITransientVector` for using `assoc!` on
transient vectors and `ITransientSet` for using `disj!` on transient
sets.

For illustrating the `ITransientVector` protocol we’ll extend the
JavaScript array type for making it an associative transient data
structure:

``` clojure
(extend-type array
  ITransientAssociative
  (-assoc! [arr key val]
    (if (number? key)
      (-assoc-n! arr key val)
      (throw (js/Error. "Array's key for assoc! must be a number."))))

  ITransientVector
  (-assoc-n! [arr n val]
    (.splice arr n 1 val)
    arr))

(def a #js [1 2 3])
;; => #js [1 2 3]

(assoc! a 0 42)
;; => #js [42 2 3]

(assoc! a 1 43)
;; => #js [42 43 3]

(assoc! a 2 44)
;; => #js [42 43 44]
```

For illustrating the `ITransientSet` protocol we’ll extend the ES6 Set
type for making it a transient set, supporting the `conj!`, `disj!` and
`persistent!` operations. Note that we’ve extended the Set type
previously for being able to convert it to ClojureScript and we’ll take
advantage of that fact.

``` clojure
(extend-type js/Set
  ITransientCollection
  (-conj! [s v]
    (.add s v)
    s)

  (-persistent! [s]
   (js->clj s))

  ITransientSet
  (-disjoin! [s v]
    (.delete s v)
    s))

(def s (js/Set.))

(conj! s 1)
(conj! s 1)
(conj! s 2)
(conj! s 2)

(persistent! s)
;; => #{1 2}

(disj! s 1)

(persistent! s)
;; => #{2}
```

## CSP (with core.async)

CSP stands for Communicating Sequential Processes, which is a formalism
for describing concurrent systems pioneered by C. A. R. Hoare in 1978.
It is a concurrency model based on message passing and synchronization
through channels. An in-depth look at the theoretical model behind CSP
is beyond the scope of this book; instead we’ll focus on presenting the
concurrency primitives that `core.async` offers.

`core.async` is not part of ClojureScript core but it’s implemented as a
library. Even though it is not part of the core language it’s widely
used. Many libraries build on top of the `core.async` primitives, so we
think it is worth covering in the book. It’s also a good example of the
syntactic abstractions that can be achieved by transforming code with
ClojureScript macros, so we’ll jump right in. You’ll need to have
`core.async` installed to run the examples presented in this section.

### Channels

Channels are like conveyor belts, we can put and take a single value at
a time from them. They can have multiple readers and writers, and they
are the fundamental message-passing mechanism of `core.async`. In order
to see how it works, we’ll create a channel to perform some operations
on it.

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(enable-console-print!)

(def ch (chan))

(take! ch #(println "Got a value:" %))
;; => nil

;; there is a now a pending take operation, let's put something on the channel

(put! ch 42)
;; Got a value: 42
;; => 42
```

In the above example we created a channel `ch` using the `chan`
constructor. After that we performed a take operation on the channel,
providing a callback that will be invoked when the take operation
succeeds. After using `put!` to put a value on the channel the take
operation completed and the `"Got a value: 42"` string was printed. Note
that `put!` returned the value that was just put to the channel.

The `put!` function accepts a callback like `take!` does but we didn’t
provide any in the last example. For puts the callback will be called
whenever the value we provided has been taken. Puts and takes can happen
in any order, let’s do a few puts followed by takes to illustrate the
point:

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(def ch (chan))

(put! ch 42 #(println "Just put 42"))
;; => true
(put! ch 43 #(println "Just put 43"))
;; => true

(take! ch #(println "Got" %))
;; Got 42
;; Just put 42
;; => nil

(take! ch #(println "Got" %))
;; Got 43
;; Just put 43
;; => nil
```

You may be asking yourself why the `put!` operations return `true`. It
signals that the put operation could be performed, even though the value
hasn’t yet been taken. Channels can be closed, which will cause the put
operations to not succeed:

``` clojure
(require '[cljs.core.async :refer [chan put! close!]])

(def ch (chan))

(close! ch)
;; => nil

(put! ch 42)
;; => false
```

The above example was the simplest possible situation but what happens
with pending operations when a channel is closed? Let’s do a few takes
and close the channel and see what happens:

``` clojure
(require '[cljs.core.async :refer [chan put! take! close!]])

(def ch (chan))

(take! ch #(println "Got value:" %))
;; => nil
(take! ch #(println "Got value:" %))
;; => nil

(close! ch)
;; Got value: nil
;; Got value: nil
;; => nil
```

We see that if the channel is closed all the `take!` operations receive
a `nil` value. `nil` in channels is a sentinel value that signals to
takers that the channel has been closed. Because of that, putting a
`nil` value on a channel is not allowed:

``` clojure
(require '[cljs.core.async :refer [chan put!]])

(def ch (chan))

(put! ch nil)
;; Error: Assert failed: Can't put nil in on a channel
```

#### Buffers

We’ve seen that pending take and put operations are enqueued in a
channel but, what happens when there are many pending take or put
operations? Let’s find out by hammering a channel with many puts and
takes:

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(def ch (chan))

(dotimes [n 1025]
  (put! ch n))
;; Error: Assert failed: No more than 1024 pending puts are allowed on a single channel.

(def ch (chan))

(dotimes [n 1025]
  (take! ch #(println "Got" %)))
;; Error: Assert failed: No more than 1024 pending takes are allowed on a single channel.
```

As the example above shows there’s a limit of pending puts or takes on a
channel, it’s currently 1024 but that is an implementation detail that
may change. Note that there can’t be both pending puts and pending takes
on a channel since puts will immediately succeed if there are pending
takes and viceversa.

Channels support buffering of put operations. If we create a channel
with a buffer the put operations will succeed immediately if there’s
room in the buffer and be enqueued otherwise. Let’s illustrate the point
creating a channel with a buffer of one element. The `chan` constructors
accepts a number as its first argument which will cause it to have a
buffer with the given size:

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(def ch (chan 1))

(put! ch 42 #(println "Put succeeded!"))
;; Put succeeded!
;; => true

(dotimes [n 1024]
  (put! ch n))
;; => nil

(put! ch 42)
;; Error: Assert failed: No more than 1024 pending puts are allowed on a single channel.
```

What happened in the example above? We created a channel with a buffer
of size 1 and performed a put operation on it that succeeded immediately
because the value was buffered. After that we did another 1024 puts to
fill the pending put queue and, when trying to put one value more the
channel complained about not being able to enqueue more puts.

Now that we know about how channels work and what are buffers used for
let’s explore the different buffers that `core.async` implements.
Different buffers have different policies and it’s interesting to know
all of them to know when to use what. Channels are unbuffered by
default.

##### Fixed

The fixed size buffer is the one that is created when we give the `chan`
constructor a number and it will have the size specified by the given
number. It is the simplest possible buffer: when full, puts will be
enqueued.

The `chan` constructor accepts either a number or a buffer as its first
argument. The two channels created in the following example both use a
fixed buffer of size 32:

``` clojure
(require '[cljs.core.async :refer [chan buffer]])

(def a-ch (chan 32))

(def another-ch (chan (buffer 32)))
```

##### Dropping

The fixed buffer allows put operations to be enqueued. However, as we
saw before, puts are still queued when the fixed buffer is full. If we
wan’t to discard the put operations that happen when the buffer is full
we can use a dropping buffer.

Dropping buffers have a fixed size and, when they are full puts will
complete but their value will be discarded. Let’s illustrate the point
with an example:

``` clojure
(require '[cljs.core.async :refer [chan dropping-buffer put! take!]])

(def ch (chan (dropping-buffer 2)))

(put! ch 40)
;; => true
(put! ch 41)
;; => true
(put! ch 42)
;; => true

(take! ch #(println "Got" %))
;; Got 40
;; => nil
(take! ch #(println "Got" %))
;; Got 41
;; => nil
(take! ch #(println "Got" %))
;; => nil
```

We performed three put operations and the three of them succeded but,
since the dropping buffer of the channel has size 2, only the first two
values were delivered to the takers. As you can observe the third take
is enqueued since there is no value available, the third put’s value
(42) was discarded.

##### Sliding

The sliding buffer has the opposite policy than the dropping buffer.
When full puts will complete and the oldest value will be discarded in
favor of the new one. The sliding buffer is useful when we are
interested in processing the last puts only and we can afford discarding
old values.

``` clojure
(require '[cljs.core.async :refer [chan sliding-buffer put! take!]])

(def ch (chan (sliding-buffer 2)))

(put! ch 40)
;; => true
(put! ch 41)
;; => true
(put! ch 42)
;; => true

(take! ch #(println "Got" %))
;; Got 41
;; => nil
(take! ch #(println "Got" %))
;; Got 42
;; => nil
(take! ch #(println "Got" %))
;; => nil
```

We performed three put operations and the three of them succeded but,
since the sliding buffer of the channel has size 2, only the last two
values were delivered to the takers. As you can observe the third take
is enqueued since there is no value available since the first put’s
value was discarded.

#### Transducers

As mentioned in the section about transducers, putting values in a
channel can be thought as a transducible process. This means that we can
create channels and hand them a transducer, giving us the ability to
transform the input values before being put in the channel.

If we want to use a transducer with a channel we must supply a buffer
since the reducing function that will be modified by the transducer will
be the buffer’s add function. A buffer’s add function is a reducing
function since it takes a buffer and an input and returns a buffer with
such input incorporated.

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(def ch (chan 1 (map inc)))

(put! ch 41)
;; => true

(take! ch #(println "Got" %))
;; Got 42
;; => nil
```

You may be wondering what happens to a channel when the reducing
function returns a reduced value. It turns out that the notion of
termination for channels is being closed, so channels will be closed
when a reduced value is encountered:

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(def ch (chan 1 (take 2)))

(take! ch #(println "Got" %))
;; => nil
(take! ch #(println "Got" %))
;; => nil
(take! ch #(println "Got" %))
;; => nil

(put! ch 41)
;; => true
(put! ch 42)
;; Got 41
;; => true
(put! ch 43)
;; Got 42
;; Got nil
;; => false
```

We used the `take` stateful transducer to allow maximum 2 puts into the
channel. We then performed three take operations on the channel and we
expect only two to receive a value. As you can see in the above example
the third take got the sentinel `nil` value which indicates that the
channel was closed. Also, the third put operation returned `false`
indicating that it didn’t take place.

#### Handling exceptions

If adding a value to a buffer throws an exception `core.async` the
operation will fail and the exception will be logged to the console.
However, channel constructors accept a third argument: a function for
handling exceptions.

When creating a channel with an exception handler it will be called with
the exception whenever an exception occurs. If the handler returns `nil`
the operation will fail silently and if it returns another value the add
operation will be retried with such value.

``` clojure
(require '[cljs.core.async :refer [chan put! take!]])

(enable-console-print!)

(defn exception-xform
  [rfn]
  (fn [acc input]
    (throw (js/Error. "I fail!"))))

(defn handle-exception
  [ex]
  (println "Exception message:" (.-message ex))
  42)

(def ch (chan 1 exception-xform handle-exception))

(put! ch 0)
;; Exception message: I fail!
;; => true

(take! ch #(println "Got:" %))
;; Got: 42
;; => nil
```

#### Offer and Poll

We’ve learned about the two basic operations on channels so far: `put!`
and `take!`. They either take or put a value and are enqueued if they
can’t be performed immediately. Both functions are asynchronous because
of their nature: they can succeed but be completed at a later time.

`core.async` has two synchronous operations for putting or taking
values: `offer!` and `poll!`. Let’s see how they work through examples.

`offer!` puts a value in a channel if it’s possible to do so
immediately. It returns `true` if the channel received the value and
`false` otherwise. Note that, unlike with `put!`, `offer!` cannot
distinguish between closed and open channels.

``` clojure
(require '[cljs.core.async :refer [chan offer!]])

(def ch (chan 1))

(offer! ch 42)
;; => true

(offer! ch 43)
;; => false
```

`poll!` takes a value from a channel if it’s possible to do so
immediately. Returns the value if succesful and `nil` otherwise. Unlike
`take!`, `poll!` cannot distinguish closed and open channels.

``` clojure
(require '[cljs.core.async :refer [chan offer! poll!]])

(def ch (chan 1))

(poll! ch)
;; => nil

(offer! ch 42)
;; => true

(poll! ch)
;; => 42
```

### Processes

We learned all about channels but there is still a missing piece in the
puzzle: processes. Processes are pieces of logic that run independently
and use channels for communication and coordination. Puts and takes
inside a process will stop the process until the operation completes.
Stopping a process doesn’t block the only thread we have in the
environments where ClojureScript runs. Instead, it will be resumed at a
later time when the operation is waiting for being performed.

Processes are launched using the `go` macro and puts and takes use the
`<!` and `>!` placeholders. The `go` macro rewrites your code to use
callbacks but inside `go` everything looks like synchronous code, which
makes understanding it straightforward:

``` clojure
(require '[cljs.core.async :refer [chan <! >!]])
(require-macros '[cljs.core.async.macros :refer [go]])

(enable-console-print!)

(def ch (chan))

(go
  (println [:a] "Gonna take from channel")
  (println [:a] "Got" (<! ch)))

(go
  (println [:b] "Gonna put on channel")
  (>! ch 42)
  (println [:b] "Just put 42"))

;; [:a] Gonna take from channel
;; [:b] Gonna put on channel
;; [:b] Just put 42
;; [:a] Got 42
```

In the above example we are launching a process with `go` that takes a
value from `ch` and prints it to the console. Since the value isn’t
immediately available it will park until it can resume. After that we
launch another process that puts a value on the channel.

Since there is a pending take the put operation succeeds and the value
is delivered to the first process, then both processes terminate.

Both `go` blocks run independently and, even though they are executed
asynchronously, they look like synchronous code. The above go blocks are
fairly simple but being able to write concurrent processes that
coordinate via channels is a very powerful tool for implementing complex
asynchronous workflows. Channels also offer a great decoupling of
producers and consumers.

Processes can wait for an arbitrary amount of time too, there is a
`timeout` function that return a channel that will be closed after the
given amount of miliseconds. Combining a timeout channel with a take
operation inside a go block gives us the ability to sleep:

``` clojure
(require '[cljs.core.async :refer [<! timeout]])
(require-macros '[cljs.core.async.macros :refer [go]])

(enable-console-print!)

(defn seconds
  []
  (.getSeconds (js/Date.)))

(println "Launching go block")

(go
  (println [:a] "Gonna take a nap" (seconds))
  (<! (timeout 1000))
  (println [:a] "I slept one second, bye!" (seconds)))

(println "Block launched")

;; Launching go block
;; Block launched
;; [:a] Gonna take a nap 9
;; [:a] I slept one second, bye! 10
```

As we can see in the messages printed, the process does nothing for one
second when it blocks in the take operation of the timeout channel. The
program continues and after one second the process resumes and
terminates.

#### Choice

Apart from putting and taking one value at a time inside a go block we
can also make a non-deterministic choice on multiple channel operations
using `alts!`. `alts!` is given a series of channel put or take
operations (note that we can also try to put and take in a channel at
the same time) and only performs one as soon as is ready; if multiple
operations can be performed when calling `alts!` it will do a pseudo
random choice by default.

We can easily try an operation on a channel and cancel it after a
certain amount of time combining the `timeout` function and `alts!`.
Let’s see how:

``` clojure
(require '[cljs.core.async :refer [chan <! timeout alts!]])
(require-macros '[cljs.core.async.macros :refer [go]])

(enable-console-print!)

(def ch (chan))

(go
  (println [:a] "Gonna take a nap")
  (<! (timeout 1000))
  (println [:a] "I slept one second, trying to put a value on channel")
  (>! ch 42)
  (println [:a] "I'm done!"))

(go
  (println [:b] "Gonna try taking from channel")
  (let [cancel (timeout 300)
        [value ch] (alts! [ch cancel])]
    (if (= ch cancel)
      (println [:b] "Too slow, take from channel cancelled")
      (println [:b] "Got" value))))

;; [:a] Gonna take a nap
;; [:b] Gonna try taking from channel
;; [:b] Too slow, take from channel cancelled
;; [:a] I slept one second, trying to put a value on channel
```

In the example above we launched a go block that, after waiting for a
second, puts a value in the `ch` channel. The other go block creates a
`cancel` channel, which will be closed after 300 miliseconds. After
that, it tries to read from both `ch` and `cancel` at the same time
using `alts!`, which will succeed whenever it can take a value from
either of those channels. Since `cancel` is closed after 300
miliseconds, `alts!` will succeed since takes from closed channel return
the `nil` sentinel. Note that `alts!` returns a two-element vector with
the returned value of the operation and the channel where it was
performed.

This is why we are able to detect whether the read operation was
performed in the `cancel` channel or in `ch`. I suggest you copy this
example and set the first process timeout to 100 miliseconds to see how
the read operation on `ch` succeeds.

We’ve learned how to choose between read operations so let’s look at how
to express a conditional write operation in `alts!`. Since we need to
provide the channel and a value to try to put on it, we’ll use a two
element vector with the channel and the value for representing write
operations.

Let’s see an example:

``` clojure
(require '[cljs.core.async :refer [chan <! alts!]])
(require-macros '[cljs.core.async.macros :refer [go]])

(enable-console-print!)

(def a-ch (chan))
(def another-ch (chan))

(go
  (println [:a] "Take a value from `a-ch`")
  (println [:a] "Got" (<! a-ch))
  (println [:a] "I'm done!"))

(go
  (println [:b] "Take a value from `another-ch`")
  (println [:a] "Got" (<! another-ch))
  (println [:b] "I'm done!"))

(go
  (println [:c] "Gonna try putting in both channels simultaneously")
  (let [[value ch] (alts! [[a-ch 42]
                           [another-ch 99]])]
    (if (= ch a-ch)
      (println [:c] "Put a value in `a-ch`")
      (println [:c] "Put a value in `another-ch`"))))

;; [:a] Take a value from `a-ch`
;; [:b] Take a value from `another-ch`
;; [:c] Gonna try putting in both channels simultaneously
;; [:c] Put a value in `a-ch`
;; [:a] Got 42
;; [:a] I'm done!
```

When running the above example only the put operation on the `a-ch`
channel has succeeded. Since both channels are ready to take a value
when the `alts!` occurs you may get different results when running this
code.

#### Priority

`alts!` default is to make a non-deterministic choice whenever several
operations are ready to be performed. We can instead give priority to
the operations passing the `:priority` option to `alts!`. Whenever
`:priority` is `true`, if more than one operation is ready they will be
tried in order.

``` clojure
(require '[cljs.core.async :refer [chan >! alts!]])
(require-macros '[cljs.core.async.macros :refer [go]])

(enable-console-print!)

(def a-ch (chan))
(def another-ch (chan))

(go
  (println [:a] "Put a value on `a-ch`")
  (>! a-ch 42)
  (println [:a] "I'm done!"))

(go
  (println [:b] "Put a value on `another-ch`")
  (>! another-ch 99)
  (println [:b] "I'm done!"))

(go
  (println [:c] "Gonna try taking from both channels with priority")
  (let [[value ch] (alts! [a-ch another-ch] :priority true)]
    (if (= ch a-ch)
      (println [:c] "Got" value "from `a-ch`")
      (println [:c] "Got" value "from `another-ch`"))))

;; [:a] Put a value on `a-ch`
;; [:a] I'm done!
;; [:b] Put a value on `another-ch`
;; [:b] I'm done!
;; [:c] Gonna try taking from both channels with priority
;; [:c] Got 42 from `a-ch`
```

Since both `a-ch` and `another-ch` had a value to read when the `alts!`
was executed and we set the `:priority` option to true, `a-ch` has
preference. You can try deleting the `:priority` option and running the
example multiple times to see that, without priority, `alts!` makes a
non-deterministic choice.

#### Defaults

Another interesting bit of `alts!` is that it can return immediately if
no operation is ready and we provide a default value. We can
conditionally do a choice on the operations if and only if any of them
is ready, returning a default value if it’s not.

``` clojure
(require '[cljs.core.async :refer [chan alts!]])
(require-macros '[cljs.core.async.macros :refer [go]])

(def a-ch (chan))
(def another-ch (chan))

(go
  (println [:a] "Gonna try taking from any of the channels without blocking")
  (let [[value ch] (alts! [a-ch another-ch] :default :not-ready)]
    (if (and (= value :not-ready)
             (= ch :default))
      (println [:a] "No operation is ready, aborting")
      (println [:a] "Got" value))))

;; [:a] Gonna try taking from any of the channels without blocking
;; [:a] No operation is ready, aborting
```

As you can see in the above example, if no operation is ready the value
returned by `alts!` is the one we supplied after the `:default` key when
calling it and the channel is the `:default` keyword itself.

### Combinators

Now that we’re acquainted with channels and processes it’s time to
explore some interesting combinators for working with channels that are
present in `core.async`. This section includes a brief description of
all of them together with a simple example of their usage.

#### pipe

`pipe` takes an input and output channels and pipes all the values put
on the input channel to the output one. The output channel is closed
whenever the source is closed unless we provide a `false` third
argument:

``` clojure
(require '[cljs.core.async :refer [chan pipe put! <! close!]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

(def in (chan))
(def out (chan))

(pipe in out)

(go-loop [value (<! out)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Got" value)
      (println [:a] "Waiting for a value")
      (recur (<! out)))))

(put! in 0)
;; => true
(put! in 1)
;; => true
(close! in)

;; [:a] Got 0
;; [:a] Waiting for a value
;; [:a] Got 1
;; [:a] Waiting for a value
;; [:a] I'm done!
```

In the above example we used the `go-loop` macro for reading values
recursively until the `out` channel is closed. Notice that when we close
the `in` channel the `out` channel is closed too, making the `go-loop`
terminate.

#### pipeline-async

`pipeline-async` takes a number for controlling parallelism, an output
channel, an asynchronous function and an input channel. The asynchronous
function has two arguments: the value put in the input channel and a
channel where it should put the result of its asynchronous operation,
closing the result channel after finishing. The number controls the
number of concurrent go blocks that will be used for calling the
asynchronous function with the inputs.

The output channel will receive outputs in an order relative to the
input channel, regardless the time each asynchronous function call takes
to complete. It has an optional last parameter that controls whether the
output channel will be closed when the input channel is closed, which
defaults to `true`.

``` clojure
(require '[cljs.core.async :refer [chan pipeline-async put! <! close!]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

(def in (chan))
(def out (chan))
(def parallelism 3)

(defn wait-and-put [value ch]
  (let [wait (rand-int 1000)]
    (js/setTimeout (fn []
                     (println "Waiting" wait "miliseconds for value" value)
                     (put! ch wait)
                     (close! ch))
                   wait)))

(pipeline-async parallelism out wait-and-put in)

(go-loop [value (<! out)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Got" value)
      (println [:a] "Waiting for a value")
      (recur (<! out)))))

(put! in 1)
(put! in 2)
(put! in 3)
(close! in)

;; Waiting 164 miliseconds for value 3
;; Waiting 304 miliseconds for value 2
;; Waiting 908 miliseconds for value 1
;; [:a] Got 908
;; [:a] Waiting for a value
;; [:a] Got 304
;; [:a] Waiting for a value
;; [:a] Got 164
;; [:a] Waiting for a value
;; [:a] I'm done!
```

#### pipeline

`pipeline` is similar to `pipeline-async` but instead of taking and
asynchronous function it takes a transducer instead. The transducer will
be applied independently to each input.

``` clojure
(require '[cljs.core.async :refer [chan pipeline put! <! close!]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

(def in (chan))
(def out (chan))
(def parallelism 3)

(pipeline parallelism out (map inc) in)

(go-loop [value (<! out)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Got" value)
      (println [:a] "Waiting for a value")
      (recur (<! out)))))

(put! in 1)
(put! in 2)
(put! in 3)
(close! in)

;; [:a] Got 2
;; [:a] Waiting for a value
;; [:a] Got 3
;; [:a] Waiting for a value
;; [:a] Got 4
;; [:a] Waiting for a value
;; [:a] I'm done!
```

#### split

`split` takes a predicate and a channel and returns a vector with two
channels, the first of which will receive the values for which the
predicate is true, the second will receive those for which the predicate
is false. We can optionally pass a buffer or number for the channels
with the third (true channel) and fourth (false channel) arguments.

``` clojure
(require '[cljs.core.async :refer [chan split put! <! close!]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

(def in (chan))
(def chans (split even? in))
(def even-ch (first chans))
(def odd-ch (second chans))

(go-loop [value (<! even-ch)]
  (if (nil? value)
    (println [:evens] "I'm done!")
    (do
      (println [:evens] "Got" value)
      (println [:evens] "Waiting for a value")
      (recur (<! even-ch)))))

(go-loop [value (<! odd-ch)]
  (if (nil? value)
    (println [:odds] "I'm done!")
    (do
      (println [:odds] "Got" value)
      (println [:odds] "Waiting for a value")
      (recur (<! odd-ch)))))

(put! in 0)
(put! in 1)
(put! in 2)
(put! in 3)
(close! in)

;; [:evens] Got 0
;; [:evens] Waiting for a value
;; [:odds] Got 1
;; [:odds] Waiting for a value
;; [:odds] Got 3
;; [:odds] Waiting for a value
;; [:evens] Got 2
;; [:evens] Waiting for a value
;; [:evens] I'm done!
;; [:odds] I'm done!
```

#### reduce

`reduce` takes a reducing function, initial value and an input channel.
It returns a channel with the result of reducing over all the values put
on the input channel before closing it using the given initial value as
the seed.

``` clojure
(require '[cljs.core.async :as async :refer [chan put! <! close!]])
(require-macros '[cljs.core.async.macros :refer [go]])

(def in (chan))

(go
  (println "Result" (<! (async/reduce + (+) in))))

(put! in 0)
(put! in 1)
(put! in 2)
(put! in 3)
(close! in)

;; Result: 6
```

#### onto-chan

`onto-chan` takes a channel and a collection and puts the contents of
the collection into the channel. It closes the channel after finishing
although it accepts a third argument for specifying if it should close
it or not. Let’s rewrite the `reduce` example using `onto-chan`:

``` clojure
(require '[cljs.core.async :as async :refer [chan put! <! close! onto-chan]])
(require-macros '[cljs.core.async.macros :refer [go]])

(def in (chan))

(go
  (println "Result" (<! (async/reduce + (+) in))))

(onto-chan in [0 1 2 3])

;; Result: 6
```

#### to-chan

`to-chan` takes a collection and returns a channel where it will put
every value in the collection, closing the channel afterwards.

``` clojure
(require '[cljs.core.async :refer [chan put! <! close! to-chan]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

(def ch (to-chan (range 3)))

(go-loop [value (<! ch)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Got" value)
      (println [:a] "Waiting for a value")
      (recur (<! ch)))))

;; [:a] Got 0
;; [:a] Waiting for a value
;; [:a] Got 1
;; [:a] Waiting for a value
;; [:a] Got 2
;; [:a] Waiting for a value
;; [:a] I'm done!
```

#### merge

`merge` takes a collection of input channels and returns a channel where
it will put every value that is put on the input channels. The returned
channel will be closed when all the input channels have been closed. The
returned channel will be unbuffered by default but a number or buffer
can be provided as the last argument.

``` clojure
(require '[cljs.core.async :refer [chan put! <! close! merge]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

(def in1 (chan))
(def in2 (chan))
(def in3 (chan))

(def out (merge [in1 in2 in3]))

(go-loop [value (<! out)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Got" value)
      (println [:a] "Waiting for a value")
      (recur (<! out)))))

(put! in1 1)
(close! in1)
(put! in2 2)
(close! in2)
(put! in3 3)
(close! in3)

;; [:a] Got 3
;; [:a] Waiting for a value
;; [:a] Got 2
;; [:a] Waiting for a value
;; [:a] Got 1
;; [:a] Waiting for a value
;; [:a] I'm done!
```

### Higher-level abstractions

We’ve learned the about the low-level primitives of `core.async` and the
combinators that it offers for working with channels. `core.async` also
offers some useful, higher-level abstractions on top of channels that
can serve as building blocks for more advanced functionality.

#### Mult

Whenever we have a channel whose values have to be broadcasted to many
others, we can use `mult` for creating a multiple of the supplied
channel. Once we have a mult, we can attach channels to it using `tap`
and dettach them using `untap`. Mults also support removing all tapped
channels at once with `untap-all`.

Every value put in the source channel of a mult is broadcasted to all
the tapped channels, and all of them must accept it before the next item
is broadcasted. For preventing slow takers from blocking the mult’s
values we must use buffering on the tapped channels judiciously.

Closed tapped channels are removed automatically from the mult. When
putting a value in the source channels when there are still no taps such
value will be dropped.

``` clojure
(require '[cljs.core.async :refer [chan put! <! close! timeout mult tap]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

;; Source channel and mult
(def in (chan))
(def m-in (mult in))

;; Sink channels
(def a-ch (chan))
(def another-ch (chan))

;; Taker for `a-ch`
(go-loop [value (<! a-ch)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Got" value)
      (recur (<! a-ch)))))

;; Taker for `another-ch`, which sleeps for 3 seconds between takes
(go-loop [value (<! another-ch)]
  (if (nil? value)
    (println [:b] "I'm done!")
    (do
      (println [:b] "Got" value)
      (println [:b] "Resting 3 seconds")
      (<! (timeout 3000))
      (recur (<! another-ch)))))

;; Tap the two channels to the mult
(tap m-in a-ch)
(tap m-in another-ch)

;; See how the values are delivered to `a-ch` and `another-ch`
(put! in 1)
(put! in 2)

;; [:a] Got 1
;; [:b] Got 1
;; [:b] Resting for 3 seconds
;; [:a] Got 2
;; [:b] Got 2
;; [:b] Resting for 3 seconds
```

#### Pub-sub

After learning about mults you could imagine how to implement a pub-sub
abstraction on top of `mult`, `tap` and `untap` but since it’s a widely
used communication mechanism `core.async` already implements this
functionality.

Instead of creating a mult from a source channel, we create a
publication with `pub` giving it a channel and a function that will be
used for extracting the topic of the messages.

We can subscribe to a publication with `sub`, giving it the publication
we want to subscribe to, the topic we are interested in and a channel to
put the messages that have the given topic. Note that we can subscribe a
channel to multiple topics.

`unsub` can be given a publication, topic and channel for unsubscribing
such channel from the topic. `unsub-all` can be given a publication and
a topic to unsubscribe every channel from the given topic.

``` clojure
(require '[cljs.core.async :refer [chan put! <! close! pub sub]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

;; Source channel and publication
(def in (chan))
(def publication (pub in :action))

;; Sink channels
(def a-ch (chan))
(def another-ch (chan))

;; Channel with `:increment` action
(sub publication :increment a-ch)

(go-loop [value (<! a-ch)]
  (if (nil? value)
    (println [:a] "I'm done!")
    (do
      (println [:a] "Increment:" (inc (:value value)))
      (recur (<! a-ch)))))

;; Channel with `:double` action
(sub publication :double another-ch)

(go-loop [value (<! another-ch)]
  (if (nil? value)
    (println [:b] "I'm done!")
    (do
      (println [:b] "Double:" (* 2 (:value value)))
      (recur (<! another-ch)))))

;; See how values are delivered to `a-ch` and `another-ch` depending on their action
(put! in {:action :increment :value 98})
(put! in {:action :double :value 21})

;; [:a] Increment: 99
;; [:b] Double: 42
```

#### Mixer

As we learned in the section about `core.async` combinators, we can use
the `merge` function for combining multiple channels into one. When
merging multiple channels, every value put in the input channels will
end up in the merged channel. However, we may want more finer-grained
control over which values put in the input channels end up in the output
channel, that’s where mixers come in handy.

`core.async` gives us the mixer abstraction, which we can use to combine
multiple input channnels into an output channel. The interesting part of
the mixer is that we can mute, pause and listen exclusively to certain
input channels.

We can create a mixer given an output channel with `mix`. Once we have a
mixer we can add input channels into the mix using `admix`, remove it
using `unmix` or remove every input channel with `unmix-all`.

For controlling the state of the input channel we use the `toggle`
function giving it the mixer and a map from channels to their states.
Note that we can add channels to the mix using `toggle`, since the map
will be merged with the current state of the mix. The state of a channel
is a map which can have the keys `:mute`, `:pause` and `:solo` mapped to
a boolean.

Let’s see what muting, pausing and soloing channels means:

  - A muted input channel means that, while still taking values from it,
    they won’t be forwarded to the output channel. Thus, while a channel
    is muted, all the values put in it will be discarded.

  - A paused input channel means that no values will be taken from it.
    This means that values put in the channel won’t be forwarded to the
    output channel nor discarded.

  - When soloing one or more channels the output channel will only
    receive the values put in soloed channels. By default non-soloed
    channels are muted but we can use `solo-mode` to decide between
    muting or pausing non-soloed channels.

That was a lot of information so let’s see an example to improve our
understanding. First of all, we’ll set up a mixer with an `out` channel
and add three input channels to the mix. After that, we’ll be printing
all the values received on the `out` channel to illustrate the control
over input channels:

``` clojure
(require '[cljs.core.async :refer [chan put! <! close! mix admix
                                   unmix toggle solo-mode]])
(require-macros '[cljs.core.async.macros :refer [go-loop]])

;; Output channel and mixer
(def out (chan))
(def mixer (mix out))

;; Input channels
(def in-1 (chan))
(def in-2 (chan))
(def in-3 (chan))

(admix mixer in-1)
(admix mixer in-2)
(admix mixer in-3)

;; Let's listen to the `out` channel and print what we get from it
(go-loop [value (<! out)]
  (if (nil? value)
    (println [:a] "I'm done")
    (do
      (println [:a] "Got" value)
      (recur (<! out)))))
```

By default, every value put in the input channels will be put in the
`out` channel:

``` clojure
(do
  (put! in-1 1)
  (put! in-2 2)
  (put! in-3 3))

;; [:a] Got 1
;; [:a] Got 2
;; [:a] Got 3
```

Let’s pause the `in-2` channel, put a value in every input channel and
resume `in-2`:

``` clojure
(toggle mixer {in-2 {:pause true}})
;; => true

(do
  (put! in-1 1)
  (put! in-2 2)
  (put! in-3 3))

;; [:a] Got 1
;; [:a] Got 3

(toggle mixer {in-2 {:pause false}})

;; [:a] Got 2
```

As you can see in the example above, the values put in the paused
channels aren’t discarded. For discarding values put in an input channel
we have to mute it, let’s see an example:

``` clojure
(toggle mixer {in-2 {:mute true}})
;; => true

(do
  (put! in-1 1)
  (put! in-2 2)  ;; `out` will never get this value since it's discarded
  (put! in-3 3))

;; [:a] Got 1
;; [:a] Got 3

(toggle mixer {in-2 {:mute false}})
```

We put a value `2` in the `in-2` channel and, since the channel was
muted at the time, the value is discarded and never put into `out`.
Let’s look at the third state a channel can be inside a mixer: solo.

As we mentioned before, soloing channels of a mixer implies muting the
rest of them by default:

``` clojure
(toggle mixer {in-1 {:solo true}
               in-2 {:solo true}})
;; => true

(do
  (put! in-1 1)
  (put! in-2 2)
  (put! in-3 3)) ;; `out` will never get this value since it's discarded

;; [:a] Got 1
;; [:a] Got 2

(toggle mixer {in-1 {:solo false}
               in-2 {:solo false}})
```

However, we can set the mode the non-soloed channels will be in while
there are soloed channels. Let’s set the default non-solo mode to pause
instead of the default mute:

``` clojure
(solo-mode mixer :pause)
;; => true
(toggle mixer {in-1 {:solo true}
               in-2 {:solo true}})
;; => true

(do
  (put! in-1 1)
  (put! in-2 2)
  (put! in-3 3))

;; [:a] Got 1
;; [:a] Got 2

(toggle mixer {in-1 {:solo false}
               in-2 {:solo false}})

;; [:a] Got 3
```

# Appendix

## Appendix A: Interactive development with Figwheel

### Introduction

In this project, we will **not** do “Hello World”— that has been done to
death. Instead, this project will be a web page that asks you for your
age in years and tells you how many days that is, using an approximation
of 365 days per year.

For this project, we will use the *figwheel* leiningen plugin. This
plugin creates a fully interactive, REPL-based, autoreloading
environment.

### First steps

The first step is to create the new project using the *figwheel* lein
template. We will name the project `age`, and create it by typing:

``` bash
$ lein new figwheel age
Retrieving figwheel/lein-template/0.3.5/lein-template-0.3.5.pom from clojars
Retrieving figwheel/lein-template/0.3.5/lein-template-0.3.5.jar from clojars
Generating fresh 'lein new' figwheel project.
$ cd age # move into newly created project directory
```

The project has the following structure:

    > tree age      # the linux "tree" utility displays dir structure
    age
    ├── .gitignore
    ├── project.clj
    ├── README.md
    ├── resources
    │   └── public
    │       ├── css
    │       │   └── style.css
    │       └── index.html
    └── src
        └── age
            └── core.cljs

The `project.clj` file contains information that Leiningen uses to
download dependencies and build the project. For now, just trust that
everything in that file is exactly as it should be.

Open the `index.html` file and make it look like the following:

``` html
<!DOCTYPE html>
<html>
  <head>
    <link href="css/style.css" rel="stylesheet" type="text/css">
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
  </head>
  <body>
    <div id="app">
      <h1>Age in Days</h1>
      <p>
        Enter your age in years:
        <input type="text" size="5" id="years">
        <button id="calculate">Calculate</button>
      </p>
      <p id="feedback"></p>
    </div>
    <script src="js/compiled/age.js" type="text/javascript"></script>
  </body>
</html>
```

The `core.cljs` file is where all the action takes place. For now, leave
it exactly as it is, and start the figwheel environment, which will load
a large number of dependencies and start a server.

``` bash
$ lein figwheel
Retrieving lein-figwheel/lein-figwheel/0.5.2/lein-figwheel-0.5.2.pom from clojars
Retrieving figwheel-sidecar/figwheel-sidecar/0.5.2/figwheel-sidecar-0.5.2.pom from clojars
Retrieving org/clojure/clojurescript/1.7.228/clojurescript-1.7.228.pom from central
... # much more output
Prompt will show when Figwheel connects to your application
```

If you are using Linux or Mac OS X, type the command as `rlwrap lein
figwheel`. In your browser, go to URL `http://localhost:3449`, and you
will see something like the following screenshot if you open up the web
console.

![Screenshot of web page and console](./images/localhost1.png)

The terminal will then give you a REPL prompt:

``` bash
$ rlwrap lein figwheel
To quit, type: :cljs/quit
cljs.user=>
```

For now, do what it says in the `core.cljs` file — change the
`(println…​)` and then save the file. When you do so, you will see
the change reflected immediately in the browser.

Then make an error by adding an extra closing parenthesis to the
`println`. When you save the file, will see a compile error in the
browser window.

### Interacting with JavaScript

In the REPL window, type the following to invoke JavaScript’s
`window.alert()` function:

``` clojure
(.alert js/window "It works!")
;; => nil
```

The general format for invoking a JavaScript function from ClojureScript
is to give the function name (preceded by a dot), the object that “owns”
the function, and any parameters to that function. You should see an
alert appear in your browser window; when you dismiss the alert, the
REPL will print `nil` and give you another prompt. You can also do it
this way:

``` clojure
(js/alert "It works!")
;; => nil
```

However, the first version always works, so, for consistency, we will
use that notation throughout this tutorial.

JavaScript objects may be instanciated from ClojureScript using the same
interop syntax as for Java/Clojure interop, using the class name
followed by a dot. JavaScript methods may also be called using the
familiar interop syntax:

``` clojure
> (def d (js/Date.))
;; => #'cljs.user/d
> d
;; => #inst "2016-04-03T21:04:29.908-00:00"
> (.getFullYear d)
;; => 2016

> (.toUpperCase "doh!")
;; => "DOH!"

> (.getElementById js/document "years")
;; => #object[HTMLInputElement [object HTMLInputElement]]
```

The next example shows where we’re headed. To retrieve an object’s
property, use the dot-dash or "access" syntax `.-` before the property
name. In the browser window, type a number into the input field (in the
example, we typed `24`), then do this in the REPL.

``` clojure
(def year-field (.getElementById js/document "years"))
;; => #'cljs.user/year-field

(.-value year-field)
;; => "24"

(set! (.-value year-field) "25")
;; => "25"
```

This works, but it is little more than a direct translation of
JavaScript to ClojureScript. The next step is to add event handling to
the button. Event handling is loaded with all sorts of cross-platform
compatibility issues, so we’d like a step up from plain ClojureScript.

The solution is the Google Closure library. To use it, you have to
modify the `:require` clause at the beginning of `core.cljs`:

``` clojure
(ns ^:figwheel-always age.core
  (:require [goog.dom :as dom]
            [goog.events :as events]))
```

Getting an element and setting its value is now slightly easier. Do this
in the REPL and see the results in the browser window.

``` clojure
(in-ns 'age.core)
(def y (dom/getElement "years"))
;; => #'age.core/y

(set! (.-value y) "26")
;; => "26"

(dom/setTextContent (dom/getElement "feedback") "This works!")
;; => nil
```

To add an event, you define a function that takes a single argument (the
event to be handled), and then tell the appropriate HTML element to
listen for it. The `events/listen` function takes three arguments: the
element to listen to, the event to listen for, and the function that
will handle the event.

``` clojure
(defn testing [evt] (js/alert "Responding to click"))
;; => #'age.core/testing

(events/listen (dom/getElement "calculate") "click" testing)
;; => #<[object Object]>
```

After doing this, the browser should respond to a click on the button.
If you would like to remove the listener, use `unlisten`.

``` clojure
(events/unlisten (dom/getElement "calculate") "click" testing)
;; => true
```

Now, put that all together in the `core.cljs` file as follows:

``` clojure
(ns ^:figwheel-always age.core
  (:require [goog.dom :as dom]
            [goog.events :as events]))

(enable-console-print!)

(defn calculate
  [event]
  (let [years (.parseInt js/window (.-value (dom/getElement "years")))
        days (* 365 years)]
    (dom/setTextContent (dom/getElement "feedback")
                        (str "That is " days " days old."))))

(defn on-js-reload [])

(events/listen (dom/getElement "calculate") "click" calculate)
```

## Appendix B: Setting up a ClojureScript development environment

### Cursive

TODO

### Emacs

TODO

### Vim

TODO

# Acknowledgments

Special thanks to:

  - **J David Eisenberg**: For the huge amount of time spend in fixing
    all kind of errors and writing entire sections of the book, as well
    as making very valuable suggestions.

And here is an inevitably incomplete list of MUCH-APPRECIATED
CONTRIBUTORS — people who have submitted corrections, new ideas and
generally made the *ClojureScript Unraveled* book much better:

  - Anler Hernández Peral (@anler)

  - Diego Sevilla Ruiz (@dsevilla)

  - Eduardo Ferro Aldama (@eferro)

  - Tyler Anderson (@Tyler-Anderson)

  - Chris Ulrich (@chrisulrich)

  - Jean Hadrien Chabran (@jhchabran)

  - Tienson Qin (@tiensonqin)

  - FungusHumungus (@FungusHumungus),

  - Chris Charles (@ccharles)

  - Jearvon Dharrie (@iamjarvo)

  - Shaun LeBron (@shaunlebron)

  - Wodin (@wodin)

  - Crocket (@crocket)

# Further Reading

Here is a list of more resources about ClojureScript.

  - [ClojureScript wiki](https://github.com/clojure/clojurescript/wiki):
    a community-mantained wiki about ClojureScript.

  - [API
    Reference](https://github.com/cljsinfo/cljs-api-docs/blob/catalog/INDEX.md):
    a community-maintained complete language api reference.

  - [ClojureScript Cheatsheet](http://cljs.info/cheatsheet/): a
    comprehensive reference of the ClojureScript language.

  - [Études for
    ClojureScript](http://catcode.com/etudes-for-clojurescript/toc.html):
    a collection of exercises for learning ClojureScript.

  - [ClojureScript made easy](http://clojurescriptmadeeasy.com/): a
    collection of short articles about solving common problems in
    ClojureScript.

  - [The Google Closure Library API
    reference](http://google.github.io/closure-library/api/)
