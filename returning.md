---
layout: page
---
Returning Values from Mocked Methods
====================================

Unless you explicitly specify otherwise, jMock will return an appropriate value from any methods that do not have a void return type. In most tests you will need to explicitly define the value returned from a mocked invocation.

The Simple Case
---------------

You can return values from mocked methods by using the `returnValue` action within the "`will`" clause of an expectation.

``` Source
oneOf (calculator).add(2, 2); will(returnValue(5));
```

jMock will fail the test if you try to return a value of the wrong type.

Returning Iterators over Collections
------------------------------------

The `returnIterator` action returns an iterator over a collection.

``` Java
final List<Employee> employees = new ArrayList<Employee>();
employees.add(alice);
employees.add(bob);

context.checking(new Expectations() {{
    oneOf (department).employees(); will(returnIterator(employees));
}});
```

A convenient overload of the returnIterator method lets you specify the elements inline:

``` Java
context.checking(new Expectations() {{
    oneOf (department).employees(); will(returnIterator(alice, bob));
}});
```

Note the difference between using `returnIterator` and using `returnValue` to return an iterator. Using `returnValue` will return the same iterator each time: once all the iterator's elements have been consumed, further calls will return the same, exhausted iterator. The `returnIterator` action returns a new iterator each time it is invoked.

Returning Different Values on Consecutive Calls
-----------------------------------------------

There are two ways to return different values on different calls. The first is to define multiple expectations and return a different value from each:

``` Java
oneOf (anObject).doSomething(); will(returnValue(10));
oneOf (anObject).doSomething(); will(returnValue(20));
oneOf (anObject).doSomething(); will(returnValue(30));
```

The first invocation of `doSomething` will return 10, the second 20 and the third 30.

However, this duplicates the definition of the expected invocation and so increases maintenance effort. A better approach is to use the `onConsecutiveCalls` action to return a different value (or perform a different action) on different invocations.

``` Source
atLeast(1).of (anObject).doSomething();
   will(onConsecutiveCalls(
       returnValue(10),
       returnValue(20),
       returnValue(30)));
```

The expectation is expected at least 1 time, rather than 3 times, to make it easier to add more results later if we want to.
