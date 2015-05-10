# Use enums instead of int constants

The basic idea behind Java's enum types is simple: they are classes that export one instance for each enumeration constant via a public final field. Enum types are effectively final, by virtue of haveing no accessible constructors. Because clients can neither create instances of an enum type nor extend it, there can be no instances but the declared enum constants.

In addition to rectifying the deficiencies of int enums enum types let yout add arbitrary methods and fields and implement arbitrary interfaces. They provide high-quality implementations of all the `Object` methods, they implement `Comparable` and `Serializable`, and their serialized form is designed to withstand most changes to the enum type.

```java
/**
 * Enum type with data and behavior
 */
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.56e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

It is easy to write a rich enum type such as `Planet`. To associate data with enum constants, declare instance fields and write a constructor that takes the data and stores it in the fileds.

There is different data associated with each `Plant` constant, but sometimes you need to associate fundamentally different *behavior* with each constant. For example, suppose you are writing an enum type to represent the operations on a basic four-function calculator, and you want to provide a method to perform the arithmetic operation represented by each constant. One way to achieve this is to switch on the value of the enum:

```java
/**
 * Enum type that switches on its own value - questionable
 */
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    
    double apply(double x, double y) {
        switch (this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("Unknown operation: " + this);
    }
}
```

This code works, but it isn't very pretty. It won't compile without the *throw* statement because the end of the method is technically reachable, even though it will never be reached. Worse, the code is fragile. If you add a new enum constant but forget to add a corresponding case to the *switch*, the enum will still compile, but it will fail at runtime when you try to apply the new operation.

Luckily, there is a better way to associate a different behavior with each enum constant: declare an abstract *apply* method in the enum type, and override it with a concrete method for each constant in a *constant-specific class body*. Such methods knows as *constant-specific method implementations*:

```java
/**
 * Enum type with constant-specific method implementations
 */
public enum Operation {
    PLUS {
        @Override
        double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        @Override
        double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        @Override
        double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        @Override
        double apply(double x, double y) {
            return x / y;
        }
    };

    abstract double apply(double x, double y);
}
```

If you add a new constant to the second version of `Operation`, it is unlikely that you'll forget to provide an apply method. In the unlikely event that you do forget, the compiler wil remind you, as abstract methods in an enum type must be overridden with concrete methods in all of its constants.

Constant-specific method implementations can be combined with constant-specific data. For example, here is a version of `Operation` that overrides the *toString* method to return the symbol commonly associated with the operation:

```java
/**
 * Enum type with constant-specific class bodies and data
 */
public enum Operation {
    PLUS("+") {
        @Override
        double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    abstract double apply(double x, double y);
}
```

Enum types have an automatically generated *valueOf(String)* method that translates a constant's name into the constant itself. If you override the *toString* method in an enum type, consider writing a *fromString* method to translate the custom string representation back to the corresponding enum. The following code will do the trick for any enum, so long as each constant has a unique string representation:

```java
/**
 * Enum type with constant-specific class bodies and data
 */
public enum Operation {

	...

    // Implementing a fromString method on an enum type
    private static final Map<String, Operation> stringToEnum = new HashMap<String, Operation>();

    static { // Initialize map from constant name to enum constant
        for (Operation operation : values()) {
            stringToEnum.put(operation.toString(), operation);
        }
    }

    // Returns Operation for string, or null if string is invalid
    public static Operation fromString(String symbol) {
        return stringToEnum.get(symbol);
    }
}
```

A disadvantage of constant-specific method implementation is that they make it harder to share code among enum constants. For example, consider an enum representing the days of the week in a payroll package. This enum has a method that calculates a worker's pay for that day given the worker's base salary and the number of hours worked on that day. On the five weekdays, any time worked in excess of a normal shift generates overtime pay; on the two weekend days, all work generates overtime pay. With a *switch* statement, it's easy to do this calculation by applying multiple case labels to each of two code fragments:

```java
/**
 *Enum that switches on its value to share code - questionable
 */
public enum PayrollDay {
    MONDAY, TUESDAY, WENDESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int HOURS_PER_SHIFT = 8;

    double pay(double hoursWorked, double payRate) {
        double basePay = hoursWorked * payRate;

        double overtimePay;

        switch (this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = hoursWorked * payRate / 2;
                break;
            default:
                overtimePay = hoursWorked <= HOURS_PER_SHIFT ? 0 : (hoursWorked - HOURS_PER_SHIFT) * payRate / 2;
                break;
        }

        return basePay + overtimePay;
    }
}
```

This code is undeniably concise, but it is dangerous from a maintenance perspective. Suppose you add an element to the enum, perhaps a special value to represent a vacation day, but forget to add a corresponding case to the *switch* statement. The program will still compile, but the *pay* method will silently pay the worker the same amount for a vocation day as for an ordinary weekday.

What you want is to ve *forced* to choose an overtime pay strategy each tiem you add an enum constant. Luckily, there is a nice way to achieve this. The idea is to move the overtime pay computation into a private nested enum, and to pass an instance of this *strategy enum* to the constrcutor for the `PayrollDay` enum. The `PayrollDay` enum then delegates the overtime pay calculation to the strategy enum, eliminating the need for a switch statement or constant-specific method implementation in `PayrollDay`. While this pattern is less concise than the switch statement, it is safer and more flexible:

```java
/**
 * The strategy enum pattern
 */
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY), 
    TUESDAY(PayType.WEEKDAY), 
    WENDESDAY(PayType.WEEKDAY), 
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY), 
    SATURDAY(PayType.WEEKEND), 
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }

    // The strategy enum type
    private enum PayType {
        WEEKDAY {
            @Override
            double overtimePay(double hrs, double payRate) {
                return hrs <= HOURS_PER_SHIFT ? 0 : (hrs - HOURS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            @Override
            double overtimePay(double hrs, double payRate) {
                return hrs * payRate / 2;
            }
        };

        private static final int HOURS_PER_SHIFT = 8;

        abstract double overtimePay(double hrs, double payRate);

        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
```

If switch statements on enums are not good choice for implementing constant-specific behavior on enums, what are they good for? **Switches on enums are good for augmenting external enum types with constant-specific behavior.**

```java
public enum Operation {

	...
    
    // Switch on an enum to simulate a missing method
    public static Operation inverse (Operation operation) {
        switch (operation) {
            case PLUS: return MINUS;
            case MINUS: return PLUS;
            case TIMES: return DIVIDE;
            case DIVIDE: return TIMES;
            default: throw new AssertionError("Unknown operation: " + operation);
        }
    }
}

```

In summary, the advantages of enum types over int constants are compelling. Enums are far more readable, safer, and more powerful. Many enums require no explicit constructor or members, but many otehrs benefit from associating data with each constant and providing methods whose behavior is affected by this data. Far fewer enums benefit fro associating multiple behaviors with a single method. In this relatively rare case, prefer constant-specific mehtods to enums that switch on their own values. Consider the strategy enum pattern if multiple enum constants share common behaviors.