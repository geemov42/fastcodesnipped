You can take advantage of stream with spring data jpa.

You need to define that your method return stream :
```java
@QueryHints(value = {
        @QueryHint(name = HINT_FETCH_SIZE, value = "100"),
        @QueryHint(name = HINT_CACHEABLE, value = "false"),
        @QueryHint(name = READ_ONLY, value = "true")
})
@Query("select s from SyncTask s")
Stream<SyncTask> getAll();
```

Query hints are not prequerries.

Once your method is created, you can use it :
```java
var tasks = new ArrayList<Task>();

this.repoSync.getAll().forEach(task -> {
    // Detach entity to don't store it in memory and obtains an outOfMemory Exception
    this.entityManager.detach(task);

    tasks.add(task);
};
```

>You need to don't forgot to detach the entity for prevent OutOfMemory.