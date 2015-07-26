### Item43 : Return empty arrays or collections, not nulls

----------

It is not uncommon to see methods that look something like this:

```java
private final List<Cheese> cheeseInSotck = ...;

/**
 * @return
 * an array containing all of the cheeses in the shop, 
 * or null if no cheeses are available for purchase.
 */
public Cheese[] getCheeses() {
	if(cheesesInStock.size() == 0) {
		return null;
	}
	...
}
```

There is no reason to make a special case for the situation where no cheeses are available for purchase. Doing so requires extra code in the client to handle the null return value, for example:

```java
Cheese[] cheeses = shot.getCheeses();
if(cheeses != null && Arrays.asList(cheeses).contains(Cheese.STILTON){
		System.out.println("Jolly good, just the things.");
}
```

instead of:

```java
if(Arrays.asList(cheeses).contains(Cheese.STILTON){
		System.out.println("Jolly good, just the things.");
}
```

It it sometimes argued that a null return value is preferable to an empty array because it avoids the expense of allocating the array. This argument fails on two counts. First, it is inadvisable to worry about performance at this level unless profiling has shown that the method in question is a real contributor to performance problems. Second, it is possible to return the same zero-length array from every invocation that returns on item because zero-length arrays are immutable and immutable objets may be shared freely.

```java
// The right way to return an array from a collection
private final List<Cheese> cheesesInStock = ...;

private static final Classes[] EMPTY_CHEESE_ARRAY = new Cheese[0];

/**
 * @return
 * an array containing all of the cheeses in the shop.
 */
public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}

```

In this idiom, an empty-array constant is passed to the `toArray` method to indicate the desired return type. Normally the `toArray` method allocates the returned array, but if the collection is empty, it fits in the zero-length input array, and the specification for `Collection.toArray(T[])` guarantees that the input array will be returned if it is large enough to hold the collection. Therefore the idiom never allocates an empty array.

In similar fashion, a collection-valued method can be made to return the same immutable empty collection every time it needs to return an empty collection. The `Collections.emptySet`, `emptyList`, and `emptyMap` methods provide exactly what you need:

```java
// The right way to return a copy of a collection
public List<Cheese> getCheeseList() {
	if(cheesesInStock.isEmpty) {
		return Collections.emptyList();
	} else {
		return new ArrayList<Cheese>(cheesesInStock);
	}
}
```

#### Summary

In summary, there is no reason ever to return null from an array-or-collection-valued method instead of returning an empty array or collection.