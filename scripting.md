---
layout: page
---
Scripting Custom Actions
========================

Using a [custom action](custom-actions.html) in an expectation is straightforward but requires quite a lot of code. You need to define a class that implements the action and a factory method to make the expectation read clearly. The jMock Scripting Extension lets you define custom actions inline in the expectation, as [BeanShell](http://www.beanshell.org/) scripts.

Because a script is represented as a string, it does not play well with refactoring tools.

First, you need to add the following JARs to your classpath:

-   jmock-script-<span class="Version 2">2.x.x</span>.jar
-   bsh-core-2.0b4.jar

Second, you need to import the "perform" factory method into your test.

``` Java
import static org.jmock.lib.script.ScriptedAction.perform;
```

You can then use the "perform" factory method to define custom actions using a BeanShell script. The script can refer to the parameters of the mocked method by the names $0 (the first parameter), $1, $2, etc. For example, the script below calls back to a Runnable passed as the first (and only) parameter:

``` Java
checking(new Expectations() {{
    oneOf (executor).execute(with(a(Runnable.class))); will(perform("$0.run()"));
}}
```

A script can pass literal values as parameters to the callback. For example, the script below adds an integer to a collection:

``` Java
checking(new Expectations() {{
    oneOf (collector).collect(with(a(Collection.class))); will(perform("$0.add(2)"));
}}
```

If you want a script to refer to an object defined by the test, you will have to define a script variable with a "where" clause. Any number of where clauses can be appended onto the perform clause to define variables. For example, to translate the code from the [custom action recipe](custom-actions.html) to use a BeanShell script, we must define a script variable for the list of mangoes to be added to the collection:

``` Java
final FruitTree mangoTree = mock(FruitTree.class);
final Mango mango1 = new Mango();
final Mango mango2 = new Mango();

context.checking(new Expectations() {{
    oneOf (mangoTree).pickFruit(with(a(Collection.class))); will(perform("$0.addAll(mangoes)")
            .where("mangoes", Arrays.asList(mango1, mango2));
}});

...
```
