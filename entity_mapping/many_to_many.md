> Cascading
> 
> For `@ManyToMany` associations, `CascadeType.REMOVE` does not make too much sense when both sides represent independent entities. 
> In this case, removing a Post entity should not trigger a Tag removal because the Tag can be referenced by other posts as well. 
> The same arguments apply to orphan removal since removing an entry from the tags collection should only delete the
> junction record and not the target Tag entity.
> 
> For both unidirectional and bidirectional associations, it is better to avoid the CascadeType.REMOVE mapping. 
> Instead of CascadeType.ALL, the cascade attributes should be declared explicitly (`e.g. CascadeType.PERSIST, CascadeType.MERGE`)

### Unidirectional

```java
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
@JoinTable(name = "post_tag",
    joinColumns = @JoinColumn(name = "post_id"),
    inverseJoinColumns = @JoinColumn(name = "tag_id")
)
private Set<Tag> tags = new HashSet<>();
```
> For a `@ManyToMany` association, it’s better to use the Set collection type, instead of List

### Bidirectional

> While in the one-to-many and many-to-one associations the child-side is the one holding
> the foreign key, for a many-to-many table relationship both ends are parent-sides and the junction table plays the child-side role.
> Because the junction table is hidden when using the default @ManyToMany mapping, the application developer must choose an owning and a mappedBy side.

owning side :
```java
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
@JoinTable(name = "post_tag",
    joinColumns = @JoinColumn(name = "post_id"),
    inverseJoinColumns = @JoinColumn(name = "tag_id")
)
private Set<Tag> tags = new HashSet<>();

// For a bidirectional @ManyToMany association, the helper methods must be added to the entity that is more likely to interact with
public void addTag(Tag tag) {
    tags.add(tag);
    tag.getPosts().add(this);
}
public void removeTag(Tag tag) {
    tags.remove(tag);
    tag.getPosts().remove(this);
}
```

mapped side :
```java
@ManyToMany(mappedBy = "tags")
private Set<Post> posts = new HashSet<>();
```