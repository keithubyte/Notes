# Use EnumSet instead of bit fields

If the elements of an enumerated type are sued primarily in sets, it is traditional to use the **int enum pattern**, assigning a different power of 2 to each constant:

```java
// Bit field enumeration constants - OBSOLETE!
public class Text {
    public static final int STYLE_BOLD              = 1 << 0; // 1
    public static final int STYLE_ITALIC            = 1 << 1; // 2
    public static final int STYLE_UNDERLINE         = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH     = 1 << 3; // 8 
    
    // Parameter is bitwise OR of zero or more STYLE_ constants
    public void applyStyles(int styles) {
        // ...
    }
}
```

This representation lets you use the bitwise **OR** operation to combine serval constants into a set, known as *a bit field*:

```java
text.applyStyles(STYLE_BOLE | STYLE_ITALIC);
```

The bit field representation also lets you perform set operations such as union and intersection efficiently using bitwise arithmetic. But bit fields hava all the distanvatges of **int** enum constants and more. It is even harder to interpret a bit field than a simple **int** enum constant when it is printed as a number. Also, there is no easy way to iterate over all of the elements represented by a bit field.

The **java.util** package provides the **EnumSet** class to efficiently represent sets of values drawn fro a single enum type. Internally, each **EnumSet** is represented as a bit vector. If the underlying enum type has sixty-four or fewer elements -- and most do -- the entire **EnumSet** is represented with a single **long**, so its performance is comparable to that of a bit field.

Here is how the previous example looks when modified to use enums instead of bit fields:

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyle(Set<Style> styles) {
        // ...
    }
}
```

**EnumSet** provides a rich set of static factories for easy set creation, one of which is illustrated in this code:

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

In summary, **just because an enumerated type will be used in sets, there is no reason to represent it with bit fields**.