The fast way to unit test a service is to use `MockitoExtension`.

```java
@ExtendWith(MockitoExtension.class)
class MyServiceUnderTest {

    @InjectMocks
    private MyService sut;

    @Mock
    private MyDependency dependency;
}
```

Sometimes, to keep in an consistent world and don't need to update test every time the configuration evolve, you want 
to load the configuration from the properties/yml files.

It is not so simpler without using a slice test annotation that load too much resources we don't want.

The config class :
```java
@Configuration
@ConfigurationProperties(prefix = "test")
@Data
public class MyConfig {

    private String firstname;
    private String lastname;
}
```

The code to do the job :
```java
@ExtendWith(MockitoExtension.class)
class MyServiceUnderTest {

    @InjectMocks
    private MyService sut;

    @Mock
    private MyDependency dependency;

    private static MyConfig config;

    @BeforeAll
    static void load() throws IOException {

        ConfigurableEnvironment environment = new StandardEnvironment();
        List<PropertySource<?>> yamlProperties = new YamlPropertySourceLoader().load(
                "applicationConfig", 
                new ClassPathResource("application.yml")
        );
        yamlProperties.forEach(environment.getPropertySources()::addLast);

        Binder binder = Binder.get(environment);
        config = binder.bind("test", Bindable.of(MyConfig.class)).get();
    }
}
```

With that config, we don't need to keep uptodate the configurations in our test and we stay consistent with the application.