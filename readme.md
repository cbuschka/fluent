Fluent
------
Fluent is an ultra-lightweight Java library to aid fluent coding in Java 8 onwards

[![Build Status](https://travis-ci.org/alexheretic/fluent.svg?branch=master)](https://travis-ci.org/alexheretic/fluent)
[![Maven Central](https://img.shields.io/maven-central/v/com.github.alexheretic/fluent.svg)](http://mvnrepository.com/artifact/com.github.alexheretic/fluent)

### Map Building
Classes to aid Map building in a declarative way. A bit nicer than the anonymous sub-class with initializer block approach, don't you think?

```java
// simple usage, assuming someMap is a Map<String, String> already declared
Map<String, String> example = new Fluent.LinkedHashMap<String, String>()
    .append("key1", "val1")
    .append("key2", "val2")
    .appendAll(someMap);

// nested structures declared all at once
Map users = new Fluent.HashMap<>()
    .append("David", new Fluent.HashMap<>()
        .append("customers", asList(
            new Fluent.HashMap<>()
                .append("name", "Darrel")
                .append("age", 33),
            new Fluent.HashMap<>()
                .append("name", "John")
                .append("age", 29)))
        .append("keywords", asList("word1", "word2")))
    .append("Karen", new Fluent.HashMap<>()
        .append("customers", emptyList())
        .append("keywords", asList("word33")));

// fluent immutable maps
Map immutable = new Fluent.HashMap<>()
    .append("one", 1)
    .append("two", 2)
    .append("three", 3)
    .unmodifiable();
```

### Checked Exception Handling With Functional Wrapping
Utility methods to wrap the propagation of a possible checked exception in an unchecked one. 
Particularly useful in making fluent functional code involving checked exceptions not a horror to behold.

```java
// in short Unchecker#unchecked lets you do this
unchecked(() -> methodThatThrowsACheckedException());

// which is equivalent to
try {
    methodThatThrowsACheckedException();
}
catch (RuntimeException | Error e) {
    throw e;
}
catch (Throwable t) {
    throw new RuntimeException(t);
}
```

Dealing with checked exception mad code in lambdas is messy, take for example these methods
```java
// some Exception throwing instance
interface CheckMe {
    void youMustHandleMyErrorCases() throws Exception;
    List<Integer> atLeastWriteThrowsAllOverYourCode() throws Throwable;
}

// some exception throwing utility class
class ImportantUtility {
    public static String dontIgnoreMeeeee(Integer i)
        throws IOException, GeneralSecurityException, NoSuchMethodException {
        /* some important logic ... */
    }
}
```
This logic could cause the following mess in simple handler code
```java
List<String> uncheckThrowerManual(CheckMe object) {
    try {
        object.youMustHandleMyErrorCases();
        return object.atLeastWriteThrowsAllOverYourCode().stream()
            .map(number -> {
                try {
                    return ImportantUtility.dontIgnoreMeeeee(number);
                }
                catch (IOException | GeneralSecurityException | NoSuchMethodException ex) {
                    throw new RuntimeException(ex);
                }
            })
            .collect(toList());
    }
    catch (RuntimeException | Error e) {
        throw e;
    }
    catch (Throwable throwable) {
        throw new RuntimeException(throwable);
    }
}
```
We can clean this up with Java 8 & Fluent
```java
/*
 * Unchecker#uncheckedGet returns a throwing supplier's result
 *          (wrapping checked in RuntimeExceptions by default)
 * Unchecker#uncheck transforms a throwing lambda/method into a unchecked one
 */
List<String> uncheckThrower(CheckMe object) {
    return uncheckedGet(() -> {
        object.youMustHandleMyErrorCases();
        return object.atLeastWriteThrowsAllOverYourCode().stream()
            .map(uncheck(ImportantUtility::dontIgnoreMeeeee))
            .collect(toList());
    });
}
```

Fluent is licensed under the [Apache 2.0 licence](http://www.apache.org/licenses/LICENSE-2.0.html).

### Releases

2.0 is the current latest release, available at maven central. Requiring JDK 1.8 or later.

```xml
<dependency>
  <groupId>com.github.alexheretic</groupId>
  <artifactId>fluent</artifactId>
  <version>2.0</version>
</dependency>
```

### Changelog
Release 2.x
* Add Fluent.Map#unmodifiable()

Release 1.x
* Fluent.Map classes
* Unchecker checked exception utility