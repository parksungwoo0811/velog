<h2 id="1-핵심-문법-치트시트">1. 핵심 문법 치트시트</h2>
<h3 id="변수와-자료형">변수와 자료형</h3>
<pre><code class="language-javascript">const name = &quot;Kim&quot;;       // 재할당하지 않을 값
let score = 80;           // 재할당할 값

typeof &quot;hello&quot;;           // &quot;string&quot;
typeof 10;                // &quot;number&quot;
typeof true;              // &quot;boolean&quot;
typeof undefined;         // &quot;undefined&quot;
typeof null;              // &quot;object&quot; - 역사적 특이점
typeof {};                // &quot;object&quot;
typeof [];                // &quot;object&quot;
typeof function () {};    // &quot;function&quot;

Array.isArray([]);        // true</code></pre>
<h3 id="문자열과-숫자">문자열과 숫자</h3>
<pre><code class="language-javascript">const message = `점수: ${score}`;

&quot; JavaScript &quot;.trim();
&quot;JavaScript&quot;.includes(&quot;Script&quot;);
&quot;a,b,c&quot;.split(&quot;,&quot;);

Number(&quot;42&quot;);
Number.parseInt(&quot;42px&quot;, 10);
Number.isNaN(Number(&quot;hello&quot;));
(12.345).toFixed(2); // &quot;12.35&quot;</code></pre>
<h3 id="비교와-기본값">비교와 기본값</h3>
<pre><code class="language-javascript">5 === &quot;5&quot;;              // false
5 !== &quot;5&quot;;              // true

const label = nickname || &quot;익명&quot;;    // 모든 falsy에 기본값
const count = savedCount ?? 0;        // null/undefined에만 기본값
const result = score &gt;= 60 ? &quot;합격&quot; : &quot;불합격&quot;;</code></pre>
<h3 id="조건과-반복">조건과 반복</h3>
<pre><code class="language-javascript">if (score &gt;= 90) {
  console.log(&quot;A&quot;);
} else if (score &gt;= 80) {
  console.log(&quot;B&quot;);
} else {
  console.log(&quot;C 이하&quot;);
}

for (let index = 0; index &lt; items.length; index += 1) {}
for (const item of items) {}
for (const key in object) {}
while (condition) {}</code></pre>
<h3 id="함수">함수</h3>
<pre><code class="language-javascript">function add(a, b = 0) {
  return a + b;
}

const multiply = (a, b) =&gt; a * b;

const functionReference = add;
const functionResult = add(1, 2);</code></pre>
<h3 id="배열">배열</h3>
<pre><code class="language-javascript">const numbers = [1, 2, 3, 4];

numbers.push(5);                              // 원본 변경
numbers.at(-1);                              // 마지막 값
numbers.includes(3);                         // 포함 여부
numbers.find((n) =&gt; n &gt; 2);                  // 첫 일치 값
numbers.map((n) =&gt; n * 2);                   // 변환한 새 배열
numbers.filter((n) =&gt; n % 2 === 0);          // 선별한 새 배열
numbers.reduce((sum, n) =&gt; sum + n, 0);      // 하나로 누적
numbers.toSorted((a, b) =&gt; a - b);           // 숫자 오름차순 새 배열</code></pre>
<h3 id="객체">객체</h3>
<pre><code class="language-javascript">const user = {
  name: &quot;Kim&quot;,
  age: 20,
  introduce() {
    return `저는 ${this.name}입니다.`;
  },
};

user.name;
user[&quot;age&quot;];
&quot;name&quot; in user;
Object.hasOwn(user, &quot;name&quot;);
Object.keys(user);
Object.values(user);
Object.entries(user);</code></pre>
<h3 id="json-set-map">JSON, Set, Map</h3>
<pre><code class="language-javascript">const text = JSON.stringify(user);
const restored = JSON.parse(text);

const unique = [...new Set([1, 1, 2])];

const map = new Map();
map.set(&quot;role&quot;, &quot;admin&quot;);
map.get(&quot;role&quot;);</code></pre>
<h3 id="dom">DOM</h3>
<pre><code class="language-javascript">const button = document.querySelector(&quot;#save-button&quot;);
const cards = document.querySelectorAll(&quot;.card&quot;);

button.textContent = &quot;저장&quot;;
button.classList.add(&quot;active&quot;);
button.setAttribute(&quot;aria-pressed&quot;, &quot;true&quot;);

const item = document.createElement(&quot;li&quot;);
item.textContent = &quot;새 항목&quot;;
document.querySelector(&quot;ul&quot;).append(item);</code></pre>
<h3 id="event">Event</h3>
<pre><code class="language-javascript">button.addEventListener(&quot;click&quot;, handleClick);

function handleClick(event) {
  console.log(event.target);
  console.log(event.currentTarget);
}

form.addEventListener(&quot;submit&quot;, (event) =&gt; {
  event.preventDefault();
});</code></pre>
<h3 id="promise와-asyncawait">Promise와 async/await</h3>
<pre><code class="language-javascript">getData()
  .then((data) =&gt; render(data))
  .catch((error) =&gt; showError(error))
  .finally(() =&gt; hideLoading());

async function run() {
  try {
    const data = await getData();
    render(data);
  } catch (error) {
    showError(error);
  } finally {
    hideLoading();
  }
}</code></pre>
<h3 id="fetch">Fetch</h3>
<pre><code class="language-javascript">async function getData() {
  const response = await fetch(&quot;/api/data&quot;);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}</code></pre>
<h3 id="storage">Storage</h3>
<pre><code class="language-javascript">localStorage.setItem(&quot;theme&quot;, &quot;dark&quot;);
localStorage.getItem(&quot;theme&quot;);
localStorage.removeItem(&quot;theme&quot;);

localStorage.setItem(&quot;settings&quot;, JSON.stringify(settings));
const saved = JSON.parse(localStorage.getItem(&quot;settings&quot;) ?? &quot;null&quot;);</code></pre>
<h3 id="module">Module</h3>
<pre><code class="language-javascript">// api.js
export async function getData() {}

// app.js
import { getData } from &quot;./api.js&quot;;</code></pre>
<pre><code class="language-html">&lt;script type=&quot;module&quot; src=&quot;script/app.js&quot;&gt;&lt;/script&gt;</code></pre>
<hr />
<h2 id="2-무엇을-선택해야-할까">2. 무엇을 선택해야 할까?</h2>
<table>
<thead>
<tr>
<th>상황</th>
<th>우선 고려</th>
</tr>
</thead>
<tbody><tr>
<td>값이 바뀌지 않음</td>
<td><code>const</code></td>
</tr>
<tr>
<td>재할당 필요</td>
<td><code>let</code></td>
</tr>
<tr>
<td>타입까지 안전하게 비교</td>
<td><code>===</code>, <code>!==</code></td>
</tr>
<tr>
<td>배열의 각 값 읽기</td>
<td><code>for...of</code></td>
</tr>
<tr>
<td>배열의 각 값을 변환</td>
<td><code>map</code></td>
</tr>
<tr>
<td>조건에 맞는 배열 생성</td>
<td><code>filter</code></td>
</tr>
<tr>
<td>배열을 하나의 값으로 누적</td>
<td><code>reduce</code></td>
</tr>
<tr>
<td>객체의 key-value 순회</td>
<td><code>Object.entries()</code></td>
</tr>
<tr>
<td>요소 하나 선택</td>
<td><code>querySelector()</code></td>
</tr>
<tr>
<td>여러 요소 선택</td>
<td><code>querySelectorAll()</code></td>
</tr>
<tr>
<td>일반 문자열을 화면에 표시</td>
<td><code>textContent</code></td>
</tr>
<tr>
<td>여러 자식 event를 부모에서 처리</td>
<td>event delegation</td>
</tr>
<tr>
<td>Promise 코드를 순차적으로 읽기</td>
<td><code>async/await</code></td>
</tr>
<tr>
<td>독립적인 Promise 여러 개</td>
<td><code>Promise.all()</code></td>
</tr>
<tr>
<td>서버 요청</td>
<td><code>fetch</code> + <code>response.ok</code> 확인</td>
</tr>
<tr>
<td>브라우저에 간단한 설정 저장</td>
<td>localStorage + JSON</td>
</tr>
<tr>
<td>파일 사이 기능 공유</td>
<td>ES module import/export</td>
</tr>
</tbody></table>
<hr />
<h2 id="3-자주-하는-실수와-교정">3. 자주 하는 실수와 교정</h2>
<table>
<thead>
<tr>
<th>실수</th>
<th>문제</th>
<th>교정</th>
</tr>
</thead>
<tbody><tr>
<td><code>var</code>를 습관적으로 사용</td>
<td>function scope·재선언으로 혼란</td>
<td>기본 <code>const</code>, 필요 시 <code>let</code></td>
</tr>
<tr>
<td><code>const</code> 객체는 수정 불가라고 생각</td>
<td>변수 재할당과 내부 변경을 혼동</td>
<td>property 변경은 가능하다고 구분</td>
</tr>
<tr>
<td><code>5 == &quot;5&quot;</code> 사용</td>
<td>암묵적 타입 변환</td>
<td><code>5 === &quot;5&quot;</code></td>
</tr>
<tr>
<td>prompt 값을 바로 더함</td>
<td>입력은 문자열</td>
<td><code>Number(input)</code> 후 검증</td>
</tr>
<tr>
<td><code>NaN === NaN</code>으로 검사</td>
<td>항상 false</td>
<td><code>Number.isNaN(value)</code></td>
</tr>
<tr>
<td><code>if ([])</code>가 false라고 생각</td>
<td>빈 배열·객체도 truthy</td>
<td>falsy 목록 암기</td>
</tr>
<tr>
<td><code>typeof null === &quot;null&quot;</code> 예상</td>
<td>실제 결과는 <code>&quot;object&quot;</code></td>
<td><code>value === null</code>로 검사</td>
</tr>
<tr>
<td>숫자 배열에 <code>.sort()</code>만 사용</td>
<td>문자열 기준 정렬</td>
<td><code>.sort((a, b) =&gt; a - b)</code></td>
</tr>
<tr>
<td><code>for...in</code>으로 배열 값 순회</td>
<td>key가 문자열이며 추가 property 위험</td>
<td><code>for...of</code> 사용</td>
</tr>
<tr>
<td><code>map()</code>에서 return 누락</td>
<td>결과가 <code>undefined</code> 배열</td>
<td>callback 결과 return</td>
</tr>
<tr>
<td>함수 참조 대신 <code>handler()</code> 전달</td>
<td>등록 시 즉시 실행</td>
<td><code>handler</code> 전달</td>
</tr>
<tr>
<td>신뢰할 수 없는 값을 <code>innerHTML</code>에 넣음</td>
<td>XSS 위험</td>
<td><code>textContent</code>, <code>createElement</code></td>
</tr>
<tr>
<td>listener를 익명 함수로 등록 후 다른 익명 함수로 제거</td>
<td>함수 참조가 다름</td>
<td>이름 있는 동일 함수 사용</td>
</tr>
<tr>
<td><code>preventDefault()</code>와 <code>stopPropagation()</code> 혼동</td>
<td>기본 동작과 전파는 다른 개념</td>
<td>목적에 맞는 메서드 사용</td>
</tr>
<tr>
<td>Promise chain에서 return 누락</td>
<td>다음 단계가 기다리지 않음</td>
<td>Promise를 return</td>
</tr>
<tr>
<td><code>await</code>가 전체 JS를 멈춘다고 생각</td>
<td>현재 async 함수만 중단</td>
<td>event loop 흐름 이해</td>
</tr>
<tr>
<td>fetch가 404에서 catch로 간다고 생각</td>
<td>HTTP 오류는 fulfilled response일 수 있음</td>
<td><code>response.ok</code> 검사</td>
</tr>
<tr>
<td>localStorage에 객체를 직접 저장</td>
<td><code>[object Object]</code>가 저장됨</td>
<td><code>JSON.stringify/parse</code></td>
</tr>
<tr>
<td>module을 파일 더블클릭으로 실행</td>
<td>CORS·경로 문제</td>
<td>Live Server로 실행</td>
</tr>
</tbody></table>
<h3 id="원본-슬라이드에서-교정해-기억할-내용">원본 슬라이드에서 교정해 기억할 내용</h3>
<ul>
<li>논리 OR는 <code>||</code>이다. <code>!!</code>는 값을 boolean으로 변환할 때 쓰는 double NOT이다.</li>
<li><code>5 ^ 1</code>의 결과는 <code>4</code>이다.</li>
<li><code>~5</code>의 결과는 <code>-6</code>이다.</li>
<li>모든 함수 인수는 값으로 전달된다. 객체에서는 그 값이 객체를 가리키는 참조이므로 내부 변경이 공유된다.</li>
<li><code>null</code>은 원시값이며 <code>typeof null</code>의 <code>&quot;object&quot;</code> 결과는 역사적 특이점이다.</li>
<li><code>innerHTML</code>보다 일반 문자열에는 <code>textContent</code>가 안전하다.</li>
<li>localStorage는 민감한 인증 정보를 무조건 안전하게 보관하는 장소가 아니다.</li>
<li>fetch는 HTTP 404·500에서 자동으로 reject되지 않으므로 <code>response.ok</code>를 확인한다.</li>
</ul>
<hr />
<h2 id="4-디버깅-순서">4. 디버깅 순서</h2>
<ol>
<li>Console에 첫 오류가 있는지 확인한다.</li>
<li>오류의 파일명과 줄 번호를 클릭한다.</li>
<li>변수 값을 <code>console.log({ variable })</code>로 확인한다.</li>
<li><code>typeof</code>, <code>Array.isArray</code>, <code>Number.isNaN</code>으로 타입·특수값을 확인한다.</li>
<li>DOM 선택 결과가 <code>null</code>인지 확인한다.</li>
<li>Event listener가 실제로 등록되었는지 첫 줄에 log를 넣는다.</li>
<li>Network 탭에서 URL, status, response를 확인한다.</li>
<li>fetch에서는 <code>response.ok</code>, JSON parsing, 데이터 property 이름을 차례로 확인한다.</li>
<li>module import 경로와 <code>.js</code> 확장자를 확인한다.</li>
</ol>
<pre><code class="language-javascript">console.log({ value, type: typeof value });
console.table(arrayOfObjects);
console.error(error);</code></pre>
<hr />
<h2 id="5-미니-실습-문제">5. 미니 실습 문제</h2>
<h3 id="문제-1---배열-통계">문제 1 - 배열 통계</h3>
<p>다음 점수에서 60점 이상만 고르고 평균을 구하라.</p>
<pre><code class="language-javascript">const scores = [45, 70, 82, 59, 100];</code></pre>
<details>
정답

<pre><code class="language-javascript">const passed = scores.filter((score) =&gt; score &gt;= 60);
const average = passed.reduce((sum, score) =&gt; sum + score, 0) / passed.length;</code></pre>
</details>

<h3 id="문제-2---객체-배열-검색">문제 2 - 객체 배열 검색</h3>
<p><code>id</code>가 2인 사용자를 찾아 이름을 출력하라.</p>
<pre><code class="language-javascript">const users = [
  { id: 1, name: &quot;Kim&quot; },
  { id: 2, name: &quot;Lee&quot; },
];</code></pre>
<details>
정답

<pre><code class="language-javascript">const user = users.find((item) =&gt; item.id === 2);
console.log(user?.name ?? &quot;사용자 없음&quot;);</code></pre>
</details>

<h3 id="문제-3---toggle-button">문제 3 - Toggle button</h3>
<p>버튼을 클릭할 때 <code>active</code> class와 <code>aria-pressed</code> 값을 함께 바꿔라.</p>
<details>
정답

<pre><code class="language-javascript">const button = document.querySelector(&quot;#toggle-button&quot;);

button.addEventListener(&quot;click&quot;, () =&gt; {
  const active = button.classList.toggle(&quot;active&quot;);
  button.setAttribute(&quot;aria-pressed&quot;, String(active));
});</code></pre>
</details>

<h3 id="문제-4---form-검증">문제 4 - Form 검증</h3>
<p>form 제출 시 새로고침을 막고, 입력값이 비어 있으면 오류를 표시하라.</p>
<details>
정답

<pre><code class="language-javascript">form.addEventListener(&quot;submit&quot;, (event) =&gt; {
  event.preventDefault();

  const value = input.value.trim();

  if (!value) {
    errorBox.textContent = &quot;값을 입력하세요.&quot;;
    input.focus();
    return;
  }

  errorBox.textContent = &quot;&quot;;
});</code></pre>
</details>

<h3 id="문제-5---안전한-fetch">문제 5 - 안전한 fetch</h3>
<p><code>/api/profile</code>을 요청하고 오류·loading을 처리하라.</p>
<details>
정답

<pre><code class="language-javascript">async function loadProfile() {
  result.textContent = &quot;로딩 중...&quot;;

  try {
    const response = await fetch(&quot;/api/profile&quot;);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const profile = await response.json();
    result.textContent = profile.name;
  } catch (error) {
    console.error(error);
    result.textContent = &quot;프로필을 불러오지 못했습니다.&quot;;
  }
}</code></pre>
</details>

<hr />
<h2 id="6-셀프-테스트">6. 셀프 테스트</h2>
<h3 id="문제">문제</h3>
<ol>
<li>Java와 JavaScript의 관계는 무엇인가?</li>
<li><code>const</code>와 <code>let</code>의 선택 기준은 무엇인가?</li>
<li>block scope란 무엇인가?</li>
<li>TDZ란 무엇인가?</li>
<li>원시값과 객체의 <code>===</code> 비교 차이는 무엇인가?</li>
<li>falsy 값 중 다섯 개를 말해 보자.</li>
<li><code>undefined</code>와 <code>null</code>의 의도 차이는 무엇인가?</li>
<li><code>||</code>와 <code>??</code>는 어떤 값에서 다르게 동작하는가?</li>
<li>배열 값 순회에 <code>for...of</code>가 적합한 이유는 무엇인가?</li>
<li>parameter와 argument의 차이는 무엇인가?</li>
<li>함수에서 <code>return</code>을 만나면 어떤 일이 일어나는가?</li>
<li><code>map</code>, <code>filter</code>, <code>reduce</code>의 결과 차이는 무엇인가?</li>
<li>숫자 배열 정렬에 비교 함수가 필요한 이유는 무엇인가?</li>
<li>객체 method의 <code>this</code>는 무엇의 영향을 받는가?</li>
<li>JSON과 JavaScript 객체는 같은 것인가?</li>
<li><code>textContent</code>와 <code>innerHTML</code>의 차이는 무엇인가?</li>
<li><code>event.target</code>과 <code>event.currentTarget</code>의 차이는 무엇인가?</li>
<li>event bubbling은 어느 방향으로 진행되는가?</li>
<li><code>preventDefault()</code>와 <code>stopPropagation()</code>의 차이는 무엇인가?</li>
<li>Promise의 세 상태는 무엇인가?</li>
<li>async 함수는 무엇을 반환하는가?</li>
<li><code>await</code>는 무엇의 실행을 멈추는가?</li>
<li>Promise callback과 timer callback 중 현재 stack 이후 어느 쪽이 보통 먼저 실행되는가?</li>
<li>fetch 후 <code>response.json()</code>을 별도로 await하는 이유는 무엇인가?</li>
<li>404 response에서 <code>catch</code>가 반드시 실행되는가?</li>
<li>Storage에 객체를 저장하려면 어떻게 해야 하는가?</li>
<li>named export와 default export의 import 문법 차이는 무엇인가?</li>
<li>browser module을 실행할 때 Live Server가 유용한 이유는 무엇인가?</li>
</ol>
<h3 id="정답">정답</h3>
<ol>
<li>이름만 비슷한 별개의 언어이다.</li>
<li>기본은 <code>const</code>, 재할당이 필요할 때 <code>let</code>을 쓴다.</li>
<li><code>{}</code> 내부에 선언한 <code>let</code>·<code>const</code>를 바깥에서 접근할 수 없는 범위이다.</li>
<li>scope 시작부터 <code>let</code>·<code>const</code> 선언문까지 접근할 수 없는 구간이다.</li>
<li>원시값은 값이 같은지, 객체는 같은 객체를 가리키는지 비교한다.</li>
<li><code>false</code>, <code>0</code>, <code>&quot;&quot;</code>, <code>null</code>, <code>undefined</code>, <code>NaN</code>, <code>0n</code> 등이다.</li>
<li>undefined는 미할당·부재, null은 의도적으로 비어 있음을 주로 표현한다.</li>
<li><code>0</code>, <code>&quot;&quot;</code>, <code>false</code> 같은 null이 아닌 falsy 값에서 다르게 동작한다.</li>
<li>index가 아닌 element 값을 직접 제공하기 때문이다.</li>
<li>parameter는 함수 정의의 이름, argument는 호출 시 전달하는 실제 값이다.</li>
<li>값을 반환하고 그 함수의 실행이 즉시 끝난다.</li>
<li>변환한 배열, 조건에 맞는 배열, 하나로 누적한 결과이다.</li>
<li>기본 sort는 문자열 기준으로 비교하기 때문이다.</li>
<li>함수의 호출 방식에 영향을 받는다.</li>
<li>아니다. JSON은 제한된 문법을 가진 문자열 데이터 형식이다.</li>
<li>textContent는 일반 텍스트, innerHTML은 문자열을 HTML로 parsing한다.</li>
<li>실제 시작 요소와 현재 listener가 실행 중인 요소이다.</li>
<li>target에서 조상 방향으로 올라간다.</li>
<li>브라우저 기본 행동 취소와 event 전파 중단이다.</li>
<li>pending, fulfilled, rejected이다.</li>
<li>항상 Promise를 반환한다.</li>
<li>해당 async 함수의 나머지 실행을 일시 중단한다.</li>
<li>microtask queue의 Promise callback이 먼저 실행되는 것이 일반적이다.</li>
<li>response body 읽기와 JSON parsing도 비동기 작업이기 때문이다.</li>
<li>아니다. <code>response.ok</code>를 직접 확인해야 한다.</li>
<li><code>JSON.stringify()</code>로 저장하고 <code>JSON.parse()</code>로 복원한다.</li>
<li>named는 <code>{ name }</code>, default는 중괄호 없이 원하는 이름으로 가져온다.</li>
<li>module의 HTTP loading, 상대 경로, CORS 관련 문제를 피하고 실제 웹 환경처럼 실행하기 위해서이다.</li>
</ol>
<hr />
<h2 id="최종-암기-문장">최종 암기 문장</h2>
<blockquote>
<p>JavaScript 프로그램은 <strong>값을 변수에 담고</strong>, <strong>조건과 반복으로 흐름을 만들며</strong>, <strong>함수로 작업을 분리하고</strong>, <strong>배열과 객체로 데이터를 구조화</strong>한다. 브라우저에서는 DOM과 Event로 화면을 제어하고, Promise·async/await와 fetch로 비동기 데이터를 처리하며, Module로 파일의 책임을 나눈다.</p>
</blockquote>