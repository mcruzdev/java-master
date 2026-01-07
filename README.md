# java-master

> My study garden about Java technologies

## Java Execution
> Source: https://docs.oracle.com/javase/specs/jls/se21/jls21.pdf (Chapter 12)

This section covers the Java execution model and runtime behavior as specified in Chapter 12 of the Java Language Specification (JLS SE 21), which details how Java programs are executed.

If you run `man java`, you'll find some useful and interesting information:

```shell
The java command starts a Java application.  It does this by starting the Java Virtual Machine (JVM), loading the specified class, and calling that class's main() method.  The method must be declared public and static, it must not
       return any value, and it must accept a String array as a parameter.  The method declaration has the following form:

              public static void main(String[] args)
```

This message is well-known, but understanding what happens behind the scenes when executing a Java program is valuable. This study focuses on Quarkus and GraalVM technologies to deepen knowledge of class loaders, static/runtime initialization, and related concepts.

## Loading

The first step is class loading:

> The initial attempt to execute the method main of class Test discovers that the class
`Test` is not loaded - that is, that the Java Virtual Machine does not currently contain
a binary representation for this class. The Java Virtual Machine then uses a class
loader to attempt to find such a binary representation.

What does **loading** mean?

> Loading refers to the process of finding the binary form of a class or interface
with a particular name, perhaps by computing it on the fly, but more typically by
retrieving a binary representation previously computed from source code by a Java
compiler, and constructing, from that binary form, a Class object to represent the
class or interface

**Example:**
```java
// When you call MyClass.method(), the JVM loads MyClass if not already loaded
public class Loader {
    public static void main(String[] args) {
        MyClass.method(); // Triggers loading of MyClass
    }
}
```

### Load Process

> [!NOTE]
> 
> You can use `defineClass` to construct `Class` objects from binary representations in the `.class` file format.

## Quarkus ClassLoaders

As of January 6, 2025, Quarkus includes the following ClassLoaders:

* ResteasyReactiveTestClassLoader
* ComponentClassLoader
* QuarkusComponentTestClassLoader
* FacadeClassLoader
* ParentLastURLClassLoader
* RuntimeLaunchClassLoader
* TestClassLoader
* JoinClassLoader
* RunnerClassLoader
* QuarkusClassLoader
* ArcTestClassLoader
* DeploymentClassLoader

Quarkus held a [Quarkus Community Call](https://github.com/quarkusio/quarkus/discussions/51772) discussing the long-term goal of moving to a modular runtime to replace existing class loading artifacts.

For more details, refer to [Quarkus Class Loading](https://quarkus.io/guides/class-loading-reference).

## Linking

We have two stages: **Loading** and **Linking**, the Linking has 3 stages, that is verify, prepare and (Optionally) Resolve.

> Additional resource about this subject: https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-5.html

**Example:**
```
Class Loading → Linking (Verify → Prepare → Resolve) → Initialization
```

> A class or interface is always loaded before it is linked

> This specification allows an implementation flexibility as to when linking activities
(and, because of recursion, loading) take place, provided that the semantics of the
Java programming language are respected, that a class or interface is completely
verified and prepared before it is initialized, and that errors detected during linkage
are thrown at a point in the program where some action is taken by the program
that might require linkage to the class or interface involved in the error.

> For example, an implementation may choose to resolve each symbolic reference
in a class or interface individually, only when it is used (lazy or late resolution), or
to resolve them all at once while the class is being verified (static resolution). This
means that the resolution process may continue, in some implementations, after a
class or interface has been initialized.

### Verification

> Verification ensures that the binary representation of a class or interface is
structurally correct

### Preparation

> Preparation involves creating the static fields (class variables and constants) for
a class or interface and initializing such fields to the default values (§4.12.5). This
does not require the execution of any source code; explicit initializers for static
fields are executed as part of initialization (§12.4), not preparation.
Implementations of the Java Virtual Machine may precompute additional data structures
at preparation time in order to make later operations on a class or interface more efficient.
One particularly useful data structure is a "method table" or other data structure that allows
any method to be invoked on instances of a class without requiring a search of superclasses
at invocation time.

### Resolution

> The binary representation of a class or interface references other classes and
interfaces and their fields, methods, and constructors symbolically, using the binary
names (§13.1) of the other classes and interfaces. For fields and methods, these
symbolic references include the name of the class or interface of which the field
or method is a member, as well as the name of the field or method itself, together
with appropriate type information.
Before a symbolic reference can be used it must undergo resolution, wherein a
symbolic reference is checked to be correct and, typically, replaced with a direct
reference that can be more efficiently processed if the reference is used repeatedly.
If an error occurs during resolution, then an error will be thrown.


Kind of possible errors (I did not add everyone):

* IllegalAccessError
* InstantiationError
* NoSuchFieldError
* NoSuchMethodError

## Initialization

> Initialization of a class consists of executing its static initializers and the initializers
for static fields (class variables) declared in the class.

GraalVM documentation about [Class Initialization](https://www.graalvm.org/latest/reference-manual/native-image/optimizations-and-performance/ClassInitialization/).

Now I have the meaning of SyntheticBeanBuildItem, the Synthetic part:

> Note that a compiler may generate synthetic default methods in an interface, that is, default
methods that are neither explicitly nor implicitly declared (§13.1). Such methods will
trigger the interface's initialization despite the source code giving no indication that the
interface should be initialized.

Now I can say: 

> Synthetic in Java means it is neither explicitly nor implicitly declared, but the compiler or a build tool creates it.