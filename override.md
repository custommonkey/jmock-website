---
layout: page
---
Overriding Expectations Defined in the Test Set-Up
==================================================

If you [define expectations](expectations.html) in the set-up of a test in order to put the object under test into a known state, you sometimes want to ignore those expectations during the test itself. For example, you might *allow* calls to some mock objects during set-up and then want to *expect* calls to those mock objects during the test. If you just define additional expectations in the test, the allowing expectations defined in the set-up will [take precedence](dispatch.html) and the test's expectations will never seem to be met, thereby failing the test.

In this situation, use a [state machine](states.html) to control when the expectations are valid. This state machine represents the state of the test, rather than the state of the objects with which the object under test communicates.

1.  Define a state machine named, for example, "test".
2.  In the test set-up, define expectations that are valid when the "test" state machine is not in the "fully-set-up" state.
3.  At the end of the test set-up, move the "test" state machine into the "fully-set-up" state.
4.  Define expectations in your tests as usual.

The result is that the expectations defined in the set-up to only apply when the test is not "running" will be ignored during the test itself.

### JUnit 3

``` Java
public class ChildTest extends MockObjectTestCase {
    States test = states("test");

    Parent parent = mock(Parent.class);

    // This is created in setUp
    Child child;
    
    @Override
    public void setUp() {
        checking(new Expectations() {{
            ignoring (parent).addChild(child); when(test.isNot("fully-set-up"));
        }});
        
        // Creating the child adds it to the parent
        child = new Child(parent);
        
        test.become("fully-set-up");
    }
    
    public void testRemovesItselfFromOldParentWhenAssignedNewParent() {
        Parent newParent = context.mock(Parent.class, "newParent");
        
        checking(new Expectations() {{
            oneOf (parent).removeChild(child);
            oneOf (newParent).addChild(child);
        }});
        
        child.reparent(newParent);
    }
}
```

### JUnit 4

``` Java
@RunWith(JMock.class)
public class ChildTest {
    Mockery context = new JUnit4Mockery();
    States test = mockery.states("test");

    Parent parent = context.mock(Parent.class);

    // This is created in setUp
    Child child;
    
    @Before
    public void createChildOfParent() {
        mockery.checking(new Expectations() {{
            ignoring (parent).addChild(child); when(test.isNot("fully-set-up"));
        }});
        
        // Creating the child adds it to the parent
        child = new Child(parent);
        
        test.become("fully-set-up");
    }
    
    @Test
    public void removesItselfFromOldParentWhenAssignedNewParent() {
        Parent newParent = context.mock(Parent.class, "newParent");
        
        context.checking(new Expectations() {{
            oneOf (parent).removeChild(child);
            oneOf (newParent).addChild(child);
        }});
        
        child.reparent(newParent);
    }
}
```

### Raw

``` Java
public class ChildTest extends TestCase {
    Mockery context = new Mockery();
    States test = context.states("test");

    Parent parent = context.mock(Parent.class);

    // This is created in setUp
    Child child;
    
    @Override
    public void setUp() {
        context.checking(new Expectations() {{
            ignoring (parent).addChild(child); when(test.isNot("fully-set-up"));
        }});
        
        // Creating the child adds it to the parent
        child = new Child(parent);
        
        test.become("fully-set-up");
    }
    
    public void testRemovesItselfFromOldParentWhenAssignedNewParent() {
        Parent newParent = context.mock(Parent.class, "newParent");
        
        context.checking(new Expectations() {{
            oneOf (parent).removeChild(child);
            oneOf (newParent).addChild(child);
        }});
        
        child.reparent(newParent);
    }
}
```
