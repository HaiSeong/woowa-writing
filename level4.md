# SPRING AOP 알아보기

## AOP(Aspect-Oriented Programming)의 개념

AOP(Aspect-Oriented Programming, 관점 지향 프로그래밍)는 소프트웨어 개발에서 횡단 관심사(Cross-Cutting Concerns)를 모듈화하여 관리하는 프로그래밍 패러다임입니다. 

전통적인 객체 지향 프로그래밍(Object-Oriented Programing)은 기능 단위로 클래스를 나눕니다. 각 클래스는 자신의 역할에 맞는 비즈니스 로직을 수행합니다. 이때 로깅, 트랜잭션 관리, 보안, 예외 처리와 같은 기능은 애플리케이션 전반에 걸쳐 여러 클래스에 공통적으로 적용되어야 합니다. 이렇게 소프트웨어 개발에서 특정 모듈이나 기능과는 직접적으로 관련이 없지만, 시스템의 여러 부분에 걸쳐 공통적으로 적용되는 기능이나 요구사항을 횡단 관심사라고 부릅니다.

![image](./image/횡단%20관심사.png)

횡단 관심사는 여러 모듈에 걸쳐 중복되어 이로 인해 코드의 유지보수성이 저하되고 모듈 간 결합도가 높아지는 문제가 발생할 수 있습니다. 예를 들어, 게시글 작성이라는 비즈니스 로직을 처리하는 메소드에 트랜잭션 관리를 도입하는 상황을 예시로 들어보겠습니다. 메소드의 본래 목적은 게시글 작성 로직을 처리하는 것임에도 불구하고 트랜잭션을 시작하고, 성공 시 커밋하며, 오류 발생 시 롤백하는 코드를 메소드 안에 직접 추가해야 합니다. 이러한 방식은 코드의 재사용성과 가독성을 저하시킬 뿐만 아니라, 유지보수 시에도 중복된 코드들을 모두 수정해야 하므로 생산성이 떨어집니다. 

```java

@Service
public class PostService {

    ...

    public void createPost(Post post) {
        try {
            // 트랜잭션 시작
            beginTransaction();
            
            // 게시글 작성 비즈니스 로직
            savePost(post);
            
            // 트랜잭션 커밋
            commitTransaction();
        } catch (Exception e) {
            // 예외 발생 시 트랜잭션 롤백
            rollbackTransaction();
            throw e;
        }
    }

    ...

}
```

AOP는 이 문제를 해결하기 위해 로직과 횡단 관심사를 분리하여 관리하는 방법을 제공합니다. AOP는 비즈니스 로직과 관련 없는 횡단 관심사를 별도의 모듈(Aspect)로 정의하고, 해당 모듈을 특정 시점에 자동으로 적용할 수 있도록 설계되었습니다. 이를 통해 핵심 비즈니스 로직은 간결하고 명확하게 유지되며, 횡단 관심사는 필요에 따라 재사용 및 관리가 용이해집니다. 

```java

@Service
public class PostService {

    ...

    @Transactional
    public void createPost(Post post) {
        // 게시글 작성 비즈니스 로직
        savePost(post);
    }

    ...

}
```

두가지 대표적인 AOP 프레임 워크를 사용하여 AOP를 구현할 수 있습니다. Spring AOP와 AspectJ 입니다. Spring AOP는 스프링 IoC 컨테이너에서 관리되는 빈(bean)에 대해 단순하고 쉽게 AOP 기능을 적용할 수 있도록 설계되었습니다. AspectJ는 더 강력하고 완전한 AOP 솔루션으로, 스프링 컨테이너에 종속되지 않고 모든 자바 객체에 적용할 수 있습니다. 하지만 AspectJ는 러닝 커브가 복잡하여 이번 글에서는 Spring AOP를 바탕으로 설명하겠습니다.

## Spring AOP의 주요 구성 요소

Spring AOP는 횡단 관심사를 처리하기 위해 사용되는 스프링 프레임워크의 기능으로, 비즈니스 로직에 공통된 기능을 쉽게 적용할 수 있도록 해줍니다. Spring AOP의 주요 구성 요소는 애스펙트(Aspect), 조인포인트(JoinPoint), 어드바이스(Advice), 포인트컷(Pointcut), 타겟(Target), 그리고 위빙(Weaving)으로 구성됩니다. 각 구성 요소는 AOP의 핵심 개념을 이해하는 데 중요한 역할을 합니다. 

![image](./image/aop의%20구성요소.png)

### 어드바이스(advice)

어드바이스는 애스펙트가 특정 조인포인트에서 수행할 작업을 정의하는 코드입니다. Spring AOP에서 메서드 실행 전후에 추가적인 로직을 삽입할 수 있게 해주는 메커니즘입니다. 이를 통해 로깅, 보안, 트랜잭션 관리 등의 횡단 관심사를 효과적으로 처리할 수 있습니다. 어드바이스는 언제 실행될지에 따라 여러 유형으로 나뉩니다. 

#### Before Advice: 메소드 실행 전에 실행됩니다. 

```java

    // Before Advice
    @Before("execution(* com.example.service.UserService.createUser(..))")
    public void logBeforeCreatingUser() {
        System.out.println("사용자 생성을 시작합니다.");
    }

```

#### After Returning Advice: 메소드가 정상적으로 실행된 후에 실행됩니다. 

```java

    // After Returning Advice
    @AfterReturning(pointcut = "execution(* com.example.service.UserService.getUser(..))", returning = "result")
    public void logAfterReturningUser(Object result) {
        System.out.println("사용자 정보를 성공적으로 조회했습니다: " + result);
    }

```

#### After Throwing Advice: 메소드 실행 중 예외가 발생했을 때 실행됩니다. 

```java

    // After Throwing Advice
    @AfterThrowing(pointcut = "execution(* com.example.service.UserService.updateUser(..))", throwing = "ex")
    public void logAfterThrowingException(Exception ex) {
        System.out.println("사용자 정보 업데이트 중 예외 발생: " + ex.getMessage());
    }

```

#### After Advice: 메소드 실행 결과와 상관없이 항상 실행됩니다. 

```java

    // After Advice
    @After("execution(* com.example.service.UserService.deleteUser(..))")
    public void logAfterDeletingUser() {
        System.out.println("사용자 삭제 작업이 완료되었습니다.");
    }

```

#### Around Advice: 메소드 실행 전후 또는 메소드 실행 자체를 제어할 수 있는 가장 강력한 어드바이스입니다. 

```java

    // Around Advice
    @Around("execution(* com.example.service.UserService.listUsers(..))")
    public Object logAroundListUsers(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("사용자 목록 조회를 시작합니다.");
        long start = System.currentTimeMillis();
        
        Object result = joinPoint.proceed(); // 실제 메소드 실행
        
        long executionTime = System.currentTimeMillis() - start;
        System.out.println("사용자 목록 조회가 완료되었습니다. 실행 시간: " + executionTime + "ms");
        
        return result;
    }

```

### 조인포인트(JoinPoint) 

조인포인트는 애플리케이션 실행 중 특정 시점으로, 애스펙트가 적용될 수 있는 위치를 말합니다. 예를 들어, 메소드 호출 전, 메소드 종료 후, 또는 메소드 실행 중 예외가 발생했을 때 등이 조인포인트로 간주됩니다. Spring AOP는 메소드 실행 시점에서만 조인포인트를 지원합니다. 

Spring AOP에서 조인포인트를 다루기 위해 org.aspectj.lang.JoinPoint 인터페이스를 제공합니다. 이 인터페이스를 통해 현재 실행 중인 메소드에 대한 정보를 얻을 수 있습니다. 주요 메소드는 다음과 같습니다:

- getArgs(): 메소드의 인자를 반환합니다.
- getThis(): 현재 프록시 객체를 반환합니다.
- getTarget(): 대상 객체를 반환합니다.
- getSignature(): 조인포인트의 시그니처(메소드 선언)를 반환합니다.
- toString(): 조인포인트에 대한 유용한 설명을 문자열로 반환합니다.

```java

@Before("execution(* com.example.service.UserService.*(..))")
public void logBefore(JoinPoint joinPoint) {
    String methodName = joinPoint.getSignature().getName();
    Object[] args = joinPoint.getArgs();
    System.out.println("메소드 " + methodName + "이(가) 다음 인자로 호출됨: " + Arrays.toString(args));
}

```

### 포인트컷(Pointcut)

포인트컷은 어떤 조인포인트에서 어드바이스가 실행될지 결정하는 기준입니다. 주로 정규 표현식이나 패턴 매칭을 사용하여 특정 메소드나 클래스에 어드바이스를 적용할 수 있습니다. 포인트컷을 통해 어드바이스가 적용될 메소드나 클래스의 조건을 세밀하게 지정할 수 있어, 불필요한 코드 실행을 줄이고 성능을 최적화할 수 있습니다. 

Spring AOP는 AspectJ의 포인트컷 표현식 문법을 사용합니다. 가장 일반적인 형태는 다음과 같습니다. 

```

// ?는 옵션값임
execution(modifiers-pattern? return-type-pattern declaring-type-pattern? method-name-pattern(param-pattern) throws-pattern?)

```

Spring AOP는 다양한 포인트컷 지시자를 지원합니다.

#### execution(): 메소드 실행 조인포인트를 매칭합니다.
```java
    // 1. execution()
    @Before("execution(* com.example.service.UserService.createUser(..))")
    public void beforeUserCreation() {
        System.out.println("사용자 생성 전 로깅");
    }
```

#### within(): 특정 타입 내의 조인포인트를 매칭합니다.
```java
    // 2. within()
    @Pointcut("within(com.example.service.*)")
    public void inServiceLayer() {}

    @Before("inServiceLayer()")
    public void beforeServiceMethod() {
        System.out.println("서비스 레이어 메소드 실행 전 로깅");
    }
```

#### this: 프록시 객체가 지정한 타입의 인스턴스인 조인포인트를 매칭합니다.
```java
    // 3. this
    @Pointcut("this(com.example.service.UserService)")
    public void thisUserService() {}

    @Before("thisUserService()")
    public void beforeUserServiceProxyMethod() {
        System.out.println("UserService 프록시 메소드 실행 전 로깅");
    }
```

#### target: 대상 객체가 지정한 타입의 인스턴스인 조인포인트를 매칭합니다.
```java
    // 4. target
    @Pointcut("target(com.example.service.UserService)")
    public void targetUserService() {}

    @Before("targetUserService()")
    public void beforeUserServiceTargetMethod() {
        System.out.println("UserService 대상 객체 메소드 실행 전 로깅");
    }
```

#### args(): 인자가 지정한 타입의 인스턴스인 조인포인트를 매칭합니다.
```java
    // 5. args()
    @Before("execution(* com.example.service.UserService.*(..)) && args(username, ..)")
    public void beforeMethodWithUsernameArg(String username) {
        System.out.println("Username 인자를 가진 메소드 실행 전 로깅: " + username);
    }

```

### 애스펙트(Aspect)

애스펙트는 횡단 관심사를 모듈화한 것입니다. 스프링 AOP에서는 주로 메소드 실행 시점을 기준으로 로깅, 트랜잭션 관리, 보안 등의 기능을 제공하는 데 사용됩니다. 애스펙트는 여러 어드바이스와 포인트컷을 결합하여, 특정 지점에서 특정 동작을 수행하도록 정의합니다. 예를 들어, 메소드 호출 전후에 로깅 기능을 추가하거나, 트랜잭션 처리를 애스펙트로 정의할 수 있습니다. `@Aspect` 어노테이션을 붙여 에스팩트 모듈임을 명시합니다.

### 타겟(Target)
타겟은 실제로 어드바이스가 적용될 객체 또는 메소드를 의미합니다. 예를 들어, 서비스 클래스의 특정 메소드에 트랜잭션 관리나 로깅 기능을 추가하려는 경우, 그 메소드가 타겟이 됩니다. 스프링 AOP는 이러한 타겟 객체에 동적으로 프록시 객체를 생성하여 어드바이스를 적용합니다. 

### 위빙(Weaving) 
위빙은 애스펙트를 실제로 타겟 객체에 연결하는 과정입니다. Spring AOP에서는 런타임 시에 프록시 객체를 생성하여 위빙을 수행합니다. 이를 런타임 위빙(Runtime Weaving)이라고 하며, 실행 중에 동적으로 프록시를 사용해 어드바이스를 타겟 메소드에 적용합니다. 위빙 방식에는 컴파일타임, 로드타임, 런타임 위빙이 있지만, Spring AOP는 런타임 위빙을 사용합니다.

## 프록시 패턴과 AOP
Spring AOP는 프록시 패턴(Proxy Pattern) 을 활용하여 횡단 관심사를 적용합니다. 프록시 패턴은 객체의 대리자 역할을 하는 또 다른 객체인 프록시를 생성하여, 실제 객체에 접근하는 방식을 제어하는 디자인 패턴입니다. 이를 통해 실제 객체에 메소드 호출을 전달하기 전후로 부가적인 작업을 처리할 수 있습니다.

Spring AOP는 주로 JDK 동적 프록시와 CGLIB(Code Generation Library) 프록시를 사용하여 런타임에 프록시 객체를 생성합니다.

- JDK 동적 프록시는 인터페이스를 구현한 클래스에 대해 프록시를 생성합니다. 이때 프록시가 인터페이스를 통해 메소드 호출을 가로채고, 어드바이스를 적용한 후, 실제 메소드를 호출합니다.
- CGLIB 프록시는 인터페이스가 없는 클래스에도 프록시를 적용할 수 있는 방식으로, 바이트코드를 조작해 실제 클래스를 상속받아 프록시를 생성합니다. 이는 인터페이스가 없는 구체 클래스에 대해 사용됩니다.

Spring AOP는 이러한 프록시 패턴을 사용하여 어드바이스(Advice) 를 런타임에 동적으로 적용합니다. 예를 들어, 비즈니스 로직이 있는 메소드 호출 전에 로깅을 하거나, 메소드가 종료된 후 트랜잭션을 커밋하는 등의 작업이 가능합니다. 다음은 프록시 패턴을 통한 AOP 적용 과정입니다.

클라이언트가 메소드를 호출하면, 프록시 객체가 해당 호출을 가로챕니다.
프록시 객체는 어드바이스를 실행합니다. 실제 타겟 메소드를 호출한 후 결과를 반환합니다.
타겟 메소드의 결과에 따라 후처리 작업을 진행할 수 있습니다. 이와 같은 방식으로 프록시 패턴을 사용하여 Spring AOP는 런타임에 동적으로 횡단 관심사를 적용할 수 있습니다. 이를 통해 코드가 간결해지고, 중복된 횡단 관심사를 쉽게 관리할 수 있습니다.

## 결론

AOP는 비즈니스 로직과 횡단 관심사를 분리함으로써 코드의 가독성과 유지보수성을 크게 향상시킵니다. 로깅, 트랜잭션 관리, 보안과 같은 공통 기능을 모듈화하여 각 클래스에서 중복되는 코드를 줄일 수 있고, 시스템의 확장성과 유지보수성도 높일 수 있습니다. Spring AOP는 이러한 AOP의 기능을 손쉽게 활용할 수 있도록 지원하며, 특히 스프링 프레임워크와 통합되어 애플리케이션 전반에서 다양한 횡단 관심사를 효율적으로 처리할 수 있게 해줍니다.

Spring AOP의 특징인 런타임 위빙 방식은 개발자가 복잡한 위빙 로직을 신경 쓰지 않도록 하면서도 필요한 시점에 정확히 어드바이스를 적용할 수 있습니다. 또한, 포인트컷 표현식을 통해 매우 세밀한 제어가 가능해 불필요한 리소스 소모를 줄일 수 있습니다.

결론적으로 AOP는 횡단 관심사를 효과적으로 관리하여 대규모 애플리케이션에서 코드 중복을 방지하고, 유지보수성과 생산성을 향상시키는 강력한 도구입니다. 개발 과정에서 반복적이고 공통적인 문제를 해결하고 싶다면 Spring AOP를 고려해보는 것이 좋은 선택이 될 것입니다.
