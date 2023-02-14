Sometimes we need to use heritage for simplify code.
In this example, we want inherit from class `Task` and inherit from it repository.

We have these classes :
```java
@MappedSuperclass
@Data
@AllArgsConstructor
@NoArgsConstructor
@SuperBuilder
public abstract class Task {

    @Id
    @GeneratedValue
    protected long id;

    protected String name;

}

@Entity
@Table(name = "async_task")
@Data
@AllArgsConstructor
@SuperBuilder
@EqualsAndHashCode(callSuper = false)
public class AsyncTask extends Task {

}

@Entity
@Table(name = "sync_task")
@Data
@AllArgsConstructor
@SuperBuilder
@EqualsAndHashCode(callSuper = false)
public class SyncTask extends Task {

}
```

We define a repository for the class `Task`:

```java
@NoRepositoryBean
public interface TaskRepository<T extends Task> extends JpaRepository<T, Long> {

    @Query("SELECT t FROM #{#entityName} t WHERE t.name = ?1")
    public List<T> findTaskByName(String taskName);
}
```

After that we can define our repositories for children classes : 
```java
public interface SyncTaskRepository extends TaskRepository<SyncTask> {
}

public interface AsyncTaskRepository extends TaskRepository<AsyncTask> {
}
```

And finally, we can use them like another repository to obtains data :

```java
@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class TaskServiceAdapter implements TaskService {

    private final AsyncTaskRepository repoAsync;
    private final SyncTaskRepository repoSync;

    @Transactional
    @Modifying(flushAutomatically = true, clearAutomatically = true)
    public void run() {

        repoAsync.saveAll(
                Arrays.asList(
                        AsyncTask.builder().name("Scheduling").build(),
                        AsyncTask.builder().name("Cleaning").build()
                )
        );

        repoSync.saveAll(
                Arrays.asList(
                        SyncTask.builder().name("Downloading").build(),
                        SyncTask.builder().name("Reporting").build()
                )
        );

        log.info(" -- finding async task  --");
        List<AsyncTask> list = repoAsync.findTaskByName("Cleaning");
        list.forEach(asyncTask -> log.info(asyncTask.toString()));

        log.info(" -- finding sync task  --");
        List<SyncTask> list2 = repoSync.findTaskByName("Reporting");
        list2.forEach(syncTask -> log.info(syncTask.toString()));
    }
}
```
