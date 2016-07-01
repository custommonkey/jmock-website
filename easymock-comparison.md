---
layout: page
---
Comparing jMock and EasyMock
============================

Like jMock, [EasyMock](http://www.easymock.org) is a dynamic mock object library for Java. What are the main differences between jMock and EasyMock?

jMock uses strings to identify methods
--------------------------------------

EasyMock uses actual method calls to define expectations. EasyMock therefore works with an IDE's code completion and refactoring tools better than jMock.

    mock.method1( a );
    mockControl.playback();

    mock.expects(once()).method("method1").with( eq(a) );

We have found that the use of strings in jMock is not a huge disadvantage, especially if you use an IDE that can apply refactorings to names in string constants. Often when doing TDD, the method named in an expectation will not yet be defined in any interface, and so code completion cannot be used. We have found that the [flexibility](yoga.html) allowed by the jMock API is better at reducing the work of refactoring than the use of direct method calls.

jMock forces you to precisely specify required behaviour.
---------------------------------------------------------

By default, EasyMock tests actual arguments for equality to expected arguments using the `equals` method. This can overspecify expected behaviour and make tests brittle. Sometimes you want to ignore an argument or want looser constraints upon an argument's value. EasyMock lets you change the way that arguments are matched on a call-by-call basis, but the syntax is awkward and same matcher applies to *all* arguments. There isn't a way to specify a different matcher for different arguments to the same call.

    mock.method1( a );
    mock.method2( b1, b2 ); // cannot specify that we don't care about argument b2
    mockControl.setReturnValue( method2Result );
    mockControl.playback();

In contrast, jMock forces the user to specify exactly how each actual argument is matched against expectations. This means that expectations are more verbose but precisely and clearly specify the expected behaviour of the object. By precisely specifying expected behaviour you get [flexible tests](yoga.html) that break only when the actual behaviour is different from expected behaviour, and do not break when you make unrelated changes to application code.

    mock.expects(once()).method("method1").with( eq(a) );
    mock.expects(once()).method("method2").with( same(b1), ANYTHING )
        .will(returnValue(method2Result));

jMock tests read as specifications
----------------------------------

jMock is a design tool not a testing tool. jMock's API is designed so that tests express the design intent of the programmer as clearly as possible and can later be read as specifications. In EasyMock's "record/playback" API, expectations are normal method invocations that do not read as specifications and constraints upon those invocations must be specified with separate API calls that duplicate information about the expected call and are harder to read than jMock's expectations.

jMock expectations require more typing
--------------------------------------

The result of the previous two design goals is that jMock requires that the programmer writing a test be very explicit about what is and isn't expected. The "call-chain" API style makes expectations longer than EasyMock's terse "record/playback" style. We were surprised, however, to find that our users *prefer* all the extra typing that jMock forced them to do because the result helps them quickly understand what tests are specifying.

jMock can stub methods with side effects
----------------------------------------

EasyMock only stubs methods that return values or throw exceptions. jMock lets you write [custom stubs](custom-stubs.html) that can call back into the tested object or perform other side effects.

    mock.expects(once()).method("collectIntoList").with( isA(List.class) )
        .will(addListElement(element));

jMock gives you fine control over invocation order
--------------------------------------------------

EasyMock lets the programmer specify that calls to a mock must occur in order, or occur in any order. There is no way to specify that call a should occur after call b but in any order compared to call c. There is no way to test the order of calls to different mock objects.

jMock lets the programmer specify partial ordering of invocations and specify the order of invocations on different mock objects.

jMock's API syntax is extensible
--------------------------------

EasyMock can be extended in a few ways by the user. However extensions do not cleanly integrate with the rest of the framework and extension points are quite limited. For example, you cannot extend the way that EasyMock stubs calls.

User extensions to jMock are used in exactly the same way as the built-in jMock constructs. This makes it easier to read tests, because unimportant details (whether a constraint is or is not an extension) are kept out of the code of test methods. jMock lets the user extend more aspects of the framework's behaviour.

jMock automatically verifies mock objects
-----------------------------------------

jMock automatically verifies mock objects at the end of the test. It is easy to forget to verify all your mock objects and this can hide test failures. However, this means that...

jMock uses its own test case class
----------------------------------

To use jMock your test cases must extend [MockObjectTestCase](java:org.jmock.MockObjectTestCase). This class performs automatic verification and provides the syntactic sugar that makes jMock tests easy to read.
