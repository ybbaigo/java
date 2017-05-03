# Java

@(topic)

> This repo is an **organized collection** of resources to help you learn java.


* [Concurrency](#concurrency)
	* [Processes and Threads](#processes-and-threads)
	* [Thread Objects](#thread-objects)
	* [Synchronization](#synchronization)
	* [Liveness](#liveness)
* [Singleton](#singleton)




## Concurrency

### Processes and Threads

In concurrent programming, there are two basic units of execution: processes and threads. In the Java programming language, concurrent programming is mostly concerned with threads. However, processes are also important.

**Processes**. A process has a self-contained execution environment. A process generally has a complete, private set of basic run-time resources; in particular, each process has its own memory space. Most implementations of the Java virtual machine run as a single process. A Java application can create additional processes using a ProcessBuilder object.

**Threads**. Threads are sometimes called lightweight processes. Both processes and threads provide an execution environment, but creating a new thread requires fewer resources than creating a new process. Threads exist within a process — every process has at least one. Threads share the process's resources, including memory and open files. This makes for efficient, but potentially problematic, communication. From the application programmer's point of view, you start with just one thread, called the main thread. This thread has the ability to create additional threads.

##### Source(s) and further reading

* [Processes and Threads](http://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html)

### Thread Objects

![Life Cycle of a Thread](https://www.tutorialspoint.com/java/images/Thread_Life_Cycle.jpg)

There are two ways to creates an instance of Thread: `Provide a Runnable object` and `Subclass Thread`

```java
public class HelloRunnable implements Runnable {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }

}

public class HelloThread extends Thread {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }

}
```

A number of methods useful for thread management:

* **Thread.sleep**, causes the current thread to suspend execution for a specified period. This is an efficient means of making processor time available to the other threads of an application or other applications that might be running on a computer system.
* **Thread.interrupt**, An interrupt is an indication to a thread that it should stop what it is doing and do something else.
* **Thread.join**, allows one thread to wait for the completion of another.

The SimpleThreads Example
```java
public class SimpleThreads {

    // Display a message, preceded by
    // the name of the current thread
    static void threadMessage(String message) {
        String threadName =
            Thread.currentThread().getName();
        System.out.format("%s: %s%n",
                          threadName,
                          message);
    }

    private static class MessageLoop
        implements Runnable {
        public void run() {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };
            try {
                for (int i = 0;
                     i < importantInfo.length;
                     i++) {
                    // Pause for 4 seconds
                    Thread.sleep(4000);
                    // Print a message
                    threadMessage(importantInfo[i]);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }

    public static void main(String args[])
        throws InterruptedException {

        // Delay, in milliseconds before
        // we interrupt MessageLoop
        // thread (default one hour).
        long patience = 1000 * 60 * 60;

        // If command line argument
        // present, gives patience
        // in seconds.
        if (args.length > 0) {
            try {
                patience = Long.parseLong(args[0]) * 1000;
            } catch (NumberFormatException e) {
                System.err.println("Argument must be an integer.");
                System.exit(1);
            }
        }

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();

        threadMessage("Waiting for MessageLoop thread to finish");
        // loop until MessageLoop
        // thread exits
        while (t.isAlive()) {
            threadMessage("Still waiting...");
            // Wait maximum of 1 second
            // for MessageLoop thread
            // to finish.
            t.join(1000);
            if (((System.currentTimeMillis() - startTime) > patience)
                  && t.isAlive()) {
                threadMessage("Tired of waiting!");
                t.interrupt();
                // Shouldn't be long now
                // -- wait indefinitely
                t.join();
            }
        }
        threadMessage("Finally!");
    }
}
```
##### Source(s) and further reading

* [Thread Objects](http://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html)
* [Java - Multithreading](https://www.tutorialspoint.com/java/java_multithreading.htm)

### Synchronization

**synchronized**, is all about different threads reading and writing to the same variables, objects and resources.  This is not a trivial topic in `Java`, but here is a quote from Sun:

> `synchronized` methods enable a simple strategy for preventing thread interference and memory consistency errors: if an object is visible to more than one thread, all reads or writes to that object's variables are done through synchronized methods.

*In a very, very small nutshell:* When you have two threads that are reading and writing to the same 'resource', say a variable named `foo`, you need to ensure that these threads access the variable in an atomic way.  Without the `synchronized` keyword, your thread 1 may not see the change thread 2 made to `foo`, or worse, it may only be half changed.  This would not be what you logically expect.


**synchronized methods**

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

* First, it is not possible for two invocations of synchronized methods on the same object to interleave. When one thread is executing a synchronized method for an object, all other threads that invoke synchronized methods for the same object block (suspend execution) until the first thread is done with the object.
* Second, when a synchronized method exits, it automatically establishes a happens-before relationship with any subsequent invocation of a synchronized method for the same object. This guarantees that changes to the state of the object are visible to all threads.

**synchronized statements**

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```
In this example, the addName method needs to synchronize changes to lastName and nameCount, but also needs to avoid synchronizing invocations of other objects' methods. (Invoking other objects' methods from synchronized code can create problems that are described in the section on Liveness.) Without synchronized statements, there would have to be a separate, unsynchronized method for the sole purpose of invoking nameList.add.

##### Source(s) and further reading

* [What does 'synchronized' mean?](http://stackoverflow.com/questions/1085709/what-does-synchronized-mean)
* [Synchronized Methods](http://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html)
* [Intrinsic Locks and Synchronization](http://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html)


### Liveness
A concurrent application's ability to execute in a timely manner is known as its liveness.

**Deadlock** describes a situation where two or more threads are blocked forever, waiting for each other. Here's an example.

```java
public class Deadlock {
    static class Friend {
        private final String name;
        public Friend(String name) {
            this.name = name;
        }
        public String getName() {
            return this.name;
        }
        public synchronized void bow(Friend bower) {
            System.out.format("%s: %s"
                + "  has bowed to me!%n", 
                this.name, bower.getName());
            bower.bowBack(this);
        }
        public synchronized void bowBack(Friend bower) {
            System.out.format("%s: %s"
                + " has bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new Runnable() {
            public void run() { alphonse.bow(gaston); }
        }).start();
        new Thread(new Runnable() {
            public void run() { gaston.bow(alphonse); }
        }).start();
    }
}
```
**Starvation**, describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress.

**Livelock**, A thread often acts in response to the action of another thread. If the other thread's action is also a response to the action of another thread, then livelock may result.

##### Source(s) and further reading

* [Deadlock](http://docs.oracle.com/javase/tutorial/essential/concurrency/deadlock.html)
* [Starvation and Livelock](http://docs.oracle.com/javase/tutorial/essential/concurrency/starvelive.html)


## Singleton

In software engineering, the singleton pattern is a software design pattern that restricts the instantiation of a class to one object. This is useful when exactly one object is needed to coordinate actions across the system.

The Right Way to Implement a Serializable Singleton in Java.

```java
public enum Elvis {
    INSTANCE;
    private final String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
This approach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. While this approach has yet to be widely adopted, a single-element enum type is the best way to implement a singleton.

Before release 1.5, there were two ways to implement singletons. Both are based on keeping the constructor private and exporting a public static member to provide access to the sole instance. In one approach, the member is a final field:

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```

In the second approach to implementing singletons, the public member is a static factory method.

```java
public final class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```
 and the second approach with **lazy initialization**, where the instance is created when the static method is first invoked
```java
public final class Singleton {
    private static volatile Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

##### Source(s) and further reading

* [Singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern)
* [What is an efficient way to implement a singleton pattern in Java?](http://stackoverflow.com/questions/70689/what-is-an-efficient-way-to-implement-a-singleton-pattern-in-java)
* [Effective Java Reloaded](https://sites.google.com/site/io/effective-java-reloaded)
* [Enforce the Singleton Property with a Private Constructor or an enum Type](http://www.drdobbs.com/jvm/creating-and-destroying-java-objects-par/208403883?pgno=3)