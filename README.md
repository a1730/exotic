# exotic [![](https://api.travis-ci.org/forax/exotic.svg?branch=master)](https://travis-ci.org/forax/exotic)
A bestiary of classes implementing exotic semantics in Java

In Java, a static final field is considered as a constant by the virtual machine,
but a final field of an object which is a constant is not itself considered as a constant.
Exotic classes allows to see a constant's field as a constant, a result of a calculation as a constant,
to change at runtime the value of a constant, etc. 

### MostlyConstant

A constant for the VM that can be changed by de-optimizing all the codes that contain the previous value of the constant.

```java
private static final MostlyConstant<Integer> FOO = new MostlyConstant<>(42, int.class);
private static final IntSupplier FOO_GETTER = FOO.intGetter();

public static int getFoo() {
  return FOO_GETTER.getAsInt();
}
public static void setFoo(int value) {
   FOO.setAndDeoptimize(value);
}
```

### StableField

A field that becomes a constant if the object itsef is constant and the field is initialized

```java
enum Option {
  a, b;
    
  private static final Function<Option, String> UPPERCASE =
      StableField.getter(lookup(), Option.class, "uppercase", String.class);
    
  private String uppercase;  // stable

  public String upperCase() {
    String uppercase = UPPERCASE.apply(this);
    if (uppercase != null) {
      return uppercase;
    }
    return this.uppercase = name().toUpperCase();
  }
}
...
Option.a.upperCase()  // constant "A"
```

### ConstantMemoizer

A function that returns a constant value if its parameter is a constant.

```java
private static final Function&lt;String, Integer&gt; MEMOIZER =
    ConstantMemoizer.memoizer(String::length, String.class, int.class);
...
MEMOIZER.apply("foo") // constant 3
```

### StructuralCall

A method call that can call different method if they have the same name and same parameter types.

```java
private final static StructuralCall IS_EMPTY =
    StructuralCall.create(lookup(), "isEmpty", methodType(boolean.class));

static boolean isEmpty(Object o) {  // can be called with a Map, a Collection or a String
  return IS_EMPTY.invoke(o);
}
```


## Build Tool Integration [![](https://jitpack.io/v/forax/exotic.svg)](https://jitpack.io/#forax/exotic)

Get latest binary distribution via [JitPack](https://jitpack.io/#forax/exotic) 


### Maven

    <repositories>
        <repository>
            <id>jitpack.io</id>
            <url>https://jitpack.io</url>
        </repository>
    </repositories>
    <dependency>
        <groupId>com.github.forax</groupId>
        <artifactId>exotic</artifactId>
        <version>master-SNAPSHOT</version>
    </dependency>


### Gradle

    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
    dependencies {
        compile 'com.github.forax:exotic:master-SNAPSHOT'
    }
