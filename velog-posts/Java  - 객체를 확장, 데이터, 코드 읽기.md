<blockquote>
<p> 범위: 상속 · 인터페이스 · 추상화 · 다형성 → Collection → Generic → Lambda → Stream/<code>java.util</code> → Reflection/Annotation</p>
<p>목표: 객체를 유연하게 설계하고, 많은 데이터를 안전하고 간결하게 처리하며, 프레임워크가 코드를 자동으로 다루는 원리를 이해한다.</p>
</blockquote>
<hr />
<h2 id="상속-인터페이스-추상화-다형성">상속, 인터페이스, 추상화, 다형성</h2>
<h3 id="1-상속inheritance-공통-기능을-물려받기">1. 상속(Inheritance): 공통 기능을 물려받기</h3>
<p>상속은 부모 클래스가 가진 상태(필드)와 행동(메서드)을 자식 클래스가 물려받아 재사용·확장하는 방식이다. <code>extends</code>를 사용한다.</p>
<pre><code class="language-java">class Stock {
    private String name;
    private double price;

    public Stock(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public void printInfo() {
        System.out.println(&quot;[일반주] &quot; + name + &quot;, &quot; + price + &quot;원&quot;);
    }
}

class PreferredStock extends Stock {
    private double dividendRate;

    public PreferredStock(String name, double price, double dividendRate) {
        super(name, price); // 부모 생성자 호출
        this.dividendRate = dividendRate;
    }
}</code></pre>
<p><code>PreferredStock</code>은 <code>Stock</code>의 한 종류다. 즉 <strong>우선주는 주식이다(is-a)</strong> 라고 자연스럽게 말할 수 있으므로 상속 관계가 어울린다. 반대로 “자동차가 엔진을 가진다(has-a)”는 상속보다 객체를 필드로 가지는 합성이 더 적절하다.</p>
<p>Java에서 다중 상속은 지원하지 않는다(대신 인터페이스를 활용하여 다중 상속을 지원)</p>
<h4 id="super와-접근-제어자"><code>super</code>와 접근 제어자</h4>
<ul>
<li><code>super(...)</code>: 자식 객체를 만들 때 부모 부분을 먼저 초기화하는 부모 생성자 호출이다. 생성자의 첫 줄에 둔다.</li>
<li><code>private</code>: 해당 클래스 내부에서만 접근한다. 자식도 직접 접근할 수 없다.</li>
<li><code>protected</code>: 같은 패키지와 자식 클래스에서 접근할 수 있다.</li>
<li><code>public</code>: 어디서나 접근할 수 있다.</li>
</ul>
<p>필드는 <code>private</code>으로 숨기고, 필요한 정보만 <code>getName()</code> 같은 메서드로 제공하는 편이 안전하다.</p>
<h3 id="2-인터페이스interface-해야-할-일을-약속하기">2. 인터페이스(Interface): 해야 할 일을 약속하기</h3>
<p>인터페이스는 “이 타입이라면 어떤 기능을 제공해야 한다”는 <strong>규약(계약)</strong> 이다. 보통 구현 내용 없이 메서드의 이름·매개변수·반환형만 선언한다.</p>
<pre><code class="language-java">interface Payment {
    void pay(int amount);
}

class CardPayment implements Payment {
    @Override
    public void pay(int amount) {
        System.out.println(amount + &quot;원 카드 결제&quot;);
    }
}

class KakaoPayment implements Payment {
    @Override
    public void pay(int amount) {
        System.out.println(amount + &quot;원 카카오페이 결제&quot;);
    }
}</code></pre>
<p>카페의 “결제 가능한 수단”이라는 공통 약속이 <code>Payment</code>라면 카드와 카카오페이는 각자 방식으로 약속을 지킨다. 인터페이스 메서드는 기본적으로 <code>public</code>이므로 구현할 때도 <code>public</code>이어야 한다.</p>
<p>Java 클래스는 클래스 하나만 <code>extends</code>할 수 있지만, 인터페이스는 여러 개 <code>implements</code>할 수 있다.</p>
<pre><code class="language-java">class SmartPhone implements Camera, MusicPlayer { }</code></pre>
<h3 id="3-추상-클래스abstract-class-공통-구현--빈칸-남기기">3. 추상 클래스(Abstract Class): 공통 구현 + 빈칸 남기기</h3>
<p>추상 클래스는 공통 필드·생성자·구현 메서드를 제공하면서, 자식이 반드시 채워야 할 기능도 남길 수 있는 중간 설계도다. <code>abstract</code> 클래스로는 직접 객체를 만들 수 없다.</p>
<pre><code class="language-java">abstract class Animal {
    String name;

    Animal(String name) {
        this.name = name;
    }

    abstract void makeSound(); // 자식이 반드시 구현

    void sleep() {             // 공통 기능은 이미 구현
        System.out.println(name + &quot;가 잠을 잡니다.&quot;);
    }
}</code></pre>
<table>
<thead>
<tr>
<th>구분</th>
<th>인터페이스</th>
<th>추상 클래스</th>
</tr>
</thead>
<tbody><tr>
<td>핵심 목적</td>
<td>할 수 있는 일을 약속</td>
<td>공통 기반을 제공하며 일부를 강제</td>
</tr>
<tr>
<td>상속/구현</td>
<td>여러 개 구현 가능</td>
<td>하나만 상속 가능</td>
</tr>
<tr>
<td>필드·생성자</td>
<td>일반 인스턴스 필드/생성자 없음</td>
<td>가질 수 있음</td>
</tr>
<tr>
<td>선택 기준</td>
<td>서로 다른 객체가 같은 역할을 수행</td>
<td>밀접한 부모-자식이 상태와 코드를 공유</td>
</tr>
</tbody></table>
<h3 id="4-다형성polymorphism-부모-타입으로-여러-자식을-다루기">4. 다형성(Polymorphism): 부모 타입으로 여러 자식을 다루기</h3>
<p>다형성은 하나의 부모 클래스 또는 인터페이스 타입으로 여러 실제 객체를 다루는 능력이다.</p>
<pre><code class="language-java">Payment payment = new CardPayment();
payment.pay(5000);

payment = new KakaoPayment();
payment.pay(5000);</code></pre>
<p>변수 타입은 계속 <code>Payment</code>지만, 실제 객체에 따라 실행되는 <code>pay()</code>가 달라진다. 이를 <strong>동적 바인딩</strong>이라고 하며, 자식이 부모 메서드를 다시 구현한 <strong>오버라이딩</strong>이 기반이다.</p>
<pre><code class="language-java">class PreferredStock extends Stock {
    @Override
    public void printInfo() { // 부모의 같은 메서드를 자식 방식으로 재정의
        System.out.println(&quot;[우선주] 배당률 포함 정보&quot;);
    }
}

static void printStockInfo(Stock stock) {
    stock.printInfo();
}

printStockInfo(new Stock(&quot;SKALA&quot;, 15000));
printStockInfo(new PreferredStock(&quot;Netflix&quot;, 60000, 10.0));</code></pre>
<h4 id="오버라이딩과-오버로딩을-구분하기">오버라이딩과 오버로딩을 구분하기</h4>
<table>
<thead>
<tr>
<th>구분</th>
<th>오버라이딩</th>
<th>오버로딩</th>
</tr>
</thead>
<tbody><tr>
<td>관계</td>
<td>부모-자식</td>
<td>같은 클래스 안에서 주로 사용</td>
</tr>
<tr>
<td>메서드 이름</td>
<td>같음</td>
<td>같음</td>
</tr>
<tr>
<td>매개변수</td>
<td>같아야 함</td>
<td>달라야 함</td>
</tr>
<tr>
<td>의미</td>
<td>물려받은 기능을 자식 방식으로 변경</td>
<td>입력 형태가 다른 같은 이름의 기능 추가</td>
</tr>
</tbody></table>
<pre><code class="language-java">// 오버로딩: String을 받는 새 형태 추가
public void printInfo(String prefix) {
    System.out.println(prefix + &quot;주식 정보&quot;);
}</code></pre>
<p><code>@Override</code>는 오버라이딩 의도를 컴파일러에 알려 오타를 잡아 주므로 항상 붙이는 습관이 좋다.가독성 향상이나, 리팩토링 안정성도 있다.</p>
<hr />
<h2 id="collection-type">Collection Type</h2>
<p>컬렉션은 많은 데이터를 저장·검색·수정·삭제하기 위한 표준 자료구조 모음이다. 배열은 크기가 고정되지만 컬렉션은 필요에 따라 크기를 늘리고 줄일 수 있다.</p>
<h3 id="1-어떤-컬렉션을-고를까">1. 어떤 컬렉션을 고를까?</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>특징</th>
<th>일상 예시</th>
<th>대표 구현체</th>
</tr>
</thead>
<tbody><tr>
<td><code>List</code></td>
<td>순서 있음, 중복 허용, 인덱스 접근</td>
<td>대기 명단</td>
<td><code>ArrayList</code>, <code>LinkedList</code>, <code>Vector</code></td>
</tr>
<tr>
<td><code>Set</code></td>
<td>중복 불가</td>
<td>출석한 사람의 학번</td>
<td><code>HashSet</code>, <code>LinkedHashSet</code>, <code>TreeSet</code></td>
</tr>
<tr>
<td><code>Map</code></td>
<td>키-값 쌍, 키 중복 불가</td>
<td>학번 → 학생 정보</td>
<td><code>HashMap</code>, <code>LinkedHashMap</code>, <code>TreeMap</code></td>
</tr>
<tr>
<td><code>Queue</code></td>
<td>먼저 넣은 것이 먼저 나감(FIFO)</td>
<td>은행 대기 줄</td>
<td><code>ArrayDeque</code>, <code>LinkedList</code>, <code>PriorityQueue</code></td>
</tr>
<tr>
<td><code>Deque</code></td>
<td>양쪽에서 넣고 뺌</td>
<td>양방향 대기 줄</td>
<td><code>ArrayDeque</code></td>
</tr>
<tr>
<td><code>Stack</code></td>
<td>마지막에 넣은 것이 먼저 나감(LIFO)</td>
<td>접시 쌓기, 브라우저 뒤로가기</td>
<td><code>ArrayDeque</code></td>
</tr>
</tbody></table>
<p><code>Map</code>은 <code>Collection</code> 인터페이스를 직접 상속하지 않지만 Java Collections Framework의 중요한 구성원이다.</p>
<h3 id="2-list">2. List</h3>
<pre><code class="language-java">List&lt;String&gt; names = new ArrayList&lt;&gt;();
names.add(&quot;민지&quot;);
names.add(&quot;철수&quot;);
names.add(&quot;민지&quot;); // 중복 가능

System.out.println(names.get(0)); // 민지
names.set(1, &quot;영희&quot;);
names.remove(0);</code></pre>
<table>
<thead>
<tr>
<th>구현체</th>
<th>선택 기준</th>
</tr>
</thead>
<tbody><tr>
<td><code>ArrayList</code></td>
<td>가장 일반적. 인덱스 조회가 많을 때</td>
</tr>
<tr>
<td><code>LinkedList</code></td>
<td>중간 삽입·삭제가 매우 잦을 때</td>
</tr>
<tr>
<td><code>Vector</code></td>
<td>오래된 동기화 List. 새 코드에서는 보통 <code>ArrayList</code>를 우선 고려</td>
</tr>
</tbody></table>
<p>실무의 웹 서비스는 데이터를 보통 DB에 저장하므로, 단순히 “LinkedList가 중간 삽입에 빠르다”만으로 선택하지 않는다. 데이터 크기, I/O, DB 쿼리 등이 더 큰 영향을 주는 경우가 많다.</p>
<table>
<thead>
<tr>
<th>종류</th>
<th>간단한 설명</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>ArrayList</code></td>
<td>크기를 자동으로 늘려주는 배열</td>
<td>조회가 빠름. 중간 삽입·삭제는 느릴 수 있음</td>
</tr>
<tr>
<td><code>LinkedList</code></td>
<td>각 데이터가 앞뒤 데이터를 연결하는 구조</td>
<td>중간 삽입·삭제에 유리. 인덱스 조회는 느림</td>
</tr>
<tr>
<td><code>Vector</code></td>
<td>동기화 기능이 포함된 <code>ArrayList</code></td>
<td>멀티스레드에 안전하지만 동기화 때문에 상대적으로 느림</td>
</tr>
</tbody></table>
<p>일상 예시로 보면:</p>
<ul>
<li><code>ArrayList</code>: 번호가 있는 사물함 - 번호로 바로 찾기 쉬움</li>
<li><code>LinkedList</code>: 앞뒤 사람의 손을 잡은 줄 - 연결을 끊고 사람을 넣고 빼기 쉬움</li>
<li><code>Vector</code>: 열고 닫을 때마다 잠그는 사물함 - 안전하지만 매번 잠금 처리로 느릴 수 있음</li>
</ul>
<h3 id="3-set">3. Set</h3>
<pre><code class="language-java">Set&lt;String&gt; attendees = new HashSet&lt;&gt;();
attendees.add(&quot;20260001&quot;);
attendees.add(&quot;20260002&quot;);
attendees.add(&quot;20260001&quot;); // 이미 있어도 하나만 보관

System.out.println(attendees.size()); // 2</code></pre>
<p>기본적으로 중복을 지원하지 않는다.</p>
<ul>
<li><code>HashSet</code>: 순서가 필요 없고 빠른 중복 검사에 적합</li>
<li><code>LinkedHashSet</code>: 넣은 순서를 유지</li>
<li><code>TreeSet</code>: 정렬된 순서를 유지</li>
</ul>
<h3 id="4-map">4. Map</h3>
<pre><code class="language-java">Map&lt;String, Integer&gt; prices = new HashMap&lt;&gt;();
prices.put(&quot;SKALA AI&quot;, 15000);
prices.put(&quot;SKALA EDU&quot;, 12000);

System.out.println(prices.get(&quot;SKALA AI&quot;));

for (Map.Entry&lt;String, Integer&gt; entry : prices.entrySet()) {
    System.out.println(entry.getKey() + &quot;: &quot; + entry.getValue());
}</code></pre>
<p>같은 키로 <code>put()</code>하면 기존 값이 새 값으로 바뀐다. <code>HashMap</code>은 순서 보장 없음, <code>LinkedHashMap</code>은 삽입 순서 유지, <code>TreeMap</code>은 키 기준 정렬이 특징이다.</p>
<h3 id="5-queue-deque-stack">5. Queue, Deque, Stack</h3>
<pre><code class="language-java">Queue&lt;String&gt; queue = new ArrayDeque&lt;&gt;();
queue.offer(&quot;첫 번째 요청&quot;);
queue.offer(&quot;두 번째 요청&quot;);
System.out.println(queue.poll()); // 첫 번째 요청

Deque&lt;String&gt; stack = new ArrayDeque&lt;&gt;();
stack.push(&quot;첫 페이지&quot;);
stack.push(&quot;두 번째 페이지&quot;);
System.out.println(stack.pop()); // 두 번째 페이지</code></pre>
<ul>
<li><code>offer()</code>/<code>poll()</code>은 실패하거나 비었을 때 예외 대신 <code>false</code>/<code>null</code>을 돌려준다.</li>
<li><code>add()</code>/<code>remove()</code>는 불가능한 상황에서 예외를 던진다.</li>
<li>새 코드에서 스택 용도로는 오래된 <code>Stack</code>보다 보통 <code>ArrayDeque</code>를 권장한다.</li>
</ul>
<h3 id="6-collection-공통-메서드와-순회">6. Collection 공통 메서드와 순회</h3>
<p><code>Collection&lt;E&gt;</code>는 <code>List</code>, <code>Set</code>, <code>Queue</code>가 공유하는 상위 인터페이스다. 대표 메서드는 <code>add</code>, <code>addAll</code>, <code>remove</code>, <code>removeAll</code>, <code>contains</code>, <code>size</code>, <code>isEmpty</code>, <code>clear</code>, <code>iterator</code>다.</p>
<pre><code class="language-java">Collection&lt;String&gt; collection = new ArrayList&lt;&gt;();
collection.add(&quot;A&quot;);
collection.add(&quot;B&quot;);

Iterator&lt;String&gt; iterator = collection.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}</code></pre>
<p>구체 구현체 대신 <code>List&lt;String&gt;</code>, <code>Set&lt;String&gt;</code>, <code>Collection&lt;String&gt;</code>처럼 <strong>인터페이스 타입으로 선언</strong>하면 구현체를 바꾸기 쉬워진다.</p>
<hr />
<h2 id="generic-type">Generic Type</h2>
<p>제네릭은 클래스·인터페이스·메서드가 사용할 타입을 <strong>사용 시점에 지정</strong>하게 하는 문법이다. 컴파일 때 잘못된 타입을 잡고, 형 변환을 줄인다.</p>
<pre><code class="language-java">class Box&lt;T&gt; {
    private T item;

    public void setItem(T item) { this.item = item; }
    public T getItem() { return item; }
}

Box&lt;String&gt; stringBox = new Box&lt;&gt;();
stringBox.setItem(&quot;Hello&quot;);
String text = stringBox.getItem(); // 형 변환 불필요</code></pre>
<p>제네릭이 없으면 <code>Object</code>로 받고 꺼낼 때마다 형 변환해야 하며, 잘못된 타입은 실행 중에야 발견될 수 있다.</p>
<pre><code class="language-java">// 관례: T=Type, E=Element, K=Key, V=Value, R=Return
class Pair&lt;K, V&gt; {
    private K key;
    private V value;
}

Pair&lt;String, Integer&gt; stock = new Pair&lt;&gt;();</code></pre>
<h3 id="1-상한-제한-t-extends-number">1. 상한 제한: <code>&lt;T extends Number&gt;</code></h3>
<pre><code class="language-java">class NumberBox&lt;T extends Number&gt; {
    private T value;

    public void setValue(T value) { this.value = value; }
    public void printDouble() { System.out.println(value.doubleValue()); }
}</code></pre>
<p><code>Number</code> 또는 그 자식인 <code>Integer</code>, <code>Double</code> 등만 허용한다. <code>String</code>은 넣을 수 없다. 공통 부모 <code>Number</code>가 제공하는 <code>doubleValue()</code>를 안전하게 쓸 수 있다.</p>
<h3 id="2-와일드카드--extends-t--super-t">2. 와일드카드: <code>? extends T</code>, <code>? super T</code></h3>
<p>제네릭은 <code>List&lt;Integer&gt;</code>가 <code>List&lt;Number&gt;</code>의 자식이 아니다. 이 문제를 유연하게 다루는 데 와일드카드를 쓴다.</p>
<pre><code class="language-java">static double sum(List&lt;? extends Number&gt; list) {
    double sum = 0;
    for (Number number : list) {
        sum += number.doubleValue();
    }
    return sum;
}</code></pre>
<p><code>? extends Number</code>는 <code>List&lt;Integer&gt;</code>, <code>List&lt;Double&gt;</code> 등을 받아 <strong>읽기</strong>에 좋다. 실제 요소가 정확히 어떤 하위 타입인지 모르므로 <code>list.add(...)</code>는 안전하지 않아 허용하지 않는다.</p>
<pre><code class="language-java">static void addIntegers(List&lt;? super Integer&gt; list) {
    list.add(10);
    list.add(20);
}</code></pre>
<p><code>? super Integer</code>는 <code>List&lt;Integer&gt;</code>, <code>List&lt;Number&gt;</code>, <code>List&lt;Object&gt;</code>에 <code>Integer</code>를 <strong>쓰기</strong>에 좋다. 꺼낼 때는 정확한 타입을 보장할 수 없어 <code>Object</code>로 받는다.</p>
<blockquote>
<p>암기: <strong>PECS</strong> - Producer Extends, Consumer Super. 값을 꺼내 제공하는 쪽은 <code>extends</code>, 값을 받아 저장하는 쪽은 <code>super</code>.</p>
</blockquote>
<h3 id="3-제네릭-메서드">3. 제네릭 메서드</h3>
<p><code>static</code> 메서드는 클래스의 타입 매개변수 <code>T</code>를 직접 쓸 수 없다. 메서드 앞에 자신의 타입 매개변수를 선언한다.</p>
<pre><code class="language-java">static &lt;T&gt; void print(T value) {
    System.out.println(value);
}

static &lt;U extends Number&gt; double sum(List&lt;U&gt; list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}</code></pre>
<hr />
<h2 id="lambda-expression">Lambda Expression</h2>
<p>람다는 이름 없는 메서드처럼 행동을 짧게 표현해 전달하는 문법이다. Java에서 람다는 <strong>함수형 인터페이스</strong>(추상 메서드가 정확히 하나인 인터페이스)를 구현하는 객체로 동작한다.</p>
<pre><code class="language-java">Runnable task = () -&gt; System.out.println(&quot;작업 실행&quot;);
task.run();</code></pre>
<p>위 코드는 아래 익명 클래스보다 간결하다.</p>
<pre><code class="language-java">Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println(&quot;작업 실행&quot;);
    }
};</code></pre>
<h3 id="1-람다-문법">1. 람다 문법</h3>
<pre><code class="language-java">() -&gt; System.out.println(&quot;입력 없음&quot;)
x -&gt; x * 2
(a, b) -&gt; a + b
(a, b) -&gt; {
    int result = a + b;
    return result;
}</code></pre>
<p><code>@FunctionalInterface</code>는 함수형 인터페이스임을 명시하고, 실수로 추상 메서드를 둘 이상 만들지 않게 검사한다.</p>
<pre><code class="language-java">@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}

Calculator add = (a, b) -&gt; a + b;
Calculator multiply = (a, b) -&gt; a * b;</code></pre>
<h3 id="2-자주-쓰는-표준-함수형-인터페이스">2. 자주 쓰는 표준 함수형 인터페이스</h3>
<table>
<thead>
<tr>
<th>인터페이스</th>
<th>핵심 메서드</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>Runnable</code></td>
<td><code>run()</code></td>
<td>입력·반환 없는 작업</td>
<td><code>() -&gt; save()</code></td>
</tr>
<tr>
<td><code>Supplier&lt;T&gt;</code></td>
<td><code>get()</code></td>
<td>입력 없이 값 제공</td>
<td><code>() -&gt; &quot;SKALA&quot;</code></td>
</tr>
<tr>
<td><code>Consumer&lt;T&gt;</code></td>
<td><code>accept(T)</code></td>
<td>값을 받아 소비, 반환 없음</td>
<td><code>s -&gt; System.out.println(s)</code></td>
</tr>
<tr>
<td><code>Function&lt;T, R&gt;</code></td>
<td><code>apply(T)</code></td>
<td>값을 다른 값으로 변환</td>
<td><code>s -&gt; s.length()</code></td>
</tr>
<tr>
<td><code>Predicate&lt;T&gt;</code></td>
<td><code>test(T)</code></td>
<td>조건을 검사해 <code>boolean</code> 반환</td>
<td><code>n -&gt; n &gt; 0</code></td>
</tr>
<tr>
<td><code>Comparator&lt;T&gt;</code></td>
<td><code>compare(T, T)</code></td>
<td>두 값을 비교해 정렬 기준 제공</td>
<td><code>(a, b) -&gt; a - b</code></td>
</tr>
</tbody></table>
<h3 id="3-메서드-참조">3. 메서드 참조</h3>
<p>기존 메서드를 그대로 전달할 때 람다를 더 줄일 수 있다.</p>
<pre><code class="language-java">names.forEach(name -&gt; System.out.println(name));
names.forEach(System.out::println);</code></pre>
<h3 id="4-외부-변수와-effectively-final">4. 외부 변수와 effectively final</h3>
<p>람다는 바깥 지역 변수를 읽을 수 있지만, 그 변수는 <code>final</code> 또는 <strong>effectively final</strong>(선언 뒤 값이 바뀌지 않음)이어야 한다.</p>
<pre><code class="language-java">int base = 10;
Function&lt;Integer, Integer&gt; addBase = x -&gt; x + base;
System.out.println(addBase.apply(5)); // 15

// base = 20; // 이 대입이 있으면 위 람다는 컴파일 불가</code></pre>
<p>외부 상태를 바꾸지 않는 순수 함수는 예측·테스트·병렬 처리에 유리하다. 같은 입력이면 같은 결과를 내고 부작용이 없도록 작성하는 방향을 지향한다.</p>
<hr />
<h2 id="stream-api-및-javautil-클래스">Stream API 및 <code>java.util</code> 클래스</h2>
<h3 id="1-stream-api">1. Stream API</h3>
<p>Stream은 컬렉션의 데이터를 “어떻게 반복할지”보다 “무엇을 할지”에 집중해 처리하는 API다. 원본 컬렉션 자체가 아니라 그 데이터를 처리하는 흐름이다.</p>
<pre><code class="language-java">List&lt;String&gt; names = List.of(&quot;Alice&quot;, &quot;Bob&quot;, &quot;Andrew&quot;, &quot;Charlie&quot;);

List&lt;String&gt; result = names.stream()
    .filter(name -&gt; name.startsWith(&quot;A&quot;)) // 중간 연산: 걸러내기
    .map(String::toUpperCase)              // 중간 연산: 변환
    .limit(2)                              // 중간 연산: 제한
    .toList();                             // 최종 연산: 결과 만들기

System.out.println(result); // [ALICE, ANDREW]</code></pre>
<h4 id="stream의-세-단계">Stream의 세 단계</h4>
<pre><code class="language-text">원본 데이터 → Stream 생성 → 중간 연산 0개 이상 → 최종 연산 1개</code></pre>
<table>
<thead>
<tr>
<th>종류</th>
<th>대표 메서드</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>생성</td>
<td><code>stream()</code>, <code>Arrays.stream()</code>, <code>IntStream.of()</code></td>
<td>처리 흐름 시작</td>
</tr>
<tr>
<td>중간 연산</td>
<td><code>filter</code>, <code>map</code>, <code>distinct</code>, <code>sorted</code>, <code>limit</code>, <code>peek</code></td>
<td>다른 Stream을 반환, 연결 가능</td>
</tr>
<tr>
<td>최종 연산</td>
<td><code>forEach</code>, <code>count</code>, <code>sum</code>, <code>min</code>, <code>max</code>, <code>reduce</code>, <code>collect</code>, <code>toList</code></td>
<td>실제 계산을 시작하고 결과 반환</td>
</tr>
</tbody></table>
<p>중간 연산은 최종 연산 전까지 실제로 실행되지 않는 <strong>지연 연산(lazy evaluation)</strong> 이다. 또한 Stream은 한 번 사용하면 다시 사용할 수 없으므로 필요하면 새로 만든다.</p>
<pre><code class="language-java">long count = products.stream()
    .filter(Product::isUsable)
    .filter(product -&gt; product.getPrice() &lt;= 100_000)
    .count();</code></pre>
<h3 id="2-javautil의-자주-쓰는-유틸리티">2. <code>java.util</code>의 자주 쓰는 유틸리티</h3>
<h4 id="collections와-arrays"><code>Collections</code>와 <code>Arrays</code></h4>
<p><code>Collections</code>는 컬렉션용 정적 유틸리티 클래스이고, <code>Collection</code> 인터페이스와 이름은 비슷하지만 다른 대상이다.</p>
<pre><code class="language-java">List&lt;Integer&gt; scores = new ArrayList&lt;&gt;(List.of(70, 90, 80));
Collections.sort(scores);
Collections.reverse(scores);
Collections.shuffle(scores);

int[] numbers = {3, 1, 2};
Arrays.sort(numbers);
System.out.println(Arrays.toString(numbers)); // [1, 2, 3]</code></pre>
<table>
<thead>
<tr>
<th>클래스</th>
<th>자주 쓰는 기능</th>
</tr>
</thead>
<tbody><tr>
<td><code>Collections</code></td>
<td><code>sort</code>, <code>reverse</code>, <code>shuffle</code>, <code>max</code>, <code>min</code>, <code>frequency</code></td>
</tr>
<tr>
<td><code>Arrays</code></td>
<td><code>sort</code>, <code>toString</code>, <code>copyOf</code>, <code>equals</code>, <code>binarySearch</code></td>
</tr>
</tbody></table>
<h4 id="optional"><code>Optional</code></h4>
<p><code>Optional&lt;T&gt;</code>는 “값이 있을 수도, 없을 수도 있다”를 명시적으로 표현해 <code>NullPointerException</code>을 줄이는 컨테이너다.</p>
<pre><code class="language-java">String nullableName = null;
Optional&lt;String&gt; name = Optional.ofNullable(nullableName);

String result = name
    .map(String::toUpperCase)
    .orElse(&quot;이름 없음&quot;);</code></pre>
<table>
<thead>
<tr>
<th>생성/처리</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>Optional.of(value)</code></td>
<td>절대 <code>null</code>이 아닐 때. <code>null</code>이면 예외</td>
</tr>
<tr>
<td><code>Optional.ofNullable(value)</code></td>
<td><code>null</code> 가능성이 있을 때</td>
</tr>
<tr>
<td><code>Optional.empty()</code></td>
<td>값 없음</td>
</tr>
<tr>
<td><code>map()</code></td>
<td>값이 있을 때만 변환</td>
</tr>
<tr>
<td><code>filter()</code></td>
<td>조건을 만족할 때만 유지</td>
</tr>
<tr>
<td><code>orElse(defaultValue)</code></td>
<td>값이 없을 때 기본값</td>
</tr>
<tr>
<td><code>ifPresent()</code></td>
<td>값이 있을 때만 실행</td>
</tr>
</tbody></table>
<p><code>Optional</code>은 <code>get()</code>을 무조건 호출하는 용도보다 <code>map</code>, <code>orElse</code>, <code>ifPresent</code>로 빈 경우까지 함께 처리할 때 의미가 있다.</p>
<hr />
<h2 id="reflection-및-annotation">Reflection 및 Annotation</h2>
<h3 id="1-reflection-실행-중에-클래스-구조를-읽고-다루기">1. Reflection: 실행 중에 클래스 구조를 읽고 다루기</h3>
<p>Reflection은 컴파일 시점이 아니라 <strong>실행 중(runtime)</strong> 에 클래스의 생성자, 필드, 메서드, 어노테이션 정보를 살펴보고 필요한 경우 조작하는 기능이다. <code>java.lang.reflect</code> 패키지의 <code>Class</code>, <code>Constructor</code>, <code>Field</code>, <code>Method</code> 등이 중심이다.</p>
<pre><code class="language-java">Class&lt;?&gt; clazz = Class.forName(&quot;com.example.Person&quot;);
Object person = clazz.getDeclaredConstructor().newInstance();

Field field = clazz.getDeclaredField(&quot;name&quot;);
field.setAccessible(true);
field.set(person, &quot;홍길동&quot;);

Method method = clazz.getDeclaredMethod(&quot;sayHello&quot;, String.class);
method.invoke(person, &quot;민지&quot;);</code></pre>
<table>
<thead>
<tr>
<th>대상</th>
<th>Reflection으로 할 수 있는 일</th>
</tr>
</thead>
<tbody><tr>
<td><code>Class&lt;?&gt;</code></td>
<td>클래스 정보 얻기</td>
</tr>
<tr>
<td><code>Constructor&lt;?&gt;</code></td>
<td>객체 동적 생성</td>
</tr>
<tr>
<td><code>Field</code></td>
<td>필드 조회·값 변경</td>
</tr>
<tr>
<td><code>Method</code></td>
<td>메서드 조회·호출</td>
</tr>
<tr>
<td><code>Parameter</code></td>
<td>매개변수 정보 조회</td>
</tr>
</tbody></table>
<p>Spring의 의존성 주입(DI), JPA의 엔티티 매핑, 테스트 도구 등은 Reflection을 활용한다. 예를 들어 Spring은 특정 어노테이션이 붙은 필드를 찾아 객체를 주입할 수 있다.</p>
<p>하지만 일반 애플리케이션 코드에서 남용하면 직접 호출보다 느리고, <code>private</code> 접근으로 캡슐화를 깨며, 컴파일 시점 검사를 놓쳐 런타임 오류가 늘 수 있다.</p>
<h3 id="2-annotation-코드에-붙이는-구조화된-힌트">2. Annotation: 코드에 붙이는 구조화된 힌트</h3>
<p>Annotation은 클래스, 메서드, 필드 등에 붙이는 메타데이터다. 주석(<code>//</code>)은 사람에게 설명하는 글이고, Annotation은 컴파일러·JVM·프레임워크 같은 프로그램이 읽어 동작을 바꾸는 표식이다.</p>
<pre><code class="language-java">@Override
public String toString() {
    return &quot;...&quot;;
}

@Deprecated
public void oldMethod() { }</code></pre>
<p>직접 Annotation을 만들 때는 <code>@interface</code>를 사용한다.</p>
<pre><code class="language-java">import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RunMe {
    int order() default 0;
}</code></pre>
<pre><code class="language-java">class TaskRunner {
    @RunMe(order = 1)
    public void taskA() {
        System.out.println(&quot;Task A&quot;);
    }
}</code></pre>
<p>Reflection으로 <code>@RunMe</code>가 붙은 메서드를 찾아 호출할 수 있다.</p>
<pre><code class="language-java">for (Method method : TaskRunner.class.getDeclaredMethods()) {
    if (method.isAnnotationPresent(RunMe.class)) {
        method.invoke(new TaskRunner());
    }
}</code></pre>
<h3 id="3-target과-retention">3. <code>@Target</code>과 <code>@Retention</code></h3>
<p><code>@Target</code>은 “어디에 붙일 수 있는가”, <code>@Retention</code>은 “언제까지 남아 있는가”를 정한다.</p>
<table>
<thead>
<tr>
<th>설정</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>ElementType.TYPE</code></td>
<td>클래스, 인터페이스, enum</td>
</tr>
<tr>
<td><code>ElementType.FIELD</code></td>
<td>필드</td>
</tr>
<tr>
<td><code>ElementType.METHOD</code></td>
<td>메서드</td>
</tr>
<tr>
<td><code>ElementType.PARAMETER</code></td>
<td>매개변수</td>
</tr>
<tr>
<td><code>ElementType.CONSTRUCTOR</code></td>
<td>생성자</td>
</tr>
<tr>
<td><code>RetentionPolicy.SOURCE</code></td>
<td>소스 코드까지만, 컴파일 후 사라짐</td>
</tr>
<tr>
<td><code>RetentionPolicy.CLASS</code></td>
<td><code>.class</code>에는 남지만 런타임 Reflection 접근 불가</td>
</tr>
<tr>
<td><code>RetentionPolicy.RUNTIME</code></td>
<td>실행 중 Reflection으로 읽을 수 있음</td>
</tr>
</tbody></table>
<p>Spring에서 흔히 보는 어노테이션도 같은 원리다.</p>
<pre><code class="language-java">@RestController
class UserController {
    @GetMapping(&quot;/users/{id}&quot;)
    void getUser(@PathVariable(&quot;id&quot;) Long id,
                 @RequestParam(&quot;name&quot;) String name) { }
}</code></pre>
<ul>
<li>클래스: <code>@RestController</code> - 이 클래스의 역할을 알림</li>
<li>메서드: <code>@GetMapping</code> - 어떤 HTTP 요청을 처리할지 알림</li>
<li>매개변수: <code>@PathVariable</code>, <code>@RequestParam</code> - 요청 데이터가 어디에서 오는지 알림</li>
</ul>
<p>프레임워크가 실행 시 Annotation을 읽고, 규칙에 따라 객체 생성·URL 연결·값 주입을 자동으로 수행한다.</p>
<hr />
<h2 id="오늘의-연결-복습">오늘의 연결 복습</h2>
<ol>
<li><strong>상속·인터페이스·추상화·다형성</strong>으로 변경에 유연한 객체 구조를 만든다.</li>
<li><strong>컬렉션</strong>으로 여러 데이터를 목적에 맞게 저장하고 처리한다.</li>
<li><strong>제네릭</strong>으로 컬렉션과 재사용 코드의 타입 안전성을 확보한다.</li>
<li><strong>람다와 Stream</strong>으로 행동과 데이터 처리 과정을 간결하게 표현한다.</li>
<li><strong>Reflection과 Annotation</strong>은 프레임워크가 개발자의 코드를 찾아 자동으로 연결하는 핵심 원리다.</li>
</ol>
<h2 id="스스로-확인하기">스스로 확인하기</h2>
<ol>
<li><code>ArrayList</code>, <code>HashSet</code>, <code>HashMap</code>은 각각 언제 선택할까?</li>
<li><code>List&lt;? extends Number&gt;</code>에 값을 추가할 수 없는 이유는 무엇일까?</li>
<li>람다는 어떤 인터페이스에서만 사용할 수 있을까?</li>
<li>Stream의 중간 연산과 최종 연산은 어떻게 다를까?</li>
<li>Reflection으로 <code>private</code> 필드까지 접근할 수 있을 때 생길 수 있는 문제는 무엇일까?</li>
<li>Reflection으로 Annotation을 읽으려면 <code>@Retention</code>에 무엇이 필요할까?</li>
</ol>