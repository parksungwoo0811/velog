<blockquote>
<p> 범위: 프로그램 기초 → Java 둘러보기 → 개발환경 구성 → 변수와 자료형/네이밍 → 클래스와 객체/주석/제어문 → 패키지/핵심 클래스 → 객체지향 → Java 내부 구조</p>
<p>목표: 문법을 외우기 전에 <strong>Java 프로그램이 왜, 어디서, 어떤 흐름으로 동작하는지</strong> 이해한다.</p>
</blockquote>
<hr />
<h2 id="프로그램-기초">프로그램 기초</h2>
<h3 id="1-프로그램-프로세스-스레드">1. 프로그램, 프로세스, 스레드</h3>
<ul>
<li><strong>프로그램</strong>: 디스크에 저장된 실행 파일 또는 코드. 아직 실행 중이 아니다.</li>
<li><strong>프로세스(Process)</strong>: 운영체제가 메모리에 올려 실제 실행하는 프로그램 한 개. 예를 들어 VS Code와 브라우저를 동시에 실행하면 각각 프로세스다.</li>
<li><strong>스레드(Thread)</strong>: 한 프로세스 안에서 실제 작업을 수행하는 실행 흐름. 하나의 웹 서버 프로세스가 여러 요청을 동시에 처리할 때 여러 스레드를 쓸 수 있다.</li>
</ul>
<blockquote>
<ul>
<li>** 하나의 프로세스에는 여러 스레드가 들어갈 수 있다</li>
<li><em>프로세스는 *</em>각자 메모리를 가지므로 서로 비교적 안전하게 분리<strong>된다. 반면 같은 프로세스의 스레드는 **메모리를 공유하므로 빠르게 협력할 수 있지만, 동시에 같은 데이터를 수정하면 문제</strong>가 생길 수 있다. 이 문제를 뒤에서 <code>synchronized</code>로 다시 만난다.</li>
</ul>
</blockquote>
<h3 id="2-ip와-port-인터넷-주소와-창구-번호">2. IP와 Port: 인터넷 주소와 창구 번호</h3>
<ul>
<li><strong>IP 주소</strong>는 네트워크에서 컴퓨터를 찾는(식별) 논리적 주소다.</li>
<li><strong>Port</strong>는 그 컴퓨터 안에서 어떤 프로그램으로 갈지 정하는 번호다(서비스/ 프로세스를 구분하는 번호).</li>
</ul>
<p>아파트 비유로 보면 IP는 아파트의 주소, Port는 각 세대의 호수다. <code>127.0.0.1:8080</code>은 “내 컴퓨터(localhost)의 8080번 창구”를 뜻한다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>뜻</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>localhost</code></td>
<td>현재 사용 중인 내 컴퓨터</td>
<td>개발 중 서버 접속</td>
</tr>
<tr>
<td><code>127.0.0.1</code></td>
<td>localhost를 가리키는 루프백 IP</td>
<td><code>http://127.0.0.1:8080</code></td>
</tr>
<tr>
<td>Public IP</td>
<td>인터넷에서 접근 가능한 공인 주소</td>
<td>배포된 서버</td>
</tr>
<tr>
<td>Private IP</td>
<td>집/회사 내부 네트워크 주소</td>
<td><code>192.168.x.x</code></td>
</tr>
</tbody></table>
<p><strong>Gateway</strong>는 내부망과 외부망의 출입구(서비스/앱으로 진입하기 위한 진입점)
 <strong>Router</strong>는 패킷의 길을 정하는 장비다.(네트워크를 연결/확장 길을 만드는 기능)
<strong>Load Balancer</strong>는 요청을 여러 서버에 나누어 한 서버에 일이 몰리지 않게 한다.(요청 트래픽을 여러 노드에 분배 및 분산)</p>
<h3 id="3-프론트엔드와-백엔드">3. 프론트엔드와 백엔드</h3>
<ul>
<li><strong>프론트엔드</strong>는 사용자가 보는 화면이다. HTML(구조), CSS(모양), JavaScript(동작)를 주로 사용한다.</li>
<li><strong>백엔드</strong>는 프론트엔드의 요청을 받고, 인증·검증·비즈니스 규칙을 적용하고, DB와 통신해 결과를 돌려준다. Java/Spring은 대표적인 백엔드 조합이다.</li>
</ul>
<p>예를 들어 “상품 주문”에서 화면의 주문 버튼은 프론트엔드, 재고 확인·결제 검증·주문 저장은 백엔드의 책임이다.</p>
<h3 id="4-json-yaml-동기비동기">4. JSON, YAML, 동기/비동기</h3>
<p><strong>JSON</strong>은 API가 데이터를 주고받을 때 자주 쓰는 형식이다.</p>
<pre><code class="language-json">{
  &quot;name&quot;: &quot;민지&quot;,
  &quot;age&quot;: 20,
  &quot;member&quot;: true
}</code></pre>
<p><strong>YAML</strong>은 들여쓰기로 구조를 표현해 설정 파일에서 많이 쓴다. Spring Boot의 <code>application.yml</code>이 대표적이다.</p>
<pre><code class="language-yaml">server:
  port: 8080</code></pre>
<p><strong>동기(Synchronous)</strong>는 요청한 일이 끝날 때까지 기다린다. <strong>비동기(Asynchronous)</strong>는 요청 후 다른 일을 하다가 결과를 나중에 받는다. 주문 접수는 즉시 응답하되, 이메일 발송은 비동기로 처리하는 경우가 흔하다.</p>
<h3 id="5-좋은-소프트웨어의-관점">5. 좋은 소프트웨어의 관점</h3>
<ul>
<li><strong>응집도</strong>: 한 모듈이 하나의 책임에 얼마나 집중하는가. 높을수록 좋다(변경/이해/재사용이 쉬움).</li>
<li><strong>복잡도</strong>: 업무 로직,아키텍쳐의 복잡도(낮을수록 유지보수와 확장 용이)</li>
<li><strong>단독 실행</strong>: 시스템의 일부를 독립적으로 실행하거나 테스트할 수 있는 능력(높을수록 분산 개발/ 배포/ 장애 복구가 유리함)</li>
<li><strong>결합도</strong>: 모듈끼리 얼마나 강하게 얽혀 있는가. 낮을수록 좋다(변경/확장이 쉽고 높으면 전체 시스템에 영향을 줌).</li>
<li><strong>Monolith</strong>: 하나의 애플리케이션에 기능을 모아 배포한다. 시작은 단순하다.</li>
<li><strong>MSA</strong>: 기능별 서비스를 나눈다. 독립 배포·확장에는 유리하지만 네트워크와 운영 복잡도가 커진다.</li>
</ul>
<p>처음에는 “주문 기능은 주문에 집중하게 만들자”라는 높은 응집도 원칙을 이해하는 것이 중요하다.</p>
<hr />
<h2 id="java-둘러보기">Java 둘러보기</h2>
<h3 id="1-java는-왜-쓰는가">1. Java는 왜 쓰는가</h3>
<p>Java는 객체지향 언어이며, JVM 위에서 실행되어 운영체제 차이를 줄인다. 개발자가 작성한 <code>.java</code> 파일은 컴파일러가 <strong>바이트코드</strong>인 <code>.class</code> 파일로 바꾸고, 각 운영체제의 JVM이 이를 실행한다.</p>
<pre><code class="language-text">Hello.java --(javac 컴파일)--&gt; Hello.class --(JVM 실행)--&gt; 화면 출력</code></pre>
<p>그래서 “한 번 작성하면 여러 환경에서 실행한다”는 장점이 있다. 실행 중 자주 쓰이는 코드는 JIT 컴파일러가 기계어로 최적화한다.</p>
<h3 id="2-인터프리터와-컴파일러">2. 인터프리터와 컴파일러</h3>
<ul>
<li><strong>컴파일</strong>: 실행 전에 소스 전체를 변환한다. Java의 <code>javac</code>가 여기에 해당한다.</li>
<li><strong>인터프리트</strong>: 실행하면서 코드를 해석한다.</li>
<li>Java는 바이트코드 컴파일과 JVM의 해석/JIT 최적화를 함께 쓰는 방식이다.</li>
<li><strong>JVM(java virtual machine)</strong>: 자바를 실행하기 위한 가상 머신</li>
</ul>
<h3 id="3-가장-작은-java-프로그램">3. 가장 작은 Java 프로그램</h3>
<pre><code class="language-java">package edu.skala; // 선택: 소속을 나타냄 , 선택사항

public class HelloSkala { // 클래스 선언
    public static void main(String[] args) { //메인 메서드 -프로그램의 시작점
        System.out.println(&quot;Hello, Java!&quot;); // 실행할 코드 작성
    }
    public void anothermethod(){ // 다른 메서드들 (선택)
    }
 }</code></pre>
<ul>
<li><code>class</code>: Java 코드의 기본 단위다(블록).</li>
<li><code>main</code>: 프로그램이 시작하는 약속된 메서드다(실행의 시작점).</li>
<li><code>System.out.println</code>: 콘솔에 한 줄 출력한다.</li>
<li>public 클래스의 파일명은 클래스명과 같아야 한다. 위 코드는 <code>HelloSkala.java</code>에 둔다.</li>
</ul>
<h3 id="4-stack과-heap-맛보기">4. Stack과 Heap 맛보기</h3>
<ul>
<li><strong>Stack</strong>: 메서드 호출 정보, 지역 변수처럼 실행 흐름에 가까운 값이 쌓이는 공간이다. 메서드가 끝나면 해당 공간도 정리된다.스레드마다 별도의 스택 영역</li>
<li><strong>Heap</strong>: <code>new</code>로 만든 객체와 배열이 살아 있는 공간이다. 더 이상 참조되지 않는 객체는 GC가 정리한다. 프러그램 실행 중 생성되는 거의 모든 데이터가 이곳에 저장된다. 여러 스레드가 공유</li>
</ul>
<p>지금은 “지역 변수는 메서드의 실행과 함께 사라지고, 객체는 Heap에 만들어진다” 정도를 기억하자. 마지막 JVM 파트에서 더 자세히 다룬다.</p>
<hr />
<h2 id="개발환경-구성">개발환경 구성</h2>
<h3 id="1-꼭-알아둘-도구">1. 꼭 알아둘 도구</h3>
<ul>
<li><strong>JDK 21</strong>: Java를 작성하고(<code>javac</code>) 실행(<code>java</code>)하는 개발 도구 묶음이다. JRE보다 넓은 개념이다.</li>
<li><strong>IDE</strong>: 코드 작성, 자동완성, 실행, 디버깅을 한곳에서 돕는 도구다. 수업에서는 VS Code와 Java Extension Pack을 사용한다.</li>
<li><strong>Git</strong>: 실습 코드 내려받기와 버전 관리에 사용한다.</li>
</ul>
<p>설치 후에는 아래를 터미널에서 확인한다.</p>
<pre><code class="language-bash">java --version
javac --version</code></pre>
<p>둘 다 버전이 나온다면 Java 실행 환경과 컴파일러가 준비된 것이다. <code>JAVA_HOME</code>은 JDK가 설치된 위치를 가리키는 환경 변수다.</p>
<h3 id="2-vs-code로-프로젝트-만들고-실행하기">2. VS Code로 프로젝트 만들고 실행하기</h3>
<ol>
<li>Java Extension Pack을 설치한다.</li>
<li><code>Java: Create Java Project</code>로 프로젝트를 만든다.</li>
<li><code>src</code>에 소스 코드를 작성한다.</li>
<li>Run 버튼 또는 디버그 메뉴로 실행한다.</li>
</ol>
<p>일반적으로 <code>src</code>는 사람이 작성하는 소스, <code>bin</code>은 컴파일된 <code>.class</code> 파일이 놓이는 출력 폴더다. <code>lib</code>에는 외부 라이브러리가 올 수 있다.</p>
<h3 id="3-cli-실행의-의미">3. CLI 실행의 의미</h3>
<p>IDE가 해 주는 일을 직접 보면 원리를 이해하기 쉽다.</p>
<pre><code class="language-bash">javac -d bin src/Hello.java
java -cp bin Hello</code></pre>
<ul>
<li><code>javac</code>: Java Compiler</li>
<li><code>-d bin</code>: 컴파일 결과를 <code>bin</code>에 저장</li>
<li><code>-cp bin</code>: 클래스 경로(classpath)를 <code>bin</code>으로 지정</li>
</ul>
<hr />
<h2 id="변수-자료형-연산자-네이밍">변수, 자료형, 연산자, 네이밍</h2>
<h3 id="1-변수와-자료형">1. 변수와 자료형</h3>
<p>변수는 값을 담는 <strong>이름표가 붙은 상자</strong>다. Java는 상자에 들어갈 값의 종류를 미리 정하는 정적 타입 언어다.</p>
<pre><code class="language-java">int age = 20;
double price = 19.99;
char grade = 'A';
boolean isMember = true;</code></pre>
<h4 id="기본형primitive과-참조형reference">기본형(Primitive)과 참조형(Reference)</h4>
<table>
<thead>
<tr>
<th>구분</th>
<th>대표 타입</th>
<th>담는 것</th>
</tr>
</thead>
<tbody><tr>
<td>기본형</td>
<td><code>byte</code>, <code>short</code>, <code>int</code>, <code>long</code>, <code>float</code>, <code>double</code>, <code>char</code>, <code>boolean</code></td>
<td>값(value) 자체</td>
</tr>
<tr>
<td>참조형</td>
<td><code>String</code>, 배열, 클래스, 인터페이스</td>
<td>객체가 있는 위치를 가리키는 참조</td>
</tr>
</tbody></table>
<p>정수는 보통 <code>int</code>, 실수는 보통 <code>double</code>을 먼저 사용한다. <code>long</code> 리터럴에는 <code>L</code>을 붙이고, <code>float</code> 리터럴에는 <code>F</code>를 붙인다.
참조형의 기본값은 null</p>
<pre><code class="language-java">long population = 5_000_000_000L;
float ratio = 0.5F;
String name = &quot;Java&quot;;</code></pre>
<p><code>String</code>은 참조형이며 <strong>불변(immutable)</strong> 객체다. 문자열을 바꾸는 것처럼 보여도 실제로는 새 문자열이 만들어진다.</p>
<h3 id="2-wrapper-클래스와-비교">2. Wrapper 클래스와 비교</h3>
<p>기본형에도 객체 형태의 짝이 있다. <code>int</code>의 짝은 <code>Integer</code>, <code>double</code>의 짝은 <code>Double</code>, <code>boolean</code>의 짝은 <code>Boolean</code>이다. 컬렉션에는 객체만 넣을 수 있으므로 Wrapper가 필요하다.</p>
<pre><code class="language-java">int number = 10;
Integer boxed = number; // auto-boxing
int again = boxed;      // auto-unboxing</code></pre>
<p>객체의 내용 비교에는 <code>==</code> 대신 <code>equals()</code>를 쓴다.</p>
<pre><code class="language-java">Integer a = 1000;
Integer b = 1000;
System.out.println(a == b);      // 같은 객체인가? (권장하지 않음)
System.out.println(a.equals(b)); // 값이 같은가? true</code></pre>
<h2 id="불변-객체immutable-object-가볍게-정리">불변 객체(Immutable Object) 가볍게 정리</h2>
<p><strong>불변 객체</strong>는 한 번 만들어지면 내부 값을 바꿀 수 없는 객체다.<br />값을 바꾸는 것처럼 보여도, 실제로는 <strong>새 객체를 만들고 변수가 새 객체를 가리키게</strong> 된다.</p>
<p>대표적인 불변 객체:</p>
<ul>
<li><code>String</code></li>
<li><code>Integer</code>, <code>Double</code>, <code>Boolean</code> 같은 Wrapper 클래스</li>
</ul>
<pre><code class="language-java">Integer a = 10;
Integer b = a; // a와 b는 처음에는 같은 10을 가리킴

a = 20;        // 10을 수정하는 것이 아니라, a가 새 Integer(20)를 가리킴</code></pre>
<p>결과는 다음과 같다.</p>
<pre><code class="language-text">a = 20
b = 10</code></pre>
<p><code>b</code>가 함께 <code>20</code>으로 바뀌지 않는 이유는 <code>Integer</code> 자체의 값 <code>10</code>을 수정한 것이 아니기 때문이다.</p>
<blockquote>
<p>참고: Java는 엄밀히 말해 항상 <strong>값 전달(Call by Value)</strong> 방식이다. 객체를 넘길 때도 객체 자체가 아니라, 객체를 가리키는 참조값을 복사해서 전달한다.</p>
</blockquote>
<h3 id="3-연산자">3. 연산자</h3>
<ul>
<li>산술: <code>+</code>, <code>-</code>, <code>*</code>, <code>/</code>, <code>%</code></li>
<li>비교: <code>==</code>, <code>!=</code>, <code>&lt;</code>, <code>&gt;</code>, <code>&lt;=</code>, <code>&gt;=</code></li>
<li>논리: <code>&amp;&amp;</code>(그리고 and), <code>||</code>(또는 or), <code>!</code>(부정 not)</li>
<li>대입: <code>=</code>, <code>+=</code>, <code>-=</code> 등</li>
<li>증감: <code>++</code>, <code>--</code></li>
<li>삼항: <code>조건 ? 참일 때 값 : 거짓일 때 값</code></li>
</ul>
<pre><code class="language-java">int score = 85;
String result = score &gt;= 80 ? &quot;합격&quot; : &quot;불합격&quot;;

int i = 5;
System.out.println(++i); // 먼저 1 증가: 6
System.out.println(i++); // 먼저 사용: 6, 그 다음 1 증가
System.out.println(i);   // 7</code></pre>
<p>정수끼리의 <code>/</code>는 소수점을 버린다. <code>10 / 3</code>은 <code>3</code>이다. 실수 결과가 필요하면 한쪽을 <code>double</code>로 만든다.</p>
<h3 id="4-문자열-만들기">4. 문자열 만들기</h3>
<pre><code class="language-java">String name = &quot;스칼라&quot;;
int age = 30;
System.out.println(String.format(&quot;이름: %s, 나이: %d&quot;, name, age));
System.out.printf(&quot;이름: %s, 나이: %d%n&quot;, name, age);</code></pre>
<p>반복해서 문자열을 이어 붙일 때는 <code>StringBuilder</code>가 효율적이다.</p>
<pre><code class="language-java">StringBuilder message = new StringBuilder();
message.append(&quot;안녕, &quot;).append(name).append(&quot;!&quot;);
System.out.println(message);</code></pre>
<p><code>StringBuilder</code>는 빠르지만 스레드 안전하지 않고, <code>StringBuffer</code>는 동기화되어 여러 스레드가 접근해도 안전하지만 조금 느리다. 대부분의 단일 스레드 문자열 조립에는 <code>StringBuilder</code>를 쓴다.</p>
<h3 id="5-동시성의-첫-단어-mutex와-synchronized">5. 동시성의 첫 단어: Mutex와 <code>synchronized</code></h3>
<p>공유 잔액을 두 스레드가 동시에 출금하면 둘 다 같은 옛 잔액을 읽어 결과가 틀릴 수 있다. 이런 경쟁 상태를 막기 위해 한 번에 하나만 접근하게 하는 것이 <strong>상호 배제(Mutex)</strong> 다.</p>
<ul>
<li>공유 자원에는 한 번에 하나의 thread 만 접근해야한다는 개념</li>
</ul>
<pre><code class="language-java">public synchronized void withdraw(int amount) {
    balance -= amount;
}</code></pre>
<p><code>synchronized</code>는 편리하지만 잠금 범위가 넓으면 성능에 영향을 줄 수 있다. 우선은 “공유 데이터를 수정하는 구간을 안전하게 만든다”로 이해하자.</p>
<h3 id="6-네이밍-규칙">6. 네이밍 규칙</h3>
<p>좋은 이름은 주석보다 먼저 읽히는 문서다. 약어보다 역할과 의도가 보이는 이름을 쓴다.</p>
<table>
<thead>
<tr>
<th>대상</th>
<th>규칙</th>
<th>좋은 예</th>
</tr>
</thead>
<tbody><tr>
<td>클래스/인터페이스</td>
<td>PascalCase</td>
<td><code>StockManager</code>, <code>Runnable</code></td>
</tr>
<tr>
<td>메서드/변수</td>
<td>camelCase</td>
<td><code>calculateTotal()</code>, <code>orderCount</code></td>
</tr>
<tr>
<td>상수</td>
<td>UPPER_SNAKE_CASE</td>
<td><code>MAX_RETRY_COUNT</code></td>
</tr>
<tr>
<td>패키지</td>
<td>소문자</td>
<td><code>com.example.order</code></td>
</tr>
</tbody></table>
<p>클래스와 변수는 대체로 <strong>명사</strong>, 메서드는 <strong>동사 + 명사</strong>가 자연스럽다.</p>
<pre><code class="language-java">// 피하기: 의미가 드러나지 않음
int a;
void data() {}

// 권장: 읽는 즉시 역할을 알 수 있음
int customerCount;
void createOrder() {}</code></pre>
<p>CRUD 메서드는 <code>findOrder</code>, <code>createOrder</code>, <code>updateOrder</code>, <code>deleteOrder</code>처럼 일관되게 짓는다.</p>
<hr />
<h2 id="클래스와-객체-주석-제어문1">클래스와 객체, 주석, 제어문(1)</h2>
<h3 id="1-클래스와-객체">1. 클래스와 객체</h3>
<p><strong>클래스</strong>는 설계도, <strong>객체(인스턴스)</strong>는 그 설계도로 <code>new</code>를 통해 실제 만든 결과물이다.</p>
<table>
<thead>
<tr>
<th>개념</th>
<th>설명</th>
<th>Java 예제 코드</th>
<th>일상 예시</th>
</tr>
</thead>
<tbody><tr>
<td>클래스<br />(Class)</td>
<td>객체를 만들기 위한 설계도. 속성과 동작을 정의한다.</td>
<td><code>class Student {}</code></td>
<td>학생을 만들기 위한 <strong>학생 정보 양식</strong></td>
</tr>
<tr>
<td>객체<br />(Object)</td>
<td>클래스를 기반으로 생성된 실제 대상(인스턴스)</td>
<td><code>Student s1 = new Student();</code></td>
<td>실제 존재하는 학생 <strong>김민지 한 명</strong></td>
</tr>
<tr>
<td>속성<br />(Field)</td>
<td>객체가 가지는 상태나 정보(변수)</td>
<td><code>String name;</code><br /><code>double grade;</code></td>
<td>학생의 <strong>이름, 학번, 성적</strong></td>
</tr>
<tr>
<td>메서드<br />(Method)</td>
<td>객체가 수행하는 동작(함수)</td>
<td><code>void study() {}</code></td>
<td>학생이 <strong>공부하기</strong>, <strong>과제 제출하기</strong></td>
</tr>
<tr>
<td>생성자<br />(Constructor)</td>
<td>객체를 만들 때 초기값을 설정하는 특별한 메서드. 클래스명과 같다.</td>
<td><code>Student(String name, int id) {}</code></td>
<td>학생 등록 시 <strong>이름과 학번을 입력해 학생 정보를 만드는 과정</strong></td>
</tr>
</tbody></table>
<pre><code class="language-java">class Stock {
    String name;       // 필드: 객체의 상태
    int price;

    void printInfo() { // 메서드: 객체의 행동
        System.out.println(name + &quot;: &quot; + price);
    }
}

public class Main {
    public static void main(String[] args) {
        Stock stock = new Stock();
        stock.name = &quot;삼성전자&quot;;
        stock.price = 70000;
        stock.printInfo();
    }
}</code></pre>
<ul>
<li><strong>필드(Field)</strong>: 객체마다 가지는 상태. 예: 종목명, 가격</li>
<li><strong>메서드(Method)</strong>: 객체가 할 수 있는 일(동작). 예: 정보 출력, 가격 변경</li>
<li><strong>생성자(Constructor)</strong>: 객체를 처음 만들 때 필요한 값을 준비한다.(클래스 이름과 동일)</li>
<li><code>this</code>: “바로 이 객체 자신”을 뜻한다. 필드와 매개변수의 이름이 같을 때 구분한다.</li>
</ul>
<pre><code class="language-java">class User {
    private String name;

    User(String name) {
        this.name = name;
    }
}</code></pre>
<h3 id="메서드의-입력과-결과">메서드의 입력과 결과</h3>
<pre><code class="language-java">double calculatePriceChange(double yesterday, double today) {
    return today - yesterday;
}</code></pre>
<ul>
<li><code>double</code>: 반환형. 결과가 있다는 뜻이다. 결과가 없으면 <code>void</code>.</li>
<li><code>yesterday</code>, <code>today</code>: <strong>매개변수(parameter)</strong>. 메서드가 받을 입력을 담는 변수다.</li>
<li>호출할 때 넣는 실제 값은 <strong>인자(argument)</strong> 다. 예: <code>calculatePriceChange(100, 120)</code>의 <code>100</code>, <code>120</code>.</li>
</ul>
<p>같은 이름으로 매개변수 목록이 다른 메서드를 여러 개 만드는 것을 <strong>오버로딩</strong>이라고 한다.</p>
<h3 id="static은-누구의-것인가"><code>static</code>은 누구의 것인가</h3>
<p>일반 필드와 메서드는 객체마다 존재한다. <strong>static(정적 필드)</strong> 멤버는 객체가 아니라 클래스 전체가 하나만 공유한다. 반면에 <strong>상수 필드(final)</strong>는 값의 수정이 불가능하다.</p>
<pre><code class="language-java">class Counter {
    static int totalCount = 0;
    int personalCount = 0;
}</code></pre>
<p><code>Counter.totalCount</code>는 모두가 공유하지만, 각 객체의 <code>personalCount</code>는 따로 가진다. <code>main</code>이 <code>static</code>인 이유도 객체를 만들기 전 JVM이 클래스 단위로 시작점을 호출해야 하기 때문이다.</p>
<h3 id="2-주석-달기">2. 주석 달기</h3>
<table>
<thead>
<tr>
<th>문법</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>// 내용</code></td>
<td>한 줄 설명, TODO</td>
</tr>
<tr>
<td><code>/* 내용 */</code></td>
<td>여러 줄 설명</td>
</tr>
<tr>
<td><code>/** 내용 */</code></td>
<td>Javadoc 문서용 주석</td>
</tr>
</tbody></table>
<pre><code class="language-java">/**
 * 두 수를 더한다.
 * @param a 첫 번째 수
 * @param b 두 번째 수
 * @return 두 수의 합
 */
public int add(int a, int b) {
    return a + b;
}</code></pre>
<p>주석은 코드를 번역하기보다 <strong>왜 이런 선택을 했는지</strong>, 제약이나 주의점을 남길 때 가치가 크다. <code>i++</code>에 “i를 증가”라고 쓰기보다, “외부 API 제한 때문에 최대 3회만 재시도”라고 쓴다.</p>
<h3 id="3-조건문-if-else-if-else">3. 조건문: <code>if</code>, <code>else if</code>, <code>else</code></h3>
<p>조건식의 결과가 <code>true</code>일 때만 해당 블록을 실행한다.</p>
<pre><code class="language-java">if (score &gt;= 90) {
    System.out.println(&quot;A&quot;);
} else if (score &gt;= 80) {
    System.out.println(&quot;B&quot;);
} else {
    System.out.println(&quot;C 이하&quot;);
}</code></pre>
<p>여러 조건의 순서가 중요하다. <code>score &gt;= 80</code>을 먼저 쓰면 90점도 B에서 걸리므로, 더 좁고 높은 기준부터 쓴다.</p>
<h3 id="switch"><code>switch</code></h3>
<p>정해진 값 중 하나를 고르는 분기에 잘 맞는다. 각 <code>case</code> 끝의 <code>break</code>를 빼먹으면 다음 case까지 실행되는 fall-through가 발생한다.</p>
<pre><code class="language-java">switch (menu) {
    case 1 -&gt; System.out.println(&quot;조회&quot;);
    case 2 -&gt; System.out.println(&quot;매수&quot;);
    default -&gt; System.out.println(&quot;잘못된 메뉴&quot;);
}</code></pre>
<p>Java 14 이후의 <strong>switch expression</strong> <code>-&gt;</code> 문법은 <code>break</code>를 생략할 수 있어 초보자에게 더 안전하다. 메뉴 종류처럼 값이 한정될 때는 <code>enum</code>과 함께 쓰면 좋다.</p>
<hr />
<h2 id="제어문2-반복-흐름-변경-예외">제어문(2): 반복, 흐름 변경, 예외</h2>
<h3 id="1-반복문-선택하기">1. 반복문 선택하기</h3>
<table>
<thead>
<tr>
<th>반복문</th>
<th>알맞은 상황</th>
</tr>
</thead>
<tbody><tr>
<td><code>while</code></td>
<td>조건이 참인 동안, 반복 횟수를 미리 모를 때</td>
</tr>
<tr>
<td><code>do-while</code></td>
<td>본문을 반드시 한 번 실행해야 할 때</td>
</tr>
<tr>
<td><code>for</code></td>
<td>반복 횟수나 인덱스가 분명할 때</td>
</tr>
<tr>
<td>enhanced <code>for</code></td>
<td>배열/컬렉션의 모든 값을 순서대로 읽을 때</td>
</tr>
</tbody></table>
<pre><code class="language-java">int count = 0;
while (count &lt; 3) {
    count++;
}

do {
    System.out.println(&quot;최소 한 번 실행&quot;);
} while (false);

for (int i = 0; i &lt; 3; i++) {
    System.out.println(i);
}

String[] names = {&quot;민지&quot;, &quot;철수&quot;};
for (String name : names) {
    System.out.println(name);
}</code></pre>
<p><code>for-each</code>는 읽기 쉽지만 인덱스가 필요하거나 배열의 특정 칸을 바꿔야 할 때는 일반 <code>for</code>가 낫다. 반복 조건을 갱신하지 않으면 무한 루프가 되므로 <code>while</code>에는 특히 주의한다.</p>
<h3 id="2-흐름을-바꾸는-키워드">2. 흐름을 바꾸는 키워드</h3>
<ul>
<li><code>break</code>: 가장 가까운 반복문 또는 <code>switch</code>를 완전히 끝낸다.</li>
<li><code>continue</code>: 이번 회차만 건너뛰고 다음 반복으로 간다.</li>
<li><code>return</code>: 현재 메서드 전체를 끝내고, 필요하면 값을 돌려준다.</li>
</ul>
<pre><code class="language-java">for (int i = 1; i &lt;= 5; i++) {
    if (i == 3) continue; // 3만 건너뜀
    if (i == 5) break;    // 반복 종료
    System.out.println(i);
}</code></pre>
<h3 id="3-예외-처리">3. 예외 처리</h3>
<p>예외(Exception)는 실행 중 발생한 비정상 상황이다. 0으로 나누기, 존재하지 않는 파일 접근 등이 예다.</p>
<pre><code class="language-java">try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println(&quot;0으로 나눌 수 없습니다.&quot;);
} finally {
    System.out.println(&quot;성공/실패와 관계없이 마무리&quot;);
}</code></pre>
<ul>
<li><code>throw</code>: 코드에서 예외를 직접 발생시킨다.</li>
<li><code>throws</code>: 이 메서드가 예외를 호출한 쪽에 넘길 수 있음을 선언한다.</li>
</ul>
<pre><code class="language-java">void buyStock(int amount) throws IllegalArgumentException {
    if (amount &lt;= 0) {
        throw new IllegalArgumentException(&quot;수량은 1 이상이어야 합니다.&quot;);
    }
}</code></pre>
<p>파일·DB·네트워크처럼 사용 후 닫아야 하는 자원은 <strong>try-with-resources</strong>로 관리한다.
외부 리소스를 사용하면 반드시 close()로 닫아줘야하는데, </p>
<pre><code class="language-java">try (BufferedReader reader = Files.newBufferedReader(path)) {
    System.out.println(reader.readLine());
} catch (IOException e) {
    System.out.println(&quot;파일을 읽지 못했습니다.&quot;);
}</code></pre>
<p><code>try()</code> 블록이 끝나면 <code>reader.close()</code>가 자동 호출된다. <code>RuntimeException</code> 계열은 주로 코드 실수로 발생하는 비검사 예외, 그 외 <code>Exception</code> 계열은 컴파일러가 처리 여부를 확인하는 검사 예외인 경우가 많다.</p>
<hr />
<h2 id="패키지와-java-필수-클래스">패키지와 Java 필수 클래스</h2>
<h3 id="1-패키지와-import">1. 패키지와 import</h3>
<p>패키지는 관련 클래스를 묶는 <strong>논리적 주소</strong>다. 이름 충돌을 막고 코드를 기능별로 정리한다.</p>
<pre><code class="language-java">package com.example.stock;

import java.util.ArrayList;</code></pre>
<p><code>package</code> 선언은 파일의 맨 위에 둔다. 패키지 이름은 보통 도메인을 거꾸로 한 소문자 형태(<code>com.company.project</code>)를 사용한다.</p>
<h3 id="2-접근-제어자">2. 접근 제어자</h3>
<table>
<thead>
<tr>
<th>접근 제어자</th>
<th align="center">같은 클래스</th>
<th align="center">같은 패키지</th>
<th align="center">자식 클래스(다른 패키지)</th>
<th align="center">모든 곳</th>
</tr>
</thead>
<tbody><tr>
<td><code>private</code></td>
<td align="center">O</td>
<td align="center">X</td>
<td align="center">X</td>
<td align="center">X</td>
</tr>
<tr>
<td>default(생략)</td>
<td align="center">O</td>
<td align="center">O</td>
<td align="center">X</td>
<td align="center">X</td>
</tr>
<tr>
<td><code>protected</code></td>
<td align="center">O</td>
<td align="center">O</td>
<td align="center">O</td>
<td align="center">X</td>
</tr>
<tr>
<td><code>public</code></td>
<td align="center">O</td>
<td align="center">O</td>
<td align="center">O</td>
<td align="center">O</td>
</tr>
</tbody></table>
<p>핵심 원칙은 “필요한 만큼만 공개한다”이다. 필드는 기본적으로 <code>private</code>으로 숨기고, 필요한 행동만 <code>public</code> 메서드로 제공한다.</p>
<h3 id="3-꼭-알아둘-javalang-클래스">3. 꼭 알아둘 <code>java.lang</code> 클래스</h3>
<table>
<thead>
<tr>
<th>클래스</th>
<th>초보자가 먼저 할 일</th>
</tr>
</thead>
<tbody><tr>
<td><code>Object</code></td>
<td>모든 클래스의 공통 부모, <code>toString()</code>, <code>equals()</code>, <code>hashCode()</code> 이해</td>
</tr>
<tr>
<td><code>System</code></td>
<td><code>out.println</code>, 시간 측정, 환경 정보</td>
</tr>
<tr>
<td><code>Thread</code></td>
<td><code>start()</code>, <code>sleep()</code>, <code>join()</code>으로 실행 흐름 맛보기</td>
</tr>
<tr>
<td><code>Exception</code></td>
<td>오류를 잡고 의미 있게 처리</td>
</tr>
<tr>
<td><code>Runtime</code></td>
<td>JVM 메모리 정보, GC, 외부 프로세스 관련 기능</td>
</tr>
<tr>
<td><code>Class</code></td>
<td>클래스 자체의 정보, 리플렉션의 출발점</td>
</tr>
</tbody></table>
<p><code>java.lang</code>은 자동 import되므로 <code>String</code>, <code>System</code>, <code>Object</code>에 매번 import를 쓰지 않는다.</p>
<hr />
<h2 id="객체지향-프로그래밍oop">객체지향 프로그래밍(OOP)</h2>
<h3 id="1-객체는-역할--상태--행동">1. 객체는 역할 + 상태 + 행동</h3>
<p>현실의 “고객이 상품을 주문한다”를 코드로 옮기면 고객, 상품, 주문을 객체로 볼 수 있다.</p>
<blockquote>
<ul>
<li>고객은 행위를 하는 주체</li>
</ul>
</blockquote>
<ul>
<li><p>상품은 행위가 향하는 대상인 객체</p>
</li>
<li><p><strong>상태(State)</strong>: 객체가 기억하는 데이터 - 상품명, 가격, 재고</p>
</li>
<li><p><strong>행동(Behavior)</strong>: 객체가 책임지는 기능 - 재고 감소, 가격 변경, 주문 취소</p>
</li>
<li><p><strong>역할(Role)/책임(Responsibility)</strong>: 객체가 시스템에서 맡은 일</p>
</li>
</ul>
<p>OOP는 기능을 줄줄이 나열하기보다, 책임 있는 객체들이 서로 메시지(메서드 호출)를 보내 협력하도록 설계하는 방법이다.</p>
<h3 id="2-oop의-핵심-4가지">2. OOP의 핵심 4가지</h3>
<h4 id="캡슐화encapsulation">캡슐화(Encapsulation)</h4>
<p>데이터를 숨기고 안전한 방법으로만 변경한다.접근 제어</p>
<pre><code class="language-java">class Stock {
    private int price;

    public void changePrice(int price) {
        if (price &lt; 0) {
            throw new IllegalArgumentException(&quot;가격은 음수일 수 없습니다.&quot;);
        }
        this.price = price;
    }
}</code></pre>
<p><code>stock.price = -1</code>처럼 아무 값이나 넣게 두는 대신, <code>changePrice()</code>에서 규칙을 지키게 한다.</p>
<h4 id="상속inheritance">상속(Inheritance)</h4>
<p>부모 클래스의 공통 기능을 자식 클래스가 물려받아 재사용한다. <code>extends</code>를 사용한다.</p>
<pre><code class="language-java">class Stock {
    void printInfo() { System.out.println(&quot;종목 정보&quot;); }
}

class PreferredStock extends Stock {
    void payDividend() { System.out.println(&quot;배당 지급&quot;); }
}</code></pre>
<p>상속은 “자식은 부모의 한 종류다(is-a)”가 자연스러울 때 사용한다. 단순 코드 재사용만을 위해 무리하게 상속하지 않는다.</p>
<h4 id="추상화abstraction">추상화(Abstraction)</h4>
<p>복잡한 내부 구현을 감추고, 사용자에게 필요한 핵심 기능만 보여 준다. Java에서는 추상 클래스와 인터페이스가 대표적인 도구다.</p>
<h4 id="다형성polymorphism">다형성(Polymorphism)</h4>
<p>하나의 상위 타입으로 여러 실제 객체를 다룬다. 같은 메서드 호출이 실제 객체에 따라 다르게 동작할 수 있다.</p>
<ul>
<li>메서드 오보로딩: 같은 이름의 메서드를 매개변수 타입이나 개수에 따라 다르게 정의</li>
<li>메서드 오버라이딩: 부모 클래스의 메서드를 자식 클래스에서 재정의</li>
</ul>
<pre><code class="language-java">Stock stock = new PreferredStock(); // 업캐스팅
stock.printInfo();</code></pre>
<p>이 방식은 사용하는 쪽이 “무엇을 할 수 있는가”에만 의존하게 하고, 구체적인 구현 변경의 영향을 줄인다.</p>
<h2 id="oop-핵심-4가지-카페-주문으로-이해하기">OOP 핵심 4가지: 카페 주문으로 이해하기</h2>
<h3 id="1-캡슐화-중요한-정보는-숨기고-정해진-방법으로만-사용">1. 캡슐화: 중요한 정보는 숨기고, 정해진 방법으로만 사용</h3>
<p>카페의 금고를 손님이 직접 열 수는 없습니다.<br />손님은 직원에게 “결제할게요”라고 요청하고, 직원은 정해진 절차로 결제합니다.</p>
<pre><code class="language-java">class Account {
    private int balance; // 외부에서 직접 변경 불가

    public void deposit(int amount) {
        if (amount &gt; 0) {
            balance += amount;
        }
    }
}</code></pre>
<p><code>balance</code>를 직접 바꾸지 못하게 하고, <code>deposit()</code>처럼 안전한 메서드를 통해서만 변경합니다.</p>
<hr />
<h3 id="2-상속-공통-특징을-물려받기">2. 상속: 공통 특징을 물려받기</h3>
<p>아메리카노와 라테는 모두 음료입니다.<br />음료라는 공통 특징(이름, 가격)을 물려받고, 각자만의 특징을 추가합니다.</p>
<pre><code class="language-java">class Beverage {
    String name;
    int price;
}

class Americano extends Beverage {
    boolean hasIce;
}

class Latte extends Beverage {
    boolean hasMilk;
}</code></pre>
<p><code>Americano</code>는 Beverage의 한 종류이고, <code>Latte</code>도 Beverage의 한 종류입니다.</p>
<hr />
<h3 id="3-추상화-복잡한-내부-과정은-숨기고-필요한-기능만-보여-주기">3. 추상화: 복잡한 내부 과정은 숨기고 필요한 기능만 보여 주기</h3>
<p>손님은 키오스크에서 메뉴를 고르고 결제 버튼만 누릅니다.<br />카드 승인, 재고 확인, 영수증 출력 과정까지 알 필요는 없습니다.</p>
<pre><code class="language-java">class Kiosk {
    public void order(String menu) {
        // 결제, 재고 확인 등의 복잡한 내부 과정
        System.out.println(menu + &quot; 주문 완료&quot;);
    }
}</code></pre>
<p>사용하는 사람에게는 “주문한다”라는 핵심 기능만 보여 주는 것이 추상화입니다.</p>
<hr />
<h3 id="4-다형성-같은-요청이-대상에-따라-다르게-동작">4. 다형성: 같은 요청이 대상에 따라 다르게 동작</h3>
<p>“음료를 만든다”는 요청은 같지만, 아메리카노와 라테의 제조 방식은 다릅니다.</p>
<pre><code class="language-java">class Beverage {
    void make() {
        System.out.println(&quot;음료를 만듭니다.&quot;);
    }
}

class Americano extends Beverage {
    @Override
    void make() {
        System.out.println(&quot;에스프레소와 물을 넣습니다.&quot;);
    }
}

class Latte extends Beverage {
    @Override
    void make() {
        System.out.println(&quot;에스프레소와 우유를 넣습니다.&quot;);
    }
}</code></pre>
<pre><code class="language-java">Beverage drink = new Latte();
drink.make();</code></pre>
<p>변수 타입은 <code>Beverage</code>지만 실제 객체가 <code>Latte</code>이므로 다음이 출력됩니다.</p>
<pre><code class="language-text">에스프레소와 우유를 넣습니다.</code></pre>
<p>즉, <strong>같은 <code>make()</code> 요청이지만 실제 음료에 따라 다르게 실행되는 것</strong>이 다형성입니다.</p>
<h2 id="오버라이딩-같은-요청-대상에-따라-다른-실행">오버라이딩: 같은 요청, 대상에 따라 다른 실행</h2>
<p>“알림을 출력해 줘”라는 요청은 같지만, 대상에 따라 출력 방식이 달라지는 상황입니다.
*<em>쉽게 암기하는 법 부모 기능을 자식이 덮어씀/ 부모 메서드를 똑같이 덮어씀
*</em>
예를 들어 일반 직원과 매니저가 출근 정보를 출력한다고 해봅시다.</p>
<pre><code class="language-java">class Employee {
    void printRole() {
        System.out.println(&quot;일반 직원입니다.&quot;);
    }
}

class Manager extends Employee {
    @Override
    void printRole() {
        System.out.println(&quot;관리자입니다.&quot;);
    }
}</code></pre>
<pre><code class="language-java">Employee employee = new Manager();
employee.printRole();</code></pre>
<p>결과:</p>
<pre><code class="language-text">관리자입니다.</code></pre>
<p>부모의 <code>printRole()</code>을 자식 <code>Manager</code>가 자기 방식으로 바꿨으므로 <strong>오버라이딩</strong>입니다.</p>
<p><code>Stock</code> 예제에서는 다음과 같습니다.</p>
<pre><code class="language-text">Stock: 일반주 정보를 출력
PreferredStock: 우선주 정보와 배당률까지 출력</code></pre>
<hr />
<h2 id="오버로딩-같은-서비스-이름-요청-내용에-따라-다른-처리">오버로딩: 같은 서비스 이름, 요청 내용에 따라 다른 처리</h2>
<p>*<em>쉬운 암기법: 같은 이름의 기능을 여러 형태로 추가/ 같은 클래스 안에서 괄호 안이 다르면 오버로딩
*</em></p>
<p>카페 직원에게 “커피 주문해 주세요”라고 말할 수 있습니다.</p>
<pre><code class="language-text">커피 주문()
커피 주문(&quot;아메리카노&quot;)
커피 주문(&quot;라테&quot;, 2)</code></pre>
<p>모두 “커피 주문”이지만 전달하는 정보가 다릅니다.</p>
<pre><code class="language-java">class Cafe {
    void orderCoffee() {
        System.out.println(&quot;기본 커피를 주문합니다.&quot;);
    }

    void orderCoffee(String menu) {
        System.out.println(menu + &quot;를 주문합니다.&quot;);
    }

    void orderCoffee(String menu, int count) {
        System.out.println(menu + &quot; &quot; + count + &quot;잔을 주문합니다.&quot;);
    }
}</code></pre>
<pre><code class="language-java">Cafe cafe = new Cafe();

cafe.orderCoffee();
cafe.orderCoffee(&quot;라테&quot;);
cafe.orderCoffee(&quot;아메리카노&quot;, 2);</code></pre>
<p>결과:</p>
<pre><code class="language-text">기본 커피를 주문합니다.
라테를 주문합니다.
아메리카노 2잔을 주문합니다.</code></pre>
<p>이처럼 메서드 이름은 같지만 입력값의 개수나 타입이 다르면 <strong>오버로딩</strong>입니다.</p>
<p>한 줄 요약:</p>
<blockquote>
<p>오버라이딩은 “같은 요청을 대상에 맞게 새로 구현”, 오버로딩은 “같은 기능 이름으로 여러 요청 형태 지원”입니다.</p>
</blockquote>
<h3 id="3-좋은-객체-설계">3. 좋은 객체 설계</h3>
<p><strong>높은 응집도, 낮은 결합도</strong>를 다시 적용한다.</p>
<ul>
<li><code>Order</code>가 주문 계산과 주문 상태를 책임지는 것은 높은 응집도에 가깝다.</li>
<li><code>Order</code>가 화면 출력, 파일 저장, 결제, 이메일을 전부 책임지면 변경에 취약해진다.</li>
<li>객체는 다른 객체의 내부 필드를 직접 만지기보다, 필요한 일을 메서드로 요청한다.</li>
</ul>
<p>절차적 프로그래밍은 함수 호출 순서 중심, 객체지향은 데이터와 행동을 묶은 객체의 협력 중심이라는 차이가 있다. 어느 한쪽이 항상 우월한 것이 아니라, 규모와 변경 가능성에 맞춰 설계한다.</p>
<hr />
<h2 id="java-내부-구조-jvm과-메모리">Java 내부 구조: JVM과 메모리</h2>
<h3 id="1-jvm-실행-흐름">1. JVM 실행 흐름</h3>
<pre><code class="language-text">.java 소스 코드
    ↓ javac
.class 바이트코드
    ↓ Class Loader
JVM 메모리에 클래스 정보 적재
    ↓ Execution Engine (인터프리터/JIT)
CPU에서 실행</code></pre>
<ul>
<li><strong>Class Loader</strong>: 필요한 <code>.class</code> 파일을 읽어 JVM에 올린다.</li>
<li><strong>Execution Engine</strong>: 바이트코드를 실행하고, 자주 실행되는 코드는 JIT가 기계어로 최적화한다.</li>
<li><strong>Garbage Collector(GC)</strong>: 더 이상 도달할 수 없는 Heap 객체의 메모리를 회수한다.</li>
</ul>
<h3 id="2-runtime-data-area">2. Runtime Data Area</h3>
<table>
<thead>
<tr>
<th>영역</th>
<th>저장하는 것</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>Method Area / Metaspace</td>
<td>클래스 메타데이터, 메서드 정보</td>
<td>클래스 수준 정보</td>
</tr>
<tr>
<td>Heap</td>
<td>객체, 배열</td>
<td>모든 스레드가 공유, GC 대상</td>
</tr>
<tr>
<td>Java Stack</td>
<td>메서드 호출 프레임, 지역 변수</td>
<td>스레드마다 독립</td>
</tr>
<tr>
<td>PC Register</td>
<td>현재 실행 위치</td>
<td>스레드마다 보유</td>
</tr>
<tr>
<td>Native Method Stack</td>
<td>네이티브 메서드 실행 정보</td>
<td>JVM 밖 코드 연동 시 사용</td>
</tr>
</tbody></table>
<h3 id="3-heap과-gc">3. Heap과 GC</h3>
<p>Heap은 흔히 새 객체가 들어오는 Young 영역(Eden, Survivor)과 오래 살아남은 객체가 이동하는 Old 영역으로 설명한다. GC는 참조할 수 없는 객체를 찾아 정리한다.</p>
<pre><code class="language-java">Stock stock = new Stock();
stock = null; // 이전 Stock 객체를 가리키는 참조가 없다면, 언젠가 GC 대상</code></pre>
<p>GC 시 일부 작업에서 애플리케이션 스레드가 잠시 멈출 수 있는데 이를 <strong>Stop The World(STW)</strong> 라고 한다. 따라서 객체를 무조건 많이 만든다고 즉시 오류가 나는 것은 아니지만, 대량의 불필요한 객체 생성은 성능에 영향을 줄 수 있다.</p>
<p><code>System.gc()</code>는 GC를 요청할 수는 있어도 “지금 즉시 실행하라”는 명령은 아니다. 메모리 관리는 JVM에 맡기고, 코드에서는 불필요한 객체 생성을 줄이고 외부 자원은 <code>try-with-resources</code>로 확실히 닫는 습관을 들이자.</p>
<hr />
<h2 id="오늘의-연결-복습">오늘의 연결 복습</h2>
<ol>
<li>백엔드는 IP/Port로 들어온 요청을 <strong>프로세스와 스레드</strong>에서 처리한다.</li>
<li>Java는 <code>.java</code>를 <code>.class</code>로 컴파일하고 <strong>JVM</strong>에서 실행한다.</li>
<li>변수와 자료형으로 데이터를 표현하고, 제어문으로 실행 흐름을 결정한다.</li>
<li>클래스와 객체로 데이터와 행동을 묶고, 패키지와 접근 제어자로 경계를 만든다.</li>
<li>OOP의 목표는 높은 응집도와 낮은 결합도로 <strong>변경하기 쉬운 코드</strong>를 만드는 것이다.</li>
</ol>
<h2 id="스스로-확인하기">스스로 확인하기</h2>
<ol>
<li><code>localhost:8080</code>에서 <code>localhost</code>와 <code>8080</code>은 각각 무엇인가?</li>
<li><code>int</code>와 <code>Integer</code>의 차이, 그리고 객체 값 비교에 <code>equals()</code>를 쓰는 이유는?</li>
<li><code>while</code>, <code>do-while</code>, <code>for-each</code>를 각각 언제 고를까?</li>
<li><code>private</code> 필드를 바로 공개하지 않고 메서드로 변경하는 이유는?</li>
<li><code>new</code>로 만든 객체는 Stack과 Heap 중 어디에 만들어지는가?</li>
</ol>
<h2 id="답변">답변</h2>
<ol>
<li><p><code>localhost</code>는 <strong>내 컴퓨터</strong>, <code>8080</code>은 내 컴퓨터 안에서 실행 중인 프로그램을 찾는 <strong>포트 번호</strong>입니다.</p>
</li>
<li><p><code>int</code>는 값 자체를 저장하는 기본형이고, <code>Integer</code>는 <code>int</code>를 객체로 감싼 참조형입니다. 객체는 <code>==</code>로 주소를 비교하므로, 값 비교에는 <code>equals()</code>를 사용합니다.</p>
</li>
<li><p><code>while</code>은 반복 횟수를 모를 때, <code>do-while</code>은 무조건 한 번은 실행해야 할 때, <code>for-each</code>는 배열이나 리스트의 모든 값을 순서대로 읽을 때 씁니다.</p>
</li>
<li><p><code>private</code>으로 필드를 숨기고 메서드로 변경하면, 잘못된 값이 들어오는 것을 막고 필요한 규칙을 적용할 수 있습니다.</p>
</li>
<li><p><code>new</code>로 만든 객체는 <strong>Heap 영역</strong>에 만들어집니다. 객체를 가리키는 변수(참조값)는 보통 메서드의 Stack 영역에 있습니다.</p>
</li>
</ol>