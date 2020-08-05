# Unit Test Tips & FAQ

## `mvn test` doesn't run the tests

If you run `mvn test` but it tells you that:

```text
$ mvn test
...
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 0, Failures: 0, Errors: 0, Skipped: 0
```

There might be a couple of reasons, like incidentally using JUnit 5 but you're using JUnit  4's mechanisms.

In my case, it was caused by excluding `org.junit.vintage:junit-vintage-engine` in `org.springframework.boot:spring-boot-starter-test` dependency.

So I have to make sure not excluding it. For example:

```text
		<!-- testing related dependencies -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
		    <groupId>junit</groupId>
		    <artifactId>junit</artifactId>
		    <scope>test</scope>
		</dependency>
		<dependency>
			<groupId>io.rest-assured</groupId>
			<artifactId>rest-assured</artifactId>
			<scope>test</scope>
		</dependency>
```

