---
layout: page
---
Testing Multithreaded Code
==========================

A question that is frequently asked on the [jMock](http://www.jmock.org) users' [mailing list](http://www.jmock.org/mailing-lists.html) is "how do I use mock objects to test a class that spawns new threads"? The problem is that jMock appears to ignore unexpected calls if they are made on a different thread than that which runs the test itself. The mock object actually does throw an `AssertionFailedError` when it receives an unexpected call, but the error terminates the concurrent thread instead of the test runner's thread.

Here's an example. We have a guard and an alarm. When the guard notices a burglar, he should ring the alarm once only.

``` Source
public interface Alarm {
    void ring();
}
```

``` Source
@Test
public void ringsTheAlarmOnceWhenNoticesABurglar() {
    final Alarm alarm = context.mock(Alarm.class);
    Guard guard = new Guard(alarm);
    
    context.checking(new Expectations() {{
        oneOf (alarm).ring();
    }});
    
    guard.notice(new Burglar());
}
```

Here's an implementation of Guard that should fail the test:

``` Source
public class Guard {
    private Alarm alarm;
    
    public Guard( Alarm alarm ) {
        this.alarm = alarm;
    }
    
    public void notice(Burglar burglar) {
        startRingingTheAlarm();
    }
    
    private void startRingingTheAlarm() {
        Runnable ringAlarmTask = new Runnable() {
            public void run() {
                for (int i = 0; i < 10; i++) {
                    alarm.ring();
                }
            }
        };
        
        Thread ringAlarmThread = new Thread(ringAlarmTask);
        ringAlarmThread.start();
    }
}
```

Although that implementation is incorrect, the test will appear to pass because the mock Alarm will throw an AssertionFailedError on the ringAlarmThread, not the test runner's thread.

The root of the problem is trying to use mock objects for integration testing. Mock objects are used for unit testing in the traditional sense: to test units in isolation from other parts of the system. Threads, however, by their very nature, require some kind of integration test. Concurrency and synchronisation are system-wide concerns and code that creates threads must make use of operating system facilities to do so.

A solution is to separate the object that needs to run tasks from the details of how tasks are run and define an interface between the two. We can test the object that needs to run tasks by mocking the task runner, and test the implementation of the task runner in integration tests. Java's standard concurrency library defines just such an interface: [java.util.concurrent.Executor](http://java.sun.com/j2se/1.5.0/docs/api/java/util/concurrent/Executor.html). In our example, we can pass an Executor to the Guard and change the Guard so that it asks the executor to run tasks instead of explicitly creating new threads.

Our test then looks like:

``` Source
@Test
public void ringsTheAlarmOnceWhenNoticesABurglar() {
    final Alarm alarm = context.mock(Alarm.class);
    Executor executor = ... // What goes here?
    Guard guard = new Guard(alarm, executor);
    
    context.checking(new Expectations() {{
        oneOf (alarm).ring();
    }});
    
    guard.notice(new Burglar());
}
```

But how should we implement the Executor for our test? If we use an Executor that creates a new thread we'll be back where we started and our tests will still, wrongly, appear to pass. We need to run the task in the same thread as the test runner. To do this we could mock the Executor interface and use a [custom action](custom-actions.html) to call back to the task's run method. However, that does not correctly reflect how a background task would behave: the task executes before the notify method has returned, instead of afterwards. This difference is significant when a task modifies the state of an object.

Instead we need an object that implements the Executor interface and lets us explicitly run the tasks after the `notice(Burglar)` method has returned. JMock provides a class that does just this: the `DeterministicExecutor`. Using it, our test looks like:

``` Java
@Test
public void ringsTheAlarmOnceWhenNoticesABurglar() {
    final Alarm alarm = context.mock(Alarm.class);
    DeterministicExecutor executor = new DeterministicExecutor();
    Guard guard = new Guard(alarm, executor);
    
    guard.notice(new Burglar());
    
    context.checking(new Expectations() {{
        oneOf (alarm).ring();
    }});
    
    executor.runUntilIdle();    
}
```

And the implementation of the Guard that uses an Executor looks like this:

``` Source
public class Guard {
    private Alarm alarm;
    private Executor executor;
    
    public Guard(Alarm alarm, Executor executor) {
        this.alarm = alarm;
        this.executor = executor;
    }
    
    public void notice(Burglar burglar) {
        startRingingTheAlarm();
    }
    
    private void startRingingTheAlarm() {
        Runnable ringAlarmTask = new Runnable() {
            public void run() {
                for (int i = 0; i < 10; i++) {
                    alarm.ring();
                }
            }
        };
        executor.execute(ringAlarmTask);
    }
}
```

It's still not correct, but at least our tests can detect the error now!

The test tells the executor to run until idle, which runs pending tasks until there are no more to execute. This allows tasks to launch new tasks. If that happened forever, the executor would, of course, enter an infinite loop. Therefore the DeterministicExecutor also let's a test run only the pending tasks to support applications that spawn an infinite sequence of tasks.

Beyond easy testing, the advantage of pulling out the Executor from the Guard is that the Guard does not hide its concurrency policy within its implementation. Concurrency is a system-wide concern and should be controlled outside the Guard object. By passing an appropriate Executor to its constructor, the application can easily adapt the Guard to the application's threading policy without having to change the Guard's implementation.

However, this technique tests the only functional behaviour of the objects. It does not test the synchronisation. You'll also need integration tests that put truly concurrent code under load, using the [Blitzer](threading-blitzer.html) for example, and try to detect synchronisation errors. By separating the concerns of functionality and synchronisation you will know that any errors in your load tests are caused by synchronisation problems, not bugs in the functional behaviour.
