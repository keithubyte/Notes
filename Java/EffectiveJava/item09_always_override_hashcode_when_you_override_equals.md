### Item 09 : Always override hashCode when you override equals

----------

**You must override hashCode in every class that overrides equals.** Failure to do so will result in a violation of the general contract for `Object.hashCode`, which will prevent your class from functioning properly in conjunction with all hash-based collections, including `HashMap`, `HashSet` and `Hashtable`.

Here is the contract, copied from the `Object` specification [JavaSE6]:

- Whenever it is invoked on the same object more than once during an execution of an application, the `hashCode` method must consistently return the same interger, provided on information used in `equlas` comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.

- If two objects are equal according to the `equals(Object)` method, then calling the `hashCode` method on each of the two objects must produce the same integer result.

- It is not required that if two objects are unequal according to the `equals(Object)` method, then calling the `hashCode` method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

**The key provision that is violated when you fail to override hashCode is the second one: equal objects must have equal hash codes.** Two distinct instances may be logically equal according to a class's `equals` method, but to `Object's hashCode` method, they're just two objects with nothing much in common. Therefore `Object's hashCode` method returns two seemingly random numbers instead of two equal numbers as required by the contract.

If a class is immutable and the cost of computing the hash code is significant, you might consider caching the hash code in the object rather than recalculating it each time it is requested. If you believe that most objects of this type will be use as hash keys, then you should calculate the hash code when the instance is created. Otherwise, you might choose to *layily initialize* it the first time `hashCode` is invoked.

```java
/**
 * Lazily initialized, cached hashCode
 * Created by keith on 15/3/18.
 */
public class PhoneNumber {
    private volatile int hashCode;
    
    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            // Calculate hash code
            hashCode = result;
        }
        return result;
    }
}
```

**Do not be tempted to exclude significant prts of an object from the hash code computation to improve performance.** While the resulting hash function may run faster, its poor quality may degrade hash tables' performance to the point where they become unusably slow. In particular, the hash function may, in practice, be confronted with a large collection of instances that differ largely in the regions that you've chosen to ignore. If this happens, the hash function will map all the instances to a very few hash codes, and hash-based colections will display quadratic performance. This is not just a theoretical problem. The `String` hash function implemented in all releases prior to 1.2 examined at most sixteen characters, evenly spaced throughout the string, starting with the first character. For large collections of hierarchical names, such as URLs, this hash function displayed exactly the pathological behavior noted here.
