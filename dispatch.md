---
layout: page
---
How Mocked Methods are Dispatched
=================================

When a method is invoked on a mock object, the Mockery searches through its expectations from oldest to newest to find one that matches the invocation. After the invocation, the matching expectation might stop matching further invocations. For example, an `one` expectation only matches once and will be ignored on future invocations while an `atLeast(1)` expectation will always be matched against invocations.

This scheme allows you to set up different expectations for the same method with different action per invocation. However, it's better to use the `onConsecutiveCalls` method to do this, as described below.

However, there are some possible "gotchas" caused by this scheme:

-   if you create an "allowing" or "ignoring" or "atLeast(n)" expectation before any other expectations that match the same invocations, the earlier expectation will always override the later expectation and the later expectation will never be triggered.
-   if you create an expectation and then a stub for the same method, the expectation will match, and when it stops matching the stub will be used instead, possibly masking test failures.

The best thing to do is *not* set up multiple expectations for the same method with exactly the same matchers. Instead, use the `onConsecutiveCalls` method to create multiple actions for a method. For example:

``` Java
atLeast(1).of (anObject).doSomething();
   will(onConsecutiveCalls(
       returnValue(10),
       returnValue(20),
       throwException(new IOException("no more stuff"))));
```

If you want to specify more complex ordering constraints use the [sequences](sequences.html) or [state machines](states.html) to explicitly define a total or partial ordering of invocations.
