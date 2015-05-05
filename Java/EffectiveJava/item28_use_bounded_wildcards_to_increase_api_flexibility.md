# Use bounded wildcards to increase API flexibility

Parameterized types are *invariant*. In other words, for any two distinct types `Type1` and `Type2`， `List<Type1>` is neither a subtype nor a supertype of `List<Type2>`. While it is counterintuitive that `List<String>` is not a subtype of `List<Object>`, it really does make sense. You can put any object into a `List<Object>`, but you can put only strings into a `List<String>`.

Sometimes you need more flexibility than invariant typing can provide. Consider the `Stack` from Item 26:

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

Suppose we want to add a method that takes a sequence of elements and pushes them all onto the stack. Here is the first attemp:

```java
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for(E e: src) {
        push(e);
    }
}
```

This method compiles cleanly, but it isn't entirely satisfactory. If the element type of the `Iterable src` exactly mathces that of the stack, it works fine. But suppose you have a `Stack<Number>` and you invoke `push(intVal)`, where `intVal` is of type `Integer`. This works, because `Integer` is a subtype of `Number`. So logically, it seems that this should work, too:

```java
Stack<Number> numberStack = new Stack<Number>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```

If you try it, however, you'll get this error message because parameterized types are invariant:

```bash
StackTest.java 7: pushAll(Iterable<Number>) in Stack<Number>
cannot be applied to (Iterable<Integer>)
    numberStack.pushAll(integers);
               ^
```

Luckily, there's a way out:

```java
// Wildcard type for parameter that serves as an E producer
// The input parameter means "Iterable of some subtype of E"
public void pushAll(Iterable<? extends E> src) {
    for(E e: src)
        push(e);
}
```

With this change, not only does `Stack` compile cleanly, but so does the client code that wouldn't compile with the original `pushAll` declaration.

Now suppose you want to write a `popAll` method to go with `pushAll`. The `popAll` method pops each element off the stack and adds the elements to the given collection. Here's how a first attemp at writting the `popAll` method might look:

```java
// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) {
    while(!isEmpty()) 
        dst.add(pop());
}
```

Again, it doesn't seem entirely satisfactory. Suppose you have a `Stack<Number>` and variable of type `Object`. If you pop an element from the stack and store it in the variable, it compiles and runs without error. So shouldn't you be able to do this, too:

```java
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

If you try to compile this client code against the version of `popAll` above, you'll get an error very similar to the one that we got with our first version of `pushAll`: `Collection<Object>` is not a subtype of `Collection<Number>`.

Once again, there is a way out:

```java
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
```

The lesson is clear. **For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.** If an input parameter is both a producer and a consumer, then wildcard types will do you no good: you need an exact type match, which is what you get without and wildcards.

Here is a mnemonic to help you remember which wildcard type to use:

> **PECS stands for producer-extends, consumer-super.**

In other words, if a parameterized type represents a `T` producer, use `<? extends T>`; if it represents a `T` consumer, use `<? super T>`.

In summary, using wildcard types in your APIs, while tricky, makes the APIs far more flexible. If you write a library that will be widely used, the proper use of wildcard types should be considered mandatory. Remember the basic rule: producer-extends, consumer-super. And remember that all comparables and comparators are consumers.