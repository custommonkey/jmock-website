---
layout: page
---
Coding Conventions
==================

Here are some rules that we follow when developing jMock:

-   Never check in a class without associated unit tests.
-   Never check a failing unit test into the CVS repository.
-   Acceptance tests may fail. A failing acceptance test indicates something that needs to be done: a todo item, feature request or bug report, for example. It's ok to check a failing acceptance test into the CVS repository.
-   Separate acceptance and unit tests. Make it easy to run only the unit tests so that we can check that they pass before committing, even when acceptance tests are failing.
-   Resolve and remove all TODO comments before checking in.
-   Avoid the following words in class names and other identifiers: Helper, Impl (or a derivative), Manager.
-   Follow the Sun: use Sun's [coding conventions for Java](http://java.sun.com/docs/codeconv/).
-   Use the [TestDox](http://agiledox.sourceforge.net/) naming conventions for unit tests.
