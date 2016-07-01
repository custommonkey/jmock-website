---
layout: page
---
Ignoring Irrelevant Mock Objects
================================

The following expectation ignores the mock object fruitbat during the test.

``` Java
ignoring (fruitbat);
```

During the test any method of *fruitbat* can be called any number of times or not called at all.

When an ignored object is invoked during the test it returns a "zero" value that depends on the result type of the invoked method:

| Return Type    | "Zero" Value           |
|----------------|------------------------|
| boolean        | `false`                |
| numeric type   | zero                   |
| String         | "" (empty string)      |
| Array          | Empty array            |
| Mockable type  | A mock that is ignored |
| Any other type | `null`                 |

The fact that a method with a mockable return type will return another ignored mock object makes the ignoring expectation very powerful. The returned object will also return "zero" values, and any mockable values it returns will also be ignored, and any mockable values *they* return will be ignored, and so on. This allows you to focus individual tests on different aspects of an object's functionality, ignoring irrelevant collaborators.

For example, the following statement ignores the [Hibernate](http://www.hibernate.org) `SessionFactory`, and the current `Session` obtained from the `getCurrentSession()` method, and the Transaction returned from the `beginTransaction()` method and the `commit()` and `rollback()` methods of the Transaction. So you can specify the behaviour of the object independently of how it coordinates transactions and test the transaction management in other tests.

``` Java
ignoring (hibernateSessionFactory);
```
