---
layout: page
---
Matching Parameter Values
=========================

You can specify different actions to be performed when a method is invoked with different parameters:

``` Java
allowing (calculator).add(1,1); will(returnValue(3));
allowing (calculator).add(2,2); will(returnValue(5));
allowing (calculator).sqrt(-1); will(throwException(new IllegalArgumentException());
```

You can specify that the same method can be invoked a different number of times with different parameters:

``` Java
one   (calculator).load("x"); will(returnValue(10));
never (calculator).load("y");
```

Actual parameters match if they are equal to the expected parameters. For most values, equality is determined by the `equals` method. Null values are checked beforehand, so it is safe to pass `null` as an expected or actual parameter value. Arrays are treated as a special case and compared recursively: two arrays are considered equal if they are the same size and all their elements are considered equal. If you want to ignore parameters or match parameter values in a different way, you will need to define [matchers](matchers.html) for each parameter of the expectation.
