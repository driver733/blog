---
date: 2019-03-31
title: Dependency injection containers vs object composition
figure: /assets/images/posts/2019/03/dependency-injection-containers-vs-object-composition/matryoshka.jpg
figcaption: © ruspeach.com
figalt: A large set of Matryoshka dolls
description: Can DI containers fully replace constructors in OOP? Absolutely not!
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
    public DataSource value() { /* ... */  }
}
```

```java
// The @Component annotation lets Spring know that this class needs
// to have its dependencies injected on app start
@Component 
class UserDao {
    @Autowired
    @Qualifier("UserDatasource")
    // injected instance from the global app context
    private final DataSource source;
    public UserDao() { this.source = source; }
}
```

Practically speaking, we have achieved the same result, however, the conceptual
difference is dramatic. Whereas in the constructor-based implementation
higher-level [entities] pass the dependencies to the lower-level ones, in the
dependency-injection containers implementation objects stand aside from each
other, relying on special classes - DI containers - to create the required
dependencies and inject them into objects that depend on them.

The code snippets below illustrate the difference between the dependency
injection using constructors and containers, respectively.

```java
// Dependency injection without DI containers
public class Application {
    public static void main(String[] args) {
        new UserVerifyJob(
            new BusinessService(
                new UserService(
                    new UserDao(
                      new UserDataSource()
                    )
                )
            )
        ).run();
    }
}
```
```java
// Dependency injection using DI containers
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
      //   Spring recursively creates
      //   all components (beans):
      //
      //   new UserVerifyJob(
      //      new BusinessService( ↑ Injected ↑
      //          new UserService( ↑ Injected ↑
      //              new UserDao( ↑ Injected ↑
      //                  new UserDataSource() ↑ Injected ↑
      //              )
      //          )
      //      )
      //   )
      //
      SpringApplication.run(Application.class, args);
    }
}
```

As you can see, the constructor-based implementation forms a hierarchy of
abstractions, a composition of objects, which serves as a path for dependencies
to reach the low-level classes. The dependency-injection container, on the other
hand, treats objects as standalone structures, which are not *explicitly*
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