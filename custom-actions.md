---
layout: page
---
Writing New Actions
===================

JMock expectations do two things: test that they receives the expected method invocations and stub the behaviour of those methods. Almost all methods can be stubbed in one of three basic ways: return nothing (a void method), return a result to the caller or throw an exception. Sometimes, however, you need to stub side effects of a method, such as a method that calls back to the caller through an object reference passed as one of its parameters. Luckily, jMock makes it easy to write custom stubs for methods with unusual behaviours and keep your tests easy to read when you do so.

Here's a simple example: we want to test a `FruitPicker` object that collects the fruit from a collection `FruitTree` objects. It will do so by passing a `Collection` to the `pickFruit` method of the fruit trees. The fruit trees implement `pickFruit` by addding their fruit to the collection they receive. The `FruitTree` interface is shown below:

``` Source
public interface FruitTree {
    void pickFruit(Collection<Fruit> collection);
}
```

To test the behaviour of our object we will need to mock the `FruitTree` interface, and in particular we need to stub the *side effect* of the `pickFruit` method. jMock provides actions for returning values and iterators, throwing exceptions and grouping other actions but we will have to write our own action class to stub the side effect.

To write an action:

1.  Write a class that implements the `Action` interface. Here's an action that adds an element to a Collection received as the first argument of the method.

    ``` Source
    public class AddElementsAction<T> implements Action {
        private Collection<T> elements;
        
        public AddElementsAction(Collection<T> elements) {
            this.element = elements;
        }
        
        public void describeTo(Description description) {
            description.appendText("adds ")
                       .appendValueList("", ", ", "", elements)
                       .appendText(" to a collection");
        }
        
        public Object invoke(Invocation invocation) throws Throwable {
            ((Collection<T>)invocation.parameterValues.get(0)).addAll(elements);
            return null;
        }
    }
    ```

2.  Write a factory method somewhere that instantiates your new action class and import it into your test. You could make the factory method a public static method of the new action class or could put all your own factory methods into a single class to make them easier to import. The factory method doesn't have to be a static import. If you only need it in one test you can write it as a method of that test.

    ``` Source
    public static <T> Action addElements(T... newElements) {
        return new AddElementsAction<T>(Arrays.asList(newElements));
    }
    ```

3.  Pass the result of your factory method to the `will` method.

    ``` Source
    final FruitTree mangoTree = mock(FruitTree.class);
    final Mango mango1 = new Mango();
    final Mango mango2 = new Mango();

    context.checking(new Expectations() {{
        oneOf (mangoTree).pickFruit(with(any(Collection.class)));
            will(addElements(mango1, mango2));
    }});

    ...
    ```
