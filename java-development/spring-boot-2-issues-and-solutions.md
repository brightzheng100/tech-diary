# Spring Boot 2 Issues & Solutions

## JPA

### Issue: Field xxx in xxx required a bean named 'entityManagerFactory' that could not be found.

#### Symptoms:

When using Spring Boot2 with JPA, I encountered a weird issue, where if I'm using H2 as the embedded DB, everything worked fine, but once I switched to external MySQL under "prod" profile, it kept failing:

```text
$ SPRING_PROFILES_ACTIVE=prod \
    mvn -e clean spring-boot:run -Dmaven.test.skip=true

...
***************************
APPLICATION FAILED TO START
***************************

Description:

Field repository in app.controller.StudentController required a bean named 'entityManagerFactory' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)


Action:

Consider defining a bean named 'entityManagerFactory' in your configuration.
```

#### Analysis & Solutions:

This was because I excluded `HikariCP` as I found it's really unfamiliar to me and didn't know what it was.

```text
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
			<exclusions>
				<exclusion>
					<groupId>com.zaxxer</groupId>
					<artifactId>HikariCP</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```

Unfortunately, this was exactly what caused the problem.

Check out the Spring Boot doc, [here](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-sql), and it clearly states that:

> Production database connections can also be auto-configured by using a pooling DataSource. Spring Boot uses the following algorithm for choosing a specific implementation:  
> 1. We prefer `HikariCP` for its performance and concurrency. If `HikariCP` is available, we always choose it.  
> 2. Otherwise, if the `Tomcat pooling DataSource` is available, we use it.  
> 3. If neither `HikariCP` nor the Tomcat pooling datasource are available and if `Commons DBCP2` is available, we use it.  
> If you use the `spring-boot-starter-jdbc` or `spring-boot-starter-data-jpa` “starters”, you automatically get a dependency to `HikariCP`.

So we either use the default by simply adding `org.springframework.boot:spring-boot-starter-data-jpa` dependency without excluding `HikariCP`, or explicitly adding your preferred pooling library.

