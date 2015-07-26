### Item 04 : Enforce noninstantiability with a private constructor

----------

```java
/**
 * Nonistantiable utility class
 * Created by keith on 15/3/12.
 */
public class UtilityClass {
    // Suppress default constructor for noninstantiability 
    private UtilityClass() {
        throw new AssertionError();
    }
    
    // Remainder omitted
}
```

The `AssertionError` isn't strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees that the class will never be instantiated under any circumstances. This idiom is mildly counterintuitive, as the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment, as shown above.
