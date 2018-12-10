---
date: 2018-12-09
title: DAO in the world of Elegant Objects
figure: /assets/images/posts/2018/12/dao-in-the-world-of-elegant-objects/dao-symbol.jpg
figcaption: Dao (chinese philosophy) symbol © New World Encyclopedia
figalt: Dao (chinese philosophy) symbol
description: DAO implementation using SQL-speaking objects
keywords:
- oop
- dao
- dto

categories: oop
comments: true
---

DAO ([Data Access Object]) is an abstraction that is used for [CRUD] database operations. In Java,
DAO is typically implemented as an interface that declares the methods through which a programmer
can interact with a database table. A [DTO] in the form of a [POJO] object is used by DAO as a data container,
which passes data from a programmer to the database and vice-a-versa.

After analysing the [criticism] about DAO, I have decided to write out my personal thoughts on this subject.
In this article I will create an implementation of DAO through the perspective of [SQL-speaking objects].

<!--more-->

Let's imagine that we need to create an abstraction for interacting with
`Employee`s and `Department`s database tables. Each `Employee` has an *id*, a *name* and a *department*,
to which he belongs. At the same time, each `Department` has an *id* and a *title*.

Here's a representation of each of these entities in a form of an interface:

```java
interface Department {
    int id();
    String title();
    void updateTitle(final String title);
}
```

```java
interface Employee {
    int id();
    String name();
    void updateName(final String name);
    Department department();
}
```

A collection of `Employee`s belonging to a specific department can be represented as follows.

```java
interface Employees {
    Collection<Employee> employees();
}
```

The requirement for a class that implements the `Employee` interface is to provide
the content of the fields of a specific `Employee` as well as an abstraction
that represents the department he belongs to. The conditions by which an
`Employee` can be found are provided through the constructors.

The `DepartmentDb` class is implemented in the same fashion as the `EmployeeDb` class.

```java
final class EmployeeDb implements Employee {

    private final DataSource database;
    private final Condition condition; // org.jooq.Condition

    public EmployeeDb(final DataSource database, final int id) {
        this(database, DSL.field("id").eq(id));
    }

    public EmployeeDb(final DataSource database, final String name) {
        this(database, DSL.field("name").eq(name));
    }

    private EmployeeDb(final DataSource database, final Condition condition) {
        this.database = database;
        this.condition = condition;
    }

    public long id() {
        this.resultSet().getLong("id");
    }

    public String name() {
        this.resultSet().getString("name");
    }

    public void updateName(final String name) {
        new JdbcSession(
                database.value()
            ).sql(
                DSL.update(
                    DSL.table("employees")
                ).set(
                    DSL.field("name"), name
                ).where(
                    this.condition
                ).toString()
            ).update(Outcome.VOID);
    }

    public Department department() {
        return new DepartmentDb(this.resultSet().getInt("department_id"));
    }

    @Cacheable(forever = true)
    private ResultSet resultSet() {
        return new JdbcSession(
            database.value()
        ).sql(
            DSL.select(
                DSL.field("id"), DSL.field("name"), DSL.field("department_id")
            ).from(
                DSL.table("employees")
            ).where(
                this.condition
            ).toString()
        ).select(
            (rset, stmt) -> {
                if (rset.next()) {
                    return rset;
                }
                throw new IllegalStateException(
                    String.format(
                        "Employee with condition \"%s\" not found",
                        condition.toString
                    )
                );
            }
        );
    }

}
```

The user of the `EmployeeDb` class, in contrast to the traditional DAO implementation,
will not need to manually fetch the `Department` by using a *department_id* in an additional
DAO operation. Instead, he will be able to get all the details using the provided interfaces.

```java
// Traditional DAO implementation
EmployeeDto employeeDto = employeesDao.findByName("Michael");
int departmentId = employeeDto.getDepartmentId();
DepartmentDto departmentDto = departmentsDao.findById(departmentId);
String departmentTitle = departmentDto.getTitle();
```

```java
// My way
Department department = new EmployeeDb("Michael").department();
String depatmentTitle = department.title();
```

The `EmployeesDb` class takes in a *department_id* and returns a collection
of `Employee`s that belong to it.

```java
final class EmployeesDb implements Employees {

    private final DataSource database;
    private final Condition condition; // org.jooq.Condition

    public EmployeesDb(final DataSource database, final int departmentId) {
        this(database, DSL.field("department_id").eq(departmentId));
    }

    private EmployeesDb(final DataSource database, final Condition condition) {
        this.database = database;
        this.condition = condition;
    }

    public Collection<Employee> employees() {
        return new JdbcSession(
            database.value()
        ).sql(
            DSL.select(
                DSL.field("id"), DSL.field("name")
            ).from(
                DSL.table("employees")
            ).where(
                this.condition
            ).toString()
        ).select(
            (rset, stmt) -> {
                final List<Employee> employees = new ArrayList<>();
                while (rset.next()) {
                    employees.add(
                        new EmployeeDbCached(
                            new EmployeeDb(
                                this.database,
                                rset.getInt("id")
                            ),
                            rset.getInt("id"),
                            rset.getString("name")
                        )
                    );
                }
                return employees;
            }
        );
    }

}
```

This is how we can use it to find all employees, which belong to the specific department.

```java
int departmentId = 1;
Collection<Employee> employees = new EmployeesDb(departmentId).employees();
Employee employee = employees.iterator.next();
String name = employee.name();
```

A programmer who works with the `EmployeesDb` class receives `Employee`s
in a form of the `Employee` interface. In reality, he is working with an `Employee`,
which has all internal properties pre-fetched. However, in case the details of a
foreign-key entity are needed, a request to the database will be made to retrieve them.
This logic is implemented using the *[Decorator]* pattern.

```java
final class EmployeeDbCached implements Employee {

    private final Employee employee;
    private final int id;
    private final String name;
    private final Condition condition; // org.jooq.Condition

    public EmployeeDb(final Employee origin, final int id, final String name) {
        this.origin = origin;
        this.id = id;
        this.name = name;
    }

    public long id() {
        return this.id;
    }

    public String name() {
        return this.name;
    }

    public void updateName(final String name) {
        this.origin.updateName(name);
    }

    public Department department() {
        return this.origin.department();
    }

}
```

I believe that the aforementioned implementation has a number of advantages over a
standard DAO implementation in Java:

1. **Absence of DTOs** (as well as getters and setters)

    Most DAOs enforce using DTOs for all interactions with a database. As a result,
    getters and setters pollute the codebase and [introduce temporal coupling], since they have to be used to extract
    data from a DTO received from DAO, to create a new DTO and fill it with data or to
    alter the attributes of a DTO.
2. **Abstractions over reference identifiers**

    Instead of leaving the programmer with a *department_id* for
    him to figure out that he needs to have a `DepartmentDao`
    to fetch the details of a particular *department*,
    we give him a clear interface, which he can use for that.

3. **Immutability** (and thread-safety)

    Since we have got ridden of DTOs, we are able to make all our objects [immutable].
    The fact that all objects are immutable also makes them thread-safe.


[Data Access Object]: https://en.wikipedia.org/wiki/Data_access_object
[criticism]: https://www.yegor256.com/2017/12/05/data-access-object.html
[CRUD]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[DTO]: /2018/10/08/entity-and-dto.html
[POJO]: https://en.wikipedia.org/wiki/Plain_old_Java_object
[SQL-speaking objects]: https://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html#sql-speaking-objects
[Decorator]: https://en.wikipedia.org/wiki/Decorator_pattern
[introduce temporal coupling]: /2018/10/21/dtos-lead-to-temporal-coupling.html
[immutable]: https://www.yegor256.com/2014/06/09/objects-should-be-immutable.html
