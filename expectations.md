---
layout: page
---
Specifying Expectations
=======================

Expectations are defined within a "Double-Brace Block" that defines the expectations in the context of the the test's Mockery (identified as context in the examples below). In outline, a jMock 2 test looks like the following:

### Raw

``` Java
public void testSomeAction() {
    ... set up ...
    
    context.checking(new Expectations() {{
        ... expectations go here ...
    }});
    
    ... code being tested ...
    
    context.assertIsSatisfied();
    
    ... other assertions ...
}
```

### JUnit 3 Integration

``` Java
public void testSomeAction() {
    ... set up ...
    
    checking(new Expectations() {{
        ... expectations go here ...
    }});
    
    ... code being tested ...
    
    ... assertions ...
}
```

### JUnit 4 Integration

``` Java
public void testSomeAction() {
    ... set up ...
    
    context.checking(new Expectations() {{
        ... expectations go here ...
    }});
    
    ... code being tested ...
    
    ... assertions ...
}
```

An expectations block can contain any number of expectations. Each expectation has the following structure:

``` Source
invocation-count (mock-object).method(argument-constraints);
    inSequence(sequence-name);
    when(state-machine.is(state-name));
    will(action);
    then(state-machine.is(new-state-name));
```

Except for the [invocation count](cardinality.html) and the mock object, all clauses are optional. You can give an expectation as many `inSequence`, `when`, `will` and `then` clauses as you wish.

The following test defines several expectations:

### JUnit3 Integration

``` Java
public void testReturnsCachedObjectWithinTimeout() {
    checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (clock).time(); will(returnValue(fetchTime));
            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
        
        oneOf (loader).load(KEY); will(returnValue(VALUE));
    }});
    
    Object actualValueFromFirstLookup = cache.lookup(KEY);
    Object actualValueFromSecondLookup = cache.lookup(KEY);
        
    assertSame("should be loaded object", VALUE, actualValueFromFirstLookup);
    assertSame("should be cached object", VALUE, actualValueFromSecondLookup);
}
```

### JUnit4 Integration

``` Java
@Test public void 
returnsCachedObjectWithinTimeout() {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (clock).time(); will(returnValue(fetchTime));
            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
        
        oneOf (loader).load(KEY); will(returnValue(VALUE));
    }});
        
    Object actualValueFromFirstLookup = cache.lookup(KEY);
    Object actualValueFromSecondLookup = cache.lookup(KEY);
        
    assertSame("should be loaded object", VALUE, actualValueFromFirstLookup);
    assertSame("should be cached object", VALUE, actualValueFromSecondLookup);
}
```

### Raw

``` Java
public void testReturnsCachedObjectWithinTimeout() {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (clock).time(); will(returnValue(fetchTime));
            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
        
        oneOf (loader).load(KEY); will(returnValue(VALUE));
    }});
        
    Object actualValueFromFirstLookup = cache.lookup(KEY);
    Object actualValueFromSecondLookup = cache.lookup(KEY);
        
    context.assertIsSatisfied();
    assertSame("should be loaded object", VALUE, actualValueFromFirstLookup);
    assertSame("should be cached object", VALUE, actualValueFromSecondLookup);
}
```

Because the expectations are defined within an anonymous inner class, any mock objects (or other values) that are stored in local variables and referenced from within the expectations block must be final. It is often more convenient to store mock objects in instance variables and define constants for the values used in the test, as the examples above do. Constants have the added benefit of making the test easier to understand than they would be if unnamed, literal values were used.

A test can contain multiple expectation blocks. Expectations in later blocks are appended to those in earlier blocks. This can be used to clarify exactly when expected calls will happen in response to calls to the object under test. For example, the test above can be rewritten as follows to more clearly express when the cache loads an object will be loaded and when it returns a cached copy:

### JUnit3 Integration

``` Java
public void testReturnsCachedObjectWithinTimeout() {
    checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (loader).load(KEY); will(returnValue(VALUE));
    }});

    Object actualValueFromFirstLookup = cache.lookup(KEY);

    checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(fetchTime));            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
    }});
    
    Object actualValueFromSecondLookup = cache.lookup(KEY);
        
    assertSame("should be loaded object", VALUE, actualValueFromFirstLookup);
    assertSame("should be cached object", VALUE, actualValueFromSecondLookup);
}
```

### JUnit4 Integration

``` Java
@Test public void 
returnsCachedObjectWithinTimeout() {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (loader).load(KEY); will(returnValue(VALUE));
    }});

    Object actualValueFromFirstLookup = cache.lookup(KEY);

    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(fetchTime));            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
    }});
    
    Object actualValueFromSecondLookup = cache.lookup(KEY);
        
    assertSame("should be loaded object", VALUE, actualValueFromFirstLookup);
    assertSame("should be cached object", VALUE, actualValueFromSecondLookup);
}
```

### Raw

``` Java
public void testReturnsCachedObjectWithinTimeout() {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (loader).load(KEY); will(returnValue(VALUE));
    }});

    Object actualValueFromFirstLookup = cache.lookup(KEY);

    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(fetchTime));            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
    }});
    
    Object actualValueFromSecondLookup = cache.lookup(KEY);
        
    context.assertIsSatisfied();
    assertSame("should be loaded object", VALUE, actualValueFromFirstLookup);
    assertSame("should be cached object", VALUE, actualValueFromSecondLookup);
}
```

Expectations do not have to be defined in the body of the test method. You can define expectations in helper methods or the `setUp` method to remove duplication or clarify the test code.

### JUnit3 Integration

``` Java
public void testReturnsCachedObjectWithinTimeout() {
    initiallyLoads(VALUE);

    Object valueFromFirstLookup = cache.lookup(KEY);

    cacheHasNotExpired();
    
    Object valueFromSecondLookup = cache.lookup(KEY);
        
    assertSame("should have returned cached object", 
               valueFromFirstLookup, valueFromSecondLookup);
}

private void initiallyLoads(Object value) {
    checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (loader).load(KEY); will(returnValue(value));
    }});
}

private void cacheHasNotExpired() {
    checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(fetchTime));            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
    }});
}
```

### JUnit4 Integration

``` Java
@Test public void 
returnsCachedObjectWithinTimeout() {
    initiallyLoads(VALUE);

    Object valueFromFirstLookup = cache.lookup(KEY);

    cacheHasNotExpired();
    
    Object valueFromSecondLookup = cache.lookup(KEY);
    
    assertSame("should have returned cached object", 
               valueFromFirstLookup, valueFromSecondLookup);
}

private void initiallyLoads(Object value) {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (loader).load(KEY); will(returnValue(value));
    }});
}

private void cacheHasNotExpired() {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(fetchTime));            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
    }});
}
```

### Raw

``` Java
public void testReturnsCachedObjectWithinTimeout() {
    initiallyLoads(VALUE);

    Object valueFromFirstLookup = cache.lookup(KEY);

    cacheHasNotExpired();
    
    Object valueFromSecondLookup = cache.lookup(KEY);
    
    context.assertIsSatisfied();
    assertSame("should have returned cached object", 
               valueFromFirstLookup, valueFromSecondLookup);
}

private void initiallyLoads(Object value) {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(loadTime));
        oneOf (loader).load(KEY); will(returnValue(value));
    }});
}

private void cacheHasNotExpired() {
    context.checking(new Expectations() {{
        oneOf (clock).time(); will(returnValue(fetchTime));            
        allowing (reloadPolicy).shouldReload(loadTime, fetchTime); will(returnValue(false));
    }});
}
```
