스프링 MVC는 서블릿 기반으로 동작
톰캣과 같은 서블릿 컨테이너가 애플리케이션을 실행하고 쓰레드 풀을 미리 생성
    - 200개 정도의 쓰레드 풀을 미리 만들어두고 재사용하면서 요청을 처리

요청이 들어오면
톰캣이 쓰레드 풀에서 쓰레드 하나를 할당
DispatcherServlet이 그 쓰레드에서 실행
Controller -> Service -> Repository 등이 모두 같은 쓰레드에서 실행
응답 반환 후 쓰레드는 쓰레드 풀로 반환

HTTP 요청당 하나의 쓰레드가 할당
해당 쓰레드는 요청 처리가 끝날 때까지 계속 사용되고
DB 조회나 외부 API 호출로 blocking이 발생하면? 쓰레드는 대기 상태로 기다린다

서블릿이란?
서블릿은 자바로 웹 애플리케이션을 만들기 위한 기술
HTTP 요청을 받아 처리하기 위한 자바 클래스

서블릿 컨테이너란?
서블릿들의 생명주기를 관리하고 실행 환경을 제공, 대표적으로 Tomcat이 있다
서블릿을 실제로 실행해주는 엔진 역할

브라우저에서 GET /hello HTTP/1.1 요청이 오면
서블릿 컨테이너가 파싱해서 HttpServletRequest 객체로 변환
서블릿 request를 받아서 처리
서블릿 컨테이너 response를 HTTP로 변환하여 전송

서블릿 생명주기 관리
1. 서블릿 인스턴스 생성(최초 1회)
2. init() 메서드 호출 - 초기화
3. 요청마다 service() -> doGet() / doPost() 호출
4. 종료시 destroy() 호출

서블릿 인스턴스는 `서블릿 컨테이너가 시작될 때`, `해당 서블릿에 첫 요청이 올 때` 생성
생성방법에는 두가지 방법이 존재

1. 지연 로딩(기본 방식)
톰캣 시작
  → 서블릿은 아직 생성 안 됨
  → 첫 번째 요청이 들어옴
  → 그때 서블릿 인스턴스 생성!
  → init() 호출
  → 이후 요청들은 이미 생성된 인스턴스 재사용

2. 즉시 로딩
```java
@WebServlet(urlPatterns = "/hello", loadOnStartup = 1)
public class HelloServlet extends HttpServlet {
    // loadOnStartup = 1 : 서버 시작 시 바로 생성
}
```

또는

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.example.HelloServlet</servlet-class>
    <load-on-startup>1</load-on-startup>  <!-- 서버 시작 시 생성 -->
</servlet>
```

톰캣 시작
  → loadOnStartup 설정된 서블릿들 생성
  → init() 호출
  → 요청 대기

init() 메서드는 서블릿이 요청을 처리하기 전 필요한 사전작업을 진행
+ 설정파일 읽기
    - properties 같은
+ DB 커넥션 풀 초기화

스프링에서 생성자도 init()과 비슷한 일을 한다

```java
@RestController
public class UserController {
    
    private final UserService userService;
    
    // 생성자 - 스프링 빈 생성 시 호출 (init()과 비슷)
    public UserController(UserService userService) {
        this.userService = userService;
        System.out.println("Controller 생성!");
    }
    
    // 초기화 작업이 필요하면
    @PostConstruct
    public void init() {
        // 의존성 주입 후 추가 초기화 작업
        System.out.println("초기화 작업 완료!");
    }
    
    @GetMapping("/user/{id}")
    public User getUser(@PathVariable Long id) {
        // 요청 처리
        return userService.findById(id);
    }
    
    @PreDestroy
    public void cleanup() {
        // 종료 전 정리 작업 (destroy()와 비슷)
        System.out.println("정리 작업 완료!");
    }
}
```

생성자도 비슷한 일을 하는데, 왜 init()을 사용하냐?

```java
public class MyServlet extends HttpServlet {
    
    // 생성자
    public MyServlet() {
        // 여기서는 ServletConfig를 받을 수 없어요!
        // ServletContext도 아직 준비 안 됨
        System.out.println("생성자 호출");
    }
    
    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        
        // 여기서는 ServletConfig, ServletContext 사용 가능!
        String initParam = config.getInitParameter("myParam");
        ServletContext context = config.getServletContext();
        
        System.out.println("init() 호출: " + initParam);
    }
}
```

**init()을 사용하는 이유:**
- ServletConfig와 ServletContext에 접근 가능
- 서블릿 컨테이너가 준비를 완료한 후 호출되므로 안전
- 표준화된 초기화 시점 제공

---

```
[서버 시작]
    ↓
1. 서블릿 클래스 로딩
    ↓
2. 서블릿 인스턴스 생성 (생성자 호출)
    ↓
3. init() 호출 ← 초기화 작업!
    ↓
[서비스 중]
    ↓
4. service() → doGet()/doPost() 호출 (매 요청마다)
    ↓
    ↓ (수백만 번 반복)
    ↓
[서버 종료]
    ↓
5. destroy() 호출 ← 정리 작업!
    ↓
6. 인스턴스 제거
```

**실행 예시:**
```
[톰캣 시작]
서블릿 생성자 호출
init() 호출 - DB 커넥션 풀 생성
init() 호출 - 캐시 데이터 로딩
서블릿 준비 완료!

[요청 1] Thread-1: doGet() 실행
[요청 2] Thread-2: doGet() 실행
[요청 3] Thread-3: doGet() 실행
... 수백만 번 ...

[톰캣 종료]
destroy() 호출 - 커넥션 풀 닫기
destroy() 호출 - 캐시 정리
서블릿 종료
```

용어정리
+ 서블릿 컨테이너
    - 톰캣, Jetty같은 **프로그램 자체**
+ 웹 서버
    - Apache, Nginx처럼 정적 파일을 서빙하는 서버
    - 톰캣은 웹서버 기능을 포함하고 있음
+ WAS
    - 서블릿 컨테이너 + 웹 서버 + 추가 엔터프라이즈 기능
    - 톰캣은 `경량 WAS`

그래서 서버 == 서블릿 컨테이너? 다르지만 틀린 답은 아닌 표현의 차이
애플리케이션 1개당 서블릿 컨테이너 1개? 
반대로 서블릿 컨테이너 1개가 여러 애플리케이션을 관리한다

전통적인 톰캣 WAR 배포 방식을 생각해보면
```
[톰캣 - 서블릿 컨테이너 1개]
  ├─ /app1 (쇼핑몰 애플리케이션)
  │    ├─ ProductServlet
  │    ├─ OrderServlet
  │    └─ UserServlet (싱글톤)
  │
  ├─ /app2 (관리자 페이지)
  │    ├─ AdminServlet
  │    └─ StatisticsServlet (싱글톤)
  │
  └─ /app3 (블로그 애플리케이션)
       ├─ PostServlet
       └─ CommentServlet (싱글톤)

# 톰캣 구조
$TOMCAT_HOME/
  ├─ bin/              (톰캣 실행 파일)
  ├─ conf/             (서버 설정)
  └─ webapps/          (여기에 여러 WAR 배포)
       ├─ app1.war
       ├─ app2.war
       └─ app3.war
```

각 애플리케이션은 독립적인 `ServletContext`를 가진다(애플리케이션 범위 컨텍스트)
그리고 서블릿은 자기 애플리케이션 범위 내에서만 싱글톤
쓰레드 풀은 톰캣 내부 애플리케이션이 모두 공유

그렇지만 스프링부트는 다르다!
```
[애플리케이션 1 실행]
  └─ 내장 톰캣 1
       └─ 이 앱만 실행

[애플리케이션 2 실행]  
  └─ 내장 톰캣 2
       └─ 이 앱만 실행
```

애플리케이션마다 `독립적인 서블릿 컨테이너`를 가짐
프로세스가 분리되어 있어 서로에게 영향이 없고
포트도 다르게 설정이 가능하다(8080, 8081, 8082...)

싱글톤
서버에 요청이 들어오면 서블릿 컨테이너가 스레드를 할당하여 요청을 처리한다
스레드마다 애플리케이션 코드를 복사할 수 없으니 싱글톤 패턴으로 사용하여 처리한다

```
[Method Area (메서드 영역) - JVM 시작 시 생성]
  ├─ UserController.class (클래스 정보, 바이트코드)
  │    └─ getUser() 메서드의 바이트코드
  ├─ UserService.class
  └─ UserRepository.class
  
[Heap (힙) - 객체 저장]
  ├─ UserController 인스턴스 (1개, 싱글톤)
  ├─ UserService 인스턴스 (1개, 싱글톤)
  └─ UserRepository 인스턴스 (1개, 싱글톤)

[Thread-1 Stack]              [Thread-2 Stack]              [Thread-3 Stack]
  └─ getUser() 실행 중          └─ getUser() 실행 중          └─ getUser() 실행 중
       ├─ userId=1                  ├─ userId=2                  ├─ userId=3
       └─ 지역 변수들                └─ 지역 변수들                └─ 지역 변수들
            ↓                            ↓                            ↓
       같은 메서드 코드 실행!       같은 메서드 코드 실행!       같은 메서드 코드 실행!

[요청 1 - Thread-1]
  → UserController 인스턴스(0x1234)의 getUser() 실행
  → id = 1
  → 출력: Thread: http-nio-8080-exec-1
  → 출력: Controller 인스턴스: UserController@1234

[요청 2 - Thread-2]  
  → UserController 인스턴스(0x1234)의 getUser() 실행  ← 같은 인스턴스!
  → id = 2
  → 출력: Thread: http-nio-8080-exec-2
  → 출력: Controller 인스턴스: UserController@1234  ← 같은 주소!

[요청 3 - Thread-3]
  → UserController 인스턴스(0x1234)의 getUser() 실행  ← 같은 인스턴스!
  → id = 3  
  → 출력: Thread: http-nio-8080-exec-3
  → 출력: Controller 인스턴스: UserController@1234  ← 같은 주소!
```

UserController 객체와 getUser() 메서드는 1개
하지만 3개의 스레드가 실행에 성공
