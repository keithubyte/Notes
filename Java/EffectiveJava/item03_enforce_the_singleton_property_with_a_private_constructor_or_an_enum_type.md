### Item 03 : Enforce the singleton property with a private constructor or an enum type

----------

Singletons typically represent a system component that is intrinsically unique, such as the window manager or file system.

#### Ways to implement singleton

###### Singleton with public final field

```java
/**
 * Singleton with public static final field
 * Created by keith on 15/3/11.
 */
public class SingletonA {
    public static final SingletonA INSTANCE = new SingletonA();

    private SingletonA() {
        // TODO
        // do the initialisation
    }

    public void leaveTheBuilding() {
        // TODO
        // other things
    }
}
```

###### Singleton with static factory

```java
/**
 * Singleton with static factory
 * Created by keith on 15/3/11.
 */
public class SingletonB {
    private static final SingletonB INSTANCE = new SingletonB();

    private SingletonB() {
        // TODO
        // do the initialisation
    }

    public static SingletonB getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() {
        // TODO
        // other things
    }
}
```

###### Singleton with enum

```java
/**
 * Enum singleton -- the preferred approach
 * Created by keith on 15/3/11.
 */
public enum SingletonC {
    INSTANCE;

    public void leaveTheBuilding() {
        // TODO
        // other things
    }
}
```

#### Summary

The first two implementations are based on keeping the constructor private and exproting a public static member to provide access to the sole instance. Both of them have a disvantage: a privileged client can invoke the private constructor reflectively with the aid of the `AccessibleObject.setAccessible` method. If you need to defend against this attack, modify the constructor to make it throw an exception if it's asked to create a second instance.

The third aproach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. **A single-element enum type is the best way to implement a singleton.**
