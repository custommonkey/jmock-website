---
layout: page
---
Upgrading from jMock 1 to jMock 2
=================================

You can add both jMock 1 and jMock 2 JARs to the classpath at the same time without class names clashing. This allows you to run jMock 1 and 2 tests simultaneously. You can write new tests with jMock 2 and gradually port older tests to the new version as you find the need.

### Beware!

Make sure your test fixture extends the correct base class: extend `org.jmock.MockObjectTestCase` in jMock 1 tests and `org.jmock.integration.junit3.MockObjectTestCase` in jMock 2 tests.
