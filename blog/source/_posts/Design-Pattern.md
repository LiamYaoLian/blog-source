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

#### Interface vs Abstract Class
##### Abstract Class
* is-a
* 抽象类不允许被实例化，只能被继承
* 抽象类可以包含属性和方法。方法既可以包含代码实现（比如 Logger 中的 log() 方法），也可以不包含代码实现（比如 Logger 中的 doLog() 方法）。不包含代码实现的方法叫作抽象方法。
* 子类继承抽象类，必须实现抽象类中的所有抽象方法。
##### interface
* has a, contract
* 接口不能包含属性（也就是成员变量）。
* 接口只能声明方法，方法不能包含代码实现。
* 类实现接口的时候，必须实现接口中声明的所有方法。

#### Program to interface
Program to an interface, not an implementation


#### SOLID principles
* Single Responsibility Principle: A class or module should have a single responsibility. Whether the module is too big depends on the situation. At the beginning, can have a big module. Later when the module becomes too big, we can break it down. Usually should be less than 200 lines and 10 members.
* Open Closed Principle: software entities (modules, classes, functions, etc.) should be open for extension , but closed for modification. 没有破坏原有代码的正常运行和单元测试。时刻具备扩展意识、抽象意识、封装意识。在识别出代码可变部分和不可变部分之后，我们要将可变部分封装起来，隔离变化，提供抽象化的不可变接口，给上层系统使用。当具体的实现发生变化的时候，我们只需要基于相同的抽象接口，扩展一个新的实现，替换掉老的实现即可，上游系统的代码几乎不需要修改。多态、依赖注入、基于接口而非实现编程。

最合理的做法是，对于一些比较确定的、短期内可能就会扩展，或者需求改动对代码结构影响比较大的情况，或者实现成本不高的扩展点，在编写代码的时候之后，我们就可以事先做些扩展性设计。但对于一些不确定未来是否要支持的需求，或者实现起来比较复杂的扩展点，我们可以等到有需求驱动的时候，再通过重构代码的方式来支持扩展的需求。

在扩展性和可读性之间做权衡

* Liskov Substitution Principle: Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it
子类在设计的时候，要遵守父类的行为约定（或者叫协议）。父类定义了函数的行为约定，那子类可以改变函数的内部实现逻辑，但不能改变函数原有的行为约定。这里的行为约定包括：函数声明要实现的功能；对输入、输出、异常的约定；甚至包括注释中所罗列的任何特殊说明。实际上，定义中父类和子类之间的关系，也可以替换成接口和实现类之间的关系。
* Interface Segregation Principle: Clients should not be forced to depend upon interfaces that they do not use

* Dependency Inversion Principle: High-level modules shouldn’t depend on low-level modules. Both modules should depend on abstractions. In addition, abstractions shouldn’t depend on details. Details depend on abstractions.调用者属于高层，被调用者属于低层。

“控制”指的是对程序执行流程的控制，而“反转”指的是在没有使用框架之前，程序员自己控制整个程序的执行。在使用框架之后，整个程序的执行流程通过框架来控制。流程的控制权从程序员“反转”给了框架。

依赖注入和控制反转恰恰相反，它是一种具体的编码技巧。我们不通过 new 的方式在类内部创建依赖类的对象，而是将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（或注入）给类来使用。


## Design Pattern
### Singleton
* create and provide only one instance by itself
* implementation
    - private constructor
    - private static object
    - public static method to create and get
* type
    - hungry: Instantiate when loading the class; thread-safe
    ```
    //饿汉式：创建对象实例的时候直接初始化  空间换时间
public class SingletonOne {
	//1、创建类中私有构造
	private SingletonOne(){

	}

	//2、创建该类型的私有静态实例
	private static SingletonOne instance=new SingletonOne();

	//3、创建公有静态方法返回静态实例对象
	public static SingletonOne getInstance(){
		return instance;
	}
}
    ```
    - lazy: instantiate when using; not thread-safe
    ```
    //懒汉式：类内实例对象创建时并不直接初始化，直到第一次调用get方法时，才完成初始化操作
//时间换空间
public class SingletonTwo {
	//1、创建私有构造方法
	private SingletonTwo(){

	}

	//2、创建静态的该类实例对象
	private static SingletonTwo instance=null;

	//3、创建开放的静态方法提供实例对象
	public static SingletonTwo getInstance(){
		if(instance==null)
			instance=new SingletonTwo();

		return instance;
	}
}
    ```
        - 同步锁
        - 双重校验锁
        - 静态内部类
        - 枚举
* pro
  - save memory
  - avoid frequent creating
* con
  - hard to extend
  - if not used for a long time, lost due to GC
* when to use
  - save memory
  - universal handling
  - multiple instances will cause problems, e.g. order number generator


## Coding Standards
* reference: 《重构》《代码大全》《代码简洁之道》

## Refactoring
* why, what, when, how
* unit test, testability
