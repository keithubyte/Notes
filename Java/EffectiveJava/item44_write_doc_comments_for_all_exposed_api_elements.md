### Write doc comments for all exposed API elements

If an API is to be usable, it must be documented. If you are not familiar with the doc comment conventions, you should learn them. These conventions are described on [How to Write Doc Comments for the Javadoc Tool](http://www.oracle.com/technetwork/articles/java/index-137868.html).

To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment.

**The doc comment for a method should describe succinctly the contract between the method and its client.** With the exception of methods in classes designed for inheritance, the contract should say what the method does rather than how it does its job. The doc comment should enumerate all of the method's *preconditions* , which are the things that have to be true in order for a client to invoke it, and its *postconditions*, which are the things that will be true after the invocation has completed successfully. Typically, preconditions are described implicitly by the `@throws` tags for unchecked exceptions; each unchecked exception corresponds to a precondition violation. Also, preconditions can be specified along with the affected parameters in their `@param` tags.

In addition to preconditions and postconditions, methods should document any *side effects*. A side effect is an observable change in the state of the system that is not obviously required in order to achieve the postcondition.

To describe a method's contract fully, the doc comment should have an `@param` tag for every parameter, an `@return` tag unless the method has a void return type, and an `@throws` tag for every exception thrown by the method, whether checked or unchecked. By convention, the text following an `@param` tag or `@return` tag should be a noun phrase describing the value represented by the parameter or return value. The text following an `@throws` tag should consist of the word "if", followed by a clause describing the conditions under which the exception is thrown. Occasionally, arithmetic expressions are used in place of noun phrases. By convention, the phrase or clause following an `@param`, `@return`, or `@throws` tag is not terminated by a period. All of these conventions are illustrated by the following short doc comment:

```java
/**
 * Returns the element at the specified position in this list.
 *  
 *  <p>This method is <i>not</i> guaranteed to run in constant time.
 *  In some implementations it may run in time proportional to 
 *  the element position.
 *   
 *  @param  index index of element to return; must be 
 *          non-negative and less than the size of this list
 *  @return the element at the specified position in this list
 *  @throws IndexOutOfBoundsException if the index is out of 
 *          range ({@code index < 0 || index >= this.size()})
 */
```

Notice the use of `HTML` tags in this doc comment. The Javadoc utility translates doc comments into `HTML`, and arbitrary `HTML` elements in doc comments end up in the resulting `HTML` document. Occasionally, programmers go so far as to embed `HTML` tables in their comments, although this is rare.

Also notice the use of the Javadoc `{@code}` tag around the code fragment in the `@throws` clause. This serves two purpose: it causes the code fragment to be rendered in *code font*, and it suppresses processing of `HTML` markup and nested Javadoc tags in the code fragment. The latter property is what allows us to use the less-than sign(<) in the code fragment even though it's an `HTML`  metacharacter.

Finally, notice the use of the word "this" in the doc comment. By convention, the word "this" always refers to the object on which the method is invoked when it is used in the doc comment for an instance method.

Don't forget that you must take special action to generate documentation containing `HTML` metacharacters, such as the less-than sign(<), the greater-than sign(>), and the ampersand(&). The best way to get these characters into documentation is to surround them with the `{@literal}` tag, which suppress processing of `HTML`  markup and nested Javadoc tags. It is like the `{@code}` tag, except that it doesn't render the text in code font. For example, this Javadoc fragment:

```java
/**
 * The triangle inequality is {@literal |x + y| < |x| + |y|}.
 */
```

produces the documentation: *"The triangle inequality is |x + y| < |x| + |y|."*

To summarize, documentation comments are the best, most effective way to document your API. Their use should be considered mandatory for all exported API elements. Adopt a consistent style that adheres to standard conventions. Remember that arbitrary `HTML` is permissible within documentation comments and that `HTML` metacharacters must be escaped.