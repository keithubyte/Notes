### Item52 : Refer to objects by their interfaces

----------

**If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types. If you get into the habit of using interfaces as types, your program will be much more flexible.**

But there are serval exceptions:

- It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists. For example, consider *value classes*, such as `String` and `BigInteger`. Value classes are rarely written with multiple implementations in mind.
- A second case in which there is no appropriate interface type is that of objects belonging to a framework whose fundamental types are classes rather than interfaces. If an object belongs to such a *class-based framework*, it is preferable to refer to it by the relevant *base class*.
- A final case in which there is no appropriate interface type is that of classes that implement an interface but provide extra methods not found in the interface.

These cases are not meant to be exhaustive but merely to convey the flavor of situations where it is appropriate to refer to an object by its class.