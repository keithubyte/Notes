### Item22 : Favor static member classes over nonstatic

----------

A *nested class* is a class defined within another class. A nested class should exist only to serve its enclosing class. If a nested class would be useful in some other context, then it should be a top-level class. There are four kinds of nested classes: *static member classes*, *nonstatic member classes*, *anonymous classes* and *local classes*.

One common use of a static member class is as a public helper class, useful only in conjunction with its outer class. For example, consider an enum describing the operations supported by a calculator. The `Operation` enum should be a public static member class of the `Calculator` class. Clients of `Calculator` could then refer to operations using names like `Calculator.Operation.PLUS` and `Calculator.Operation.MINUS`.

Each instance of a nonstatic member class is implicitly associated with an *enclosing instance* of its containing class. The association between a nonstatic member class instance and its enclosing instance is established when the former is created; it cannot be modified thereafter.

One common use of a nonstatic member class is to define an *Adapter* that allows an instance of the outer class to be viewed as an instance of some unrelated class. For example, implementations of the `Map` interface typically use nonstatic member classes to implement their *collection views*, which are returned by `Map`'s *keySet*, *entrySet*, and *values* methods. Similarly, implementations of the collection interfaces, such as `Set` and `List`, typically use nonstatic member classes to implement their iterators:

```java
public class MySet<E> extends AbstractSet<E> {

    private static final int DEFAULT_CAPACITY = 8;

    private Object[] elements;
    private int position = 0;

    public MySet() {
        elements = new Object[DEFAULT_CAPACITY];
    }
    
    public MySet(int capacity) {
        elements = new Object[capacity];
    }

    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    @Override
    public int size() {
        return elements.length;
    }

    private class MyIterator implements Iterator<E> {

        @Override
        public boolean hasNext() {
            return position < elements.length - 1;
        }

        @Override
        public E next() {
            return (E) elements[++position];
        }

        @Override
        public void remove() {
            elements[position] = null;
        }
    }
}
```

**If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration**, making it a static rather nonstatic member class. If you omit this modifier, each instance will have an extraneous reference to its enclosing instance. Storing this reference costs time and space, and can result in the enclosing instance being retained when it would otherwise be eligible for garbage collection. And should you ever need to allocate an instance without an enclosing instance, you'll be unable to do so, as nonstatic member class instances are required to have an enclosing instance.

A common use of private static member classes is to represent components of the object represented by their enclosing class. For example, consider a `Map` instace, which associated keys with values. Many `Map` implementations have an internal `Entry` object for each key-value pair in the map. While each entry is associated with a map, the methods on an entry do not need access to the map. Therefore, it would be wasteful to use a nonstatic member class to represent entries: a private static member class is best. If you accidentally omit the static modifier in the entry declaration, the map will still work, but each entry will contain a superfluous reference to the map, which wastes space and time.

One common use of anonymous classes is to create *function objects* on the fly. Another common use of anonymous classes is to create *pocess objects*, such as `Runnable`, `Thread`, or `TimerTask` instances. A third common use is within static factory mehods.

Local classes have attributes in common with each of the other kinkds of nested classes. Like member classes, they have names and can be used repeatedly. Like anonymous classes, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members. And like anonymous classes, they should be kept short so as not to harm readability.

To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of the member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.