### Item24 : Eliminate unchecked warnings

----------

When you program with generics, you will see many compiler warnings: unchecked cast warnings, unchecked method inbocation warnings, unchecked generic array creation warnings, and unchecked conversion warnings.

When you get warnings that require some thought, perserver! **Eliminate every unchecked warning that you can.** If you eliminate all warnings, you are assured that your code is typesafe, which is a very good thing. It means that you won't get a `ClassCastException` at runtime, and it increases your confidences that your program is behaving as you intended.

**If you can't eliminate a warning, and you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an @SuppressWarnings("unchecked") annotation.**

**Always use the SuppressWarnings annotation on the smallest scope possible.** Never use SuppressWarnings on the entire class. Doing so could mask critical warnings.

**Every tiem you use an @SuppressWarnings("unchecked)" annotation, add a comment saying why it's safe to do so.** This will help others understand the code, and more importantly, it will decrease the odds that someone will modify the code so as to make the computation unsafe, If you find it hard to write such a comment, keep thinking. You may end up figuring out that the unchecked operation isn't safe after all.

#### Summary

In summary, unchecked warnings are important. Don't ignore them. Every unchecked warning represents the potential for a `ClassCastException` at runtime. Do your best to eliminate these warnings. If you can't eliminate an unchecked warning and you can prove that the code the provoked it is typesafe, suppress the warning with an @SuppressWarnings("unchecked") annotation in the narrowest possible. Record the rationale for your decision to suppress the warning in a comment.