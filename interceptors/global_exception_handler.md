It is preferable to intercept all exception to return a controlled message to the client.

To do that, we need a custom exception :
```java
@Getter
public class BusinessException extends RuntimeException {

    private final HttpStatus httpStatus;
    private final String reason;

    @Builder
    public BusinessException(HttpStatus httpStatus, String reason, String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);

        this.httpStatus = httpStatus;
        this.reason = reason;
    }
}
```

That we can use like that :
```java
try {
    // Do something

} catch (Exception exception) {

    throw BusinessException.builder()
            .httpStatus(HttpStatus.INTERNAL_SERVER_ERROR)
            .message(exception.getMessage())
            .cause(exception.getCause())
            .build();
}
```

And finally, our global exception handler :
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandlerAdvice extends ResponseEntityExceptionHandler {

    private static final URI TYPE_BAD_REQUEST = URI.create("https://www.gcloud.belgium.be/rest/problems/badRequest");
    private static final URI TYPE_RESOURCE_NOT_FOUND = URI.create("https://www.gcloud.belgium.be/rest/problems/resourceNotFound");
    private static final URI TYPE_PROBLEMS = URI.create("https://www.gcloud.belgium.be/rest/problems");

    private String calculateTitle(HttpStatus status) {

        return "Internal server error";
    }

    private URI calculateProblemType(HttpStatus status) {

        return TYPE_PROBLEMS;
    }

    @Override
    protected ResponseEntity<Object> handleExceptionInternal(Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {

        log.error(String.format("Request %s raised exception", request.toString()), ex);

        headers.add(HttpHeaders.CONTENT_TYPE, "application/problem+json");

        if (!(body instanceof Problem)) {

            Problem problem = new Problem();

            problem.setType(this.calculateProblemType(status));
            problem.setTitle(this.calculateTitle(status));
            problem.setStatus(status.value());
            problem.setDetail(ex.getMessage());

            return super.handleExceptionInternal(ex, problem, headers, status, request);
        } else {
            return super.handleExceptionInternal(ex, body, headers, status, request);
        }
    }

    @ExceptionHandler({Exception.class})
    @Nullable
    public final ResponseEntity<Object> handleOtherException(Exception ex, WebRequest request) {

        HttpHeaders headers = new HttpHeaders();
        HttpStatus status = HttpStatus.INTERNAL_SERVER_ERROR;

        Problem problem = new Problem();
        problem.setType(this.calculateProblemType(status));
        problem.setTitle(this.calculateTitle(status));
        problem.setStatus(status.value());
        problem.setDetail(ex.getMessage());

        return handleExceptionInternal(ex, problem, headers, status, request);
    }

    @ExceptionHandler({BusinessException.class})
    @Nullable
    public final ResponseEntity<Object> handleBusinessException(BusinessException ex, WebRequest request) {

        HttpHeaders headers = new HttpHeaders();

        Problem problem = new Problem();
        problem.setType(this.calculateProblemType(ex.getHttpStatus()));
        problem.setTitle(this.calculateTitle(ex.getHttpStatus()));
        problem.setStatus(ex.getHttpStatus().value());

        if (nonNull(ex.getReason())) {
            problem.setDetail(ex.getReason());
        } else {
            problem.setDetail(ex.getMessage());
        }

        return handleExceptionInternal(ex, problem, headers, ex.getHttpStatus(), request);
    }
}
```