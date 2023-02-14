That arrive we need to intercept response before it be returned to the client.

We can simply do that with this special component :
```java
@ControllerAdvice
@AllArgsConstructor
@Slf4j
public class CustomControllerAdvice implements ResponseBodyAdvice<Object> {

    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {

        // Do something

        return body;
    }
}
```