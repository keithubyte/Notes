### Consider using a custom serialized form

**Do not accept the default serialized form without first considering whether it is appropriate**. Accepting the default serialized form should be a conscious decision that this encoding is reasonable from the standpoint of flexibility, performance, and correctness. Generally speaking, you should accept the default serialized form only if it is largely identical to the encoding that you would choose if you were designing a custom serialized form.

The default serialized form of an object is a reasonably efficient encoding of the *physical* representation of the object graph rooted at the object. In other words, it describes the data contained in the object and in every object that is reachable from this object. It also describes the topology by which all of these objects are interlinked. The ideal serialized form of an object contains only the *logical* data represented by the object. It is independent of the physical representation.

**The default serialized form is likely to be appropriate if an object's physical representation is identical to its logical content**. For example, the default serialized form would be reasonable for the following class, which simplistically represents a person's name:

```java
// Good candidate for default serialized form
public class Name implements Serializable {

    /**
     * First name. Must be non-null.
     * @serial
     */
    private final String firstName;

    /**
     * Middle name.
     * @serial
     */
    private final String middleName;

    /**
     * Last name. Must be non-null.
     * @serial
     */
    private final String lastName;
    
    public Name(String firstName, String middleName, String lastName) {
        this.firstName = firstName;
        this.middleName = middleName;
        this.lastName = lastName;
    }
}
```

Logically speaking, a name consists of three strings that represent a first name, a middle name, and a last name. The instance fields in `Name` precisely mirror this logical content.

**Even if you decide that the default serialized form is appropriate, you often must provide a `readObject` method to ensure invariants and security**. In the case of `Name`, the `readObject` method must ensure that `firstName` and `lastName` are non-null. 

Note that there are documentation comments on the fields, even though they are private. That is because these private fields define a public API, which is the serialized form of the class, and this public API must be codumented. The presence of the `@serial` tag tells the Javadoc utility to place this documentation on a special page that documents serialized forms.

Near the opposite end of the spectrum from `Name`, consider the following class, which represents a list of strings:

```java
// Awful candidate for default serialized form
public final class StringList implements Serializable {
	private int size = 0;
	private Entry head = null;

	private static class Entry implements Serializable {
		String data;
		Entry next;
		Entry previous;
	}

	... // Remainder omitted
}
```

Logically speaking, this class represents a sequence of strings. Physically, it represents the sequence as a doubly linked list. If you accept the default serialized form, the serialized form will painstakingly mirror every entry in the liked list and all the links between the entries, in both directions.

**Using the default serialized form when an object's physical representation differs substantially from its logical data content has four disadvantages**:

- It permanently ties the exported API to the current internal representation.
In the above example, the private *StringList.Entry* class becomes part of the public API. If the representation is changed in a future release, the *StringList* class will still need to accept the linked representation on input and generate it on output. The class will never be rid of the code dealing with linked entries, even if it doesn't use them anymore.

- It can consume excessive space.
In the above example, the serialized form unnecessarily represents each entry in the linked list and all the links. These entries and links are mere implementation details, not worthy of inclusion the serialized form. Because the serialized form is excessively large, writing it to disk or sending it across the network will be excessively slow.

- It can consume excessive time.
The serialization logic has no knowledge of the topology of the object graph, so it must go through an expensive graph traversal. In the example above, it would be sufficient simply to follow the `next` references.

- It can cause stack overflows.
The default serialization procedure performs a recursive traversal of the object graph, which can cause stack overflows even for moderately sized object graphs. Serializing a `StringList` instance with 1,258 elements causes the stack to overflow on my machine.

A reasonable serialized form for `StringList` is simply the number of strings in the list, followed by the strings themselves. This constitutes the logical data represented by a `StringList`, stripped of the details of its physical representation. Here is a revised version of `StringList` containing `writeObject` and `readObject` methods implementing this serialized form (as a reminder, the `transient` modifier indicates that an instance field is to be omitted form a class's default serialized form):

```java
/**
 * StringList with a reasonable custom serialized form
 */
public class StringList implements Serializable{
    
    private transient int size = 0;
    private transient Entry head = null;
    
    // No longer serializable!
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }    
    
    // Appends the specified string to the list
    public final void add(String s) {}
    
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        
        // Write out all elements in the proper order.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // Read in all elements and insert them in list
        for (int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }
    
}
```

Note that the first thing `writeObject` does is to invoke `defaultWriteObject`, and the first thing `readObject` does is to invoke `defaultReadObject`, even though all of `StringList`'s fields are transient. **If all instance fields are `transient`, it is technically permissible to dispense with invoking `defaultWriteObject` and `defaultReadObject`, but it is no recommended**. Even if all instance fields are `transient`, invoking `defaultWriteObject` affects the serialized form, resulting in greatly enhanced flexibility. The resulting serialized form makes it possible to add nontransient instance fields in a later release while preserving backward and forward compatibility. 

**Regardless of what serialized form you choose, declare an explicit serial version UID in every serializable class you write**. This eliminates the serial version UID as a potential source of incompatibility. There is also a small performance benefit. If no serial version UID is provided, an expensive computation is required to generate one at runtime.

To summarize, when you have decided that a class should be serializable, think hard about what the serialized form should be. Use the default serialized form only if it is a reasonable description of the logical state of the object; otherwise design a custom serialized form that aptly describes the object.