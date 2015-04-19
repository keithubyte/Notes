# Use interfaces only to define types

When a class implements an interface, ther interface serves as a *type* that can be used to refer to instances of the class. That a class implements an interface should therefore say something about what a client can do with instances of the class. It is inappropriate to define an interface for any purpose.

```java
// The constant inerface pattern is a poor use of interfaces.
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
    // Avogardro's number
    static final double AVOGADROS_NUMBER = 6.0221419e23;
    
    // Boltzmann constant
    static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
    
    // Mass of the electron
    static final double ELECTRON_MASS = 9.10938188e-31;
}
```

You should export the constants with a noninstantiable *utility class*:

```java
public class PhysicalConstants {
    private PhysicalConstants(){} // Prevents instantiation
    
    public static final double AVOGADROS_NUMBER = 6.0221419e23;
    public static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
    public static final double ELECTRON_MASS = 9.10938188e-31;
}
```

In summary, interfaces should be used to define types. They should not be used to export constants.