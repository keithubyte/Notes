### Item06 : Eliminate obsolete object references

----------

#### Eliminate obsolete object references

Java is a garbage-collected language, and it can easily lead to the impression that you don't have to think about memory management, but this isn't quite true.

Consider the following simple stack implementation:

```java
/**
 * This Stack may cause "memory leak".
 * Created by keith on 15/3/14.
 */
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[size--];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

If a stack grows and then shrinks, the objects that were popped off the stack will not be garbage collected, even if the program using the stack has no more references to them. This is because the stack maintains `obsolete references` to these objects. An obsolete reference is simply a reference that will never be dereferenced again. In this case, any references outside of the "active portion" of the element array are obsolete. The activity portion consists of the elemetns whose index is less than size.

To fix for this sort of problem is simple : null out references once they become obsolete. In the case of our `Stack` class, the reference to an item becomes obsolete as soon as it's popped off the stack. The corrected version of the *pop* method looks like this:

```java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object object = elements[size--];
        elements[size] = null; // Eliminate obsolete reference
        return object;
    }
```

An added benefit of nulling out obsolete references is that, if they are subsequently dereferenced by mistake, the program will immediately fail with a *NullPointerException*, rather than quietly doing the wrong thing. It is always beneficial to detect programming errors as quickly as possible.

#### When should null out a reference

- Whenever a class manages its own memory, the programmer should be alert for memory leaks.
- Another common source of memory leaks is caches.
- A third common source of memory leaks is listeners and other callbacks.
