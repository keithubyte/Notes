### Item53 : Prefer interfaces to reflection

----------

The *core reflection facility*, `java.lang.reflect`, offers programmatic access to information about loaded classes. Given a `Class` object, you can obtain `Constructor`, `Method`, and `Field` instances representing the constructors, methods, fields of the class represented by the `Class` instance. These objects provide programmatic access to the class's member names, field types, method signatures, and so on.

Moreover, `Constructor`, `Method`, and `Field` instances let you manipulate their underlying counterparts *reflectively*: you can construct instances, invoke methods, and access fields of the underlying class by invoking methods on the `Constructor`, `Method`, and `Field` instances. Reflection allows one class to use another, even if the latter class did not exist when the former was compiled. This power, however, comes at a price:

- You lose all the benefits of compile-time type checking, including exception checking.
- The code required to perform reflective access is clumsy and verbose.
- Performance suffers.

The core reflection facility was originally designed for component-based application builder tools. Such tools generally load classes on demand and use reflection to find out what methods and constructors the support. The tools let their users interactively construct applications that access these classes, but the generated applications access the classes normally, not reflectively. Reflection is used only at *design time*. **As a rule, objects should not be accessed reflectively in normal applications at runtime**.

**You can obtain many of the benefits of reflection while incurring few of its costs by using it only in a very limited form**. For many programs that must use a class is unavailable at compile time, there exists at compile time an appropriate interface or superclass by which to refer to the class. If this is the case, you can **create instances reflectively and access then normally via their interface or superclass**. If the appropriate constructor has no parameters, then you don't even need to use `java.lang.reflect`, the `Class.newInstance` method provides the required functionality. For example, here's a program that create a `Set<String>` instance whose class is specified by the *set name argument*. If you specify `java.util.HashSet`, the elements are printed in apparently random order; if you specify `java.util.TreeSet`, they're printed in alphabetical order:

```java
/**
 * Reflective instantiation with interface access
 * Created by keith on 15/7/18.
 */
public class Reflective {

    private static final String[] SETS = {
            "java.util.HashSet",
            "java.util.TreeSet"
    };

    private static final String[] STRINGS = {
            "effective",
            "java",
            "second",
            "edition"
    };

    public static void main(String[] args) {
        for (String set : SETS) {
            printElementOrder(set);
        }

    }

    private static void printElementOrder(String name) {
        Class<?> clazz = null;
        try {
            clazz = Class.forName(name);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        Set<String> set = null;
        try {
            set = (Set<String>) clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        set.addAll(Arrays.asList(STRINGS));
        System.out.println(set);
    }
}
```

Here is the results:

```java
[edition, second, java, effective]
[edition, effective, java, second]
```

While this program is just a toy, the technique it demonstrates is very powerful. The toy program could easily be turned into a generic set tester that validates the specified `Set` implementation by aggressively manipulating one or more instances and checking that they obey the `Set` contract. Similarly, it could be turned into a generic set performance analysis tool.

At the same time, this program demonstrates two disadvantages of reflection. First, it can generate three runtime errors, all of which would have been compile-time errors if reflective instantiation were not used. Second, it takes about twenty lines of tedious code to generate an instance of the class from its name, whereas a constructor invocation would fit neatly on a single line.

a legitimate, if rare, use of reflection is to manage a class's dependencies on other classes, methods, or fields that may be absent at runtime. This can be useful if you are writing a package that must run against multiple versions of some other package. The technique is to compile your package against the minimal environment required to support it, typically the oldest version ,and to access any newer classes or methods reflectively. To make this work, you have to take appropriate action if a newer class or method that you are attempting to access does not exist at runtime. Appropriate action might consist of using some alternate means to accomplish the same goal or operating with reduced functionality.

#### Summary

In summary, reflection is a powerful facility that is required for certain sophisticated system programming tasks, but it has many disadvantages. If you are writing a program that has to work with classes unknown at compile time, you should, if at all possible, use reflection only to instantiate objects, and access the objects using some interface or superclass that is known at compile time.

