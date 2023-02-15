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

When we use the forEach directly on the stream, the stream will be automatically closed at the end.
Otherwise, we can have this kind of notation:

```java
var tasks = new ArrayList<Task>();

try (
        Stream<SyncTask> syncTaskStream = this.repoSync.getAll();
        Stream<AsyncTask> asyncTaskStream = this.repoAsync.getAll();
        Stream<Task> finalStream = Stream.concat(syncTaskStream, asyncTaskStream)
) {

    finalStream.forEach(task -> {
        // Detach entity to don't store it in memory and obtains an outOfMemory Exception
        this.entityManager.detach(task);

        tasks.add(task);
    });
}

return tasks;
```