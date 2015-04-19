# Prefer interfaces to abstract classes

The Java programming language provides two machanisms for defining a type that permits multiple implementations: `interfaces` and `abstract classes`.

**Existing classes can be easily retrofitted to implement a new interface.** If you want to have two classes extend the same abstract class, you have to place the bastract class high up in the type hierarchy where it sublcasses an ancestor of both classes. Unfortunately, this cause great collateral damage to the type hierarchy, forcin gall descendants of the common ancestor to extend the new abstract class whether or not it is appropriate for them to do so.

**Interfaces are ideal for defining mixins.**

**Interfaces allow the construction of nonhierarchical type frameworks.** There is an example:

```java
public interface Singer {
    void sing();
}
```

```java
public interface Songwriter {
    void compose();
}
```

In real life, some singers are also songwriters. Because we used interfaces rather than abstract classes to define these types, it is perfectly permissible for a single class to implement both *Singer* and *Songwriter*:

```java
public interface SingerSongwriter extends Singer, Songwriter {
    // Because SingerSongwriter is an interface
    // you don't need to implement the methods
    // in the Singer and Songwriter interfaces.
}
```

You don't always need this level of flexibility, but when you do, interfaces are lifesaver.

While interfaces are not permitted to contain method implementations, using interfaces to define types does not prevent you from providing implementation assistance to programmers. **You can combine the virtues of interfaces and abstract classes by providing an abstract skeletal implementation class to go with each nontrivial interface that you export.** The interface still defines the type, but the skeletal implementation takes all of the work out of implementing it.

By convention, skeletal impementations are called *AbstractInterface*, where *Interface* is the name of the interface they implement. For example, the *Collections Framework* provides a skeletal implementation to go along with each main collection interface: *AbstractCollection*, *AbstractSet*, *AbstractList*, and *AbstractMap*.

Using abstract classes to define types that permit multiple implementations has one great advantage over using interfaces: **It is far easier to evolve an abstract class than an interface.** If, in a subsequent release, you want to add a new method to an abstract class, you can always add a concrete method containing a reasonable default implementation. All existing implementations of the abstract class will then provide the new method. This does not work for interfaces.

To summarize, an interface is generally the best way to define a type that permits multiple implementations. An exception to this rule is the case where ease fo evolution is deemed more important than flexibility and power. Under these circumstances, you should use an abstract class to define the type, but only if you understand and can acceept the limitations. If you export a nontrivial inteface, you should strongly consider prividing a skeletal implementation to go with it. Finally, you should design all of your public interfaces with the utmost care and test them thoroughly by writing multiple implements.