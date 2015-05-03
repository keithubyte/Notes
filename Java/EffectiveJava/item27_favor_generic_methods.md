# Favor generic methods

Static utility methods are particularly good candidates for generification. For example:

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new Hash<E>(s1);
    result.addAll(s2);
    return result;
}
```

A limitation of the `union` methods is that the types of all thress sets have to be the same. You can make the method more flexible by using *boulded wildcard types*.

One noteworthy feature of generic methods is that you needn't specify the value of the type parameter explicitly as you must when invoking generic constructors. The compiler figures out the value of the type parameters by examining the types of the method arguments. This process is called *type inference*.

You can exploit the type inference provided by generic method invocation to ease the process of creating parameterized type instances.

```java
// Parameterized type instance creation with constructor
Map<String, List<String>> anagrams = new HashMap<String, List<String>>();
```

To eliminate this redundancy, write a *generic static factory method* corresponding to each constructor that you want to use. For example, here is a generic static factory mehtod corresponding to the parameterless `HashMap` constructor:

```java
// Generic static factory method
public static <K, V> HashMap<K, V> newHashMap() {
    return new HashMap<K, V>();
}
```

With this generic static factory method, you can replace the repetitions declaration above with this concise one:

```java
// Parameterized type instance creation with static factory
Map<String, List<String>> anagrams = newHashMap();
```

It would be nice if the language did the same kind of type interface when invoking constructors on generic types as it does when invoking generic methods. Someday it might, but as of release 1.6, it does not. (ps: it does in 1.7, :))

A related pattern is the `generic singleton factory`. On occasion, you will need to create an object that is immutable but applicable to many different types. Because generics are implemented by erasure, you can use a single object for all required type paramaterizations, but you need to write a static factory method to repeatedly dole out the object for each requested type parameterization.

Suppose you have an interface that describes a function that accepts and returns a value of some type `T`:

```java
public interface UnaryFunction<T> {
    T apply(T arg);
}
```

Now suppose that you want to provide an identit function. It would be wasteful to create a new one each time it's required, as it's stateless. If generices were reified, you would need one identity function per type, but since they're erased you need only a generic singleton. Here's how it looks:

```java
// Generic singleton factory pattern
private static UnaryFunction<Object> IDENTITY_FUNCTION = 
    new UnaryFunction<Object>() {
        public Object apply(Object arg) {
            return arg;
        }
    };
    
// IDENTITY_FUNCTION is stateless and its type parameter is
// unbounded so its safe to share one instance across all types.
@SuperressWarnings("unchecked")
public static <T> UnaryFunction<T> identityFunction() {
    return (UnaryFunction<T>) IDENTITY_FUNCTION;
}
```

Here is a sample program that uses our generic singleton as a `UnaryFunction<String>` and a `UnaryFunction<Number>`. As usual, it contains on casts and compiles without errors or warnings:

```java
// Sample program to exercise generic singleton
public static void main(String[] args) {
    String[] strings = {"jute", "hemp", "nylon"};
    UnaryFunction<String> sameString = identityFunction();
    for(String s: strings) {
        System.out.println(sameString.apply(s));
    }
    
    Number[] numbers = {1, 2.0, 3L};
    UnaryFunction sameNumber = identityFunction();
    for(Number n: numbers) {
        System.out.println(sameNubmer.apply(n));
    }
}
```

In summary, generic methods, like generic types, are safer and easier to use than mehtods that require theri clients to cast input parameters and return values. Like types, you should make sure that your new methods can be used without casts, which will often mean making generic. And like types, you should generify your existing methods to make life easier for new user without breaking existing clients.