---
layout: page
---
By default, jMock is not thread safe and so its mock objects should not be called from multiple threads. However, people frequently do use it this way. They may want to use mock objects when stress-testing the synchronisation of an object they've written and tested using the deterministic executor. Or they may want to test a class that they are implementing with a third-party library that calls into their code from multiple threads. To use jMock with multiple threads, you have to plug a different ThreadingPolicy into the Mockery.

A ThreadingPolicy is an object that the Mockery uses to synchronise access to its state. A ThreadingPolicy wraps a decorator around every Mock Object before it is imposterised and so intercepts all mocked calls entering the Mockery and all results and exceptions that it returns. The default ThreadingPolicy complains if the mock objects are used by more than one thread. But if you want to test multithreaded code you can plug a Synchroniser into the Mockery instead.

You use the Synchroniser like this:

``` Java
Mockery mockery = new JUnit4Mockery() {{
    setThreadingPolicy(new Synchroniser());
}};
```

Now any mock objects created by the Mockery can be safely invoked from multiple threads. It's that easy. The Synchroniser has other features that help test multithreaded code, that are described elsewhere in the [cookbook](cookbook.html).

Error Diagnostics
-----------------

Testing multithreaded code is made difficult because failures do not get thrown on the test thread. The test framework cannot easily trap those exceptions and fail the test with a helpful diagnostic message. Often tests fail with a timeout, which has no message explaining why the timeout was exceeded. Or, an assertion on the test thread fails because an error that occurred on a background thread left the object under test in an invalid state, but information about the root cause of the failure is lost. JMock has a few mechanisms to help with this.

JMock captures the first expectation failure that is thrown from a mock object. If the error is swallowed, either by a background thread or by a try/catch block in the foreground thread, the test will continue running and the mockery will throw the error when the test asserts that all expected calls have been made. This is usually done automatically at the end of the test.

The Synchroniser's wait methods also fail fast and rethrow expectation violations that occur on background threads. A test can wait for the system to enter an expected state but if it does not do so because the system violated an expectation, the wait will immediately abort and throw the error that originally reported the violated expectation.

The Mockery records a trace of all the invocations that any mock object has received during the test and includes the trace recorded so far when it throws an expectation error.

The combination of failing fast, reporting the root cause of failure, and reporting the sequence of calls that led to the failure makes diagnosis much easier than a bare "test timed out" message.
