---
layout: page
---
jMock 1: Mocking Classes with CGLIB
===================================

Because it uses Java's standard reflection capability, the default configuration of the jMock framework can only mock interfaces, not classes. The optional `org.jmock.cglib` extension package uses the [CGLIB 2](http://cglib.sourceforge.net) library to create mock objects of classes as well as interfaces.

To use the CGLIB extension:

1.  Download [CGLIB 2](http://sourceforge.net/project/showfiles.php?group_id=56933&package_id=98218) and the [jMock CGLIB extension JAR](http://www.jmock.org/download.html) or a [full jMock source snapshot](http://www.jmock.org/download.html).
2.  Add CGLIB and the `jmock-cglib-`version`.jar` file to the CLASSPATH. If you have downloaded a source snapshot, compile the source and add the output directory to your classpath.
3.  Make your test cases extend `org.jmock.cglib.MockObjectTestCase`:
        import org.jmock.Mock;
        import org.jmock.cglib.MockObjectTestCase;

        class MyTest extends MockObjectTestCase {
            ...
        }

4.  Your tests can now create mocks of abstract or concrete classes:

        Mock mockGraphics = mock(java.awt.Graphics.class,"mockGraphics");

    You can pass arguments to the constructor by passing the argument types and values to the `mock` method:

        Mock mockedClass = mock(SomeClass.class,"mockList",
                                new Class[]{String.class, int.class},
                                new Object[]{"Hello", new Integer(1)});

The `MockObjectTestsCase` class defined in the org.jmock.cglib package is completely compatible with that of the vanilla jMock API. The only difference is that it uses CGLIB to create proxies instead of the Java reflection API. This makes it easy to convert a test class to use the CGLIB extension when you find that there is no way you can possibly avoid mocking a concrete class: just change the base class of the test class to `org.jmock.cglib.MockObjectTestsCase` and all the mock objects will then use CGLIB to proxy calls.
