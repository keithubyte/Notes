### Prefer primitive types to boxed primitives

There are three major differences between primitives and boxed primitives:

- Primitives have only their values, whereas boxed primitives have identities distinct from their values. In other words, two boxed primitive instances can have the same value and different identities.
- Primitive types have only fully functional values, whereas each boxed primitive type has one nonfunctional value, which is `null`, in addition to all of the functional values of its corresponding primitive type.
- Primitives are generally more time-and-space-efficient than boxed primitives.

Consider the following comparator, which is designed to represent ascending numerical order on `Integer` values.

```java
// Broken comparator
Comparator<Integer> naturalOrder = new Comparator<Integer>() {
	public int compare(Integer first, Integer second) {
		return first < second ? -1 : (first == second ? 0 : 1); 
	}
}
```

This comparator is deeply flawed. To convince yourself of this, merely print the value of  `naturalOrder.compare(new Integer(42), new Integer(42))`. Both `Integer` instances represent the same value 42, so the value of this expression should be 0, but it's 1, which indicates that the first `Integer` is greater than the second.

Evaluating the expression `first < second` causes the `Integer` instances referred to by `first` and `second` to be `auto-unboxed`; that is , it extracts their primitive values. The evaluation proceeds to check if the first of the resulting `int` values is less than the `second`. But suppose it is not. Then the next test evaluates the expression `first == second`, which performs an *identity comparison* on the two object references. If `first` and `second` refer to distinct `Integer` instances that represent the same `int` value, this comparison will return `false`, and the comparator will incorrectly return 1, indicating that the first `Integer` value is greater than the second. **Applying the == operator to boxed primitives is almost always wrong**.