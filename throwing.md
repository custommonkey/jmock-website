---
layout: page
---
Throwing Exceptions from Mocked Methods
=======================================

Use the throwException action to throw an exception from a mocked method.

``` Java
allowing (bank).withdraw(with(any(Money.class))); 
    will(throwException(new WithdrawalLimitReachedException());
```

JMock will detect if you try to throw a checked exception that is incompatible with the invoked method and fail the test with a descriptive error message. It allows you to throw any RuntimeException or Error from any method.

``` Java
allowing (bank).withdraw(Money.ZERO); 
    will(throwException(new IllegalArgumentException("you cannot withdraw nothing!");
```
