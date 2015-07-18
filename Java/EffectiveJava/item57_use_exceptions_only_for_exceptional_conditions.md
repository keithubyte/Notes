### Use exceptions only for exceptional conditions

Someday, if you are unlucky, you may stumble across a piece of code that looks something like this:

```java
// Horrible abuse of exceptions. Don't ever do this!
try {
	int i = 0;
	while(true) {
		range[i++].climb();
	}
} catch(ArrayIndexOutOfBoundsException e) {
	// TODO
}
```

Just don't use exceptions to do this kind of horrible things!