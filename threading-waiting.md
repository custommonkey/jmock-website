---
layout: page
---
One of the trickiest aspects of testing multithreaded code is synchronizing between the test thread and the threads started by the code being tested. If you get the synchronisation wrong, the test can finish before all the threads that it started have terminated. This might make the test return false positives, because it does not detect failures on the runaway threads. Or the threads might interfere with later tests, causing intermittent test failures that are hard to track down and eliminate.

[JMock's](http://www.jmock.org "JMock Website") [Synchroniser](000762.html "Synchronising Imposteriser") has a cunning yet simple solution that builds upon jMock's existing [state machine construct](http://www.jmock.org/states.html "JMock State Machines"). A test can tell the Synchroniser to wait for a state machine to enter or leave some state.

The test needs to store the Synchroniser in a field:

``` Java
Synchroniser synchroniser = new Synchroniser();

Mockery context = new JUnit4Mockery() {{
    setThreadingPolicy(synchroniser);
}};
```

Now the test can define state machines and use the synchroniser to wait for those state machines to enter or leave a given state.

``` Java
final States searching = context.states("searching")

context.checking(new Expectations() {{
    oneOf(consumer).searchFound(result("A1"));
        when(searching.isNot("finished"));

    oneOf(consumer).searchFound(result("B1"), result("B2"));
        when(searching.isNot("finished"));

    oneOf(consumer).searchFinished();
        then(searching.is("finished"));
}});

search.searchFor("sheep", "cheese");

synchroniser.waitUntil(searching.is("finished"));
```

This must be combined with a timeout to ensure that the test does not wait forever if the system under test never meets the awaited criteria. A timeout can be passed to the waitUntil method; it throws a TimeoutException with an informative message if the timeout expires. Or, you can get your test framework to timeout the test. For example, in [JUnit 4](http://www.junit.org "JUnit Website") you can pass a timeout parameter to the @Test annotation.
