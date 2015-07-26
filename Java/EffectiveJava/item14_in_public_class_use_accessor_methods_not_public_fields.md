### Item14 : In public class, use accessor methods, not public fields

----------

Occasionally, you may be tempted to rite degenerate classes that serve no purpose other than to group instance fields:

```java
// Degenerate classes like this should not be public!!!
class Point {
    public double x;
    public double y;
}
```

Hard-line object-oriented programmers feel that such classes are anathema and should always be replaced by classes with private fields and public accessor methods and for mutable classes, mutators:

```java
// Encapsulation of data by accessor methods and mutators
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```

Certainly, the hard-liners are correct when it comes to public classes: **if a class is accessible outside its package, provide accessor methods, to preserve the flexibility to change the class's internal representation**.

However, **if a class is package-private or is a private nested class, there is nothing niherently wrong with exposing its data fields** -- assuming they do an adequate job of describing the abstraction provided by the class.

While it's never a good idea for a public class to expose fields directly, it is less harmful if the fields are immutable.

#### Summary

In summary, public classes should never expose mutable fields. It is less harmful, though still questionalbe, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.
