---
title: Language (advanced topics)
---

# Transducers

## Data transformation

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

## Generalizing to process transformations

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

## Transducers in ClojureScript core

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

## Initialisation

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

## Stateful transducers

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

## Eductions

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

## More transducers in ClojureScript core

We learned about `map`, `filter`, `mapcat`, `take` and `partition-all`,
but there are a lot more transducers available in ClojureScript. Here is
an incomplete list of some other intersting ones:

  - `drop` is the dual of `take`, dropping up to `n` values before
    passing inputs to the reducing function

  - `distinct` only allows inputs to occur once

  - `dedupe` removes succesive duplicates in input values

We encourage you to explore ClojureScript core to see what other
transducers are out there.

## Defining our own transducers

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

## Transducible processes

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

# Transients

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

# Metadata

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

## Vars

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

## Values

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

## Syntax for metadata

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

## Functions for working with metadata

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

# Core protocols

One of the greatest qualities of the core ClojureScript functions is
that they are implemented around protocols. This makes them open to work
on any type that we extend with such protocols, be it defined by us or a
third party.

## Functions

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

## Printing

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

## Sequences

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

## Collections

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

### Collection traits

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

## Associative

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

## Comparison

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

## Metadata

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

## Interoperability with JavaScript

Since ClojureScript is hosted in a JavaScript VM we often need to
convert ClojureScript data structures to JavaScript ones and viceversa.
We also may want to make native JS types participate in an abstraction
represented by a protocol.

### Extending JavaScript types

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

### Converting data

For converting values from ClojureScript types to JavaScript ones and
viceversa we use the `clj→js` and `js→clj` functions, which are based in
the `IEncodeJS` and `IEncodeClojure` protocols respectively.

For the examples we’ll use the Set type introduced in ES6. Note that is
not available in every JS runtime.

#### From ClojureScript to JS

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

#### From JS to ClojureScript

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

## Reductions

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

## Asynchrony

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

## State

The ClojureScript state constructs such as the Atom and the Volatile
have different characteristics and semantics, and the operations on them
like `add-watch`, `reset!` or `swap!` are backed by protocols.

### Atom

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

### Volatile

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

## Mutation

In the [section about transients](#transients) we learned about the
mutable counterparts of the immutable and persistent data structures
that ClojureScript provides. These data structures are mutable, and the
operations on them end with a bang (`!`) to make that explicit. As you
may have guessed every operation on transients is based on protocols.

### From persistent to transient and viceversa

We’ve learned that we can transform a persistent data structure with the
`transient` function, which is based on the `IEditableCollection`
protocol; for transforming a transient data structure to a persistent
one we use `persistent!`, based on `ITransientCollection`.

Implementing immutable and persistent data structures and their
transient counterparts is out of the scope of this book but we recommend
taking a look at ClojureScript’s data structure implementation if you
are curious.

### Case study: the hodgepodge library

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

### Transient vectors and sets

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

# CSP (with core.async)

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

## Channels

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

### Buffers

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

#### Fixed

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

#### Dropping

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

#### Sliding

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

### Transducers

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

### Handling exceptions

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

### Offer and Poll

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

## Processes

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

### Choice

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

### Priority

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

### Defaults

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

## Combinators

Now that we’re acquainted with channels and processes it’s time to
explore some interesting combinators for working with channels that are
present in `core.async`. This section includes a brief description of
all of them together with a simple example of their usage.

### pipe

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

### pipeline-async

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

### pipeline

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

### split

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

### reduce

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

### onto-chan

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

### to-chan

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

### merge

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

## Higher-level abstractions

We’ve learned the about the low-level primitives of `core.async` and the
combinators that it offers for working with channels. `core.async` also
offers some useful, higher-level abstractions on top of channels that
can serve as building blocks for more advanced functionality.

### Mult

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

### Pub-sub

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

### Mixer

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
