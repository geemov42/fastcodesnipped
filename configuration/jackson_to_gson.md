It is possible to replace the default json converter (jackson) by the google implementation `gson`.
For this purpose, consult this article :[https://www.callicoder.com/configuring-spring-boot-to-use-gson-instead-of-jackson/](https://www.callicoder.com/configuring-spring-boot-to-use-gson-instead-of-jackson)

But globally :

```properties
# Preferred JSON mapper to use for HTTP message conversion.
spring.http.converters.preferred-json-mapper=gson

# It exist more properties to customize the behaviour, consult the article
```

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<!-- Exclude the default Jackson dependency -->
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-json</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```
