---
title: Spring
date: 2022-03-25 18:24:02
tags: ['Spring']
categories: Spring
---
## What
Spring Framework is a Java platform that provides comprehensive <u>infrastructure</u> support for developing Java applications.
![Spring Framework](../image/spring/spring_framework.png)
Portlet has been removed
* Spring Core: IoC / DI
* Spring AOP: Aspect-oriented-programming implementation
* Spring Aspects: supports integration with AspectJ
* Spring JDBC: connects to databases
* Spring JMS: Java message service
* Spring ORM: supports ORM tools such as Hibernate
* Spring Web: supports creating web application
* Spring Test: JUnit, TestNG

## @RestController vs @Controller
 * @Controller without @ResponseBoby returns a view, used when you don't want to separate front end and back end
 * @RestController returns JSON or XML
 * @Controller + @ResponseBody = @RestController

# Configure Application Context
* XML
* Java config


## Spring IoC


## Spring AOP

## Spring AOP vs AspectJ AOP

## Bean

## Configure Bean
* XML: better for maintenance at a large company
* Annotation: faster to write, more readable, suitable for light app
* Java config: avoid XML, avoid making annotations everywhere, check dependencies during compilation to prevent errors, agile


### Bean Scope

### Bean Lifecycle

### @Component vs @Bean

### Spring Framework Stereotype Annotations
* @Component: will be created and managed by IoC container
* @Controller: request and response
* @service: business logic
* @Repository: persistency

### Injection Annotations
* @Autowired: by class
* @Inject: by class, not support "required"
* @Named: by name
* @Resource: by name first, then by class

### Meta Annotations
* @Primary: inject this object among objects of the same type
* @PostConstruct: on a method, "init-method"
* @PreDestroy: on a method, "destroy-method"
* @Scope
* @Value: inject value

Reference: https://springframework.guru/spring-framework-annotations/

 ... To be continued
