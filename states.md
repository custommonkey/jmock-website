---
layout: page
---
States
======

States are used to specify that invocations can occur only during some state that is initiated and/or terminated by other expected invocations. A test can define multiple state machines and an invocation can be constrained to occur during a state of one more more state machines.

To define a new state machine:

### Raw

``` Java
final States state-machine-name = context.states("state-machine-name").startsAs("initial-state");
```

### JUnit 3

``` Java
final States state-machine-name = states("state-machine-name").startsAs("initial-state");
```

### JUnit 4

``` Java
final States state-machine-name = context.states("state-machine-name").startsAs("initial-state");
```

The intial state is optional. If not specified, the state machine starts in an unnamed initial state.

JMock can [auto-instantiate States](auto.html) to reduce boilerplate code.

The following clauses constrain invocations to occur within specific states and define how an invocation will change the current state of a state machine.

|                                          |                                                                                                 |
|------------------------------------------|-------------------------------------------------------------------------------------------------|
| when(state-machine.is("state-name"));    | Constrains the last expectation to occur only when the state machine is in the named state.     |
| when(state-machine.isNot("state-name")); | Constrains the last expectation to occur only when the state machine is not in the named state. |
| then(state-machine.is("state-name"));    | Changes the state of state-machine to the named state when the invocation occurs.               |

For example:

``` Java
final States pen = context.states("pen").startsAs("up");
oneOf (turtle).penDown(); then(pen.is("down"));
oneOf (turtle).forward(10); when(pen.is("down"));
oneOf (turtle).turn(90); when(pen.is("down"));
oneOf (turtle).forward(10); when(pen.is("down"));
oneOf (turtle).penUp(); then(pen.is("up"));
```
