### Item08 : Obey the general contract when overriding equals

----------

Overriding the `equals` method seems simple, but there are many ways to get it wrong, and consequences can be dire. The easiest way to avoid problems is not to override the `equals` method, in which case each instance of the class is equal only to itself. This is the right thing to do if any of the following conditions apply:

- Each instance of the class is inherently unique.
- You don't care whether the class provides a "logical equality" test.
- A superclass has already overidden `equals`, and the superclass behavior is appropriate for this class.
- The class is private or package-private, and you are certain that its `equals` method will never be incoked.

So when is it appropriate to override `Object.equals`? When a class has a notion of *logical equality* that differs from mere object identity, and a superclass has not already overridden `equals` to implement the desired behavior. This is generally the case for *value classes*. A value class is simply a class that represents a value, such as `Integer` or `Date`. A programmer who compares references to value objects using the `equals` method expects to find out whether they are logically equivalent, not whether they refer to the same object.

When you oberride the `equals` method, you must adhere to its general contract. Here is the contract, copied from the specification for `Object[Java 6]`:

- Reflexive: For any non-null reference value `x`, `x.equals(x)` must return `true`.
- Symmetric: For any non-null reference values `x` and `y`, `x.equals(y)` returns `trun` if and only if `y.equals(x)` returns true.
- Transitive: For any non-null reference values `x`, `y`, `z`，if `x.equals(y)` returns `true` and `y.equals(z)` return `true`, then `x.equals(z)` must return `true`.
- Consistent: For any non-null reference values `x` and `y`, multiple invocations of `x.equals(y)` consistently return `true` or consistently return `false`, provided no information used in `equals` comparisons on the objects is modified.
- For any non-null reference value `x`, `x.equals(null)` must return `false`.

Here's a recipe for a high-quality `equals` method:

- Use the `==` operator to check if the argument is a reference to this object.
- Use the `instanceof` operator to check if the argument has the correct type.
- Cast the argument to the correct type.
- For each "significant" field in the class, check if that field of the argument matches the corresponding fiedl of this object.
- When you are finished writing your `equals` method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?
- Always override `hasCode` when you override `equals`.
- Don't try to be too clever.
- Don't substitute another type for `Object` in the `equals` declaration.
