---
title: Design Pattern
date: 2022-04-07 17:51:29
tags: "Design Pattern"
---
## Code Quality
* maintainability: quickly update without bugs
  - well layered
  - modularity
  - high cohesion loose coupling
  - well-documented
  - interface-oriented programming
* readability
* extensibility
* simplicity
* reusability
  - inheritance, polymorphism
  - SRP
  - high cohesion loose coupling
  - modularity
* testability

## Paradigm
* Process-oriented programming: a process is a unit, featuring separation of data structures and methods.
* Object-oriented programming: based on class and object. Data structures and methods are in a class.   
* Functional programming



### OOP
#### Four features

* encapsulation: control how to access (get and set) and hide details
 - to increase maintainability

* abstraction: hide details of implementation and only expose what to do
   - may use `interface`, `abstract`, or function
   - when naming a function, avoid implementation details. E.g. `getPictureUrl()` is better than `getAWSPictureUrl()` is bad, in case later we change AWS to something else
   - to increase maintainability and extensibility

* inheritance: is-a, e.g. dog is an animal
  - Java: a class can only extend a superclass rater than multiple superclasses
  - to increase reusability
  - if there are too many inheritance layers, maintainability and readability suffer

* polymorphism: a subclass can replace a superclass
  - override
  - to increase extensibility and reusability

#### Compared to Process-oriented Programming
pros:
* complex application, not linear but like a network
* modularity
* four features
* think like a human

#### Fake OOP
* add unnecessary getter and setter
* the getter returns an object, which may be modified unintendedly. Can use `Collections.unmodifiableList()`

```java
public class ShoppingCart {
  // ...other codes...
  public List<ShoppingCartItem> getItems() {
    return Collections.unmodifiableList(this.items);
  }
}

public class UnmodifiableList<E> extends UnmodifiableCollection<E> implements List<E> {
  public boolean add(E e) {
    throw new UnsupportedOperationException();
  }
  public void clear() {
    throw new UnsupportedOperationException();
  }
  // ...other codes...
}

ShoppingCart cart = new ShoppingCart();
List<ShoppingCartItem> items = cart.getItems();
items.clear();//will throw UnsupportedOperationException
```

* global variable and method
  - singleton
  - static attribute
  - static method: used to manipulate static attribute or data outside
  - constant: can put config attributes in a class `Constants`

##### POP example: Constants

```Jave
public class Constants {
  public static final String MYSQL_ADDR_KEY = "mysql_addr";
  public static final String MYSQL_DB_NAME_KEY = "db_name";
  public static final String MYSQL_USERNAME_KEY = "mysql_username";
  public static final String MYSQL_PASSWORD_KEY = "mysql_password";

  public static final String REDIS_DEFAULT_ADDR = "192.168.7.2:7234";
  public static final int REDIS_DEFAULT_MAX_TOTAL = 50;
  public static final int REDIS_DEFAULT_MAX_IDLE = 50;
  public static final int REDIS_DEFAULT_MIN_IDLE = 20;
  public static final String REDIS_DEFAULT_KEY_PREFIX = "rt:";

}
```

Using a single Constants may be bad because:
 - when there are too many lines, hard to find one and may conflict with others
 - long compilation time. Each time `Constants` is changed, other classes depend on that will be compiled again. When the project is large and we need to do unit test, it will take long.
 - reusability. If we need to use only some variables in  `Constants` in another project, we have to import the whole file, which is a waste.

 How to improve?
 - method 1: Break down  `Constants` into `MysqlConstants`, `RedisConstants`, etc
 - method 2: put constant in the class that uses it. E.g.  `RedisConfig` class

##### POP example: Utils
Class A and class B use the same methods. To avoid repetition, we put it in `XxxUtils`. Use static methods.

##### POP example: VO/BO/Entity and Controller/Service/Repository
* Anemic Demain Model: separation of data and logic
  - Entity and Repository
  - BO and Service
  - VO and Controller
* Rich Demain Model: data and logic are in the same class. Service layer includes service classes and domain classes (similar to BO). 贫血模式重Service轻BO，充血模式轻Service重Domain。适合业务复杂，improve reusability. 


#### SOLID principles

## Design Pattern

## Coding Standards
* reference: 《重构》《代码大全》《代码简洁之道》

## Refactoring
* why, what, when, how
* unit test, testability
