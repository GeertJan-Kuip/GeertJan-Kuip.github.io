## Spring course topics

As I want to get their certificate, I started on their [learning materials](https://spring.academy/courses/spring-framework-essentials/lessons/spring-essentials-overview-lab). There are 9 modules, each containing videos and some hands-on code work and review. To provide an overview for myself, I use this blog post for notes.


### Module 1 - Spring Essentials Overview

#### What is the Spring Framework (video)

What is Spring Framework 
- Open Source
- Lightweight
- DI Container (IoC Container)
- Framework

LightWeigth
- Doesn't require a Java EE application server
- Not invasive, does not require to extend framework classes or implement framework interfaces for most usage.
- You write your code as POJOs.
- Low overhead. Spring JARs are relatively small.

DI Container
- Your objects do not have to worry about finding/connecting to each other.
- Spring instantiates and injects dependencies in your objects.
- Spring also serves as a lifecycle manager.

#### The Dependencey Injection (DI) Container (video)

Goal of the Spring Framework
- Provide comprehensive infrastructural support for developing enterprise Java applications
- Don't repeat yourself (DRY), Separation of concerns, Convention over configuration, Testability

Rely on interfaces to conceal the complexity of the implementation and to allow for swapping out implementation. Will also make testing easier.

#### Spring Framework History (video)

Rod Johnson. First release 2003.

Success factors:
- Provide choice at every level.
- Embrace change and different perspectives.
- Strong backwards compatibility.
- Careful API design.
- High standard for code quality.
- OSS comunity.
- Developer support on forums, Stack Overflow.
- Support of conferences and user groups.

Adaptable to change:
- Initial integration with other open source projects. Hibernate, Quartz, Multiple View Technologies.
- Spring projects created for common enterprise domains. Spring Security, Batch, Integration.
- Spring projects created for new domains. Spring Data: NoSQL + JPA, Spring Cloud.
- Spring Boot created to further simplify DevEx.
- Spring Framework support for Kotlin.

Spring adapts to
- JDK Versions
- Native compilation
- Reactive programming
- Stream processing
- Kotlin support
- Kubernetes

#### Spring Overview Lab



#### Demo - Spring Overview Lab