---
layout: page
---
Expecting Methods More (or Less) than Once
==========================================

The [Getting Started](getting-started.html) recipe only demonstrates tests that expect one call to one mock object. Tests usually need to specify expectations with different cardinalities to allow some method calls to occur but not fail if they don't, expect methods to be called more than once or not all, or ignore mock objects that are not relevant.

The "invocation count" of an expectation defines the minimum and maximum number of times that the expected method is allowed to be invoked. It is specified before the mock object in the expectation.

    invocation-count (mock).method(parameters); ...

JMock defines the following cardinalities:

|                      |                                                                                                              |
|----------------------|--------------------------------------------------------------------------------------------------------------|
| one                  | The invocation is expected once and once only.                                                               |
| exactly(n).of        | The invocation is expected exactly n times. Note: `one` is a convenient shorthand for `exactly(1)`.          |
| atLeast(n).of        | The invocation is expected at least n times.                                                                 |
| atMost(n).of         | The invocation is expected at most n times.                                                                  |
| between(min, max).of | The invocation is expected at least min times and at most max times.                                         |
| allowing             | The invocation is allowed any number of times but does not have to happen.                                   |
| ignoring             | The same as `allowing`. Allowing or ignoring should be chosen to make the test code clearly express intent.  |
| never                | The invocation is not expected at all. This is used to make tests more explicit and so easier to understand. |

Expecting vs. Allowing
----------------------

The most important distinction that cardinalities express is between *expecting* and *allowing* an invocation.

If an invocation is allowed, it can occur during a test run, but the test still passes if the invocation does not occur. If an invocation is expected, on the other hand, it must occur during a test run; if it does not occur, the test fails. When defining expectations you must choose whether to expect or allow each invocation. In general, we find that tests are kept flexible if you allow queries and expect commands. A query is a method with no side effects that does nothing but query the state of an object and a command is a method with side effects that may, or may not, return a result. Of course, this does not hold all the time but it's a useful rule of thumb to follow if there are no other constraints on the number of times you expect a method to be called in your test.
