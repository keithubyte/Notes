### Item69 : Prefer concurrency utilities to wait and notify

----------

The concurrent collections provide high-performance concurrent implementations of standard collection interfaces such as `List`, `Queue`, and `Map`. To provide high concurrency, these implementations manage their own synchronization internally. Therefore, **it is impossible to exclude concurrent activity from a concurrent collection; locking it will have no effect but to slow the program**.