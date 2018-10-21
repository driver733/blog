---
date: 2018-10-21
title: Why do DTOs lead to temporal coupling?
figure: /assets/images/posts/2018/10/dtos-lead-to-temporal-coupling/similar-houses.jpg
figcaption: DTOs in practice. © writingfordesigners.com
figalt: Many very similar houses placed close to each other
description: |
  Why DTOs are evil?
  Why do DTOs result in temporal coupling?
  Why DTOs increase code complexity and decrease maintainability?
keywords:
- oop
- dto
- maintainability

categories: oop
comments: true
---

In the comments section of [one my recent articles] I have been [asked]
the following question: "Why do [DTOs] lead to temporal coupling?". Seriously, why?
Have we not been using them for years in Java without critically thinking about what
could be wrong with them?

<!--more-->

As always, let's start with a short piece of code that demonstrates the topic of discussion.

```java

    StudentDao studentDao = new StudentDaoImpl();
    //update student
    Student student = studentDao.getAllStudents().get(0);
    student.setName("Michael");
    studentDao.updateStudent(student);

```
*Courtesy of TutorialsPoint.com*

This block of code shows a very typical way to use DTO in the context of DAO operations.
At first sight, the ability to fetch data in a form of an object, update it, and then
send it back seems lucrative. However, let's look at this code again from the perspective
of OOP and [Elegant Objects].

In the presented context, it is visible that a certain [controller] acts upon the *Student*
DTO in various ways. First, it changes the name of the student by using a [setter]. Next,
it sends the updated data to the database, by using a DAO instance, which extracts the
data fields from the *Student* DTO. What exactly is wrong here and why DTO is the one to blame?

1. First of all, being nothing, but a plain data container, the *Student* **DTO is a dependent object**.
Being able to provide *data*, but not *functionality* or *behavior* (a combination of data and
context) it loses independence as an [entity]. As a result, the *Student* DTO, just like any other DTO
(read-only or not), requires a context before it can be used. The object which provides context is
conventionally called *controller*.

2. Requiring *someone to act upon it*, due to its surrogate nature, **DTO becomes a utility
component of a controller**, the necessity of which is created by the DTO itself.
Consequently, the controller becomes an object that collects various DTOs, works with them however
it needs to and then throws them away.

3. The consequences of the procedural nature of the controller bring us
back to the beginning of the article (The connection between DTOs and temporal coupling).
**The more DTOs the controller works with, the more complex it becomes and the more lines of code it
ends up having**.

By working with multiple data containers in one place, **a controller becomes a
procedural block of code** that is intolerant to any changes, leading to **increased temporal coupling
and complexity** (due to the increased context, introduced by different DTOs) **and lower maintainability**
as a consequence.


[DTOs]:                     /2018/10/08/entity-and-dto.html
[one my recent articles]:   /2018/10/11/information-vs-data.html
[asked]:                    http://disq.us/p/1wln4wi
[Elegant Objects]:          https://www.elegantobjects.org
[controller]:               https://www.yegor256.com/2016/12/13/mvc-vs-oop.html
[entity]:                   /2018/10/08/entity-and-dto.html
[setter]:                   https://www.yegor256.com/2014/09/16/getters-and-setters-are-evil.html
