---
layout: page
---
jMock 1: Stubs, Expectations and the Dispatch of Mocked Methods
===============================================================

In jMock, stubs and expectations are basically the same thing. A stub is just an expectation of zero or more invocations. The `stubs()` method is syntactic sugar to make the intent of the test more explicit.

When a method is invoked on a mock object, the mock object searches through its expectations from newest to oldest to find one that matches the invocation. After the invocation, the matching expectation might stop matching further invocations. For example, an `expects(once())` expectation only matches once and will be ignored on future invocations while an `expects(atLeastOnce())` expectation will always be matched against invocations.

This scheme allows you to:

-   Set up default stubs in your the `setUp` method of your test class and override some of those stubs in individual tests.
-   Set up different "once" expectations for the same method with different action per invocation. However, it's better to use the `onConsecutiveCalls` method to do this, as described below.

However, there are some possible "gotchas" caused by this scheme:

-   if you create an expectation and then a stub for the same method, the stub will always override the expectation and the expectation will never be met.
-   if you create a stub and then an expectation for the same method, the expectation will match, and when it stops matching the stub will be used instead, possibly masking test failures.
-   if you create different expectations for the same method, they will be invoked in the opposite order than that in which they were specified, rather than the same order.

The best thing to do is *not* set up multiple expectations and stubs for the same method with exactly the same matchers. Instead, use the `onConsecutiveCalls` method to create multiple actions for a method. For example:

    mock.expects(atLeastOnce()).method(m).with(...)
       .will( onConsecutiveCalls(
           returnValue(10),
           returnValue(20),
           throwException(new IOException("end of stream")) ) );

If you want to specify more complex ordering or order invocations across different mock objects, use the `after` method to explicitly define a total or partial ordering of invocations.
