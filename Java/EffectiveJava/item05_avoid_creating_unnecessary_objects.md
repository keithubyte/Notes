### Item05 : Avoid creating unnecessary objects

----------

It is often appropriate to reuse a single object instead of creating a new functionally equivalent object each time it is needed. Reuse can be both faster and more stylish. An object can always be reused if it is **immutable**.

```java
/**
 * Created by keith on 15/3/12.
 */
public class AvoidCreating {

    String s1 = new String("Hello World."); // DON'T DO THIS!
    String s2 = "Hello World.";

    Boolean bool1 = new Boolean("true"); // DON'T DO THIS!
    Boolean bool2 = Boolean.valueOf(true);

}
```

There are two examples that showing the bad and the good, respectively.

```java
/**
 * Bad version.
 * Created by keith on 15/3/12.
 */
public class Person {
    private final Date birthDate;

    private Person(Date birthday) {
        this.birthDate = birthday;
    }

    public boolean isBabyBoomer() {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomStart = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomEnd = gmtCal.getTime();
        return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
    }
}
```

`isBabyBoomer` method in the bad version unnecessarily creates a new `Calendar`、`TimeZone` and two `Date` instances each time it is invoked. The version that follows avoids this inefficiency with a static initializer:

```java
/**
 * Good version.
 * Created by keith on 15/3/12.
 */
public class Person {
    private final Date birthDate;

    private static final Date BOOM_START;
    private static final Date BOOM_END;

    static {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }

    private Person(Date birthday) {
        this.birthDate = birthday;
    }

    public boolean isBabyBoomer() {
        return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
    }
}
```

There's a new way to create unnecessary objects in release 1.5. It is called `autoboxing`, and it allows the programmer to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types.

```java
/**
 * Autoboxing Hideously slow program.
 * Created by keith on 15/3/13.
 */
public class AutoBoxing {
    public static void main(String[] args) {
        Long sum = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum += i; // Autoboxing creates unnecessary Long instances.
        }
        System.out.println(sum);
    }
}
```

#### Summary

This item should not be misconstrued to imply that object creation is expensive and should be avoided. On the contray, the creation and reclamation of small objects whose constructors do little explicit work is cheap, especially on modern JVM implementations. Creating additional objects to enhance the clarity, simplicity, or power of a program is generally a good thing.

Conversely, avoiding object creation by maintaining your own `object pool` is a bad idea unless the objects in  the pool are extremely heavyweight. The classic example of an object theat does justify an object pool is a database connection.
