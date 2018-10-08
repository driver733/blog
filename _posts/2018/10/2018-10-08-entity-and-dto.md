---
layout: post
date: 2018-10-08
title: "Entity and DTO: What's the difference?"
description: |
  What is an entity? What is a DTO?
  What is the actual difference between them?
  What makes the entity an entity?
keywords:
- oop
- eo

categories: oop
comments: true
---

DTO ([Data Transfer Object]) has long been a source of [discussions] on the subject of it's place
in OOP. Most often, the debate occurs on the topic of practical difference between a DTO and an
[entity] (an object which represents a real-world subject). Some programmers, who are against DTO
as a concept, have been able to demonstrate through convincing arguments and [examples] why DTOs
should be avoided. However, the question of [practical alternatives] and common mistakes in them
still remains open.

<!--more-->

In this article I will compare and contrast entity and DTO using practical examples and will also
show how the former can become a viable alternative to the latter.

![](/assets/images/posts/2018/10/entity-vs-dto/mac-vs-pc.jpg)
*© Mac vs PC. Apple.*

I will start with a common data retrieval example, which I have found on another [blog].
Suppose your java application receives key-value information in JSON format. In order to
process it, you decide to use a common approach - DTO.


```java

// JSON received
{
  "mark": "BMW",
  "hp": 200,
  "model": "X5",
  "price": 30.000,
  "currency": "USD"
}

// DTO for the JSON
public class Car {

  private String mark;
  private int hp;
  private String model;
  private int price;
  private String currency;

  //getters and setters
}

```

Then you have discovered for yourself the idea of [Elegant Objects] and have decided to
improve your code and move away from DTOs by eliminating setters and by encapsulating the received
JSON and exposing it's contents through interface methods instead of getters.

```java

public class JsonCar implements Car {

  private JsonObject car;

  public String mark() {
    return this.car.getString("mark");
  }
  //other accessor methods
}

```

In order to evaluate the refactored code in comparison to the original one (DTO) and in terms of
an object becoming an entity we first have to recall the [definition] of what DTO is:

> The difference between data transfer objects and business objects or data access objects is that
> a DTO does not have any behavior except for storage, retrieval, serialization and deserialization
> of its own data (mutators, accessors, parsers and serializers).
> In other words, DTOs are simple objects that should not contain any business logic but may contain
> serialization and deserialization mechanisms.

I hope that your thoughts match mine: Although the object design has been changed, it is still a DTO.
Why? Because the refactored object's responsibilities still match the ones of the original object.
The refactored object only encapsulates the JSON data and allows us to read it field by field
using the keys. This is exactly what the original DTO did.

You might be wondering: "What is an entity then? And how exactly does it look like?"
An entity, in contrast to DTO, is a subject to which you can *delegate a responsibility*,
which takes a form of *action*. While a DTO is more similar to a drawer, which gives you
access to the tax documents, an entity is an accountant who you call and ask if the taxes are paid
in time and in the right manner.

You sure want to see the object we have been working with in a form of entity. Here it is.

```java

public interface Car {
    void race();
}

```

```java

public class CarBasic implements Car {

    private final String mark;
    private final int hp;
    private final String model;

    public CarBasic(final String mark, final int hp, final String model) {
      this.mark = mark;
      this.hp = hp;
      this.model = model;
    }

    public void race() {
        //
    }

}

```


```java

public class CarDb implements Car {

  private int id;
  private String table;
  private DataSource database;

  public DbCar(final int id, final String table, final DataSource database) {
    this.id = id;
    this.table = table;
    this.database = database;
  }

  public Car car(){
    final Car car = new JbdbSession(this.database).sql(
        "SELECT mark, hp, model" +
        "FROM " + this.table +
        "WHERE id = " + this. id
    ).select(
        (ResultSet rset, Statement stmt) {
            final String mark = rset.getString("mark");
            final int hp = rset.getInt("hp");
            final String model = rset.getString("model");
            return new CarBasic(mark, hp, model);
        }
    );
    return car;
  }

}

```

Do you the difference? Now our object is all about *responsibility* and *action*.
When we are dealing with a `Car`, the only moment we care about data is when we create
the object. After that, the data is used by the object internally for the action
that the object is responsible for.

The battle between entity and DTO is about the way you approach problems, rather
than code: Do you think about interfaces or data first? Do you layout the actions that
you need to have to be done first or you analyze the incoming JSON data and then fill
your objects with it? The next time you touch the keyboard in the IDE stop for a second
and think what you are going to write will end up with something that can race or another
instance of the [Map] interface.


[Data Transfer Object]:     https://martinfowler.com/eaaCatalog/dataTransferObject.html
[discussions]:              https://www.yegor256.com/2016/07/06/data-transfer-object.html
[entity]:                   https://www.yegor256.com/2016/07/14/who-is-object.html
[examples]:                 https://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html
[practical alternatives]:   https://www.driver733.com/2018/07/27/props-file.html
[blog]:                     https://www.amihaiemil.com/2017/09/01/data-should-be-animated-not-represented.html
[Elegant Objects]:          https://www.elegantobjects.org
[definition]:               https://en.wikipedia.org/wiki/Data_transfer_object
[Map]:                      https://docs.oracle.com/javase/8/docs/api/java/util/Map.html
