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

The audit fields (reuse ;-) ):
```java
@Getter
@Setter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class Auditable {

    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;

    @CreatedDate
    @Column(name = "created_date", updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @LastModifiedBy
    @Column(name = "last_modified_by")
    private String lastModifiedBy;

    @LastModifiedDate
    @Column(name = "last_modified_date")
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
}
```

And now we can audit our entities like that :
```java
@Entity
@Table(name = "certificates")
@Audited
@AuditOverride(forClass = Auditable.class)
@Data
@EqualsAndHashCode(callSuper = true)
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CertificateEntity extends Auditable {
    // Noting more because we use inheritence
}
```