## How to inject scoped beans and test the code

### Problem

When you execute code in separate thread, the spring context don't inject the scoped beans in the new thread.
A way to do that is the following.

The scoped bean :
```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ScopedBean {

    private String value;
}
```

A stupid pojo :
```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.NoArgsConstructor;

@lombok.Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Data {

    private String value;
}
```

My main service that execute the method in a non-blocking way until I want to block :
```java
import lombok.AllArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.core.scheduler.Schedulers;

import java.util.concurrent.atomic.AtomicReference;

@Service
@AllArgsConstructor
public class MyService {

    private final MyOtherService myOtherService;

    public Data myMethod() {

        AtomicReference<Data> myContentFromMono = new AtomicReference<>(new Data());

        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        Flux.merge(
                Mono.fromCallable(myOtherService::doSomethingWithScopedBean)
                        .doFirst(() -> RequestContextHolder.setRequestAttributes(requestAttributes))
                        .doOnNext(returnedValueFromCallable -> myContentFromMono.get().setValue(returnedValueFromCallable))
                        .subscribeOn(Schedulers.boundedElastic())
        ).blockLast();

        return myContentFromMono.get();
    }
}
```

My other service depend on the scoped bean :
```java
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@AllArgsConstructor
@Slf4j
public class MyOtherService {

    private final ScopedBean scopedBean;

    public String doSomethingWithScopedBean() {

        final String value = scopedBean.getValue();
        log.info("value acceded {}", value);

        return value;
    }
}
```

We can test in this way :
```java
import org.assertj.core.api.Assertions;
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
        scopedBean.setValue("value");
        Assertions.assertThat(myService.myMethod())
                .isNotNull()
                .extracting("value")
                .isEqualTo("value");
    }
}
```