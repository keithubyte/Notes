### Item01 : Consider static factory methods instead of constructors

----------

A class can provide its clients with static factory methods instead of , or in addition to, constructors. Providing a static factory method instead of a public constructor has both advantages and disadvantages.

#### Advantages

##### They have names.
For example, the constructor `BigInteger(int, int, Random)`, which returns a `BigInteger` that is probably prime, would have been better expressed as a static factory method named `BigInteger.probablePrime`.

##### They are not required to create a new object each time they're invoked.
This allows immutable classes to use pre-constructed instances, or to cache instances as they're constructed, and dispense them repeatedly to avoid creating unnecessary duplicate objects.
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

##### They can return an object of any subtype of their return type.
The `EnumSet` has no public constructors, only static factories. They return one of two implementations, depending on the size of the underlying enum type: if it has 64 or fewer elements, as most enum types do, the static factories return a `RegularEnumSet` instance, which is backed by a single long; if the enum type has 65 or more elements, the factories return a `JumboEnumSet` instance, backed by a long array.

The existence of these two implementation classes is invisible to clients. If `RegularEnumset` ceased to offer performance advantages for small enum types, it could be eleminated from a future release with no ill effects. Similarly, a future release could add a third or fourth implementation of `EnumSet` if it beneficial for performance. Clients neither know nor care about the class of the object they get back from the factory; they care only that it is some subclass of `EnumSet`.

```java
/**
 * Demonstrate the use of EnumSet.
 * Created by keith on 15/3/7.
 */
public class EnumSetDemo {
    public static void main(String[] args) {
        EnumSet<Color> colors = EnumSet.allOf(Color.class);
        System.out.println("There are " + colors.size()
                + " colors, so enumset type is " + colors.getClass().getName());

        EnumSet<Word> words = EnumSet.allOf(Word.class);
        System.out.println("There are " + words.size()
                + " words, so enumset type is " + words.getClass().getName());
    }

    private enum Color {
        RED, GREEN, YELLOW, BLUE
    }

    private enum Word {
        THIS, IS, JUST, A, DEMO, TO, SHOW, THE, ENUMSET,
        PLEASE, DO, NOT, TRY, DEFINING, SUCH, KIND, OF, ENUM, CLASS,
        WHEN, LENGTH, LESS, THAN, SIXTY, FOUR,
        REGULARENUMSET, WOULD, RETURN, OTHERWISES,
        JUMBOENUMSET, INSTEAD, SO, IT, DESIGNED, FOR, PERFROMENCE,
        CLIENTS, DONT, KNOW, INTERNAL, IMPLEMENTATION,
        ACTUALLY, THEY, ARE, UNNESSARRY, KNOWING, THAT,
        GREAD, IDEA, USED, HOW, MANY, WORDS, I, HAVE, TYPED,
        ALMOST, ENOUGH, ONLY, NEED, SOME, MORE, ACHIEVED,
        HAHA, FUNNY
    }
}
```

The output would be:

```bash
There are 4 colors, so enumset type is java.util.RegularEnumSet
There are 65 words, so enumset type is java.util.JumboEnumSet
```

##### They reduce the verbosity of creating parameterized type instances.

#### Disadvantages

##### The main disadvantage of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.

##### The second disadvantage of static factory methods is that they are not readily distinguishable from other static methods.

You can reduce this disadvantage by drawing attention to static factories in class or interface comments, and by adhering to common naming conventions. Here are some common names for static factory methods:

 - valueOf
Returns an instance that has, loosely speaking, the same value as its parameters.

 - of
A concise alternative to `valueOf`.

 - getInstance
Returns an instance that is described by the parameters but cannot be said to have the same value. In the case of a singleton, it takes no parameters and returns the sole instance.

 - newInstance
Like `getInstance`, except that it guarantees that each instance returned is distinct from all others.

 - getType
Like `getInstance`, but used when the factory method is in a different class. `Type` indicates the type of object returned by the factory method.

 - newType
Like `newInstance`, but used when the factory method is in a different class. `Type` indicates the type of object returned by the factory method.

#### Summary

Static factory methods and public constructors both have their uses, and it pays to understand their relative merits. Often static factories are preferable, so avoid the reflex to provide public constructors without first considering static factories.
