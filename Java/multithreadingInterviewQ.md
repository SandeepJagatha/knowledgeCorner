### What makes java application concurrent?
The very first class, you will need to make a java class concurrent, is `java.lang.Thread` class. This class is the basis of all concurrency concepts in java. Then you have `java.lang.Runnable` interface to abstract the thread behavior out of thread class.

Other classes you will need to build advance applications can be found at `java.util.concurrent` package added in Java 1.5.

### Java Multi-threading Evolution

#### JDK release-wise multi-threading concepts
As per JDK 1.x release, there were only few classes present in this initial release. To be very specific, there classes/interfaces were:

* `java.lang.Thread`
* `java.lang.ThreadGroup`
* `java.lang.Runnable`
* `java.lang.Process`
* `java.lang.ThreadDeath`
* and some exception classes
e.g.

1. `java.lang.IllegalMonitorStateException`
2. `java.lang.IllegalStateException`
3. `java.lang.IllegalThreadStateException.`

It also had few synchronized collections e.g. `java.util.Hashtable.`

**JDK 1.2 and JDK 1.3** had no noticeable changes related to multi-threading. (Correct me if I have missed anything).

**JDK 1.4,** there were few JVM level changes to suspend/resume multiple threads with single call. But no major API changes were present.

**JDK 1.5** was first big release after JDK 1.x; and it had included multiple concurrency utilities. `Executor`, `semaphore`, `mutex`, `barrier`, `latches`, `concurrent collections` and `blocking queues`; all were included in this release itself. The biggest change in java multi-threading applications cloud happened in this release.

[Read full set of changes in this link](http://docs.oracle.com/javase/1.5.0/docs/guide/concurrency/overview.html)

**JDK 1.6** was more of platform fixes than API upgrades. So new change was present in JDK 1.6.

**JDK 1.7** added support for `ForkJoinPool` which implemented work-stealing technique to maximize the throughput. Also `Phaser` class was added.

**JDK 1.8** is largely known for Lambda changes, but it also had few concurrency changes as well. Two new interfaces and four new classes were added in `java.util.concurrent` package e.g. `CompletableFuture` and `CompletionException.`

The Collections Framework has undergone a major revision in Java 8 to add aggregate operations based on the newly added streams facility and lambda expressions; resulting in large number of methods added in almost all Collection classes, and thus in concurrent collections as well.

[Read full set of changes in this link](http://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/changes8.html)

### Thread Safety?
A class is thread-safe when it continues to behave correctly when accessed from multiple threads.

### Object level Locking vs. Class level Locking in Java
Java supports multiple threads to be executed. This may cause two or more threads to access the same fields or objects. Synchronization is a process which keeps all concurrent threads in execution to be in synch. Synchronization avoids memory consistence errors caused due to inconsistent view of shared memory.

Synchronization in java is achieved using synchronized keyword. **You can use synchronized keyword in your class on defined methods or blocks. Keyword can not be used with variables or attributes in class definition.**

#### Object level locking
Object level locking is mechanism when you want to synchronize a non-static method or non-static code block such that only one thread will be able to execute the code block on given instance of the class. This should always be done to make instance level data thread safe. 

```java
public class DemoClass
{
    public synchronized void demoMethod(){}
}
 
or
 
public class DemoClass
{
    public void demoMethod(){
        synchronized (this)
        {
            //other thread safe code
        }
    }
}
 
or
 
public class DemoClass
{
    private final Object lock = new Object();
    public void demoMethod(){
        synchronized (lock)
        {
            //other thread safe code
        }
    }
}
```

#### Class level locking
Class level locking prevents multiple threads to enter in synchronized block in any of all available instances on runtime. This means if in runtime there are 100 instances of  DemoClass, then only one thread will be able to execute demoMethod() in any one of instance at a time, and all other instances will be locked for other threads. This should always be done to make static data thread safe.

```java
public class DemoClass
{
    public synchronized static void demoMethod(){}
}
 
or
 
public class DemoClass
{
    public void demoMethod(){
        synchronized (DemoClass.class)
        {
            //other thread safe code
        }
    }
}
 
or
 
public class DemoClass
{
    private final static Object lock = new Object();
    public void demoMethod(){
        synchronized (lock)
        {
            //other thread safe code
        }
    }
}
```

Some points :

* Synchronization in java guarantees that no two threads can execute a synchronized method which requires same lock simultaneously or concurrently.
* synchronized keyword can be used only with methods and code blocks. These methods or blocks can be static or non-static both.
* When ever a thread enters into java synchronized method or block it acquires a lock and whenever it leaves java synchronized method or block it releases the lock. Lock is released even if thread leaves synchronized method after completion or due to any Error or Exception.
* java synchronized keyword is re-entrant in nature it means if a java synchronized method calls another synchronized method which requires same lock then current thread which is holding lock can enter into that method without acquiring lock.
* Java Synchronization will throw NullPointerException if object used in java synchronized block is null. For example, in above code sample if lock is initialized as null, the synchronized (lock) will throw NullPointerException.
* Synchronized methods in Java put a performance cost on your application. So use synchronization when it is absolutely required. Also, consider using synchronized code blocks for synchronizing only critical section of your code.
* It’s possible that both static synchronized and non static synchronized method can run simultaneously or concurrently because they lock on different object.
* According to the Java language specification you can not use java synchronized keyword with constructor it’s illegal and result in compilation error.
* Do not synchronize on non final field on synchronized block in Java. because reference of non final field may change any time and then different thread might synchronizing on different objects i.e. no synchronization at all. Best is to use String class, which is already immutable and declared final.

### How to Work With wait(), notify() and notifyAll() in Java?

The `Object` class in Java has three final methods that allow threads to communicate about the locked status of a resource. These are :

**wait() :** It tells the calling thread to give up the lock and go to sleep until some other thread enters the same monitor and calls `notify()`. The wait() method releases the lock prior to waiting and reacquires the lock prior to returning from the `wait()` method. The `wait()` method is actually tightly integrated with the synchronization lock, using a feature not available directly from the synchronization mechanism. In other words, it is not possible for us to implement the `wait()` method purely in Java: it is a native method.

General syntax for calling wait() method is like this:

```java
synchronized( lockObject )
{
    while( ! condition )
    {
        lockObject.wait();
    }
     
    //take the action here;
}
```

**notify() :** It wakes up one single thread that called `wait()` on the same object. It should be noted that calling `notify()` does not actually give up a lock on a resource. It tells a waiting thread that that thread can wake up. However, the lock is not actually given up until the notifier’s synchronized block has completed. So, if a notifier calls `notify()` on a resource but the notifier still needs to perform 10 seconds of actions on the resource within its synchronized block, the thread that had been waiting will need to wait at least another additional 10 seconds for the notifier to release the lock on the object, even though `notify()` had been called.

General syntax for calling `notify()` method is like this:

```java
synchronized(lockObject)
{
    //establish_the_condition;
 
    lockObject.notify();
     
    //any additional code if needed
}
```

**notifyAll() :** It wakes up all the threads that called `wait()` on the same object. The highest priority thread will run first in most of the situation, though not guaranteed. Other things are same as `notify()` method above.

General syntax for calling `notifyAll()` method is like this:

```java
synchronized(lockObject)
{
    establish_the_condition;
 
    lockObject.notifyAll();
}
```

### How to Use with wait(), notify() and notifyAll() Methods
we will solve producer consumer problem using `wait()` and `notify()` methods. To keep program simple and to keep focus on usage of `wait()` and `notify()` methods, we will involve only one producer and one consumer thread.

[@article](https://howtodoinjava.com/core-java/multi-threading/how-to-work-with-wait-notify-and-notifyall-in-java/)

### Difference between Runnable vs Thread in Java

```java
public class DemoRunnable implements Runnable {
    public void run() {
        //Code
    }
}
//with a "new Thread(demoRunnable).start()" call
 
public class DemoThread extends Thread {
    public DemoThread() {
        super("DemoThread");
    }
    public void run() {
        //Code
    }
}
//with a "demoThread.start()" call
```

* Implementing Runnable is the preferred way to do it. Here, you’re not really specializing or modifying the thread’s behavior. You’re just giving the thread something to run. That means composition is the better way to go.
* Java only supports single inheritance, so you can only extend one class.
* Instantiating an interface gives a cleaner separation between your code and the implementation of threads.
* Implementing Runnable makes your class more flexible. If you extend thread then the action you’re doing is always going to be in a thread. However, if you extend Runnable it doesn’t have to be. You can run it in a thread, or pass it to some kind of executor service, or just pass it around as a task within a single threaded application.
* By extending Thread, each of your threads has a unique object associated with it, whereas implementing Runnable, many threads can share the same runnable instance.

### Difference between yield() and join()

#### yield() method
Theoretically, to ‘yield’ means to let go, to give up, to surrender. A yielding thread tells the virtual machine that it’s willing to let other threads be scheduled in its place. This indicates that it’s not doing something too critical. Note that it’s only a hint, though, and not guaranteed to have any effect at all.

#### join() method
The join() method of a Thread instance can be used to “join” the start of a thread’s execution to the end of another thread’s execution so that a thread will not start running until another thread has ended. If join() is called on a Thread instance, the currently running thread will block until the Thread instance has finished executing.

### Difference between sleep() and wait()?
`sleep()` is a method which is used to hold the process for few seconds or the time you wanted but in case of `wait()` method thread goes in waiting state and it won’t come back automatically until we call the `notify()` or `notifyAll()`.

**Call on:**
* wait(): Call on an object; current thread must synchronize on the lock object.
* sleep(): Call on a Thread; always currently executing thread.
 
**Synchronized:**
* wait(): when synchronized multiple threads access same Object one by one.
* sleep(): when synchronized multiple threads wait for sleep over of sleeping thread.

**Hold lock:**
* wait(): release the lock for other objects to have chance to execute.
* sleep(): keep lock for at least t times if timeout specified or somebody interrupt.

**Wake-up condition:**
* wait(): until call notify(), notifyAll() from object
* sleep(): until at least time expire or call interrupt().

**Usage:**
* sleep(): for time-synchronization and;
* wait(): for multi-thread-synchronization.

### Java Executor Framework Tutorial and Best Practices
`Executors framework` (java.util.concurrent.Executor), released with the JDK 5 in package `java.util.concurrent` is used to run the Runnable objects without creating new threads every time and mostly re-using the already created threads.

**Basic usage demo application**

In our demo application, we have two tasks running. Neither is expected to terminate, and both should run for the duration of the application’s life. I will try to write a main wrapper class such that:

If any task throws an exception, the application will catch it and restart the task.
If any task runs to completion, the application will notice and restart the task.

Below is the code sample of above required application:

```java
package com.howtodoinjava.multiThreading.executors;
 
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class DemoExecutorUsage {
 
    private static ExecutorService executor = null;
    private static volatile Future taskOneResults = null;
    private static volatile Future taskTwoResults = null;
 
    public static void main(String[] args) {
        executor = Executors.newFixedThreadPool(2);
        while (true)
        {
            try
            {
                checkTasks();
                Thread.sleep(1000);
            } catch (Exception e) {
                System.err.println("Caught exception: " + e.getMessage());
            }
        }
    }
 
    private static void checkTasks() throws Exception {
        if (taskOneResults == null
                || taskOneResults.isDone()
                || taskOneResults.isCancelled())
        {
            taskOneResults = executor.submit(new TestOne());
        }
 
        if (taskTwoResults == null
                || taskTwoResults.isDone()
                || taskTwoResults.isCancelled())
        {
            taskTwoResults = executor.submit(new TestTwo());
        }
    }
}
 
class TestOne implements Runnable {
    public void run() {
        while (true)
        {
            System.out.println("Executing task one");
            try
            {
                Thread.sleep(1000);
            } catch (Throwable e) {
                e.printStackTrace();
            }
        }
 
    }
}
 
class TestTwo implements Runnable {
    public void run() {
        while (true)
        {
            System.out.println("Executing task two");
            try
            {
                Thread.sleep(1000);
            } catch (Throwable e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Executing multiple tasks in a single thread
It’s not necessary that each Runnable should be executed in a separate thread. Sometimes, we need to do multiple jobs in a single thread and each job is instance of Runnable. To design this type of solution, a multi runnable should be used. This multi runnable is nothing but a collection of runnables which needs to be executed. Only addition is that this multi runnable is also a Runnable itself.

Below is the list of tasks which needs to be executed in a single thread.

```java
package com.howtodoinjava.multiThreading.executors;
 
public class TaskOne implements Runnable {
    @Override
    public void run() {
        System.out.println("Executing Task One");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
 
public class TaskTwo implements Runnable {
    @Override
    public void run() {
        System.out.println("Executing Task Two");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
 
public class TaskThree implements Runnable {
    @Override
    public void run() {
        System.out.println("Executing Task Three");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

Lets create out multi runnable wrapper of above Tasks.

```java
package com.howtodoinjava.demo.multiThread;
 
import java.util.List;
 
public class MultiRunnable implements Runnable {
 
    private final List<Runnable> runnables;
 
    public MultiRunnable(List<Runnable> runnables) {
        this.runnables = runnables;
    }
 
    @Override
    public void run() {
        for (Runnable runnable : runnables) {
             new Thread(runnable).start();
        }
    }
}
```

Now above multi runnable can be executed this way as in below program:

```java
package com.howtodoinjava.demo.multiThread;
 
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
public class MultiTaskExecutor {
 
    public static void main(String[] args) {
         
        BlockingQueue<Runnable> worksQueue = new ArrayBlockingQueue<Runnable>(10);
        RejectedExecutionHandler rejectionHandler = new RejectedExecutionHandelerImpl();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(3, 3, 10, TimeUnit.SECONDS, worksQueue, rejectionHandler);
 
        executor.prestartAllCoreThreads();
         
        List<Runnable> taskGroup = new ArrayList<Runnable>();
        taskGroup.add(new TestOne());
        taskGroup.add(new TestTwo());
        taskGroup.add(new TestThree());
  
        worksQueue.add(new MultiRunnable(taskGroup));
    }
}
 
class RejectedExecutionHandelerImpl implements RejectedExecutionHandler
{
    @Override
    public void rejectedExecution(Runnable runnable,
            ThreadPoolExecutor executor)
    {
        System.out.println(runnable.toString() + " : I've been rejected ! ");
    }
}
```

**Best practices which must be followed**

* Always run your java code against static analysis tools like PMD and FindBugs to look for deeper issues. They are very helpful in determining ugly situations which may arise in future.
* Always cross check and better plan a code review with senior guys to detect and possible deadlock or livelock in code during execution. Adding a health monitor in your application to check the status of running tasks is an excellent choice in most of the scenarios.
* In multi-threaded programs, make a habit of catching errors too, not just exceptions. Sometimes unexpected things happen and Java throws an error at you, apart from an exception.
* Use a back-off switch, so if something goes wrong and is non-recoverable, you don’t escalate the situation by eagerly starting another loop. Instead, you need to wait until the situation goes back to normal and then start again.
* Please note that the whole point of executors is to abstract away the specifics of execution, so ordering is not guaranteed unless explicitly stated.

### ScheduledThreadPoolExecutor – Task Scheduling with Executors
The Java Executor Framework provides the `ThreadPoolExecutor` class to execute Callable and Runnable tasks with a pool of threads, which avoid you writing lots of boiler plate complex code. The way executors work is when you send a task to the executor, it’s executed as soon as possible. But there may be used cases when you are not interested in executing a task as soon as possible. Rather You may want to execute a task after a period of time or to execute a task periodically. For these purposes, the Executor framework provides the `ScheduledThreadPoolExecutor` class.

**Task to be executed**
Let’s write a very basic task which we can use for demo purpose.

```java
class Task implements Runnable
{
    private String name;
 
    public Task(String name) {
        this.name = name;
    }
     
    public String getName() {
        return name;
    }
 
    @Override
    public void run()
    {
        try {
            System.out.println("Doing a task during : " + name + " - Time - " + new Date());
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**Execute a task after a period of time**

```java
package com.howtodoinjava.demo.multithreading;
 
import java.util.Date;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
 
public class ScheduledThreadPoolExecutorExample
{
    public static void main(String[] args)
    {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
        Task task1 = new Task ("Demo Task 1");
        Task task2 = new Task ("Demo Task 2");
         
        System.out.println("The time is : " + new Date());
         
        executor.schedule(task1, 5 , TimeUnit.SECONDS);
        executor.schedule(task2, 10 , TimeUnit.SECONDS);
         
        try {
              executor.awaitTermination(1, TimeUnit.DAYS);
        } catch (InterruptedException e) {
              e.printStackTrace();
        }
         
        executor.shutdown();
    }
}
 
Output:
 
The time is : Wed Mar 25 16:14:07 IST 2015
Doing a task during : Demo Task 1 - Time - Wed Mar 25 16:14:12 IST 2015
Doing a task during : Demo Task 2 - Time - Wed Mar 25 16:14:17 IST 2015
```

As with class `ThreadPoolExecutor`, to create a scheduled executor, Java recommends the utilization of the `Executors` class. In this case, you have to use the `newScheduledThreadPool()` method. You have passed the number 1 as a parameter to this method. This parameter is the number of threads you want to have in the pool.

To execute a task in this scheduled executor after a period of time, you have to use the `schedule()` method. This method receives the following three parameters:

* The task you want to execute
* The period of time you want the task to wait before its execution
* The unit of the period of time, specified as a constant of the TimeUnit class

Also note that You can also use the `Runnable` interface to implement the tasks, because the `schedule()` method of the `ScheduledThreadPoolExecutor` class accepts both types of tasks.

Moreover ,although the `ScheduledThreadPoolExecutor` class is a child class of the `ThreadPoolExecutor` class and, therefore, inherits all its features, Java recommends the utilization of `ScheduledThreadPoolExecutor` only for scheduled tasks.

Finally, you can configure the behavior of the `ScheduledThreadPoolExecutor` class when you call the `shutdown()` method and there are pending tasks waiting for the end of their delay time. The default behavior is that those tasks will be executed despite the finalization of the executor. You can change this behavior using the `setExecuteExistingDelayedTasksAfterShutdownPolicy()` method of the `ScheduledThreadPoolExecutor` class. With false, at the time of `shutdown()`, pending tasks won’t get executed.

#### Execute a task periodically

Now let’s learn how to use ScheduledThreadPoolExecutor to schedule a periodic task.

```java
public class ScheduledThreadPoolExecutorExample
{
    public static void main(String[] args)
    {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        Task task1 = new Task ("Demo Task 1");
         
        System.out.println("The time is : " + new Date());
         
        ScheduledFuture<?> result = executor.scheduleAtFixedRate(task1, 2, 5, TimeUnit.SECONDS);
         
        try {
            TimeUnit.MILLISECONDS.sleep(20000);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
         
        executor.shutdown();
    }
}
 
Output:
 
The time is : Wed Mar 25 16:20:12 IST 2015
Doing a task during : Demo Task 1 - Time - Wed Mar 25 16:20:14 IST 2015
Doing a task during : Demo Task 1 - Time - Wed Mar 25 16:20:19 IST 2015
Doing a task during : Demo Task 1 - Time - Wed Mar 25 16:20:24 IST 2015
Doing a task during : Demo Task 1 - Time - Wed Mar 25 16:20:29 IST 2015
```

In this example, we have created `ScheduledExecutorService` instance just like above example using `newScheduledThreadPool()` method. Then we have used the `scheduledAtFixedRate()` method. This method accepts four parameters:

* the task you want to execute periodically,
* the delay of time until the first execution of the task,
* the period between two executions,
* and the time unit of the second and third parameters.

An important point to consider is that the period between two executions is the period of time between these two executions that begins. If you have a periodic task that takes 5 seconds to execute and you put a period of 3 seconds, you will have two instances of the task executing at a time.

### Java Fixed Size Thread Pool Executor Example
**Fixed size thread pool executor** which will help in improved performance and better system resource utilization by limiting the maximum number of threads in thread pool.

#### 1) Create a task to execute
Obviously, first step is to have a task which you would like to execute using executors.

```java
class Task implements Runnable
{
    private String name;
 
    public Task(String name)
    {
        this.name = name;
    }
     
    public String getName() {
        return name;
    }
 
    @Override
    public void run()
    {
        try
        {
            Long duration = (long) (Math.random() * 10);
            System.out.println("Doing a task during : " + name);
            TimeUnit.SECONDS.sleep(duration);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }
}
```

#### 2) Execute tasks using Executors

Now all you have to do is to create an instance of `ThreadPoolExecutor` with fixed size and pass the tasks to be executed into it’s `execute()` method.

```java
package com.howtodoinjava.demo.multithreading;
 
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
public class FixedThreadPoolExecutorExample
{
    public static void main(String[] args)
    {
        ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(4);
        for (int i = 0; i < 10; i++)
        {
            Task task = new Task("Task " + i);
            System.out.println("A new task has been added : " + task.getName());
            executor.execute(task);
        }
        System.out.println("Maximum threads inside pool " + executor.getMaximumPoolSize());
        executor.shutdown();
    }
}
 
Output:
 
A new task has been added : Task 0
A new task has been added : Task 1
A new task has been added : Task 2
A new task has been added : Task 3
A new task has been added : Task 4
A new task has been added : Task 5
A new task has been added : Task 6
A new task has been added : Task 7
Doing a task during : Task 0
Doing a task during : Task 2
Doing a task during : Task 1
A new task has been added : Task 8
Doing a task during : Task 3
A new task has been added : Task 9
 
Maximum threads inside pool 4
 
Doing a task during : Task 4
Doing a task during : Task 5
Doing a task during : Task 6
Doing a task during : Task 7
Doing a task during : Task 8
Doing a task during : Task 9
```

1) newFixedThreadPool() method creates an executor with a maximum number of threads at any time. If you send more tasks than the number of threads, the remaining tasks will be blocked until there is a free thread to process them This method receives the maximum number of threads as a parameter you want to have in your executor. In your case, you have created an executor with four threads.

2) The Executors class also provides the newSingleThreadExecutor() method. This is an extreme case of a fixed-size thread executor. It creates an executor with only one thread, so it can only execute one task at a time.

### 















































