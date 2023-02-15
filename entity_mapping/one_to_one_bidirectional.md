Prefer unidirectional because more predictible.

Parent looks like :

```java
@Entity
@Table(name = "post")
public class Post {
 
    @Id
    @GeneratedValue
    private Long id;
 
    @OneToOne(mappedBy = "post", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private PostDetails details;

    //Getters and setters omitted for brevity
}
```

And the child :
```java
@Entity
@Table(name = "post_details")
public class PostDetails {
 
    @Id
    private Long id;
 
    @OneToOne(fetch = FetchType.LAZY)
    @MapsId
    @JoinColumn(name = "id") // Help to define on which field we want to map our foreign key (depend of the name of the postdetail id. Othewise will be post_id)
    private Post post;
 
    ...
}
```