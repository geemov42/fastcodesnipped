### Unidirectional

> Because the `@ManyToOne` association controls the foreign key directly, the automatically
> generated DML statements are very efficient.
> Actually, the best-performing JPA associations always rely on the child-side to translate the JPA state to the foreign key column value.
> This is one of the most important rules in JPA relationship mapping, and it will be
> further emphasized for @OneToMany, @OneToOne and even @ManyToMany associations

how to do the mapping :
```java
@ManyToOne
@JoinColumn(name = "post_id")
private Post post;
```

### Bidirectional

> In a bidirectional association, only one side can control the underlying table relationship.
> For the bidirectional `@OneToMany` mapping, it is the child-side `@ManyToOne` association in charge of keeping the foreign key column value in sync with the in-memory Persistence Context.
> This is the reason why the bidirectional `@OneToMany relationship must define the mappedBy` attribute, indicating that it `only mirrors the @ManyToOne` child-side mapping.

Parent side mapping:
```java
@OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
private List<PostComment> comments = new ArrayList<>();

// To synchronize both ends, it is practical to provide parent-side helper methods that add/remove child entities.
public void addComment(PostComment comment) {
    comments.add(comment);
    comment.setPost(this);
}

public void removeComment(PostComment comment) {
    comments.remove(comment);
    comment.setPost(null);
}
```

One of the major advantages of using a bidirectional association is that entity state transitions
can be cascaded from the parent entity to its children. In the following example, when
persisting the parent Post entity, all the PostComment child entities are persisted as well.


Child side mapping :
```java
@ManyToOne
@JoinColumn(name = "post_id")
private Post post;
```