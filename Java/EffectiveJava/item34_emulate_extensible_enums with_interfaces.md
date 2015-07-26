### Item34 : Emulate extensible enums with interfaces

----------

For the most part, extensibility of enums turns out to be a bad idea. It is confusing that elements of an extension type are instances of the base type and not vice versa. There is no good way to enumerate over all of the elements of a base type and its extension. Finally, extensibility would complicate many aspects of the deign and implementation.

That said, there is at least one compelling use case for extensible enumberated types, which is `operation codes`, alos known as `opcodes`. An opcode is an enumerated type whose elements represent operations on some machine, such as the `Operation` type in Item 30, which is represents the functions on a simple calculator.

Luckily, there is a nice way to achieve this effect using enum types. The basic idea is to take advantage of the fact that enum types can implement arbitrary interfaces by defining an interface for the opcode type and enum that is the standard implementation of this interface.

For example, here is an extensible version of `Operation` type from Item 30:

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BaseOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BaseOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return this.symbol;
    }
}
```

While the enum type(BaseOperation) is not extensible, the interface type(Operation) is, and it is the interface that is used to represent operations in APIs. You can define another enum type that implenents this interface and use instances of the new type in place of the base type. For example, suppose you want to define an extension to the operation type above, consisting of the exponentiation and remainder operations:

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

Now, you can use your new operations anywhere you could use the basic operations, provided that APIs are written to take the interface type, not the implementation.

#### Summary

In summary, **while you cannot write an extensible enum type, you can emulate it by writing an interface to go with a baisc enum type that implements the interface.** This allows clients to write their own enums that implement the interface. These enums can be used wherever the basic enum type can be used, assuming APIs are written in terms of the interface.