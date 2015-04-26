# Use function objects to represent strategies

Some languages support *function pointer, delegates, lambda expressions* or similar facilities that allow programs to store and transimt the ability to invoke a particular function. Such facilities are typically used to allow the caller of a function to specialize its behavior by passing in a second function. For example, the *qsort* function in C's standard library takes a pointer to a *comparator* function, which *qsort* uses to compare the elements to be stored. The comparator function takes two parameters, each of which is a pointer to an element.

Java does not provide function pointers, but object references can be used to achieve a similar effect. Invoking a method on an object typically performs some operation on *that object*. However, it is possible to define an object whose mehtods perform operations on *other objects*, passed explicitly to the methods. An instance of a class that exports exactly one such method is effectively a pointer to that method. Such instances are known as *function objects*.

```java
public class FunctionObjectDemo {

    /**
     * function object
     */
    private static class StringLengthComparator implements Comparator<String> {

        @Override
        public int compare(String s1, String s2) {
            return s1.length() - s2.length();
        }
    }
    
    private static final Comparator<String> STRING_COMPARATOR = new StringLengthComparator();
    
    public static void main(String[] args) {
        String[] strings = {"1", "123", "12345", "12", "1234"};
        
        Arrays.sort(strings, STRING_COMPARATOR);

        /*
         * if use only once, declared and instantiated as an anonymous class
        Arrays.sort(strings, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return  s1.length() - s2.length();
            }
        });
        */
        
        StringBuilder sb = new StringBuilder();
        sb.append("{");
        for (int i = 0; i < strings.length; i++) {
            sb.append("\"" + strings[i] + "\"");
            if (i != strings.length - 1) {
                sb.append(", ");
            }
        }
        sb.append("}");

        System.out.println(sb);
    }

}
```

To summarize, a primary use of function pointers is to implement the *Strategy Pattern*. To implement this pattern in Java, declare an interface to represent the strategy, and a class that implements this interface for each concrete strategy. When a concrete strategy is used only once, it is typically declared and instantiated as an anonymous class. When a concrete strategy is designed for repeted use, it is generally implemented as a privated static member class and exproted in a public static final field whose type is the strategy interface.