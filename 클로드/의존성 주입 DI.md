
# 왜 사용하는가?
## 질문 1. 왜
### 답변
The main benefits of constructor injection, despite having slightly more code, are:

1. Guaranteed Initialization
```java
// With field injection - risky!
@Autowired
private UserService userService;  // Could be null if something goes wrong

// With constructor - guaranteed to have UserService
private final UserService userService;
public UserController(UserService userService) {
    this.userService = userService;
}
```

2. Easier Testing
```java
// Easy to test with constructor injection
@Test
void testController() {
    UserService mockService = mock(UserService.class);
    UserController controller = new UserController(mockService);
    // Now test with mockService
}

// Field injection makes testing harder - you need reflection or special test utilities
```

3. Clear Dependencies
- You can instantly see what a class needs by looking at its constructor
- With field injection (`@Autowired`), dependencies are scattered throughout the class
클래스가 필요로 하는 의존성을 명확하게 볼 수 있다는 것.
즉, 필드나 생성자에서 한번에 확인할 수 있기 때문에 명확히 볼 수 있다는 점.
생성자나 필드에서 의존성을 주입하지 안는다면 그때마다 객체를 생성해야 하는데 그에 대한 단점은 다음 질문에서.

1. Immutability with `final`
- Once set in constructor, the service reference can't be changed
- Prevents bugs from accidental reassignment

So while it's a few more lines of code, these benefits make your code more *reliable* and *maintainable* in the *long run*. It's a classic case of "an ounce of prevention is worth a pound of cure" - the extra code helps prevent bugs and makes testing easier.
#### 질문 1-1. 정확히 의존성 주입이란 무엇인가?
A 클래스가 B클래스를 필요로 하기 때문에, 그래서 생성하기 때문에, A에게 B에 대한 의존성을 주입한다는 것인가?
##### 답변
의존성 주입의 핵심은 클래스 자신이 의존성을 생성하는 것이 아니라, 외부(스프링 컨테이너 등)에서 이미 생성된 객체를 받아서 사용한다는 점입니다. 이렇게 하면 더 유연하고 결합도가 낮은 코드를 작성할 수 있습니다.
##### 정리
그니까 비슷한데, A가 B를 필요로해서 의존한다는 것인데 그 의존을 외부에서(스프링부트)에서 객체를 만들어 의존하는 객체에게 주입한다.
그 방법으로 생성자 주입, 필드 주입이 있는것이다.

## 질문2 : new 생성자는 안되는 이유는?
### 답변
The issue with `private UserService userService = new UserService();` is that it creates tight coupling and makes your code harder to maintain:

1. Tight Coupling Problem: 결합도가 높아짐
```java
// Tightly coupled - UserController is directly responsible for creating UserService
private UserService userService = new UserService();

// But wait, what if UserService needs its own dependencies?
public class UserService {
    private UserMapper mapper;
    private UserRepository repo;
    
    // Now UserController would need to know about these too!
    public UserService(UserMapper mapper, UserRepository repo) {
        this.mapper = mapper;
        this.repo = repo;
    }
}
```

1. Testing Problem:
```java
// With direct instantiation, you can't swap in a mock
private UserService userService = new UserService();

// With constructor injection, easy to mock
@Test
void testController() {
    UserService mockService = mock(UserService.class);
    when(mockService.getUser(1L)).thenReturn(new UserDTO());
    
    UserController controller = new UserController(mockService);
    // Now you can test without hitting real database
}
```

3. Configuration Problem:
```java
// What if UserService needs different configurations for dev/prod?
public class UserService {
    private final String apiUrl;
    private final int timeout;
    
    // Direct instantiation can't handle different configurations easily
    // But Spring can inject different configs based on environment
}
```

The key benefit is that *your classes don't need to know how to create their dependencies* - they just declare what they need and let Spring handle the rest. This is especially important in large applications where services might have complex initialization requirements or different configurations in different environments.

## 질문3 : Tight Coupling Problem?
### 답
Let me show you why direct instantiation creates this problem:

If you write:
```java
public class UserController {
    // Trying direct instantiation
    private UserService userService = new UserService();
}
```

But then UserService needs its dependencies:
```java
public class UserService {
    private UserMapper mapper;
    private UserRepository repo;
    
    public UserService(UserMapper mapper, UserRepository repo) {
        this.mapper = mapper;
        this.repo = repo;
    }
}
```

Now your UserController code would have to become:
```java
public class UserController {
    // Now forced to create everything!
    private UserService userService = new UserService(
        new UserMapper(),  // But wait, UserMapper is an interface - can't instantiate!
        new UserRepository()  // This might need database connection details!
    );
}
```

With dependency injection instead:
```java
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

UserController only knows it needs a UserService - it doesn't care how UserService gets its mapper and repo. Spring handles creating everything in the right order with the right dependencies.

Think of it like a restaurant - you just tell the waiter what meal you want (UserService), you don't need to know all the ingredients (mapper, repo) or how to make it. The kitchen (Spring) handles all that complexity for you.

#### 질문 3-1. 결합도가 높아진다는게 무슨 의미?
##### 답
강한 결합(Tight Coupling)의 문제점

1. **변경의 파급 효과**: 한 클래스가 변경되면 이에 의존하는 다른 클래스들도 함께 수정해야 할 가능성이 높아집니다.
2. **테스트 어려움**: 의존 관계가 강하게 결합되어 있으면 단위 테스트 시 가짜 객체(mock)로 대체하기 어렵습니다.
3. **재사용성 저하**: 특정 구현에 강하게 결합된 클래스는 다른 컨텍스트에서 재사용하기 어렵습니다.
4. **유연성 부족**: 구현체를 쉽게 교체할 수 없어 시스템의 유연성이 떨어집니다.

의존성 주입(DI)은 이러한 강한 결합 문제를 해결하기 위한 디자인 패턴으로, 결합도를 낮추어(loose coupling) 더 유연하고 테스트 가능한 코드를 작성할 수 있게 해줍니다.
##### 정리
의존성은 여러 클래스끼리 상호작용하는 객체지향 프로그래밍에서 불가피한 현상이다. 그런데 이걸 그냥 둔다면 개발 과정(유지보수, 테스트 등)에서 큰 번거로움과 불편함이 따른다. 
