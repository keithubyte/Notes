### Item17 : Design and document for inheritance or else prohibit it

----------

First, the class must document precisely the effects of overriding any method. In other words, the class must document it self-use of overridable methods. For each public or protected method or constructor, the documentation must indicate which overridable methods the method or constructor invokes, in what sequence, and how the results of each invocation affect subsequent processing.

Design for inheritance involves more than just documenting patterns of self-use. To allow programmers to write efficient subclasses without undue pain, **a class may have to provide hooks into its internal workings in the form of judiciously chosen protected methods** or , in rare instances, protected fields.

**The only way to test a class designed for inheritance is to write subclasses. Therefore, you must test your class by writing subclasses before you release it.**

There are a few more restrictions that a class must obey to allow inheritance. **Constructors must not invoke overridable methods, directly or indirectly.** If you violate this rule, program failure will result. The superclass constructor runs before the subclass constructor, so the overriding method in the subclass will get invoked before the subclass constructor has run. If the overriding method depends on any initialization performed by the subclass constructor, the method will not behave as expected.

```java
public class Super {

    // Broken -- constructor invokes an overridable method
    public Super() {
        overrideMe();
    }

    public void overrideMe() {

    }

}
```

Here's a sublcass that overrides the `overrideMe` method which is erroneously invoked by `Super`'s sole constructor:

```java
public class Sub extends Super {

    private final Date date;

    public Sub() {
        date = new Date();
    }

    // Override method invoked by superclass constructor
    @Override
    public void overrideMe() {
        super.overrideMe();
        System.out.println(date);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

And the output is:

```bash
null
Mon Apr 13 21:56:47 CST 2015
```

The `Cloneable` and `Serializable` interfaces present special difficulties when designing for inheritance. It is generally not a good idea for a class designed for inheritance to implement either of these interfaces, as they place a substantial burden on programmers who extend the class. There are, however, special actions that you can take to allow subclases to implement these interfaces without mandating that they do so.

If you decide to implement `Cloneable` or `Serializable` in a class designed for inheritance, you should be aware that because the `clone` and `readObject` methods behave a lot like constructors, a similar restriction applies: **neither clone nor readObject may invoke an overridable method, directly or indirectly.** In the case of the `readObject` method, the overriding method will run before the subclass's state has been deserialized. In the case of the `clone` method, the overriding method will run before the subclass's `clone` method has a chance to fix the clone's state. In either case, a program failure is likely to follow. In the case of `clone`, the failure can damage the original object as well as the clone. This can happen, for example, if the overriding method assumes it is modifying the clone's copy of the object's deep structure, but the copy hasn't been made yet.

Finally, if you decide to implement `Serializable` in a class designed for inheritance and the class has a `readResolve` or `writeReplace` method, you must make the `readResolve` or `writeReplace` method protected rather than private. If these methods are private, they will be silently ignored by subclasses. This is one more case where an implementation detail becomes part of a class's API to permit inheritance.

Traditionally, the ordininary concrete classes are neither final nor designed and documented for subclassing, but this state of affairs is dangerous. Each time a change is made in such a class, there is a chance that client classes that extend the class will break.

**The best solution to this problem is to prohibit subclassing in classes that are not designed and documented to be safely subclassed.** There are two ways to prohibit subclassing. The easier of the two is to declare the class final. The alternative is to make all the constructors private or package-private and to add public static factories in place of the constructors.
