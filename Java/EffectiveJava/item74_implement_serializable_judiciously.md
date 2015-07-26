### Item74 : Implement Serializable judiciously

----------

Serialization provides the standard wire-level object representation for remote communication, and the standard persistent data format for the JavaBeans component architecture.

Allowing a class's instances to be serialized can be as simple as adding the words `implements Serializable` to its declaration. Because this is so easy to do, there is a common misconception that serialization requires little effort on the part of the programmer. The true is far more complex. While the immediate coast to make a class serializable can be negligible, the long-term costs are often substantial.

- **A major cost of implementing `Serializable` is that is decreases the flexibility to change a class's implementation once it has been released.** When a class implements `Serializable`, its byte-stream encoding (or serialized form) becomes part of its exported API.A simple example of the constraints on evolution that accompany serializability concerns *stream unique identifiers*, more commonly known as *serial version UIDs*. Every serializable class has a unique identification number associated with it. If you do not specify this number explicitly by declaring a static final `long` field named `serialVersionUID`, the system automatically generates it at runtime by applying a complex procedure to the class. The automatically generated value is affected by the class's name, the names of the interfaces it implements, and all of its public and protected members. If you change any of these things in any way, for example, by adding a trivial convenience method, the automatically generated serial version UID changes. If you fail to declare an explicit serial version UID, compatibility will be broke, resulting in an `InvalidClassException` at run time.

- **A second cost of implementing `Serializable` is that it increases the likelihood of bugs and security holds**. Normally, objects are created using constructors; serialization is an *extralinguistic mechanism* for creating objects. Whether you accept the default behavior or override it, deserialization is a "hidden constructor" with all of the sam issues as other constructors. Because there is no explicit constructor associated with deserialization, it is easy to forget that you must ensure that it guarantees all of the invariants established by the constructors and that it dose not allow an attacker to gain access to the internals of the object under construction. Relying on the default deserialization mechanism can easily leave objects open to invariant corruption and illegal access.

- **A third cost of implementing `Serializable` is that it increases the testing burden associated with releasing a new version of a class.** When a serializable class is revised, it is important to check that it is possible to serialize an instance in the new release and deserialize it in old releases, and vice versa. The amount of testing required is thus proportional to the product of the number of serializable classes and the number of releases, which can be large.

**Classes designed for inheritance should rarely implement `Serializable`**, and interfaces should rarely extend it. Classes designed for inheritance that do implement `Serializable` include `Throwable`, `Component`, and `HttpServlet`. `Throwable` implements `Serializable` so exceptions from remote method invocation can be passed from server to client. `Component` implements `Serializable` so GUIs can be sent, saved, and restored. `HttpServlet` implements `Serializable` so session state can be cached.