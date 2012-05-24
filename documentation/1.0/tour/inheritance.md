---
layout: tour
title: Tour of Ceylon&#58; Inheritance and Refinement
tab: documentation
unique_id: docspage
author: Emmanuel Bernard
doc_root: ../..
---

# #{page.title}

This is the third leg of the Tour of Ceylon. In the [previous leg](../classes)
you learned about classes. In this leg you're going to learn about 
*inheritance* and *refinement* (known as "overriding" in many other languages).

## Inheritance and refinement

In object-oriented programming, we often replace conditionals (`if`, and 
especially `switch`) with subtyping. Indeed, according to some folks, this is 
what makes a program object-oriented. Let's try refactoring the `Polar` class 
[from the previous leg of the tour](../classes) into two classes, with two 
different implementations of `description`. Here's the superclass:

<!-- id:polar -->
    doc "A polar coordinate"
    class Polar(Float angle, Float radius) {
 
        shared Polar rotate(Float rotation) {
            return Polar(angle+rotation, radius);
        }
     
        shared Polar dilate(Float dilation) {
            return Polar(angle, radius*dilation);
        }
        
        doc "The default description"
        shared default String description = "(" radius "," angle ")";
    
    }

Notice that Ceylon forces us to declare attributes or methods that can be 
refined (overridden) by annotating them `default`.

Subclasses specify their superclass using the `extends` keyword
([here's why](#{page.doc_root}/faq/language-design/#colon_vs_extends_in_class_definition)), 
followed by the name of the superclass, followed by a list of arguments to be 
sent to the superclass initializer parameters. It looks just like an expression 
that instantiates the superclass:

<!-- cat-id:polar -->
    doc "A polar coordinate with a label"
    class LabeledPolar(Float angle, Float radius, String label)
            extends Polar(angle, radius) {
         
        doc "The labeled description"
        shared actual String description {
            return label + " " + super.description;
        }
     
    }

Ceylon also forces us to declare that an attribute or method refines 
(overrides) an attribute or method of a superclass by annotating it `actual`
([not "overrides" like Java](#{page.doc_root}/faq/language-design/#_override_vs_actual)). 
All this annotating stuff costs a few extra keystrokes, but it helps the 
compiler detect errors. We can't inadvertently refine a member or the 
superclass, or inadvertently fail to refine it.

Notice that Ceylon goes out of its way to repudiate the idea of "duck" typing 
or structural typing. If it `walks()` like a `Duck`, then it should be a 
subtype of `Duck` and must explicitly refine the definition of `walk()` 
in `Duck`. We don't believe that the name of a method or attribute alone is 
sufficient to identify its semantics.

## Refining the members of `IdentifiableObject`

Our `Polar` class is an implicit subtype of the class 
[`IdentifiableObject`](#{site.urls.apidoc_current}/ceylon/language/class_IdentifiableObject.html)
in the package `ceylon.language`. 
If you take a look at this class, you'll see 
that it has a `default` attribute named 
[`string`](#{site.urls.apidoc_current}/ceylon/language/class_IdentifiableObject.html#string). 
It's common to refine this
attribute to provide a developer-friendly representation of the object.
`IdentifiableObject` also defines default implementations of 
[`equals()`](#{site.urls.apidoc_current}/ceylon/language/class_IdentifiableObject.html#equals)  and
[`hash`](#{site.urls.apidoc_current}/ceylon/language/class_IdentifiableObject.html#hash).
We should _definitely_ refine those:

<!-- cat: Float pi = 3.1415926535; -->
    doc "A polar coordinate"
    class Polar(Float angle, Float radius, String description) {
        
        // ...

        shared actual String string { return description; }
        
        value azimuth {
            return pi*(angle/pi).fractionalPart;
        }
        
        shared actual Boolean equals(Object that) {
            if (is Polar that) {
                return azimuth==that.azimuth && 
                       radius==that.radius; 
            }
            else {
                return false;
            }
        }
        
        shared actual Integer hash { return radius.hash; }
        
    }

Don't worry if the syntax `if (is Polar that)` throws you. We'll come back to
this construct this later in the tour.


## Abstract classes

Now let's consider a much more interesting problem: abstracting over the polar
and cartesian coordinate systems. Since a cartesian coordinate isn't just a 
special kind of polar coordinate, this is a case for introduction of an abstract 
superclass:

<!-- cat-id:polar -->
<!-- cat: class Cartesian(Float x, Float y) {} -->
    doc "A coordinate-system free abstraction 
         of a geometric point"
    abstract class Point() {
         
         shared formal Polar polar;
         shared formal Cartesian cartesian;
         
         shared formal Point rotate(Float rotation);
         shared formal Point dilate(Float dilation);
         
    }

Ceylon requires us to annotate abstract classes `abstract`, just like Java. 
This annotation specifies that a class cannot be instantiated, and can define 
abstract members. Like Java, Ceylon also requires us to annotate "abstract" 
members that don't specify an implementation. However, in this case, the 
required annotation is `formal`. The reason for having two different 
annotations, as we'll see [later](../introduction#its_even_possible_to_define...), 
is that nested classes may be either 
`abstract` or `formal`, and `abstract` nested classes are slightly different 
to `formal` member classes. A `formal` member class may be instantiated; 
an abstract class may not be.

Note that an attribute that is never initialized is always a `formal` 
attribute. Ceylon doesn't initialize attributes to zero or `null` unless you 
explicitly tell it to!

One way to define an implementation for an inherited abstract attribute is to 
simply *assign* a value to it in the subclass.

<!-- check:parse:Requires ceylon.math -->
    doc "A polar coordinate"
    class Polar(Float angle, Float radius) 
            extends Point() {
        
        polar = this;
        cartesian = Cartesian(radius*cos(angle), radius*sin(angle));
        
        ...
        
    }

Alternatively, we can also define an implementation for an inherited abstract 
attribute by *refining* it.

<!-- check:parse:Requires ceylon.math -->
    doc "A polar coordinate"
    class Polar(Float angle, Float radius) 
            extends Point() {
        
        shared actual Polar polar { return this; }
        
        shared actual Cartesian cartesian {
            return Cartesian(radius*cos(angle), radius*sin(angle));
        }
        
        shared actual Polar rotate(Float rotation) {
            return Polar(angle+rotation, radius);
        }
        
        shared actual Polar dilate(Float dilation) {
            return Polar(angle, radius*dilation);
        }
           
    }

Note that there's no way to prevent other code from extending a class in 
Ceylon (like a `final` class in Java). Since only members explicitly declared as supporting refinement using 
either `formal` or `default` can be refined, a subtype can never break the
implementation of a supertype. Unless the supertype was explicitly designed 
to be extended, a subtype can add members, but never change the behavior of
inherited members.

Oh, I suppose you would like to see `Cartesian`...

<!-- check:parse:Requires ceylon.math -->
    doc "A cartesian coordinate"
    class Cartesian(Float x, Float y) 
            extends Point() {
        
        shared actual Polar polar { 
            return Polar( (x**2+y**2)**0.5, arctan(y/x) ); 
        }
        
        shared actual Cartesian cartesian {
            return this;
        }
        
        shared actual Cartesian rotate(Float rotation) {
            return polar.rotate(rotation).cartesian;
        }
        
        shared actual Cartesian dilate(Float dilation) {
            return Cartesian(x*dilation, y*dilation);
        }
                
    }


## Interfaces and "mixin" inheritance

From time to time we come across a case where a class needs to inherit
functionality from more than one supertype. Java's inheritance model doesn't 
support this, since an interface can never define a member with a concrete 
implementation. Interfaces in Ceylon are a little more flexible:

* An interface may define concrete methods, attribute getters, and attribute setters.
* It may not define simple attributes or initialization logic.

Notice that prohibiting simple attributes and initialization logic makes 
interfaces completely stateless. An interface can't hold references to other 
objects.

Let's take advantage of mixin inheritance to define a reusable `Writer` 
interface for Ceylon.

<!-- check:none:concrete members of interfaces not yet supported -->
<!-- id:writer -->
<!-- cat: 
    shared interface Formatter { 
        shared formal String format(String formatString, Object... args);
    } -->
    shared interface Writer {
     
        shared formal Formatter formatter;
         
        shared formal void write(String string);
         
        shared void writeLine(String string) {
            write(string);
            write("\n");
        }
         
        shared void writeFormattedLine(String formatString, Object... args) {
            writeLine( formatter.format(formatString, args) );
        }
         
    }

Note that we can't define a concrete value for the `formatter` attribute, 
since an interface may not define a simple attribute, and may not hold a 
reference to another object.

Now let's define a concrete implementation of this interface.

<!-- check:none:depends on above:concrete members of interfaces not yet supported -->
<!-- cat-id: writer -->
    shared class ConsoleWriter()
            satisfies Writer {
         
        formatter = StringFormatter();
         
        shared actual void write(String string) {
            print(string);
        }
         
    }

The `satisfies` keyword 
([not `implements` like Java](#{page.doc_root}/faq/language-design/#_implements_vs_satisfies))
is used to specify that an interface extends 
another interface or that a class implements an interface. 
Unlike an `extends` declaration, a `satisfies` declaration does not specify 
arguments, since interfaces do not have parameters or initialization logic. 
Furthermore, the `satisfies` declaration can specify more than one interface.

Ceylon's approach to interfaces eliminates a common pattern in Java 
where a separate abstract class defines a default implementation of some 
of the members of an interface. In Ceylon, the default implementations can 
be specified by the interface itself. Even better, it's possible to add a 
new member to an interface without breaking existing implementations of the 
interface.


## Ambiguities in mixin inheritance

It's illegal for a type to inherit two members with the same name, unless the 
two members both (directly or indirectly) refine a common member of a common 
supertype, and the inheriting type itself also refines the member to eliminate 
any ambiguity. The following results in a compilation error:

<!-- check:none:demos error -->
    interface Party {
        shared formal String legalName;
        shared default String name {
            return legalName;
        }
    }
     
    interface User {
        shared formal String userId;
        shared default String name {
            return userId;
        }
    }
     
    class Customer(String name, String email)
            satisfies User & Party {
        shared actual String legalName = name;
        shared actual String userId = email;
        shared actual String name = name;    //error: refines two different members
    }

To fix this code, we'll factor out a `formal` declaration of the attribute 
`name` to a common supertype. The following is legal:

<!-- check:none:concrete members of interfaces not yet supported -->
    interface Named {
        shared formal String name;
    }
     
    interface Party satisfies Named {
        shared formal String legalName;
        shared actual default String name {
            return legalName;
        }
    }
     
    interface User satisfies Named {
        shared formal String userId;
        shared actual default String name {
            return userId;
        }
    }
     
    class Customer(String name, String email)
            satisfies User & Party {
        shared actual String legalName = name;
        shared actual String userId = email;
        shared actual String name = name;
    }

Oh, of course, the following is illegal:

<!-- check:none:demos error -->
    interface Named {
        shared formal String name;
    }
     
    interface Party satisfies Named {
        shared formal String legalName;
        shared actual String name {
            return legalName;
        }
    }
     
    interface User satisfies Named {
        shared formal String userId;
        shared actual String name {
            return userId;
        }
    }
     
    class Customer(String name, String email)
            satisfies User & Party {    //error: inherits multiple definitions of name
        shared actual String legalName = name;
        shared actual String userId = email;
    }

To fix this code, `name` must be declared `default` in both `User` and `Party` 
and explicitly refined in `Customer`.


## There's more...

Next, to finish off our discussion of object oriented programming
in Ceylon, we're going to learn about 
[anonymous classes and member classes](../anonymous-member-classes).

 
