<blockquote>
<p>이번 시간에는 Spring이 객체를 관리하는 방법인 <strong>IoC와 DI</strong>, 반복되는 부가 기능을 분리하는 <strong>Proxy와 AOP</strong>, 안전한 API를 위한 <strong>입력값 검증</strong>, 시간이 오래 걸리는 작업을 분리하는 <strong>비동기 처리</strong>를 학습한다.</p>
</blockquote>
<hr />
<h1 id="1-lombok-및-properties-이해">1. Lombok 및 Properties 이해</h1>
<h2 id="11-lombok이란">1.1 Lombok이란?</h2>
<p>Lombok은 Java에서 반복적으로 작성하는 코드를 애너테이션으로 자동 생성해 주는 라이브러리다.</p>
<p>Java 클래스를 만들면 일반적으로 다음 코드를 반복해서 작성한다.</p>
<ul>
<li>Getter</li>
<li>Setter</li>
<li>생성자</li>
<li><code>toString()</code></li>
<li><code>equals()</code></li>
<li><code>hashCode()</code></li>
</ul>
<p>예를 들어 Lombok을 사용하지 않으면 다음과 같이 작성해야 한다.</p>
<pre><code class="language-java">public class User {

    private Long id;
    private String name;
    private String email;

    public User() {
    }

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}</code></pre>
<p>Lombok을 사용하면 다음처럼 간단해진다.</p>
<pre><code class="language-java">import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class User {

    private Long id;
    private String name;
    private String email;
}</code></pre>
<p>소스 코드에는 메서드가 보이지 않지만, 컴파일 과정에서 Lombok이 필요한 코드를 자동 생성한다.</p>
<pre><code class="language-text">작성한 소스 코드
→ Lombok이 코드 생성
→ Java 컴파일
→ class 파일 생성</code></pre>
<hr />
<h2 id="12-자주-사용하는-lombok-애너테이션">1.2 자주 사용하는 Lombok 애너테이션</h2>
<table>
<thead>
<tr>
<th>애너테이션</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>@Getter</code></td>
<td>Getter 자동 생성</td>
</tr>
<tr>
<td><code>@Setter</code></td>
<td>Setter 자동 생성</td>
</tr>
<tr>
<td><code>@NoArgsConstructor</code></td>
<td>매개변수가 없는 생성자 생성</td>
</tr>
<tr>
<td><code>@AllArgsConstructor</code></td>
<td>모든 필드를 받는 생성자 생성</td>
</tr>
<tr>
<td><code>@RequiredArgsConstructor</code></td>
<td><code>final</code> 필드를 받는 생성자 생성</td>
</tr>
<tr>
<td><code>@ToString</code></td>
<td><code>toString()</code> 생성</td>
</tr>
<tr>
<td><code>@EqualsAndHashCode</code></td>
<td><code>equals()</code>, <code>hashCode()</code> 생성</td>
</tr>
<tr>
<td><code>@Data</code></td>
<td>Getter, Setter 등 여러 기능을 한 번에 생성</td>
</tr>
<tr>
<td><code>@Builder</code></td>
<td>Builder 방식으로 객체 생성</td>
</tr>
<tr>
<td><code>@Slf4j</code></td>
<td>로그 객체 생성</td>
</tr>
</tbody></table>
<h3 id="getter-setter"><code>@Getter</code>, <code>@Setter</code></h3>
<pre><code class="language-java">@Getter
@Setter
public class User {

    private String name;
}</code></pre>
<p>다음 메서드가 자동으로 만들어진다.</p>
<pre><code class="language-java">public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}</code></pre>
<p>사용할 때는 일반 Java 메서드처럼 호출한다.</p>
<pre><code class="language-java">User user = new User();

user.setName(&quot;alice&quot;);

System.out.println(user.getName());</code></pre>
<hr />
<h2 id="13-requiredargsconstructor">1.3 <code>@RequiredArgsConstructor</code></h2>
<p>Spring에서 생성자 주입을 사용할 때 자주 활용한다.</p>
<p>기존 코드는 다음과 같다.</p>
<pre><code class="language-java">@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}</code></pre>
<p>Lombok을 사용하면 생성자를 생략할 수 있다.</p>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
}</code></pre>
<p><code>@RequiredArgsConstructor</code>가 <code>final</code> 필드를 받는 생성자를 자동으로 만든다.</p>
<p>컴파일 결과는 다음 코드와 비슷하다.</p>
<pre><code class="language-java">public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
}</code></pre>
<hr />
<h2 id="14-data-사용-시-주의점">1.4 <code>@Data</code> 사용 시 주의점</h2>
<p><code>@Data</code>는 편리하지만 많은 기능을 한 번에 생성한다.</p>
<pre><code class="language-java">@Data
public class User {

    private Long id;
    private String name;
    private String email;
}</code></pre>
<p>자동 생성되는 주요 기능은 다음과 같다.</p>
<ul>
<li>모든 필드의 Getter</li>
<li>모든 수정 가능한 필드의 Setter</li>
<li><code>toString()</code></li>
<li><code>equals()</code></li>
<li><code>hashCode()</code></li>
<li>필수 생성자</li>
</ul>
<p>하지만 Entity에 무조건 <code>@Data</code>를 사용하는 것은 주의해야 한다.</p>
<ul>
<li>모든 필드에 Setter가 생겨 의도하지 않은 변경이 가능하다.</li>
<li>연관관계가 있는 Entity에서 <code>toString()</code> 무한 호출 문제가 생길 수 있다.</li>
<li><code>equals()</code>와 <code>hashCode()</code>가 DB 식별자 처리와 충돌할 수 있다.</li>
</ul>
<p>필요한 기능만 선택하는 편이 안전하다.</p>
<pre><code class="language-java">@Getter
@NoArgsConstructor
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private String email;

    public void update(String name, String email) {
        this.name = name;
        this.email = email;
    }
}</code></pre>
<hr />
<h2 id="15-builder">1.5 <code>@Builder</code></h2>
<p>필드가 많은 객체를 읽기 쉽게 생성할 때 사용한다.</p>
<p>일반 생성자 방식은 필드의 의미를 바로 파악하기 어렵다.</p>
<pre><code class="language-java">User user = new User(null, &quot;alice&quot;, &quot;alice@example.com&quot;);</code></pre>
<p>Builder 방식은 값의 의미가 명확하다.</p>
<pre><code class="language-java">User user = User.builder()
        .name(&quot;alice&quot;)
        .email(&quot;alice@example.com&quot;)
        .build();</code></pre>
<p>일상적인 예로 보면 주문서의 항목을 하나씩 작성한 뒤 최종 제출하는 방식과 비슷하다.</p>
<hr />
<h2 id="16-slf4j">1.6 <code>@Slf4j</code></h2>
<p>프로그램의 실행 상태를 기록하는 로그 객체를 자동으로 만든다.</p>
<pre><code class="language-java">@Slf4j
@Service
public class UserService {

    public void createUser(String name) {
        log.info(&quot;사용자 생성 시작: {}&quot;, name);
    }
}</code></pre>
<p>다음과 같이 직접 로그 객체를 선언하지 않아도 된다.</p>
<pre><code class="language-java">private static final Logger log =
        LoggerFactory.getLogger(UserService.class);</code></pre>
<p>개발 중에는 <code>System.out.println()</code>보다 로그 사용이 권장된다.</p>
<pre><code class="language-java">log.debug(&quot;디버깅 정보&quot;);
log.info(&quot;일반 실행 정보&quot;);
log.warn(&quot;주의가 필요한 상황&quot;);
log.error(&quot;오류 발생&quot;, exception);</code></pre>
<hr />
<h2 id="17-properties란">1.7 Properties란?</h2>
<p>Properties는 프로그램에서 사용하는 설정값을 코드와 분리해 관리하는 방법이다.</p>
<p>Spring Boot에서는 주로 다음 파일을 사용한다.</p>
<pre><code class="language-text">src/main/resources/application.properties</code></pre>
<p>예시:</p>
<pre><code class="language-properties">spring.application.name=myapp
server.port=8080

spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.hibernate.ddl-auto=create-drop</code></pre>
<p>각 항목은 <code>키=값</code> 형식이다.</p>
<pre><code class="language-text">server.port = 8080
     키         값</code></pre>
<h3 id="설정을-코드와-분리하는-이유">설정을 코드와 분리하는 이유</h3>
<p>다음처럼 코드에 직접 값을 작성하면 변경하기 어렵다.</p>
<pre><code class="language-java">String serverName = &quot;production-server&quot;;
int timeout = 3000;</code></pre>
<p>설정 파일로 분리하면 코드 수정 없이 값을 바꿀 수 있다.</p>
<pre><code class="language-properties">app.server-name=production-server
app.timeout=3000</code></pre>
<hr />
<h2 id="18-value로-설정값-가져오기">1.8 <code>@Value</code>로 설정값 가져오기</h2>
<pre><code class="language-properties">app.title=SKALA 사용자 관리
app.max-users=100</code></pre>
<p>Java 코드에서는 다음과 같이 가져올 수 있다.</p>
<pre><code class="language-java">@Component
public class AppInfo {

    @Value(&quot;${app.title}&quot;)
    private String title;

    @Value(&quot;${app.max-users}&quot;)
    private int maxUsers;
}</code></pre>
<p>기본값도 지정할 수 있다.</p>
<pre><code class="language-java">@Value(&quot;${app.timeout:3000}&quot;)
private int timeout;</code></pre>
<p><code>app.timeout</code> 설정이 없으면 <code>3000</code>을 사용한다.</p>
<hr />
<h2 id="19-configurationproperties">1.9 <code>@ConfigurationProperties</code></h2>
<p>관련된 설정이 많다면 하나의 객체로 묶는 것이 좋다.</p>
<pre><code class="language-properties">app.name=SKALA
app.version=1.0
app.admin-email=admin@example.com</code></pre>
<pre><code class="language-java">@ConfigurationProperties(prefix = &quot;app&quot;)
public class AppProperties {

    private String name;
    private String version;
    private String adminEmail;

    // Getter, Setter
}</code></pre>
<p>Spring이 다음과 같이 값을 연결한다.</p>
<pre><code class="language-text">app.name        → name
app.version     → version
app.admin-email → adminEmail</code></pre>
<hr />
<h2 id="110-profile별-설정">1.10 Profile별 설정</h2>
<p>개발 환경과 운영 환경의 설정은 보통 다르다.</p>
<pre><code class="language-text">application.properties
application-dev.properties
application-prod.properties</code></pre>
<p>개발 환경:</p>
<pre><code class="language-properties"># application-dev.properties
server.port=8080
spring.datasource.url=jdbc:h2:mem:testdb</code></pre>
<p>운영 환경:</p>
<pre><code class="language-properties"># application-prod.properties
server.port=80
spring.datasource.url=jdbc:mysql://production-server/mydb</code></pre>
<p>실행할 Profile을 지정한다.</p>
<pre><code class="language-properties">spring.profiles.active=dev</code></pre>
<pre><code class="language-text">dev  → 개발 환경 설정
prod → 운영 환경 설정</code></pre>
<hr />
<h1 id="2-di-및-ioc-container">2. DI 및 IoC Container</h1>
<h2 id="21-객체의-의존관계">2.1 객체의 의존관계</h2>
<p>객체가 다른 객체를 사용해야 하는 관계를 의존관계라고 한다.</p>
<p>class A는 class B에 의존적이다? -&gt; A가 B의 메서드를 호출한다</p>
<p>예를 들어 <code>UserService</code>가 사용자 데이터를 저장하려면 <code>UserRepository</code>가 필요하다.</p>
<pre><code class="language-java">public class UserService {

    private UserRepository userRepository;
}</code></pre>
<pre><code class="language-text">UserService
    ↓ 사용
UserRepository</code></pre>
<p>이때 <code>UserService</code>는 <code>UserRepository</code>에 의존한다고 표현한다.</p>
<hr />
<h2 id="22-직접-객체를-생성하는-방식의-문제">2.2 직접 객체를 생성하는 방식의 문제</h2>
<p>다음 코드에서는 <code>UserService</code>가 필요한 Repository를 직접 만든다.</p>
<pre><code class="language-java">public class UserService {

    private UserRepository userRepository =
            new UserRepository();
}</code></pre>
<p>이 방식은 두 객체가 강하게 결합된다.</p>
<p>나중에 Repository를 변경하고 싶다면 Service 코드도 수정해야 한다.</p>
<pre><code class="language-java">private UserRepository userRepository =
        new DatabaseUserRepository();</code></pre>
<p>테스트용 Repository로 바꾸려고 해도 Service를 수정해야 한다.</p>
<pre><code class="language-java">private UserRepository userRepository =
        new FakeUserRepository();</code></pre>
<p>즉 객체를 사용하는 클래스와 객체를 만드는 코드가 섞여 있다.</p>
<hr />
<h2 id="23-di란">2.3 DI란?</h2>
<p>DI는 Dependency Injection의 약자로, 의존성 주입을 의미한다.</p>
<p>객체가 필요한 의존 객체를 직접 만들지 않고 외부에서 전달받는 방식이다.</p>
<pre><code class="language-java">public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}</code></pre>
<p><code>UserService</code>는 Repository를 어떻게 만드는지 알 필요가 없다.</p>
<pre><code class="language-text">기존 방식
UserService → 직접 UserRepository 생성

DI 방식
외부(Spring) → UserRepository 생성
             → UserService에 전달</code></pre>
<h3 id="일상-예시">일상 예시</h3>
<p>식당의 요리사는 채소를 직접 재배하지 않는다.</p>
<pre><code class="language-text">농장 또는 공급 업체 → 재료 준비
식당 관리자         → 재료 전달
요리사              → 받은 재료로 요리</code></pre>
<p>Spring에서는 다음처럼 대응된다.</p>
<pre><code class="language-text">IoC Container → 객체 생성 및 관리
DI             → 필요한 객체 전달
Service        → 전달받은 객체 사용</code></pre>
<hr />
<h2 id="24-ioc란">2.4 IoC란?</h2>
<p>IoC는 Inversion of Control의 약자로, 제어의 역전을 의미한다.</p>
<p>일반 Java에서는 개발자가 객체의 생성과 연결을 직접 제어한다.</p>
<pre><code class="language-java">UserRepository repository = new UserRepository();
UserService service = new UserService(repository);
UserController controller = new UserController(service);</code></pre>
<p>Spring에서는 Spring Container가 대신 처리한다.</p>
<pre><code class="language-text">Spring Container
├── UserRepository 생성
├── UserService 생성
│   └── UserRepository 주입
└── UserController 생성
    └── UserService 주입</code></pre>
<p>객체 생성의 제어권이 개발자 코드에서 Spring으로 넘어갔으므로 제어의 역전이라고 한다.</p>
<hr />
<h2 id="25-ioc-container란">2.5 IoC Container란?</h2>
<p>IoC Container는 Spring이 관리하는 객체 저장소이자 관리자다.</p>
<p>주요 역할은 다음과 같다.</p>
<ul>
<li>객체 생성</li>
<li>객체 보관</li>
<li>의존관계 연결</li>
<li>객체 생명주기 관리</li>
<li>설정 적용</li>
<li>필요한 객체 검색</li>
</ul>
<p>Spring Container가 관리하는 객체를 <strong>Bean</strong>이라고 한다.</p>
<pre><code class="language-text">Spring Bean
= Spring Container가 생성하고 관리하는 Java 객체</code></pre>
<hr />
<h2 id="26-bean-등록-애너테이션">2.6 Bean 등록 애너테이션</h2>
<h3 id="빈이란">빈이란?</h3>
<ul>
<li>스프링이 관리하는 객체를 의미</li>
<li><blockquote>
<p>ioc container가 bean을 생성하고 생명주기를 관리</p>
</blockquote>
</li>
<li>싱글턴 방식으로 관리<h3 id="component"><code>@Component</code></h3>
</li>
</ul>
<p>일반적인 Spring Bean을 등록한다.</p>
<pre><code class="language-java">@Component
public class EmailSender {
}</code></pre>
<h3 id="controller-restcontroller"><code>@Controller</code>, <code>@RestController</code></h3>
<p>HTTP 요청을 처리하는 Bean이다.</p>
<pre><code class="language-java">@RestController
public class UserController {
}</code></pre>
<h3 id="service"><code>@Service</code></h3>
<p>업무 로직을 담당하는 Bean이다.</p>
<pre><code class="language-java">@Service
public class UserService {
}</code></pre>
<h3 id="repository"><code>@Repository</code></h3>
<p>DB 접근을 담당하는 Bean이다.</p>
<pre><code class="language-java">@Repository
public class UserRepository {
}</code></pre>
<p>이 애너테이션들은 역할을 표현하지만 모두 Spring Bean으로 등록된다는 공통점이 있다.</p>
<hr />
<h2 id="27-component-scan">2.7 Component Scan</h2>
<p>Spring Boot는 시작 클래스가 있는 패키지부터 하위 패키지를 탐색한다.</p>
<pre><code class="language-java">package com.sk.skala.myapp;

@SpringBootApplication
public class MyappApplication {
}</code></pre>
<p>다음 구조라면 자동 탐색된다.</p>
<pre><code class="language-text">com.sk.skala.myapp
├── MyappApplication
├── controller
├── service
├── repository
└── domain</code></pre>
<p>하지만 다음처럼 시작 클래스의 상위 또는 다른 패키지에 있으면 자동 탐색되지 않을 수 있다.</p>
<pre><code class="language-text">com.other.service
└── UserService</code></pre>
<p>따라서 일반적으로 시작 클래스를 최상위 패키지에 둔다.</p>
<hr />
<h1 id="3-ioc-container-및-di-실습">3. IoC Container 및 DI 실습</h1>
<h2 id="31-생성자-주입">3.1 생성자 주입</h2>
<p>Spring에서 가장 권장되는 의존성 주입 방식이다.</p>
<pre><code class="language-java">@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}</code></pre>
<p>Spring이 실행되면 다음 순서로 동작한다.</p>
<pre><code class="language-text">1. UserRepository Bean 생성
2. UserService 생성자 확인
3. UserRepository가 필요하다는 사실 확인
4. 생성자에 UserRepository 전달
5. UserService Bean 생성 완료</code></pre>
<h3 id="lombok과-함께-사용하기">Lombok과 함께 사용하기</h3>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
}</code></pre>
<p><code>@RequiredArgsConstructor</code>가 생성자를 자동으로 만들어 준다.</p>
<hr />
<h2 id="32-필드-주입">3.2 필드 주입</h2>
<pre><code class="language-java">@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;
}</code></pre>
<p>코드가 짧지만 권장되지 않는다.</p>
<ul>
<li>의존관계가 겉으로 잘 드러나지 않는다.</li>
<li><code>final</code>을 사용할 수 없다.</li>
<li>순수 Java 단위 테스트가 어려워진다.</li>
<li>Spring 없이 객체를 생성하기 불편하다.</li>
</ul>
<hr />
<h2 id="33-setter-주입">3.3 Setter 주입</h2>
<pre><code class="language-java">@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(
            UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}</code></pre>
<p>객체 생성 후 의존성을 변경할 수 있다. 선택적이거나 변경 가능한 의존관계에 사용할 수 있지만, 필수 의존성에는 생성자 주입이 더 안전하다.</p>
<hr />
<h2 id="34-생성자-주입을-권장하는-이유">3.4 생성자 주입을 권장하는 이유</h2>
<pre><code class="language-java">private final UserRepository userRepository;</code></pre>
<p><code>final</code>을 사용하면 생성 후 다른 Repository로 변경할 수 없다.</p>
<p>또한 필요한 의존성이 없으면 객체를 만들 수 없다.</p>
<pre><code class="language-java">UserService service = new UserService(repository);</code></pre>
<p>잘못된 상태의 객체가 만들어지는 것을 예방할 수 있다.</p>
<hr />
<h2 id="35-인터페이스를-이용한-느슨한-결합">3.5 인터페이스를 이용한 느슨한 결합</h2>
<pre><code class="language-java">public interface MessageSender {

    void send(String message);
}</code></pre>
<p>이메일 구현체:</p>
<pre><code class="language-java">@Component
public class EmailMessageSender
        implements MessageSender {

    @Override
    public void send(String message) {
        System.out.println(&quot;이메일 전송: &quot; + message);
    }
}</code></pre>
<p>문자메시지 구현체:</p>
<pre><code class="language-java">@Component
public class SmsMessageSender
        implements MessageSender {

    @Override
    public void send(String message) {
        System.out.println(&quot;SMS 전송: &quot; + message);
    }
}</code></pre>
<p>서비스는 구체적인 구현체 대신 인터페이스에 의존한다.</p>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class NotificationService {

    private final MessageSender messageSender;

    public void notify(String message) {
        messageSender.send(message);
    }
}</code></pre>
<p>하지만 <code>MessageSender</code> 구현체가 두 개이면 Spring은 무엇을 넣어야 할지 결정할 수 없다.</p>
<pre><code class="language-text">MessageSender
├── EmailMessageSender
└── SmsMessageSender</code></pre>
<hr />
<h2 id="36-primary">3.6 <code>@Primary</code></h2>
<p>기본으로 사용할 Bean을 지정한다.</p>
<pre><code class="language-java">@Primary
@Component
public class EmailMessageSender
        implements MessageSender {
}</code></pre>
<p>이제 특별한 지정이 없으면 이메일 구현체가 주입된다.</p>
<hr />
<h2 id="37-qualifier">3.7 <code>@Qualifier</code></h2>
<p>특정 Bean을 직접 지정한다.</p>
<pre><code class="language-java">@Service
public class NotificationService {

    private final MessageSender messageSender;

    public NotificationService(
            @Qualifier(&quot;smsMessageSender&quot;)
            MessageSender messageSender) {

        this.messageSender = messageSender;
    }
}</code></pre>
<pre><code class="language-text">@Primary   → 기본 선택
@Qualifier → 이름으로 정확하게 선택</code></pre>
<hr />
<h2 id="38-bean과-configuration">3.8 <code>@Bean</code>과 <code>@Configuration</code></h2>
<p>직접 작성하지 않은 외부 라이브러리 객체는 클래스 위에 <code>@Component</code>를 붙일 수 없다. 이럴 때 설정 클래스에서 Bean을 등록한다.</p>
<pre><code class="language-java">@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}</code></pre>
<p><code>@Bean</code>이 붙은 메서드의 반환 객체를 Spring이 관리한다.</p>
<pre><code class="language-text">objectMapper() 실행
→ ObjectMapper 객체 생성
→ Spring Container에 Bean으로 등록</code></pre>
<hr />
<h1 id="4-proxy-pattern">4. Proxy Pattern</h1>
<h2 id="41-proxy란">4.1 Proxy란?</h2>
<p>Proxy는 대리인이라는 뜻이다.</p>
<p>실제 객체를 바로 호출하지 않고 중간의 대리 객체를 거쳐 호출하는 패턴이다.</p>
<pre><code class="language-text">사용자
→ Proxy
→ 실제 객체</code></pre>
<p>일상에서는 연예인에게 직접 일정을 요청하지 않고 매니저를 통해 요청하는 상황과 비슷하다.</p>
<pre><code class="language-text">요청자 → 매니저 → 연예인
          ↑
        Proxy</code></pre>
<p>매니저는 실제 업무 전후에 여러 작업을 처리할 수 있다.</p>
<ul>
<li>요청 권한 확인</li>
<li>일정 기록</li>
<li>요청 시간 측정</li>
<li>접근 제한</li>
<li>결과 전달</li>
</ul>
<hr />
<h2 id="42-proxy가-필요한-이유">4.2 Proxy가 필요한 이유</h2>
<p>핵심 기능과 부가 기능을 분리하기 위해 사용한다.</p>
<p>예를 들어 주문 기능의 핵심은 주문을 처리하는 것이다.</p>
<pre><code class="language-java">public void order() {
    System.out.println(&quot;주문 처리&quot;);
}</code></pre>
<p>실행 시간 측정까지 직접 추가하면 다음과 같다.</p>
<pre><code class="language-java">public void order() {
    long start = System.currentTimeMillis();

    System.out.println(&quot;주문 처리&quot;);

    long end = System.currentTimeMillis();
    System.out.println(&quot;실행 시간: &quot; + (end - start));
}</code></pre>
<p>모든 메서드에 시간 측정 코드를 넣으면 핵심 코드가 복잡해진다.</p>
<p>Proxy를 사용하면 부가 기능을 분리할 수 있다.</p>
<hr />
<h2 id="43-정적-proxy-예시">4.3 정적 Proxy 예시</h2>
<p>인터페이스:</p>
<pre><code class="language-java">public interface OrderService {

    void order();
}</code></pre>
<p>실제 객체:</p>
<pre><code class="language-java">public class OrderServiceImpl
        implements OrderService {

    @Override
    public void order() {
        System.out.println(&quot;주문 처리&quot;);
    }
}</code></pre>
<p>Proxy 객체:</p>
<pre><code class="language-java">public class OrderServiceProxy
        implements OrderService {

    private final OrderService target;

    public OrderServiceProxy(OrderService target) {
        this.target = target;
    }

    @Override
    public void order() {
        long start = System.currentTimeMillis();

        target.order();

        long end = System.currentTimeMillis();
        System.out.println(&quot;실행 시간: &quot; + (end - start));
    }
}</code></pre>
<p>실행:</p>
<pre><code class="language-java">OrderService target = new OrderServiceImpl();
OrderService proxy = new OrderServiceProxy(target);

proxy.order();</code></pre>
<p>처리 순서:</p>
<pre><code class="language-text">proxy.order()
→ 시작 시간 기록
→ 실제 target.order() 호출
→ 종료 시간 기록
→ 실행 시간 출력</code></pre>
<hr />
<h2 id="44-proxy의-활용">4.4 Proxy의 활용</h2>
<table>
<thead>
<tr>
<th>활용 분야</th>
<th>Proxy가 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td>보안</td>
<td>권한 확인 후 실제 메서드 호출</td>
</tr>
<tr>
<td>로깅</td>
<td>요청과 결과 기록</td>
</tr>
<tr>
<td>트랜잭션</td>
<td>시작, 커밋, 롤백</td>
</tr>
<tr>
<td>캐시</td>
<td>저장된 결과가 있으면 재사용</td>
</tr>
<tr>
<td>실행 시간 측정</td>
<td>메서드 전후 시간 측정</td>
</tr>
<tr>
<td>지연 로딩</td>
<td>실제 필요할 때 객체 조회</td>
</tr>
</tbody></table>
<p>Spring의 다음 기능에도 Proxy가 활용된다.</p>
<pre><code class="language-java">@Transactional
@Async
@Cacheable</code></pre>
<hr />
<h1 id="5-aop">5. AOP</h1>
<h2 id="51-aop란">5.1 AOP란?</h2>
<p>AOP는 Aspect-Oriented Programming의 약자로, 관점 지향 프로그래밍을 의미한다.</p>
<p>여러 클래스에 반복되는 공통 기능을 핵심 로직과 분리하는 방법이다.</p>
<p>예를 들어 여러 Service 메서드에 로그가 필요할 수 있다.</p>
<pre><code class="language-java">public void createUser() {
    System.out.println(&quot;시작&quot;);
    // 사용자 생성
    System.out.println(&quot;종료&quot;);
}

public void deleteUser() {
    System.out.println(&quot;시작&quot;);
    // 사용자 삭제
    System.out.println(&quot;종료&quot;);
}</code></pre>
<p>로그 코드가 모든 메서드에 반복된다.</p>
<p>AOP를 사용하면 다음처럼 분리한다.</p>
<pre><code class="language-text">핵심 기능
- 사용자 생성
- 사용자 조회
- 사용자 수정
- 사용자 삭제

공통 기능
- 로그
- 실행 시간
- 보안
- 트랜잭션</code></pre>
<hr />
<h2 id="52-aop-용어">5.2 AOP 용어</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Aspect</td>
<td>공통 기능을 모아 놓은 클래스</td>
</tr>
<tr>
<td>Advice</td>
<td>실제로 실행할 공통 기능</td>
</tr>
<tr>
<td>Join Point</td>
<td>공통 기능을 적용할 수 있는 지점</td>
</tr>
<tr>
<td>Pointcut</td>
<td>공통 기능을 적용할 대상을 선택하는 조건</td>
</tr>
<tr>
<td>Target</td>
<td>실제 핵심 기능을 가진 객체</td>
</tr>
<tr>
<td>Proxy</td>
<td>Target 앞에서 공통 기능을 실행하는 대리 객체</td>
</tr>
<tr>
<td>Weaving</td>
<td>핵심 기능과 공통 기능을 연결하는 과정</td>
</tr>
</tbody></table>
<hr />
<h2 id="53-advice-종류">5.3 Advice 종류</h2>
<table>
<thead>
<tr>
<th>애너테이션</th>
<th>실행 시점</th>
</tr>
</thead>
<tbody><tr>
<td><code>@Before</code></td>
<td>대상 메서드 실행 전</td>
</tr>
<tr>
<td><code>@After</code></td>
<td>대상 메서드 종료 후</td>
</tr>
<tr>
<td><code>@AfterReturning</code></td>
<td>정상 반환 후</td>
</tr>
<tr>
<td><code>@AfterThrowing</code></td>
<td>예외 발생 후</td>
</tr>
<tr>
<td><code>@Around</code></td>
<td>실행 전후 전체 제어</td>
</tr>
</tbody></table>
<hr />
<h2 id="54-실행-시간-측정-aop">5.4 실행 시간 측정 AOP</h2>
<pre><code class="language-java">@Aspect
@Component
public class LoggingAspect {

    @Around(&quot;execution(* com.sk.skala.myapp.service..*(..))&quot;)
    public Object measureTime(
            ProceedingJoinPoint joinPoint) throws Throwable {

        long start = System.currentTimeMillis();

        Object result = joinPoint.proceed();

        long end = System.currentTimeMillis();

        System.out.println(
                joinPoint.getSignature().getName()
                + &quot; 실행 시간: &quot;
                + (end - start)
                + &quot;ms&quot;
        );

        return result;
    }
}</code></pre>
<h3 id="joinpointproceed"><code>joinPoint.proceed()</code></h3>
<p>실제 대상 메서드를 실행한다.</p>
<pre><code class="language-text">AOP 시작
→ 시작 시간 기록
→ joinPoint.proceed()
→ 실제 Service 메서드 실행
→ 종료 시간 기록
→ 결과 반환</code></pre>
<p><code>proceed()</code>를 호출하지 않으면 실제 메서드가 실행되지 않는다.</p>
<hr />
<h2 id="55-pointcut-읽는-방법">5.5 Pointcut 읽는 방법</h2>
<pre><code class="language-java">execution(* com.sk.skala.myapp.service..*(..))</code></pre>
<p>의미는 다음과 같다.</p>
<pre><code class="language-text">execution  → 메서드 실행을 대상으로 함
*          → 반환형 제한 없음
service..  → service 패키지와 하위 패키지
*          → 모든 메서드 이름
(..)       → 매개변수 제한 없음</code></pre>
<p>즉 <code>service</code> 패키지의 모든 메서드에 적용한다.</p>
<hr />
<h2 id="56-aop와-proxy의-관계">5.6 AOP와 Proxy의 관계</h2>
<p>AOP는 공통 기능을 분리한다는 프로그래밍 개념이고, Proxy는 이를 구현하는 대표적인 방법이다.</p>
<pre><code class="language-text">AOP
└── 무엇을 분리할 것인가?
    └── 로그, 보안, 트랜잭션 등

Proxy
└── 분리한 기능을 어떻게 끼워 넣을 것인가?</code></pre>
<p>Spring AOP는 주로 Proxy 객체를 만들어 동작한다.</p>
<pre><code class="language-text">Controller
→ Spring이 만든 Proxy
   → AOP 공통 기능 실행
   → 실제 Service 실행</code></pre>
<hr />
<h2 id="57-spring-aop-사용-시-주의점">5.7 Spring AOP 사용 시 주의점</h2>
<p>같은 클래스 내부에서 자기 메서드를 직접 호출하면 Proxy를 거치지 않을 수 있다.</p>
<pre><code class="language-java">@Service
public class UserService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
        // 작업
    }
}</code></pre>
<p>외부에서 <code>methodA()</code>를 호출한 뒤 내부에서 <code>methodB()</code>를 호출하면, <code>this.methodB()</code> 형태의 내부 호출이므로 Proxy를 우회할 수 있다.</p>
<pre><code class="language-text">외부 호출 → Proxy → methodA()
                     ↓
                  methodB()
                  Proxy 우회 가능</code></pre>
<p>이것을 자기 호출 또는 Self Invocation 문제라고 한다.</p>
<hr />
<h1 id="6-입력값-검증">6. 입력값 검증</h1>
<h2 id="61-입력값-검증이-필요한-이유">6.1 입력값 검증이 필요한 이유</h2>
<p>클라이언트가 항상 정상적인 데이터를 보내는 것은 아니다.</p>
<pre><code class="language-json">{
  &quot;name&quot;: &quot;&quot;,
  &quot;email&quot;: &quot;잘못된 이메일&quot;
}</code></pre>
<p>검증 없이 저장하면 DB에 잘못된 데이터가 쌓인다.</p>
<p>입력값 검증은 다음 문제를 예방한다.</p>
<ul>
<li>필수값 누락</li>
<li>잘못된 이메일</li>
<li>너무 짧거나 긴 문자열</li>
<li>허용 범위를 벗어난 숫자</li>
<li>잘못된 형식</li>
<li>비정상 데이터 저장</li>
</ul>
<hr />
<h2 id="62-validation-의존성">6.2 Validation 의존성</h2>
<p>Maven 프로젝트에서는 일반적으로 다음 의존성을 사용한다.</p>
<pre><code class="language-xml">&lt;dependency&gt;
    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
    &lt;artifactId&gt;
        spring-boot-starter-validation
    &lt;/artifactId&gt;
&lt;/dependency&gt;</code></pre>
<hr />
<h2 id="63-주요-검증-애너테이션">6.3 주요 검증 애너테이션</h2>
<table>
<thead>
<tr>
<th>애너테이션</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>@NotNull</code></td>
<td><code>null</code> 금지</td>
</tr>
<tr>
<td><code>@NotEmpty</code></td>
<td><code>null</code>과 빈 문자열 금지</td>
</tr>
<tr>
<td><code>@NotBlank</code></td>
<td><code>null</code>, 빈 문자열, 공백 금지</td>
</tr>
<tr>
<td><code>@Size</code></td>
<td>문자열 또는 Collection 크기 제한</td>
</tr>
<tr>
<td><code>@Min</code></td>
<td>최솟값</td>
</tr>
<tr>
<td><code>@Max</code></td>
<td>최댓값</td>
</tr>
<tr>
<td><code>@Positive</code></td>
<td>양수만 허용</td>
</tr>
<tr>
<td><code>@PositiveOrZero</code></td>
<td>0 이상</td>
</tr>
<tr>
<td><code>@Email</code></td>
<td>이메일 형식 확인</td>
</tr>
<tr>
<td><code>@Pattern</code></td>
<td>정규표현식 확인</td>
</tr>
<tr>
<td><code>@Past</code></td>
<td>과거 날짜</td>
</tr>
<tr>
<td><code>@Future</code></td>
<td>미래 날짜</td>
</tr>
</tbody></table>
<h3 id="문자열-검증-차이">문자열 검증 차이</h3>
<table>
<thead>
<tr>
<th>값</th>
<th align="right"><code>@NotNull</code></th>
<th align="right"><code>@NotEmpty</code></th>
<th align="right"><code>@NotBlank</code></th>
</tr>
</thead>
<tbody><tr>
<td><code>null</code></td>
<td align="right">실패</td>
<td align="right">실패</td>
<td align="right">실패</td>
</tr>
<tr>
<td><code>&quot;&quot;</code></td>
<td align="right">성공</td>
<td align="right">실패</td>
<td align="right">실패</td>
</tr>
<tr>
<td><code>&quot;   &quot;</code></td>
<td align="right">성공</td>
<td align="right">성공</td>
<td align="right">실패</td>
</tr>
<tr>
<td><code>&quot;alice&quot;</code></td>
<td align="right">성공</td>
<td align="right">성공</td>
<td align="right">성공</td>
</tr>
</tbody></table>
<p>이름처럼 공백도 허용하면 안 되는 값에는 <code>@NotBlank</code>가 적절하다.</p>
<hr />
<h2 id="64-요청-dto-만들기">6.4 요청 DTO 만들기</h2>
<p>Entity를 API 입력값으로 바로 사용하는 것보다 요청 전용 DTO를 만드는 것이 좋다.</p>
<pre><code class="language-java">public class UserCreateRequest {

    @NotBlank(message = &quot;이름은 필수입니다.&quot;)
    @Size(min = 2, max = 20,
          message = &quot;이름은 2~20자여야 합니다.&quot;)
    private String name;

    @NotBlank(message = &quot;이메일은 필수입니다.&quot;)
    @Email(message = &quot;이메일 형식이 올바르지 않습니다.&quot;)
    private String email;

    // Getter, Setter
}</code></pre>
<p>DTO는 Data Transfer Object의 약자로 계층 간 전달에 사용하는 객체다.</p>
<pre><code class="language-text">클라이언트 JSON
→ UserCreateRequest DTO
→ 검증
→ User Entity 생성
→ DB 저장</code></pre>
<hr />
<h2 id="65-valid">6.5 <code>@Valid</code></h2>
<p>Controller에서 <code>@Valid</code>를 붙여야 검증이 실행된다.</p>
<pre><code class="language-java">@PostMapping(&quot;/users&quot;)
public User createUser(
        @Valid @RequestBody UserCreateRequest request) {

    return userService.createUser(request);
}</code></pre>
<p><code>@Valid</code>가 없으면 DTO에 검증 애너테이션을 붙여도 검증되지 않을 수 있다.</p>
<p>정상 요청:</p>
<pre><code class="language-json">{
  &quot;name&quot;: &quot;alice&quot;,
  &quot;email&quot;: &quot;alice@example.com&quot;
}</code></pre>
<p>잘못된 요청:</p>
<pre><code class="language-json">{
  &quot;name&quot;: &quot; &quot;,
  &quot;email&quot;: &quot;alice&quot;
}</code></pre>
<p>검증 실패 시 일반적으로 HTTP <code>400 Bad Request</code>가 반환된다.</p>
<hr />
<h2 id="66-검증-오류-처리">6.6 검증 오류 처리</h2>
<p><code>@RestControllerAdvice</code>를 사용하면 여러 Controller의 예외를 한곳에서 처리할 수 있다.</p>
<pre><code class="language-java">@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
            MethodArgumentNotValidException.class
    )
    public Map&lt;String, String&gt; handleValidation(
            MethodArgumentNotValidException exception) {

        Map&lt;String, String&gt; errors = new HashMap&lt;&gt;();

        exception.getBindingResult()
                .getFieldErrors()
                .forEach(error -&gt;
                        errors.put(
                                error.getField(),
                                error.getDefaultMessage()
                        )
                );

        return errors;
    }
}</code></pre>
<p>응답 예시:</p>
<pre><code class="language-json">{
  &quot;name&quot;: &quot;이름은 필수입니다.&quot;,
  &quot;email&quot;: &quot;이메일 형식이 올바르지 않습니다.&quot;
}</code></pre>
<hr />
<h2 id="67-검증과-예외-처리는-어디에서-할까">6.7 검증과 예외 처리는 어디에서 할까?</h2>
<table>
<thead>
<tr>
<th>종류</th>
<th>적절한 위치</th>
</tr>
</thead>
<tbody><tr>
<td>빈 값, 길이, 이메일 형식</td>
<td>DTO의 Validation</td>
</tr>
<tr>
<td>사용자가 존재하는지 확인</td>
<td>Service</td>
</tr>
<tr>
<td>이메일 중복 확인</td>
<td>Service 및 Repository</td>
</tr>
<tr>
<td>DB 제약조건 위반</td>
<td>DB와 예외 처리 계층</td>
</tr>
<tr>
<td>HTTP 오류 응답 구성</td>
<td>Controller Advice</td>
</tr>
</tbody></table>
<p>예를 들어 이메일 형식은 DTO가 검사하고, 이미 가입된 이메일인지는 Service가 검사한다.</p>
<pre><code class="language-java">if (userRepository.existsByEmail(request.getEmail())) {
    throw new IllegalArgumentException(
            &quot;이미 사용 중인 이메일입니다.&quot;
    );
}</code></pre>
<hr />
<h1 id="7-async">7. Async</h1>
<h2 id="71-동기-처리란">7.1 동기 처리란?</h2>
<p>동기 방식에서는 한 작업이 끝날 때까지 다음 작업이 기다린다.</p>
<pre><code class="language-text">요청 시작
→ 회원 저장
→ 이메일 전송 시작
→ 이메일 전송 완료까지 대기
→ 응답</code></pre>
<p>회원 저장에 0.1초, 이메일 전송에 3초가 걸리면 사용자는 약 3.1초 동안 기다려야 한다.</p>
<pre><code class="language-java">public void register() {
    saveUser();
    sendEmail();
}</code></pre>
<hr />
<h2 id="72-비동기-처리란">7.2 비동기 처리란?</h2>
<p>비동기 방식에서는 시간이 오래 걸리는 작업을 별도의 흐름에 맡기고 현재 작업을 계속 진행한다.</p>
<pre><code class="language-text">요청 시작
→ 회원 저장
→ 이메일 전송 작업 요청
→ 즉시 HTTP 응답
       ↓
  별도 Thread가 이메일 전송</code></pre>
<p>일상 예시로는 세탁기를 작동시킨 후 세탁이 끝날 때까지 앞에서 기다리지 않고 다른 일을 하는 것과 비슷하다.</p>
<hr />
<h2 id="73-spring에서-async-활성화">7.3 Spring에서 Async 활성화</h2>
<p>시작 클래스나 설정 클래스에 <code>@EnableAsync</code>를 추가한다.</p>
<pre><code class="language-java">@EnableAsync
@SpringBootApplication
public class MyappApplication {

    public static void main(String[] args) {
        SpringApplication.run(
                MyappApplication.class,
                args
        );
    }
}</code></pre>
<hr />
<h2 id="74-async-사용">7.4 <code>@Async</code> 사용</h2>
<pre><code class="language-java">@Service
public class EmailService {

    @Async
    public void sendEmail(String email) {
        System.out.println(
                &quot;이메일 전송 시작: &quot;
                + Thread.currentThread().getName()
        );

        // 시간이 오래 걸리는 작업
    }
}</code></pre>
<p>호출하는 Service:</p>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class UserService {

    private final EmailService emailService;

    public User createUser(User user) {
        User savedUser = userRepository.save(user);

        emailService.sendEmail(savedUser.getEmail());

        return savedUser;
    }
}</code></pre>
<p>사용자 생성 요청은 이메일 발송 완료를 기다리지 않고 응답할 수 있다.</p>
<hr />
<h2 id="75-thread란">7.5 Thread란?</h2>
<p>Thread는 프로그램 안에서 작업이 실행되는 흐름이다.</p>
<p>동기 방식:</p>
<pre><code class="language-text">main Thread
├── 사용자 저장
├── 이메일 전송
└── 응답</code></pre>
<p>비동기 방식:</p>
<pre><code class="language-text">main Thread
├── 사용자 저장
├── 이메일 작업 전달
└── 응답

async Thread
└── 이메일 전송</code></pre>
<hr />
<h2 id="76-비동기-반환값">7.6 비동기 반환값</h2>
<p>반환값이 없다면 <code>void</code>를 사용할 수 있다.</p>
<pre><code class="language-java">@Async
public void sendEmail() {
}</code></pre>
<p>결과를 나중에 받고 싶다면 <code>CompletableFuture</code>를 사용할 수 있다.</p>
<pre><code class="language-java">@Async
public CompletableFuture&lt;String&gt; process() {
    String result = &quot;처리 완료&quot;;

    return CompletableFuture.completedFuture(result);
}</code></pre>
<p>사용 예시:</p>
<pre><code class="language-java">CompletableFuture&lt;String&gt; future =
        asyncService.process();

future.thenAccept(result -&gt;
        System.out.println(result)
);</code></pre>
<hr />
<h2 id="77-thread-pool">7.7 Thread Pool</h2>
<p>요청마다 Thread를 무제한 생성하면 서버 자원이 부족해질 수 있다.</p>
<p>그래서 미리 준비된 일정 수의 Thread를 재사용하는 Thread Pool을 사용한다.</p>
<pre><code class="language-java">@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = &quot;taskExecutor&quot;)
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor =
                new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix(&quot;async-&quot;);
        executor.initialize();

        return executor;
    }
}</code></pre>
<table>
<thead>
<tr>
<th>설정</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>corePoolSize</code></td>
<td>기본으로 유지할 Thread 수</td>
</tr>
<tr>
<td><code>maxPoolSize</code></td>
<td>최대 Thread 수</td>
</tr>
<tr>
<td><code>queueCapacity</code></td>
<td>대기 가능한 작업 수</td>
</tr>
<tr>
<td><code>threadNamePrefix</code></td>
<td>Thread 이름 앞부분</td>
</tr>
</tbody></table>
<hr />
<h2 id="78-async-사용-시-주의점">7.8 Async 사용 시 주의점</h2>
<h3 id="같은-클래스-내부-호출-문제">같은 클래스 내부 호출 문제</h3>
<p><code>@Async</code>도 Spring Proxy를 사용한다.</p>
<pre><code class="language-java">@Service
public class EmailService {

    public void start() {
        sendEmail();
    }

    @Async
    public void sendEmail() {
    }
}</code></pre>
<p>같은 객체 안에서 직접 <code>sendEmail()</code>을 호출하면 Proxy를 거치지 않아 비동기로 실행되지 않을 수 있다.</p>
<p>다른 Bean으로 분리한 후 외부에서 호출하는 것이 좋다.</p>
<pre><code class="language-text">UserService
→ EmailService Proxy
→ 비동기 sendEmail()</code></pre>
<h3 id="예외-처리">예외 처리</h3>
<p><code>void</code> 비동기 메서드에서 발생한 예외는 호출한 쪽에서 일반적인 <code>try-catch</code>로 받기 어렵다.</p>
<pre><code class="language-java">try {
    emailService.sendEmail();
} catch (Exception e) {
    // 비동기 내부 예외를 잡지 못할 수 있음
}</code></pre>
<p>비동기 메서드 내부에서 로그와 예외 처리를 꼼꼼하게 해야 한다.</p>
<h3 id="모든-작업을-비동기로-만들면-안-됨">모든 작업을 비동기로 만들면 안 됨</h3>
<p>다음 작업은 결과가 반드시 필요하므로 무작정 비동기로 처리하면 안 된다.</p>
<ul>
<li>결제 승인</li>
<li>재고 차감 결과 확인</li>
<li>사용자 인증</li>
<li>핵심 DB 저장</li>
<li>다음 작업이 결과에 의존하는 경우</li>
</ul>
<p>비동기에 적합한 작업은 다음과 같다.</p>
<ul>
<li>이메일 발송</li>
<li>문자 발송</li>
<li>알림 전송</li>
<li>로그 처리</li>
<li>대용량 파일 처리</li>
<li>통계 데이터 생성</li>
</ul>
<hr />
<h1 id="8-전체-개념-연결하기">8. 전체 개념 연결하기</h1>
<p>이번 과정의 개념들은 각각 독립된 내용처럼 보이지만 실제 Spring 프로그램에서는 서로 연결된다.</p>
<pre><code class="language-text">Properties
→ 외부 설정값 관리

IoC Container
→ Spring Bean 생성 및 보관

DI
→ Bean 사이의 의존관계 연결

Proxy
→ 실제 객체 대신 요청을 먼저 받음

AOP
→ Proxy를 활용해 로그·보안 등 공통 기능 적용

Validation
→ Controller에 들어온 값 검사

Async
→ Proxy를 활용해 별도 Thread에서 작업 실행</code></pre>
<p>사용자 등록 API를 예로 들면 다음과 같이 동작한다.</p>
<pre><code class="language-text">POST /api/users
→ Controller가 JSON 수신
→ Validation으로 입력값 검사
→ DI 받은 UserService 호출
→ AOP Proxy가 로그 및 실행 시간 기록
→ Repository가 사용자 저장
→ Async Proxy가 이메일 전송을 별도 Thread에 전달
→ 사용자 정보를 JSON으로 응답</code></pre>
<hr />
<h1 id="9-핵심-요약">9. 핵심 요약</h1>
<table>
<thead>
<tr>
<th>개념</th>
<th>한 줄 설명</th>
<th>일상 예시</th>
</tr>
</thead>
<tbody><tr>
<td>Lombok</td>
<td>반복 Java 코드를 자동 생성</td>
<td>반복 서류 자동 작성</td>
</tr>
<tr>
<td>Properties</td>
<td>환경 설정을 코드 밖에서 관리</td>
<td>기기의 설정 메뉴</td>
</tr>
<tr>
<td>DI</td>
<td>필요한 객체를 외부에서 전달</td>
<td>요리사에게 재료 공급</td>
</tr>
<tr>
<td>IoC</td>
<td>객체 관리 권한을 Spring에 맡김</td>
<td>직원 배치를 관리자가 담당</td>
</tr>
<tr>
<td>IoC Container</td>
<td>Bean을 생성하고 연결하는 공간</td>
<td>회사의 인사관리부</td>
</tr>
<tr>
<td>Proxy</td>
<td>실제 객체 앞에서 요청을 대신 처리</td>
<td>연예인의 매니저</td>
</tr>
<tr>
<td>AOP</td>
<td>공통 기능을 핵심 기능과 분리</td>
<td>모든 출입구의 보안 검사</td>
</tr>
<tr>
<td>Validation</td>
<td>입력값이 규칙에 맞는지 확인</td>
<td>신청서 접수 전 항목 검사</td>
</tr>
<tr>
<td>Async</td>
<td>작업을 별도 흐름에서 실행</td>
<td>세탁기를 돌리고 다른 일 수행</td>
</tr>
</tbody></table>
<p>가장 중요한 연결 관계는 다음과 같다.</p>
<pre><code class="language-text">Spring Container가 객체를 만든다.        → IoC
필요한 객체를 서로 연결한다.             → DI
객체 앞에 대리 객체를 만든다.             → Proxy
대리 객체로 공통 기능을 적용한다.         → AOP
대리 객체로 별도 Thread 실행을 지원한다.  → Async
입력 데이터는 사용 전에 검사한다.         → Validation
반복 코드는 자동 생성한다.                → Lombok
환경별 값은 코드 밖에서 관리한다.         → Properties</code></pre>