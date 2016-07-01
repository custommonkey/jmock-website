---
layout: page
---
jMock: Yoga for Your Unit Tests
===============================

The examples in this article use the jMock 1 API. The ideas are just as applicable to jMock 2 (or EasyMock or any other mock object framework).

Brittle tests are a common "gotcha" of test driven development. A brittle unit test will stop passing when you make unrelated changes to your application code. A test suite that contains a lot of brittle tests will slow down development and inhibit refactoring. Thests become brittle when they overspecify the behaviour of the unit being tested. You can keep your tests flexible by following a simple rule of thumb: <span class="RuleOfThumb">specify exactly what you want to happen *and no more*</span>. This article describes how various features of jMock can help you strike the right balance between an accurate specification of a unit's required behaviour and a flexible test that allows easy evolution of the code base.

Stubs and Expectations
----------------------

jMock distinguishes between stubbed and expected methods. A stubbed method call can occur during a test run, but if it does not occur the test still passes. An expected method call, on the other hand, *must* occur during a test run; if it does not occur, the test fails when it verifies the mock. When setting up your mocks you must choose whether to stub or expect each invocation. In general, we have found that tests are kept flexible when we follow this rule of thumb: <span class="RuleOfThumb">stub queries and expect commands</span>, where a query is a method with no side effects that does nothing but query the state of an object and a command is a method with side effects that may, or may not, return a result. Of course, this rule does not hold all the time, but it's a useful starting point.

A stub can be called any number of times. The number of times a method is expected is defined by the expectation itself. The `once()` expectation allows the method to be called once only; subsequent calls will make the test fail. The `atLeastOnce()` expectation allows the method to be called any number of times. The test will only fail if the method is never called at all. You can define further expectation types if these are not sufficient for your needs.

In the following example, the `getLoggingLevel` is stubbed: it does not have to be called. The `setLoggingLevel` must be called once and once only.

    logger.stubs().method("getLoggingLevel").noParams()
        .will(returnValue(Logger.WARNING));
    logger.expects(once()).method("setLoggingLevel").with(eq(newLevel));

Parameter Constraints
---------------------

jMock requires that you specify a *constraint* on each actual parameter of a expected invocation, rather than specify just the expected parameter value. Specifying equality to an expected value is the most common case, but is too strict for many scenarios. For example, consider a system that logs errors to a `Logger` object. To test that an object correctly detects and logs an error, such as a failed attempt to connect to a database, you could set up an expectation on a mock logger to check the value of the message passed to the logger:

    String expectedErrorMessage = "unable to connect to ORDERS database: network timeout";

    logger.expects(once()).method("error").with(eq(expectedErrorMessage));

This will work in the short term, but will cause problems long term because it too precisely specifies the expected value of the error message, including minor details of punctuation and whitespace. If you change that formatting later, the test will fail even though the code under test still does what you want it to do. You really only care that the error message contains the information you want to report to the user (the action that failed and the cause of the failure) not how that information is formatted. You can use the constraint functions defined by the `MockObjectTestCase` class to specify exactly what you want to happen:

    String action = "connect to ORDERS database";
    String cause = "network timeout";

    logger.expects(once()).method("error")
        .with( and(stringContains(action),stringContains(cause)) );

The `MockObjectTestCase` class defines several functions that can be used to define constraints. There are more constraint types defined in the `org.jmock.constraint` package. You can even [define your own constraints](custom-constraints.html).

Constraints require you to type a little more when writing your tests, and require you to think more carefully about what behaviour you expect, but the result is that your tests are easier to read because they clearly express your intent and more flexible because they don't overspecify expected behaviour.

Invocation Order
----------------

By default, jMock allows invocations upon a mock object to occur in any order. Sometimes, however, the order of calls is important, such as when you are testing an object that fires events or writes data to an output stream. In such cases you can explicitly specify that a call should occur after one or more others.

    logger.expects(once()).method("setLoggingLevel").with(eq(Logger.WARNING))
        .id("warning level set");
    logger.expects(once()).method("warn").with(warningMessage)
        .after("warning level set");
    logger.stubs().method("getLoggingLevel").noParams()
        .after("warning level set")
        .will(returnValue(Logger.WARNING));

A rule of thumb to follow when specifying the expected order of method calls is: <span class="RuleOfThumb">test the ordering of *only* those calls you want to occur in order</span>. The example above allows the `warn` and `getLoggingLevel` methods to occur in any order, as long as they occur after the call to `setLoggingLevel`. Thus we can change the order in which our tested object calls `warn` and `getLoggingLevel` without breaking our tests.

Specifying the order of calls is orthogonal to whether those calls are expectations or stubs. So, the example above specifies that the `getLoggingLevel` method does not have to be called, but if it is, it must be called after `setLoggingLevel`, and that the `warn` method must be called and must be called after `setLoggingLevel`.

Matchers
--------

The methods `method(`methodName`)`, `with(`argument constraints`)` and `after(`prior call ID`)` are syntactic sugar that define matching rules that match an incoming invocation to an object that can mock the behaviour of the invocation and test its expectations. If jMock does not have a matching rule that you need, or the matching rules supported by jMock are not accurate enough to keep your tests flexible, you can extend jMock with matching rules of your own.

If you need more control over how method names are matched, you can specify a constraint over names instead of a precise name value. An invocation will match if the constraint evaluates to `true` when applied to the method name. Suppose that you want to stub all calls to the property getter methods of a mock object by synthesising a default result based on the type of the property. You could achieve with a constraint that matches method names that begin with "get" and that have no parameters. (See [Writing Custom Constraints](custom-constraints.html) for the definition of the `StringStartsWith` class).

    ...

    private DefaultResultStub returnADefaultValue = new DefaultResultStub();

    public void testMethod() {
        ...
        mock.stubs().method(startingWith("get")).noParams().will(returnADefaultValue);
        ...
    }

    private Constraint startingWith( String prefix ) {
        return new StringStartsWith(prefix);
    }

    ...

This example is, however, still brittle. You want the mock to stub all property getter methods but that is not actually what the test specifies. To precisely specify the required behaviour you can [write your own matching rule](custom-matchers.html) that uses the Java Bean API to test if a method is a property getter.

    ...

    private DefaultResultStub returnADefaultValue = new DefaultResultStub();

    public void testMethod() {
        ...
        mock.stubs().match(aBeanPropertyGetter()).will(returnADefaultValue);
        ...
    }

    private InvocationMatcher aBeanPropertyGetter() {
        return new BeanPropertyGetterMatcher();
    }

    ...

Conclusion
----------

This article should have given you some idea of why the jMock API is designed as it is and how that API can help you avoid brittle tests. The most important rule of thumb we follow to keep our tests flexible is:

> <span class="RuleOfThumb">Specify exactly what you want to happen *and no more*.</span>

This guideline is just as applicable when writing unit tests that do not use mock objects.

Nat Pryce