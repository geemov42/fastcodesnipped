The specification exist to replace the criteria syntax for spring data jpa.

Define a specification :

```java
public interface AccountJpaSpecification {

    static Specification<AccountEntity> hasFirstname(String firstname) {
        return (root, query, criteriaBuilder) -> criteriaBuilder.equal(root.get("firstname"), firstname);
    }
}
```

Add the behaviour to the repository :

```java
public interface AccountRepository extends JpaRepository<AccountEntity, UUID>, JpaSpecificationExecutor<AccountEntity> {

}
```

And finally use it :

```java
List<AccountEntity> all = this.accountRepository.findAll(where(hasFirstname(firstname)));
```

Find more info on [https://www.baeldung.com/spring-data-criteria-queries](https://www.baeldung.com/spring-data-criteria-queries)
