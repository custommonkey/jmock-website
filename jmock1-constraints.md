---
layout: page
---
jMock 1: Constraints
====================

When defining expectations upon your mock objects, jMock forces you to be explicit about the argument values that will be passed to the expected methods. Expected argument values are defined using "constraints". jMock provides a set of constraints that are suitable for almost all tests. These constraints can be combined to tighten or loosen the specification if necessary. The set of constraints is [extensible](jmock1-custom-constraints.html) so you can write new constraints to cover unusual testing scenarios.

Constraints are specified by calling "syntactic sugar" methods of the MockObjectTestClass. The following constraints are defined by jMock:

Basic Constraints
-----------------

### Object Equality

The most commonly used constraint is `eq`, which specifies that the received argument must be equal to a given value. The code below, for example, specifies that the method "m" must be called with one argument of value 1.

    mock.expects(once()).method("m").with( eq(1) );

The `eq` constraint uses the `equals` method of the expected value to compare the expected and actual values for equality. Null values are checked beforehand, so it is safe to specify `eq(null)` or apply the constraint to a null actual value. The `eq` constraint is overridden for all primitive types; primitive values are boxed into objects that are then compared using the `equals` method. Arrays are treated as a special case: two arrays are considered equal by `eq` if they are the same size and all their elements are considered equal by `eq`.

### Numeric Equality with Error Margin

An overloaded version of the `eq` constraint specifies floating point values as equal to another value with some margin of error to account for rounding error. The following code specifies that the method "m" will be called with one argument of value 1 plus or minus 0.002.

    mock.expects(once()).method("m").with( eq(1, 0.002) );

### Object Identity

The `same` constraint specifies that the actual value of the argument must be the same object as the expected value. This is a tighter constraint than `eq`, but is usually what you want for arguments that pass references to behavioural objects. The following code specifies that method "m" will be called with one argument that refers to the same object as expected.

    Object expected = new Object();

    mock.expects(once()).method("m").with( same(expected) );

As a rule of thumb, use `eq` for value objects and `same` for behavioural objects.

### Instance of a Type

The `isA` constraint specifies that the actual argument must be an instance of the given type. The following code specifies that method "m" must be called with an argument that is an ActionEvent.

    mock.expects(once()).method("m").with( isA(ActionEvent.class) );

### String Contains a Substring

The `stringContains` constraint specifies that the expected argument must be a string that contains the given substring. The following code specifies that method "m" must be called with an argument that is a string containing the text "hello".

    mock.expects(once()).method("m").with( stringContains("hello") );

The `stringContains` constraint is especially useful for testing string contents but isolating tests from the exact details of punctuation and formatting. For example, the code above would accept any of the following argument values: "hello world"; "hello, world"; "hello!"; and so on.

### Null or Not Null

The constraints `NULL` and `NOT_NULL` are specify that an argument is null or is not null, respectively. These are constants, not methods. The following code specifies that method "m" must be called with two arguments, the first must be null and the second must not be null.

    mock.expects(once()).method("m").with( NULL, NOT_NULL );

### Anything

The `ANYTHING` constraint specifies that any value is allowed. This is useful for ignoring arguments that are not germane to the scenario being tested. Judicious use of the `ANYTHING` constraint can ensure that your tests are flexible and do not require constant maintenance when tested code changes. The following code specifies that the method "m" must be called with two arguments, the first of which is equal to 1 and the second of which is ignored in this test.

    mock.expects(once()).method("m").with( eq(1), ANYTHING );

Combining Constraints
---------------------

Constraints can be composed to create a tighter or looser specification. Composite constraints are themselves constraints and can therefore be further composed.

### Not — Logical Negation

The `not` constraint specifies that the actual argument must *not* meet a given constraint. The following code specifies that the method "m" must be called with an argument that is not equal to 1.

    mock.expects(once()).method("m").with( not(eq(1)) );

### And — Logical Conjunction

The `and` constraint specifies that the actual argument must meet *both* of two constraints given as arguments. The following code specifies that the method "m" must be called with a string that contains the text "hello" and the text "world".

    mock.expects(once()).method("m").with( and(stringContains("hello"),
                                               stringContains("world")) );

### Or — Logical Disjunction

The `or` constraint specifies that the actual argument must meet *either* of two constraints given as arguments. The following code specifies that the method "m" must be called with a string that contains the text "hello" or the text "howdy".

    mock.expects(once()).method("m").with( or(stringContains("hello"),
                                              stringContains("howdy")) );

More Constraints
----------------

That covers all of the constraints provided by jMock. Most of your tests will only need the basic constraints. Sometimes you may need to define constraints by composing existing constraints. Occasionally, however, you might find that no existing constraint exactly meets the need of your test. In these cases, you will have to extend jMock's set of constraints by [writing a custom constraint](jmock1-custom-constraints.html).
