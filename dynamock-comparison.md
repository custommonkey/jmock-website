---
layout: page
---
Comparing jMock and Dynamock
============================

jMock was born as a fork of DynaMock, the dynamic mock object implementation from [mockobjects.com](http://www.mockobjects.com/). Initially we wanted to clean up the code, fix some omissions in the API and make an honest-to-goodness 1.0 release that guaranteed API stability. In the end we produced a significantly different API that encapsulates the lessons we have learned about unit testing with mock objects.

What are the main differences that a DynaMock user will notice when they start using jMock?

jMock Has a "Call-Chain" API Syntax
-----------------------------------

The DynaMock API had many methods for defining stubs and expectations: `expect`, `expectAndReturn`, `expectAndThrow`, `match`, `matchAndReturn` and `matchAndThrow`. The jMock API has two methods, `expect` and `stub` that return a builder object with methods for specifying the method that is expected or stubbed. Those methods also return builder objects with methods that return builder objects, and so on. The chain of builders enforces a common order in which expectations are specified, which makes it easier to skim-read tests with mock objects. The call chain also makes expectations more descriptive, and so it is easier to understand what an expectation means.

Here is an example showing the same expectation set up using the DynaMock and jMock APIs:

    mock.expectAndReturn( "method1", a, result1 );
    mock.matchAndReturn( "method2", C.eq(b1,b2), result2 );

    mock.expects(once()).method("method1").with(eq(a))
        .will(returnValue(result1));
    mock.stubs().method("method2").with(eq(b1),eq(b2))
        .will(returnValue(result2));

The jMock API needs a bit more typing, but is easier to read, is more explicit about what it specifies and, a big benefit for us, is much easier to maintain and extend because there are far fewer methods that must be overloaded for Java's primitive types.

jMock Forces the User to be Explicit
------------------------------------

The DynaMock API implied information that the user did not explicitly specify. For example, passing no information about arguments to `expectAndReturn` (and related methods) implied that the expected method had no arguments, passing a value to `expectAndReturn` instead of a constraint implied that the actual value should be equal to the value given, and so on. Although intended to help slothful programmers, this feature was very confusing in practice. The maintainers of DynaMock often received support requests that were caused by a misunderstanding of exactly what was implied by eliding arguments from an an API call.

The jMock API requires that the user explicitly specify expectations and constraints. This makes tests much easier to read. We have not found that the additional typing required is a problem in practice.

    // Does this expect a method with no arguments?  Or ignore the arguments altogether?
    mock.expectAndReturn( "methodName", result1 );

    // Should the actual argument be the same object as 'arg', or an object equal to 'arg'?
    mock.expectAndReturn( "methodName", arg, result2 );

    mock.expects(once()).method("methodName").noParams()
        .will(returnValue(result1));
    mock.expects(once()).method("methodName").with(eq(arg))
        .will(returnValue(result2));

jMock Allows Different Kinds of Expectation
-------------------------------------------

A DynaMock expectation requires the expected method to be called once and only once. We sometimes want to specify that a method must be called at least once. Implementing this in DynaMock made the number of overloaded methods get completely out of hand. At-least-once expectations are supported by jMock API. If necessary, the user can even add new kinds of expectation without changing jMock itself.

    mock.expects(atLeastOnce()).method("methodName").with(eq(argument))
        .will(returnValue(resultValue));

jMock Allows Flexible Constraints on the Order of Invocations
-------------------------------------------------------------

DynaMock supports ordered and unordered mock objects. Ordered mock objects expect calls in a strict order; unordered mocks allow them to happen in any order. DynaMock does not support anything between these two extremes.

jMock, on the other hand, has a very flexible way of specifying how calls are ordered. If call B must occur after call A, the user must first give call A an identifier. They can then specify that call B must occur after A, using the identifier they assigned to A. Ordering constraints can be applied to expectations or stubs, and multiple ordering constraints can be defned on the same call. This allows a range of ordering constraints to be specified, including partial ordering of calls.

In the example below, `methodA` must be called before `methodB` and `methodC`, but `methodB` and `methodC` can occur in any order:

    mock.expects(once()).method("methodA").noParams()
        .id("first call to methodA");
    mock.expects(once()).method("methodB").noParams()
        .after("first call to methodA");
    mock.expects(once()).method("methodC").noParams()
        .after("first call to methodA");

Even better, jMock can define ordering constraints over method calls to *different* mocks:

    mock1.expects(once()).method("methodA").noParams()
        .id("first call to methodA");
    mock2.expects(once()).method("methodB").noParams()
        .after(mock1,"first call to methodA");

jMock Puts Support Functions in a Test Case Base Class
------------------------------------------------------

DynaMock uses a class named `C` to hold support functions that help set up expectations on a mock. This led to ugly code and was not easy to extend. jMock implements support functions in a test case class, `MockObjectTestCase` that the user must extend when they use mock objects in their tests. This has both advantages and disadvantages. It means that the support functions can be seamlessly extended by the user by writing new functions in their own class (see below). It also lets the test case automatically verify mock objects (also see below). However, thanks to Java's single inheritance of classes, the user cannot inherit support functions from more than one test case superclass.

    import junit.framework.TestCase;
    import com.mockobjects.dynamic.Mock;

    class MyTestCase extends TestCase {
        public void testSomething() {
            Mock mock = new Mock(MyInterface.class);
                    
            mock.expectAndReturn( "set", C.args(C.eq("key"),C.eq("value")), "oldValue" );
            
            ...
        }
    }

    import org.jmock.MockObjectTestCase;
    import org.jmock.Mock;

    class MyTestCase extends MockObjectTestCase {
        public void testSomething() {
            Mock mock = new Mock(MyInterface.class);
                    
            mock.expects(once()).method("set").with(eq("key"),eq("value"))
                .will(returnValue("oldValue"));
            ...
        }
    }

jMock Can Automatically Verify Mock Objects
-------------------------------------------

The `MockObjectTestCase` class automatically verifies any `Verifiable` objects stored in its instance variables. This helps the user avoid common errors caused by forgetting a verify statement. This automatic verification can be overridden if necessary. It is actually defined by the `VerifyingTestCase` so that it can be reused by other sugary APIs that need different support functions from the jMock API.

jMock is Easy to Extend
-----------------------

DynaMock allowed the user to create new constraint types and seamlessly use them in DynaMock tests. Adding new stubs was possible but ugly. New expectation types were not supported at all. jMock is much easier to extend: the user can seamlessly add new types of constraint, expectation, matcher, and stub.

Under the hood, jMock has a layered architecture. The "jMock API" that is used in unit tests is a thin layer of syntactic sugar on top of a flexible but inconvenient "core API". The core API can be used to extend the sugary API, or even replace it completely. For example, an optional extension uses the core API to make jMock use [CGLIB](http://cglib.sourceforge.net/) so that tests can mock concrete classes. The core API has also been used to write an API very similar to that of [EasyMock](http://www.easymock.org/).
