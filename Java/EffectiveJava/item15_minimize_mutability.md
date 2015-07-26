### Item15 : Minimize mutability

----------

An immutable class is simply a class whose instances cannot be modified. All of the information contained in each instance is provided when it is created and is fixed for the lifetime of the object.

To make a class immutable, follow these five rules:

- Don't provide any methods that modify the object's state.
- Ensure that the class can't extended.
- Make all fields final.
- Make all fields private.
- Ensure exclusive access to any mutable components.

Immutable objects are inherently thread-safe; they require no synchronization.

An immutable class can provide static factories that cache frequently requested instances to avoid creating new instances when existing ones would do.

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Cmplex(0, 1);
```

The only real disadvantage of immutable classes is that they requir a separate object for each distinct value. Creating these objects can be costly, especially if they are large.

To guarantee immutability, a class must not permit itself to be subclasses. Typically this is done by making the class final, but there is another, more flexible way to do it. The alternative to making an immutable class final is to make all of its constructors private or package-private, and to add public static factories in place of the public constructors.

```java
// Immutable class with static factories instead of constructors
public class Complex {
	private final double re;
	private final double im;

	private Complex(double re, double im) {
		this.re = re;
		this.im = im;
	}

	public static Complex valueOf(double re, double im) {
		return new Complex(re, im);
	}
}
```

While this approach is not commonly used, it is often the best alternative. It is the most flexible because it allows the use of multiple package-private implementation classes. To its clients that reside outside its package, the immutable class is effectively final because it is impossible to extend a class that comes from another package and that lacks a public or protected constructor. Besides allowing the flexibility of multiple implementation classes, this approach makes it possible to tune the performance of the class in subsequent release by improvig the object-caching capabilities of the static factories.

The list of rules for immutable classes at the begining of this item says that no methods may modify the object and that all its fields must be final. In fact these rules are a bit stronger than necessary and can be rlaxed to improve performance. In truth, no method may produce an externally visible change in the object's state. However, some immutable classes have one or more nonfinal fields in which they cache the results of exprensive computations the first time they are neededs. IF the same value is requested again, the cached value is returned, saving the cost of recalculation. This trick works precisely because the object is immutable, which guarantees that the computation would yield the same result if it were repeated.

#### Summary

To summarize, resist the urge to write a set method for every get method. **Class should be immutable unless there's a very good reason to make them mutable.** You should always make small value objects immutable. There are some classes for which immutability is impractical. **If a class cannot be made immutable, limit its mutability as much as possible.** Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors. Therefore, **make every field final unless there is a compelling reason to make it nonfinal.**
