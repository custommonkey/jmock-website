---
layout: page
---
Writing New Matchers
====================

jMock and [Hamcrest](http://code.google.com/p/hamcrest) provide many Matcher classes and factory functions that let you [specify the acceptable parameter values of a method invocation](matchers.html). However, sometimes the predefined constraints do not let you specify an expectation accurately enough to convey what you mean or to keep your tests flexible. In such cases, you can easily define new matchers that seamlessly extend the existing set defined by jMock.

A Matcher is an object that implements the [org.hamcrest.Matcher](http://code.google.com/p/hamcrest/source/browse/trunk/hamcrest-java/hamcrest-core/src/main/java/org/hamcrest/Matcher.java) interface. It does two things:

-   reports whether a parameter value meets the constraint (the `matches` method).
-   generates a readable description to be included in test failure messages (the `describeTo` method inherited from the [SelfDescribing](http://code.google.com/p/hamcrest/source/browse/trunk/hamcrest-java/hamcrest-core/src/main/java/org/hamcrest/SelfDescribing.java) interface).

To create a new matcher:

1.  Write a class that extends Hamcrest's BaseMatcher or [TypeSafeMatcher]() base classes. The following matcher class tests whether a string starts with a given prefix.

        import org.hamcrest.AbstractMatcher;

        public class StringStartsWithMatcher extends TypeSafeMatcher<String> {
            private String prefix;

            public StringStartsWithMatcher(String prefix) {
                this.prefix = prefix;
            }

            public boolean matchesSafely(String s) {
                return s.startsWith(prefix);
            }

            public StringBuffer describeTo(Description description) {
                return description.appendText("a string starting with ").appendValue(prefix);
            }
        }

2.  Write a well-named factory method that creates an instance of your new matcher.

        @Factory
        public static Matcher<String> aStringStartingWith( String prefix ) {
            return new StringStartsWithMatcher(prefix);
        }

    The point of the factory method is to make the test code read clearly, so consider how it will look when used in an expectation.

3.  Use your factory method to create matchers in your tests. The following expectation specifies that the error method of the `logger` object must be called once with an argument that is a string starting with "FATAL".

        public class MyTestCase {
            ...

            public void testSomething() {
                ...

                context.checking(new Expectations() {{
                    oneOf (logger).error(with(aStringStartingWith("FATAL")));
                }});
                ...
            }
        }

An Important Rule
-----------------

**Matcher objects must be stateless.**

When [dispatching](dispatch.html) each invocation, jMock uses the matchers to find an expectation that matches the invocation's arguments. This means that it will call the matchers many times during the test, maybe even after the expectation has already been matched and invoked. In fact, jMock gives no guarantees of when and how many times it will call the matchers. This has no effect on stateless matchers but means that the function of stateful matchers cannot be predicted.

If you want to maintain state in response to invocations, use an [Action](custom-actions.html), not a Matcher.

More Information
----------------

More documentation about how to write Matchers is available from the [Hamcrest project](http://code.google.com/p/hamcrest/).
