<blockquote>
<p>5일차에는 Java 객체를 데이터베이스에 저장하는 JPA부터 Repository, Entity 관계, 트랜잭션, REST API 문서화, Actuator까지 학습한다.<br />이전에 만든 <code>Controller → Service → Repository → Database</code> 구조에서 데이터 저장과 운영 관리 부분을 확장하는 과정이다.</p>
</blockquote>
<hr />
<h1 id="1-jpa">1. JPA</h1>
<h2 id="11-jpa란">1.1 JPA란?</h2>
<p>JPA는 <strong>Java Persistence API</strong>의 약자로, Java 객체와 관계형 데이터베이스를 연결하기 위한 표준 규칙이다.</p>
<pre><code class="language-text">Java 객체
   ↕ JPA
DB 테이블</code></pre>
<p>예를 들어 Java에는 <code>User</code> 객체가 있고 데이터베이스에는 <code>users</code> 테이블이 있다.</p>
<pre><code class="language-java">public class User {

    private Long id;
    private String name;
    private String email;
}</code></pre>
<pre><code class="language-text">users 테이블

id | name  | email
---|-------|-------------------
1  | alice | alice@example.com</code></pre>
<p>JPA는 다음 연결을 담당한다.</p>
<table>
<thead>
<tr>
<th>Java</th>
<th>데이터베이스</th>
</tr>
</thead>
<tbody><tr>
<td>클래스</td>
<td>테이블</td>
</tr>
<tr>
<td>객체</td>
<td>행(Row)</td>
</tr>
<tr>
<td>필드</td>
<td>열(Column)</td>
</tr>
<tr>
<td>객체의 ID</td>
<td>기본키</td>
</tr>
<tr>
<td>객체 관계</td>
<td>외래키 관계</td>
</tr>
</tbody></table>
<hr />
<h2 id="12-orm이란">1.2 ORM이란?</h2>
<p>JPA는 ORM 방식을 사용한다.</p>
<p>ORM은 <strong>Object-Relational Mapping</strong>, 즉 객체 관계 매핑이다.</p>
<pre><code class="language-text">Object     → Java 객체
Relational → 관계형 데이터베이스
Mapping    → 서로 연결</code></pre>
<p>JPA 없이 JDBC를 사용하면 SQL과 변환 코드를 직접 작성해야 한다.</p>
<pre><code class="language-java">String sql =
        &quot;INSERT INTO users(name, email) VALUES (?, ?)&quot;;

PreparedStatement statement =
        connection.prepareStatement(sql);

statement.setString(1, user.getName());
statement.setString(2, user.getEmail());
statement.executeUpdate();</code></pre>
<p>JPA를 사용하면 객체 중심으로 처리할 수 있다.</p>
<pre><code class="language-java">userRepository.save(user);</code></pre>
<p>JPA 구현체가 필요한 SQL을 생성한다.</p>
<pre><code class="language-sql">INSERT INTO users(name, email)
VALUES ('alice', 'alice@example.com');</code></pre>
<hr />
<h2 id="13-jpa-hibernate-spring-data-jpa의-차이">1.3 JPA, Hibernate, Spring Data JPA의 차이</h2>
<p>세 가지를 같은 기술로 생각하기 쉽지만 역할이 다르다.</p>
<table>
<thead>
<tr>
<th>기술</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>JPA</td>
<td>객체와 DB를 연결하는 표준</td>
</tr>
<tr>
<td>Hibernate</td>
<td>JPA 표준을 실제로 구현한 기술</td>
</tr>
<tr>
<td>Spring Data JPA</td>
<td>Repository를 통해 JPA를 쉽게 사용하도록 지원</td>
</tr>
<tr>
<td>JDBC</td>
<td>실제 SQL을 DB에 전달</td>
</tr>
<tr>
<td>H2, MySQL</td>
<td>데이터를 저장하는 DB</td>
</tr>
</tbody></table>
<p>전체 구조는 다음과 같다.</p>
<pre><code class="language-text">개발자가 작성한 코드
→ Spring Data JPA
→ JPA 표준
→ Hibernate
→ JDBC
→ 데이터베이스</code></pre>
<p>일상 예시로 보면 다음과 같다.</p>
<pre><code class="language-text">JPA             = 운전 규칙
Hibernate       = 규칙에 따라 운전하는 운전자
Spring Data JPA = 목적지만 입력하는 내비게이션
DB              = 목적지</code></pre>
<hr />
<h2 id="14-jpa-의존성">1.4 JPA 의존성</h2>
<p>Maven 프로젝트에서는 <code>pom.xml</code>에 다음 의존성을 추가한다.</p>
<pre><code class="language-xml">&lt;dependency&gt;
    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
    &lt;artifactId&gt;spring-boot-starter-data-jpa&lt;/artifactId&gt;
&lt;/dependency&gt;</code></pre>
<p>학습용 H2 데이터베이스:</p>
<pre><code class="language-xml">&lt;dependency&gt;
    &lt;groupId&gt;com.h2database&lt;/groupId&gt;
    &lt;artifactId&gt;h2&lt;/artifactId&gt;
    &lt;scope&gt;runtime&lt;/scope&gt;
&lt;/dependency&gt;</code></pre>
<hr />
<h2 id="15-entity-만들기">1.5 Entity 만들기</h2>
<p>Entity는 데이터베이스 테이블과 연결되는 Java 클래스다.</p>
<pre><code class="language-java">@Entity
@Table(name = &quot;users&quot;)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    protected User() {
    }

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}</code></pre>
<h3 id="entity"><code>@Entity</code></h3>
<pre><code class="language-java">@Entity</code></pre>
<p>JPA가 관리하는 클래스라는 뜻이다.</p>
<h3 id="table"><code>@Table</code></h3>
<pre><code class="language-java">@Table(name = &quot;users&quot;)</code></pre>
<p><code>User</code> Entity를 <code>users</code> 테이블과 연결한다.</p>
<h3 id="id"><code>@Id</code></h3>
<pre><code class="language-java">@Id
private Long id;</code></pre>
<p>이 필드가 테이블의 기본키임을 나타낸다.</p>
<h3 id="generatedvalue"><code>@GeneratedValue</code></h3>
<pre><code class="language-java">@GeneratedValue(strategy = GenerationType.IDENTITY)</code></pre>
<p>ID를 데이터베이스가 자동 생성한다.</p>
<pre><code class="language-text">저장 전 id = null
→ DB에 INSERT
→ 저장 후 id = 4</code></pre>
<hr />
<h2 id="16-기본-생성자가-필요한-이유">1.6 기본 생성자가 필요한 이유</h2>
<p>JPA는 조회한 데이터를 객체로 만들 때 기본 생성자를 사용한다.</p>
<pre><code class="language-java">protected User() {
}</code></pre>
<p>개념적으로 다음과 같은 작업을 수행한다.</p>
<pre><code class="language-java">User user = new User();

user.id = 1L;
user.name = &quot;alice&quot;;
user.email = &quot;alice@example.com&quot;;</code></pre>
<p>기본 생성자는 외부에서 함부로 사용하지 못하도록 <code>protected</code>로 만드는 경우가 많다.</p>
<p>Lombok을 사용한다면 다음과 같이 작성할 수 있다.</p>
<pre><code class="language-java">@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class User {
}</code></pre>
<hr />
<h2 id="17-column">1.7 <code>@Column</code></h2>
<p>필드와 DB 열의 세부 조건을 설정한다.</p>
<pre><code class="language-java">@Column(
    name = &quot;user_name&quot;,
    nullable = false,
    length = 30
)
private String name;</code></pre>
<table>
<thead>
<tr>
<th>속성</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>name</code></td>
<td>DB 열 이름</td>
</tr>
<tr>
<td><code>nullable</code></td>
<td><code>null</code> 허용 여부</td>
</tr>
<tr>
<td><code>length</code></td>
<td>문자열 최대 길이</td>
</tr>
<tr>
<td><code>unique</code></td>
<td>중복 허용 여부</td>
</tr>
<tr>
<td><code>insertable</code></td>
<td>INSERT 포함 여부</td>
</tr>
<tr>
<td><code>updatable</code></td>
<td>UPDATE 포함 여부</td>
</tr>
</tbody></table>
<p>예시:</p>
<pre><code class="language-java">@Column(nullable = false, unique = true)
private String email;</code></pre>
<hr />
<h2 id="18-entity-생명주기">1.8 Entity 생명주기</h2>
<p>Entity는 JPA가 관리하는지에 따라 상태가 달라진다.</p>
<h3 id="비영속">비영속</h3>
<p>일반 Java 객체로만 존재하는 상태다.</p>
<pre><code class="language-java">User user =
        new User(&quot;alice&quot;, &quot;alice@example.com&quot;);</code></pre>
<p>아직 DB와 관계가 없다.</p>
<h3 id="영속">영속</h3>
<p>JPA가 객체를 관리하는 상태다.</p>
<pre><code class="language-java">userRepository.save(user);</code></pre>
<h3 id="준영속">준영속</h3>
<p>한때 관리됐지만 현재 영속성 컨텍스트에서 분리된 상태다.</p>
<h3 id="삭제">삭제</h3>
<p>삭제 대상으로 등록된 상태다.</p>
<pre><code class="language-java">userRepository.delete(user);</code></pre>
<hr />
<h1 id="2-jpa-repository">2. JPA Repository</h1>
<h2 id="21-repository란">2.1 Repository란?</h2>
<p>Repository는 Entity를 데이터베이스에 저장하고 조회하는 통로다.</p>
<p>현재 프로젝트에서는 다음과 같이 작성한다.</p>
<pre><code class="language-java">public interface UserRepository
        extends JpaRepository&lt;User, Long&gt; {
}</code></pre>
<p>각 타입의 의미는 다음과 같다.</p>
<pre><code class="language-text">User → 관리할 Entity
Long → User 기본키의 자료형</code></pre>
<p>Spring Data JPA가 실행 중에 Repository 구현 객체를 자동으로 만든다.</p>
<pre><code class="language-text">UserRepository 인터페이스
→ Spring Data JPA가 Proxy 생성
→ Bean으로 등록
→ UserService에 DI</code></pre>
<hr />
<h2 id="22-기본-crud-메서드">2.2 기본 CRUD 메서드</h2>
<p><code>JpaRepository</code>를 상속하면 기본 메서드를 사용할 수 있다.</p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>기능</th>
</tr>
</thead>
<tbody><tr>
<td><code>save(entity)</code></td>
<td>추가 또는 수정</td>
</tr>
<tr>
<td><code>findAll()</code></td>
<td>전체 조회</td>
</tr>
<tr>
<td><code>findById(id)</code></td>
<td>ID 조회</td>
</tr>
<tr>
<td><code>delete(entity)</code></td>
<td>Entity 삭제</td>
</tr>
<tr>
<td><code>deleteById(id)</code></td>
<td>ID로 삭제</td>
</tr>
<tr>
<td><code>existsById(id)</code></td>
<td>존재 여부 확인</td>
</tr>
<tr>
<td><code>count()</code></td>
<td>전체 개수</td>
</tr>
<tr>
<td><code>saveAll(list)</code></td>
<td>여러 Entity 저장</td>
</tr>
</tbody></table>
<h3 id="전체-조회">전체 조회</h3>
<pre><code class="language-java">List&lt;User&gt; users =
        userRepository.findAll();</code></pre>
<p>예상 SQL:</p>
<pre><code class="language-sql">SELECT *
FROM users;</code></pre>
<h3 id="id-조회">ID 조회</h3>
<pre><code class="language-java">Optional&lt;User&gt; user =
        userRepository.findById(1L);</code></pre>
<p>예상 SQL:</p>
<pre><code class="language-sql">SELECT *
FROM users
WHERE id = 1;</code></pre>
<h3 id="추가">추가</h3>
<pre><code class="language-java">User user =
        new User(&quot;skala&quot;, &quot;skala@example.com&quot;);

User savedUser =
        userRepository.save(user);</code></pre>
<p>ID가 없으면 새로운 Entity로 판단해 <code>INSERT</code>한다.</p>
<h3 id="삭제-1">삭제</h3>
<pre><code class="language-java">userRepository.deleteById(1L);</code></pre>
<p>예상 SQL:</p>
<pre><code class="language-sql">DELETE FROM users
WHERE id = 1;</code></pre>
<hr />
<h2 id="23-save의-insert와-update">2.3 <code>save()</code>의 INSERT와 UPDATE</h2>
<p><code>save()</code>는 상황에 따라 INSERT 또는 UPDATE를 처리한다.</p>
<h3 id="id가-없는-새-entity">ID가 없는 새 Entity</h3>
<pre><code class="language-java">User user =
        new User(&quot;alice&quot;, &quot;alice@example.com&quot;);</code></pre>
<pre><code class="language-text">id = null
→ 새로운 Entity
→ INSERT</code></pre>
<h3 id="이미-존재하는-entity">이미 존재하는 Entity</h3>
<pre><code class="language-java">User user =
        userRepository.findById(1L)
                .orElseThrow();

user.changeName(&quot;alicia&quot;);

userRepository.save(user);</code></pre>
<pre><code class="language-text">기존 ID 존재
→ 기존 Entity
→ UPDATE</code></pre>
<hr />
<h2 id="24-메서드-이름으로-쿼리-생성">2.4 메서드 이름으로 쿼리 생성</h2>
<p>Spring Data JPA는 Repository의 메서드 이름을 분석해 쿼리를 만든다.</p>
<pre><code class="language-java">List&lt;User&gt; findByName(String name);</code></pre>
<p>해석:</p>
<pre><code class="language-text">find → 조회
By   → 조건 시작
Name → name 필드가 조건</code></pre>
<p>예상 SQL:</p>
<pre><code class="language-sql">SELECT *
FROM users
WHERE name = ?;</code></pre>
<p>호출:</p>
<pre><code class="language-java">userRepository.findByName(&quot;alice&quot;);</code></pre>
<h3 id="여러-조건">여러 조건</h3>
<pre><code class="language-java">Optional&lt;User&gt; findByEmail(String email);

List&lt;User&gt; findByNameContaining(String keyword);

List&lt;User&gt; findByNameAndEmail(
        String name,
        String email
);

boolean existsByEmail(String email);

long countByName(String name);</code></pre>
<table>
<thead>
<tr>
<th>키워드</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>And</code></td>
<td>두 조건 모두 만족</td>
</tr>
<tr>
<td><code>Or</code></td>
<td>둘 중 하나 만족</td>
</tr>
<tr>
<td><code>Containing</code></td>
<td>문자열 포함</td>
</tr>
<tr>
<td><code>StartingWith</code></td>
<td>특정 문자열로 시작</td>
</tr>
<tr>
<td><code>GreaterThan</code></td>
<td>기준보다 큼</td>
</tr>
<tr>
<td><code>LessThan</code></td>
<td>기준보다 작음</td>
</tr>
<tr>
<td><code>OrderBy</code></td>
<td>정렬</td>
</tr>
<tr>
<td><code>ExistsBy</code></td>
<td>존재 여부</td>
</tr>
</tbody></table>
<p>Entity에 존재하지 않는 필드 이름을 사용하면 애플리케이션 실행 중 오류가 발생한다.</p>
<pre><code class="language-java">findByUsername(String username);</code></pre>
<p><code>User</code>에 <code>username</code> 필드가 없다면 잘못된 메서드다.</p>
<hr />
<h2 id="25-query">2.5 <code>@Query</code></h2>
<p>메서드 이름만으로 표현하기 어려운 조회는 직접 쿼리를 작성할 수 있다.</p>
<pre><code class="language-java">@Query(&quot;&quot;&quot;
    select u
    from User u
    where u.email = :email
&quot;&quot;&quot;)
Optional&lt;User&gt; findUserByEmail(
        @Param(&quot;email&quot;) String email
);</code></pre>
<p>여기서 <code>User</code>는 테이블 이름이 아니라 Entity 클래스 이름이고, <code>u.email</code>은 Entity 필드다.</p>
<p>이러한 객체 중심 쿼리를 JPQL이라고 한다.</p>
<pre><code class="language-text">SQL  → 테이블과 열을 대상으로 작성
JPQL → Entity와 필드를 대상으로 작성</code></pre>
<hr />
<h2 id="26-paging과-sorting">2.6 Paging과 Sorting</h2>
<p>데이터가 많을 때 전체를 한 번에 가져오지 않고 페이지 단위로 조회한다.</p>
<pre><code class="language-java">Pageable pageable =
        PageRequest.of(0, 10);

Page&lt;User&gt; users =
        userRepository.findAll(pageable);</code></pre>
<pre><code class="language-text">0 → 첫 번째 페이지
10 → 한 페이지에 10개</code></pre>
<p>정렬도 추가할 수 있다.</p>
<pre><code class="language-java">Pageable pageable = PageRequest.of(
        0,
        10,
        Sort.by(&quot;name&quot;).ascending()
);</code></pre>
<hr />
<h1 id="3-관계-매핑-및-entity-manager">3. 관계 매핑 및 Entity Manager</h1>
<h1 id="31-관계-매핑이란">3.1 관계 매핑이란?</h1>
<p>실제 데이터는 하나의 테이블에만 존재하지 않는다.</p>
<p>예를 들어 사용자 한 명이 여러 주문을 만들 수 있다.</p>
<pre><code class="language-text">User 1명
→ Order 여러 개</code></pre>
<p>DB에서는 외래키로 표현한다.</p>
<pre><code class="language-text">users
id | name

orders
id | product | user_id</code></pre>
<p>Java 객체에서는 객체 참조로 표현한다.</p>
<pre><code class="language-java">class Order {
    private User user;
}</code></pre>
<p>JPA는 객체 관계와 DB 외래키 관계를 연결한다.</p>
<hr />
<h2 id="32-관계-종류">3.2 관계 종류</h2>
<table>
<thead>
<tr>
<th>관계</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>@OneToOne</code></td>
<td>일대일</td>
<td>사용자와 프로필</td>
</tr>
<tr>
<td><code>@OneToMany</code></td>
<td>일대다</td>
<td>사용자와 주문 목록</td>
</tr>
<tr>
<td><code>@ManyToOne</code></td>
<td>다대일</td>
<td>여러 주문과 한 사용자</td>
</tr>
<tr>
<td><code>@ManyToMany</code></td>
<td>다대다</td>
<td>학생과 수업</td>
</tr>
</tbody></table>
<hr />
<h2 id="33-다대일-관계">3.3 다대일 관계</h2>
<p>여러 주문이 한 사용자를 참조하는 경우다.</p>
<pre><code class="language-java">@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private String productName;

    @ManyToOne
    @JoinColumn(name = &quot;user_id&quot;)
    private User user;
}</code></pre>
<pre><code class="language-text">Order 객체의 user
↔ orders 테이블의 user_id 외래키</code></pre>
<p>일반적으로 외래키를 가진 쪽이 관계의 주인이다.</p>
<pre><code class="language-text">orders 테이블이 user_id 보유
→ Order가 관계의 주인</code></pre>
<hr />
<h2 id="34-일대다-관계">3.4 일대다 관계</h2>
<p>사용자 입장에서 자신이 만든 주문 목록을 표현할 수 있다.</p>
<pre><code class="language-java">@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = &quot;user&quot;)
    private List&lt;Order&gt; orders =
            new ArrayList&lt;&gt;();
}</code></pre>
<p><code>mappedBy = &quot;user&quot;</code>는 <code>Order</code> 클래스의 <code>user</code> 필드가 관계를 관리한다는 뜻이다.</p>
<pre><code class="language-java">@ManyToOne
private User user;</code></pre>
<pre><code class="language-text">User.orders
→ 관계를 조회하는 반대편

Order.user
→ 외래키를 관리하는 관계의 주인</code></pre>
<hr />
<h2 id="35-양방향-관계-설정">3.5 양방향 관계 설정</h2>
<p>Java 객체의 양쪽 관계를 모두 맞춰주는 편의 메서드를 만들 수 있다.</p>
<pre><code class="language-java">public void addOrder(Order order) {
    orders.add(order);
    order.setUser(this);
}</code></pre>
<p>다음처럼 사용한다.</p>
<pre><code class="language-java">User user = new User(&quot;alice&quot;);
Order order = new Order(&quot;노트북&quot;);

user.addOrder(order);</code></pre>
<p>그러면 양쪽 객체가 일치한다.</p>
<pre><code class="language-text">user.orders에 order 추가
order.user에 user 지정</code></pre>
<hr />
<h2 id="36-fetchtype">3.6 FetchType</h2>
<p>연관된 Entity를 언제 조회할지 결정한다.</p>
<h3 id="즉시-로딩">즉시 로딩</h3>
<pre><code class="language-java">@ManyToOne(fetch = FetchType.EAGER(디폴트))</code></pre>
<p>Order를 조회할 때 User도 바로 조회한다.</p>
<pre><code class="language-text">Order 조회
→ User도 즉시 조회</code></pre>
<h3 id="지연-로딩">지연 로딩</h3>
<pre><code class="language-java">@ManyToOne(fetch = FetchType.LAZY()</code></pre>
<p>User가 실제로 필요할 때 조회한다.</p>
<pre><code class="language-text">Order 조회
→ User는 아직 조회하지 않음
→ order.getUser() 사용
→ User 조회</code></pre>
<p>실무에서는 예상하지 못한 대량 조회를 예방하기 위해 지연 로딩을 기본으로 고려하는 경우가 많다.</p>
<hr />
<h2 id="37-cascade">3.7 Cascade</h2>
<p>부모 Entity의 작업을 자식 Entity에도 전달한다.</p>
<pre><code class="language-java">@OneToMany(
    mappedBy = &quot;user&quot;,
    cascade = CascadeType.ALL
)
private List&lt;Order&gt; orders;</code></pre>
<pre><code class="language-text">User 저장
→ 연결된 Order도 저장

User 삭제
→ 연결된 Order도 삭제 가능</code></pre>
<p>무조건 <code>ALL</code>을 사용하는 것은 위험하다. 자식의 생명주기가 부모에게 완전히 종속되는 경우에 사용해야 한다.</p>
<hr />
<h2 id="38-entitymanager란">3.8 EntityManager란?</h2>
<p><code>EntityManager</code>는 JPA의 핵심 인터페이스로 Entity의 저장, 조회, 수정, 삭제를 관리한다.</p>
<pre><code class="language-text">Repository
→ 내부적으로 EntityManager 사용
→ EntityManager가 Hibernate에 작업 요청
→ SQL 실행</code></pre>
<p>직접 사용할 수도 있다.</p>
<pre><code class="language-java">@PersistenceContext
private EntityManager entityManager;</code></pre>
<p>주요 메서드는 다음과 같다.</p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>persist()</code></td>
<td>새 Entity 저장</td>
</tr>
<tr>
<td><code>find()</code></td>
<td>기본키로 조회</td>
</tr>
<tr>
<td><code>remove()</code></td>
<td>Entity 삭제</td>
</tr>
<tr>
<td><code>merge()</code></td>
<td>준영속 Entity 병합</td>
</tr>
<tr>
<td><code>flush()</code></td>
<td>변경 내용을 DB에 반영</td>
</tr>
<tr>
<td><code>clear()</code></td>
<td>영속성 컨텍스트 초기화</td>
</tr>
<tr>
<td><code>detach()</code></td>
<td>특정 Entity 분리</td>
</tr>
</tbody></table>
<h3 id="저장">저장</h3>
<pre><code class="language-java">entityManager.persist(user);</code></pre>
<h3 id="조회">조회</h3>
<pre><code class="language-java">User user =
        entityManager.find(User.class, 1L);</code></pre>
<h3 id="삭제-2">삭제</h3>
<pre><code class="language-java">entityManager.remove(user);</code></pre>
<hr />
<h2 id="39-영속성-컨텍스트">3.9 영속성 컨텍스트</h2>
<p>영속성 컨텍스트는 EntityManager가 Entity를 관리하는 공간이다.</p>
<pre><code class="language-text">Java 애플리케이션
→ 영속성 컨텍스트
→ 데이터베이스</code></pre>
<p>쉽게 말하면 DB와 Java 객체 사이에 있는 Entity 작업 공간이다.</p>
<hr />
<h2 id="310-1차-캐시">3.10 1차 캐시</h2>
<p>같은 트랜잭션에서 같은 Entity를 두 번 조회하면 영속성 컨텍스트가 보관한 객체를 사용할 수 있다.</p>
<pre><code class="language-java">User user1 =
        entityManager.find(User.class, 1L);

User user2 =
        entityManager.find(User.class, 1L);</code></pre>
<pre><code class="language-text">첫 번째 조회 → DB 조회 후 1차 캐시에 저장
두 번째 조회 → 1차 캐시에서 반환</code></pre>
<pre><code class="language-java">user1 == user2 // 같은 영속성 컨텍스트에서는 true 가능</code></pre>
<hr />
<h2 id="311-변경-감지">3.11 변경 감지</h2>
<p>JPA가 관리하는 Entity의 값이 변경되면 트랜잭션 종료 시 변경 내용을 감지한다.</p>
<pre><code class="language-java">@Transactional
public void updateUser(Long id, String name) {
    User user = userRepository.findById(id)
            .orElseThrow();

    user.changeName(name);
}</code></pre>
<p><code>save()</code>를 다시 호출하지 않아도 변경 감지가 작동할 수 있다.</p>
<pre><code class="language-text">Entity 조회
→ 최초 상태 저장
→ name 변경
→ 트랜잭션 종료
→ 최초 상태와 현재 상태 비교
→ UPDATE 실행</code></pre>
<p>이를 Dirty Checking이라고 한다.</p>
<hr />
<h2 id="312-flush">3.12 Flush</h2>
<p>Flush는 영속성 컨텍스트에 쌓인 변경 내용을 DB에 SQL로 반영하는 작업이다.</p>
<pre><code class="language-java">entityManager.flush();</code></pre>
<p>주의할 점은 Flush가 트랜잭션 확정을 뜻하지 않는다는 것이다.</p>
<pre><code class="language-text">flush  → SQL을 DB에 전달
commit → 트랜잭션을 최종 확정</code></pre>
<hr />
<h1 id="4-transactional">4. Transactional</h1>
<h2 id="41-트랜잭션이란">4.1 트랜잭션이란?</h2>
<p>트랜잭션은 여러 DB 작업을 하나의 작업 단위로 묶는 것이다.</p>
<p>은행 이체를 생각하면 쉽다.</p>
<pre><code class="language-text">1. A 계좌에서 10,000원 차감
2. B 계좌에 10,000원 추가</code></pre>
<p>두 작업이 모두 성공해야 한다.</p>
<pre><code class="language-text">모두 성공 → COMMIT
중간 실패 → ROLLBACK</code></pre>
<p>A 계좌에서 돈만 빠지고 B 계좌에는 들어가지 않는 상황을 막는 것이 트랜잭션이다.</p>
<hr />
<h2 id="42-acid">4.2 ACID</h2>
<p>트랜잭션의 주요 특징을 ACID라고 한다.</p>
<table>
<thead>
<tr>
<th>특징</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Atomicity</td>
<td>작업이 전부 성공하거나 전부 실패</td>
</tr>
<tr>
<td>Consistency</td>
<td>처리 전후 데이터 규칙 유지</td>
</tr>
<tr>
<td>Isolation</td>
<td>동시에 실행되는 트랜잭션이 서로 방해하지 않음</td>
</tr>
<tr>
<td>Durability</td>
<td>커밋된 결과가 유지됨</td>
</tr>
</tbody></table>
<hr />
<h2 id="43-transactional">4.3 <code>@Transactional</code></h2>
<p>Spring에서는 메서드 또는 클래스에 <code>@Transactional</code>을 붙인다.</p>
<pre><code class="language-java">@Transactional
public UserResponse createUser(
        UserRequest request
) {
    User user = new User(
            request.name(),
            request.email()
    );

    User savedUser =
            userRepository.save(user);

    return UserResponse.from(savedUser);
}</code></pre>
<p>정상 종료:</p>
<pre><code class="language-text">트랜잭션 시작
→ 사용자 저장
→ 정상 반환
→ COMMIT</code></pre>
<p>예외 발생:</p>
<pre><code class="language-text">트랜잭션 시작
→ 사용자 저장
→ 예외 발생
→ ROLLBACK</code></pre>
<hr />
<h2 id="44-트랜잭션을-service에-두는-이유">4.4 트랜잭션을 Service에 두는 이유</h2>
<p>Controller는 HTTP 요청과 응답을 담당하고, Service는 하나의 업무 단위를 담당한다.</p>
<pre><code class="language-text">Controller
→ 요청 해석

Service
→ 업무 처리와 트랜잭션

Repository
→ DB 접근</code></pre>
<p>따라서 일반적으로 Service 메서드에 트랜잭션을 적용한다.</p>
<pre><code class="language-java">@Service
@Transactional(readOnly = true)
public class UserService {

    @Transactional
    public UserResponse createUser(
            UserRequest request
    ) {
        // 쓰기 작업
    }

    public List&lt;UserResponse&gt; getUsers() {
        // 읽기 작업
    }
}</code></pre>
<hr />
<h2 id="45-readonly--true">4.5 <code>readOnly = true</code></h2>
<p>조회 전용 트랜잭션에 사용한다.</p>
<pre><code class="language-java">@Transactional(readOnly = true)
public List&lt;User&gt; getAllUsers() {
    return userRepository.findAll();
}</code></pre>
<p>의미는 다음과 같다.</p>
<pre><code class="language-text">이 메서드는 데이터를 변경하지 않고 조회만 한다.</code></pre>
<p>JPA 구현체와 DB가 조회에 맞게 최적화할 수 있고, 의도를 명확하게 표현할 수 있다.</p>
<hr />
<h2 id="46-기본-롤백-규칙">4.6 기본 롤백 규칙</h2>
<p>Spring은 기본적으로 다음 예외에서 롤백한다.</p>
<pre><code class="language-text">RuntimeException과 그 하위 예외
Error</code></pre>
<p>체크 예외까지 롤백하려면 지정할 수 있다.</p>
<pre><code class="language-java">@Transactional(rollbackFor = Exception.class)</code></pre>
<hr />
<h2 id="47-spring-트랜잭션과-proxy">4.7 Spring 트랜잭션과 Proxy</h2>
<p><code>@Transactional</code>은 Spring AOP Proxy를 사용한다.</p>
<pre><code class="language-text">Controller
→ Transaction Proxy
→ 실제 Service</code></pre>
<p>동작 과정:</p>
<pre><code class="language-text">1. Proxy가 트랜잭션 시작
2. 실제 Service 메서드 실행
3. 정상 종료 시 COMMIT
4. 예외 발생 시 ROLLBACK</code></pre>
<hr />
<h2 id="48-같은-클래스-내부-호출-문제">4.8 같은 클래스 내부 호출 문제</h2>
<pre><code class="language-java">@Service
public class UserService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
        // DB 작업
    }
}</code></pre>
<p>외부에서 <code>methodA()</code>를 호출한 후 내부에서 <code>methodB()</code>를 호출하면 Proxy를 다시 거치지 않는다.</p>
<pre><code class="language-text">외부
→ Proxy
→ methodA()
   → this.methodB()
      Proxy 우회</code></pre>
<p>따라서 <code>methodB()</code>의 트랜잭션이 예상대로 적용되지 않을 수 있다. 트랜잭션이 필요한 기능을 별도의 Spring Bean으로 분리하는 방법을 고려한다.</p>
<hr />
<h2 id="49-트랜잭션-내부에서-예외를-삼키는-문제">4.9 트랜잭션 내부에서 예외를 삼키는 문제</h2>
<pre><code class="language-java">@Transactional
public void createUser() {
    try {
        userRepository.save(user);
        throw new RuntimeException();
    } catch (Exception e) {
        log.error(&quot;오류 발생&quot;, e);
    }
}</code></pre>
<p>예외를 내부에서 잡고 정상 종료하면 Spring은 성공한 것으로 판단할 수 있다.</p>
<pre><code class="language-text">예외 발생
→ catch에서 처리
→ 메서드 정상 종료
→ COMMIT 가능</code></pre>
<p>롤백하려면 예외를 다시 던져야 한다.</p>
<pre><code class="language-java">catch (Exception e) {
    log.error(&quot;오류 발생&quot;, e);
    throw e;
}</code></pre>
<h2 id="410-트랜잭션-동시성-제어와-lock-정리">4.10 트랜잭션 동시성 제어와 Lock 정리</h2>
<h3 id="4101-동시성-제어란">4.10.1 동시성 제어란?</h3>
<p>여러 사용자나 서버가 <strong>같은 데이터를 동시에 조회하고 수정할 때 발생하는 오류를 막는 것</strong>입니다.</p>
<p>중요한 점:</p>
<blockquote>
<p>@Transactional을 붙였다고 동시성 문제가 자동으로 해결되는 것은 아니다.</p>
</blockquote>
<p>예를 들어 재고가 10개인 상품을 두 사용자가 동시에 1개씩 주문한다고 가정합니다.</p>
<pre><code class="language-text">트랜잭션 A: 재고 10 조회 → 9로 변경
트랜잭션 B: 재고 10 조회 → 9로 변경</code></pre>
<p>정상 결과는 8이어야 하지만, 두 트랜잭션이 모두 9를 저장하면 최종 재고는 9가 됩니다. 이를 <strong>갱신 손실(Lost Update)</strong>이라고 합니다.</p>
<hr />
<h3 id="4102-대표적인-동시성-문제">4.10.2 대표적인 동시성 문제</h3>
<table>
<thead>
<tr>
<th>문제</th>
<th>설명</th>
<th>간단한 예</th>
</tr>
</thead>
<tbody><tr>
<td>Lost Update</td>
<td>나중에 저장한 값이 먼저 저장한 값을 덮어씀</td>
<td>두 사용자가 동시에 포인트 증가</td>
</tr>
<tr>
<td>Dirty Read</td>
<td>아직 커밋되지 않은 값을 다른 트랜잭션이 읽음</td>
<td>취소될 수도 있는 결제 정보를 미리 조회</td>
</tr>
<tr>
<td>Non-repeatable Read</td>
<td>한 트랜잭션에서 같은 데이터를 두 번 읽었는데 값이 달라짐</td>
<td>처음에는 재고 10, 다시 읽으니 8</td>
</tr>
<tr>
<td>Phantom Read</td>
<td>같은 조건으로 다시 조회했는데 새로운 행이 나타남</td>
<td>주문 목록을 다시 조회하니 주문이 추가됨</td>
</tr>
<tr>
<td>Race Condition</td>
<td>실행 순서에 따라 비즈니스 결과가 달라짐</td>
<td>두 사용자가 마지막 상품을 동시에 주문</td>
</tr>
</tbody></table>
<p>실제 발생 여부는 DB의 <strong>트랜잭션 격리 수준(Isolation Level)</strong>에 따라서도 달라집니다.</p>
<hr />
<h3 id="4103-jpa의-lock-방식">4.10.3 JPA의 Lock 방식</h3>
<p>JPA에서는 대표적으로 두 가지 방법을 제공합니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>낙관적 Lock</th>
<th>비관적 Lock</th>
</tr>
</thead>
<tbody><tr>
<td>기본 생각</td>
<td>충돌이 드물 것이라고 가정</td>
<td>충돌이 자주 발생한다고 가정</td>
</tr>
<tr>
<td>DB Lock</td>
<td>직접 걸지 않음</td>
<td>DB 레코드에 직접 걸음</td>
</tr>
<tr>
<td>충돌 처리</td>
<td>커밋할 때 버전 비교</td>
<td>조회할 때부터 다른 작업을 대기시킴</td>
</tr>
<tr>
<td>충돌 발생 시</td>
<td>예외 발생 후 재시도</td>
<td>다른 트랜잭션이 대기</td>
</tr>
<tr>
<td>장점</td>
<td>성능과 동시 처리량이 좋음</td>
<td>데이터 충돌을 확실하게 방지</td>
</tr>
<tr>
<td>단점</td>
<td>충돌 시 재시도 로직 필요</td>
<td>대기, 성능 저하, 데드락 가능</td>
</tr>
<tr>
<td>적합한 상황</td>
<td>조회가 많고 충돌이 적을 때</td>
<td>재고처럼 충돌이 빈번할 때</td>
</tr>
</tbody></table>
<hr />
<h3 id="4104-낙관적-lockoptimistic-lock">4.10.4 낙관적 Lock(Optimistic Lock)</h3>
<p>낙관적 Lock은 DB 레코드를 잠그지 않고 자유롭게 읽고 수정합니다. 대신 엔티티의 <strong>버전 번호</strong>를 이용해 커밋할 때 충돌을 검사합니다.</p>
<h4 id="엔티티-설정">엔티티 설정</h4>
<pre><code class="language-java">@Entity
public class Product {

    @Id
    private Long id;

    private int stock;

    @Version
    private Long version;
}</code></pre>
<p>@Version 필드가 핵심입니다.</p>
<h4 id="동작-과정">동작 과정</h4>
<p>처음 상태:</p>
<pre><code class="language-text">재고: 10
version: 0</code></pre>
<p>두 트랜잭션이 동시에 조회합니다.</p>
<pre><code class="language-text">A: 재고 10, version 0 조회
B: 재고 10, version 0 조회</code></pre>
<p>A가 먼저 커밋합니다.</p>
<pre><code class="language-text">재고: 9
version: 1</code></pre>
<p>B는 자신이 읽었던 version 0으로 수정하려고 합니다. 하지만 DB는 이미 version 1이므로 충돌이 감지됩니다.</p>
<pre><code class="language-text">B의 수정 실패
→ OptimisticLockException 발생
→ 필요하면 전체 트랜잭션 재시도</code></pre>
<p>실제 SQL은 개념적으로 다음과 비슷합니다.</p>
<pre><code class="language-sql">UPDATE product
SET stock = 9,
    version = 1
WHERE id = 1
  AND version = 0;</code></pre>
<p>version = 0인 데이터가 이미 없으면 수정된 행이 0개가 되고, JPA가 충돌로 판단합니다.</p>
<h4 id="장단점">장단점</h4>
<ul>
<li>DB를 직접 잠그지 않아 읽기가 많은 환경에서 성능이 좋습니다.</li>
<li>충돌이 발생하면 예외를 처리하고 트랜잭션을 처음부터 다시 실행해야 합니다.</li>
<li>충돌이 너무 자주 발생하면 재시도가 많아져 오히려 비효율적입니다.</li>
</ul>
<hr />
<h3 id="4105-비관적-lockpessimistic-lock">4.10.5 비관적 Lock(Pessimistic Lock)</h3>
<p>비관적 Lock은 데이터를 조회할 때부터 DB 레코드를 잠급니다.</p>
<pre><code class="language-text">트랜잭션 A가 상품 조회 및 Lock 획득
    ↓
트랜잭션 B가 같은 상품을 수정하려고 함
    ↓
A가 끝날 때까지 B는 대기
    ↓
A가 commit 또는 rollback
    ↓
Lock 해제 후 B 실행</code></pre>
<p>DB에서는 일반적으로 SELECT FOR UPDATE 형태로 동작합니다.</p>
<h4 id="repository-예시">Repository 예시</h4>
<pre><code class="language-java">public interface ProductRepository
        extends JpaRepository&lt;Product, Long&gt; {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query(&quot;select p from Product p where p.id = :id&quot;)
    Optional&lt;Product&gt; findByIdWithWriteLock(
            @Param(&quot;id&quot;) Long id
    );
}</code></pre>
<p>서비스에서는 반드시 트랜잭션 안에서 사용합니다.</p>
<pre><code class="language-java">@Transactional
public void decreaseStock(Long productId, int quantity) {
    Product product = productRepository
            .findByIdWithWriteLock(productId)
            .orElseThrow();

    product.decreaseStock(quantity);
}</code></pre>
<p>Lock은 트랜잭션이 commit 또는 rollback될 때 해제됩니다.</p>
<h4 id="lock-모드">Lock 모드</h4>
<table>
<thead>
<tr>
<th>모드</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>PESSIMISTIC_WRITE</td>
<td>다른 트랜잭션의 수정을 막고 기다리게 함</td>
</tr>
<tr>
<td>PESSIMISTIC_READ</td>
<td>다른 트랜잭션의 수정을 제한하며, 읽기 허용 여부는 DB에 따라 다름</td>
</tr>
<tr>
<td>NONE</td>
<td>별도의 Lock 없이 일반 조회</td>
</tr>
</tbody></table>
<h4 id="장단점-1">장단점</h4>
<ul>
<li>재고처럼 동시에 수정될 가능성이 높은 데이터의 정합성을 지키기 좋습니다.</li>
<li>다른 트랜잭션이 기다리므로 처리 속도가 느려질 수 있습니다.</li>
<li>여러 트랜잭션이 서로의 Lock을 기다리면 데드락(Deadlock)이 발생할 수 있습니다.</li>
<li>Lock을 잡은 상태에서 외부 API 호출이나 오래 걸리는 작업을 하지 말고 트랜잭션 범위를 짧게 유지해야 합니다.</li>
</ul>
<hr />
<h3 id="4106-어떤-lock을-선택할까">4.10.6 어떤 Lock을 선택할까?</h3>
<pre><code class="language-text">충돌이 드물다
→ 낙관적 Lock
→ @Version으로 충돌 감지
→ 실패하면 재시도

충돌이 자주 발생한다
→ 비관적 Lock
→ 조회 시 DB Lock
→ 다른 트랜잭션은 대기</code></pre>
<p>예시:</p>
<table>
<thead>
<tr>
<th>상황</th>
<th>선택 예시</th>
</tr>
</thead>
<tbody><tr>
<td>게시글 수정</td>
<td>낙관적 Lock</td>
</tr>
<tr>
<td>사용자 프로필 수정</td>
<td>낙관적 Lock</td>
</tr>
<tr>
<td>일반적인 상품 정보 수정</td>
<td>낙관적 Lock</td>
</tr>
<tr>
<td>한정 수량 재고 차감</td>
<td>비관적 Lock 고려</td>
</tr>
<tr>
<td>좌석 예약</td>
<td>비관적 Lock 고려</td>
</tr>
<tr>
<td>동시에 요청이 몰리는 쿠폰 발급</td>
<td>비관적 Lock 또는 별도의 원자적 처리 고려</td>
</tr>
</tbody></table>
<p>마지막으로 Java의 synchronized와 DB Lock은 적용 범위가 다릅니다.</p>
<pre><code class="language-text">synchronized: 같은 JVM 프로세스 안의 스레드 제어
JPA/DB Lock: 여러 서버와 프로세스가 함께 접근하는 DB 데이터 제어</code></pre>
<p>따라서 서버가 여러 대인 실제 서비스에서는 synchronized만으로 DB 동시성 문제를 완전히 해결할 수 없습니다.</p>
<hr />
<h1 id="5-rest-api-문서화">5. REST API 문서화</h1>
<h2 id="51-api-문서가-필요한-이유">5.1 API 문서가 필요한 이유</h2>
<p>REST API를 만든 사람과 사용하는 사람이 다를 수 있다.</p>
<p>API 사용자는 다음 정보를 알아야 한다.</p>
<ul>
<li>요청 URL</li>
<li>HTTP Method</li>
<li>Path Variable</li>
<li>Query Parameter</li>
<li>Request Body</li>
<li>Response Body</li>
<li>상태 코드</li>
<li>입력값 제약조건</li>
<li>오류 응답</li>
</ul>
<p>문서가 없다면 Controller 코드를 직접 읽어야 한다.</p>
<hr />
<h2 id="52-openapi와-swagger">5.2 OpenAPI와 Swagger</h2>
<h3 id="openapi">OpenAPI</h3>
<p>REST API의 구조를 표현하는 표준 명세다.</p>
<pre><code class="language-text">어떤 URL이 있는가?
어떤 값을 보내야 하는가?
어떤 응답이 돌아오는가?</code></pre>
<h3 id="swagger-ui">Swagger UI</h3>
<p>OpenAPI 명세를 웹 화면으로 보여주고 API를 직접 실행할 수 있게 하는 도구다.</p>
<pre><code class="language-text">OpenAPI → API 설명서 규칙
Swagger UI → 설명서를 보여주는 화면</code></pre>
<p>Spring Boot에서는 보통 <code>springdoc-openapi</code>를 사용한다.</p>
<hr />
<h2 id="53-의존성-추가">5.3 의존성 추가</h2>
<p>Spring Boot 버전에 맞는 <code>springdoc-openapi</code> Starter를 추가한다.</p>
<pre><code class="language-xml">&lt;dependency&gt;
    &lt;groupId&gt;org.springdoc&lt;/groupId&gt;
    &lt;artifactId&gt;
        springdoc-openapi-starter-webmvc-ui
    &lt;/artifactId&gt;
    &lt;version&gt;프로젝트와 호환되는 버전&lt;/version&gt;
&lt;/dependency&gt;</code></pre>
<p>서버 실행 후 일반적으로 다음 주소에서 확인한다.</p>
<pre><code class="language-text">http://localhost:8080/swagger-ui/index.html</code></pre>
<p>OpenAPI JSON:</p>
<pre><code class="language-text">http://localhost:8080/v3/api-docs</code></pre>
<hr />
<h2 id="54-controller-문서화">5.4 Controller 문서화</h2>
<pre><code class="language-java">@Tag(
    name = &quot;User API&quot;,
    description = &quot;사용자 관리 API&quot;
)
@RestController
@RequestMapping(&quot;/api&quot;)
public class UserController {
}</code></pre>
<h3 id="operation"><code>@Operation</code></h3>
<pre><code class="language-java">@Operation(
    summary = &quot;전체 사용자 조회&quot;,
    description = &quot;등록된 사용자를 모두 조회합니다.&quot;
)
@GetMapping(&quot;/users&quot;)
public List&lt;UserResponse&gt; getUsers() {
    return userService.getAllUsers();
}</code></pre>
<h3 id="parameter"><code>@Parameter</code></h3>
<pre><code class="language-java">@GetMapping(&quot;/users/{id}&quot;)
public UserResponse getUser(
        @Parameter(
            description = &quot;조회할 사용자 ID&quot;,
            example = &quot;1&quot;
        )
        @PathVariable Long id
) {
    return userService.getUser(id);
}</code></pre>
<h3 id="요청-dto-문서화">요청 DTO 문서화</h3>
<pre><code class="language-java">public record UserRequest(

    @Schema(
        description = &quot;사용자 이름&quot;,
        example = &quot;alice&quot;
    )
    String name,

    @Schema(
        description = &quot;이메일&quot;,
        example = &quot;alice@example.com&quot;
    )
    String email

) {
}</code></pre>
<hr />
<h2 id="55-응답-상태-문서화">5.5 응답 상태 문서화</h2>
<pre><code class="language-java">@ApiResponses({
    @ApiResponse(
        responseCode = &quot;200&quot;,
        description = &quot;조회 성공&quot;
    ),
    @ApiResponse(
        responseCode = &quot;404&quot;,
        description = &quot;사용자를 찾을 수 없음&quot;
    )
})
@GetMapping(&quot;/users/{id}&quot;)
public UserResponse getUser(
        @PathVariable Long id
) {
    return userService.getUser(id);
}</code></pre>
<p>문서와 실제 상태 코드가 일치해야 한다.</p>
<pre><code class="language-text">문서: 404 반환
실제 코드: 200 + null 반환
→ 잘못된 문서</code></pre>
<p>따라서 실제 예외 처리와 응답 코드도 함께 설계해야 한다.</p>
<hr />
<h2 id="56-api-문서화의-장점">5.6 API 문서화의 장점</h2>
<ul>
<li>프론트엔드 개발자가 API를 쉽게 이해할 수 있다.</li>
<li>URL과 요청 형식을 한눈에 확인할 수 있다.</li>
<li>Swagger UI에서 API를 직접 테스트할 수 있다.</li>
<li>요청과 응답 예시를 공유할 수 있다.</li>
<li>코드 변경 시 문서도 함께 갱신할 수 있다.</li>
</ul>
<p>하지만 문서가 있다고 테스트가 필요 없는 것은 아니다.</p>
<pre><code class="language-text">Swagger/OpenAPI → 사용법 설명
JUnit/통합 테스트 → 실제 동작 검증</code></pre>
<hr />
<h1 id="6-actuator">6. Actuator</h1>
<h2 id="61-actuator란">6.1 Actuator란?</h2>
<p>Spring Boot Actuator는 실행 중인 애플리케이션의 상태를 확인하고 관리하기 위한 기능이다.</p>
<p>쉽게 말하면 서버의 건강검진 도구다.</p>
<pre><code class="language-text">애플리케이션
├── 서버가 살아 있는가?
├── DB 연결이 정상인가?
├── 어떤 Bean이 등록됐는가?
├── 어떤 URL이 연결됐는가?
├── 메모리를 얼마나 사용하는가?
└── 환경 설정은 무엇인가?</code></pre>
<hr />
<h2 id="62-의존성-추가">6.2 의존성 추가</h2>
<pre><code class="language-xml">&lt;dependency&gt;
    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
    &lt;artifactId&gt;
        spring-boot-starter-actuator
    &lt;/artifactId&gt;
&lt;/dependency&gt;</code></pre>
<hr />
<h2 id="63-기본-health-확인">6.3 기본 Health 확인</h2>
<p>서버 실행 후 다음 주소를 호출한다.</p>
<pre><code class="language-text">http://localhost:8080/actuator/health</code></pre>
<p>정상 응답:</p>
<pre><code class="language-json">{
  &quot;status&quot;: &quot;UP&quot;
}</code></pre>
<pre><code class="language-text">UP   → 애플리케이션 정상
DOWN → 연결된 구성요소에 문제 발생</code></pre>
<hr />
<h2 id="64-주요-endpoint">6.4 주요 Endpoint</h2>
<table>
<thead>
<tr>
<th>Endpoint</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>/actuator/health</code></td>
<td>서버 건강 상태</td>
</tr>
<tr>
<td><code>/actuator/info</code></td>
<td>애플리케이션 정보</td>
</tr>
<tr>
<td><code>/actuator/metrics</code></td>
<td>각종 수치 정보</td>
</tr>
<tr>
<td><code>/actuator/beans</code></td>
<td>등록된 Spring Bean</td>
</tr>
<tr>
<td><code>/actuator/mappings</code></td>
<td>Controller URL 매핑</td>
</tr>
<tr>
<td><code>/actuator/env</code></td>
<td>환경 설정</td>
</tr>
<tr>
<td><code>/actuator/loggers</code></td>
<td>로그 레벨 확인 및 변경</td>
</tr>
<tr>
<td><code>/actuator/threaddump</code></td>
<td>Thread 상태</td>
</tr>
<tr>
<td><code>/actuator/heapdump</code></td>
<td>Heap 메모리 정보</td>
</tr>
</tbody></table>
<hr />
<h2 id="65-endpoint-공개-설정">6.5 Endpoint 공개 설정</h2>
<p>기본적으로 모든 Endpoint가 외부에 공개되지는 않는다.</p>
<pre><code class="language-properties">management.endpoints.web.exposure.include=health,info,metrics</code></pre>
<p>학습 목적으로 모두 공개하려면 다음처럼 설정할 수 있다.</p>
<pre><code class="language-properties">management.endpoints.web.exposure.include=*</code></pre>
<p>하지만 운영 환경에서 전체 공개는 위험하다.</p>
<pre><code class="language-text">beans → 내부 Bean 구조 노출
env   → 환경 정보 노출 가능
heapdump → 메모리 데이터 노출 가능</code></pre>
<p>운영 환경에서는 필요한 Endpoint만 공개하고 Spring Security 등으로 보호해야 한다.</p>
<hr />
<h2 id="66-health-상세-정보">6.6 Health 상세 정보</h2>
<pre><code class="language-properties">management.endpoint.health.show-details=always</code></pre>
<p>응답 예시:</p>
<pre><code class="language-json">{
  &quot;status&quot;: &quot;UP&quot;,
  &quot;components&quot;: {
    &quot;db&quot;: {
      &quot;status&quot;: &quot;UP&quot;
    },
    &quot;diskSpace&quot;: {
      &quot;status&quot;: &quot;UP&quot;
    }
  }
}</code></pre>
<p>이 설정도 운영 환경에서는 보안에 주의해야 한다.</p>
<hr />
<h2 id="67-애플리케이션-정보-제공">6.7 애플리케이션 정보 제공</h2>
<pre><code class="language-properties">info.app.name=SKALA MyApp
info.app.version=1.0.0
info.app.description=사용자 관리 API</code></pre>
<p>다음 주소에서 확인할 수 있다.</p>
<pre><code class="language-text">http://localhost:8080/actuator/info</code></pre>
<hr />
<h2 id="68-metrics-확인">6.8 Metrics 확인</h2>
<p>전체 Metrics 목록:</p>
<pre><code class="language-text">http://localhost:8080/actuator/metrics</code></pre>
<p>JVM 메모리 확인:</p>
<pre><code class="language-text">http://localhost:8080/actuator/metrics/jvm.memory.used</code></pre>
<p>HTTP 요청 수 확인:</p>
<pre><code class="language-text">http://localhost:8080/actuator/metrics/http.server.requests</code></pre>
<p>확인 가능한 정보의 예:</p>
<ul>
<li>JVM 메모리</li>
<li>CPU 사용량</li>
<li>실행 중인 Thread</li>
<li>HTTP 요청 횟수</li>
<li>요청 처리 시간</li>
<li>DB Connection Pool 상태</li>
<li>Garbage Collection 상태</li>
</ul>
<hr />
<h2 id="69-mappings-확인">6.9 Mappings 확인</h2>
<pre><code class="language-text">http://localhost:8080/actuator/mappings</code></pre>
<p>현재 등록된 Controller 주소를 확인할 수 있다.</p>
<pre><code class="language-text">/api/hello
/api/users
/api/users/{id}</code></pre>
<p>주소가 404일 때 Controller 매핑이 실제로 등록됐는지 확인하는 데 유용하다.</p>
<hr />
<h2 id="610-actuator와-운영-모니터링">6.10 Actuator와 운영 모니터링</h2>
<p>Actuator가 제공하는 Metrics는 Prometheus 같은 수집 도구와 연결할 수 있다.</p>
<pre><code class="language-text">Spring Boot Actuator
→ Micrometer
→ Prometheus
→ Grafana</code></pre>
<table>
<thead>
<tr>
<th>도구</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Actuator</td>
<td>애플리케이션 상태 제공</td>
</tr>
<tr>
<td>Micrometer</td>
<td>Metrics 형식 통합</td>
</tr>
<tr>
<td>Prometheus</td>
<td>Metrics 수집과 저장</td>
</tr>
<tr>
<td>Grafana</td>
<td>그래프와 대시보드 표시</td>
</tr>
</tbody></table>
<hr />
<h1 id="7-현재-프로젝트와-연결해서-보기">7. 현재 프로젝트와 연결해서 보기</h1>
<p>현재 <code>myapp</code>의 전체 요청 흐름을 5일차 내용과 연결하면 다음과 같다.</p>
<pre><code class="language-text">POST /api/users
       ↓
UserController
- REST API 요청 수신
- DTO와 Validation으로 입력 검사
       ↓
UserService
- @Transactional로 업무 단위 관리
       ↓
UserRepository
- JpaRepository 메서드 호출
       ↓
EntityManager
- User Entity 관리
- 영속성 컨텍스트 사용
       ↓
Hibernate
- INSERT SQL 생성
       ↓
H2 Database
- users 테이블에 저장</code></pre>
<p>부가 기능까지 합치면 다음과 같다.</p>
<pre><code class="language-text">Swagger UI
→ API 사용법과 테스트 화면 제공

Controller
→ 요청 수신

AOP
→ 실행 시간과 로그 기록

Validation
→ 입력값 검사

Service
→ 트랜잭션 관리

Repository/JPA
→ 데이터 저장과 조회

Actuator
→ 실행 중인 서버 상태 확인</code></pre>
<hr />
<h1 id="8-전체-예제">8. 전체 예제</h1>
<h2 id="entity-1">Entity</h2>
<pre><code class="language-java">@Entity
@Table(name = &quot;users&quot;)
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public void update(String name, String email) {
        this.name = name;
        this.email = email;
    }
}</code></pre>
<h2 id="repository">Repository</h2>
<pre><code class="language-java">public interface UserRepository
        extends JpaRepository&lt;User, Long&gt; {

    List&lt;User&gt; findByName(String name);

    boolean existsByEmail(String email);
}</code></pre>
<h2 id="request-dto">Request DTO</h2>
<pre><code class="language-java">public record UserRequest(

    @NotBlank
    String name,

    @NotBlank
    @Email
    String email

) {
}</code></pre>
<h2 id="response-dto">Response DTO</h2>
<pre><code class="language-java">public record UserResponse(
    Long id,
    String name,
    String email
) {
    public static UserResponse from(User user) {
        return new UserResponse(
            user.getId(),
            user.getName(),
            user.getEmail()
        );
    }
}</code></pre>
<h2 id="service">Service</h2>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserService {

    private final UserRepository userRepository;

    @Transactional
    public UserResponse createUser(
            UserRequest request
    ) {
        if (userRepository.existsByEmail(
                request.email()
        )) {
            throw new IllegalArgumentException(
                &quot;이미 등록된 이메일입니다.&quot;
            );
        }

        User user = new User(
            request.name(),
            request.email()
        );

        User savedUser =
                userRepository.save(user);

        return UserResponse.from(savedUser);
    }

    public List&lt;UserResponse&gt; getAllUsers() {
        return userRepository.findAll()
                .stream()
                .map(UserResponse::from)
                .toList();
    }
}</code></pre>
<h2 id="controller">Controller</h2>
<pre><code class="language-java">@Tag(name = &quot;User API&quot;)
@RestController
@RequestMapping(&quot;/api/users&quot;)
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @Operation(summary = &quot;전체 사용자 조회&quot;)
    @GetMapping
    public List&lt;UserResponse&gt; getUsers() {
        return userService.getAllUsers();
    }

    @Operation(summary = &quot;사용자 생성&quot;)
    @PostMapping
    public UserResponse createUser(
            @Valid @RequestBody
            UserRequest request
    ) {
        return userService.createUser(request);
    }
}</code></pre>
<hr />
<h1 id="9-핵심-요약">9. 핵심 요약</h1>
<table>
<thead>
<tr>
<th>개념</th>
<th>한 줄 설명</th>
</tr>
</thead>
<tbody><tr>
<td>JPA</td>
<td>Java 객체와 DB 테이블을 연결하는 표준</td>
</tr>
<tr>
<td>Hibernate</td>
<td>JPA를 실제로 구현한 기술</td>
</tr>
<tr>
<td>Spring Data JPA</td>
<td>Repository 방식으로 JPA 사용을 간단하게 해주는 기술</td>
</tr>
<tr>
<td>Entity</td>
<td>DB 테이블과 연결된 Java 클래스</td>
</tr>
<tr>
<td>Repository</td>
<td>Entity의 CRUD를 담당하는 인터페이스</td>
</tr>
<tr>
<td>관계 매핑</td>
<td>객체 관계와 DB 외래키 관계를 연결</td>
</tr>
<tr>
<td>EntityManager</td>
<td>Entity의 저장·조회·수정·삭제를 관리</td>
</tr>
<tr>
<td>영속성 컨텍스트</td>
<td>JPA가 Entity를 관리하는 작업 공간</td>
</tr>
<tr>
<td>변경 감지</td>
<td>Entity 변경을 감지해 UPDATE하는 기능</td>
</tr>
<tr>
<td>Transaction</td>
<td>여러 DB 작업을 하나로 묶는 단위</td>
</tr>
<tr>
<td><code>@Transactional</code></td>
<td>트랜잭션을 자동으로 시작·커밋·롤백</td>
</tr>
<tr>
<td>OpenAPI</td>
<td>REST API 문서 표준</td>
</tr>
<tr>
<td>Swagger UI</td>
<td>API 문서 확인 및 테스트 화면</td>
</tr>
<tr>
<td>Actuator</td>
<td>실행 중인 Spring Boot 서버 상태 확인</td>
</tr>
</tbody></table>
<p>전체 관계를 한 줄로 정리하면 다음과 같다.</p>
<pre><code class="language-text">REST API 요청
→ Controller
→ Transactional Service
→ JPA Repository
→ EntityManager와 영속성 컨텍스트
→ Hibernate가 SQL 생성
→ DB 저장
→ Swagger로 문서 확인
→ Actuator로 서버 상태 확인</code></pre>