---
layout: page
---
Match Objects or Methods
========================

Although [matchers](matchers.html) are normally used to [specify acceptable parameter values](parameters.html), they can also be used to specify acceptable objects or methods in an expectation, using an API syntax similar to that of jMock 1. To do so, use a matcher where you would normally refer to a mock object directly in the [invocation count clause](cardinality.html). Then chain clauses together to define the expected invocation. The following clauses are supported:

|                        |                                                                         |
|------------------------|-------------------------------------------------------------------------|
| `method(m)`            | A method matching the Matcher m is expected.                            |
| `method(r)`            | A method with a name that matches the regular expression r is expected. |
| `with(m1, m2, ... mn)` | The parameters must match matchers m<sub>1</sub> to m<sub>n</sub>.      |
| `withNoArguments()`    | There must be no parameters                                             |

An expectation of this form can be followed by ordering constraints ([sequences](sequences.html) and [states](states.html)) and actions (e.g. [returning values](returning.html) or [throwing exceptions](throwing.html)) like a normal, literal expectation.

### Examples

To allow invocations of any bean property getter on any mock object:

    allowing (any(Object.class)).method("get.*").withNoArguments();

Allow a method to be called once on any one of a collection of mock objects:

    oneOf (anyOf(same(o1),same(o2),same(o3))).method("doSomething");
