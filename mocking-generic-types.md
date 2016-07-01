---
layout: page
---
Mocking Generic Types
=====================

Java's generic type system does not work well with its dynamic reflection APIs. For jMock, this means that the compiler will warn you about possible static type errors when you mock generic types. The warnings are incorrect. The best way to avoid them is to suppress the warning with an annotation on the variable declaration of the mock object.

Consider, for example, the following generic interface:

``` Java
public interface Juicer<T extends Fruit> {
    Liquid juice(T fruit);
}
```

In a test, you would want to mock this interface as follows:

``` Java
Juicer<Orange> orangeJuicer = context.mock(Juicer<Orange>.class, "orangeJuicer");
Juicer<Coconut> coconutJuicer = context.mock(Juicer<Coconut>.class, "coconutJuicer");
```

However, that is not syntactically correct Java. You actually have to write the following:

``` Java
Juicer<Orange> orangeJuicer = (Juicer<Orange>)context.mock(Juicer.class, "orangeJuicer");
Juicer<Coconut> coconutJuicer = (Juicer<Coconut>)context.mock(Juicer.class, "coconutJuicer");
```

Both those lines, though syntactically and semantically correct, generate type safety warnings. To avoid these warnings, you have to annotate the variable declarations with `@SuppressWarnings`:

``` Java
@SuppressWarnings("unchecked")
Juicer<Orange> orangeJuicer = context.mock(Juicer.class, "orangeJuicer");

@SuppressWarnings("unchecked")
Juicer<Coconut> coconutJuicer = context.mock(Juicer.class, "coconutJuicer");
```

Hopefully future versions Java will improve the type system so that this is not necessary.
