### Item66 : Synchronize access to shared mutable data

----------

The *synchronized* key word ensures that only a single thread can execute a method or block at one time. Many programmers think of synchronization solely as a means of mutual exclusion, to prevent an object from being observed in an inconsistent state while it's being modified by another thread.

This view is correct, but it's only half the story. Without synchronization, one thread's changes might not be visible to other threads. Not only does synchronization prevent a thread from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock.

The language specification guarantees that reading or writing a variable is *atomic* unless the variable is of type `long` or `double`. In other words, reading a variable other than a `long` or `double` is guaranteed to return a value that was stored into that variable by some thread, even if multiple threads modify the variable concurrently and without synchronization.

While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another. **Synchronization is required for reliable communication between threads as well as for mutual exclusion**. This is due to a part of the language specification known as the *memory model*, which specifies when and how changes made by one thread become visible to others.

The consequences of failing to synchronize access to shared mutable data can be dire even if the data is atomically readable and writable. Consider the task of stopping one thread from another. Because reading and writing a `boolean` field is atomic, some programmers dispense with synchronization when accessing the field:

```java
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while (!stopRequested) {
                    System.out.println("i = " + i++);
                }
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(5L);

        stopRequested = true;
    }
}
```

You might expect this program to run for about 5 seconds, after which the main thread sets `stopRequested` to `true`, causing the background thread's loop to terminate. Actually, the result is different in different machine. It may stop in 5 seconds later, but on the other hand, it may never stop!

The problem is that in the absence of synchronization, there is no guarantee as to when, if ever, the background thread will see the change in the value of `stopRequested` that was mad by the main thread. In the absence of synchronization, it's quite acceptable for the virtual machine to transform this code:

```java
while(!done)
	i++;
```

into this code:

```java
if(!done) 
	while(true)
		i++;
```

This optimization is known as *hoisting*, and it is precisely what the HotSpot server VM does. The result is a *liveness failure:* the program fails to make progress. One way to fix the problem is to synchronize access to the `stopRequested` field. This program terminates in about 5 seconds, as expected:

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while (!stopRequested()) {
                    System.out.println("i = " + i++);
                }
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(5L);

        requestStop();
    }
}
```

Note that both the write method and the read method are synchronized. It is not sufficient to synchronize only the write method! In fact, **synchronization has no effect unless both read and write operations are synchronized**.

The actions of the synchronized methods in `StopThread` would be atomic even without synchronization. In other words, the synchronization on these methods is used solely for its communication effects, not for mutual exclusion. While the cost of synchronizing on each iteration of the loop is small, there is a correct alternative that is less verbose and whose performance is likely to be better: The locking in the second version of `StopThread` can be omitted if `stopRequested` is declared volatile. While the `volatile` modifier performs no mutual exclusion,  it guarantees that any thread that reads the field will see the most recently written value:

```java
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while (!stopRequested) {
                    System.out.println("i = " + i++);
                }
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(5L);

        stopRequested = true;
    }
}
```

You do have to be careful when using `volatile`. Consider the following method, which is supposed to generate serial numbers:

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++;
}
```

The method won't work properly without synchronization. The problem is that the increment operator (++) is not atomic. It performs 2 operations on the `nextSerialNumber` field: first it reads the value, then is writes back a new value, equal to the old value plus one. If a second thread reads the field between the time a thread reads the old value and writes back a new one, the second thread will see the same value as the first and return the same serial number. This is a *safety failure*: the program computes the wrong results.

One way to fix the `generateSerialNumber` method is to add the `synchronized` modifier to its declaration and remove the `volatile` modifier.

A better way is :

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
	return nextSerialNum.getAndIncrement();
}
```

The best way to avoid the problems discussed in this item is not to share mutable data. Either share immutable data, or don't share at all. In other words, **confine mutable data to a single thread**.

It is acceptable for one thread to modify a data object for a while and then to share it with other threads, synchronizing only the act of sharing the object reference. Other threads can then read the object without further synchronization, so long as it isn't modified again. Such objects are said to be **effectively immutable**. Transferring such an object reference from one thread to others is called **safe publication**. There are many ways to safely publish an object reference: you can store it in a static field as part of class initialization; you can store it in a volatile field, a final field, or a field that is accessed with normal locking; or you can put it into a concurrent collection.

#### Summary

In summary, **when multiple threads share mutable data, each thread that reads or writes the data must perform synchronization**. Without synchronization, there is no guarantee that one thread's changes will be visible to another. The penalties for failing to synchronize shared mutable data are liveness and safety failures. These failures are among the most difficult to debug. They can be intermittent and timing-dependent, and program behavior can vary radically from one VM to another. If you need only inter-thread communication, and not mutual exclusion, the `volatile` modifier is an acceptable form of synchronization,  but it can be tricky to use correctly.