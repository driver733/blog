---
date: 2020-02-24
title: "Objects and functions - friends or foes? Neither!"
figure: /assets/images/posts/2020/02/object-and-functions-friends-or-foes-neither/Functional-Programming-vs-OOP.png
figcaption: © educba.com
figalt: Functional programming vs OOP
description: OOP and functional programming are often contrasted, but maybe they should be compared instead?
keywords:
- oop

categories: oop
comments: true
---

Programmers often view [object-oriented] and [functional programming] as two completely different paradigms. As a result,
some developers advocate for one them, pointing out the advantages of their favorite paradigm and criticizing the
"opposing" one. However, I have rarely seen anyone *comparing* OOP with functional programming instead of
*contrasting* them.

<!--more-->

Let's look at this piece of Kotlin code, which filters and then maps the elements of the given list:

```kotlin
val result = listOf("Adam", "Bob", "Boris")
    .filter { it.startsWith("B") }
    .map { it.length }
```

This is how the code above usually looks like in classic Java, one of the most popular OOP languages:

```java
List<String> originalList = Arrays.asList("Adam", "Bob", "Boris");
List<String> filteredList = new ArrayList<>();
List<String> mappedList = new ArrayList<>();

for (String name : originalList) {
    if (name.startsWith("B")) {
        filteredList.add(employee);
    } 
}

for (String name : filteredList) {
    mappedList.add(name.length());
}
```

Is there any way we could *compare* this code snippet with the initial one? I don't think so. However, we can
and should rather *contrast* it. This seems quite apparent, right? The reason for it is that this piece of Java code is
not object-oriented, but *imperative*.

The *object-oriented* Java code should like this:

```java
List<String> result = new ListOf<>(
    new Mapped<>(
      s -> s.length(),
      new Filtered<>(
          s -> s.startsWith("B"),
          new IterableOf<>("Adam", "Bob", "Boris")
        ),
    )
);
// The "ListOf", "Mapped" and "Filtered" classes are from the
// "yegor256/cactoos" java lib
```
 
This [solution] is very similar to the initial functional one, although the paradigms are completely different, which brings
us to the main point of the article - **OOP and functional programming are two paradigms that solve the given problem
in a *declarative* manner through a *composition* of smaller logical blocks**. In other words, *what* functional and OOP
are doing is the same (a composition of smaller logical blocks), but *how* they are doing it is different
(OOP relies on [objects], while functional programming relies on functions).

I believe that by reapproaching OOP with the idea of **adding new functionality *declaratively*, *incrementally*
and through a *composition* of small logical blocks**,
we can not only improve our code, but also to change the way we analyze the problems we encounter (from imperative to
declarative). In addition, it can enable us to revisit the OOP patterns (such as the *Decorator* pattern demonstrated above)
to see which of them apply to this concept and how.

The next time you use an OOP language to solve a problem, try to do it *declaratively* and *incrementally*, and then
see if what you come up with can *compare* to the code snippets in this article. 


[functional programming]: https://en.wikipedia.org/wiki/Functional_programming
[object-oriented]: https://en.wikipedia.org/wiki/Object-oriented_programming
[solution]: https://www.yegor256.com/2015/02/26/composable-decorators.html
[objects]: /2018/07/27/props-file.html
