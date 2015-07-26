### Item63 : Include failure-capture information in detail messages

----------

**To capture the failure, the detail message of an exception should contain the values of all parameters and fields that "contributed to the exception"**. For example, the detail message of an `IndexOutOfBoundsException` should contain the lower bound, the upper bound, and the index value that failed to lie between the bounds. This information tells a lot about the failure.

One way to ensure that exceptions contain adequate failure-capture information in their detail messages is to require this information in their constructors instead of a string detail message. The detail message can then be generated automatically to include the information. For example, instead of a `String` constructor, `IndexOutOfBoundsException` could have had a constructor that looks like this:

```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
	// Generate a detail message that captures the failure
	super("Lower bound: " + lowerBound
		+ ", Upper bound: " + upperBound
		+ ", Index: " + index);
	
	// save failure information for programmatic access
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;
}
```

Unfortunately, the Java platform libraries do not make heavy  use of this idiom, but it is highly recommended. It makes it easy for the programmer throwing an exception to capture the failure. In fact, it makes it hard for the programmer not to capture the failure! In effect, the idiom centralizes the code to generate a high-quality detail message for an exception in the exception class itself, rather than requiring each user of the class to generate the detail message redundantly.