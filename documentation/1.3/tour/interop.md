---
layout: tour13
title: Interoperation with Java
tab: documentation
unique_id: docspage
author: Gavin King
doc_root: ../..
---

# #{page.title}

Ceylon is designed to execute in a virtual machine environment 
(loosely speaking, in an environment with built-in garbage
collection), but it doesn't have its own native virtual machine.
Instead, it "borrows" a virtual machine designed for some other
language. For now the options are:

- the Java Virtual Machine, or 
- a JavaScript virtual machine. 

In this chapter, we're going to learn about how to interoperate 
with native code running on the Java Virtual Machine. In the
[next chapter](../dynamic) we'll discuss JavaScript.

## Defining a native Java module

Many Ceylon modules that call native Java code are designed
to execute only on the JVM. In this case, we declare the whole
Ceylon module as a _native JVM module_ using the [`native`][]
annotation.

<!-- try: -->
    native ("jvm")
    module ceylon.formatter "1.3.1" {
        shared import java.base "7";
        shared import com.redhat.ceylon.typechecker "1.3.1";
        import ceylon.collection "1.3.1";
        ...
    }

A cross-platform module may call native Java code too, but in
this case we need to apply the `native` import to `import`
statements that declare dependencies on native Java `jar`
archives.

<!-- try: -->
    module ceylon.formatter "1.3.1" {
        native ("jvm") shared import java.base "7";
        shared ("jvm") import com.redhat.ceylon.typechecker "1.3.1";
        import ceylon.collection "1.3.1";
        ...
    }

Furthermore, in this case, we must annotate declarations which 
make use of the Java classes in these archives.

<!-- try: -->
    import java.lang { System }
    
    native ("jvm")
    void hello() => System.out.println("Hello, world!");

It's *not* necessary to annotate individual declarations in a
module that is already declared as a native JVM module.

[`native`]: /documentation/current/reference/annotation/native/

## Importing Java modules containing native code

A Ceylon program can only use native code belonging to a module.
Ceylon doesn't have the notion of a global class path, and not
every Ceylon program can be assumed to be executing on the JVM, 
so dependencies to native Java code must always be expressed 
explicitly in the module descriptor.

The native Java modules your Ceylon program depends on might 
come from a Ceylon module repository, from a Maven repository,
or they might be part of the Java or Android SDK.

### Depending on the Java SDK

In particular, the Java SE SDK is available as a set of modules, 
according to the modularization proposed by the [Jigsaw][] project.
So to make use of the Java SE SDK we need to import one or more
of these modules. For example:

- the module `java.base` contains core packages including
  `java.lang`, `java.util`, and `javax.security`,
- the module `java.desktop` contains the AWT and Swing desktop
  UI frameworks, and
- `java.jdbc` contains the JDBC API.

So, if we need to use the Java collections framework in our
Ceylon program, we need to create a Ceylon module that depends
on `java.base`.

<!-- try: -->
    native ("jvm")
    module org.jboss.example "1.0.0" {
        import java.base "7";
    }

Now, we can simply import the Java class we're interested in
and use it like any ordinary Ceylon class:

<!-- try: -->
    import java.util { HashMap }
    
    void hashyFun() {
        value hashMap = HashMap<String,Object>();
    }

_TODO: instructions for using JavaFX here._

### Depending on the Android SDK

For Android development, it's necessary to use the 
[Ceylon Android plugin for Gradle][android gradle] to create 
a Ceylon module repository containing the bits of the Android 
SDK that are needed to compile an app written in Ceylon.
This repository will be created in the subdirectory
`build/intermediates/ceylon-android/repository` of your
Ceylon Android project. This repository must be specified as 
the source of all Android-related Java modules using `--rep`, 
and as the provider of the Java SDK using the command-line 
argument [`--jdk-provider`][] of `ceylon compile`.

However, when compiling using Gradle, or from inside Android 
Studio, the repository is created by the build, and the 
compiler option is set automatically, so you don't need to 
mess with it explicitly. You can find it in `.ceylon/config`,
if you're interested:

<!-- lang: none -->
    [compiler]
    jdkprovider=android/24

    [repositories]
    lookup=./build/intermediates/ceylon-android/repository

_Note: Android itself has an immensely complicated toolchain, 
and Android apps cannot reasonably be compiled using only 
commmand line tools. In practice, you'll always use Gradle to
build an Android app._

A basic boilerplate module descriptor for Android development 
looks like this:

<!-- try: -->
    native ("jvm")
    module com.acme.example.androidapp "1.0" {
        import java.base "7";
        import android "24";
        import "com.android.support.appcompat-v7" "24.2.1";
        import "com.android.support.design" "24.2.1";
    }

You can get started with Ceylon on Android by following this 
[getting started guide][android blog].

[`--jdk-provider`]: /documentation/reference/tool/ceylon/subcommands/ceylon-compile.html#option--jdk-provider
[android blog]: /blog/2016/06/02/ceylon-on-android/
[android gradle]: https://github.com/ceylon/ceylon-gradle-android

### Depending on a Java archive

To make use of native code belonging to a packaged `.jar`
archive, you have two options:

- add the archive to a Ceylon module repository, along with
  JBoss Modules metadata defined in a [`module.xml`][] or 
  [`module.properties`][] file, or
- import the archive directly from a 
  [legacy Maven repository](../../reference/repository/maven).

To add a Java `.jar` to a Ceylon module repository, you need to
provide some metadata describing its dependencies. A `module.xml` 
or `module.properties` file specifies dependency information for 
a `.jar`.

- The format of the Ceylon `module.properties` file is documented
  [here][`module.properties`], and
- the JBoss Modules `module.xml` descriptor format is defined 
  [here][`module.xml`].

The command line tool [`ceylon import-jar`][] can help make this 
task easier.

If you're using Ceylon IDE for Eclipse, and you don't want to 
write the `module.xml` descriptor by hand, go to 
`File > Export ... > Ceylon > Java Archive to Module Repository`.

Alternatively, the Ceylon module architecture interoperates with 
Maven via Aether. You can import a module from maven by specifying
the `maven:` repository type:

<!-- try: -->
    import maven:"org.hibernate:hibernate-core" "5.0.4.Final";

You can find more information [here](../../reference/repository/maven).

[`ceylon import-jar`]: /documentation/current/reference/tool/ceylon/subcommands/ceylon-import-jar.html
[`module.properties`]: /documentation/current/reference/structure/module-properties/
[`module.xml`]: https://docs.jboss.org/author/display/MODULES/Module+descriptors

## Interoperation with Java types

Calling and extending Java types from Ceylon is mostly 
completely transparent. You don't need to do anything "special"
to use or extend a Java class or interface in Ceylon, or to
call its methods.

There's a handful of things to be aware of when writing Ceylon 
code that calls a Java class or interface, arising out of the
differences between the type systems of the two languages.

### Certain very abstract Java supertypes are mapped to Ceylon types

There are three Java classes and one interface which simply 
can't be used at all in Ceylon code, because Ceylon provides
types which are exactly equivalent in the package 
[`ceylon.language`][]. When these types occur in the signature
of an operation in Java, they're always represented by the
equivalent Ceylon type:

- `java.lang.Object` is represented by Ceylon's 
  [`Object`][] class,
- `java.lang.Exception` is represented by Ceylon's 
  [`Exception`][] class,
- `java.lang.Throwable` is represented by Ceylon's 
  [`Throwable`][] class, and
- `java.lang.annotation.Annotation` is represented by the
  interface [`Annotation`][].

It's an error to attempt to `import` one of these Java types
in Ceylon code. 

[`ceylon.language`]: #{site.urls.apidoc_1_3}/index.html
[`Object`]: #{site.urls.apidoc_1_3}/Object.type.html
[`Exception`]: #{site.urls.apidoc_1_3}/Exception.type.html
[`Throwable`]: #{site.urls.apidoc_1_3}/Throwable.type.html
[`Annotation`]: #{site.urls.apidoc_1_3}/Annotation.type.html

### Java primitive types are mapped to Ceylon types

You're never exposed to Java primitive types when calling a
Java method or field from Ceylon. Instead:

- `boolean` is represented by Ceylon's [`Boolean`][] class,
- `char` is represented by Ceylon's [`Character`][] class,
- `long`, `int`, and `short` are represented by Ceylon's 
  [`Integer`][] class,
- `byte` is represented by Ceylon's [`Byte`][] class,
- `double` and `float` are represented by Ceylon's [`Float`][]
  class, and
- `java.lang.String` is represented by Ceylon's [`String`][] 
  class.

Almost all of the time, this behavior is completely intuitive.
But there's two wrinkles to be aware of...

[`Boolean`]: #{site.urls.apidoc_1_3}/Boolean.type.html
[`Character`]: #{site.urls.apidoc_1_3}/Character.type.html
[`Integer`]: #{site.urls.apidoc_1_3}/Integer.type.html
[`Byte`]: #{site.urls.apidoc_1_3}/Byte.type.html
[`Float`]: #{site.urls.apidoc_1_3}/Float.type.html
[`String`]: #{site.urls.apidoc_1_3}/String.type.html

### Gotcha!

According to these rules, all conversions from a Java primitive 
to a Ceylon type are _widening_ conversions, and are guaranteed 
to succeed at runtime. However, conversion from a Ceylon type 
to a Java primitive type might involve an implicit _narrowing_ 
conversion. For example, if:

- a Ceylon `Integer` is assigned to a Java `int` or `short`,
- a Ceylon `Float` is assigned to a Java `float`, or if
- a Ceylon UTF-32 `Character` is assigned to a Java 16-bit
  `char`,

the assignment can result in _silent_ overflow or loss of
precision at runtime.

_Note: it is not a goal of Ceylon's type system to warn about
operations which might result in numeric overflow. In general,
almost _any_ operation on a numeric type, including `+` or `*`, 
can result in numeric overflow._ 

### Gotcha again!

There's no mapping between Java's *wrapper* classes like 
`java.lang.Integer` or `java.lang.Boolean` and Ceylon basic 
types, so these conversions must be performed explicitly by
calling, for example, `longValue()` or `booleanValue()`, or
by explicitly instantiating the wrapper class, just like you
would do in Java when converting between a Java primitive
type and its wrapper class.

This is mainly important when working with Java collection
types. For example, a Java `List<Integer>` doesn't contain
_Ceylon_ `Integer`s.

<!-- try: -->
    import java.lang { JInteger=Integer }
    import java.util { JList=List } 
    
    JList<JInteger> integers = ... ;
    for (integer in integers) { //integer is a JInteger!
        Integer int = integer.longValue(); //convert to Ceylon Integer
        ...
    }

(This isn't really much worse than Java, by the way: a Java
 `List<Integer>` doesn't contain Java `int`s either!)

Worse, a Java `List<String>` doesn't contain _Ceylon_
`String`s.

<!-- try: -->
    import java.lang { JString=String }
    import java.util { JList=List } 
    
    JList<JString> strings = ... ;
    for (string in strings) { //string is a JString!
        String str = string.string; //convert to Ceylon String
        ...
    }

Watch out for this!

### Tip: converting Java strings

Explicitly converting between [`String`][] and Java's 
`java.lang.String` is easy:

- the `.string` attribute of a Java string returns a Ceylon
  string, and
- one of the constructors of `java.lang.String` accepts a
  Ceylon `String`, or, alternatively,
- the function [`javaString`][] in the module 
  [`ceylon.interop.java`][] converts a Ceylon string to a 
  Java string without requiring an object instantiation.

[`javaString`]: #{site.urls.apidoc_current_interop_java}/index.html#javaString

### Tip: converting Java primitive wrapper types

Likewise, conversions between Ceylon types and Java primitive
wrapper types are just as trivial, for example:

- the `.longValue()` methods of `java.lang.Long` and
  `java.lang.Integer` return a Ceylon `Integer`, and
- the constructors of `java.lang.Integer` and `java.lang.Long`
  accept a Ceylon `Integer`.

Likewise:

- the `.doubleValue()` methods of `java.lang.Double` and
  `java.lang.Float` return a Ceylon `Float`, and
- the constructors of `java.lang.Double` and `java.lang.Float`
  accept a Ceylon `Float`.

The story is similar for other primitive wrapper types.

### Tip: using the `small` annotation

If, for some strange reason, you _really_ need a 32-bit `int` 
or `float` at the bytecode level, instead of the 64-bit `long`
or `double` that the Ceylon compiler uses by default, you can
use the `small` annotation.

<!-- try: -->
    small Integer int = string.hash;

It's important to understand that `small Integer` isn't a 
different type to `Integer`. So any `Integer` is directly
assignable to a declaration of type `small Integer`, and
the compiler will silently produce a narrowing conversion,
which could result in silent overflow or loss of precision
at runtime.

Note also that `small` is defined as a _hint_, and may be 
completely ignored by the compiler.

### Java array types are represented by special Ceylon classes

Since there are no primitively-defined array types in Ceylon, 
arrays are represented by special classes. These classes are
considered to belong to the package `java.lang`. (Which belongs
to the module `java.base`.) So these classes must be explicitly
imported!

- `boolean[]` is represented by the class `BooleanArray`,
- `char[]` is represented by the class `CharArray`,
- `long[]` is represented by the class `LongArray`,
- `int[]` is represented by the class `IntArray`,
- `short[]` is represented by the class `ShortArray`,
- `byte[]` is represented by the class `ByteArray`,
- `double[]` is represented by the class `DoubleArray`,
- `float[]` is represented by the class `FloatArray`, and, 
  finally,
- `T[]` for any object type `T` is represented by the class 
  `ObjectArray<T>`.

We can obtain a Ceylon [`Array`][] without losing the identity 
of the underlying Java array.

<!-- try: -->
    import java.lang { ByteArray }
    
    ByteArray javaByteArray = ByteArray(10);
    Array<Byte> byteArray = javaByteArray.byteArray;

You can think of the `ByteArray` as the actual underlying
`byte[]` instance, and the `Array<Byte>` as an instance of 
the Ceylon class `Array` that wraps the `byte[]` instance.

The module [`ceylon.interop.java`][] contains a raft of 
additional methods for working with these Java array types.

[`Array`]: #{site.urls.apidoc_1_3}/Array.type.html
[`ceylon.interop.java`]: #{site.urls.apidoc_current_interop_java}/index.html

### Java generic types are represented without change

A generic instantiation of a Java type like `java.util.List<E>` 
is representable without any special mapping rules. The Java 
type `List<String>` may be written as `List<String>` in Ceylon, 
after importing `List` and `String` from the module `java.base`.

<!-- try: -->
    import java.lang { String }
    import java.util { List, ArrayList }
    
    List<String> strings = ArrayList<String>();

The only problem here is that it's really easy to mix up
Java's `String`, `List`, and `ArrayList` with the Ceylon 
types with the same names!

### Tip: disambiguating Java types using aliases

It's usually a good idea to distinguish a Java type with the 
same name as a common Ceylon type using an `import` alias. So 
we would rewrite the code fragment above like this:

<!-- try: -->
    import java.lang { JString = String }
    import java.util { JList = List, JArrayList = ArrayList }
    
    JList<JString> strings = JArrayList<JString>();

That's much less likely to cause confusion!

### Wildcards and raw types are represented using use-site variance

In pure Ceylon code, we almost always use 
[declaration-site variance](../generics/#covariance_and_contravariance).
However, this approach doesn't work when we interoperate with 
Java generic types, which are by nature all invariant. In Java, 
covariance or contravariance is represented at the point of use 
of the generic type, using _wildcards_. Therefore, Ceylon 
also supports use-site variance (wildcards).

Use-site variance is indicated using the keywords `out` and `in`:

- `List<out Object>` has a covariant wildcard, and is 
  equivalent to `List<? extends Object>` in Java, and
- `Topic<in Object>` has a contravariant wildcard, and is 
  equivalent to `Topic<? super Object>` in Java.

Java raw types are also represented with a covariant wildcard. 
The raw type `List` is represented as `List<out Object>` in 
Ceylon.

Note that for any invariant generic type `T<X>`, the 
instantiations `T<Anything>` and `T<Nothing>` are exactly
equivalent, and are supertypes of all other instantiations
of `T`.

Wildcard types are unavoidable when interoperating with Java, 
and perhaps occasionally useful in pure Ceylon code. But we 
recommend avoiding them, except where there's a really good 
reason.

### Gotcha!

Instances of Java generic types don't carry information about
generic type arguments at runtime. So certain operations which
are legal in Ceylon are rejected by the Ceylon compiler when
a Java generic type is involved. For example, the following code
is illegal, since `JList` is a Java generic type, with erased
type arguments:

<!-- try: -->
    import java.lang { JString = String }
    import java.util { JList = List, JArrayList = ArrayList }
    
    Object something = JArrayList<JString>();
    if (is JList<JString> something) { //error: type condition cannot be fully checked at runtime
        ... 
    }

This isn't a limitation of Ceylon; the equivalent code is also
illegal in Java!

### Tip: dealing with Java type erasure

When you encounter the need to narrow to an instantiation of a
Java generic type, you can use an _unchecked_ type assertion:

<!-- try: -->
    import java.lang { JString = String }
    import java.util { JList = List, JArrayList = ArrayList }
    
    Object something = JArrayList<JString>();
    if (is JList<out Anything> something) { //ok
        assert (is JList<JString> something); //warning: type condition might not be fully checked at runtime
        ...
    }

This code is essentially the same as what you would do in Java:

1. test against the raw type `List` using `instanceof`, and then
2. narrow to `List<String>` using an unchecked typecast.

Just like in Java, the above code produces a warning, since the
type arguments in the assertion cannot be checked at runtime.

### Null values are checked at runtime

Java types offer no information about whether a field or method
can produce a null value, except in the very special case of a
primitive type. Therefore, the compiler inserts runtime null 
value checks wherever Ceylon code calls a Java function that 
returns an object type, or evaluates a Java field of object 
type, and assigns the result to an non-optional Ceylon type.

In this example, no runtime null value check is performed, since
the return value of `System.getProperty()` is assigned to an 
optional type:
    
<!-- try: -->
    import java.lang { System }
    
    void printUserHome() {
        String? home  //optional type
                = System.getProperty("user.home");
        print(home);
    }

In this example, however, a runtime type check occurs when
the return value of `System.getProperty()` is assigned to the
non-optional type `String`:

<!-- try: -->
    import java.lang { System }
    
    void printUserHome() {
        String home  //non-optional type, possible runtime exception
                = System.getProperty("user.home");
        print(home);
    }

The runtime check ensures that `null` can never unsoundly 
propagate from native Java code with unchecked null values 
into Ceylon code with checked null values, resulting in an 
eventual `NullPointerException` in Ceylon code far from the
original call to Java.

### Gotcha!

The Ceylon compiler doesn't usually have any information that 
a Java method returning a class or interface type could return 
`null`, and so it won't warn you at compile time if you call a 
Java method that sometimes returns `null`.

You need to be especially careful with inferred types when 
calling native Java methods. In this code, the type `String`
is inferred for `home`:

<!-- try: -->
    import java.lang { System }
    
    void printUserHome() {
        value home  //inferred type String, not String?
                = System.getProperty("user.home");
        print(home);
    }

It's a good idea to be explicit here.

### Java types annotated `@Nullable` are exposed as optional types

There are now a number of Java frameworks that provide 
annotations (`@Nullable` and `@Nonnull`/`@NonNull`/`@NotNull`) 
for indicating whether a Java type can contain `null` values. 
The Ceylon compiler understands these annotations.

So, as an exception to the above gotcha, when one of these
annotations is present, Java null values _are_ checked at
compile time.

### Java properties are exposed as Ceylon attributes

A getter or getter/setter pair belonging to a Java class will 
appear to a Ceylon program as an 
[attribute](../classes/#abstracting_state_using_attributes). 
For example:

<!-- try: -->
    import java.util { Calendar, TimeZone } 

    void calendaryFun() {
        Calendar calendar = Calendar.instance;
        TimeZone timeZone = calendar.timeZone;
        Integer timeInMillis = calendar.timeInMillis;
    }

If you want to call the Java setter method, assign a value to
the attribute using `=`, the assignment operator:

<!-- try: -->
    calendar.timeInMillis = system.milliseconds;

However, if there is no exactly-matching getter method for a 
setter, the setter is treated as a regular method.

### Gotcha!

Note that there are certain corner cases here which might be
confusing. For example, consider this Java class:

<!-- lang: java -->
    public class Foo {
        public String getBar() { ... }
        public void setBar(String bar) { ... }
        public void setBar(String bar, String baz) { ... }
    }

From Ceylon, this will appear as if it were defined like this:

<!-- try: -->
    shared class Foo {
        shared String bar { ... }
        assign bar { ... }
        shared void setBar(String bar, String baz) { ... }
    }

Therefore: 

- to call the single-argument setter, use an assignment
  statement like `foo.bar = bar`, but
- to call the two-argument setter, use a method call like
  `foo.setBar(bar,baz)`.

### Java constants and enum values

Java constants like `Integer.MAX_VALUE` and Java enum values, 
like `RetentionPolicy.RUNTIME`, follow an all-uppercase naming 
convention. Since this looks rather alien in Ceylon code, it's
acceptable to refer to them using camel case. This:

<!-- try: -->
    Integer maxInteger = JInteger.maxValue;
    RetentionPolicy policy = RetentionPolicy.runtime;

is preferred to this:

<!-- try: -->
    Integer maxInteger = JInteger.\iMAX_VALUE;
    RetentionPolicy policy = RetentionPolicy.\iRUNTIME;

However, both options are accepted by the compiler.

Java enum types are treated as enumerated classes with no 
visible constructor. Each enumerated value of the enum type 
is treated as a static member `object` belonging to the enum 
type. Thus, it's possible to `switch` over the members of
a Java `enum`, just like you can in Java.

### Methods accepting a SAM interface

_Warning: this subsection describes new pre-release 
functionality that will be made available in Ceylon 1.3.1._

Java has no true function types, so there's no equivalent to
Ceylon's `Callable` interface in Java. Instead, Java features
SAM (Single Abstract Method) conversion where an anonymous
function is converted by the compiler to an instance of an
interface type like `java.util.function.Predicate` which 
declares only one abstract method.

Thus, there are many operations in the Java SDK and other
Java libraries which accept a SAM interface. For example, 
`Stream.filter()` accepts a `Predicate`. Such methods are 
represented in Ceylon as an overloaded method:

- one overload accepts the SAM, like
  `Stream<T> filter(Predicate<in T> predicate)`, and
- the second overload accepts a Ceylon function type, like
  `Stream<T> filter(Boolean(T) predicate)`.

Thus, we can pass anonymous functions and function references
to such Java APIs without needing to explicitly implement the
SAM interface type.

<!-- try: -->
    import java.util {
        Arrays
    }
    import java.util.stream {
        Collectors { toList }
    }
    
    value list
        = Arrays.asList("hello", "world", "goodbye")
            .stream()
            .filter((s) => s.longerThan(2))
            .map(String.uppercased)
            .collect(toList<String>());

As you can see, use of the Java streams API is completely 
natural in Ceylon.

### Tip: getting a function from a SAM

There's no inverse conversion from a SAM type to a Ceylon
function type. A `Predicate<T>` can't be passed directly to 
an operation that expects a `Boolean(T)`. But this is 
perfectly fine, because it's completely trivial to perform
the conversion explicitly, just by taking a reference to the
`test` method of `Predicate`:

<!-- try: -->
    Predicate<String> javaPredicate = ... ;
    Boolean(String) ceylonPredicate = javaPredicate.test;

Remember, the Ceylon language defines no implicit type 
conversions _anywhere_!

## Inheriting Java types

A Ceylon class may extend a Java class and/or implement Java
interfaces, even if the Java types are generic. There's 
nothing particularly interesting to say about this topic, 
except to point out one small wrinkle. In Java, the following 
inheritance hierarchy is legal:

<!-- try: -->
<!-- lang: java -->
    interface Foo {
        public boolean test();
    }
    
    interface Bar {
        public boolean test();
    }
    
    class Baz implements Foo, Bar {
        @Override
        public boolean test() {
            return true;
        }
    }

However, a Ceylon class which implements the Java types `Foo`
and `Bar` [is _not_ legal](/inheritance/#ambiguities_in_mixin_inheritance), 
since a Ceylon class may not define an operation that refines 
two completely different but same-named operations unless they 
both descend from a common supertype definition:
    
<!-- try: -->
    class Baz() satisfies Foo & Bar {
        //error: May not inherit two declarations with the 
        //same name that do not share a common supertype
        test() => true;
    }

In this case, it's necessary to either:

- split `Baz` into two classes, 
- define an intermediate Java interface that inherits `Foo` 
  and `Bar`, or
- define a common superinterface of `Foo` and `Bar` that
  declares `test()`.

## Java annotations

Java annotations may be applied to Ceylon program elements.
An initial lowercase identifier must be used to refer to the
Java annotation type.

For example:

<!-- try: -->
    import javax.persistence {
        id,
        generatedValue,
        entity,
        column,
        manyToOne,
    }
    
    entity
    class Person(firstName, lastName, city) {
        
        id generatedValue
        column { name = "pid"; }
        value _id = 0;
        
        shared String firstName;
        shared String lastName;
        
        manyToOne { optional=false; }
        shared City city;
        
        string => "Person(``_id``, ``firstName`` ``lastName``, ``city.name``)";
    }

Note that `id` here refers to the Java annotation 
`javax.persistence.Id`.

### Tip: specifying arguments to anotation parameters of type `Class`

Annotation values of type `java.util.Class` may be specified
by passing the corresponding Ceylon `ClassDeclaration`. For
example, you would use `` type = `class Person` `` where you 
would have used `type = Person.class` in Java.

## Syntax sugar that applies to Java types

Certain syntactic constructs that are defined by the language 
specification in terms of types defined in `ceylon.language` 
are also supported for similar Java types. These constructs
are:

- the `for` loop and comprehensions,
- resource expressions in `try`, and
- the element lookup and `in` operators.

### Java `Iterable` or array in `for` 

It's possible to use a `java.lang.Iterable` or Java array in 
a Ceylon `for` statement or comprehension.

<!-- try: -->
    import java.lang { JIterable=Iterable }
    
    JIterable<Object> objects = ... ;
    for (obj in objects) {
        ...
    }

Wait, a quick reminder...

### Gotcha! (redux)

As we already noted above, `for` does nothing special to the
type of the iterated elements of a Java array or `Iterable`: 
if the elements are Java `String`s, they're _not_ magically 
transformed to Ceylon `String`s:

<!-- try: -->
    import java.lang { ObjectArray, JString=String }
    
    ObjectArray<JString> strings = ...;
    for (s in strings) {  // s is a JString
        String string = s.string;  //string is a Ceylon String
        ...
    }

Just don't forget to use `s.string` to get the Ceylon `String` 
if that's what you need!

### Gotcha again!

Also note that `java.lang.Iterable` is not supported together 
with entry or tuple destructuring:

<!-- try: -->
    JIterable<String->String> stringPairs = ...;
    for (key->item in stringPairs) {
        // not supported
    }

In practice it's unusual to have a Java `Iterable` containing 
Ceylon `Entry`s or `Tuple`s.

### Java `AutoCloseable` in `try`

Similarly, it's possible to use a Java `AutoCloseable` in a
Ceylon `try` statement.

<!-- try: -->
    import java.io { File, FileInputStream }
    
    File file = ... ;
    try (inputStream = FileInputStream(file)) {
         ...
    }

The semantics, naturally, are identical to what you get in
Java.

### Java collections and operators 

Two of Ceylon's built-in operators may be applied to Java types:

- the element lookup operator (`list[index]`) may be used with 
  Java arrays, `java.util.List`, and `java.util.Map`, and
- the containment operator (`element in container`) may be used 
  with the type `java.util.Collection`.

<!-- try: -->
    import java.util { JArrayList=ArrayList }
    
    value list = JArrayList<String>();
    list.add("hello");
    list.add("world");
    
    list[0] = "goodbye";
    assert ("world" in list);
    printAll { for (word in list) word };

There's almost no friction here; if you prefer to use Java's
collection types instead of the mutable collection types 
defined by the module `ceylon.collection`, go right ahead!

### Tip: creating Java lists

The following idioms are very useful for instantiating Java 
`List`s:

<!-- try: -->
    import java.util { Arrays }
    
    value words = Arrays.asList("hello", "world");
    value squares = Arrays.asList(for (x in 0..100) x^2);
    value args = Arrays.asList(*process.arguments);

(Note that these code examples work because `Arrays.asList()`
has a variadic parameter.)

## Utility functions and classes

In the module [`ceylon.interop.java`][] you'll find a suite 
of useful utility functions and classes for Java interoperation. 
For example, there are classes that adapt between Ceylon 
collection types and Java collection types.

### Tip: converting between `Iterable`s

An especially useful adaptor is [`CeylonIterable`][], which lets 
you apply any of the usual operations of a Ceylon [stream][] to a 
Java `Iterable`.

<!-- try: -->
    import java.util { JList=List, JArrayList=ArrayList }
    import ceylon.interop.java { CeylonIterable }
    
    ...
    
    JList<String> strings = JArrayList<String>();
    strings.add("hello");
    strings.add("world");
    
    CeylonIterable(strings).each(print);

(Alternatively, we could have used [`CeylonList`][] in this example.)

Similarly there are `CeylonStringIterable`, `CeylonIntegerIterable`, 
`CeylonFloatIterable`,`CeylonByteIterable` and `CeylonBooleanIterable` 
classes which as well as converting the iterable type also convert
the elements from their Java types to the corresponding Ceylon type.

[`CeylonIterable`]: #{site.urls.apidoc_current_interop_java}/CeylonIterable.type.html
[`CeylonList`]: #{site.urls.apidoc_current_interop_java}/CeylonList.type.html
[stream]: #{site.urls.apidoc_1_3}/Iterable.type.html

### Tip: getting a `java.util.Class`

Another especially useful function is [`javaClass`][], which obtains 
an instance of `java.util.Class` for a given type.

<!-- try: -->
    import ceylon.interop.java { javaClass }
    import java.lang { JClass=Class }
    
    JClass<Integer> jc = javaClass<Integer>();
    print(jc.protectionDomain);

The functions [`javaClassFromInstance`][] and 
[`javaClassFromDeclaration`][] are also useful.

[`javaClass`]: #{site.urls.apidoc_current_interop_java}/index.html#javaClass
[`javaClassFromInstance`]: #{site.urls.apidoc_current_interop_java}/index.html#javaClassFromInstance
[`javaClassFromDeclaration`]: #{site.urls.apidoc_current_interop_java}/index.html#javaClassFromDeclaration

## `META-INF` and `WEB-INF`

Some Java frameworks and environments require metadata packaged 
in the `META-INF` or `WEB-INF` directory of the module archive, 
or sometimes even in the root directory of the module archive. 
We've already seen how to [package resources](../modules/#resources) 
in Ceylon module archives by placing the resource files in the 
module-specific subdirectory of the resource directory, named 
`resource` by default.

Then, given a module named `net.example.foo`:

- resources to be packaged in the root directory of the module 
  archive should be placed in `resource/net/example/foo/ROOT/`,
- resources to be packaged in the `META-INF` directory of the 
  module archive should be placed in 
  `resource/net/example/foo/ROOT/META-INF/`, and
- resources to be packaged in the `WEB-INF` directory of the 
  module archive should be placed in 
  `resource/net/example/foo/ROOT/WEB-INF/`.

## Interoperation with Java's `ServiceLoader`

Annotating a Ceylon class with the `service` annotation makes
the class available to Java's [service loader architecture][].

[service loader architecture]: https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html

<!-- try: -->
     service (`Manager`)
     shared class DefautManager() satisfies Manager {}
 
A Ceylon module may gain access to Java services by calling
[`Module.findServiceProviders()`][].

<!-- try: -->
     {Manager*} managers = `module`.findServiceProviders(`Manager`);
     assert (exists manager = managers.first);"

This code will find the first `service` which implements 
`Manager` and is defined in the dependencies of the module in 
which this code occurs. To search a different module and its 
dependencies, specify the module explicitly:

<!-- try: -->
     {Manager*} managers = `module com.acme.manager`.findServiceProviders(`Manager`);
     assert (exists manager = managers.first);"

The `service` annotation and `Module.findServiceProviders()`
work portably across the JVM and JavaScript environments.

[`Module.findServiceProviders()`]: #{site.urls.apidoc_1_3}/meta/declaration/Module.type.html#findServiceProviders

## Limitations

Here's a couple of limitations to be aware of:

- You can't call Java methods using the named argument 
  syntax, since Java 7 doesn't expose the names of 
  parameters at runtime, and Ceylon doesn't yet depend on 
  language features only available in Java 8.
- You can't obtain a method reference, nor a static method 
  reference, to an overloaded method.
- Java generic types don't carry reified type arguments at 
  runtime, so certain operations that depend upon reified 
  generics (for example, some `is` tests) are rejected at 
  compile time, or unchecked at runtime.

## Alternative module runtimes

When you execute a Ceylon program using `ceylon run`, it
executes on a lightweight module container that is based on
JBoss Modules. But it's also possible to execute a Ceylon 
module in certain other module containers, or without any
module container at all.

### Deploying Ceylon on OSGi

Ceylon is fully interoperable with OSGi, so that Ceylon 
modules:

- can be deployed as pure OSGi bundles in an OSGi container 
  out-of-the-box, without any modification of the module 
  archive file,
- can embed additional OSGi metadata, to declare services for 
  example, and
- can easily use OSGi standard services.

This provides a great and straightforward way to run Ceylon 
modules inside Java EE application servers or enterprise 
containers that support or are are built around OSGi.

You can learn more about Ceylon and OSGi from the 
[reference documentation](../../reference/interoperability/osgi-overview).

[OSGi]: https://www.osgi.org/

### Deploying Ceylon on Jigsaw

The command line tool [`ceylon jigsaw`] deploys a module and
all its dependencies to a Jigsaw-style `mlib/` directory. Since
Jigsaw is not yet released, and is still undergoing changes, this
functionality is considered experimental.

[`ceylon jigsaw`]: /documentation/current/reference/tool/ceylon/subcommands/ceylon-jigsaw.html
[Jigsaw]: http://openjdk.java.net/projects/jigsaw/

### Deploying Ceylon as a fat jar

Finally, if you wish to run a Ceylon program on a machine with
no Ceylon distribution available, and without the runtome module 
isolation provided by JBoss Modules, the command line tool
[`ceylon fat-jar`][] is indispensible. The command simply produces
a Java `.jar` archive that contains a Ceylon module and everything
it depends on at runtime.

[`ceylon fat-jar`]: /documentation/current/reference/tool/ceylon/subcommands/ceylon-fat-jar.html

## There's more ...

You can learn more about `native` declarations from the 
[reference documentation](/documentation/reference/interoperability/native/).

Ceylon's experimental support for [Jigsaw][] is covered in 
greater detail in [this blog post](/blog/2015/12/17/java9-jigsaw/).

In a mixed Java/Ceylon project, you'll probably need to use a 
build system like Gradle, Maven, or Apache `ant`. Ceylon has 
[plugins](../../reference/tool/) for each of these build 
systems.

Finally, we're going to learn about interoperation with 
languages like JavaScript with [dynamic typing](../dynamic).
