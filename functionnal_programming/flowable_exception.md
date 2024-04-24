Sometimes, when we use functional programming for a more fluent code, we are faced with exceptions that force us to wrap 
the code inside a `try-catch` block.

So the code is less readable and fluent.

```java
this.sut.findAll(20, 0)
    .stream()
    .map(user -> {
        try {
            return objectMapper.writeValueAsString(user);
        } catch (JsonProcessingException e) {
            throw new FakeRuntimeException(e);
        }
    })
    .forEach(log::info);
```

The code is just ugly and the block `try-catch` don't add really useful information.

The solution to that is these methods :
```java
@FunctionalInterface
public interface CheckedFunction<T, R> {
    R apply(T t) throws Exception;
}

/**
 * This method wrap the code to execute and handle it in a try-catch block
 * @param checkedFunction the function to execute
 * @return the result of the executed function
 * @param <T> The type of the input
 * @param <R> the type of the output
 */
public static <T, R> Function<T, R> wrap(CheckedFunction<T, R> checkedFunction) {
    return t -> {
        try {
            return checkedFunction.apply(t);
        } catch (Exception e) {
            throw new RuntimeException(e); // or custom subclass of RuntimeException
        }
    };
}

/**
 * This method wrap the code to execute and handle it in a try-catch block
 * @param exception The exception that will be raised if an exception occurs during the function execution
 * @param checkedFunction the function to execute
 * @return the result of the executed function
 * @param <T> The type of the input
 * @param <R> the type of the output
 */
public static <T, R> Function<T, R> wrap(Class<? extends RuntimeException> exception, CheckedFunction<T, R> checkedFunction) {

    return t -> {
        try {
            return checkedFunction.apply(t);
        } catch (Exception e) {
            try {
                throw exception.getDeclaredConstructor(Throwable.class).newInstance(e);
            } catch (InstantiationException | IllegalAccessException | InvocationTargetException |
                     NoSuchMethodException ex) {

                throw new RuntimeException(ex);
            }
        }
    };
}
```

And finally, the corrected code : 
```java
this.sut.findAll(20, 0)
    .stream()
    .map(wrap(FakeRuntimeException.class, objectMapper::writeValueAsString))
    .forEach(log::info);
```

This code is beautifuler than the previous one and more readable.