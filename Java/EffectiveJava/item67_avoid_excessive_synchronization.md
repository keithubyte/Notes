### Item67 : Avoid excessive synchronization

----------

**To avoid liveness and safety failures, never cede control to the client within a synchronized method or block**. In other words, inside a synchronized region, do not invoke a method that is designed to be overridden, or one provided by a client in the form of a function object. From the perspective of the class with the synchronized region, such methods are *aline*. The class has no knowledge of what the method does and has no control over it. Depending on what an alien method does, calling it from a synchronized region can cause exceptions, deadlocks, or data corruption.

To make it concrete, consider the following class, which implements an *observable* set wrapper. It allows clients to subscribe to notifications when elements are added to the set:

```java
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
	public ObservableSet(Set<E> set) {
		super(set);
	}

	private final List<SetObserver<E>> observers = new ArrayList<SetObserver<E>>();

	public void addObserver(SetObserver<E> observer) {
		synchronized(observers) {
			observers.add(observer);
		}
	}

	public boolean removeObserver(SetObserver<E> observer) {
		synchronized(observers) {
			observers.remove(observer);
		}
	}

	private void notifyElementAdded(E element) {
		synchronized(observers) {
			for (SetObserver<E> observer : observers) {
				observer.added(this, element);
			}
		}
	}

	@Override
	public boolean add(E element) {
		boolean added = super.add(element);
		if(added) {
			notifyElementAdded(element);
		}
		return added;
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		boolean result = false;
		for (E element : c) {
			result |= add(element); // calls notifyElementAdded
		}
		return result;
	}
}
```

Observers subscribe to notifications by invoking the `addObserver` method and unsubscribe by invoking the `removeObserver` method. In both case, an instance of this *callback* interface is passed to the method:

```java
public interface SetObserver<E> {
	// Invoked when an element is added to the observable set
	void added(ObservableSet<E> set, E element);
}
```

On cursory inspection, `ObservableSet` appears to work. For example, the following program prints the numbers from 0 through 99:

```java
public static void mani(String[] args) {
	ObservableSet<Integer> set = new ObservableSet<Integer>(new HashSet<Integer>());

	set.addObserver(new SetObserver<Integer>)() {
		public void added(ObservableSet<Integer> s, Integer s) {
			System.out.println(e);
		}
	});

	for (int i = 0; i < 100; i++) {
		set.add(i);
	}
}
```

Now suppose we replace the `addObserver` call with one that passes an observer that prints the `Integer` value that was added to the set and removes itself if the value is 23:

```java
set.addObserver(n)	set.addObserver(new SetObserver<Integer>)() {
		public void added(ObservableSet<Integer> s, Integer s) {
			System.out.println(e);
			if(e == 23) s.removeObserver(this);
		}
	});
```

You might expect the program to print the numbers 0 through 23, after which the observer would unsubscribe and the program complete its work silently. What actually happens is that it prints the numbers 0 through 23, and then throws a `ConcurrentModificationException`. The problem is that `notifyElementAdded` is in the process of iterating over the `observers` list when it invokes the `observer`'s `added` method.The `added` method calls the observer set's `removeObserver` method, which in turn calls `observers.remove`. Now we are in trouble. We are trying to remove an element from a list in the midst of iterating over it, which is illegal. The iteration in the `notifyElementAdded` method is in a synchronized block to prevent concurrent modification, but it doesn't prevent the iterating thread itself from calling back into the observable set and modifying its `observers` list. 

Now let's try something odd: let's write an observer that attempts to unsubscribe, but instead of calling `removeObserver` directly, it engages the services of another thread to do the deed. This observer uses an *executor service*:

```java
// Observer that uses a background thread needlessly
set.addObserver(n)	set.addObserver(new SetObserver<Integer>)() {
		public void added(ObservableSet<Integer> s, Integer s) {
			System.out.println(e);
			if(e == 23) {
				ExecutorService executor = Executors.newSingleThreadExecutor();
				final SetObserver<Integer> observer = this;
				try {
					executor.submit(new Runnable() {
						public void run() {
							s.removeObserver(observer);
						}
					}).get();
				} catch(ExecutionException ex) {
					throw new AssertionError(ex.getCause());
				} catch(InterruptedException ex) {
					throw new AssertionError(ex.getCause());
				} finally {
					executor.shutdown();
				}
			}
		}
	});
```

This time we don't get an exception; we get a deadlock. The background thread calls `s.removeObserver`, which attempts to lock `observers`, but it can't acquire the lock, because the main thread already has the lock. All the while, the main thread is waiting for the background thread to finish removing the observer, which explains the deadlock.

#### Summary

In summary, to avoid deadlock and data corruption, never call an alien method from within a synchronized region. More generally, try to limit the amount of work that you do from within synchronized regions. **As a rule, you should do as little work as possible inside synchronized regions**.When you are designing a mutable class, think about whether it should do its own synchronization. 