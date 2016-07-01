---
layout: page
---
Automagically Creating Mocks, States & Sequences
================================================

If you define fields to hold mock objects, states or sequences , jMock can automagically instantiate them, reducing boilerplate code.

To automagically create a mock object, annotate the field with `@Mock`. JMock will initialise the field to hold a new mock object with the same name as the field before the test runs.

For example, the following declarations create two mock objects named "cheddar" and "jarlsberg".

``` Java
@Mock Cheese cheddar;
@Mock Cheese jarlsberg;
```

To automagically create [State](states.html) and [Sequence](sequences.html) objects, annotate the fields with `@Auto`. Again, jMock will initialise the field with a new object named after the field before the test runs:

``` Java
@Auto States pen;
@Auto Sequence events;
```
