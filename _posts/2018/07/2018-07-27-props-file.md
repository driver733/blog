---
date: 2018-07-27
title: From procedures to objects - A practical example using JDK Properties
figure: /assets/images/posts/2018/07/props-file/old-ibm-computer.jpg
figcaption: © National Museum of Computing
figalt: Old IBM computer with two scientist reading its printed output.
description: |
  What are the drawbacks of JDK Properties?
  What consequences do they have?
  How object-oriented design helps to resolve them?
keywords:
- properties
- jdk
- eo

categories: oop
comments: true
---

John is a newcomer to Java and has faced the need to save key-value pairs
to a file. He reaches stackoverflow for help and finds the JDK [Properties] class...

<!--more-->

John has not seen it before, so as he opens it, he expects to see code like this.

```java
public interface Props {
    Props with(String key, String value);
    String property(String key);
}
```

To John's surprise, when he finds out the Oracle's solution
to his problem, he sees this:

```java
public void load(InputStream out, String comments)
public void store(OutputStream out, String comments)
```

Nevertheless, he accepts the solution and moves on with his project.



After a couple of months, John's code starts to look like this:

```java
Properties props = new Properties();
...
props.setProperty("k1","v1");
props.load(file2);
...
props.store(file1);
...
props.setProperty("k2","v2");
...
// In another file (debugging)
Properties props = new Properties(file1);
if (!"v1".equals(props.getProperty("k1")) {
    // expection thrown
}
```

He notices that he spends more time on debugging than on anything else.
John decides that he cannot continue working on the project, before he resolves
the problems introduced by the usage of the Properties class:

1. [Temporal coupling]
- Whenever John changed the order of the lines of code in the methods
    which dealt with properties, something would always brake. He would become
    more and more cautious about changing something, especially legacy code.

2. Lack of [self-sufficiency]
- John was not able to integrate the properties objects anywhere,
      such as in constructors,
      because these objects were not self-sufficient.
      "Do they know where to save themselves? How to save?
      How can I be sure that the contents of the file
      to which the properties are saved is up-to-date with object
      content?" These are the questions that John could not answer himself.
      He decided to use [controllers] for that,
      however, at some point John lost control of them,
      failing to remember all states they can be in.

At last, John creates an abstraction for the Properties class:

(using [yegor256/cactoos] library)

```java
public final class PropsFile implements Props {
    public PropsFile(
        final File file
    ) {
        this(
            new Constant<>(
                () -> file
            )
        );
    }
    private PropsFile(
        final Scalar<Scalar<File>> scalar
    ) {
        this.origin = new UncheckedScalar<>(
            new StickyScalar<>(
                () -> {
                    final File file = scalar.value().value();
                    final Properties props = new Properties();
                    if (file.exists()) {
                        try (FileInputStream fis = new FileInputStream(file)) {
                            props.load(fis);
                        }
                    } else {
                        try (FileOutputStream fos = new FileOutputStream(file)) {
                            props.store(fos, "");
                        }
                    }
                    return props;
                }
            )
        );
        this.output = new UncheckedScalar<>(
            new StickyScalar<>(
                () -> new FileOutputStream(
                    scalar.value().value()
                )
            )
        );
    }
    public String property(final String key) {
        return this.origin.value().getProperty(key);
    }
    public PropsFile with(
        final String key,
        final String value
    ) throws IOException {
        final Properties props = this.origin.value();
        props.setProperty(key,value);
        props.store(this.output.value(),"");
        return this;
    }
}
```

Now, he is able to use the same Properties in a much [cleaner] way:

```java
final PropsFile props = new PropsFile(file);
props.with("k1", "v1"); // property added to file.
new Something(props);
```

The object-oriented abstraction [PropsFile],
thus, gives the following benefits over the standard Properties class:

1. No temporal coupling
- Because properties are auto-saved,
     there is no need to manually update the contents of the file.
     Also, the properties are auto-loaded meaning that the manual
     call of the `load()` is also not needed.

2. Destination file initial state abstraction
- John does not care whether the destination file exists initially or not.
     He wants to save the properties. And the interface does that for him,
     creating an empty file if it missing.

3. Self-sufficiency over third-party control
- After the PropsFile object is created, it can be used by anyone,
     without any unintended consequences.
     As a result, there is no need for any controllers who would [manage]
     the properties as now each properties
     object is self-sufficient and self-manageable.

[cleaner]:              https://www.yegor256.com/2014/11/20/seven-virtues-of-good-object.html
[self-sufficiency]:     https://www.yegor256.com/2017/05/10/inversion-of-control.html
[PropsFile]:            https://github.com/driver733/VK-Uploader/blob/master/src/main/java/com/driver733/vkuploader/wallpost/PropsFile.java
[yegor256/cactoos]:     https://github.com/yegor256/cactoos
[controllers]:          https://www.yegor256.com/2016/12/13/mvc-vs-oop.html
[Properties]:           https://docs.oracle.com/javase/8/docs/api/java/util/Properties.html
[Temporal coupling]:    https://www.yegor256.com/2015/12/08/temporal-coupling-between-method-calls.html
[manage]:               /2018/10/08/entity-and-dto.html