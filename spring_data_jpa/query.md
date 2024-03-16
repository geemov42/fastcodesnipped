### Use a named query

You can define your named query inside the entity : 

```java
@Table(name = "employees")
@NamedQuery(name = "EmployeeEntity.findAllWithJob", query = "from EmployeeEntity e left join fetch e.jobsByJobId ")
public class EmployeeEntity {
}
```

And use it without the need to name it inside the jpa repository :
```java
public interface EmployeeRepository extends CrudRepository<EmployeeEntity, Integer> {

    @Query
    List<EmployeeEntity> findAllWithJob();
}
```