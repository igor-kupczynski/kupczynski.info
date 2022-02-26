---
title: Metaprogramming in Groovy
tags:
- java
aliases:
- /2013/12/07/groovy-method-resolution.html
---

[Groovy][groovy] the language offers neat metaprogramming features. To
_metaprogram_ means to write programs manipulating other
programs. In context of groovy this enables us to synthesize methods
and classes, intercept calls and make objects behave like instances of
different types. All of these manipulations happen at runtime. Groovy
also offers compile-time AST (Abstract Syntax Tree) manipulation.

[groovy]: http://groovy.codehaus.org/


Before we move out to the features themselves, let's start with
figuring out how Groovy can change behavior of various classes of
objects.


Metaclasses
-----------

When we create an object in JVM its class is fixed and can't be
changed at runtime. Groovy introduces the concept of
metaclasses. [`Metaclass`][metacls] is a _class-like_ object specific
to Groovy. Contrary to the regular Java [`Class`][cls] object, it does
not bear a special meaning and can be changed at leisure.

[metacls]: http://groovy.codehaus.org/api/groovy/lang/MetaClass.html
[cls]: http://docs.oracle.com/javase/7/docs/api/java/lang/Class.html


Before invoking a method on an object, Groovy first consults its
metaclass. So changing or extending a metaclass makes an impression of
object changing its class at runtime. But where the metaclass is
stored? To find out we need to explore various kinds of object first.


We can have following kinds of objects in Groovy:

* **Plain Old Java Objects (POJOs)** - instances of regular java
  objects created on JVM.

* **Plain Old Groovy Objects (POGOs)** - subclasses of
[`GroovyObject`][gobj]. An interface defined as follows.

<pre>
public interface GroovyObject {
    Object invokeMethod(String name, Object args);
    Object getProperty(String property);
    Object setProperty(String property, Object newValue);
    MetaClass getMetaClass();
    void setMetaClass(MetaClass metaclass);
}</pre>


* **Groovy Interceptors** - subclasses of [`GroovyInterceptable`][gint]

<pre>
public interface GroovyInterceptable extends GroovyObject {}
</pre>

[gobj]: http://groovy.codehaus.org/api/groovy/lang/GroovyObject.html
[gint]: http://groovy.codehaus.org/api/groovy/lang/GroovyInterceptable.html


Metaprogramming is available for POJOs and POGOs. Groovy interceptors
have their own way of achieving similar flexibility.

With a POGO it is simple. You need to call its `setMetaClass` method
and a reference to this metaclass is stored within the object. With
POJO this is impossible - they are not designed to store a metaclass
reference. For this reason Groovy maintains an application wide
[`MetaClassRegistry`][mcr] which maps `java.lang.Class`es to
metaclasses.

[mcr]: http://groovy.codehaus.org/gapi/groovy/lang/MetaClassRegistry.html


Method Resolution Order
-----------------------

We know that each POJO and POGO has an associated metaclass (either
holds the reference directly or has it stored in application-wide
registry).

### POJO

If you invoke a method on POJO then Groovy first inspects its
metaclass. If the method exists in the metaclass it takes precedence
over the regular one.

An example is given below.

<pre>
// PlainOldJavaClass.java
public class PlainOldJavaClass {
    public void pojoOnly() { System.out.println("From PlainOldJavaClass"); }
    public void alsoInMetaClass() { System.out.println("From PlainOldJavaClass"); }
}
</pre>

<pre>
// Runner.groovy
// Set-up metaclass
PlainOldJavaClass.metaClass.alsoInMetaClass = { println 'From metaclass' }
PlainOldJavaClass.metaClass.metaclassOnly = { println 'From metaclass' }
PlainOldJavaClass pojo = new PlainOldJavaClass();
pojo.pojoOnly()
pojo.alsoInMetaClass()
pojo.metaclassOnly()
</pre>

This prints:
<pre>
From PlainOldJavaClass
From metaclass
From metaclass
</pre>

We can see that the metaclass methods take precedence. A regular method is
even not required.


### POGO

With POGOs there are more steps. If you want to invoke a method the
steps are as follows.

1. If the method exists in metaclass it is invoked;
2. If the method exists in regular class it is invoked;
3. If there is a field with method name and this field holds a
   reference to a closure then the closure is called;
4. If there is a `methodMissing()` then it is called;
5. If there is `invokeMethod()` then it is called;
6. Else, `MethodMissingException` is thrown.

A simple example.

<pre>
// CoolNewGroovyObject.groovy
class CoolNewGroovyObject {
    void regularMethod() { println "GroovyObject#regularMethod" }
    def closureProperty = { println "GroovyObject#property" }
    Object methodMissing(String name, Object args) { println "Method missing: ${name}" }
    def regularProperty = "This is a nice String"
}
</pre>

<pre>
// Runner.groovy
GroovyObject.metaClass.metaMethod = { println 'From metaclass' }
GroovyObject go = new CoolNewGroovyObject()
go.metaMethod()
go.regularMethod()
go.closureProperty()
go.regularProperty()
</pre>

This prints:

<pre>
From metaclass
GroovyObject#regularMethod
GroovyObject#property
Method missing regularProperty
</pre>

### Groovy Interceptors

Groovy Interceptors are simply POGOs implementing the
`GroovyInterceptable` marker interface.

In case of the interceptors all method invocations (both existing and non-
existing) are routed to `invokedMethod()`.

## Summary

We examined various types of objects available in Groovy, we also found out
how the method invocations are dispatched to concrete method definitions. With
this knowledge we should be able to tell which methods gets invoked when.
