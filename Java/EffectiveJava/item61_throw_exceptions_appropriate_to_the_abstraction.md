### Item61 : Throw exceptions appropriate to the abstraction

----------

It is disconcerting when a method throws an exception that has no apparent connection to the task that it performs. This often happens when a method propagates an exception thrown by a a lower-level abstraction. Not only is this disconcerting, but it pollutes the API of the higher layer with implementation details. If the implementation of the higher layer changes in a subsequent release, the exception that it throws will change too, potentially breaking existing client programs.

To avoid this problem, **higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstractions**. This idiom is known as *exception translation*:

```java
// Exception Translation
try {
	// Use lower-level abstraction to do our bidding
	...
} catch(LowerLevelException e) {
	throw new HigherLevelException(...);
}
```

A special form of exception translation called *exception chaining* is appropriate in cases where the lower-level exception might be helpful to someone debugging the problem that caused the higher-level exception. The lower-level exception (the cause) is passed to the higher-level exception, which provides an accessor method (Throwable.getCause) to retrieve the lower-level exception:

```java
// Exception Chaining
try {
	...// Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause){
	throw new HigherLevelException(cause);
}
```

The higher-level exception's constructor passes the cause to a chaining-aware superclass constructor, so it is ultimately passed to one of `Throwable`'s chaining-aware constructors, such as `Throwable(Throwable)`:

```java
// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

Most standard exceptions have chaining-aware constructors.

#### Summary

In summary, if it isn't feasible to prevent or to handle exceptions from lower layers, use exception translation, unless the lower-level method happens to guarantee that all of its exceptions are appropriate to the higher level. Chaining provides the best of both worlds: it allows you to throw an appropriate higher-level exception, while capturing the underlying cause for failure analysis.