---
layout: page
---
Expect a Sequence of Invocations
================================

Sequences are used to define expectations that must occur in the order in which they appear in the test code. A test can create more than one sequence and an expectation can be part of more than once sequence at a time.

To define a new sequence:

### Raw

``` Java
final Sequence sequence-name = context.sequence("sequence-name");
```

### JUnit 3

``` Java
final Sequence sequence-name = sequence("sequence-name");
```

### JUnit 4

``` Java
final Sequence sequence-name = context.sequence("sequence-name");
```

JMock can [auto-instantiate Sequences](auto.html) to reduce boilerplate code.

To expect a sequence of invocations, write the expectations in order and add the `inSequence(sequence)` clause to each one. For example:

``` Java
oneOf (turtle).forward(10); inSequence(drawing);
oneOf (turtle).turn(45); inSequence(drawing);
oneOf (turtle).forward(10); inSequence(drawing);
```

Expectations in a sequence can have any [invocation count](cardinality.html). If an expectation in a sequence is allowed, rather than expected, it can be skipped in the sequence.
