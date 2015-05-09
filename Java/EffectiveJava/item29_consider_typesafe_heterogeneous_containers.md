# Consider typesafe heterogenrous containers

The most common use of generics is for collections, such `Set` and `Map`, and single-element containers, such as `TheadLocal` and `AtomicReference`. In all of these uses, it is the container that is parameterized. This limits you to a fixed number of type parameters per container. Normally that is exactly what you want. A `Set` has a single type parameter, representing its element type; a `Map` has two, representing its key and value types; and so forth.

Sometimes, however, you need more flexibility. For example, a database row can have arbitrarily many columns, and it would be nice to be able to accesss all of them in a typesace manner. Luckily, there is an easy way to achieve this effect. The idea is to parameterize the *key* instead of the *container*. Then present the parameterized key to the container to insert or retrieve a value. The generic type system is used to guarantee that the type of the value agrees with its key.

As a simple example of this approach, consider a `Favorites` class that allows its clients to store and retrieve a "favorite" instance of arbitrarily many other classes. The `Class` object will play the part of the parameterized key. The reason this works is that class `Class` was generified in release 1.5. The type of a class literal is no longer simply `Class`, but `Class<T>`. For example, `String.class` is of type `Class<String>`, and `Integer.class` is type of `Class<Integer>`. When a class literal is passed among methods to communicate both compile-time and runtime type information, it is called a *type token*.

Here is the `Favorites` API:

```java
public class Favorites {
    
    private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
        if (type == null) {
            throw new NullPointerException("type is null");
        }
        favorites.put(type, instance);
    }
    
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
    
}
```

Here is a sample program that exercises the `Favorites` class:

```java
public class DatabaseDemo {
    public static void main(String[] args) {
        Favorites f = new Favorites();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebace);
        f.putFavorite(Class.class, Favorites.class);

        String string = f.getFavorite(String.class);
        int i = f.getFavorite(Integer.class);
        Class<?> clazz = f.getFavorite(Class.class);

        System.out.printf("%s %x %s%n", string, i, clazz.getName());
    }
}
```

A `Favorites` instance is *typesafe*: it will never return an `Integer` when you ask it for a `String`. It is also *hererogeneous*: unlike an ordinary map, all the keys are of different types. Therefore, we call `Favorites` a *typesafe heterogeneous container*.

There are two limitations to the `Favorites` class that are worth noting. First, a malicious client could easily corrupt the type safety of a `Favorites` instance, simply by using a `Class` object in its raw form. But the resulting client code would generate an unchecked warning when it was compiled. This is no different fro the normal collection implementations such `HashSet` and `HashMap`. You can easily put a `String` into a `HashSet<Integer>` by using the raw type `HashMap`. That said, you can have runtime type safety if you're willing to pay for it. The way to ensure that `Favorites` never violates its type invariant is to have the *putFavorite* method check that *instance* is indeed an instance of the type represented by type. And we already know how to do this:

```java
    public <T> void putFavorite(Class<T> type, T instance) {
        if (type == null) {
            throw new NullPointerException("type is null");
        }
        // Achieving runtime type safety with a dynamic cast
        favorites.put(type, type.cast(instance));
    }
```

The second limitation is that it cannot be used on a non-reifiable type. In other words, you can store your favorite `String` or `String[]`, but not your favorite `List<String>`. If you try to store your favorite `List<String>`, your program won't compile. The reason is that you can't get a `Class` object for `List<String>`: `List<String>.class` is a syntax error, and it's a good thing, too. `List<String>` and `List<Integer>` share a single `Class` object, which is `List.class`. It would wreak havoc with the internals of a `Favorites` object if the "type literals" `List<String>.class` and `List<Integer>.class` were legal and returned the same object reference.

There is no entirely satisfactory workaround for the second limitation. There is a technique called *super type tokens* that goes a long way toward addressing the limitation, but this technique has limitations of its own.

The type tokens used by `Favorites` are unbounded: *getFavorite* and *putFavorite* accept any `Class` object. Sometimes you may need to limit the types that can be passed to a method. This can be achieved with a *bounded type token*, which is simply a type token that places a bound on what type can be represented, using a bounded type parameter or a bounded wildcard.

The annotations API make extensive use of bounded type tokens. For example, here is the method to read an annotation at runtime. This method comes from the `AnnotatedElement` interface, which is implemented by the reflective types that represet classes, methods, fields, and other program elements:

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

Suppose you have an object of type `Class<?>` and you want to pass it to a method that requires a bounded type toke, such as *getAnnotation*. You could cast the object to `Class<? extends Annotation>`, but this cast is unchecked, so ti would generate a compile-time warning. Luckily, class `Class` provides an instance method that performs this sort of cast safely and dynamically. The method is called *asSubclass*, and it casts the `Class` object on which it's called to represent a subclass of the class represented by its argument. If the cast succeeds, the method returns its argument; if it fails, it throws a `ClassCastException`.

Here's how you use the *asSubclass* method to read an annotation whose type is unknown at compile time. This method compiles without error or warning:

```java
// Use of asSubclass to safely cast to a bounded type token
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null;
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
    
    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```

In summary, the normal use of generics, exemplified by the collection APIs, restricts you to a fixed number of type parameters per container. You can get around this restriction by placing the type parameter on the key rather than the container. You can use `Class` objects as keys for such typesafe heterogeneous containers. A `Class` object used in this fashion is called a type token. You can also use a custom key type. For example, you could have a `DatabaseRow` type representing a database row, and a generic type `Column<T>` as its key.