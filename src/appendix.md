---
title: Appendix
---

# Appendix A: Interactive development with Figwheel

## Introduction

In this project, we will **not** do “Hello World”— that has been done to
death. Instead, this project will be a web page that asks you for your
age in years and tells you how many days that is, using an approximation
of 365 days per year.

For this project, we will use the *figwheel* leiningen plugin. This
plugin creates a fully interactive, REPL-based, autoreloading
environment.

## First steps

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

![Screenshot of web page and console](localhost1.png)

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

## Interacting with JavaScript

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

# Appendix B: Setting up a ClojureScript development environment

## Cursive

TODO

## Emacs

TODO

## Vim

TODO
