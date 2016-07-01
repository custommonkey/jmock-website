---
layout: page
title: Getting Started
---
Getting Started
===============

In this simple example we are going to write a mock object test for a publish/subscribe message system. A Publisher sends messages to zero or more Subscribers. We want to test the Publisher, which involves testing its interactions with its Subscribers.

The Subscriber interface looks like this:

``` Source
interface Subscriber {
    void receive(String message);
}
```

We will test that a Publisher sends a message to a single registered Subscriber. To test interactions between the Publisher and the Subscriber we will use a mock Subscriber object.

Set Up the Class Path
---------------------

To use jMock <span class="Version 2">2.1.0</span> you must add the following JAR files to your class path:

-   jmock-<span class="Version 2"><span class="Version 2">2.x.x</span></span>.jar
-   hamcrest-core-1.3.jar
-   hamcrest-library-1.3.jar
-   jmock-junit3-<span class="Version 2"><span class="Version 2">2.x.x</span></span>.jar
-   jmock-junit4-<span class="Version 2"><span class="Version 2">2.x.x</span></span>.jar

Write the Test Case
-------------------

First we must import the jMock classes, define our test fixture class and create a "Mockery" that represents the context in which the Publisher exists. The context mocks out the objects that the Publisher collaborates with (in this case a Subscriber) and checks that they are used correctly during the test.

### Raw

``` Source
import org.jmock.Mockery;
import org.jmock.Expectations;

public class PublisherTest extends TestCase {
    Mockery context = new Mockery();
    ...    
}
```

This is a JUnit 3 test case but apart from the test case class the code will be the same when using any test framework for which jMock 2 does not have an integration layer.

### JUnit 3

``` Source
import org.jmock.integration.junit3.MockObjectTestCase;
import org.jmock.Expectations;

public class PublisherTest extends MockObjectTestCase {
    ...    
}
```

### JUnit 4

``` Java
import org.jmock.Expectations;
import org.jmock.Mockery;
import org.jmock.integration.junit4.JMock;
import org.jmock.integration.junit4.JUnit4RuleMockery;

public class PublisherTest {
    @Rule public JUnitRuleMockery context = new JUnitRuleMockery();
    ...
}
```

**Note**: this currently only works with the latest jMock release candidate (2.6.0RC1) and JUnit 4.7 and above.

In older versions of jMock and JUnit 4 you can use the JMock test runner, which is less flexible than the Rules mechanism shown above.

``` Java
import org.jmock.Expectations;
import org.jmock.Mockery;
import org.jmock.integration.junit4.JMock;
import org.jmock.integration.junit4.JUnit4RuleMockery;

@RunWith(JMock.class)
public class PublisherTest {
    Mockery context = new JUnit4Mockery();
    ...    
}
```

Now we want to write the method that will perform our test:

### JUnit 3

``` Java
public void testOneSubscriberReceivesAMessage() {
    ...
}
```

### JUnit 4

``` Java
@Test 
public void oneSubscriberReceivesAMessage() {
    ...
}
```

### Raw

``` Java
public void testOneSubscriberReceivesAMessage() {
    ...
}
```

We will now write the body of the test method.

We first *set up* the context in which our test will execute. We create a Publisher to test. We create a mock Subscriber that should receive the message. We then register the Subscriber with the Publisher. Finally we create a message object to publish.

### JUnit 3

``` Source
final Subscriber subscriber = mock(Subscriber.class);

Publisher publisher = new Publisher();
publisher.add(subscriber);

final String message = "message";
```

### JUnit 4

``` Source
final Subscriber subscriber = context.mock(Subscriber.class);

Publisher publisher = new Publisher();
publisher.add(subscriber);

final String message = "message";
```

### Raw

``` Source
final Subscriber subscriber = context.mock(Subscriber.class);

Publisher publisher = new Publisher();
publisher.add(subscriber);

final String message = "message";
```

Next we define *[expectations](expectations.html)* on the mock Subscriber that specify the methods that we expect to be called upon it during the test run. We expect the receive method to be called once with a single argument, the message that will be sent.

### JUnit 3

``` Source
checking(new Expectations() {{
    oneOf (subscriber).receive(message);
}});
```

### JUnit 4

``` Source
context.checking(new Expectations() {{
    oneOf (subscriber).receive(message);
}});
```

### Raw

``` Source
context.checking(new Expectations() {{
    oneOf (subscriber).receive(message);
}});
```

We then *execute* the code that we want to test.

``` Source
publisher.publish(message);
```

After the code under test has finished our test must *verify* that the mock Subscriber was called as expected. If the expected calls were not made, the test will fail. <span class="JUnit3">The `MockObjectTestCase` does this automatically. You don't have to explicitly verify the mock objects in your tests.</span> <span class="JUnit4">The `JMock` test runner does this automatically. You don't have to explicitly verify the mock objects in your tests.</span>

``` Java
context.assertIsSatisfied();
```

Here is the complete test.

### JUnit 3

``` Source
import org.jmock.integration.junit3.MockObjectTestCase;
import org.jmock.Expectations;

public class PublisherTest extends MockObjectTestCase {
    public void testOneSubscriberReceivesAMessage() {
        // set up
        final Subscriber subscriber = mock(Subscriber.class);

        Publisher publisher = new Publisher();
        publisher.add(subscriber);
        
        final String message = "message";
        
        // expectations
        checking(new Expectations() {{
            oneOf (subscriber).receive(message);
        }});

        // execute
        publisher.publish(message);
    }
}
```

### JUnit 4

``` Source
import org.jmock.integration.junit4.JMock;
import org.jmock.integration.junit4.JUnit4Mockery;
import org.jmock.Expectations;

@RunWith(JMock.class)
public class PublisherTest {
    Mockery context = new JUnit4Mockery();
    
    @Test 
    public void oneSubscriberReceivesAMessage() {
        // set up
        final Subscriber subscriber = context.mock(Subscriber.class);

        Publisher publisher = new Publisher();
        publisher.add(subscriber);
        
        final String message = "message";
        
        // expectations
        context.checking(new Expectations() {{
            oneOf (subscriber).receive(message);
        }});

        // execute
        publisher.publish(message);
    }
}
```

### Raw

``` Source
import org.jmock.Mockery;
import org.jmock.Expectations;

public class PublisherTest extends TestCase {
    Mockery context = new Mockery();

    public void testOneSubscriberReceivesAMessage() {
        // set up
        final Subscriber subscriber = context.mock(Subscriber.class);

        Publisher publisher = new Publisher();
        publisher.add(subscriber);
        
        final String message = "message";
        
        // expectations
        context.checking(new Expectations() {{
            oneOf (subscriber).receive(message);
        }});

        // execute
        publisher.publish(message);
        
        // verify
        context.assertIsSatisfied();
    }
}
```

Where Next?
-----------

The jMock library is explored in more depth in [other Cookbook recipes](cookbook.html). The [Cheat Sheet](cheat-sheet.html) is an overview of the entire jMock API.
