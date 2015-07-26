### Item65 : Don't ignore exceptions

----------

**An empty `catch` block defeats the purpose of exceptions**, which is to force you to handle exceptional conditions. Ignoring an exception is analogous to ignoring a fire alarm - and turing it off so no one else gets a chance to see if there's a real fire. You may get away with it, or the results may be disastrous. Whenever you see an empty `catch` block, alarm bells should go off in your head. **At the very least, the `catch` block  should contain a comment explaining why it is appropriate to ignore the exception**.

An example of the sort of situation where is might be appropriate to ignore an exception is when closing a `FileInputStream`. You haven't changed the state of the file, so there's no need to perform any recovery action, and you've already read the information that you need from the file, so there's no reason to abort the operation in progress. Even in this case, it is wise to log the exception, so that you can investigate the matter if these exceptions happen often.