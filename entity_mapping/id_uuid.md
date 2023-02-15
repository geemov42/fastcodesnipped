Define correctly the primary key is essential for different reasons.
If your definition is bad, you can have performance issues.

Hibernate 6 use the new uuid generator named `uuid2` that follow the spec RFC4122 variant 2.

You can define your primary key as :
```java
@Id
@Column(columnDefinition = "BINARY(16)")
@UuidGenerator(style = UuidGenerator.Style.TIME)
private UUID uuid;
```

Or :
```java
@Id @Column(columnDefinition = "BINARY(16)")
@GeneratedValue(generator = "uuid2")
@GenericGenerator(name = "uuid2", strategy = "uuid2")
private UUID id;
```