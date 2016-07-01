---
layout: page
---
Mocking GWT Asynchronous Services
=================================

You can use jMock to unit test objects that use asynchronous GWT services by using a custom action to capture the asynchronous callback passed to the service request. Once it has captured the callback, the action can invoke it to indicate success or failure of the asynchronous call.

Here's an implementation of this idea:

``` Java
public class AsyncAction<T> implements Action {
    private AsyncCallback<T> callback;

    public void succeedGiving(T result) {
        checkCallback();
        callback.onSuccess(result);
    }

    public void fail(Throwable caught) {
        checkCallback();
        callback.onFailure(caught);
    }

    public void describeTo(Description description) {
        description.appendText("requests an async callback");
    }

    private void checkCallback() {
        if (callback == null) {
            Assert.fail("asynchronous action not scheduled");
        }
    }

    @SuppressWarnings("unchecked")
    public Object invoke(Invocation invocation) throws Throwable {
        callback = (AsyncCallback<T>)invocation.getParameter(invocation.getParameterCount()-1);
        return null;
    }
}
```

You can use it in tests like this:

``` Java
ServiceAsync service = context.mock(ServiceAsync.class, "service");
ObjectUnderTest objectUnderTest = new ObjectUnderTest(service);

@Test
public void loadsScenariosAsynchronously() throws Exception {
    final AsyncAction<String> makeAsyncRequest = new AsyncAction<String>();

    context.checking(new Expectations() {{
        oneOf(service).getSomething(with(any(AsyncCallback.class))); will(makeAsyncRequest);
    }});
    
    objectUnderTest.doSomething();
    makeAsyncRequest.succeedGiving("foo");
    
    assertThat(objectUnderTest.thingThatItGotAsynchronously(), equalTo("foo"));
}
```
