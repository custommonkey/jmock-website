---
layout: page
---
Mocking Classes in Eclipse Plug-in Tests
========================================

by Ulli Hafner.

Mocking classes in Eclipse plug-ins requires some extra work (mocking interfaces works out of the box). Typically, an Eclipse application consists of several source plug-ins. The unit tests of these plug-ins are also stored in different plug-ins. E.g., if the main code of an application is stored in `com.foo.core` then the tests are stored in the plug-in `com.foo.core.tests`.

Within Eclipse you have the possibility to start your tests as JUnit test or plug-in tests. For Junit tests Eclipse simply calls the JUnit runner, tests your class and reports the results. You can use jMock as a library on your classpath without any additional configuration.

However, when running tests as plug-in tests Eclipse starts the target Eclipse platform, deploys your tests in this platform and starts these tests. Eclipse loads each plug-in from the [OSGI classloader mechanism](http://www.eclipsezone.com/articles/eclipse-vms/). Each plug-in will be loaded by an individual classloader. In this case you cannot create mocks for classes that are part of another plug-in out of the box. Since the classes to mock are loaded by a different classloader jMock cannot access them (actually cglib does not see these classes).

In order to mock classes in other plug-ins you need to use the Eclipse buddy declarations in your plug-ins. Basically, you need to add in the MANIFEST of the plug-ins that contain the actual classes the following statement: Eclipse-BuddyPolicy: registered. This statement instructs the Eclipse class loader to let other plug-ins access these classes. In your jMock test plug-in you need to add in the MANIFEST file the counterpart statement "`Eclipse-RegisterBuddy: com.foo.plugin1, com.foo.plugin2`". Now you can mock classes of the plug-ins `com.foo.plugin1` and `com.foo.plugin2` in your test cases.

Example MANIFEST of plug-in jmock.example
-----------------------------------------

    Manifest-Version: 1.0
    Bundle-ManifestVersion: 2
    Bundle-Name: Example Plug-in
    Bundle-SymbolicName: jmock.example
    Bundle-Version: 1.0.0
    Bundle-Activator: jmock.example.Activator
    Require-Bundle: org.eclipse.core.runtime
    Eclipse-LazyStart: true
    Export-Package: jmock.example
    Eclipse-BuddyPolicy: registered

Example MANIFEST of plug-in jmock.example.tests
-----------------------------------------------

    Manifest-Version: 1.0
    Bundle-ManifestVersion: 2
    Bundle-Name: Tests Plug-in
    Bundle-SymbolicName: jmock.example.tests
    Bundle-Version: 1.0.0
    Bundle-Activator: jmock.example.tests.Activator
    Require-Bundle: org.eclipse.core.runtime, org.junit, jmock.example
    Eclipse-LazyStart: true
    Eclipse-RegisterBuddy: jmock.example
    Bundle-ClassPath: ., lib/cglib-2.1_3-src.jar, lib/cglib-nodep-2.1_3.jar, lib/hamcrest-api-1.0.jar, lib/hamcrest-library-1.0.jar, lib/jmock-2.1.0.jar, lib/jmock-junit3-2.1.0.jar, lib/jmock-legacy-2.1.0.jar, lib/objenesis-1.0.jar
