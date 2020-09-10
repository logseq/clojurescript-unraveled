---
title: Language (the basics)
---

# First steps with Lisp syntax

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

# The base data types

The ClojureScript language has a rich set of data types like most
programming languages. It provides scalar data types that will be very
familiar to you, such as numbers, strings, and floats. Beyond these, it
also provides a great number of others that might be less familiar, such
as symbols, keywords, regexes (regular expressions), vars, atoms, and
volatiles.

*ClojureScript* embraces the host language, and where possible, uses the
host’s provided types. For example: numbers and strings are used as is
and behave in the same way as in JavaScript.

## Numbers

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

## Keywords

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

### Namespaced keywords

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

## Symbols

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

## Strings

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

## Characters

*ClojureScript* also lets you write single characters using Clojure’s
character literal syntax.

``` clojure
\a        ; The lowercase a character
\newline  ; The newline character
```

Since the host language doesn’t contain character literals,
*ClojureScript* characters are transformed behind the scenes into single
character JavaScript strings.

## Collections

Another big step in explaining a language is to explain its collections
and collection abstractions. *ClojureScript* is not an exception to this
rule.

*ClojureScript* comes with many types of collections. The main
difference between *ClojureScript* collections and collections in other
languages is that they are persistent and immutable.

Before moving on to these (possibly) unknown concepts, we’ll present a
high-level overview of existing collection types in *ClojureScript*.

### Lists

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

### Vectors

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

### Maps

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

### Sets

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

# Vars

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

# Functions

## The first contact

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

## Defining your own functions

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

## Functions with multiple arities

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

## Variadic functions

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

## Short syntax for anonymous functions

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

# Flow control

*ClojureScript* has a very different approach to flow control than
languages like JavaScript, C, etc.

## Branching with `if`

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

## Branching with `cond`

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

## Branching with `case`

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

# Truthiness

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

# Locals, Blocks, and Loops

## Locals

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

## Blocks

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

## Loops

The functional approach of *ClojureScript* means that it does not have
standard, well-known, statement-based loops such as `for` in JavaScript.
The loops in *ClojureScript* are handled using recursion. Recursion
sometimes requires additional thinking about how to model your problem
in a slightly different way than imperative languages.

Many of the common patterns for which `for` is used in other languages
are achieved through higher-order functions - functions that accept
other functions as parameters.

### Looping with loop/recur

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

### Replacing for loops with higher-order functions

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

### `for` sequence comprehensions

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

# Collection types

## Immutable and persistent

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

## The sequence abstraction

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

### nil-punning

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

### Functions that work on sequences

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

### Laziness

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

## Collections in depth

Now that we’re acquainted with ClojureScript’s sequence abstraction and
some of the generic sequence manipulating functions, it’s time to dive
into the concrete collection types and the operations they support.

### Lists

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

### Vectors

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

### Maps

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

### Sets

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

### Queues

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

# Destructuring

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

# Threading Macros

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

## The thread-first macro (`->`)

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

## The thread-last macro (`->>`)

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

## The thread-as macro (`as->`)

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

## The thread-some macros (`some->` and `some->>`)

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

## The thread-cond macros (`cond->` and `cond->>`)

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

## Additional Readings

  - <http://www.spacjer.com/blog/2015/11/09/lesser-known-clojure-variants-of-threading-macro/>

  - <http://clojure.org/guides/threading_macros>

# Reader Conditionals

This language feature allows different dialects of Clojure to share
common code that is mostly platform independent but need some platform
dependent code.

To use reader conditionals, all you need is to rename your source file
with `.cljs` extension to one with `.cljc`, because reader conditionals
only work if they are placed in files with `.cljc` extension.

## Standard (`#?`)

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

## Splicing (`#?@`)

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

## More readings

  - <http://clojure.org/guides/reader_conditionals>

  - <https://danielcompton.net/2015/06/10/clojure-reader-conditionals-by-example>

  - <https://github.com/funcool/cuerdas> (example small project that
    uses reader conditionals)

# Namespaces

## Defining a namespace

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

## Loading other namespaces

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

## Namespaces and File Names

When you have a namespace like `myapp.core`, the code must be in a file
named *core.cljs* inside the *myapp* directory. So, the preceding
examples with namespaces `myapp.core` and `myapp.main` would be found in
project with a file structure like this:

    myapp
    └── src
        └── myapp
            ├── core.cljs
            └── main.cljs

# Abstractions and Polymorphism

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

## Protocols

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

### Extending existing types

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

### Participate in ClojureScript abstractions

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

### Introspection using Protocols

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

## Multimethods

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

## Hierarchies

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

### Defining a hierarchy

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

### Hierarchies and introspection

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

### Locally defined hierarchies

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

### Hierarchies in multimethods

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

# Data types

Until now, we have used maps, sets, lists, and vectors to represent our
data. And in most cases, this is a really great approach. But sometimes
we need to define our own types, and in this book we will call them
**data types**.

A data type provides the following:

  - A unique host-backed type, either named or anonymous.

  - The ability to implement protocols (inline).

  - Explicitly declared structure using fields or closures.

  - Map-like behavior (via records, see below).

## Deftype

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

## Defrecord

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

## Implementing protocols

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

## Reify

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

## Specify

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

# Host interoperability

*ClojureScript*, in the same way as its brother Clojure, is designed to
be a "guest" language. This means that the design of the language works
well on top of an existing ecosystem such as JavaScript for
*ClojureScript* and the JVM for *Clojure*.

## The types

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

## Interacting with platform types

*ClojureScript* comes with a little set of special forms that allows it
to interact with platform types such as calling object methods, creating
new instances, and accessing object properties.

### Access to the platform

*ClojureScript* has a special syntax for access to the entire platform
environment through the `js/` special namespace. This is an example of
an expression to execute JavaScript’s built-in `parseInt` function:

``` clojure
(js/parseInt "222")
;; => 222
```

### Creating new instances

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

### Invoke instance methods

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

### Access to object properties

Access to an object’s properties is really very similar to calling a
method. The difference is that instead of using the `.` you use `.-`.
Let’s see an example:

``` clojure
(.-multiline re)
;; => false
(.-PI js/Math)
;; => 3.141592653589793
```

### Property access shorthand

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

### JavaScript objects

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

### Conversions

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

### Arrays

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

# State management

We’ve learned that one of ClojureScript’s fundamental ideas is
immutability. Both scalar values and collections are immutable in
ClojureScript, except those mutable types present in the JS host like
`Date`.

Immutability has many great properties but we are sometimes faced with
the need to model values that change over time. How can we achieve this
if we can’t change data structures in place?

## Vars

Vars can be redefined at will inside a namespace but there is no way to
know **when** they change. The inability to redefine vars from other
namespaces is a bit limiting; also, if we are modifying state, we’re
probably interested in knowing when it occurs.

## Atoms

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

### Observation

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

## Volatiles

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
