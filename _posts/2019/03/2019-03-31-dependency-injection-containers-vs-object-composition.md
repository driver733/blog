---
date: 2019-03-31
title: Dependency injection containers vs object composition
figure: /assets/images/posts/2019/03/dependency-injection-containers-vs-object-composition/matryoshka.jpg
figcaption: © ruspeach.com
figalt: A large set of Matryoshka dolls
description: Can DI containers fully replace constructors in OOP? Absolutely, not!
keywords:
- oop

categories: oop
comments: true
---

[Dependency injection] is one of the core ideas of object-oriented programming.
Being an implementation of the *[Tell, don't ask]* principle, it lowers
coupling, makes software components more independent from each other, and, as a
result, increases maintainability. There two ways the dependencies can be
injected: using constructors ([object composition]) and using
[dependency injection containers].

<!--more-->

Here's how dependency injection is implemented in Java using constructors. The
application entry class creates a dependency and passes it over (through
constructors) to the classes that belong to the lower abstraction levels.

```java
class UserDb {
 
    private final DataSource source;
    
    public User(final DataSource source) {
        this.source = source;
    }
    
}
```

Now, let's see how the same result can be achieved using a dependency injection
container. I am using the [Spring Framework IoC container] for the purpose of
this example.

```java
// A DI container that creates the dependency
class UserDatasource {
    
    @Bean("UserDatasource")
    public DataSource datasource() {
    }
    
}
```

```java
// The @Component annotation lets Spring know that this class needs
// to have its dependencies injected on app start
@Component 
class UserDb {
 
    @Autowired
    @Qualifier("UserDatasource")
    private final DataSource source; // injected instance from the global app context
    
    public User() {
    }
    
}
```

Practically speaking, we have achieved the same result, however, the conceptual
difference is dramatic. Whereas in the constructor-based implementation
higher-level [entities] pass the dependencies to the lower-level ones, in the
dependency-injection containers implementation objects stand aside from each
other, relying on special classes - DI containers - to create the required
dependencies and inject them into objects that depend on them.

The diagrams below illustrate the differences between the dependency injection
using containers and constructors, respectively.

<div class="container">
    <div class="row align-items-center">
        <div class="col">
            <img src="/assets/images/posts/2019/03/dependency-injection-containers-vs-object-composition/dependency-injection-container-diagram.jpg" alt="Dependency injection container diagram">
        </div>
        <div class="col">
            <img src="/assets/images/posts/2019/03/dependency-injection-containers-vs-object-composition/object-composition-diagram.jpg" alt="Object composition diagram">
        </div>
    </div> 
</div>

As you can see, the constructor-based implementation forms a hierarchy of
abstractions, a composition of objects, which serves as a path for dependencies
to reach the low-level classes. The dependency-injection container, on the other
hand, treats objects as standalone structures, which are not explicitly
connected through the means of compositional arrangement.
 
I strongly believe that constructors are still preferable to the DI containers.
The way you compose your objects into each other, the way you manage your
dependencies allows your codebase to be expressive, allowing the reader to
easily understand what business problem your object composition solves. By
following the hierarchy of objects from abstract ones at the top to the most
concrete ones at the bottom, the reader will be able to understand the purpose
of each unit and its place in the object composition: who it depends on as well
as who depends on it. This is something that dependency-injection containers
fail at: while they do fulfill the requirement to instantiate the dependencies
and inject them where needed, they disintegrate [entities] from each other in an
attempt to setup an object-oriented application without using object
composition.

[object composition]: https://en.wikipedia.org/wiki/Composition_over_inheritance
[Tell, don't ask]: https://martinfowler.com/bliki/TellDontAsk.html
[Dependency injection]: https://en.wikipedia.org/wiki/Dependency_injection
[dependency injection containers]: https://stackoverflow.com/questions/50718586/what-is-a-di-container
[entities]: /2018/10/08/entity-and-dto.html
[Spring Framework IoC container]: https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/beans.html