# Getting Started with the Compiler

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

## Execution environments

What is an execution environment? An execution environment is an engine
where JavaScript can be executed. For example, the most popular
execution environment is a browser (Chrome, Firefox, …​) followed by the
second most popular - [nodejs](https://nodejs.org/).

There are others, such as Rhino (JDK 6+), Nashorn (JDK 8), QtQuick
(QT),…​ but none of them have significant differences from the first
two. So, *ClojureScript* at the moment may compile code to run in the
browser or in nodejs-like environments out of the box.

## Download the compiler

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

## Compile for Node.js

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

### Create the example application

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

### Compile the example application

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

## Compile for the Browser

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

### Compile the example application

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

## Watch process

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

## Optimization levels

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

### none

This optimization level causes the generated JavaScript to be written
into separate output files for each namespace, without any additional
transformations to the code.

### whitespace

This optimization level causes the generated JavaScript files to be
concatenated into a single output file, in dependency order. Line breaks
and other whitespace are removed.

This reduces compilation speed somewhat, resulting in a slower
compilations. However, it is not terribly slow and it is quite usable
for small-to-medium sized applications.

### simple

The simple compilation level builds on the work from the `whitespace`
optimization level, and additionally performs optimizations within
expressions and functions, such as renaming local variables and function
parameters to have shorter names.

Compilation with the `:simple` optimization always preserves the
functionality of syntactically valid JavaScript, so it does not
interfere with the interaction between the compiled *ClojureScript* and
other JavaScript.

### advanced

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

# Working with the REPL

## Introduction

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

## Nashorn REPL

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

## Node.js REPL

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

## Browser REPL

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

# The Closure Library

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

# Dependency management

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

## Installing leiningen

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

## First project

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

## Managing dependencies

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

# External dependencies

In some circumstances you may found yourself that you need some library
but that does not exists in *ClojureScript* but it is already
implemented in javascript and you want to use it on your project.

There are many ways that you can do it mainly depending on the library
that you want to include. Let see some ways.

## Closure Module compatible library

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

## CommonJS modules compatible libraries

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

## Legacy, module-less (global scope) libraries

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

# Unit testing

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

## First steps

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

## Async Testing

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

## Fixtures

TBD

## Integrating with CI

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
