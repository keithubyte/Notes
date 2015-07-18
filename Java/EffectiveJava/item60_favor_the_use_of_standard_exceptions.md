### Favor the use of standard exceptions

Reusing preexisting exceptions has several benefits. Chief among these, it makes your API easier to learn and use because it matches established conventions with which programmers are already familiar. A close second is that programs using your API are easier to read because they aren't cluttered with unfamiliar exceptions. Last (and least), fewer exception classes mean a smaller memory footprint and less time spent loading classes.

| Exception | Description |
|---- | ---- |
| IllegalArgumentException | Throw when the caller passes in an argument whose value is inappropriate. |
| IllegalStateException | Throw if the invocation is illegal because of the state of the receiving object. |

Another commonly reused exceptions are `NullPointerException`, `IndexOutOfBoundsException`, `ConcurrentModificationException`, `UnsupportedOperationException`.