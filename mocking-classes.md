---
layout: page
---
Mocking Classes with jMock and the ClassImposteriser
====================================================

Because it uses Java's standard reflection capability, the default configuration of the jMock framework can only mock interfaces, not classes. (Actually, we consider that to be a good thing because it encourages the design to focus on communication between objects rather than static classification or data storage). However, the `ClassImposteriser` extension class uses the [CGLIB 2.1](http://cglib.sourceforge.net) and [Objenesis](http://code.google.com/p/objenesis/) libraries to create mock objects of classes as well as interfaces. This is useful when working with legacy code to tease apart dependencies between tightly coupled classes.

The `ClassImposteriser` creates mock instances *without* calling the constructor of the mocked class. So classes with constructors that have arguments or call overideable methods of the object can be safely mocked. However, the `ClassImposteriser` cannot create mocks of final classes or mock final methods.

If you want to mock final classes or final methods, the [JDave library](http://www.jdave.org) includes an [unfinalizer Instrumentation agent](http://www.jdave.org/documentation.html#mocking) that can unfinalise classes before they are loaded by the JVM. They can then be mocked by the `ClassImposteriser`.

To use the `ClassImposteriser`:

1.  Add jmock-legacy-<span class="Version 2">version</span>.jar, cglib-nodep-2.1\_3.jar and objenesis-1.0.jar to your CLASSPATH.
2.  Plug the `ClassImposteriser` into the Mockery of your test class:
    ### Raw

    ``` Source
    import org.jmock.Mockery;
    import org.jmock.Expectations;
    import org.jmock.lib.legacy.ClassImposteriser;

    public class ConcreteClassTest extends TestCase {
        private Mockery context = new Mockery() {{
            setImposteriser(ClassImposteriser.INSTANCE);
        }};
        
        ...
    }
    ```

    ### JUnit 3

    ``` Source
    import org.jmock.Expectations;
    import org.jmock.integration.junit3.MockObjectTestCase;
    import org.jmock.lib.legacy.ClassImposteriser;

    public class ConcreteClassTest extends MockObjectTestCase {
        {
            setImposteriser(ClassImposteriser.INSTANCE);
        }
        
        ...
    }
    ```

    ### JUnit 4

    ``` Source
    import org.jmock.Mockery;
    import org.jmock.Expectations;
    import org.jmock.integration.junit4.JUnit4Mockery;
    import org.jmock.lib.legacy.ClassImposteriser;

    @RunWith(JMock.class)
    public class ConcreteClassTest {
        private Mockery context = new JUnit4Mockery() {{
            setImposteriser(ClassImposteriser.INSTANCE);
        }};
        
        ...
    }
    ```

3.  Your tests can now create mocks of abstract or even concrete classes:

    ``` Source
    Graphics g = context.mock(java.awt.Graphics.class);
    ```
