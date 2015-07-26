### Item13 : Minimize the accessibility of classes and members

----------

The rule of thumb is simple: **make each class or member as inaccessible as possible.**

If a package-private top-level class(or interface) is used by only one class, consider making the top-level class a private nested class of the sole class that used it. This reduce its accessibility from all the classes in its package to the one class that uses it. But it is far more important to reduce the accessibility of a gratuitously public class than of a package-privete top-level class: the public class is part of the package's API, while the package-private top-level class is already part of its implementation.

There are four possible access levels, listed here i order of increasing accessibility:

- private
The member is accessible only from the top-level class where it is declared.

- package-private/default
The member is accessible from any class in the package where it is declared.

- protected
The member is accessible from subclasses of the class where it is declared and from any class in the package where it is declared.

- public
The member is accessible from anywhere.

After carefully designing your class's public API, you reflex should be to make all other members private. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private. If you find youself doing this often, you should reexamine the design of your system to see if another decomposition might yield classes that are better decoupled from one another.

For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. A protecte member is part of the class's exported API and must be supported forever. Also, a protected member of an exported class represents a public commitment to an implementation detail.

There is noe rule that restricts your ability ro reduce the accessibility of methods. If a method overrides a superclass method, it is not permitted to have a lower access level in the subclass than it does in the superclass. This is necessary to ensure that an instance of the sublcass is usable anywhere that an instance of the superclass is usable.

You can expose constants via public static final fields, assuming the constants from an integral part of the abstraction provided by the class. It is critical that these contain either primitive values or refrences to immutable objects. A final filed containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified -- with disastrous results.

Note that a nonzero-length array is always mutable, so **It is wrong for a class to have a public static final array field, or an accessor that returns such a field.** If a class has such field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes:

```java
// Protential security hole!
public static final Thing[] VALUES = {...};
```

There are 2 ways to fix the problem. You can make the public array private and add a public immutable list:

```java
private static final Thine[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

Alternatively, you can make the array private and add a public method that returns a copy of a private array:

```java
private static final Thine[] PRIVATE_VALUES = { ... };
public static final thing[] values() {
	return PRIVATE_VALUES.clone();
}
```

To choose between these alternatives, think about what the client is likely to do with the result.

#### Summary

To summarize, you should always reduce accessibility as much as possible. After carefully designing a minimal public API, you should prevent any stray classes, interfaces , or members from becoming a part of the API. With the eception of public static final fields, public classes should have no public fileds. Ensure that objects referenced by public static final fields are immutable.