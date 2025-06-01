# Record in Java 16

Record classes are special types of classes, [introduced in Java 16 back in 2021](https://openjdk.org/jeps/395). Record classes eliminate the need of creating classes for immutable data and having to add boilerplate code for `equals()`, `hashCode()`, `toString()`.

#### Definition

**Before record**

```java

class Name {
    private String firstName;
    private String lastName;

    public Name(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
        return this.firstName;
    }

    public String getLastName() {
        return this.lastName();
    }

    @Override
    public boolean equals(Object o) {
        ...
    }

    @Override
    public int hashCode() {
        ...
    }
}
```

**With record**

```java
record Name(String firstName, String lastName) {}

Name name = new Name("Arya", "Stark");
```

So simple and concise!

#### Accessors

```java

Name name = new Name("Arya", "Stark");
System.out.println("A girl is " + name.firstName() + " " + name.lastName() 
    + " of Winterfell and I'm going home.");
```

#### Equality

```java
Name name = new Name("Arya", "Stark");
Name facelessMan = new Name("Arya", "Stark");
name.equals(facelessMan) // true
name.toString().equals("Name[firstName=Arya, lastName=Stark]") // true
name.equals(new Name("Sansa", "Stark")) // false
```

#### Mutability

No way to modify members externally. But can be modified in the compact constructor, i.e., only _while_ constructing the record, not _after_ it is constructed.

```java
record Name(String firstName, String lastName) {
    Name {
        firstName = firstName.toUpperCase();
        lastName = lastName.toUpperCase();
    }
}
```

#### Multiple constructors

Compiler provides the canonical constructor, i.e., the constructor that corresponds to the list of members in the record definition. One can add additional constructors, but they should all eventually reach the canonical constructor.

```java
record Name(String firstName, String lastName, String home) {
    Name(String firstName, String lastName) {
        this(firstName, lastName, "Winterfell");
    }
}
```

#### Adding methods to record

```java
record Name(String firstName, String lastName) {
    String sayIt() {
        return "A girl is " + firstName() + " " + lastName() 
            + " of Winterfell and I'm going home.";
    }
}
```
