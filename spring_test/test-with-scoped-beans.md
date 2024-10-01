## How to inject scoped beans and test the code

### Problem

When you execute code in separate thread, the spring context don't inject the scoped beans in the new thread.
A way to do that is the following.

My main service that execute the method in a non-blocking way until I want to block :
```java
@Service
public class MyService {

    @Autowired
    private MyOtherService myOtherService;
    
    public void myMethod() {

        AtomicReference<Data> myContentFromMono = new AtomicReference<>(new Data());
        
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        Flux.merge(
                Mono.fromCallable(() -> myOtherService.doSomethingWithScopedBean())
                        .doFirst(() -> RequestContextHolder.setRequestAttributes(requestAttributes))
                        .doOnNext(returnedValueFromCallable -> myContentFromMono.get().setValue(returnedValueFromCallable))
                        .subscribeOn(Schedulers.boundedElastic())
        ).blockLast();
    }
}
```

My other service depend on the scoped bean :
```java
@Service
public class MyOtherService {

    @Autowired
    private ScopedBean scopedBean;
    
    public String doSomethingWithScopedBean() {

        final String value = scopedBean.getValueFromRequest();
        System.out.println("value acceded %s".formatted(value));
        
        return value;
    }
}
```

We can test in this way :
```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.mock.web.MockHttpServletRequest;
import org.springframework.web.context.annotation.RequestScope;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

@SpringBootTest(
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, 
        classes = MyServiceTest.TestContextConfiguration.class,
        properties = {
                "spring.main.allow-bean-definition-overriding=true"
        })
class MyServiceTest {

    @Autowired
    private MyService myService;

    @Autowired
    private ScopedBean scopedBean;

    @TestConfiguration
    static class TestContextConfiguration {

        @Bean
        @RequestScope
        public ScopedBean initScopedBean() {
            return new ScopedBean();
        }
    }

    @BeforeEach
    void setUp() {
        RequestContextHolder.setRequestAttributes(new ServletRequestAttributes(new MockHttpServletRequest()));
    }

    @Test
    void myMethod_shouldBeOkWithScopedBeans() {
        scopedBean.setValueFromRequest("value");
        myService.myMethod();
    }
}
```