---
layout: page
---
jMock 1: Writing Custom Stubs
=============================

A mock object does two things: it tests that it receives the expected method invocations and it stubs the behaviour of those methods. Almost all methods can be stubbed in one of three basic ways: return nothing (a void method), return a result to the caller or throw an exception. Sometimes, however, you need to stub side effects of a method, such as a method that calls back to the caller through an object reference passed as one of its parameters. Luckily, jMock makes it easy to write custom stubs for methods with strange behaviours and keep your tests easy to read when you do so.

Here's a simple example: we want to test a `NameCollector` object that collects the names of a collection `Named` objects. It will do so by passing a `List` to the `collectName` method of the named objects. The named objects implement `collectName` by addding their name to the list they receive. The `Named` interface is shown below:

    public interface Named {
        void collectName( List list );
    }

To test the behaviour of our object we will need to mock the `Named` interface, and in particular we need to stub the *side effect* of the `collectName` method. jMock provides stubs for returning values and throwing exceptions but we will have to write our own class to stub the side effect of the `Named` objects.

To write a stub:

1.  Write a class that implements the `Stub` interface. Here's a stub that adds an element to a List received as the first argument of the method.

        public class AddToListStub implements Stub
        {
            Object element;

            public AddToListStub( Object element ) {
                this.element = element;
            }

            public StringBuffer describeTo( StringBuffer buffer ) {
                return buffer.append("adds <"+element+"> to a list");
            }

            public Object invoke( Invocation invocation ) throws Throwable {
                ((List)invocation.parameterValues.get(0)).add(element);
                return null;
            }
        }

2.  Write a factory method in your tests class that instantiates your new stub class.

        private Stub addListElement( Object newElement ) {
            return new AddToListStub(newElement);
        }

3.  Call your factory method within the actual parameter list of the mock's `will` method.

        Mock mockNamed = mock(Named.class);
        String name = "name";

        mockNamed.expects(once()).method("collectName").with(NOT_NULL)
            .will(addListElement(name));
