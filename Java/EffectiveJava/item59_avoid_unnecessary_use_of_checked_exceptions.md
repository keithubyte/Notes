### Item59 : Avoid unnecessary use of checked exceptions

----------

One technique for turning a checked exception into an unchecked exception is to break the method that throws the exception into two methods, the first of which returns a `boolean` that indicates whether the exception would be thrown. This API refactoring transforms the calling sequence from this:

```java
// Invocation with checked exception
try {
	obj.action(args);
} catch (TheCheckedException e) {
	// Handle exceptional condition
	...
}
```

to this:

```java
// Invocation with state-testing method and unchecked exception
if (obj.actionPermitted(args)) {
	obj.action(args);
} else {
	// Handle exceptional condition
	...
}
```

This refactoring is not always appropriate, but where it is appropriate, it can make an API more pleasant to use. While the latter calling sequence is no prettier than the former, the resulting API is more flexible.