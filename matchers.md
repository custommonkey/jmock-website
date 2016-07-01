---
layout: page
---
Matchers
========

Most of the time expectations specify literal parameter values that are compared for equality against the actual parameters of invoked methods. For example:

``` Java
allowing (calculator).add(2,2); will(returnValue(5));
```

Sometimes, however, you will need to define looser constraints over parameter values to clearly express the intent of the test or to ignore parameters (or parts of parameters) that are not relevant to the behaviour being tested. For example:

``` Java
allowing (calculator).sqrt(with(lessThan(0)); will(throwException(new IllegalArgumentException());
oneOf (log).append(with(equal(Log.ERROR)), with(stringContaining("sqrt"));
```

Loose parameter constraints are defined by specifying *matchers* for each parameter. Matchers are created by factory methods, such as `lessThan`, `equal` and `stringContaining` in the example above, to ensure that the expectation is easy to read. The result of each factory method must be wrapped by a call to the `with` method.

An expectation that uses parameter matchers must use the "with" method to wrap *every* parameter, whether a matcher function or a literal value.

Factory methods for commonly used matchers are defined in the `Expectations` class. More matchers are defined as static methods in the `org.hamcrest.Matchers` class. If you need these, statically import thse Matchers into your test code:

``` Java
import static org.hamcrest.Matchers.*;
```

Matchers can be combined to tighten or loosen the specification if necessary. The set of matchers is [extensible](custom-matchers.html) so you can write new matchers to cover unusual testing scenarios.

Basic Matchers
--------------

### Object Equality

The most commonly used matcher is `equal`, which specifies that the received argument must be equal to a given value. The code below, for example, specifies that the method "doSomething" must be called with one argument of value 1.

    oneOf (mock).doSomething(with(equal(1)));

The `equalTo` constraint uses the `equals` method of the expected value to compare the expected and actual values for equality. Null values are checked beforehand, so it is safe to specify `equal(null)` or apply the matcher to a null actual value. Arrays are treated as a special case: two arrays are considered equal if they are the same size and all their elements are considered equal.

### Object Identity

The `same` matcher specifies that the actual value of the argument must be the same object as the expected value. This is a tighter constraint than `equal`, but is usually what you want for arguments that pass references to behavioural objects. The following code specifies that method "doSomething" will be called with one argument that refers to the same object as expected.

    Object expected = new Object();

    oneOf (mock).doSomething(with(same(expected)));

As a rule of thumb, use `equal` for value objects and `same` for behavioural objects.

### Substring

The `stringContaining` matcher specifies that the expected argument must be a string that contains the given substring. The following code specifies that method "doSomething" must be called with an argument that is a string containing the text "hello".

    oneOf (mock).doSomething(with(stringContaining("hello")));

The `stringContaining` matcher is especially useful for testing string contents but isolating tests from the exact details of punctuation and formatting. For example, the code above would accept any of the following argument values: "hello world"; "hello, world"; "hello!"; and so on.

### Null or Not Null

The matchers `aNull<T>(Class<T>)` and `aNonNull<T>(Class<T>)` specify that an argument is null or is not null, respectively. The following code specifies that method "doSomething" must be called with two Strings, the first must be null and the second must not be null.

    oneOf (mock).doSomething(with(aNull(String.class)), aNonNull(String.class)));

### Anything

The `any<T>(Class<T>)` constraint specifies that any value of the given type is allowed. This is useful for ignoring arguments that are not relevant to the scenario being tested. Judicious use of the `any` constraint ensure that your tests are flexible and do not require constant maintenance when tested code changes. The following code specifies that the method "doSomething" must be called with two arguments, the first of which is equal to 1 and the second of which is ignored in this test.

    oneOf (mock).doSomething(with(eq(1)), with(any(String.class)));

### Numeric Equality with Error Margin

An overloaded version of the `equal` constraint specifies floating point values as equal to another value with some margin of error to account for rounding error. The following code specifies that the method "doSomething" will be called with one argument of value 1 plus or minus 0.002.

    oneOf (mock).doSomething(with(equal(1, 0.002)));

Combining Matchers
------------------

Matchers can be composed to create a tighter or looser specification. Composite matchers are themselves matchers and can therefore be further composed.

### Not — Logical Negation

The `not` matcher specifies that the actual argument must *not* match a given matcher. The following code specifies that the method "doSomething" must be called with an argument that is not equal to 1.

    oneOf (mock).doSomething(with(not(eq(1)));

### AllOf — Logical Conjunction

The `allOf` matcher specifies that the actual argument must meet *all* of the matchers given as arguments. The following code specifies that the method "doSomething" must be called with a string that contains the text "hello" and the text "world".

    oneOf (mock).doSomething(with(allOf(stringContaining("hello"),
                                      stringContaining("world"))));

### AnyOf — Logical Disjunction

The `anyOf` matcher specifies that the actual argument must meet *at least one* of the matchers given as arguments. The following code specifies that the method "doSomething" must be called with a string that either contains the text "hello" or the text "howdy".

    oneOf (mock).doSomething(with(anyOf(stringContains("hello"),
                                      stringContains("howdy"))));
