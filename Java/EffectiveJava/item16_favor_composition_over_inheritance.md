### Item16 : Favor composition over inheritance

----------

It is safe to use inheritance within a package, where the subclass and the superclass implementation are under the control o fthe same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension. Inheritaning from ordinary concrete classes across package boundaries, however, is dangerous.

**Unlike method invocation, inheritance violates encapsulation.** In other words, a sublcass depends on the implementation details of its superclass for its proper function. The superclass's implementation may change from release to release, and if it does, the subclass may break, even though its conde has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass's authors have designed and documented it specifically for the purpose of being extended.

To make this concrete, let's see the example below:

```java
/**
 * Broken - Inappropriate use of inheritance!
 * Created by keith on 15/4/9.
 */
public class InstrumentedHashSet<E> extends HashSet<E> {
    
    private int addCount = 0;
    
    public InstrumentedHashSet() {
        
    }
    
    public InstrumentedHashSet( int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> collection) {
        addCount += collection.size();
        return super.addAll(collection);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

Suppose we add three elements into the InstrumentedHashSet, and we would expect the `getAddCount` method to return three. But it returns six. This is because the HashSet's `addAll` method is implemented on top of its `add` method.

There is a way to avoid this problem. Instead of extending an existing class, give you new class a private field that references an instance of the exsiting class. This design is called *composition* because the existing class becomes a component of the new one. Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as *forwarding*, and the methods in the new class are known as *forwarding methods*. The resulting clas will be rock solid, with no dependencies on the implementation details of the existing class. Even adding new methods to the existing class will have no impact on the new class.

There is a concrete example below:

```java
/**
 * Reusable forwarding class
 * Created by keith on 15/4/9.
 */
public class ForwardingSet<E> implements Set<E> {
    
    private final Set<E> s;
    
    public ForwardingSet(Set<E> s) {
        this.s = s;
    }
    
    @Override
    public int size() {
        return s.size();
    }

    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }

    @Override
    public Object[] toArray() {
        return s.toArray();
    }

    @Override
    public <T> T[] toArray(T[] ts) {
        return s.toArray(ts);
    }

    @Override
    public boolean add(E e) {
        return s.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return s.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> collection) {
        return s.containsAll(collection);
    }

    @Override
    public boolean addAll(Collection<? extends E> collection) {
        return s.addAll(collection);
    }

    @Override
    public boolean retainAll(Collection<?> collection) {
        return s.retainAll(collection);
    }

    @Override
    public boolean removeAll(Collection<?> collection) {
        return s.removeAll(collection);
    }

    @Override
    public void clear() {
        s.clear();
    }
}
```

```java
/**
 * Wrapper class - uses composition in place of inheritance
 * Created by keith on 15/4/9.
 */
public class InstrumentedSet<E> extends ForwardingSet<E> {

    private int addCount;

    public  InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> collection) {
        addCount += collection.size();
        return super.addAll(collection);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

The disadvantages of wrapper classes are few. One caveat is that wrapper classes are not suited for use in *callback frameworks*, wherein objects pass self-references to other objects for subsequent invocations("callbacks"). Because a wrapped object doesn't know of its wrapper. This is known as the *SELF problem*. Some people worry about the performance impact of forwarding method invocations or the memory footprint impact of wrapper objects. Neither turn out to have much impact in practice. It's tedious to write forwarding methods, but you have to write the forwarding class for each interface only once, and forwarding classes may be provided for you by the package containing the interface.

Inheritance is appropriate only in circumstances where the subclass really is a subtype of the superclass. In other words, a class **B** should extend a class **A** only if an "is-a" relationship exists between the two classes. If you are tempted to have a class **B** extend a class **A**, ask yourself the question: Is every **B** is an **A**? If you cannot turthfully answer yes to this question, **B** should not extend **A**. If the answer is no, it is often the case that **B** should contain a private instance of **A** and expose a smaller and simpler API: **A** is not an essential part of **B**,merely a detail of its implementation.

There is one last set of questions you should ask yourslef before deciding to use inheritance in place of composition. Does the class you contemplate extending have any flaws in its API? If so, are you comfortable propagating those flaws into your class's API? Inheritance propagates any flaws in the superclass's API, while composition lets you design a new API that hides these flaws.

#### Summary

To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.
