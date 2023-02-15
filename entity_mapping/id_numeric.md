Define correctly the primary key is essential for different reasons.
If your definition is bad, you can have performance issues.

Don't use table stategy <- bad performance.

It exists several strategy :
* IDENTITY : The incrementation process is very efficient since it uses a lightweight locking mechanism, as opposed to the more heavyweight transactional course-grain locks. The only drawback is that the newly assigned value can only be known after executing the actual insert statement
* SEQUENCE : The most efficient because we can generate id before and hibernate can take advantage of batching.
* TABLE : The least officient because we need to add a special table and manage it to obtains id.

```java
@Id
@GeneratedValue(generator = "userdetail_id_seq", strategy = GenerationType.SEQUENCE)
@SequenceGenerator(name = "userdetail_id_seq", sequenceName = "userdetail_id_seq")
private int id;
```

For modern version of hibernate (>5.0), there is a new algorithm to generate id in sequence correctly : pooled
For old version, you need to activate :

```xml
<property name="hibernate.id.new_generator_mappings" value="true"/>
```

You can precise the algorithm like that :

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "pooled")
@GenericGenerator(
    name = "pooled",
    strategy = "org.hibernate.id.enhanced.SequenceStyleGenerator",
    parameters = {
        @Parameter(name = "sequence_name", value = "sequence"),
        @Parameter(name = "initial_value", value = "1"),
        @Parameter(name = "increment_size", value = "3"),
        @Parameter(name = "optimizer", value = "pooled")
    }
)
private Long id;
```

And choose between optimizer :

| Optimizer type | Implementation class | Description |
| ------ | ------ | ------ |
|   none   |  NoopOptimizer    |   Every identifier is fetched using a new roundtrip to the database   |
|   hi/lo   |   HiLoOptimizer   |   It allocates identifiers by using the legacy hi/lo algorithm   |
| pooled | PooledOptimizer | It is an enhanced version of the hi/lo algorithm which is interoperable with other systems unaware of this identifier generator |
| pooled-lo | PooledLoOptimizer | It is a variation of the pooled optimizer, the database sequence value representing the lo value instead of the hi one |

