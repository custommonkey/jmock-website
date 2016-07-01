---
layout: page
title: Articles and Papers
---
Articles and Papers
===================

[Evolving an Embedded Domain-Specific Language in Java](oopsla2006.pdf)
-----------------------------------------------------------------------

Steve Freeman, Nat Pryce

In Proc. OOPSLA 2006

This paper describes the experience of evolving a domain-specific language embedded in Java over several generations of the jMock test framework. We describe how the framework changed from a library of classes to an embedded language. We describe the lessons we have learned from this experience for framework developers and language designers.

[Mock Roles, Not Objects](oopsla2004.pdf)
-----------------------------------------

Steve Freeman, Nat Pryce, Tim Mackinnon, Joe Walnes

In Proc. OOPSLA 2004

Mock Objects is an extension to Test-Driven Development that supports good Object-Oriented design by guiding the discovery of a coherent system of types within a code base. It turns out to be less interesting as a technique for isolating tests from third-party libraries than is widely thought. This paper describes the process of using Mock Objects with an extended example and reports best and worst practices gained from experience of applying the process. It also introduces jMock, a Java framework that embodies our collective experience.

[jMock: Yoga for your Unit Test](yoga.html)
-------------------------------------------

Nat Pryce

Brittle tests are a common "gotcha" of test driven development. A brittle unit test will stop passing when you make unrelated changes to your application code. A test suite that contains a lot of brittle tests will slow down development and inhibit refactoring. A brittle test overspecifies the behaviour of the unit being tested. You can keep your tests flexible by following a simple rule of thumb: specify exactly what you want to happen *and no more*. This article describes how various features of jMock can help you strike the right balance between an accurate specification of a unit's required behaviour and a flexible test that allows easy evolution of the code base.
