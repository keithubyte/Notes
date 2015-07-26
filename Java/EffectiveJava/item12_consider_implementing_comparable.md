### Item 12 : Consider implementing Comparable

----------

By implementing `Comparable`, a class indicates that its instances have a natural ordering. At the same time, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on this interface. You gain a tremendous amount of power for a small amount of effort. Virtually allo of the value classes in the Java platform libraries implement `Comparable`. If you are writing a value class with an obvious natural ordering, such as alphabetical order, numberiacal order, or chronological order, you should strongly consider implementing the interface.

The general contract of the `compareTo` method is similar to that of `equals`: Compares this object with the specified object for order. Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object. Throws `ClassCastException` if the specified object's type prevents it from being compared to this object.