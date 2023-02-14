 In modern software we use hexagonal architecture. In that case, we need to multiply data object to not pass an object of extern layer to the main layer (domain). Mapstruct help us to accomplish this goal.

In my code I have a generic interface that define methods for all mappers I will implements : 

```java
public interface GenericMapper<DTO, ENTITY> {

    DTO toDataObject(ENTITY entity);

    Collection<DTO> toDataObjects(Collection<ENTITY> entities);

    @InheritInverseConfiguration
    ENTITY toEntity(DTO dto);

    Collection<ENTITY> toEntities(Collection<DTO> dtos);

    @Condition
    default boolean isInitializedCollection(Collection<?> value) {
        return this.isInitialized(value);
    }

    @Named("isInitialized")
    @Condition
    default boolean isInitialized(Object value) {
        return value != null && Hibernate.isInitialized(value);
    }
}
```

 This interface define a condition on collection to automatically check if they are initialized.

In my object, I have an entity that contains simple object and collection object mapped by hibernate to be lazy. Hibernate proxied the lazy objects and if you try to access the proxied object outside of a transaction, you rise an LazyInitializationException. So, you need to test if an object is initialized before access it.

For simple object, I need to use a condition expression. In that way, I minimized the work about mapping.

Exemple of mapper : 

```java
@Mapper(
        componentModel = "spring",
        injectionStrategy = InjectionStrategy.CONSTRUCTOR,
        uses = {
                CommentMapper.class,
                VideoDetailsMapper.class
        }
)
public interface VideoTvShowMapper extends GenericMapper<TvShow, Video> {

    @Mapping(target = "videoDetails", source = "videoDetails", conditionQualifiedByName = "isInitialized")
    TvShow toDataObject(Video video);

    @Mapping(target = "videoType", expression = "java(io.geemov42.web.myproject.domain.video.VideoType.TV_SHOW)")
    Video toEntity(TvShow movie);
}
```

And finally, a part of the generated code :
```java
@Override
public TvShow toDataObject(Video video) {
    if ( video == null ) {
        return null;
    }

    TvShow.TvShowBuilder tvShow = TvShow.builder();

    if ( isInitialized( video.getVideoDetails() ) ) {
        tvShow.videoDetails( videoDetailsMapper.toDataObject( video.getVideoDetails() ) );
    }
    tvShow.id( video.getId() );
    tvShow.title( video.getTitle() );
    tvShow.description( video.getDescription() );
    if ( isInitializedCollection( video.getComments() ) ) {
        tvShow.comments( commentSetToCommentList( video.getComments() ) );
    }

    return tvShow.build();
}
```

As we can see, my collection is automatically checked with my custom method isInitializedCollection and my simple object use my other custom method through my condition.
