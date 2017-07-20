---
layout: tour13
title: Dynamic typing and interoperation with JavaScript
tab: documentation
unique_id: docspage
author: Gavin King
doc_root: ../..
---

# #{page.title}

Interoperation with a dynamic language like JavaScript poses a 
special challenge for Ceylon. Since no typing information for 
dynamically typed values is available at compile time, the 
compiler can't validate the usual typing rules of the language. 
Therefore, Ceylon lets us write dynamically typed code where 
typechecking is performed at _runtime_.

We're not allowed to write dynamically typed code in a regular
pure-Ceylon module, so let's see how to define a module that 
interoperates with native JavaScript code.

## Defining a native JavaScript module

Before we can start writing code that interacts with 
dynamically-typed JavaScript code, we must declare a 
_native JavaScript module_ using the `native` annotation.

<!-- try: -->
    native ("js") module hello {}

Or, alternatively, we must declare a `native` function or
class within a regular cross-platform module.

<!-- try: -->
    native ("js")
    void hello() {
        dynamic {
            console.log("hello")
        }
    }

We've already seen how to define a 
[native Java module](../interop/#defining_a_native_java_module)
in the previous chapter, and the approach here is very similar.

### Tip: defining an operation with a `native` header

When writing a cross-platform module that interacts with native
Java and JavaScript code, we usually need to define `native`
functions and classes that work on both platforms. In this case,
we use a _[native header][native reference]_.

<!-- try: -->
    //native header
    native void hello();
    
    //native implementation for the JVM
    native ("jvm") void hello() {
        import java.lang { System }
        System.out.println("hello");
    }
    
    //native implementation for JavaScript
    native ("js") void hello() {
        dynamic {
            console.log("hello");
        }
    }
    
    //cross-platform function that calls
    //the native function
    shared void run() => hello();

Once we have a native header, we can safely call the `native`
functions from non-`native` cross-platform Ceylon code.

We're not going to delve further into the topic of `native`
code in cross-platform modules as part of this tour, but
you can read more [here][native reference], and find lots of 
examples in the source code of the language module and other 
Ceylon platform modules.

[`StringBuilder`][] is a great starting point.

[native reference]: /documentation/reference/interoperability/native/
[`StringBuilder`]: https://modules.ceylon-lang.org/repo/1/ceylon/language/1.3.1/module-doc/api/StringBuilder.ceylon.html

## Dynamic typing

Now that we know how to declare a `native("js")` module or 
function, we can start writing code which uses dynamic 
typing. When we talk about "dynamic typing", we're talking
about the *absence* of information&mdash;we're saying that 
we're missing information about the type of a thing at 
compile time.

### Partially typed declarations

The keyword `dynamic` may be used to declare a function or 
value with missing type information. Such a declaration is 
called _partially typed_.

<!-- try: -->
    dynamic xmlHttpRequest = ... ;

<br/>

<!-- try: -->
    void handle(dynamic event) { ... }

<br/>

<!-- try: -->
    dynamic findDomNode(String id) { ... }

Note that `dynamic` is not itself a type. Rather, it represents 
the _absence_ of typing information. Therefore any value is 
considered assignable to a `dynamic` value or returnable by a 
`dynamic` function, whatever its type, and _whether we know its
type or not_.

### Dynamically typed expressions

A _dynamically typed expression_ is an expression that involves 
references to program elements for which no typing information 
is available. That includes references to values and functions
declared `dynamic`, along with things defined in a dynamic 
language like JavaScript.

A dynamically typed expression may only occur within a `dynamic`
block. The `dynamic` block serves to suppress certain type checks
that the compiler normally performs.

<!-- try: -->
    dynamic xmlHttpRequest;
    dynamic {
        xmlHttpRequest = XMLHttpRequest();
    }

<br/>

<!-- try: -->
    void handle(dynamic event) {
        dynamic {
            print(event.info);
        }
    }

Note: you _cannot_ make use of a partially typed declaration
outside of a `dynamic` block. The following is not accepted by 
the compiler:

<!-- try: -->
    void handle(dynamic event) {
        print(event.info); //compile error: event has unknown type
    }

When a dynamically typed expression is evaluated, certain 
runtime type checks are performed, which can result in a 
runtime typing exception. For example, in the code examples
above, the compiler can't determine at compile time:

- whether there really is a function named `XMLHttpRequest`, 
  nor
- whether `event` has a member named `info`.

Therefore, the expressions `XMLHttpRequest()` and `event.info`
can, in principle, result in a runtime error when evaluated.

### Interoperating with native JavaScript

The reason Ceylon supports partially typed declarations and
dynamically typed expressions is to allow interoperation with
JavaScript objects written in JavaScript. The next example 
illustrates the use of a native JavaScript API. Try it:

    dynamic { 
        dynamic req = XMLHttpRequest();
        req.open("HEAD", "https://try.ceylon-lang.org/", true);
        req.onreadystatechange = () {
            if (req.readyState == 4) {
                String headers = req.getAllResponseHeaders();
                for (header in headers.lines) {
                    print(header.replaceFirst(": ", " = "));
                }
            }
        };
        req.send();
    }

Note that this code isn't very different in appearance or
semantics to what one would write in JavaScript itself. To
port a fragment of JavaScript code to Ceylon, often the only 
thing you need to do is replace `var` and `function` with 
`dynamic`!

### Gotcha!

A `dynamic` reference to a native JavaScript object like 
`xmlHttpRequest` or `event` lacks a known type at compile 
time. Moreover, the _actual JavaScript object itself_ lacks
a Ceylon class at *runtime*.

We can't even assign the JavaScript object to Ceylon's 
`Object` type, since it doesn't have the operations declared 
by `Object` (`string`, `equals()`, and `hash`). Nor can we 
assign it to the enumerated type `Anything`, since it's 
neither an `Object`, nor `null`.

But of course that's not true for every value that can be 
assigned to a `dynamic` reference. For example, the following
values *are* instances of Ceylon's `Object` type:

- JavaScript `String`s, `Number`s, and `Boolean`s,
- every object obtained by instantiating a Ceylon class, and,
- as we're just about to see, any native JavaScript object 
  assigned to a _dynamic interface type_.

Let's learn about dynamic interfaces.

## Dynamic interfaces

Writing dynamically-typed code is a frustrating, tedious, 
error-prone activity involving lots of debugging and lots of 
finger-typing, since the IDE can't autocomplete the names of 
members of a dynamic type, nor even show us the documentation 
of an object or member when we hover over it.

Therefore, Ceylon makes it possible to write a special sort of 
interface that captures the typing information that is missing 
from a JavaScript API. For example:

<!-- try: -->
    dynamic IXMLHttpRequest {
        shared formal void open(String method, String url, Boolean async);
        shared formal variable Anything()? onreadystatechange;
        shared formal void send();
        shared formal Integer readyState;
        shared formal String? getAllResponseHeaders();
        //TODO: more operations
    }

    IXMLHttpRequest newXMLHttpRequest() {
        dynamic { return XMLHttpRequest(); }
    }

Now we can rewrite the example above, without the use of 
`dynamic`, using regular static typing:

<!-- try-pre:
    dynamic IXMLHttpRequest {
        shared formal void open(String method, String url, Boolean async);
        shared formal variable Anything()? onreadystatechange;
        shared formal void send();
        shared formal Integer readyState;
        shared formal String? getAllResponseHeaders();
        //TODO: more operations
    }

    IXMLHttpRequest newXMLHttpRequest() {
        dynamic { return XMLHttpRequest(); }
    }
    
-->
    IXMLHttpRequest req = newXMLHttpRequest();
    req.open("HEAD", "https://try.ceylon-lang.org/", true);
    req.onreadystatechange = () {
        if (req.readyState==4) {
            print(req.getAllResponseHeaders());
        }
    };
    req.send();

Thus, it's possible to create Ceylon libraries that provide a 
typesafe view of native JavaScript APIs.

### Gotcha!

Note that a `dynamic` interface is a convenient fiction! The
Ceylon compiler can't do anything at compilation time to ensure 
that the native JavaScript object you assign to the `dynamic` 
interface type _actually implements the operations_ that the 
interface declares!

So, if you're not careful when writing your `dynamic` interface,
or when assigning a dynamically typed value to a `dynamic` 
interface type, you can _still_ get runtime type exceptions!

### Runtime type checks for assignment to dynamic interfaces

When, at runtime, a dynamically typed expression is evaluated 
and assigned to a dynamic interface type, a runtime type check
is performed to verify that either the assigned value:

- is already "known" to be an instance of the type (it has 
  previously been "tagged" as an instance of the dynamic 
  interface type), or, if not, that it
- has a member with the right name for every member of the 
  dynamic interface, and that each member has the expected 
  type.

In the second case, the value may be tagged as an instance of 
the dynamic interface type.

### Gotcha!

Note that this runtime typecheck is far from foolproof! In an 
environment as dynamic as JavaScript, there are all sorts of 
ways to defeat it. However, it's a basic sanity check that will
help you find bugs faster, and make it easier to trace them to
their root cause.

### Dynamic interfaces in `is` conditions

An `is` condition for a dynamic interface, for example,
`is IXMLHttpRequest val`, is only satisfied if the value 
`val` has previously been assigned to the dynamic interface 
type. For a native JavaScript object that has never been 
assigned to a dynamic interface type, an `is` condition is 
never satisfied.

    dynamic Window {
        shared formal void alert(String message);
    }
    
    dynamic {
        print(window is Window);
        Window w = window;
        print(window is Window);
    }

This behavior is unintuitive but reasonable.

As a special exception, an `is` condition _in an `assert` 
statement_ will first attempt to coerce a value which has 
not been tagged as an instance of any Ceylon type to the 
specified dynamic interface type by performing the runtime 
checks outlined above, and then tagging the value as an 
instance of the type. This coercion *never* occurs in `if`, 
`while`, or `switch` conditions!

## Dynamic instantiation expressions

Occasionally it's necessary to instantiate a JavaScript `Array` 
or plain JavaScript `Object` (which is not the same thing as a 
Ceylon `Object`!). We may use a special-purpose _dynamic 
enumeration expression_. This comes in two flavors:

- with named arguments, to instantiate a JavaScript `Object`, 
  or
- with positional arguments, to instantiate a JavaScript `Array`.

The example demonstrates both flavors:

    dynamic {
        dynamic obj = dynamic [ hello = "Hello, World"; count = 11; ];
        print(obj.hello);
        print(obj.count);
        print(obj["hello"]);
        
        dynamic arr = dynamic [ 12, 13, 14 ];
        print(arr[0]);
        for (n in arr) {
            print(n^2);
        }
        print(13 in arr);
        print(15 in arr);
    }

Notice how we've used:

- the lookup operator `[]` to obtain elements of the JavaScript 
  array and attributes of the JavaScript object, 
- `for` to iterate the elements of the JavaScript array, and 
- `in` to determine if a value belongs to the array.

It's even possible to use the
[spread operator](../functions/#the_spread_operator), or a 
[comprehension](../comprehensions) inside a dynamic enumeration 
expression:

    dynamic {
        dynamic oneToTen = dynamic [*(1..10)];
        dynamic letters = dynamic [for (ch in "hello") ch.uppercased];
    }

Furthermore, we can define named `function`s, `value`s and 
`object`s in a dynamic enumeration:

    dynamic {
        dynamic obj = dynamic [ 
            void greet() => print("Hello!");
            value time = system.milliseconds;
            object thing { string => "Just some object"; }
        ];
        obj.greet();
        print(obj.time);
        print(obj.thing);
    }

Thus, a dynamic enumeration expression accepts the full syntax of 
a [named argument list](../named-arguments).

### Gotcha!

A dynamic enumeration expression is _not_ considered to produce 
an instance of a Ceylon class, and the resulting value is not 
even considered an instance of Ceylon's `Object` type. This code
produces an exception at runtime:

    dynamic {
        dynamic obj = dynamic [ name = "Ceylon"; ];
        Object thing = obj;
    }

The reason for this is that the value produced by the dynamic
enumeration expression just doesn't have the operations of
`Object` (`string`, `equals()`, and `hash`).

### Tip: assigning a dynamic enumeration to a dynamic interface type

On the other hand, if you assign the value produced by a dynamic 
enumeration expression to a `dynamic` interface type, you'll get 
something that _is_ a Ceylon `Object`.

    dynamic Named {
        shared formal String name;
        shared formal void greet();
    }
    
    dynamic {
        dynamic obj = dynamic [
            name = "Ceylon"; 
            void greet() => print("Hello!");
        ];
        print(obj is Object);
        print(obj is Named);
        Named named = obj;  //assigns a Ceylon type to obj
        print(obj is Named);
        print(obj is Object);
    }
 
Run this code to see the effect of the assignment to the dynamic
interface type `Named`.

Now try removing the definition of `greet` from the dynamic 
value, leaving the following unsound code:

    dynamic Named {
        shared formal String name;
        shared formal void greet();
    }
    
    dynamic {
        dynamic obj = dynamic [
            name = "Ceylon";
            //missing definition of greet()
        ];
        Named named = obj;  //runtime error!
    }

Run this code to see it how cleanly it fails at runtime. 

## Importing npm packages containing native JavaScript code

A Ceylon module may express a dependency on a native 
JavaScript module by importing the module from npm (the node 
package manager), specifying the `npm:` repository type:

<!-- try: -->
    native ("js")
    module com.example.npm "1.0.0" {
        import npm:"left-pad" "1.1.3";
        import npm:"pixi.js" "4.5.3";
        import npm:"angular":"router" "4.3.1"; // @angular/router
    }

### Package names for imported modules
The imported npm packages are made visible using Ceylon package
names constructed from the npm package names. The Ceylon package
name for an imported module is constructed by replacing all
instances of `- _ :` in the npm package name with `.`. Therefore
the above npm packages are available using the following
packages:

* left.pad
* pixi.js
* angular.router

e.g. `import left.pad { /* ... */ }`.

### CommonJS packages

Modules that respect the CommonJS format, exporting only named
entries work as-is.

    import some.module { SomeClass, someFunction, someObject }

Since the JS compiler has no way of knowing if those are valid
declarations in the module, it will just create very simple
declarations so they're usable. Uppercased declarations result
in classes with a single variadic constructor, while lowercase
declarations result in dynamically typed objects/functions, so
they can only be used inside `dynamic` blocks. Classes declared
as coming from npm packages are instantiated using `new` in the
generated JS code.

### Non-standard packages

Npm packges that don't follow the CommonJS format and instead
export a single object/function are treated in a different way.
They are exposed as the sole item that is available for import.
The name of the item is constructed from the npm package name by
first removing the scope prefix if present and then removing all
instances of `. _ -` and uppercasing the letters that were
immediately to the right of the removed characters. Examples:
* left-pad → leftPad
* pixi.js → pixiJs
* @angular/router → router

If the exported object/function contains some members (for
example, the `express` module) then the object/function itself
is added as a member under the module's name.

<!-- try: -->
    import left.pad {
        leftPad
    }
    
    void run() {
        dynamic {
            for (i in 1..10) {
                print(leftPad("hello", i));
            }
        }
    }

# Calling npm

Both the `compile-js` and the `run-js` commands will install npm
packages if needed. A `node_modules` directory will be created
under the working directory, by simply calling the `npm`
command, which must be on your executable path.

# Publishing your Ceylon modules to npm

If you wish to export your own Ceylon module to npm, you can
specify the npm package name explicitly in the module
descriptor:

<!-- try: -->
    native ("js")
    module com.example.npm          //Ceylon module name
            npm:"ceylon-example"    //npm package name
            "1.0.0" {               //module version
        import npm:"left-pad" "1.1.3";
    }

## There's more ...

Well, no, actually, we've finished the tour! Of course, there's 
still plenty of scope for you to explore Ceylon on your own. 
You should now know enough to start writing Ceylon code for 
yourself, and start getting to know the platform modules.

Alternatively, if you want to keep reading you can browse the 
[reference documentation](#{page.doc_root}/reference) or (if 
you're sitting comfortably) read the 
[specification](#{site.urls.spec_current}).
