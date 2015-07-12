### Prefer for-each loops to traditional for loops

The `for-each` loop provides compelling advantages over the traditional `for` loop in clarity and bug prevention, with no performance penalty. You should use it wherever you can. Unfortunately, there are three common situations where you can't use a `for-each` loop:

- Filtering
If you need to traverse a collection and remove selected elements, then you need to use an explicit iterator so that you can call its `remove` method.
- Transforming
If you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to set the value of an element.
- Parallel iteration
If you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable, so that all iterators or index variables can be advanced in lockstep.

If you find yourself in any of these situations, use an ordinary for loop.