---
date: 2019-01-12
title: How to motivate developers to write tests
figure: /assets/images/posts/2019/01/how-to-motivate-developers-to-write-tests/plane-crash.jpg
figcaption: © USA Today
figalt: Plane crash in the ocean.
description: If the project manager does not force the developers to write tests, who will? I believe that it is the project itself.
keywords:
- testing

categories: testing
comments: true
---

Creating a motivation for developers to make unit testing part
of the software development process can be challenging.
Although tests can be made a requirement, without clear understanding of
the practical purpose of testing, developers might write tests only
to pass the code review and keep the code coverage metrics high.
However, if the project manager does not force the developers to write tests, who will?
I believe that it is the project itself.

<!--more-->

Imagine a software developer, John, who has recently joined a new project.
He certainly needs time to get himself acquainted with the software
practices that take place in the project: How do developers write their code?
How do they verify it works and works correctly? And so on. However, the
management hired John because it needs to release the product more
more frequently and fix bugs faster, so it cannot give newcomers free time
just to get themselves comfortable. As a result, sooner than he realizes, he gets
a ticket assigned to him.

The ticket asks John to refactor one of the classes, so that it can properly
handle the recently added database table fields. John is familiar with the database
interactions, so he completes the task very soon. However, John is hesitated
to push his changes as he has not checked that they work properly. He realizes
that he has no idea how to run the project locally, how to execute
the code he has altered or how to verify that these changes are correct
and have no errors. He feels lost by the questions to which he has no answer:
"Do I need to run a database instance locally or to connect to a remote one?",
"How do I set up a configuration file, so that it starts the scheduled job to run and
trigger my code?", "What configuration profile should I use locally and what are the differences
between them?" At last, John decides to jump into the "test" folder to see if its contents
can give him some guidance.

Lucky for him, John finds multiple test suits, which verify the operation of the
database interaction layer. He also notices that these test suites have
database migration scripts and docker scripts attached to them. His [IDE]
also "recognizes" these tests and allows him to run them with a click of a button.
Intuition guides John to see if he can use an existing test to help him check his code.
After some time, he finds the right test, fixes it to match the new changes and runs it.

John's experience showcases a number of important practices associated with tests.

1. **Project design and configuration must lead the developers towards using
tests for verification of any made changes.**

    Instead of instructing every developer how to set up the necessary environment to run the project locally, create
    test suites that can automatically set up the necessary environment before running the tests and
    teardown it afterwards. (using [TestContainers] or [DbUnit], for example). With every
    pull request the amount of tests will naturally grow, giving the developers
    (and the management) the ability to introduce changes easily with the support of the
    test safety net.

2. **Tests must be treated as a project knowledge base.**

    Every added test will take away the information from the developers and make the
    project more team-independent. Whenever a practical example of how
    to use a certain complex class is needed, developers will refer to
    the tests, rather than colleagues (who might have already left the company),
    wiki (which might be out of date) or endless trial and error experiments.

3. **All unit tests must pass before submitting a pull request.**

    By running tests every time a new piece of code is added we put trust in our project,
    rather than in our people. Without passing all tests beforehand, developers must not
    be able to apply to the code review process, which will quickly get them
    used to run all unit tests before pushing their local changes to the remote repository.


[IDE]: https://en.wikipedia.org/wiki/Integrated_development_environment
[TestContainers]: https://github.com/testcontainers/testcontainers-java
[DbUnit]: http://dbunit.sourceforge.net
