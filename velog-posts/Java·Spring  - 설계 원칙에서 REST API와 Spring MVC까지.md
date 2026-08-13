<blockquote>
<p>학습 범위: 생성 패턴 및 SOLID → Network 및 Socket 통신 → REST API 및 Spring 개요 → Spring Boot 프로젝트와 MVC → Controller와 HTTP 매개변수 → Service와 Repository → Spring MVC 내부 구조</p>
<p>학습 목표: 좋은 객체를 설계하고, 네트워크 요청이 Spring Controller·Service·Repository를 거쳐 응답으로 돌아오는 전체 흐름을 이해한다.</p>
</blockquote>
<hr />
<h2 id="생성-패턴creational-patterns">생성 패턴(Creational Patterns)</h2>
<p>생성 패턴은 객체를 무조건 <code>new</code>로 직접 만드는 대신, <strong>객체를 만드는 책임을 분리하고 관리하는 방법</strong>이다. 객체 생성 과정이 복잡하거나, 생성 방식을 쉽게 바꿔야 할 때 유용하다.</p>
<h3 id="1-singleton-객체를-하나만-만들기">1. Singleton: 객체를 하나만 만들기</h3>
<p>Singleton은 프로그램 전체에서 인스턴스를 하나만 생성해 공유하는 패턴이다.</p>
<pre><code class="language-java">public class AppConfig {
    private static final AppConfig INSTANCE = new AppConfig();

    private AppConfig() { }

    public static AppConfig getInstance() {
        return INSTANCE;
    }
}</code></pre>
<pre><code class="language-java">AppConfig config1 = AppConfig.getInstance();
AppConfig config2 = AppConfig.getInstance();

System.out.println(config1 == config2); // true</code></pre>
<p>일상적으로는 모든 직원이 한 회사의 공용 안내판을 함께 사용하는 모습과 비슷하다. 설정, 로깅처럼 공통 자원을 관리할 때 활용할 수 있지만, 전역 상태가 많아지면 테스트와 변경이 어려워진다.</p>
<p>Spring은 기본적으로 Bean을 Singleton으로 관리하므로 개발자가 Singleton 코드를 직접 작성할 일은 많지 않다.</p>
<h3 id="2-factory-method-객체-생성을-공장에-맡기기">2. Factory Method: 객체 생성을 공장에 맡기기</h3>
<p>Factory Method는 사용할 객체를 직접 <code>new</code>하지 않고, 생성 전용 메서드에 맡긴다.</p>
<pre><code class="language-java">interface Payment {
    void pay(int amount);
}

class CardPayment implements Payment {
    public void pay(int amount) {
        System.out.println(amount + &quot;원 카드 결제&quot;);
    }
}

class KakaoPayment implements Payment {
    public void pay(int amount) {
        System.out.println(amount + &quot;원 카카오 결제&quot;);
    }
}

class PaymentFactory {
    static Payment create(String type) {
        return switch (type) {
            case &quot;card&quot; -&gt; new CardPayment();
            case &quot;kakao&quot; -&gt; new KakaoPayment();
            default -&gt; throw new IllegalArgumentException(&quot;지원하지 않는 결제 방식&quot;);
        };
    }
}</code></pre>
<pre><code class="language-java">Payment payment = PaymentFactory.create(&quot;card&quot;);
payment.pay(10_000);</code></pre>
<p>사용하는 쪽은 어떤 구체 클래스가 만들어지는지 몰라도 <code>Payment</code> 계약만 알고 사용할 수 있다.</p>
<h3 id="3-abstract-factory-관련-객체-묶음을-함께-만들기">3. Abstract Factory: 관련 객체 묶음을 함께 만들기</h3>
<p>Abstract Factory는 서로 관련된 여러 객체를 한 묶음으로 생성한다. 예를 들어 Windows용 버튼·체크박스와 Mac용 버튼·체크박스를 각각 일관되게 만드는 방식이다.</p>
<pre><code class="language-java">interface UIFactory {
    Button createButton();
    CheckBox createCheckBox();
}</code></pre>
<p>제품군 전체를 교체해야 할 때 편하지만 인터페이스와 클래스가 늘어 구조가 복잡해질 수 있다.</p>
<h3 id="4-builder-복잡한-객체를-단계적으로-만들기">4. Builder: 복잡한 객체를 단계적으로 만들기</h3>
<p>필드가 많은 객체를 생성자 하나로 만들면 인자의 의미를 알기 어렵다.</p>
<pre><code class="language-java">User user = new User(&quot;박성우&quot;, 26, &quot;서울&quot;, true);</code></pre>
<p>Builder를 사용하면 생성 과정이 읽기 쉬워진다.</p>
<pre><code class="language-java">User user = User.builder()
        .name(&quot;박성우&quot;)
        .age(26)
        .city(&quot;서울&quot;)
        .active(true)
        .build();</code></pre>
<p>Builder는 선택 항목이 많거나 불변 객체를 만들 때 유용하다.</p>
<table>
<thead>
<tr>
<th>패턴</th>
<th>핵심 목적</th>
<th>일상 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Singleton</td>
<td>객체 하나를 공유</td>
<td>회사 공용 안내판</td>
</tr>
<tr>
<td>Factory Method</td>
<td>객체 생성을 전용 메서드에 위임</td>
<td>메뉴에 따라 제품을 만드는 공장</td>
</tr>
<tr>
<td>Abstract Factory</td>
<td>관련 객체들을 한 묶음으로 생성</td>
<td>같은 브랜드의 가구 세트</td>
</tr>
<tr>
<td>Builder</td>
<td>복잡한 객체를 단계적으로 조립</td>
<td>샌드위치 옵션 선택</td>
</tr>
</tbody></table>
<hr />
<h2 id="solid-설계-원칙">SOLID 설계 원칙</h2>
<p>SOLID는 유지보수하기 쉽고 확장에 유연한 객체지향 프로그램을 만들기 위한 다섯 가지 원칙이다. 목표는 <strong>높은 응집도와 낮은 결합도</strong>다.</p>
<h3 id="s---단일-책임-원칙srp">S - 단일 책임 원칙(SRP)</h3>
<blockquote>
<p>클래스는 하나의 책임, 즉 하나의 변경 이유만 가져야 한다.</p>
</blockquote>
<pre><code class="language-java">class UserService {
    void createUser() { }
    void sendEmail() { }
    void saveFile() { }
}</code></pre>
<p>한 클래스가 사용자 처리, 이메일, 파일 저장을 모두 담당하면 책임이 섞여 있다. 다음처럼 분리하는 편이 좋다.</p>
<pre><code class="language-java">class UserService { void createUser() { } }
class EmailService { void sendEmail() { } }
class FileRepository { void saveFile() { } }</code></pre>
<p>일상 비유로는 한 직원에게 주문, 요리, 결제, 청소를 모두 맡기지 않고 역할을 나누는 것이다.</p>
<h3 id="o---개방·폐쇄-원칙ocp">O - 개방·폐쇄 원칙(OCP)</h3>
<blockquote>
<p>확장에는 열려 있고, 기존 코드 수정에는 닫혀 있어야 한다.</p>
</blockquote>
<pre><code class="language-java">interface Payment {
    void pay(int amount);
}

class PaymentService {
    void processPayment(Payment payment, int amount) {
        payment.pay(amount);
    }
}</code></pre>
<p>새로운 결제 수단은 <code>Payment</code> 구현체로 추가하면 된다. <code>PaymentService</code>의 기존 코드는 바꿀 필요가 없다.</p>
<h3 id="l---리스코프-치환-원칙lsp">L - 리스코프 치환 원칙(LSP)</h3>
<blockquote>
<p>자식 객체는 부모 타입을 사용하는 자리에 넣어도 정상 동작해야 한다.</p>
</blockquote>
<pre><code class="language-java">static void printArea(Shape shape) {
    System.out.println(shape.getArea());
}</code></pre>
<p><code>Circle</code>, <code>Rectangle</code> 중 무엇을 전달해도 <code>Shape</code>가 약속한 <code>getArea()</code> 규칙을 지켜야 한다. 자식이 부모의 약속을 깨거나 예상하지 못한 예외를 던진다면 잘못된 상속일 수 있다.</p>
<h3 id="i---인터페이스-분리-원칙isp">I - 인터페이스 분리 원칙(ISP)</h3>
<blockquote>
<p>사용하지 않는 기능에 의존하도록 강요하지 않는다.</p>
</blockquote>
<pre><code class="language-java">interface Printer { void print(); }
interface Copier { void copy(); }
interface Fax { void fax(); }</code></pre>
<p>단순 프린터는 <code>Printer</code>만 구현하고, 복합기는 필요한 인터페이스를 모두 구현하면 된다. 하나의 거대한 인터페이스보다 역할별로 작은 인터페이스를 만든다.</p>
<h3 id="d---의존-역전-원칙dip">D - 의존 역전 원칙(DIP)</h3>
<blockquote>
<p>구체 클래스가 아니라 인터페이스 같은 추상화에 의존한다.</p>
</blockquote>
<pre><code class="language-java">interface UserRepository {
    void save(User user);
}

class UserService {
    private final UserRepository repository;

    UserService(UserRepository repository) {
        this.repository = repository;
    }
}</code></pre>
<p><code>UserService</code>는 MySQL이나 파일 저장 방식에 직접 의존하지 않는다. <code>UserRepository</code> 구현체를 바꿔 끼울 수 있다. 이 구조가 Spring의 의존성 주입(DI)과 자연스럽게 연결된다.</p>
<hr />
<h2 id="network-기초-및-socket-통신">Network 기초 및 Socket 통신</h2>
<h3 id="1-네트워크-계층을-가볍게-이해하기">1. 네트워크 계층을 가볍게 이해하기</h3>
<p>네트워크 통신은 여러 계층이 역할을 나누어 처리한다.</p>
<pre><code class="language-text">애플리케이션: HTTP, JSON 등 서비스 규칙
전송 계층: TCP/UDP, Port로 프로세스 구분
인터넷 계층: IP로 컴퓨터를 찾아감
네트워크 접근 계층: 실제 장비와 신호로 전달</code></pre>
<ul>
<li><strong>IP</strong>: 네트워크에서 컴퓨터를 찾는 주소</li>
<li><strong>Port</strong>: 컴퓨터 안에서 프로그램을 찾는 번호</li>
<li><strong>Protocol</strong>: 통신 참가자가 지키는 약속</li>
<li><strong>TCP</strong>: 연결을 맺고 데이터의 순서와 전달을 보장</li>
<li><strong>UDP</strong>: 전달 보장보다 속도와 단순성을 우선</li>
</ul>
<p><code>localhost:8080</code>은 내 컴퓨터의 8080번 프로그램에 접속한다는 뜻이다.</p>
<h3 id="2-socket이란">2. Socket이란?</h3>
<p>Socket은 프로그램이 네트워크를 통해 데이터를 주고받는 <strong>통신 출입구(Endpoint)</strong> 다.</p>
<p>전화에 비유하면:</p>
<ul>
<li>IP: 상대방의 지역·주소</li>
<li>Port: 담당 부서의 내선번호</li>
<li>Socket: 실제 통화가 연결된 전화기</li>
<li>데이터 스트림: 전화로 주고받는 대화</li>
</ul>
<h3 id="3-java-tcp-서버">3. Java TCP 서버</h3>
<pre><code class="language-java">try (ServerSocket serverSocket = new ServerSocket(8080)) {
    System.out.println(&quot;서버 대기 중&quot;);

    try (Socket socket = serverSocket.accept();
         BufferedReader reader = new BufferedReader(
                 new InputStreamReader(socket.getInputStream()));
         PrintWriter writer = new PrintWriter(
                 socket.getOutputStream(), true)) {

        String message = reader.readLine();
        System.out.println(&quot;받은 메시지: &quot; + message);
        writer.println(&quot;서버 응답: &quot; + message);
    }
}</code></pre>
<ol>
<li><code>new ServerSocket(8080)</code>: 8080번 포트를 열고 서버를 준비한다.</li>
<li><code>accept()</code>: 클라이언트 접속이 올 때까지 기다린다.</li>
<li>연결되면 통신용 <code>Socket</code>이 만들어진다.</li>
<li>입력 스트림으로 요청을 읽고 출력 스트림으로 응답한다.</li>
<li>사용이 끝난 Socket을 닫는다.</li>
</ol>
<h3 id="4-java-tcp-클라이언트">4. Java TCP 클라이언트</h3>
<pre><code class="language-java">try (Socket socket = new Socket(&quot;localhost&quot;, 8080);
     BufferedReader reader = new BufferedReader(
             new InputStreamReader(socket.getInputStream()));
     PrintWriter writer = new PrintWriter(
             socket.getOutputStream(), true)) {

    writer.println(&quot;Hello Server!&quot;);
    System.out.println(reader.readLine());
}</code></pre>
<p>클라이언트가 서버의 IP와 Port로 연결한 뒤 메시지를 보내고 응답을 읽는다.</p>
<h3 id="5-socket과-http의-관계">5. Socket과 HTTP의 관계</h3>
<p>HTTP는 Socket 위에서 주고받는 <strong>메시지 형식과 처리 규칙</strong>이다.</p>
<pre><code class="language-http">GET /users HTTP/1.1
Host: localhost:8080
Connection: close</code></pre>
<pre><code class="language-http">HTTP/1.1 200 OK
Content-Type: application/json

[{&quot;id&quot;:1,&quot;name&quot;:&quot;민지&quot;}]</code></pre>
<p>Tomcat은 내부적으로 TCP 연결을 받고 HTTP 메시지를 해석한다. 개발자는 Spring MVC 덕분에 Socket과 HTTP 파싱을 직접 구현하지 않고 Controller 메서드에 집중할 수 있다.</p>
<hr />
<h2 id="rest-api">REST API</h2>
<h3 id="1-웹-애플리케이션의-구조">1. 웹 애플리케이션의 구조</h3>
<p>웹 애플리케이션은 클라이언트가 HTTP로 요청하고 서버가 처리 결과를 응답하는 프로그램이다.</p>
<ul>
<li><strong>Web Server</strong>: HTML, CSS, JavaScript, 이미지 같은 정적 파일 제공. 예: Nginx</li>
<li><strong>WAS</strong>: Java 코드와 비즈니스 로직을 실행해 동적 응답 생성. 예: Tomcat</li>
<li><strong>Frontend</strong>: 화면과 사용자 상호작용 담당</li>
<li><strong>Backend</strong>: 데이터 처리, 업무 규칙, DB 접근 담당</li>
</ul>
<p>Spring Boot Web 애플리케이션은 내장 Tomcat을 이용해 HTTP 요청을 받는다.</p>
<h3 id="2-rest란">2. REST란?</h3>
<p>REST는 웹의 자원(Resource)을 URI로 식별하고, HTTP Method로 행위를 표현하는 아키텍처 스타일이다.</p>
<pre><code class="language-text">자원: User
식별자: /users/10
행위: GET
표현: JSON</code></pre>
<pre><code class="language-http">GET /users/10</code></pre>
<p>위 요청은 “10번 사용자를 조회하라”는 뜻이다.</p>
<h3 id="3-uri와-http-method">3. URI와 HTTP Method</h3>
<p>URI에는 동작보다 명사를 쓰고, 동작은 HTTP Method로 표현한다.</p>
<table>
<thead>
<tr>
<th>기능</th>
<th>HTTP Method</th>
<th>URI</th>
</tr>
</thead>
<tbody><tr>
<td>사용자 목록 조회</td>
<td><code>GET</code></td>
<td><code>/users</code></td>
</tr>
<tr>
<td>사용자 한 명 조회</td>
<td><code>GET</code></td>
<td><code>/users/10</code></td>
</tr>
<tr>
<td>사용자 생성</td>
<td><code>POST</code></td>
<td><code>/users</code></td>
</tr>
<tr>
<td>사용자 전체 수정</td>
<td><code>PUT</code></td>
<td><code>/users/10</code></td>
</tr>
<tr>
<td>사용자 일부 수정</td>
<td><code>PATCH</code></td>
<td><code>/users/10</code></td>
</tr>
<tr>
<td>사용자 삭제</td>
<td><code>DELETE</code></td>
<td><code>/users/10</code></td>
</tr>
</tbody></table>
<pre><code class="language-text">좋은 예: GET /users/10
피할 예: GET /getUser/10</code></pre>
<h3 id="4-자원의-표현representation">4. 자원의 표현(Representation)</h3>
<p>서버의 Java <code>User</code> 객체 자체가 네트워크로 이동하는 것이 아니다. 클라이언트가 이해할 수 있는 JSON 같은 표현으로 변환되어 전달된다.</p>
<pre><code class="language-json">{
  &quot;id&quot;: 10,
  &quot;name&quot;: &quot;박성우&quot;,
  &quot;email&quot;: &quot;park@example.com&quot;
}</code></pre>
<p>Spring에서는 Jackson과 <code>HttpMessageConverter</code>가 Java 객체와 JSON 사이 변환을 담당한다.</p>
<h3 id="5-rest의-핵심-원칙">5. REST의 핵심 원칙</h3>
<ul>
<li><strong>Client-Server 분리</strong>: 화면과 데이터 처리의 책임을 나눈다.</li>
<li><strong>Stateless</strong>: 서버가 이전 요청 상태에 의존하지 않는다. 각 요청에 필요한 정보를 담는다.</li>
<li><strong>Cacheable</strong>: 응답의 캐시 가능 여부를 명확히 한다.</li>
<li><strong>Uniform Interface</strong>: URI, HTTP Method, 응답 형식을 일관되게 사용한다.</li>
<li><strong>Layered System</strong>: Gateway, Proxy, Load Balancer 같은 중간 계층을 둘 수 있다.</li>
<li><strong>Code on Demand(선택)</strong>: 서버가 실행 가능한 코드를 내려줄 수 있다.</li>
</ul>
<hr />
<h2 id="spring과-spring-boot-개요">Spring과 Spring Boot 개요</h2>
<h3 id="1-spring이란">1. Spring이란?</h3>
<p>Spring은 Java 백엔드 애플리케이션 개발을 위한 프레임워크다. 객체 생성과 연결, 웹 요청 처리, DB 접근, 트랜잭션, 보안 등 반복적인 기반 작업을 제공한다.</p>
<p>핵심 개념은 다음과 같다.</p>
<ul>
<li><strong>IoC</strong>: 객체 생성과 관리의 제어권을 개발자가 아니라 Spring이 가진다.</li>
<li><strong>DI</strong>: 객체가 필요한 의존 객체를 Spring이 외부에서 주입한다.</li>
<li><strong>AOP</strong>: 로깅·트랜잭션처럼 여러 곳에 반복되는 관심사를 분리한다.</li>
<li><strong>MVC</strong>: 웹 요청 처리 책임을 역할별로 분리한다.</li>
</ul>
<h3 id="2-spring-boot란">2. Spring Boot란?</h3>
<p>Spring Boot는 Spring의 복잡한 초기 설정을 자동화하고 빠르게 실행하도록 돕는다.</p>
<table>
<thead>
<tr>
<th>Spring Boot 기능</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Auto Configuration</td>
<td>라이브러리와 환경에 맞는 설정 자동 적용</td>
</tr>
<tr>
<td>Starter</td>
<td>필요한 관련 라이브러리를 묶어 제공</td>
</tr>
<tr>
<td>Embedded Server</td>
<td>Tomcat 등을 포함해 JAR로 바로 실행</td>
</tr>
<tr>
<td>Actuator</td>
<td>애플리케이션 상태와 지표 확인</td>
</tr>
</tbody></table>
<blockquote>
<p>Spring은 핵심 프레임워크이고, Spring Boot는 Spring을 빠르고 편하게 시작하게 해주는 실행·설정 도구다.</p>
</blockquote>
<hr />
<h2 id="spring-boot-프로젝트-만들기">Spring Boot 프로젝트 만들기</h2>
<h3 id="1-spring-initializr-설정">1. Spring Initializr 설정</h3>
<p>초기 프로젝트의 대표 설정은 다음과 같다.</p>
<pre><code class="language-text">Project: Maven
Language: Java
Packaging: Jar
Java: 21
Group: com.sk.skala
Artifact: myapp</code></pre>
<p>수업 프로젝트에서 사용하는 주요 의존성:</p>
<ul>
<li>Spring Web</li>
<li>Spring Data JPA</li>
<li>Spring Configuration Processor</li>
<li>Lombok</li>
<li>H2 Database</li>
</ul>
<h3 id="2-프로젝트-기본-구조">2. 프로젝트 기본 구조</h3>
<pre><code class="language-text">myapp/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/sk/skala/myapp/
    │   │   └── MyappApplication.java
    │   └── resources/
    │       └── application.properties
    └── test/</code></pre>
<p><code>pom.xml</code>은 Maven 프로젝트와 의존성을 설명한다. <code>src/main/java</code>에는 Java 코드, <code>resources</code>에는 설정과 SQL 같은 자원이 들어간다.</p>
<h3 id="3-애플리케이션-시작점">3. 애플리케이션 시작점</h3>
<pre><code class="language-java">@SpringBootApplication
public class MyappApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyappApplication.class, args);
    }
}</code></pre>
<p><code>@SpringBootApplication</code>은 자동 설정, 컴포넌트 스캔 등을 묶은 Annotation이다. <code>SpringApplication.run()</code>이 Spring 컨테이너와 내장 서버를 시작한다.</p>
<h3 id="4-첫-번째-api">4. 첫 번째 API</h3>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/api&quot;)
public class HelloController {

    @GetMapping(&quot;/hello&quot;)
    public Map&lt;String, String&gt; hello() {
        return Map.of(&quot;message&quot;, &quot;SKALA에 오신 것을 환영합니다.&quot;);
    }
}</code></pre>
<p>브라우저 또는 Postman에서 다음 주소를 호출한다.</p>
<pre><code class="language-text">GET http://localhost:8080/api/hello</code></pre>
<p>응답:</p>
<pre><code class="language-json">{&quot;message&quot;:&quot;SKALA에 오신 것을 환영합니다.&quot;}</code></pre>
<h3 id="5-component-scan과-bean">5. Component Scan과 Bean</h3>
<p>Spring은 <code>@SpringBootApplication</code>이 붙은 클래스의 패키지와 그 하위 패키지를 탐색한다.</p>
<pre><code class="language-text">com.sk.skala.myapp
├── MyappApplication.java
├── controller/
├── service/
└── repository/</code></pre>
<p>다음 Annotation이 붙은 클래스를 찾아 객체로 생성하고 Spring 컨테이너에 등록한다.</p>
<table>
<thead>
<tr>
<th>Annotation</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>@Component</code></td>
<td>일반 Spring Bean</td>
</tr>
<tr>
<td><code>@Controller</code></td>
<td>View 기반 웹 Controller</td>
</tr>
<tr>
<td><code>@RestController</code></td>
<td>REST API Controller</td>
</tr>
<tr>
<td><code>@Service</code></td>
<td>비즈니스 로직 계층</td>
</tr>
<tr>
<td><code>@Repository</code></td>
<td>데이터 접근 계층</td>
</tr>
</tbody></table>
<p>Spring이 생성·관리하는 객체를 <strong>Bean</strong>이라고 한다. 시작 클래스보다 상위나 형제 패키지에 클래스를 두면 기본 Component Scan에서 찾지 못할 수 있다.</p>
<hr />
<h2 id="mvc-모델">MVC 모델</h2>
<p>MVC는 애플리케이션의 책임을 Model, View, Controller로 나누는 패턴이다.</p>
<table>
<thead>
<tr>
<th>구성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Model</td>
<td>데이터와 상태, 업무 처리 결과</td>
</tr>
<tr>
<td>View</td>
<td>사용자에게 보여 줄 화면 또는 표현</td>
</tr>
<tr>
<td>Controller</td>
<td>요청을 받아 적절한 로직으로 연결</td>
</tr>
</tbody></table>
<p>전통적인 Spring MVC에서 <code>@Controller</code>는 HTML View 이름을 반환한다. REST API에서는 <code>@RestController</code>가 Java 객체를 반환하고, <code>HttpMessageConverter</code>가 이를 JSON으로 변환한다.</p>
<pre><code class="language-java">@Controller
class PageController {
    @GetMapping(&quot;/home&quot;)
    String home() {
        return &quot;home&quot;; // View 이름
    }
}</code></pre>
<pre><code class="language-java">@RestController
class ApiController {
    @GetMapping(&quot;/users/1&quot;)
    User user() {
        return new User(1L, &quot;민지&quot;, &quot;minji@example.com&quot;); // JSON 변환
    }
}</code></pre>
<hr />
<h2 id="controller-및-http-매개변수">Controller 및 HTTP 매개변수</h2>
<h3 id="1-controller의-책임">1. Controller의 책임</h3>
<p>Controller는 HTTP 요청의 진입점이다.</p>
<ul>
<li>요청 URL과 HTTP Method 연결(라우팅)</li>
<li>경로, Query String, Header, Body 값 받기</li>
<li>입력값을 Java 타입으로 변환하고 검증</li>
<li>Service 호출</li>
<li>적절한 HTTP 응답 반환</li>
</ul>
<p>Controller에 DB 조회나 복잡한 업무 규칙을 모두 넣지 않는다. Controller는 교통정리, Service는 실제 업무를 맡는다.</p>
<h3 id="2-요청-매핑">2. 요청 매핑</h3>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/api/users&quot;)
public class UserController {

    @GetMapping
    public List&lt;User&gt; findAll() { ... }

    @PostMapping
    public User create(@RequestBody User user) { ... }

    @PutMapping(&quot;/{id}&quot;)
    public User update(@PathVariable long id,
                       @RequestBody User user) { ... }

    @DeleteMapping(&quot;/{id}&quot;)
    public void delete(@PathVariable long id) { ... }
}</code></pre>
<p>클래스의 <code>/api/users</code>와 메서드의 <code>/{id}</code>가 합쳐져 <code>/api/users/{id}</code>가 된다.</p>
<h3 id="3-pathvariable">3. <code>@PathVariable</code></h3>
<p>URL 경로 자체에 포함된 값을 받는다. 특정 자원을 식별할 때 사용한다.</p>
<pre><code class="language-java">@GetMapping(&quot;/{id}&quot;)
public User findById(@PathVariable long id) {
    return userService.findById(id);
}</code></pre>
<pre><code class="language-text">GET /api/users/10 → id = 10</code></pre>
<h3 id="4-requestparam">4. <code>@RequestParam</code></h3>
<p><code>?key=value</code> 형태의 Query String을 받는다. 검색, 필터, 정렬, 페이징에 적합하다.</p>
<pre><code class="language-java">@GetMapping
public List&lt;User&gt; search(
        @RequestParam(required = false) String name,
        @RequestParam(defaultValue = &quot;0&quot;) int page) {
    return userService.search(name, page);
}</code></pre>
<pre><code class="language-text">GET /api/users?name=민지&amp;page=0</code></pre>
<h3 id="5-requestbody">5. <code>@RequestBody</code></h3>
<p>HTTP Body의 JSON을 Java 객체로 변환한다. 주로 POST, PUT, PATCH에 사용한다.</p>
<pre><code class="language-java">@PostMapping
public User create(@RequestBody User user) {
    return userService.create(user);
}</code></pre>
<pre><code class="language-json">{
  &quot;name&quot;: &quot;민지&quot;,
  &quot;email&quot;: &quot;minji@example.com&quot;
}</code></pre>
<h3 id="6-header와-cookie">6. Header와 Cookie</h3>
<pre><code class="language-java">@GetMapping(&quot;/me&quot;)
public User me(@RequestHeader(&quot;Authorization&quot;) String token) {
    return userService.findByToken(token);
}</code></pre>
<pre><code class="language-java">@GetMapping(&quot;/visit&quot;)
public String visit(@CookieValue(&quot;visitTime&quot;) String visitTime) {
    return visitTime;
}</code></pre>
<table>
<thead>
<tr>
<th>Annotation</th>
<th>가져오는 위치</th>
<th>대표 사용</th>
</tr>
</thead>
<tbody><tr>
<td><code>@PathVariable</code></td>
<td>URL 경로</td>
<td>특정 ID 조회</td>
</tr>
<tr>
<td><code>@RequestParam</code></td>
<td>Query String/Form</td>
<td>검색·필터</td>
</tr>
<tr>
<td><code>@RequestBody</code></td>
<td>HTTP Body</td>
<td>JSON 객체 생성·수정</td>
</tr>
<tr>
<td><code>@RequestHeader</code></td>
<td>HTTP Header</td>
<td>인증 토큰, User-Agent</td>
</tr>
<tr>
<td><code>@CookieValue</code></td>
<td>Cookie</td>
<td>방문 정보, 세션 식별</td>
</tr>
</tbody></table>
<h3 id="7-responseentity">7. <code>ResponseEntity</code></h3>
<p><code>ResponseEntity</code>는 응답 Body뿐 아니라 HTTP 상태 코드와 Header도 직접 설정한다.</p>
<pre><code class="language-java">@PostMapping
public ResponseEntity&lt;User&gt; create(@RequestBody User user) {
    User saved = userService.create(user);
    return ResponseEntity.status(201).body(saved);
}</code></pre>
<p>조회 성공은 <code>200 OK</code>, 생성 성공은 <code>201 Created</code>, 잘못된 요청은 <code>400 Bad Request</code>, 자원 없음은 <code>404 Not Found</code>처럼 상황에 맞는 상태 코드를 사용한다.</p>
<hr />
<h2 id="service-및-repository">Service 및 Repository</h2>
<p>Spring 백엔드는 보통 다음 계층으로 책임을 나눈다.</p>
<pre><code class="language-text">Controller → Service → Repository → Database</code></pre>
<h3 id="1-controller">1. Controller</h3>
<p>HTTP 요청과 응답에 집중한다.</p>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/api/users&quot;)
class UserController {
    private final UserService userService;

    UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping(&quot;/{id}&quot;)
    User findById(@PathVariable long id) {
        return userService.findById(id);
    }
}</code></pre>
<h3 id="2-service">2. Service</h3>
<p>비즈니스 규칙과 작업 순서를 담당한다. 여러 Repository를 조합하고 트랜잭션의 경계가 된다.</p>
<pre><code class="language-java">@Service
class UserService {
    private final UserRepository userRepository;

    UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findById(long id) {
        return userRepository.findById(id)
                .orElseThrow(() -&gt;
                        new IllegalArgumentException(&quot;사용자가 없습니다.&quot;));
    }

    public User create(User user) {
        if (user.getEmail() == null) {
            throw new IllegalArgumentException(&quot;이메일은 필수입니다.&quot;);
        }
        return userRepository.save(user);
    }
}</code></pre>
<h3 id="3-repository">3. Repository</h3>
<p>DB에 데이터를 저장·조회·수정·삭제하는 데이터 접근 계층이다. SQL이나 JPA 같은 구체적인 저장 기술을 Service로부터 숨긴다.</p>
<pre><code class="language-java">@Repository
public interface UserRepository extends JpaRepository&lt;User, Long&gt; {
    List&lt;User&gt; findByName(String name);
}</code></pre>
<p>Spring Data JPA는 인터페이스의 구현 객체를 자동으로 만들어 준다.</p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>기능</th>
</tr>
</thead>
<tbody><tr>
<td><code>findAll()</code></td>
<td>모든 데이터 조회</td>
</tr>
<tr>
<td><code>findById(id)</code></td>
<td>ID로 한 건 조회, <code>Optional</code> 반환</td>
</tr>
<tr>
<td><code>save(entity)</code></td>
<td>새 객체 INSERT, 기존 객체 UPDATE</td>
</tr>
<tr>
<td><code>deleteById(id)</code></td>
<td>ID로 삭제</td>
</tr>
<tr>
<td><code>findByName(name)</code></td>
<td>이름 규칙으로 Query 자동 생성</td>
</tr>
</tbody></table>
<h3 id="4-entity">4. Entity</h3>
<p>Entity는 DB 테이블과 연결되는 객체다.</p>
<pre><code class="language-java">@Entity
@Table(name = &quot;users&quot;)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    protected User() { } // JPA 기본 생성자
}</code></pre>
<h3 id="5-생성자-주입di">5. 생성자 주입(DI)</h3>
<p><code>UserController</code>는 <code>UserService</code>를 직접 <code>new</code>하지 않고 생성자로 받는다. <code>UserService</code>도 <code>UserRepository</code>를 생성자로 받는다.</p>
<pre><code class="language-text">Spring이 UserRepository 구현체 생성
              ↓ 주입
Spring이 UserService 생성
              ↓ 주입
Spring이 UserController 생성</code></pre>
<p>생성자 주입은 의존 관계가 명확하고, 필드를 <code>final</code>로 만들 수 있으며, 테스트에서 가짜 구현체를 넣기 쉽다.</p>
<h3 id="6-h2와-설정">6. H2와 설정</h3>
<p>H2는 학습과 테스트에 편리한 인메모리 DB다.</p>
<pre><code class="language-properties">spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.defer-datasource-initialization=true
spring.sql.init.mode=always</code></pre>
<p><code>resources/data.sql</code>에 초기 데이터를 넣을 수 있다.</p>
<pre><code class="language-sql">INSERT INTO users (name, email)
VALUES ('홍길동', 'hong@sk.com');</code></pre>
<hr />
<h2 id="spring-mvc-framework의-내부-동작">Spring MVC Framework의 내부 동작</h2>
<p>Spring MVC를 이해하려면 요청이 Controller에 바로 도착하는 것이 아니라는 점을 알아야 한다.</p>
<h3 id="1-전체-요청-흐름">1. 전체 요청 흐름</h3>
<pre><code class="language-text">클라이언트
   ↓ HTTP Request
Tomcat(Socket 수신, HTTP 해석)
   ↓
DispatcherServlet
   ↓
HandlerMapping: 처리할 Controller 메서드 검색
   ↓
HandlerAdapter: 매개변수 준비 및 메서드 실행
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
   ↑
Java 객체 반환
   ↑
HttpMessageConverter: Java 객체 → JSON
   ↑
HTTP Response</code></pre>
<h3 id="2-핵심-구성요소">2. 핵심 구성요소</h3>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Tomcat</td>
<td>Port를 열고 TCP/HTTP 요청 수신</td>
</tr>
<tr>
<td><code>DispatcherServlet</code></td>
<td>모든 Spring MVC 요청을 받는 Front Controller</td>
</tr>
<tr>
<td><code>HandlerMapping</code></td>
<td>URL과 HTTP Method에 맞는 Controller 메서드 탐색</td>
</tr>
<tr>
<td><code>HandlerAdapter</code></td>
<td>Controller 매개변수를 준비하고 메서드 호출</td>
</tr>
<tr>
<td>Argument Resolver</td>
<td><code>@PathVariable</code>, <code>@RequestParam</code> 등의 값 생성</td>
</tr>
<tr>
<td>Controller</td>
<td>요청을 Service에 전달하고 결과 반환</td>
</tr>
<tr>
<td><code>HttpMessageConverter</code></td>
<td>JSON과 Java 객체 사이 변환</td>
</tr>
<tr>
<td>ViewResolver</td>
<td>View 기반 MVC에서 화면 템플릿 검색</td>
</tr>
</tbody></table>
<h3 id="3-annotation과-reflection이-연결되는-지점">3. Annotation과 Reflection이 연결되는 지점</h3>
<p>Spring은 시작 시 Component Scan으로 <code>@RestController</code> 클래스를 찾아 Bean으로 등록하고, <code>@GetMapping</code>, <code>@PostMapping</code>이 붙은 메서드 정보를 저장한다.</p>
<p>요청이 들어오면 다음과 같이 동작한다.</p>
<pre><code class="language-text">GET /users/10 요청
       ↓
@GetMapping(&quot;/users/{id}&quot;) 검색
       ↓
{id}의 10을 long 타입으로 변환
       ↓
Reflection으로 getUser(10) 호출
       ↓
반환 객체를 JSON으로 변환</code></pre>
<ul>
<li>Annotation: Spring이 읽을 수 있는 구조화된 표식</li>
<li>Reflection: 클래스와 메서드를 실행 중에 찾고 호출하는 도구</li>
<li>DI Container: 필요한 객체를 생성하고 서로 연결하는 공간</li>
<li>Socket/Tomcat: 실제 네트워크 요청을 받아 HTTP 객체로 바꾸는 기반</li>
</ul>
<p>지금까지 배운 Java 개념이 Spring 안에서 모두 연결된다.</p>
<hr />
<h2 id="전체-흐름으로-복습하기">전체 흐름으로 복습하기</h2>
<p>사용자가 다음 요청을 보냈다고 가정하자.</p>
<pre><code class="language-http">GET /api/users/10 HTTP/1.1
Host: localhost:8080</code></pre>
<ol>
<li>OS와 Tomcat이 8080번 Port의 Socket으로 요청을 받는다.</li>
<li>Tomcat이 HTTP를 해석해 요청 객체를 만든다.</li>
<li><code>DispatcherServlet</code>이 요청을 받는다.</li>
<li><code>HandlerMapping</code>이 <code>@GetMapping(&quot;/{id}&quot;)</code> 메서드를 찾는다.</li>
<li><code>HandlerAdapter</code>가 <code>10</code>을 <code>@PathVariable long id</code>에 연결한다.</li>
<li>Controller가 <code>UserService.findById(10)</code>을 호출한다.</li>
<li>Service가 업무 규칙을 적용하고 Repository를 호출한다.</li>
<li>Repository가 DB에서 사용자를 조회한다.</li>
<li>반환된 <code>User</code> 객체가 Controller까지 올라온다.</li>
<li><code>HttpMessageConverter</code>가 <code>User</code>를 JSON으로 변환한다.</li>
<li>Tomcat이 HTTP 응답을 클라이언트에 전달한다.</li>
</ol>
<h2 id="스스로-확인하기">스스로 확인하기</h2>
<ol>
<li>Factory Method와 Builder는 객체 생성의 어떤 문제를 해결할까?</li>
<li>DIP가 Spring의 DI와 자연스럽게 연결되는 이유는 무엇일까?</li>
<li>Socket과 HTTP는 어떤 관계일까?</li>
<li>REST URI에 동사보다 명사를 사용하는 이유는 무엇일까?</li>
<li><code>@PathVariable</code>, <code>@RequestParam</code>, <code>@RequestBody</code>는 각각 어디의 값을 받을까?</li>
<li>Controller에 DB 코드와 업무 규칙을 모두 넣으면 어떤 문제가 생길까?</li>
<li><code>DispatcherServlet</code>, <code>HandlerMapping</code>, <code>HandlerAdapter</code>는 각각 무슨 역할을 할까?</li>
</ol>