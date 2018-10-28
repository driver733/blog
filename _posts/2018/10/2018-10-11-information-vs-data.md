---
date: 2018-10-11
title: Information vs Data in OOP
figure: /assets/images/posts/2018/10/information-vs-data/sat.jpg
figcaption: © Mikhail Yakushin
figalt: Data - &quot;Low SAT scores&quot;. Information - &quot;You won't get into the college of your choice&quot;
description: |
  What is data and what is information?
  What is the difference between them in programming?
  How can these concepts be applied in OOP?
keywords:
- oop
- data
- information

categories: oop
comments: true
---

The concepts of information and data have drawn a lot of attention
from the public in today's age of [big data] and [data mining].
However, the application of these terms in OOP is yet to happen.

<!--more-->

Let's start with a basic (conventionally [procedural]) example. Suppose you have been assigned a
task that requires you to determine whether the last operator the
user was chatting with is online. If the operator is online, you
pass the user's message to the operator.

Your work project utilizes [ORM], so you decide to utilize the object mappers conveniently
available in your [controller]. You end up with code that looks something this.

```java

public final class HttpRequestHandler {

    public Object responseForRequest(Request request) {
        //
        User user = usersMapper.getById(message.getUserId);
        String lastOperator = user.getLastOperator();
        Operator operDb = operatorsMapper.getById(lastOperator);
        if (operDb.status == "online") {
            // Pass message to the operator
        }
        //
    }

}

```

Let's for a minute forget about the fact that [DAO] and [DTO] have no place
in OOP and focus on another aspect of the code above. All of the variables
in the aforementioned block of code represent *data*. In other words, they represent
something which requires further analysis or processing in order to be utilized in context.
Just like I said in my previous [blog post], the project architecture reflects the
architect's way of thinking. If every task for you is about collecting the required
*data* and *processing* it in one place, then you are destined to end up with the code
looking similar to the one presented above.

What is the alternative, you might be wondering. In order to demonstrate it
we need to start from scratch. Our task was to pass the user's
message to the last operator if *the last operator is available*. A procedural
programmer looks for *data*, while an object thinker looks for *information*.
The *current availability of the last operator* is information while *user*, *operator*
and *operator status* are pieces of data. As a result we need to create an [entity],
which would provide us this *information*.

```java

public final class IsLastOperatorOnline implements Scalar<Boolean> {

    private final long user;
    private final Scalar<DataSource> database;

    public IsLastOperatorOnline(
        final Scalar<DataSource> database,
        final long user
    ) {
        this.user = user;
        this.database = database;
    }

    public Boolean value() throws Exception {
        final long count = new JdbcSession(
            this.database.value()
        ).sql(
           // SELECT COUNT(*)
           //
           // WHERE status == 'online'
        ).select(
            new SingleOutcome<>(Long.class)
        );
        return count > 0;
    }

}

```

With this implementation our http request handler changes to something like this.

```java

public final class HttpRequestHandler {

    public Object responseForRequest(Request request) {
        //
        boolean available = new IsLastOperatorOnline(
            message.getUserId()
        ).value();
        if (available) {
            // Pass message to the last operator
        }
        //
    }

}

```

Now that we working with information, rather than data, our code becomes [cleaner
and clearer]. Since all data and operations with it are encapsulated, the scope is reduced,
resulting in a lower cognitive complexity of the code, [temporal coupling] is eliminated and
maintainability is improved.

If someone's telling you that in order to improve your code you need to focus on the code
itself and everything associated with it, don't trust him. Change the way you approach
problems and your code will follow you.

[big data]:                     https://en.wikipedia.org/wiki/Big_data
[data mining]:                  https://en.wikipedia.org/wiki/Data_mining
[ORM]:                          /2018/10/28/orm-alternative.html
[controller]:                   https://www.yegor256.com/2016/12/13/mvc-vs-oop.html
[DAO]:                          https://www.yegor256.com/2017/12/05/data-access-object.html
[DTO]:                          /2018/10/08/entity-and-dto.html
[procedural]:                   /2018/07/27/props-file.html
[blog post]:                    /2018/10/08/entity-and-dto.html
[entity]:                       /2018/10/08/entity-and-dto.html
[cleaner and clearer]:          https://www.yegor256.com/2018/09/12/clear-code.html
[temporal coupling]:            /2018/10/21/dtos-lead-to-temporal-coupling.html
[improved maintainability]:     https://www.yegor256.com/2016/08/30/decomposition-of-responsibility.html