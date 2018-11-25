---
date: 2018-10-28
title: "In search of an ORM alternative: Creating a SQL-speaking object"
figure: /assets/images/posts/2018/10/orm-alternative/concrete-spalling.jpg
figcaption: Data and logic are inseparable. © Sydney Strata Specialists
figalt: Concrete spalling
description: |
  What SQL-speaking objects are and how create them?
  Why SQL-speaking objects are better than ORM?
keywords:
- oop
- orm

categories: orm
comments: true
---

The disadvantages of [Object-Relational Mapping] (ORM) have [already been discussed] by a number of developers.
The main argument against ORM is that it leads to the creation of [DTOs], which encapsulate [data, rather than behavior].
If one decides to accept this claim, what alternative solutions does he have? One of them is a *[SQL-Speaking object]*.

<!--more-->

Suppose you need to create a [XMPP] connection for each user in a certain database table without using ORM or DTOs.
First, you need to fetch the users from the database table. I am using [jcabi-jdbc] for database interactions
and [jooq] to create SQL queries. The *Connections* object is used to cache connections.

```java
public final class Visitors implements Scalar<List<Scalar<XmppConnection>>> {

    private final Connections connections;
    private final Scalar<DataSource> database;
    // Constructor
    public List<Scalar<XmppConnection>> value() throws Exception {
        return new JdbcSession(
            this.database.value()
        ).sql(
            DSL.select(
                DSL.field("user_id"),
            ).from(
                DSL.table("users")
            ).toString()
        ).select(
            new OutcomeListVisitor(
                connections
            )
        );
    }
}
```

Next, you create a *Visitor* object for each user. This object creates a XMPP connection using the provided *User ID*.

```java
public final class OutcomeListVisitor implements Outcome<List<Scalar<XmppConnection>>> {

    private final Connections connections;
    // Constructor
    public List<Scalar<XmppConnection>> handle(
        final ResultSet rset,
        final Statement stmt
    ) throws SQLException {
        final List<Scalar<XmppConnection>> result = new ArrayList<>(
            rset.getFetchSize()
        );
        while (rset.next()) {
            result.add(
                new Visitor(
                    connections,
                    rset.getInt("user_id")
                )
            );
        }
        return result;
    }
}
```

The *Visitor* object uses the provided *Connections* to create a reusable XMPP connection for the user.

```java
public final class Visitor implements Scalar<XmppConnection> {

    private final Connections connections;
    // Constructor
    public XmppConnection value() throws Exception {
        return this.connections.connection(
           this.user
        );
    }
}
```

Looking over this example, the pervasive usage of ORM becomes questionable. The creation of each abstraction must
be guided by the responsibility that can be delegated to it. **ORM, on the contrary, pollutes the codebase with numerous dummy
data containers**, [which are managed by a controller]. In other words, ORM gives much less than you except from it:
Instead of providing [entities] that can used independently from each other, ORM forces us to split data and behavior
by storing data in a DTO and logic in a controller. **A *SQL-Speaking object*, on the other hand, encapsulates
data and behavior together**, lowering the scope and coupling and increasing coherency.

In essence, I strongly believe that the presence of ORM and DTOs is one of the indicators of a poor architectural design.
A sequence of steps required to produce a certain result should be composed of independent and self-sufficient
entities. ORM separates each step into components, which lowers coherency as data and the operations with it are split apart.


[Object-Relational Mapping]: https://en.wikipedia.org/wiki/Object-relational_mapping
[already been discussed]: https://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html
[SQL-Speaking object]: https://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html#sql-speaking-objects
[DTOs]: /2018/10/08/entity-and-dto.html
[data, rather than behavior]: /2018/10/11/information-vs-data.html
[XMPP]: https://en.wikipedia.org/wiki/XMPP
[jcabi-jdbc]: https://www.yegor256.com/2014/08/18/fluent-jdbc-decorator.html
[jooq]: https://www.jooq.org
[which are managed by a controller]: /2018/10/21/dtos-lead-to-temporal-coupling.html
[entities]: /2018/10/08/entity-and-dto.html