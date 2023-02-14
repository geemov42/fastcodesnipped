Sometimes, we need to add methods that ask specific behaviour. Generally to do that, we used the service layer.

This page expose a solution to extends your jpa repository with custom methods.

First, we need to declare a custom interface :

```java
public interface AccountCustomRepository {

    AccountEntity retrieveAccount(UUID uuid);
}
```

We add the interface on our jpa repository :

```java
@Repository
public interface AccountRepository extends AccountCustomRepository, JpaRepository<AccountEntity, UUID> {
}
```

And finally, we need to implements this interface outside of our jpa repository in a class with a name that match 'interfaceName'Impl

```java
public class AccountCustomRepositoryImpl implements AccountCustomRepository {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public AccountEntity retrieveAccount(UUID uuid) {

        CriteriaBuilder criteriaBuilder = this.entityManager.getCriteriaBuilder();
        CriteriaQuery<AccountEntity> criteriaQuery = criteriaBuilder.createQuery(AccountEntity.class);

        Root<AccountEntity> entityRoot = criteriaQuery.from(AccountEntity.class);
        Predicate predicate = criteriaBuilder.equal(entityRoot.get("uuid"), uuid);
        criteriaQuery.where(predicate);

        TypedQuery<AccountEntity> query = this.entityManager.createQuery(criteriaQuery);

        return query.getSingleResult();
    }
}
```

Once it's done, we can use our jpa repository in our service layer and call the method `retrieveAccount`. Spring data jpa will find this

method to the right class. We don't need to add indication.

```java
accountRepository.retrieveAccount(uuid);
```
