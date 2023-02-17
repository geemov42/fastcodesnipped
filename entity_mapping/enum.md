To map an enum, you have several ways :
* Enumerated mapping
* Attribute converter

### Attribute converter

```java
public enum VideoType {
    MOVIE(1),
    TV_SHOW(2);

    private final int value;
}

public class VideoTypeConverter implements AttributeConverter<VideoType, Integer> {

    @Override
    public Integer convertToDatabaseColumn(VideoType attribute) {

        return attribute.getValue();
    }

    @Override
    public VideoType convertToEntityAttribute(Integer dbData) {

        return VideoType.fromValue(dbData);
    }
}

@Column(name = "type", nullable = false)
@Convert(converter = VideoTypeConverter.class)
private VideoType videoType;
```

### String mapping

```java
@Enumerated(EnumType.STRING)
@Column(name = "type", nullable = false)
private VideoType videoType;
```

### Ordinal mapping

> It is preferable to don't use `ORDINAL` because if you change the enum, you break data.

```java
@Enumerated
@Column(name = "type", nullable = false)
private VideoType videoType;
```
