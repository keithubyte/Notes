### Item 11 : Override clone judiciously

----------

The `Cloneable` interface was intended as a *mixin interface* for objects to advertise that they permit clong. Unfortunately, it fails to serve this purpose. Its primary flaw is that it lacks a `clone` method, and `Object's clone` method is protected. You cannnot, without resoring to *reflection* , invoke the `clone` metho onn object merely because it implements `Cloneable`.

`Cloneable` contains no methods, but it determines the behavior of `Object`'s protected `clone` method implementation: if a class implements `Cloneable`, `Object`'s `clone` method returns a field-by-field copy of the object; otherwise it throws `CloneNotSupportedException`.

The general contract for the `clone` method is weak. Here it is, copied from the specification for `java.lang.Object`:

For any object `x`, the expression `x.clone() != x` will be `true`, and the expression `x.clone().getClass() == x.getClass()` will be `true`, but these are not absolute requirements. While it is typically the case that `x.clone().equals(x)` will be `true`, this is not an absolute requirement.

A well-behaved `clone` method can call constructors to create objects internal to the clone under construction.

The provision that `x.clone().getClass()` should generally be identical to `x.getClass()`, however, is too weak. In practice, programmers assume that if they extend a class and invoke `super.clone` from the subclass, the returned object will be an instance of the subclass. The only way a superclass can provide this functionality is to return an object obtained by calling `super.clone`. If a `clone` method returns an object created by a constructor, it wll have the wrong class. Therefore, **if you override the clone method in a nonfinal class, you should return an object obtained by invoking super.clone**. If all of a class's superclasses obey this rule, then invoking `super.clone` will eventually invoke `Object` 's `clone` method, creating an instance of the right class.

**In practice, a class that implements Cloneable is expected to provide a properly functioning public clone method.**

**In effect, the clone method functions as another constructor; you must ensure that it does no harm to the original object and that it properly establishes invariants on the clone.**

```java
/**
 * A demo to show how to ensure
 * that it does no harm to the original object
 * Created by keith on 15/3/22.
 */
public class Person implements Cloneable {

    private String name;
    private int age;
    private boolean male;
    private String[] parents;

    public Person(String name, int age, boolean male) {
        this.name = name;
        this.age = age;
        this.male = male;
        this.parents = new String[2];
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return name + "'s age is " + age + ", and is a " + (male ? "male" : "female") + ".\n";
    }

    @Override
    protected Person clone() throws CloneNotSupportedException {
        Person person = (Person) super.clone();
        // Ensure no harm to the original object
        person.parents = parents.clone();
        return person;
    }
}
```

Note that the above solution would not work if the elements field were `final`, because `clone` would be prohibited from assigning a new value to the field. This is fundamental problem: `thes clone architecture is incompatible with normal use of final fields referring to mutable objects`, except in cases where the mutable objects may be safely shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove `final` modifiers from some fields.

```java
public class Person implements Cloneable {

    ...
    private final String[] parents;

    public Person(String name, int age, boolean male) {
        ...
        this.parents = new String[2];
    }

    @Override
    protected Person clone() throws CloneNotSupportedException {
        Person person = (Person) super.clone();
        // the clone architecture is incompatible with normal use of final
        // fields referring to mubable objects.
        person.parents = parents.clone();
        return person;
    }
}
```

Like a constructor, a `clone` methjod should not invoke any nonfinal methods on the clone under construction. If `clone` invokes an overridden method, this method will execute before the subclass in which it is defined has had a chance to fix its state in the clone, quite possibly leading to corruption in the clone and the original.

`Object`'s `clone` method is declared to throw `CloneNotSupportedException`, but overriding `clone` methods can omit this declaration. public `clone` methods should omit it because methods don't throw checked exceptions are easier to use. If a class that is designed for innheritance overrides `clone`, the overriding method should mimic the behavior of `Object.clone`: it should be declared `protected`, it should be declared to throw `CloneNotSupportedException`, and the class should not implement `Cloneable`. This gives subclasses the freedom to implement `Cloneable` or not, just as if they extended `Object` directly.

To recap, all classes that implement `Cloneable` should override the `clone` with a public method whose return type is the class itself. This method should first call `super.clone` and then fix any fields that need to be fixed. Typically, this means coping any mutable objects that comprise the internal "deep structure" of the object being cloned, and replacing the clon's references to these objects with references to the copies. While these internal copies can generally be made by calling `clone` recursively, this is not always the best approach. If the class contains only primitive fields or references to immutable objects, then it is probablye the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID or a field representing the object's creation time will need to be fixe, even if it is primitive or immutable.

If you extend a class that implements `Cloneable`, you have little choice but to implement a well-behaved `clone` method. Otherwise, **you are better off providing an alternative measn of object copying, or simply not providing the capability.**

**A find approach to object copying is to provide a copy constructor or copy factory.**

Given all of the problems associated with `Cloneable`, it's safe to say that other interfaces should not extend it, and that classes designed for inheritance should not implement it. Because of its many shortcomings, some expert programmers simply choose never to override the `clone` method and never to invoke it except, perhaps, to copy arrays. If you design a class for inheritance, be aware that if you choose not to provide a well-behaved protected `clone` method, it will be impossible for subclasses to implement `Cloneable`.
