### Projection
Using projection is interesting because it improve performance comparing to manage entity.
When you read an entity, hibernate need to manage this entity, that costs ressources.

Instead, we can used projection to read data in a database because they are not managed.

It exist different way to do projection :
* Scalar value projections using Object[]
* Scalar value projections using Tuple
* Class projection
* Interface projection

Entity definition :
```java
@Entity
public class Comment {
    @Id
    private Integer id;
    private Integer year;
    private boolean approved;
    private String content;
}
```

### Scalar way using Object[]

```java
public interface CommentRepository extends JpaRepository<Comment, Integer> {

    @Query("SELECT c.year, COUNT(c.year) FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
    List<Object[]> countTotalCommentsByYear();
}
```

### Scalar way using Tuple

```java
public interface CommentRepository extends JpaRepository<Comment, Integer> {

    @Query("SELECT c.year, COUNT(c.year) FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
    List<Tuple> countTotalCommentsByYear();
}
```

### Class way

class projection :
```java
public class CommentCount {
    private Integer year;
    private Long total;

    public CommentCount(Integer year, Long total) {
        this.year = year;
        this.total = total;
    }
}
```

Usage :
```java
public interface CommentRepository extends JpaRepository<Comment, Integer> {

    @Query("SELECT new mypackage.CommentCount(c.year, COUNT(c.year)) FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
    List<CommentCount> countTotalCommentsByYearClass();
}
```

### Interface way

interface projection :
```java
public interface ICommentCount {

    Integer getYearComment();

    Long getTotalComment();
}
```

Usage :
```java
public interface CommentRepository extends JpaRepository<Comment, Integer> {

    @Query("SELECT c.year AS yearComment, COUNT(c.year) AS totalComment FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
    List<ICommentCount> countTotalCommentsByYearInterface();

    @Query(value = "SELECT c.\"year\" AS yearComment, COUNT(c.*) AS totalComment FROM \"comment\" AS c GROUP BY c.\"year\" ORDER BY c.\"year\" DESC", nativeQuery = true)
    List<ICommentCount> countTotalCommentsByYearNative();

}
```