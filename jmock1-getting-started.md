---
layout: page
---
jMock 1: Getting Started
========================

This guide assumes you are familiar with unit-testing and [JUnit](http://junit.org/).

For a simple example we are going to test a publish/subscribe message system. A Publisher sends objects to zero or more Subscribers. We want to test the Publisher, which involves testing its interactions with its Subscribers.

The Subscriber interface looks like this:

    interface Subscriber {
        void receive(String message);
    }

We will test that a Publisher sends a message to a single registered Subscriber. To test interactions between the Publisher and the Subscriber we will use a mock Subscriber object.

First we must import the jMock classes, define our test fixture class and define a test case method.

    import org.jmock.*;

    class PublisherTest extends MockObjectTestCase {
        public void testOneSubscriberReceivesAMessage() {
        }
    }

We will now write the body of the `testOneSubscriberReceivesAMessage` method.

We first *set up* the context in which our test will execute. We create a Publisher to test. We create a mock Subscriber that should receive the message. We then register the Subscriber with the Publisher. Finally we create a message object to publish.

    Mock mockSubscriber = mock(Subscriber.class);
    Publisher publisher = new Publisher();
    publisher.add( (Subscriber)mockSubscriber.proxy() );

    final String message = "message";

Next we define *expectations* on the mock Subscriber that specify the methods that we expect to be called upon it during the test run. We expect the receive method to be called with a single argument, the message that will be sent. The `eq` method is defined in the MockObjectTestCase class and specifies a "[constraint](jmock1-constraints.html)" on the value of the argument passed to the subscriber: we expect the argument to be the equal to the message, but not necessarily the same object. (jMock provides [several constraint types](jmock1-constraints.html) that can be used to precisely specify expected argument values). We don't need to specify what will be returned from the receive method because it has a void return type.

    mockSubscriber.expects(once()).method("receive").with( eq(message) );

We then *execute* the code that we want to test.

    publisher.publish(message);

After the test has finished, jMock will *verify* that the mock Subscriber was called as expected. If the expected calls were not made, the test will fail.

Here is the complete test.

    import org.jmock.*;

    class PublisherTest extends MockObjectTestCase {
        public void testOneSubscriberReceivesAMessage() {
            // set up
            Mock mockSubscriber = mock(Subscriber.class);
            Publisher publisher = new Publisher();
            publisher.add((Subscriber) mockSubscriber.proxy());
            
            final String message = "message";
            
            // expectations
            mockSubscriber.expects(once()).method("receive").with( eq(message) );
            
            // execute
            publisher.publish(message);
        }
    }

That concludes this quick introduction. More advanced topics are covered in [other tutorials](jmock1.html).
