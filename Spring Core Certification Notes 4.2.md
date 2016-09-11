#Dependency Injection In Spring
###DI Basics
- Reduces coupling between classes
- Classes do not obtain their dependencies (collaborators), Spring container provides proper dependency based on configuration provided
- Enables easy switching of dependencies based en current needs - local development automaticallyenvironment, 
easy mocking for unit tests, in memory database vs production one, etc.
- Promotes programming to interfaces
- Spring managed objects are called "beans"
- Beans are written as POJOs, no need to inherit Spring base classes or implement spring interfaces
- Classes can be completely independent on Spring framework (although for convenience Spring annotations are often used)
- Lifecycle of beans is managed centrally by Spring container (eg. enforces singleton state of bean/ one isntance pers session. one instance per http request/...)
- Spring configuration (incl. DI) is possible either via xml or Java configuration files (newer, more popular approach)

###Spring Application Context
- Spring beans are managed by Application Context (container)
- Application context is initialized with one or more configuration files (java or xml), which contains setting for framework, DI, persistence, transactions,...
- Application context can be also instantiated in unit tests
- Ensures beans are always created in right order based on their dependencies
- Ensures all beans are fully initialized before the first use
- Each bean has its unique identificator, by which it can be later referenced (should not contain specific implementation details, based on circumstances different implementations can be provided)
- No need for full fledged java EE application server



***Creating app context***
```java
    //From single configuration class
    ApplicationContext context = SpringApplication.run(ApplicationConfiguration.class);
    
    //With string args
    ApplicationContext context = SpringApplication.run(ApplicationConfiguration.class, args);
    
    //With additional configuration
       SpringApplication app = new SpringApplication(ApplicationConfiguration.class);
       // ... customize app settings here
       context = app.run(args);
```

***Obtaining beans form Application Context***
```java
    //Obtain bean by type
    context.getBean(MyBean.class);
```


###Bean Scopes
- Spring manages lifecycle of beans, each bean has its scope
- Default scope is singleton - one instance per application context
- If none of the Spring scopes is appropriate, custom scopes can be defined
- Scope can be defined by @Scope (eg. @Scope(BeanDefinition.SCOPE_SINGLETON)) annotation on the class-level of bean class

***Available Scopes***
- Singleton - One instance per application context, default if bean scope is not defined
- Prototype - New instance is created every time bean is requested
- Session - One instance per user session - Web Environment only
- Global-session - One global session shared among all portlets - Only in Web Portlet Environment
- Request - One instance per each HTTP Request - Web Environment only
- Custom - Can define multiple custom scopes with different names and lifecycle
- Additional scopes available for Spring Web Flow applications (not needed for the certification)


# Spring Configuration
- can be xml or java based

###Composite configuration
- preferred way is to divide configuration to multiple files and then import them when used
- Usually based on application layers - web layer, service layer, DAO layer,...
- Enables better configuration organization
- Enables to separate layers, which often change based on environment, etc. - eg. switching services for mocks in unit tests, having in-memory database, ...
- Can import other config files in java config using @Import annotation
```java
@Configuration
@Import({WebConfiguration.class, InfrastructureConfiguration})
public class ApplicationConfiguration {
    //Config here
}

```
- Beans defined in another @Configuration file can be injected using @Autowired annotation on field or setter level, cannot use constructors here
- Another way of referencing beans form other @configuration is to declare them as method parameters of defined beans in current config file
```java
@Configuration
@Import({WebConfiguration.class, InfrastructureConfiguration})
public class ApplicationConfiguration {
    
    @Bean //Object returned by this method will be spring managed bean
    public MyBean myBean(BeanFromOtherConfig beanFromOtherConfig) {
        //beanFromOtherConfig bean is automatically injected by Spring
        ...
        return myBean;
    }
```
- In case multiple same beans are found among configuration, spring injects the one, which is discovered as the last