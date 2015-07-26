### Item41 : Use overloading judiciously

----------

The following program is a well-intentioned attempt to classify collections according to whether they are sets, lists, or some other kind of collection:

```java
// Broken - What does this program print?

public class CollectionClassifier {

	public static String classify(Set<?> s) {
		return "Set";
	}

	public static String classify(List<?> l) {
		return "List";
	}

	public static String classify(Collection<?> c) {
		return "Unknown Collection";
	}

	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>(),
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		}


		for(Collection<?> c : collections) {
			System.out.println(classify(c));
		}
	}

}
```

You might expect this program to print `Set`, followed by `List` and `Unknown Collection`, but it doesn't.  It prints `Unknown Collection` three times. This is because the `classify` method is *overloaded*, and **the choice of which overloading to invoke is made at compile time**.

The behavior of this program is counterintuitive because **selection among overloaded method is static, while selection among overridden methods is dynamic**.

The compile-time type of an object has no effect on which method is executed when an overridden method is invoked; the "most specific" overriding method always gets executed. Compare this to overloading, where the runtime type of an object has no effect on which overloading is executed; the selection is made at compile time, based entirely on the compile-time types of parameters.

Because overriding is the norm and overloading is the exception, overriding sets people's expectations for the behavior of method invocation. As demonstrated by the `CollectionClassifier` example, overloading can easily confound these expectations. It is bad practice to write code whose behavior is likely to confuse programmers. This is especially true for APIs. If the typical user of an API does not know which of several method overloadings will get invoked for a given set of parameters, use of the API is likely to result in errors. The errors will likely manifest themselves as erratic behavior at runtime, and many programmers will be unable to diagnose. Therefore you should **avoid confusing uses of overloading**.

Exactly what constitutes a confusing use of overloading is open to some debate. **A safe, conservative policy is never to export two overloadings with the same number of parameters**. If a method uses varargs, a conservative policy is not to overload it at all. The restrictions are not terribly onerous because you can always give methods different names instead of overloading them.

Prior to release 1.5, all primitive types were radically different from all reference types, but this is no longer true in the presence of autoboxing, and it has caused real trouble. Consider the following program:

```java
public class SetList {
	public static void main(String[] args) {
		Set<Integer> set = new TreeSet<Integer>();
		List<Integer> list = new ArrayList<Integer>();
		
		for(int i = -3; i < 3; i++) {
			set.add(i);
			list.add(i);
		}

		for(int i = 0; i < 3; i++) {
			set.remove(i);
			list.remove(i);
		}

		System.out.println(set + " " + list);
	}	
}
```

The program adds the integers from -3 to 2 to a sorted set and to a list, and then makes three identical calls to remove on both the set and the list. You may expect the program to remove the non-negative values from the set and the list. In fact, the program removes the non-negative values from the set and the odd values from the list and prints `[-3, -2, -1] [-2, 0, 2]`.

The call to `set.remove(i)` selects the overloading `remove(E)`, where `E` is the element type of the set, and autoboxes `i` from `int` to `Integer`. This is the behavior you'd expect, so the program ends up removing the positive values from the set. The call to `list.remove(i)`, on the other hand, selects the overloading `remove(int i)`, which removes the element at the specified *position* from a list. You can fixed it by this:

```java
for(int i = 0; i < 3; i++) {
	set.remove(i);
	list.remove((Integer) i);
}
```

So, adding generics and autoboxing to the language damaged the `List` interface.

#### Summary

To summarize, just because you can overload methods doesn't mean you should. You should generally refrain from overloading methods with multiple signatures that have the same number of parameters. In some cases, especially where constrcutors are involved, it may be impossible to follow this advice. In that case, you should at least avoid situations where the same set of parameters can be passed to different overloadings by the addition of casts. If such a situation cannot be avoided, for example, because you are retrofitting an existing class to implement a new interface, you should ensure that all overloadings behave identically when passed the same parameters. If you fail to do this, programmers will be hard pressed to make effective use of the overloaded method or constructor, and they won't understand why it doesn't work.