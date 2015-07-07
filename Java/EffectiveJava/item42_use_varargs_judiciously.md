### Use varargs judiciously

In release 1.5, varargs methods, formally known as *variable arity methods*, were added to the language. Varargs methods accept zero or more arguments of a specified type. Sometimes it's appropriate to write a method that requires `one or more` arguments of some type, rather than `zero or more`.

```java
// The WRONG way to use varargs to pass one or more arguments!
static int min(int... args) {
	if(args.length == 0) {
		throw new IllegalArgumentException("Too few arguments");
	}
	int min = args[0];
	for(int i = 1; i < args.length; i++) {
		if(args[i] < min) {
			min = args[i];
		}
		return min;
	}
}
```

This solution has serval problems. The most serious is that if the client invokes this method with no arguments, it fails at runtime rather than compile time. Another problem is that it is ugly. You have to include an explicit validity check on `args`, and you can't use a for-each loop unless you initialize `min` to `Integer.MAX_VALUE`, which is also ugly.

And here is a much better way to achieve the desired effect:

```java
// The RIGHT way to use varargs to pass one or more arguments
static int min(int firstArg, int... remainingArgs) {
	int min = firstArg;
	for(int arg : remainingArgs) {
		if(arg < min) {
			min = arg;
		}
	}
	return min;
}
```

Don't retrofit every method that has a final array parameter; use varargs only when a call really operates on a variable-length sequence of values.

Two method signatures are particularly suspect:

```java

ReturnType1 suspect1(Object... args) {}
<T> ReturnType2 suspect2(T... args){}

```

Methods with either of these signatures will accept any parameter list. Any compile-time type-checking that you had prior to the retrofit will be lost.

Exercise care when using the varargs facility in performance-critical situations. Every invocation of a varargs method causes an array allocation and initialization. If you have determined empirically that you can't afford this cost but you need the flexibility of varargs, there is a pattern that lets you have your cake and eat it too. Suppose you've determined that 95% of the calls to a method have three or fewer parameters. Then declare five overloadings of the method, one each with zero through three ordinary parameters, and a single varargs method for use when the number of arguments exceeds three:

```java

public void foo(){}
public void foo(int a1){}
public void foo(int a1, int a2){}
public void foo(int a1, int a2, int a3){}
public void foo(int a1, int a2, int a3, int... rest){}

``` 

Now you know that you'll pay the cost of the array creation only in the 5% of all invocations where the number of parameters exceeds three. Like most performance optimizations, this technique usually isn't appropriate, but when it is, it's a lifesaver.

In summary, varargs methods are a convenient way to define methods that require a variable number of arguments, but they should not be overused. The can produce confusing results if used inappropriately.