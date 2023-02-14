We need sometimes to audit any change executed on rows, for that we have hibernate envers and some classes.

pom.xml
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-envers</artifactId>
    <version>6.1.7.Final</version>
</dependency>
```

Now we need some classes to activate and manage audit.

Activate audit :
```java
@Configuration
@EnableJpaAuditing(auditorAwareRef="auditorAware")
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<String> auditorAware() {
        return new SpringSecurityAuditorAware();
    }
}

class SpringSecurityAuditorAware implements AuditorAware<String> {

    public Optional<String> getCurrentAuditor() {

        return Optional.of("ADMIN");
    }
}
```

An interface that define audit method :
```java
public interface Auditable {

    Audit getAudit();

    void setAudit(Audit audit);
}
```

The audit fields (reuse ;-) ):
```java
@Embeddable
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Audit {

    @Column(name = "created_by")
    @CreatedBy
    private String createdBy;

    @Column(name = "updated_by")
    @LastModifiedBy
    private String updatedBy;

    @Column(name = "created_on")
    @CreatedDate
    private LocalDateTime createdOn;

    @Column(name = "updated_on")
    @LastModifiedDate
    private LocalDateTime updatedOn;
}
```

A custom audit listener that use our previous interface `Auditable` :
```java
@RequiredArgsConstructor
public class AuditListener {

    private final AuditorAware auditorAware;

    @PrePersist
    public void setCreatedOn(Object object) {

        if (!(object instanceof Auditable)) {
            return;
        }

        Auditable auditable = (Auditable) object;
        Audit audit = auditable.getAudit();

        if (audit == null) {
            audit = new Audit();
            auditable.setAudit(audit);
        }

        String auditor = (String) auditorAware.getCurrentAuditor().orElse("UNKNOWN");

        audit.setCreatedOn(LocalDateTime.now());
        audit.setCreatedBy(auditor);
    }

    @PreUpdate
    public void setUpdadtedOn(Object object) {

        if (!(object instanceof Auditable)) {
            return;
        }

        Auditable auditable = (Auditable) object;
        Audit audit = auditable.getAudit();

        String auditor = (String) auditorAware.getCurrentAuditor().orElse("UNKNOWN");

        audit.setUpdatedOn(LocalDateTime.now());
        audit.setUpdatedBy(auditor);
    }
}
```

And now we can audit our entities like that :
```java
@Entity
@Table(name = "account")
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Audited
@EntityListeners(AuditListener.class)
public class AccountEntity implements Auditable {

    ...

    @Embedded
    private Audit audit;

}
```
