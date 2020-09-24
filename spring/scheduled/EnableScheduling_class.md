# @interface EnableScheduling

스프링의 `<task : *>` XML 네임 스페이스에 있는 기능과 유사한 스프링의 예약된 작업 실행 기능을 활성화 한다.

다음과 같이 `@Configuration` 클래스에서 사용된다.

```java
@Configuration
@EnableScheduling
public class AppConfig {
    // 빈 정의
}
```

이를 통해 컨테이너의 모든 스프링 관리 Bean 에서 `@Scheduled` 어노테이션을 감지 할 수 있다.

```java
package com.myco.tasks;

public class MyTask {
    @Scheduled(fixedRate=1000)
    public void work() {
        // logic ...
    }
}
```

다음 설정은 `MyTask.work()` 이 1000ms 마다 한 번씩 호출되도록 한다.

```java
@Configuration
@EnableScheduling
public class AppConfig {
    @Bean
    public MyTask task() {
        return new MyTask();
    }
}
```

또는 `MyTask` 에 `@Component` 어노테이션이 추가된 경우 다음 설정은 `@Scheduled` 메소드가 원하는 간격으로 호출되도록 한다.

```java
@Configuration
@ComponentScan(basePackages="com.myco.tasks")
public class AppConfig {
}
```

`@Scheduled` 어노테이션이 달린 메소드는 `@Configuration` 클래스 내에서 직접 선언 할 수도 있다.

```java
@Configuration
@EnableScheduling
public class AppConfig {
    @Scheduled(fixedRate=1000)
    public void work() {
        // task execution logic
    }
}
```

위의 모든 시나리오에서 기본 단일 스레드 작업 실행기가 사용된다.

더 많은 제어가 필요한 경우 `@Configuration` 클래스가 `SchedulingConfigurer` 를 구현할 수 있다.

이를 통해 기본 `ScheduledTaskRegistrar` 인스턴스에 접근 할 수 있다.

예를 들어 다음 예시는 예약 된 실행을 실행하는 데 사용되는 `Executor` 를 맞춤 설정하는 방법을 보여준다.

```java
@Configuration
@EnableScheduling
public class AppConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskExecutor());
    }

    @Bean(destroyMethod="shutdown")
    public Executor taskExecutor() {
        return Executors.newScheduledThreadPool(100);
    }
}
```

위의 예에서 `@Bean(destroyMethod="shutdown")` 을 사용하였다.

이렇게하면 스프링 애플리케이션 컨텍스트 자체가 닫힐 때 태스크 실행기가 올바르게 종료된다.

`SchedulingConfigurer` 를 구현하면 `ScheduledTaskRegistrar` 를 통해 작업 등록을 세밀하게 제어 할 수 있다.

예를 들어 다음은 맞춤 `Trigger` 구현에 따라 특정 Bean 메소드의 실행을 설정한다.

```java
@Configuration
@EnableScheduling
public class AppConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskScheduler());
        taskRegistrar.addTriggerTask(
            new Runnable() {
                public void run() {
                    myTask().work();
                }
            },
            new CustomTrigger()
        );
    }

    @Bean(destroyMethod="shutdown")
    public Executor taskScheduler() {
        return Executors.newScheduledThreadPool(42);
    }

    @Bean
    public MyTask myTask() {
        return new MyTask();
    }
}
```

참고로 위의 예는 다음 스프링 XML 구성과 비교할 수 있다.

```xml
<beans>
    <task:annotation-driven scheduler="taskScheduler"/>
    <task:scheduler id="taskScheduler" pool-size="42"/>
    <task:scheduled ref="myTask" method="work" fixed-rate="1000"/>
    <bean id="myTask" class="com.foo.MyTask"/>
</beans>
```

XML 에서 사용자 정의 `@Trigger` 구현 대신 *고정 비율* 기간이 사용된다는 점을 제외하면 예제는 동일하다.

이는 task 네임 스페이스 schedule 이 이러한 지원을 쉽게 노출 할 수 없기 때문이다.

이것은 코드 기반 접근 방식이 실제 구성 요소에 대한 직접 접근을 통해 구성 가능성을 극대화하는 방법을 보여주는 하나의 데모 일뿐이다.
