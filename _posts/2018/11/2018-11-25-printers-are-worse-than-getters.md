---
date: 2018-11-25
title: Printers are even worse than getters
figure: /assets/images/posts/2018/11/printers-are-worse-than-getters/viktor-chernomyrdin.jpg
figcaption: "Viktor Chernomyrdin: \"We wanted the best, you know the rest\" © aif.ru"
figalt: Viktor Chernomyrdin - the author of the quote We wanted the best, you know the rest
description: |
  Using printers for data access is even worse than accessor methods (getters).
keywords:
- oop
- getters

categories: oop
comments: true
---

The [well-known] disadvantages of getters have already been analyzed by a number of programmers.
[Yegor Bugayenko] has gone even further and suggested a workaround.
The pattern called [Printer] gives indirect access to the internal parts of an object by
serializing it into one of the data formats, such XML or JSON and reading it's content
using an XML or a JSON reader. As a result, as Yegor states, the object stays intact and it's
encapsulation is not violated by getters. Although this approach might seem viable at first
sight, I believe that it does not solve the fundamental problem of accessor methods.

<!--more-->

Let's start with an example that Yegor uses in his [Printer] article:

```java
public class Book {
  private final long isbn = 123456;
  private final String title = "Object Thinking";
  public String toJSON() {
    return String.format(
      "{\"isbn\":\"%d\", \"title\":\"%s\"}",
      this.isbn, this.title
    );
  }
}
```

The `Book` object provides a JSON `printer`. However, what this method actually does
is that it serializes the object into JSON data format.

The process of reading serialized data is straightforward:

```java
final JsonReader reader = Json.createReader(
    new StringReader(
        book.toJSON()
    )
);
final long isbn = reader.getLong("isbn");
final String title = reader.getString("title");
```

My argument against the so-called `printers` is that they do not actually differ from the
accessor methods. Does it matter, conceptually, that we access object's data directly
through getters or indirectly through JSON? I believe not.

**The real problem is data extraction process itself**. The questions we should ask here are:
"Why do need to take away data from the `Book`? Why do we treat the `Book` as a data storage container?
Why does the `Book` have no behavior or functionality?" After we answer these questions the `Book`
object will no longer exist in its original form.

I have an alternative to the `Printer` technique. (I am using [jcabi-jdbc] and [jcabi-aspects] libraries)

```java
public interface Book {
    String title();
    long isbn();
}
```

```java
public class BookById implements Book {

	private final DataSource database;
	private final int id;

	public BookById(final int id) {
	    this.id = id;
	}

	public String title() {
	    return this.resultSet().getString("title");
	}

	public long isbn() {
	    return this.resultSet().getLong("isbn");
	}

	@Cacheable(forever = true)
	private ResultSet resultSet() {
	    return new JdbcSession(
	        database.value()
	    ).sql(
            DSL.select(
                DSL.field("isbn"), DSL.field("title")
            ).from(
                DSL.table("books")
            ).where(
                DSL.field("id").equal(this.id)
            ).toString()
        ).select(
            (rset, stmt) -> {
                if (rset.next()) {
                    return rset;
                }
                throw new IllegalStateException(
                	String.format(
                		"Book with id %d not found!",
                		this.id
                	)
                );
            }
        );
    }

}
```


```java
final Book book = new BookById(1);
String title = book.title();
long isbn = book.isbn();
```

The `BookById` object is an [entity] that encapsulates specific behavior. The object design
is clear and transparent: all logic is kept inside the object, and data is not extracted through getters
or serialization. The process of retrieving [results] from the object is as simple as calling
one method. **A true object is a functionality provider that exposes its behavior through a contract** (interface),
not a data container with getters or an external microservice that sends you JSON or XML strings.

[well-known]: https://www.yegor256.com/2014/09/16/getters-and-setters-are-evil.html
[Printer]: https://www.yegor256.com/2016/04/05/printers-instead-of-getters.html
[Yegor Bugayenko]: https://www.yegor256.com/about-me.html
[jcabi-jdbc]: https://jdbc.jcabi.com/example-select.html
[jcabi-aspects]: https://aspects.jcabi.com/annotation-cacheable.html
[entity]: /2018/10/08/entity-and-dto.html
[results]: /2018/10/11/information-vs-data.html