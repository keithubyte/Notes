### Item40 : Design method signatures carefully

----------

- Choose method names carefully
Names should always obey the standard naming conventions.

- Don't go overboard in providing convenience methods
Every method should "pull its weight". Too many methods make difficult to learn, use, document, test, and maintain.

- Avoid long parameter lists
Aim for four parameters or fewer. There are three techniques for shortening overly long parameter lists.
	1. Break the method up into multiple methods, each of which requires only a subset of the parameters.
	2. Create *helper class* to hold groups of parameters.
	3. Adapt the *Builder Pattern* from object construction to method invocation.

- For parameter types, favor interfaces over classes

- Prefer two-element enum types to boolean parameters