
## AOP

### Spring AOP Unit Test

```java
@SpringBootTest
@SpringJUnitConfig(classes = {AopReflectionTestsTest.AopConfig.class, AopReflectionTestsTest.ToolAspect.class})
@Slf4j
public class AopReflectionTestsTest {

    @Autowired
    private ToolHandler toolHandler;

    @Test
    void test_find_tool_methods() throws Exception {
        var toolObject = toolHandler;
        var toolObjectClass = toolObject.getClass();
        var targetClass = AopUtils.getTargetClass(toolObject);
        
        assertTrue(AopUtils.isAopProxy(toolObject));
        assertNotEquals(ToolHandler.class, toolObjectClass);
        assertEquals(ToolHandler.class, targetClass);

        log.info("toolObject.getClass(): {}", toolObject.getClass());
        log.info("AopUtils.getTargetClass(toolObject): {}", AopUtils.getTargetClass(toolObject));

        var methodFromTargetClass = Stream.of(ReflectionUtils.getDeclaredMethods(targetClass)).filter(m -> m.getName().equals("annotatedMethod")).findFirst().get();
        var methodFromObjectClass = Stream.of(ReflectionUtils.getDeclaredMethods(toolObjectClass)).filter(m -> m.getName().equals("annotatedMethod")).findFirst().orElse(null);

        assertNotNull(methodFromTargetClass);
        assertNotNull(methodFromObjectClass);

        var args = new Object[] { null, null };
        methodFromTargetClass.invoke(toolObject, args);
        methodFromObjectClass.invoke(toolObject, args);
    }

    @Configuration
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    public static class AopConfig {
        @Bean
        ToolHandler toolHandler() { return new ToolHandler(); }
    }

    // Target class for testing
    public static class ToolHandler {
        @Tool
        public String annotatedMethod(String input, Boolean flag) { return "Processed: " + input; }
        // public String sayHello(String name) { return "Hello, " + name; }
    }

    // Aspect for handling @Tool annotation
    @Aspect
    @Component
    public static class ToolAspect {

        @Pointcut("@annotation(org.springframework.ai.tool.annotation.Tool)")
        public void toolAnnotatedMethods() {}

        @Around("toolAnnotatedMethods()")
        public Object aroundToolMethods(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("@Around(before) - {}", joinPoint.getTarget().getClass());

            var result = joinPoint.proceed();

            log.info("@Around(after) - {}", joinPoint.getTarget().getClass());
            return result;
        }

        @After("toolAnnotatedMethods()")
        public void afterToolMethods() {
            log.info("@After");
        }
    }

}
```

## Configuration

### @Bean 메소드에서 Nullable Bean 객체 주입 받기

스프링 Java 설정 클래스에서 `@Bean` 메서드의 파라미터로 다른 빈 객체를 받을 때, 해당 빈이 없으면 오류를 내지 않고 처리하고 싶다면 다음 방법을 사용할 수 있습니다.

- 1) `@Nullable` 어노테이션을 붙여 파라미터가 없어도 null로 받게 하기  
- 2) 스프링의 `ObjectProvider<T>` 타입을 받아, `getIfAvailable()` 메서드로 빈이 있는지 선택적으로 조회하기  

예시 코드:

```java
@Bean
public MyBean myBean(@Nullable OtherBean otherBean) {
    if (otherBean == null) {
        // 다른 빈이 없을 때 처리 로직
    }
    return new MyBean(otherBean);
}
```

또는

```java
@Bean
public MyBean myBean(ObjectProvider<OtherBean> otherBeanProvider) {
    OtherBean otherBean = otherBeanProvider.getIfAvailable();
    // otherBean이 null일 수 있어서 선택적으로 처리 가능
    return new MyBean(otherBean);
}
```

이렇게 하면 스프링 컨텍스트에 해당 빈이 없어도 오류가 발생하지 않고, 프로그래밍적으로 빈의 존재 여부를 확인해서 처리할 수 있습니다. `ObjectProvider`를 쓰는 방법이 좀 더 유연해서 많이 사용됩니다.  

요약하면, `@Bean` 메서드 파라미터에 빈이 선택적으로 주입되도록 하려면 `@Nullable` 또는 `ObjectProvider`를 활용하세요.