Sometimes you want to have a primary key on several fields.

> It is preferable to don't do that for performance issue

To do that, you need to define an embeddable :

```java
@Embeddable
public class EmployeeIdentity {
    @NotNull
    @Size(max = 20)
    private String employeeId;

    @NotNull
    @Size(max = 20)
    private String companyId;

```

After, in the main entity :
```java
@Entity
@Table(name = "employees")
public class Employee {

    @EmbeddedId
    private EmployeeIdentity employeeIdentity;

```

You need to declare your repository with the entity and the composite key:
```java
public interface EmployeeRepository extends JpaRepository<Employee, EmployeeIdentity> {
    List<Employee> findByEmployeeIdentityCompanyId(String companyId);
}
```

And simply use it :
```java
// Insert a new Employee in the database
Employee employee = new Employee(new EmployeeIdentity("E-123", "D-457"),
        "Rajeev Kumar Singh",
        "rajeev@callicoder.com",
        "+91-9999999999");

employeeRepository.save(employee);

// Retrieving an Employee Record with the composite primary key
employeeRepository.findById(new EmployeeIdentity("E-123", "D-457"));

// Retrieving all the employees of a particular company
employeeRepository.findByEmployeeIdentityCompanyId("D-457");
```