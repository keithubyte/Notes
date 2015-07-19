### Use lazy initialization judiciously

As is the case for most optimizations, the best advice for lazy initialization is "don't do it unless you need to". Lazy initialization is a double-edged sword. It decreases the cost of initializing a class or creating an instance, at the expense of increasing the cost of accessing the lazily initialized field.

That said, lazy initialization has its uses. If a field is accessed only on a fraction of the instances of a class and it is costly to initialize the field, then lazy initialization may be worthwhile.

**Under most circumstances, normal initialization is preferable to lazy initialization**.

```java
// Normal initialization of an instance field
private final FieldType field = computeFileValue();
```

**If you use lazy initialization to break an initialization circularity, use a synchronized accessor, as it is the simplest, clearest alternative**:

```java
// Lazy initialization of instance field - synchronized accessor
private FieldType field;

synchronized FieldType getField() {
	if(field == null) {
		field = computeFieldValue();
	}
	return field;
}
```

Both of these idioms are unchanged when applied to static fields, except that you add the static modifier to the field and accessor declarations.

**If you need to use lazy initialization for performance on a static field, use the *lazy initialization holder class idiom***. This idiom exploits the guarantee that a class will not be initialized until it is used. Here's how it looks:

```java
// Lazy initialization holder class idiom for static fields
private static class FieldHolder {
	static final FieldType field = computeFieldValue();
}

static FieldType getField() {
	return FieldHolder.field;
}
```

When the `getField` method is invoked for the first time, it reads `FieldHolder.field` for the first time, causing the `FieldHolder` class to get initialized. The beauty of this idiom is that the `getField` method is not synchronized and performs only a field access, so lazy initialization adds practically nothing to the cost of access. A modern VM will synchronize field access only to initialize the class. Once the class is initialized, the VM will patch the code so that subsequent access to the field does not involve any testing or synchronization.

**If you need to use lazy initialization for performance on an instance field, use the *double-check idiom***. This idiom avoids the cost of locking when accessing the field after it has been initialized. The idea behind the idiom is to check the value of the field twice: once without locking, and then, if the field appears to be uninitialized, a second time with locking. Only if the second check indicates that the field is uninitialized does the call initialize the field. Because there is no locking if the field is already initialized, it is critical that the field be declared `volatile`. Here is the idiom:

```java
// Double-check idiom for lazy initialization of instance fields
private volatile FieldType field;
FieldType getField() {
	FieldType result = field;
	if(result == null) { // First check (no locking)
		synchronized(this) {
			result = field;
			if(result == null) { // Second check (with locking)
				field = result = computeFieldValue();
			}
		}
	}
	return result;
}
```

This code may appear a bit convoluted. In particular, the need for the local variable `result` may be unclear. What this variable does is to ensure that `field` is read only once in the common case where it's already initialize. While not strictly necessary, this may improve performance and is more elegant by the standards applied to low-level concurrent programming. On my machine, the method above is about 25 percent faster than the obvious version without a local variable.

The*double-check idiom* is the technique of choice for lazily initializing an instance field. While you can apply the *double-check idiom* to static fields as well, there is no reason to do so: the *lazy initialization holder class idiom* is a better choice.

Two variants of the *double-check idiom* bear noting. Occasionally, you may need to lazily initialize an instance field that can tolerate repeated initialization. If you find yourself in this situation, you can use a variant of the *double-check idiom* that dispenses with the second check. It is, not surprisingly, known as the *single-check idiom*. Here is how it looks. Note that `field` is till declared `volatile`:

```java
// Single-check idiom - can cause repeated initialization!
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;
	if(result == null) {
		field = result = computeFieldValue();
	}
	return result;
}
```

All of the initialization techniques discussed in this item apply to primitive fields as well as object reference fields. When the double-check or single-check idiom is applied to a numerical primitive field, the field's value is checked against 0 rather than `null`.

If you don't care whether every thread recalculates the value of a field, and the type of the field is a primitive other than `long` or `double`, then you may choose to remove the `volatile` modifier from the field declaration in the single-check idiom. This variant is known  as *racy single-check idiom*. It speeds up field access on some architectures, at the expense of additional initializations. This is definitely an exotic technique, not for everyday use. It is, however, used by `String` instances to cache their hash codes.

In summary, you should initialize most fields normally, not lazily. If you must initialize a field lazily in order to achieve your performance goals, or to break a harmful initialization circularity, then use the appropriate lazy initialization technique. For instance fields, it is the double-check idiom; for static fields, the lazy initialization holder class idiom. For instance fields that can tolerate repeated initialization, you may also consider the single-check idiom.