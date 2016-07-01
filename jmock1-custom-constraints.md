---
layout: page
---
jMock 1: Writing Custom Constraints
===================================

jMock provides several classes and factory functions that define [constraints over the acceptable parameter values of a method invocation](jmock1-constraints.html). Sometimes, however, the predefined constraints do not let you specify an expectation accurately enough to keep your tests flexible. In such cases, you can easily define new constraints that seamlessly extend the existing constraints defined by jMock.

A constraint is an object that implements the `Constraint` interface. It does two things:

-   report whether a parameter value meets the constraint (the `eval` method).
-   provide a readable description to be included in test failure messages (the `describeTo` method from the `SelfDescribing` interface).

The `org.jmock.constraint` package contains many constraint implementations and the programmer can easily specify their own constraints by writing their own implementations of the `Constraint` interface.

To create a new constraint:

1.  Write a class that implements the `Constraint` interface. The following constraint class tests whether a string starts with a given prefix.

        import org.jmock.core.Constraint;

        public class StringStartsWith implements Constraint {
            private String prefix;

            public StringStartsWith( String prefix ) {
                this.prefix = prefix;
            }

            public boolean eval( Object o ) {
                return o instanceof String && ((String)o).startsWith(prefix);
            }

            public StringBuffer describeTo( StringBuffer buffer ) {
                return buffer.append("a string starting with ")
                             .append(Formatting.toReadableString(prefix));
            }
        }

    Note that the `describeTo` method calls `Formatting.toReadableString` to format the prefix. The `toReadableString` method formats any raw value in a way that subtly indicates the value's type and should be used to format any raw values stored in your constraints so that error messages are easy to read.

2.  Write a factory method in your test case that creates an instance of your new constraint.

        public class MyTestCase extends MockObjectTestCase {
            ...

            private Constraint aStringStartingWith( String prefix ) {
                return new StringStartsWith(prefix);
            }
        }

3.  Use your factory method to create constraints in your tests. The following expectation specifies that the error method of the mockLogger object must be called once with an argument that is a string starting with "FATAL".

        public class MyTestCase extends MockObjectTestCase {
            ...

            public void testSomething() {
                ...

                mockLogger.expects(once()).method("error").with( aStringStartingWith("FATAL") );

                ...
            }

            private Constraint aStringStartingWith( String prefix ) {
                return new StringStartsWith(prefix);
            }
        }

An Important Rule
-----------------

**Constraint objects must be stateless.**

When [dispatching](jmock1-dispatch.html) each invocation, jMock uses the constraints to find an expectation that matches the invocation's arguments. This means that it will call the constraints many times during the test, maybe even after the expectation has been matched and invoked. In fact, jMock gives no guarantees of when and how many times it will call the constraints. This has no effect on stateless constraint objects but means that the function of stateful constraint objects cannot be predicted.

If you want to maintain state in response to invocations, use a custom [Stub](jmock1-custom-stubs.html) or [Matcher](jmock1-custom-matchers.html).
