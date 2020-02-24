---
date: 2019-03-18
title: Extension methods are not OOP
figure: /assets/images/posts/2019/03/extension-methods-are-not-oop/twilight-zone-nightmare-at-20000-feet.jpg
figcaption: © The Twilight Zone (Nightmare at 20,000 Feet)
figalt: William Shatner with Nick Cravat as the gremlin
description: Extension methods (introduced by Kotlin and Swift) is an evil anti-pattern, which has nothing in common with OOP.
keywords:
- oop

categories: oop
comments: true
---

[Extension method] is a special language feature introduced by Kotlin, Swift and
other programming languages, which allows developers to add new methods to
otherwise unchangeable classes. While at first it seems like a great idea, in
reality it contradicts the core OOP concepts and ideas.

<!--more-->

First, let's see how it can be used in Kotlin.

```kotlin
fun String.escapeXml() : String {
  return this
    .replace("&", "&amp;")
    .replace("<", "&lt;")
    .replace(">", "&gt;")
}
```

[String], as you may know, is a final class in Java. As a result, whenever
additional functionality is required, developers usually place it in
[utility classes]. With extension methods, new instance methods can be
seamlessly injected into any class.

```java
// Java
StringUtils.escapeXml("&<>");

// Kotlin
"&<>".escapeXml("&<>")
```

This seems to be a great feature, right? No doubts, calling an instance method
is more intuitive compared to a static one. In addition, no new classes have to
be created. Although extension methods might seem like a viable alternative to
[utility classes], in reality it is a terrible anti-pattern which violates OOP
practices in a number of ways.

1. **The main idea of OOP is to create new classes for new functionality.**

    The core idea of OOP is to improve code maintainability by [reducing scope]
    as much as possible. Therefore, whenever a new kind functionality is needed,
    it should be placed in a new class, which implements one or more interfaces.
    
    An apparent OOP solution to the aforementioned problem of escaping a XML
    string, would be a simple [decorator] class.
    
    ```java
    public class StringEscapedXml implements Supplier<String> {
        
        private final String origin;
        
        public StringEscapedXml(final String origin) {
            this.origin = origin;
        }
        
        public String get() {
            return origin
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;");
        }
        
    }
    ```     

2. **Extension methods violate encapsulation by injecting new properties into
   objects.**
   
   OOP resides on an idea that [entities] must be [fully encapsulated] and that
   encapsulation must never be broken. However, some language features violate
   encapsulation. For instance, [reflection] allows us to peek inside an object,
   see its private properties and their values. Similar to reflection, extension
   methods break encapsulation by injecting new properties (in the form of
   methods) into otherwise independent and unchangeable [objects].

3. **Extension methods split class declaration apart.**
    
    After reading documentation on extension methods in Kotlin, your first
    question would probably be: "Where the heck do I write them?". The answer is
    \- anywhere! That's right, Kotlin ([as well as Swift]) allows you to place
    static functions and extension methods [in any code file] that belongs to
    the project classpath (just like in C or C++). And, yes, you are right again
    \- now you cannot treat class declaration as the only source of instance or
    static methods.

[utility classes]: https://www.yegor256.com/2014/05/05/oop-alternative-to-utility-classes.html
[entities]: /2018/10/08/entity-and-dto.html
[objects]: /2018/07/27/props-file.html
[reducing scope]: https://www.yegor256.com/2019/03/12/data-and-maintainability.html
[fully encapsulated]: https://g4s8.github.io/fully-encapsulated/
[reflection]: https://en.wikipedia.org/wiki/Reflection_(computer_programming)
[in any code file]: https://kotlinlang.org/docs/reference/functions.html#function-scope
[as well as Swift]: https://docs.swift.org/swift-book/LanguageGuide/Functions.html
[String]: https://docs.oracle.com/javase/8/docs/api/java/lang/String.html
[Extension method]: https://en.wikipedia.org/wiki/Extension_method
[decorator]: https://www.yegor256.com/2015/02/26/composable-decorators.html