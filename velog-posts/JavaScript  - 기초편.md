<hr />
<h2 id="1-javascript란">1. JavaScript란?</h2>
<p><strong>한 문장 이해:</strong> JavaScript는 웹 페이지의 데이터와 동작을 제어하는 프로그래밍 언어이다.</p>
<p>JavaScript로 할 수 있는 일:</p>
<ul>
<li>HTML의 글자·구조 변경</li>
<li>CSS class와 style 변경</li>
<li>클릭·입력·스크롤 같은 사용자 행동 처리</li>
<li>서버에서 데이터 가져오기</li>
<li>브라우저 저장소 사용</li>
<li>Node.js 환경에서 서버 프로그램 실행</li>
</ul>
<h3 id="java와-javascript">Java와 JavaScript</h3>
<p>이름만 비슷할 뿐 문법 체계와 실행 환경이 다른 별개의 언어이다.</p>
<h3 id="주요-특징">주요 특징</h3>
<table>
<thead>
<tr>
<th>특징</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td>동적 타입</td>
<td>변수의 타입을 미리 선언하지 않고 현재 값에 따라 결정</td>
</tr>
<tr>
<td>일급 함수</td>
<td>함수를 값처럼 변수에 저장하고 인수로 전달 가능</td>
</tr>
<tr>
<td>객체 기반</td>
<td>배열, 함수, 날짜 등 많은 값이 객체로 동작</td>
</tr>
<tr>
<td>단일 실행 스레드</td>
<td>한 시점에 JavaScript 코드는 하나씩 실행</td>
</tr>
<tr>
<td>비동기 지원</td>
<td>브라우저 API와 이벤트 루프를 통해 오래 걸리는 작업을 기다리는 동안 다른 코드 실행</td>
</tr>
</tbody></table>
<blockquote>
<p>현대 JavaScript 엔진은 단순히 한 줄씩 해석하기만 하는 것이 아니라 JIT 컴파일 등의 최적화도 수행한다.</p>
</blockquote>
<h3 id="ecmascript">ECMAScript</h3>
<ul>
<li>ECMAScript(ES)는 JavaScript의 표준 명세이다.</li>
<li>ES6 또는 ES2015에서 <code>let</code>, <code>const</code>, 화살표 함수, class, module 등이 도입되며 현대 JavaScript의 기준점이 되었다.</li>
</ul>
<h2 id="2-실행-방법">2. 실행 방법</h2>
<h3 id="html-내부에서-실행">HTML 내부에서 실행</h3>
<pre><code class="language-html">&lt;script&gt;
  console.log(&quot;Hello JavaScript&quot;);
&lt;/script&gt;</code></pre>
<h3 id="외부-파일-연결---권장">외부 파일 연결 - 권장</h3>
<pre><code class="language-html">&lt;script src=&quot;script/app.js&quot; defer&gt;&lt;/script&gt;</code></pre>
<pre><code class="language-javascript">// script/app.js
console.log(&quot;외부 JavaScript 파일&quot;);</code></pre>
<ul>
<li><code>defer</code>는 HTML 분석과 script 다운로드를 병행하고, HTML 분석이 끝난 뒤 실행한다.</li>
<li>module을 사용할 때는 <code>&lt;script type=&quot;module&quot;&gt;</code>을 쓴다. module은 기본적으로 지연 실행된다.</li>
</ul>
<h3 id="결과-확인-방법">결과 확인 방법</h3>
<table>
<thead>
<tr>
<th>방법</th>
<th>용도</th>
<th>주의</th>
</tr>
</thead>
<tbody><tr>
<td><code>console.log(value)</code></td>
<td>개발 중 값 확인</td>
<td>가장 많이 사용</td>
</tr>
<tr>
<td><code>alert(message)</code></td>
<td>간단한 알림</td>
<td>실행을 잠시 막으므로 남용 금지</td>
</tr>
<tr>
<td><code>element.textContent = value</code></td>
<td>HTML 요소에 안전하게 텍스트 표시</td>
<td>일반 텍스트 출력에 권장</td>
</tr>
<tr>
<td><code>element.innerHTML = html</code></td>
<td>HTML 구조 삽입</td>
<td>신뢰할 수 없는 문자열 사용 시 보안 위험</td>
</tr>
<tr>
<td><code>document.write()</code></td>
<td>문서 출력 스트림에 기록</td>
<td>일반 웹 개발에서는 사용하지 않는 편이 좋음</td>
</tr>
</tbody></table>
<p>개발자 도구 Console에서 로그와 오류 메시지를 확인한다.</p>
<h2 id="3-기본-문법">3. 기본 문법</h2>
<pre><code class="language-javascript">const price = 1000;          // statement
const total = price * 3;     // expression: price * 3
console.log(total);          // 3000</code></pre>
<ul>
<li><strong>literal</strong>: 코드에 직접 쓴 고정값. <code>10</code>, <code>&quot;hello&quot;</code>, <code>true</code></li>
<li><strong>variable</strong>: 값을 저장하고 이름으로 참조하는 공간</li>
<li><strong>identifier</strong>: 변수·함수 등에 붙인 이름</li>
<li><strong>operator</strong>: 값을 계산하는 기호. <code>+</code>, <code>===</code>, <code>&amp;&amp;</code></li>
<li><strong>expression</strong>: 평가하면 하나의 값이 되는 코드</li>
<li><strong>statement</strong>: 실행할 하나의 명령</li>
<li>JavaScript는 대소문자를 구분한다. <code>userName</code>과 <code>username</code>은 다르다.</li>
</ul>
<pre><code class="language-javascript">// 한 줄 주석

/*
  여러 줄 주석
*/</code></pre>
<hr />
<h2 id="4-변수와-scope">4. 변수와 Scope</h2>
<h3 id="const-let-var"><code>const</code>, <code>let</code>, <code>var</code></h3>
<table>
<thead>
<tr>
<th>키워드</th>
<th>scope</th>
<th align="right">재할당</th>
<th align="right">같은 scope에서 재선언</th>
<th>사용 원칙</th>
</tr>
</thead>
<tbody><tr>
<td><code>const</code></td>
<td>block</td>
<td align="right">불가</td>
<td align="right">불가</td>
<td>기본 선택</td>
</tr>
<tr>
<td><code>let</code></td>
<td>block</td>
<td align="right">가능</td>
<td align="right">불가</td>
<td>값이 바뀌어야 할 때</td>
</tr>
<tr>
<td><code>var</code></td>
<td>function</td>
<td align="right">가능</td>
<td align="right">가능</td>
<td>오래된 코드 외에는 지양</td>
</tr>
</tbody></table>
<pre><code class="language-javascript">const course = &quot;JavaScript&quot;;
let score = 70;
score = 85;</code></pre>
<p>변수 이름 규칙과 관례:</p>
<ul>
<li>문자, <code>_</code>, <code>$</code>로 시작할 수 있다.</li>
<li>숫자로 시작할 수 없다.</li>
<li>예약어를 사용할 수 없다.</li>
<li>의미가 드러나는 camelCase를 주로 사용한다: <code>totalScore</code>, <code>userName</code>.</li>
</ul>
<h3 id="const가-객체를-얼리는-것은-아니다"><code>const</code>가 객체를 얼리는 것은 아니다</h3>
<p><code>const</code>는 변수에 다른 값을 다시 대입하지 못하게 한다. 객체 내부 변경까지 막지는 않는다.</p>
<pre><code class="language-javascript">const user = { name: &quot;Kim&quot; };
user.name = &quot;Lee&quot;;      // 가능
// user = { name: &quot;Park&quot; }; // TypeError</code></pre>
<h3 id="scope">Scope</h3>
<pre><code class="language-javascript">const globalMessage = &quot;전역&quot;;

function run() {
  const functionMessage = &quot;함수 내부&quot;;

  if (true) {
    const blockMessage = &quot;블록 내부&quot;;
    console.log(blockMessage);
  }

  // console.log(blockMessage); // ReferenceError
}</code></pre>
<table>
<thead>
<tr>
<th>scope</th>
<th>접근 가능 범위</th>
</tr>
</thead>
<tbody><tr>
<td>Global</td>
<td>프로그램의 넓은 영역에서 접근 가능</td>
</tr>
<tr>
<td>Function</td>
<td>선언된 함수 내부</td>
</tr>
<tr>
<td>Block</td>
<td><code>{}</code> 내부. <code>let</code>, <code>const</code>에 적용</td>
</tr>
</tbody></table>
<p>전역 변수는 여러 코드가 함께 수정할 수 있어 버그를 만들기 쉽다. 필요한 범위 안에서만 선언한다.</p>
<h3 id="hoisting과-tdz">Hoisting과 TDZ</h3>
<p>Hoisting은 선언이 실제로 코드 위로 이동한다는 뜻이 아니라, 실행 전에 선언을 처리하는 JavaScript의 동작을 설명하는 표현이다.</p>
<pre><code class="language-javascript">console.log(a); // undefined
var a = 10;

// console.log(b); // ReferenceError
let b = 20;</code></pre>
<ul>
<li><code>var</code> 선언은 초기값 <code>undefined</code>로 접근 가능하다.</li>
<li><code>let</code>과 <code>const</code>도 scope 시작부터 처리되지만 선언문 전에는 접근할 수 없다.</li>
<li>이 접근 불가 구간을 <strong>Temporal Dead Zone(TDZ)</strong>이라고 한다.</li>
<li>함수 선언문은 선언 전에도 호출할 수 있지만, 읽는 흐름을 위해 사용 전 선언하는 습관이 좋다.</li>
</ul>
<hr />
<h2 id="5-자료형">5. 자료형</h2>
<h3 id="원시-타입과-객체-타입">원시 타입과 객체 타입</h3>
<table>
<thead>
<tr>
<th>원시 타입(Primitive)</th>
<th>객체 타입(Object)</th>
</tr>
</thead>
<tbody><tr>
<td>String, Number, BigInt, Boolean, Undefined, Null, Symbol</td>
<td>Object, Array, Function, Date, RegExp, Map, Set 등</td>
</tr>
<tr>
<td>값 자체를 비교</td>
<td>같은 객체를 가리키는지 비교</td>
</tr>
<tr>
<td>값이 불변</td>
<td>내부 속성을 변경할 수 있는 경우가 많음</td>
</tr>
</tbody></table>
<pre><code class="language-javascript">console.log(&quot;a&quot; === &quot;a&quot;);        // true
console.log({} === {});           // false: 서로 다른 객체

const first = { count: 1 };
const second = first;
console.log(first === second);    // true</code></pre>
<p>JavaScript는 동적 타입 언어이다.</p>
<pre><code class="language-javascript">let value;
value = 10;       // number
value = &quot;ten&quot;;   // string</code></pre>
<p>가능하다고 해서 한 변수에 여러 종류의 값을 계속 넣는 것이 좋은 설계는 아니다.</p>
<h2 id="51-string">5.1 String</h2>
<pre><code class="language-javascript">const single = '작은따옴표';
const double = &quot;큰따옴표&quot;;
const name = &quot;민수&quot;;
const template = `안녕하세요, ${name}님`;</code></pre>
<p>Template literal은 backtick <code>`</code>을 사용하며 여러 줄과 <code>${expression}</code> 삽입을 지원한다.</p>
<pre><code class="language-javascript">const text = &quot;  JavaScript Study  &quot;;

console.log(text.length);
console.log(text.trim());
console.log(text.toUpperCase());
console.log(text.includes(&quot;Script&quot;));
console.log(text.startsWith(&quot;Java&quot;));
console.log(text.slice(2, 12));
console.log(text.replace(&quot;Study&quot;, &quot;Review&quot;));
console.log(&quot;HTML,CSS,JS&quot;.split(&quot;,&quot;));</code></pre>
<p>문자열은 불변이므로 메서드는 원본을 바꾸지 않고 새 문자열을 반환한다.</p>
<pre><code class="language-javascript">const original = &quot;hello&quot;;
const upper = original.toUpperCase();

console.log(original); // hello
console.log(upper);    // HELLO</code></pre>
<h2 id="52-number">5.2 Number</h2>
<p>JavaScript의 일반 숫자는 정수와 실수를 모두 <code>number</code>로 다룬다.</p>
<pre><code class="language-javascript">const integer = 10;
const decimal = 3.14;
const hex = 0xff;</code></pre>
<p>주의할 값과 현상:</p>
<pre><code class="language-javascript">console.log(0.1 + 0.2);              // 0.30000000000000004
console.log(10 / 0);                 // Infinity
console.log(Number(&quot;hello&quot;));       // NaN
console.log(Number.isNaN(NaN));      // true
console.log(Number.isFinite(10));    // true</code></pre>
<p>변환과 표시:</p>
<pre><code class="language-javascript">console.log(Number(&quot;42&quot;));          // 42
console.log(Number.parseInt(&quot;42px&quot;, 10)); // 42
console.log(Number.parseFloat(&quot;3.14kg&quot;)); // 3.14
console.log((12.3456).toFixed(2));   // &quot;12.35&quot; - 결과는 문자열</code></pre>
<p>입력값을 숫자로 사용할 때는 명시적으로 변환하고 검증한다.</p>
<pre><code class="language-javascript">const raw = prompt(&quot;점수 입력&quot;);
const score = Number(raw);

if (Number.isNaN(score)) {
  alert(&quot;숫자를 입력하세요.&quot;);
}</code></pre>
<h2 id="53-bigint">5.3 BigInt</h2>
<p><code>Number.MAX_SAFE_INTEGER</code>보다 큰 정수를 정확히 다룰 때 사용한다.</p>
<pre><code class="language-javascript">const large = 12345678901234567890n;
const other = BigInt(&quot;9007199254740993&quot;);

// large + 1;  // TypeError: BigInt와 Number 직접 연산 불가
console.log(large + 1n);</code></pre>
<h2 id="54-boolean과-truthyfalsy">5.4 Boolean과 truthy/falsy</h2>
<pre><code class="language-javascript">const isAdult = 20 &gt;= 19; // true</code></pre>
<p>조건식에서는 값이 boolean으로 변환된다.</p>
<p>대표적인 falsy 값:</p>
<pre><code class="language-text">false, 0, -0, 0n, &quot;&quot;, null, undefined, NaN</code></pre>
<p>그 외 값은 대부분 truthy이다. 특히 빈 배열 <code>[]</code>과 빈 객체 <code>{}</code>도 truthy이다.</p>
<pre><code class="language-javascript">console.log(Boolean(&quot;&quot;));  // false
console.log(Boolean(&quot;0&quot;)); // true
console.log(Boolean([]));   // true</code></pre>
<h2 id="55-undefined와-null">5.5 <code>undefined</code>와 <code>null</code></h2>
<table>
<thead>
<tr>
<th>값</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>undefined</code></td>
<td>값이 아직 할당되지 않았거나 존재하지 않음</td>
</tr>
<tr>
<td><code>null</code></td>
<td>개발자가 의도적으로 비어 있음을 표현</td>
</tr>
</tbody></table>
<pre><code class="language-javascript">let notAssigned;
const selectedUser = null;

console.log(typeof notAssigned); // &quot;undefined&quot;
console.log(typeof selectedUser); // &quot;object&quot; - 오래된 역사적 동작</code></pre>
<p><code>null</code>은 원시 타입이다. <code>typeof null</code>이 <code>&quot;object&quot;</code>인 것은 JavaScript의 오래된 특이점이다.</p>
<h2 id="56-symbol">5.6 Symbol</h2>
<p>매번 고유한 값을 만들며 객체의 충돌하지 않는 property key 등에 사용한다.</p>
<pre><code class="language-javascript">const firstId = Symbol(&quot;id&quot;);
const secondId = Symbol(&quot;id&quot;);

console.log(firstId === secondId); // false</code></pre>
<h2 id="57-wrapper와-auto-boxing">5.7 Wrapper와 auto-boxing</h2>
<p>원시 문자열에 메서드를 호출하면 엔진이 일시적으로 wrapper처럼 다룬다.</p>
<pre><code class="language-javascript">console.log(&quot;hello&quot;.toUpperCase());
console.log((123.456).toFixed(2));</code></pre>
<p>명시적인 wrapper 객체는 혼란을 만들 수 있으므로 피한다.</p>
<pre><code class="language-javascript">const primitive = &quot;hello&quot;;
const wrapper = new String(&quot;hello&quot;);

console.log(typeof primitive); // string
console.log(typeof wrapper);   // object
console.log(primitive === wrapper); // false</code></pre>
<hr />
<h2 id="6-연산자">6. 연산자</h2>
<h3 id="산술·대입-연산자">산술·대입 연산자</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>연산자</th>
</tr>
</thead>
<tbody><tr>
<td>산술</td>
<td><code>+</code>, <code>-</code>, <code>*</code>, <code>/</code>, <code>%</code>, <code>**</code></td>
</tr>
<tr>
<td>1 증가·감소</td>
<td><code>++</code>, <code>--</code></td>
</tr>
<tr>
<td>대입</td>
<td><code>=</code>, <code>+=</code>, <code>-=</code>, <code>*=</code>, <code>/=</code>, <code>%=</code></td>
</tr>
</tbody></table>
<pre><code class="language-javascript">let count = 5;
count += 2;             // 7
console.log(5 % 2);     // 1
console.log(2 ** 3);    // 8</code></pre>
<p><code>+</code>는 문자열이 섞이면 연결 연산을 할 수 있다.</p>
<pre><code class="language-javascript">console.log(5 + 3);     // 8
console.log(&quot;5&quot; + 3);   // &quot;53&quot;
console.log(Number(&quot;5&quot;) + 3); // 8</code></pre>
<h3 id="비교-연산자">비교 연산자</h3>
<table>
<thead>
<tr>
<th>연산자</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>===</code>, <code>!==</code></td>
<td>값과 타입까지 엄격하게 비교 - 권장</td>
</tr>
<tr>
<td><code>==</code>, <code>!=</code></td>
<td>타입 변환 후 느슨하게 비교 - 예측이 어려워 지양</td>
</tr>
<tr>
<td><code>&lt;</code>, <code>&gt;</code>, <code>&lt;=</code>, <code>&gt;=</code></td>
<td>크기 비교</td>
</tr>
</tbody></table>
<pre><code class="language-javascript">console.log(5 == &quot;5&quot;);  // true
console.log(5 === &quot;5&quot;); // false</code></pre>
<h3 id="논리-연산자와-단축-평가">논리 연산자와 단축 평가</h3>
<pre><code class="language-javascript">const canEnter = hasTicket &amp;&amp; age &gt;= 19;
const displayName = nickname || &quot;익명&quot;;
const isClosed = !isOpen;</code></pre>
<p><code>&amp;&amp;</code>와 <code>||</code>는 반드시 boolean만 반환하는 것이 아니라 평가를 멈춘 위치의 값을 반환한다.</p>
<pre><code class="language-javascript">console.log(&quot;hello&quot; &amp;&amp; 42); // 42
console.log(&quot;&quot; || &quot;기본값&quot;); // &quot;기본값&quot;</code></pre>
<p><code>??</code>는 왼쪽이 <code>null</code> 또는 <code>undefined</code>일 때만 오른쪽을 사용한다.</p>
<pre><code class="language-javascript">const savedVolume = 0;
console.log(savedVolume || 50); // 50
console.log(savedVolume ?? 50); // 0</code></pre>
<h3 id="삼항-연산자와-기타-연산자">삼항 연산자와 기타 연산자</h3>
<pre><code class="language-javascript">const result = score &gt;= 60 ? &quot;합격&quot; : &quot;불합격&quot;;
console.log(typeof result);             // &quot;string&quot;
console.log(&quot;name&quot; in user);           // property 존재 여부
console.log(items instanceof Array);    // prototype 관계 확인</code></pre>
<h3 id="비트-연산자">비트 연산자</h3>
<p><code>&amp;</code>, <code>|</code>, <code>^</code>, <code>~</code>, <code>&lt;&lt;</code>, <code>&gt;&gt;</code>, <code>&gt;&gt;&gt;</code>는 숫자를 32비트 정수로 변환해 비트 단위로 연산한다. 일반적인 입문 웹 UI에서는 자주 쓰이지 않는다.</p>
<pre><code class="language-javascript">console.log(5 &amp; 1);  // 1
console.log(5 | 1);  // 5
console.log(5 ^ 1);  // 4
console.log(~5);     // -6
console.log(5 &gt;&gt; 1); // 2</code></pre>
<hr />
<h2 id="7-조건문">7. 조건문</h2>
<h3 id="if-else-if-else"><code>if</code>, <code>else if</code>, <code>else</code></h3>
<pre><code class="language-javascript">function getGrade(score) {
  if (score &gt;= 90) {
    return &quot;A&quot;;
  } else if (score &gt;= 80) {
    return &quot;B&quot;;
  } else if (score &gt;= 70) {
    return &quot;C&quot;;
  } else {
    return &quot;F&quot;;
  }
}</code></pre>
<p>조건 범위를 위에서부터 순서대로 검사하므로 넓은 조건을 먼저 쓰면 뒤 조건이 실행되지 않을 수 있다.</p>
<h3 id="guard-clause">Guard clause</h3>
<p>예외 상황을 먼저 반환하면 중첩을 줄일 수 있다.</p>
<pre><code class="language-javascript">function divide(a, b) {
  if (b === 0) {
    return &quot;0으로 나눌 수 없습니다.&quot;;
  }

  return a / b;
}</code></pre>
<h3 id="switch"><code>switch</code></h3>
<pre><code class="language-javascript">function getDayType(day) {
  switch (day) {
    case 0:
    case 6:
      return &quot;주말&quot;;
    default:
      return &quot;평일&quot;;
  }
}</code></pre>
<ul>
<li>case 비교는 엄격 비교처럼 타입을 구분한다.</li>
<li><code>break</code> 또는 <code>return</code>이 없으면 다음 case까지 이어서 실행되는 fall-through가 발생한다.</li>
<li><code>default</code>는 일치하는 case가 없을 때 실행된다.</li>
</ul>
<hr />
<h2 id="8-반복문">8. 반복문</h2>
<h3 id="반복-방식-선택">반복 방식 선택</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>적합한 상황</th>
</tr>
</thead>
<tbody><tr>
<td><code>for</code></td>
<td>횟수·index가 필요</td>
</tr>
<tr>
<td><code>while</code></td>
<td>반복 횟수보다 종료 조건이 중요</td>
</tr>
<tr>
<td><code>do...while</code></td>
<td>최소 한 번 실행해야 함</td>
</tr>
<tr>
<td><code>for...of</code></td>
<td>배열·문자열의 값 순회</td>
</tr>
<tr>
<td><code>for...in</code></td>
<td>객체의 enumerable property key 순회</td>
</tr>
<tr>
<td><code>forEach()</code></td>
<td>배열의 각 요소에 작업 수행</td>
</tr>
</tbody></table>
<h3 id="for"><code>for</code></h3>
<pre><code class="language-javascript">const fruits = [&quot;사과&quot;, &quot;바나나&quot;, &quot;포도&quot;];

for (let index = 0; index &lt; fruits.length; index += 1) {
  console.log(index, fruits[index]);
}</code></pre>
<h3 id="while과-dowhile"><code>while</code>과 <code>do...while</code></h3>
<pre><code class="language-javascript">let count = 0;

while (count &lt; 3) {
  console.log(count);
  count += 1;
}

do {
  console.log(&quot;최소 한 번 실행&quot;);
} while (false);</code></pre>
<p>조건을 영원히 만족하면 무한 반복이 되므로 상태가 실제로 변하는지 확인한다.</p>
<h3 id="break-continue"><code>break</code>, <code>continue</code></h3>
<pre><code class="language-javascript">for (let number = 1; number &lt;= 10; number += 1) {
  if (number === 8) break;
  if (number % 2 === 0) continue;
  console.log(number); // 1, 3, 5, 7
}</code></pre>
<h3 id="forof"><code>for...of</code></h3>
<pre><code class="language-javascript">for (const fruit of fruits) {
  console.log(fruit);
}</code></pre>
<p>배열의 <strong>값</strong>을 읽을 때 가장 직관적이다.</p>
<h3 id="forin"><code>for...in</code></h3>
<pre><code class="language-javascript">const user = { name: &quot;Kim&quot;, age: 28 };

for (const key in user) {
  if (Object.hasOwn(user, key)) {
    console.log(key, user[key]);
  }
}</code></pre>
<p><code>for...in</code>은 배열보다 일반 객체의 key 순회에 사용한다. 배열에서는 <code>for...of</code>, <code>forEach</code>, <code>map</code> 등이 더 적합하다.</p>
<h3 id="중첩-반복문">중첩 반복문</h3>
<pre><code class="language-javascript">for (let row = 1; row &lt;= 3; row += 1) {
  for (let column = 1; column &lt;= 3; column += 1) {
    console.log({ row, column });
  }
}</code></pre>
<p>깊게 중첩하면 실행 횟수와 복잡도가 빠르게 늘어난다. 로직을 함수로 분리할 수 있는지 검토한다.</p>
<hr />
<h2 id="9-함수">9. 함수</h2>
<p><strong>한 문장 이해:</strong> 함수는 입력을 받아 특정 작업을 수행하고 결과를 반환할 수 있는 재사용 가능한 코드이다.</p>
<pre><code class="language-javascript">function calculateTotal(price, quantity) {
  return price * quantity;
}

const total = calculateTotal(3000, 2);</code></pre>
<ul>
<li>parameter(매개변수): 함수 정의의 <code>price</code>, <code>quantity</code></li>
<li>argument(인수): 호출할 때 전달한 <code>3000</code>, <code>2</code></li>
<li><code>return</code>: 결과를 돌려주고 함수 실행을 즉시 종료</li>
<li>return이 없으면 결과는 <code>undefined</code></li>
</ul>
<h3 id="기본-매개변수">기본 매개변수</h3>
<pre><code class="language-javascript">function greet(name = &quot;손님&quot;) {
  return `안녕하세요, ${name}님`;
}

console.log(greet());</code></pre>
<h3 id="호출과-참조">호출과 참조</h3>
<pre><code class="language-javascript">function sayHello() {
  return &quot;Hello&quot;;
}

const reference = sayHello; // 함수 자체
const result = sayHello();  // 함수를 호출한 결과</code></pre>
<p>이벤트에 함수를 전달할 때는 보통 참조를 전달한다.</p>
<pre><code class="language-javascript">button.addEventListener(&quot;click&quot;, sayHello);</code></pre>
<h3 id="선언문-표현식-화살표-함수">선언문, 표현식, 화살표 함수</h3>
<pre><code class="language-javascript">// 함수 선언문
function add(a, b) {
  return a + b;
}

// 함수 표현식
const subtract = function (a, b) {
  return a - b;
};

// 화살표 함수
const multiply = (a, b) =&gt; a * b;</code></pre>
<ul>
<li>함수 선언문은 선언 전 호출이 가능하다.</li>
<li><code>const</code> 함수 표현식은 선언문 이후에 사용한다.</li>
<li>화살표 함수는 간결하지만 자신의 <code>this</code>와 <code>arguments</code>가 없다.</li>
<li>객체 method나 constructor처럼 동적인 <code>this</code>가 필요한 곳에서는 일반 함수가 더 적합할 수 있다.</li>
</ul>
<h3 id="인수-전달의-정확한-이해">인수 전달의 정확한 이해</h3>
<p>JavaScript의 인수는 모두 <strong>값으로 전달</strong>된다. 객체를 전달할 때 복사되는 값이 객체를 가리키는 참조이므로 두 코드가 같은 객체를 공유하게 된다.</p>
<pre><code class="language-javascript">function changeNumber(number) {
  number = 20;
}

let score = 10;
changeNumber(score);
console.log(score); // 10

function changeUser(object) {
  object.name = &quot;Lee&quot;;
}

const user = { name: &quot;Kim&quot; };
changeUser(user);
console.log(user.name); // Lee</code></pre>
<p>함수 안에서 parameter 자체에 새 객체를 대입하면 원래 변수는 바뀌지 않는다.</p>
<pre><code class="language-javascript">function replaceUser(object) {
  object = { name: &quot;Park&quot; };
}

replaceUser(user);
console.log(user.name); // Lee</code></pre>
<hr />
<h2 id="10-배열">10. 배열</h2>
<p>배열은 순서가 있는 값의 모음이며 index는 0부터 시작한다.</p>
<pre><code class="language-javascript">const fruits = [&quot;사과&quot;, &quot;바나나&quot;, &quot;포도&quot;];

console.log(fruits[0]);      // 사과
console.log(fruits.at(-1));  // 포도
console.log(fruits.length);  // 3</code></pre>
<p>배열 리터럴 <code>[]</code>을 쓰는 방식이 가장 명확하다.</p>
<h3 id="원본을-변경하는-메서드">원본을 변경하는 메서드</h3>
<table>
<thead>
<tr>
<th>메서드</th>
<th>동작</th>
<th>반환값</th>
</tr>
</thead>
<tbody><tr>
<td><code>push(value)</code></td>
<td>뒤에 추가</td>
<td>새 length</td>
</tr>
<tr>
<td><code>pop()</code></td>
<td>뒤에서 제거</td>
<td>제거된 값</td>
</tr>
<tr>
<td><code>unshift(value)</code></td>
<td>앞에 추가</td>
<td>새 length</td>
</tr>
<tr>
<td><code>shift()</code></td>
<td>앞에서 제거</td>
<td>제거된 값</td>
</tr>
<tr>
<td><code>splice()</code></td>
<td>중간 추가·삭제</td>
<td>제거된 값 배열</td>
</tr>
<tr>
<td><code>sort()</code></td>
<td>배열 자체 정렬</td>
<td>같은 배열</td>
</tr>
<tr>
<td><code>reverse()</code></td>
<td>배열 자체 뒤집기</td>
<td>같은 배열</td>
</tr>
</tbody></table>
<h3 id="원본을-변경하지-않는-주요-메서드">원본을 변경하지 않는 주요 메서드</h3>
<table>
<thead>
<tr>
<th>메서드</th>
<th>결과</th>
</tr>
</thead>
<tbody><tr>
<td><code>slice(start, end)</code></td>
<td>일부를 복사한 새 배열</td>
</tr>
<tr>
<td><code>concat()</code></td>
<td>배열을 합친 새 배열</td>
</tr>
<tr>
<td><code>toSorted()</code></td>
<td>정렬된 새 배열</td>
</tr>
<tr>
<td><code>toReversed()</code></td>
<td>뒤집힌 새 배열</td>
</tr>
<tr>
<td><code>toSpliced()</code></td>
<td>splice 결과를 반영한 새 배열</td>
</tr>
</tbody></table>
<h3 id="검색">검색</h3>
<pre><code class="language-javascript">const numbers = [10, 20, 30, 40];

numbers.includes(20);                  // true
numbers.indexOf(30);                   // 2
numbers.find((number) =&gt; number &gt; 25); // 30
numbers.findIndex((number) =&gt; number &gt; 25); // 2</code></pre>
<h3 id="순회·변환">순회·변환</h3>
<pre><code class="language-javascript">const numbers = [1, 2, 3, 4];

numbers.forEach((number) =&gt; console.log(number));

const doubled = numbers.map((number) =&gt; number * 2);
const evens = numbers.filter((number) =&gt; number % 2 === 0);
const total = numbers.reduce((sum, number) =&gt; sum + number, 0);
const allPositive = numbers.every((number) =&gt; number &gt; 0);
const hasEven = numbers.some((number) =&gt; number % 2 === 0);</code></pre>
<table>
<thead>
<tr>
<th>메서드</th>
<th>핵심 질문</th>
<th>반환</th>
</tr>
</thead>
<tbody><tr>
<td><code>forEach</code></td>
<td>각 값을 이용해 무엇을 실행할까?</td>
<td><code>undefined</code></td>
</tr>
<tr>
<td><code>map</code></td>
<td>각 값을 무엇으로 바꿀까?</td>
<td>같은 길이의 새 배열</td>
</tr>
<tr>
<td><code>filter</code></td>
<td>어떤 값만 남길까?</td>
<td>새 배열</td>
</tr>
<tr>
<td><code>find</code></td>
<td>조건을 만족하는 첫 값은?</td>
<td>값 또는 <code>undefined</code></td>
</tr>
<tr>
<td><code>reduce</code></td>
<td>하나의 결과로 어떻게 누적할까?</td>
<td>누적 결과</td>
</tr>
<tr>
<td><code>every</code> / <code>some</code></td>
<td>모두/하나라도 조건을 만족하는가?</td>
<td>boolean</td>
</tr>
</tbody></table>
<h3 id="숫자-정렬-주의">숫자 정렬 주의</h3>
<pre><code class="language-javascript">const scores = [100, 2, 30];

console.log(scores.toSorted());           // [100, 2, 30] 문자열 기준
console.log(scores.toSorted((a, b) =&gt; a - b)); // [2, 30, 100]</code></pre>
<h3 id="spread">Spread</h3>
<pre><code class="language-javascript">const first = [1, 2];
const second = [3, 4];
const combined = [...first, ...second];

console.log(Math.min(...combined));</code></pre>
<p>spread로 만든 배열 복사는 얕은 복사이므로 중첩 객체까지 새로 복사하지 않는다.</p>
<hr />
<h2 id="11-객체">11. 객체</h2>
<p>객체는 key와 value의 쌍으로 데이터를 표현한다.</p>
<pre><code class="language-javascript">const person = {
  firstName: &quot;길동&quot;,
  lastName: &quot;홍&quot;,
  age: 25,
  getFullName() {
    return `${this.lastName}${this.firstName}`;
  },
};</code></pre>
<h3 id="property-접근">Property 접근</h3>
<pre><code class="language-javascript">console.log(person.age);             // dot notation
console.log(person[&quot;firstName&quot;]);   // bracket notation

const key = &quot;age&quot;;
console.log(person[key]);            // 동적인 key는 bracket 필요</code></pre>
<pre><code class="language-javascript">person.job = &quot;개발자&quot;;   // 추가
person.age = 26;         // 변경
delete person.job;       // 삭제
console.log(&quot;age&quot; in person);</code></pre>
<p>property 존재 여부를 구분할 때 값과 직접 비교하지 않는다. 존재하지만 값이 <code>undefined</code>일 수도 있다.</p>
<pre><code class="language-javascript">console.log(Object.hasOwn(person, &quot;age&quot;));</code></pre>
<h3 id="this와-method"><code>this</code>와 method</h3>
<pre><code class="language-javascript">console.log(person.getFullName());</code></pre>
<p>위 method의 <code>this</code>는 <code>person.getFullName()</code>처럼 호출했을 때 점 앞의 객체를 가리킨다. <code>this</code>는 함수가 어디에 작성되었는지보다 <strong>어떻게 호출되었는지</strong>의 영향을 받는다. 화살표 함수에는 자체 <code>this</code>가 없다.</p>
<h3 id="객체를-읽기-좋은-형태로-출력">객체를 읽기 좋은 형태로 출력</h3>
<pre><code class="language-javascript">console.log(Object.keys(person));
console.log(Object.values(person));
console.log(Object.entries(person));
console.log(JSON.stringify(person, null, 2));</code></pre>
<p>문자열과 객체를 <code>+</code>로 단순 결합하면 <code>[object Object]</code>가 나올 수 있다.</p>
<h3 id="constructor-function">Constructor function</h3>
<pre><code class="language-javascript">function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.introduce = function () {
  return `저는 ${this.name}입니다.`;
};

const student = new Person(&quot;민수&quot;, 20);</code></pre>
<p>constructor 함수는 관례상 대문자로 시작하고 <code>new</code>로 호출한다. 현대 코드에서는 <code>class</code> 문법이나 객체 생성 함수를 쓰기도 한다.</p>
<hr />
<h2 id="12-주요-내장-객체">12. 주요 내장 객체</h2>
<h2 id="121-math">12.1 Math</h2>
<p><code>Math</code>는 instance를 만들지 않고 바로 사용한다.</p>
<pre><code class="language-javascript">Math.PI;
Math.round(4.6); // 5
Math.floor(4.9); // 4
Math.ceil(4.1);  // 5
Math.trunc(4.9); // 4
Math.min(3, 1, 5);
Math.max(3, 1, 5);</code></pre>
<p>1~50 무작위 정수:</p>
<pre><code class="language-javascript">const randomNumber = Math.floor(Math.random() * 50) + 1;</code></pre>
<p><code>Math.random()</code>은 보안용 난수로 사용하면 안 된다.</p>
<h2 id="122-date">12.2 Date</h2>
<pre><code class="language-javascript">const now = new Date();
const date = new Date(&quot;2026-07-24T09:00:00+09:00&quot;);

console.log(now.getFullYear());
console.log(now.getMonth()); // 0~11
console.log(now.getDate());  // 1~31
console.log(now.toISOString());</code></pre>
<ul>
<li>내부적으로 1970-01-01T00:00:00Z 이후의 millisecond를 기반으로 표현한다.</li>
<li><code>getMonth()</code>는 0부터 시작한다.</li>
<li>날짜 문자열 parsing은 형식과 시간대 때문에 오해가 생기기 쉬우므로 ISO 형식과 명시적인 시간대를 사용한다.</li>
</ul>
<h2 id="123-json">12.3 JSON</h2>
<p>JSON은 데이터 교환에 쓰는 <strong>문자열 형식</strong>이다.</p>
<pre><code class="language-javascript">const user = { name: &quot;홍길동&quot;, age: 15 };
const jsonText = JSON.stringify(user);
const restored = JSON.parse(jsonText);</code></pre>
<table>
<thead>
<tr>
<th>메서드</th>
<th>방향</th>
</tr>
</thead>
<tbody><tr>
<td><code>JSON.stringify(value)</code></td>
<td>JavaScript 값 → JSON 문자열</td>
</tr>
<tr>
<td><code>JSON.parse(text)</code></td>
<td>JSON 문자열 → JavaScript 값</td>
</tr>
</tbody></table>
<p>주의:</p>
<ul>
<li>JSON의 property 이름과 문자열은 큰따옴표를 사용한다.</li>
<li>함수, <code>undefined</code>, Symbol은 일반적인 JSON 데이터로 온전히 표현되지 않는다.</li>
<li>BigInt를 그대로 stringify하면 오류가 발생한다.</li>
<li>신뢰할 수 없는 JSON은 <code>try...catch</code>로 parsing 오류를 처리한다.</li>
</ul>
<h2 id="124-regexp">12.4 RegExp</h2>
<pre><code class="language-javascript">const phonePattern = /^010-\d{4}-\d{4}$/;

console.log(phonePattern.test(&quot;010-1234-5678&quot;)); // true</code></pre>
<ul>
<li><code>/pattern/flags</code> 형식</li>
<li><code>i</code>: 대소문자 무시</li>
<li><code>g</code>: 모든 일치 검색</li>
<li><code>^</code>: 문자열 시작, <code>$</code>: 문자열 끝</li>
<li><code>\d</code>: 숫자, <code>{4}</code>: 정확히 네 번</li>
</ul>
<p>정규식만으로 모든 입력을 검증하려 하지 말고, 사용 목적에 맞는 수준으로 사용한다.</p>
<h2 id="125-set">12.5 Set</h2>
<p>중복 없는 값의 집합이다.</p>
<pre><code class="language-javascript">const tags = new Set([&quot;html&quot;, &quot;css&quot;, &quot;html&quot;]);

tags.add(&quot;javascript&quot;);
tags.has(&quot;css&quot;);
tags.delete(&quot;html&quot;);

const unique = [...new Set([1, 1, 2, 3])];</code></pre>
<h2 id="126-map">12.6 Map</h2>
<p>다양한 타입의 key를 사용할 수 있는 key-value collection이다.</p>
<pre><code class="language-javascript">const roles = new Map();

roles.set(&quot;kim&quot;, &quot;admin&quot;);
roles.set(42, &quot;guest&quot;);

console.log(roles.get(&quot;kim&quot;));
console.log(roles.has(42));
console.log(roles.size);</code></pre>
<p>객체는 일반적인 record 표현에, Map은 key 추가·삭제와 순회가 빈번하거나 문자열 외 key가 필요할 때 잘 맞는다.</p>
<hr />
<h2 id="13-원본-과제-복습">13. 원본 과제 복습</h2>
<h3 id="131-up-down-게임-핵심-로직">13.1 Up-Down 게임 핵심 로직</h3>
<pre><code class="language-javascript">function playUpDown() {
  const answer = Math.floor(Math.random() * 50) + 1;
  let attempts = 0;

  while (true) {
    const input = prompt(&quot;1부터 50 사이 숫자를 입력하세요. 취소하면 종료합니다.&quot;);

    if (input === null) {
      alert(&quot;게임을 종료합니다.&quot;);
      return;
    }

    const guess = Number(input);

    if (!Number.isInteger(guess) || guess &lt; 1 || guess &gt; 50) {
      alert(&quot;1부터 50 사이의 정수를 입력하세요.&quot;);
      continue;
    }

    attempts += 1;

    if (guess === answer) {
      alert(`축하합니다! ${attempts}번 만에 맞혔습니다.`);
      return;
    }

    alert(guess &gt; answer ? &quot;Down!&quot; : &quot;Up!&quot;);
  }
}</code></pre>
<p>복습 포인트: 무작위 숫자, <code>while</code>, 입력 변환·검증, <code>continue</code>, <code>return</code>, 삼항 연산자.</p>
<h3 id="132-성적-계산기-핵심-로직">13.2 성적 계산기 핵심 로직</h3>
<pre><code class="language-javascript">function calculateGrades() {
  const subjects = [&quot;HTML&quot;, &quot;CSS&quot;, &quot;JavaScript&quot;];
  const scores = [];

  for (const subject of subjects) {
    const score = Number(prompt(`${subject} 점수를 입력하세요.`));

    if (!Number.isFinite(score) || score &lt; 0 || score &gt; 100) {
      alert(&quot;0~100 사이 점수를 입력하세요.&quot;);
      return;
    }

    scores.push(score);
  }

  const total = scores.reduce((sum, score) =&gt; sum + score, 0);
  const average = total / scores.length;
  const pass = average &gt;= 60 ? &quot;합격&quot; : &quot;불합격&quot;;

  alert(`총점: ${total}, 평균: ${average.toFixed(1)}, 결과: ${pass}`);
}</code></pre>
<h3 id="133-가방-객체-출력">13.3 가방 객체 출력</h3>
<pre><code class="language-javascript">function showMyBag() {
  const myBag = [
    { name: &quot;노트북&quot;, quantity: 1 },
    { name: &quot;충전기&quot;, quantity: 2 },
    { name: &quot;노트&quot;, quantity: 1 },
  ];

  return myBag
    .map((item) =&gt; `${item.name}: ${item.quantity}개`)
    .join(&quot;\n&quot;);
}

console.log(showMyBag());</code></pre>
<hr />
<h2 id="14-기초편-핵심-정리">14. 기초편 핵심 정리</h2>
<ul>
<li>값이 바뀌지 않는 변수는 <code>const</code>, 재할당이 필요하면 <code>let</code>을 쓴다.</li>
<li>원시값은 값으로 비교하고 객체는 같은 객체를 가리키는지 비교한다.</li>
<li>비교에는 기본적으로 <code>===</code>, <code>!==</code>를 사용한다.</li>
<li>반복 방식은 목적에 맞게 고른다. 배열 값은 <code>for...of</code>, 객체 key는 <code>for...in</code> 또는 <code>Object.keys()</code>가 적합하다.</li>
<li>함수는 매개변수로 입력을 받고 <code>return</code>으로 결과를 돌려준다.</li>
<li><code>map</code>, <code>filter</code>, <code>reduce</code>는 배열을 변환·선별·누적하는 핵심 도구이다.</li>
<li>객체는 관련 데이터를 key-value로 묶고 method로 행동을 표현한다.</li>
<li>JSON은 객체가 아니라 객체와 데이터를 주고받기 위한 문자열 형식이다.</li>
</ul>