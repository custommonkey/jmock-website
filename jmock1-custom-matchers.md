---
layout: page
---
jMock1: Writing Custom Invocation Matchers
==========================================

Invocation matchers control how a mock object matches and verifies incoming invocations against expectations and stubbed behaviour. The jMock API is a convenient syntactic sugar for specifying how matchers are added to a mock object. Sometimes the convenient API does not allow you to specify matching rules precisely or flexibly enough. In these cases you will need to write your own invocation matcher classes. This article will show you how to do just that.

A mock object contains a list of expectations (and stubs, but we can consider a stub to be a degerate form of expectation and jMock uses the same type of object to implement both). When a mock object receives an invocation it searches for an expectation to process the invocation, starting with the most recently specified expectation and working backwards until it finds match. Each expectation contains matching rules that control which invocations it matches. An expectation matches an invocation if all of its matching rules match the invocation. The jMock API provides a concise, descriptive syntax for adding expectations to a mock and adding matching rules to expectations.

An matching rule is an object that implements the `InvocationMatcher` interface. It has the following responsiblities:

-   report whether it matches against an invocation (the `matches` method).
-   check and record information about a dispatched invocation, if required (the `invoked` method).
-   verify that expected invocations have actually been received. (the `verify` method from the `Verifiable` interface).
-   provide a readable description to be included in test failure messages (the `describeTo` method from the `SelfDescribing` interface).
-   report if its description is not an empty string (the `hasDescription` method).

In most cases, you will want to define a new matching rule that does change state when an invocation occurs. Also, you should always provide a description for your matching rule, so the `hasDescription` method should always return `true`. This common behaviour is already implemented in the abstract `StatelessInvocationMatcher` class. You can therefore define a new matching rule by extending `StatelessInvocationMatcher` and defining the `matches` and `describeTo` methods.

An Example Custom Matcher
-------------------------

Suppose we want to stub all Java Bean property getters to return some default results. Returning default results is performed by a `DefaultResultStub`. But how do we match property getter methods? We need to write a custom `InvocationMatcher` that only matches invocations of bean property getter methods. Here's what a test will look like that uses such a custom matcher:

    InvocationMatcher beanPropertyGetters = new BeanPropertyGetterMatcher();
    DefaultResultStub returnDefaultValue = new DefaultResultStub();
    Mock mock = ...

    public void testListChildren() {
        ...
        mockPerson.stubs().match(beanPropertyGetters).will(returnDefaultValue);
        ...
    }

We can write the `BeanPropertyGetterMatcher` class by extending `StatelessInvocationMatcher`:

    public class BeanPropertyGetterMatcher extends StatelessInvocationMatcher {
       ...
    }

We now need to write the `matches` method to return whether the invocation is the getter method of a Java Bean property, using the Java Beans API. The method will use Bean introspection to list the property descriptors of the class that defines the invoked method. It then searches the list of property descriptors for a descriptor with a getter method that is the same as the invoked method.

    public boolean matches(Invocation invocation) {
        Class beanClass = invocation.invokedMethod.getDeclaringClass();

        try {
            BeanInfo beanInfo = Introspector.getBeanInfo(beanClass);
            PropertyDescriptor[] properties = beanInfo.getPropertyDescriptors();

            for( int i = 0; i < properties.length; i++ ) {
                if( invocation.invokedMethod.equals( properties[i].getReadMethod() ) ) return true;
            }
            return false;
        }
        catch( IntrospectionException ex ) {
            throw new AssertionFailedError("could not introspect bean class" + beanClass + ": " + ex.getMessage() );
        }
    }

Finally we must write the `describeTo` method to generate a description of what our new class matches. The description will be included in the description of any expectation that uses the matcher, which itself will be included in error messages when tests fail. Our new class matches any bean property getter, so that's what our description will be:

    public StringBuffer describeTo(StringBuffer buffer) {
        return buffer.append("any bean property getter");
    }

The entire matcher class is shown below:

    public class BeanPropertyGetterMatcher extends StatelessInvocationMatcher {

        public boolean matches(Invocation invocation) {
            Class beanClass = invocation.invokedMethod.getDeclaringClass();

            try {
                BeanInfo beanInfo = Introspector.getBeanInfo(beanClass);
                PropertyDescriptor[] properties = beanInfo.getPropertyDescriptors();

                for( int i = 0; i < properties.length; i++ ) {
                    if( invocation.invokedMethod.equals( properties[i].getReadMethod() ) ) return true;
                }
                return false;
            }
            catch( IntrospectionException ex ) {
                throw new AssertionFailedError("could not introspect bean class" + beanClass + ": " + ex.getMessage() );
            }
        }

        public StringBuffer describeTo(StringBuffer buffer) {
            return buffer.append("any bean property getter");
        }
    }
