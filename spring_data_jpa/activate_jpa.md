We can simply activate jpa with this configuration :

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "...")
@EntityScan(basePackages = "...")
public class JpaConfig {
}
```