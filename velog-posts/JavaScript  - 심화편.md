<h2 id="1-dom이란">1. DOM이란?</h2>
<p><strong>한 문장 이해:</strong> DOM은 브라우저가 HTML 문서를 JavaScript로 다룰 수 있도록 만든 객체 트리이다.</p>
<p>HTML:</p>
<pre><code class="language-html">&lt;body&gt;
  &lt;h1&gt;여행 정보&lt;/h1&gt;
  &lt;p id=&quot;message&quot;&gt;도시를 선택하세요.&lt;/p&gt;
&lt;/body&gt;</code></pre>
<p>DOM의 단순화된 모습:</p>
<pre><code class="language-text">Document
└─ html
   └─ body
      ├─ h1
      │  └─ Text &quot;여행 정보&quot;
      └─ p#message
         └─ Text &quot;도시를 선택하세요.&quot;</code></pre>
<ul>
<li><code>document</code>: 현재 HTML 문서를 나타내는 객체</li>
<li>element node: <code>&lt;body&gt;</code>, <code>&lt;h1&gt;</code>, <code>&lt;p&gt;</code> 같은 요소</li>
<li>text node: 요소 안의 실제 문자열</li>
<li>JavaScript는 DOM API로 요소를 찾고, 읽고, 변경한다.</li>
</ul>
<h2 id="11-요소-선택">1.1 요소 선택</h2>
<pre><code class="language-javascript">const message = document.getElementById(&quot;message&quot;);
const firstCard = document.querySelector(&quot;.card&quot;);
const cards = document.querySelectorAll(&quot;.card&quot;);</code></pre>
<table>
<thead>
<tr>
<th>메서드</th>
<th>결과</th>
</tr>
</thead>
<tbody><tr>
<td><code>getElementById(&quot;id&quot;)</code></td>
<td>일치하는 요소 하나 또는 <code>null</code></td>
</tr>
<tr>
<td><code>querySelector(&quot;css&quot;)</code></td>
<td>CSS 선택자와 처음 일치하는 요소 또는 <code>null</code></td>
</tr>
<tr>
<td><code>querySelectorAll(&quot;css&quot;)</code></td>
<td>일치하는 모든 요소의 정적 NodeList</td>
</tr>
<tr>
<td><code>getElementsByClassName()</code></td>
<td>live HTMLCollection</td>
</tr>
<tr>
<td><code>getElementsByTagName()</code></td>
<td>live HTMLCollection</td>
</tr>
</tbody></table>
<p>현대 코드에서는 CSS 선택자를 그대로 사용할 수 있는 <code>querySelector</code>와 <code>querySelectorAll</code>이 편리하다.</p>
<pre><code class="language-javascript">const selected = document.querySelector(&quot;.weather-card strong&quot;);

if (selected) {
  console.log(selected.textContent);
}</code></pre>
<p>요소가 없을 수 있으므로 <code>null</code>을 고려한다.</p>
<h2 id="12-내용-변경">1.2 내용 변경</h2>
<pre><code class="language-javascript">message.textContent = &quot;서울의 현재 날씨입니다.&quot;;</code></pre>
<table>
<thead>
<tr>
<th>property</th>
<th>동작</th>
<th>사용 원칙</th>
</tr>
</thead>
<tbody><tr>
<td><code>textContent</code></td>
<td>내용을 일반 텍스트로 읽고 변경</td>
<td>일반 문자열 출력에 우선 사용</td>
</tr>
<tr>
<td><code>innerText</code></td>
<td>화면에 보이는 텍스트 중심</td>
<td>layout 계산 영향을 받을 수 있음</td>
</tr>
<tr>
<td><code>innerHTML</code></td>
<td>문자열을 HTML로 parsing</td>
<td>신뢰할 수 있는 고정 markup에 제한</td>
</tr>
</tbody></table>
<pre><code class="language-javascript">const userInput = '&lt;img src=x onerror=&quot;alert(1)&quot;&gt;';
message.textContent = userInput; // 문자열 그대로 표시
// message.innerHTML = userInput; // 신뢰할 수 없는 입력이면 XSS 위험</code></pre>
<h2 id="13-속성-class-style-변경">1.3 속성, class, style 변경</h2>
<pre><code class="language-javascript">const image = document.querySelector(&quot;img&quot;);

image.setAttribute(&quot;alt&quot;, &quot;서울의 야경&quot;);
console.log(image.getAttribute(&quot;src&quot;));

message.classList.add(&quot;success&quot;);
message.classList.remove(&quot;loading&quot;);
message.classList.toggle(&quot;is-open&quot;);
message.classList.contains(&quot;success&quot;);

message.style.color = &quot;royalblue&quot;;</code></pre>
<p>많은 스타일을 바꿀 때는 inline style을 하나씩 설정하기보다 class를 바꾸고 CSS에 표현을 맡긴다.</p>
<pre><code class="language-css">.message { color: #374151; }
.message.loading { color: #6b7280; }
.message.error { color: #dc2626; }</code></pre>
<h2 id="14-요소-생성·추가·삭제">1.4 요소 생성·추가·삭제</h2>
<pre><code class="language-javascript">const list = document.querySelector(&quot;#city-list&quot;);
const item = document.createElement(&quot;li&quot;);

item.textContent = &quot;서울&quot;;
item.dataset.cityId = &quot;seoul&quot;;
list.append(item);

// item.remove();</code></pre>
<table>
<thead>
<tr>
<th>메서드</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>createElement(tag)</code></td>
<td>새 element 생성</td>
</tr>
<tr>
<td><code>append()</code> / <code>prepend()</code></td>
<td>자식의 뒤/앞에 추가</td>
</tr>
<tr>
<td><code>before()</code> / <code>after()</code></td>
<td>요소 앞/뒤에 추가</td>
</tr>
<tr>
<td><code>replaceWith()</code></td>
<td>요소 교체</td>
</tr>
<tr>
<td><code>remove()</code></td>
<td>요소 삭제</td>
</tr>
</tbody></table>
<p><code>document.write()</code>는 페이지 로드 후 호출하면 기존 문서를 덮어쓸 수 있으므로 일반적인 DOM 변경에 사용하지 않는다.</p>
<hr />
<h2 id="2-event">2. Event</h2>
<p><strong>한 문장 이해:</strong> Event는 클릭, 입력, 로드처럼 브라우저에서 일어난 일을 알리는 신호이다.</p>
<h3 id="주요-event">주요 Event</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>event</th>
<th>발생 시점</th>
</tr>
</thead>
<tbody><tr>
<td>Mouse/Pointer</td>
<td><code>click</code>, <code>dblclick</code>, <code>pointerdown</code>, <code>pointerup</code></td>
<td>클릭·포인터 동작</td>
</tr>
<tr>
<td>Keyboard</td>
<td><code>keydown</code>, <code>keyup</code></td>
<td>키를 누르고 뗄 때</td>
</tr>
<tr>
<td>Form</td>
<td><code>input</code>, <code>change</code>, <code>submit</code></td>
<td>입력 즉시, 값 확정, form 제출</td>
</tr>
<tr>
<td>Focus</td>
<td><code>focus</code>, <code>blur</code></td>
<td>초점을 얻고 잃을 때</td>
</tr>
<tr>
<td>Document</td>
<td><code>DOMContentLoaded</code></td>
<td>HTML parsing이 끝났을 때</td>
</tr>
<tr>
<td>Window</td>
<td><code>load</code>, <code>resize</code>, <code>scroll</code></td>
<td>자원 로드, 크기·스크롤 변화</td>
</tr>
</tbody></table>
<p><code>input</code>은 입력할 때마다, <code>change</code>는 값 변경이 확정될 때 주로 발생한다.</p>
<h2 id="21-event-등록-방법">2.1 Event 등록 방법</h2>
<h3 id="inline-handler---지양">Inline handler - 지양</h3>
<pre><code class="language-html">&lt;button onclick=&quot;showMessage()&quot;&gt;확인&lt;/button&gt;</code></pre>
<p>HTML과 동작이 섞여 유지보수가 어렵다.</p>
<h3 id="dom-property---한-handler만-유지">DOM property - 한 handler만 유지</h3>
<pre><code class="language-javascript">button.onclick = showMessage;</code></pre>
<p>다시 대입하면 이전 handler를 덮어쓴다.</p>
<h3 id="addeventlistener---권장"><code>addEventListener()</code> - 권장</h3>
<pre><code class="language-javascript">button.addEventListener(&quot;click&quot;, showMessage);

function showMessage(event) {
  console.log(&quot;클릭&quot;, event.currentTarget);
}</code></pre>
<p>같은 요소와 event type에 여러 listener를 등록할 수 있고, 전파 단계와 제거를 제어할 수 있다.</p>
<blockquote>
<p><code>showMessage()</code>가 아니라 <code>showMessage</code>를 전달한다. 괄호를 붙이면 등록 시점에 즉시 호출한 결과를 전달하게 된다.</p>
</blockquote>
<h2 id="22-event-객체">2.2 Event 객체</h2>
<pre><code class="language-javascript">button.addEventListener(&quot;click&quot;, (event) =&gt; {
  console.log(event.type);
  console.log(event.target);
  console.log(event.currentTarget);
});</code></pre>
<table>
<thead>
<tr>
<th>property</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td><code>event.type</code></td>
<td>event 종류</td>
</tr>
<tr>
<td><code>event.target</code></td>
<td>event가 실제 시작된 가장 안쪽 요소</td>
</tr>
<tr>
<td><code>event.currentTarget</code></td>
<td>현재 listener가 실행 중인 요소</td>
</tr>
<tr>
<td><code>event.key</code></td>
<td>keyboard event의 key 값</td>
</tr>
</tbody></table>
<h2 id="23-form-제출-처리">2.3 Form 제출 처리</h2>
<pre><code class="language-html">&lt;form id=&quot;signup-form&quot;&gt;
  &lt;label for=&quot;name&quot;&gt;이름&lt;/label&gt;
  &lt;input id=&quot;name&quot; name=&quot;name&quot; required&gt;
  &lt;button type=&quot;submit&quot;&gt;가입&lt;/button&gt;
&lt;/form&gt;</code></pre>
<pre><code class="language-javascript">const form = document.querySelector(&quot;#signup-form&quot;);

form.addEventListener(&quot;submit&quot;, (event) =&gt; {
  event.preventDefault();

  const formData = new FormData(form);
  console.log(formData.get(&quot;name&quot;));
});</code></pre>
<p>submit button의 click보다 form의 <code>submit</code> event를 처리하면 키보드 Enter 제출도 함께 처리할 수 있다.</p>
<h2 id="24-event-propagation">2.4 Event propagation</h2>
<p>요소가 중첩되어 있을 때 event는 세 단계를 거친다.</p>
<pre><code class="language-text">Capturing: window → document → 부모 → target
Target:    event가 발생한 요소
Bubbling:  target → 부모 → document → window</code></pre>
<p>기본 listener는 주로 bubbling 단계에서 실행된다.</p>
<pre><code class="language-javascript">parent.addEventListener(&quot;click&quot;, handleParent); // bubbling
parent.addEventListener(&quot;click&quot;, handleCapture, { capture: true });</code></pre>
<h3 id="전파와-기본-동작-제어">전파와 기본 동작 제어</h3>
<table>
<thead>
<tr>
<th>메서드</th>
<th>효과</th>
</tr>
</thead>
<tbody><tr>
<td><code>event.preventDefault()</code></td>
<td>링크 이동, form 제출 같은 브라우저 기본 동작 취소</td>
</tr>
<tr>
<td><code>event.stopPropagation()</code></td>
<td>다음 조상으로의 event 전파 중단</td>
</tr>
<tr>
<td><code>event.stopImmediatePropagation()</code></td>
<td>전파와 현재 요소의 뒤 listener 실행까지 중단</td>
</tr>
</tbody></table>
<p>전파 중단은 다른 기능까지 막을 수 있으므로 꼭 필요한 경우에만 사용한다.</p>
<h2 id="25-event-listener-제거">2.5 Event listener 제거</h2>
<p>등록할 때와 같은 함수 참조가 필요하다.</p>
<pre><code class="language-javascript">function handleClick() {
  console.log(&quot;clicked&quot;);
}

button.addEventListener(&quot;click&quot;, handleClick);
button.removeEventListener(&quot;click&quot;, handleClick);</code></pre>
<p>다음 코드는 서로 다른 함수 객체이므로 제거되지 않는다.</p>
<pre><code class="language-javascript">button.addEventListener(&quot;click&quot;, () =&gt; console.log(&quot;click&quot;));
button.removeEventListener(&quot;click&quot;, () =&gt; console.log(&quot;click&quot;));</code></pre>
<p>한 번만 실행하려면 option을 사용할 수 있다.</p>
<pre><code class="language-javascript">button.addEventListener(&quot;click&quot;, handleClick, { once: true });</code></pre>
<h2 id="26-event-delegation">2.6 Event delegation</h2>
<p>부모 하나에 listener를 등록하고 bubbling을 이용해 여러 자식을 처리하는 패턴이다.</p>
<pre><code class="language-html">&lt;ul id=&quot;todo-list&quot;&gt;
  &lt;li&gt;&lt;button data-action=&quot;remove&quot;&gt;HTML 삭제&lt;/button&gt;&lt;/li&gt;
  &lt;li&gt;&lt;button data-action=&quot;remove&quot;&gt;CSS 삭제&lt;/button&gt;&lt;/li&gt;
&lt;/ul&gt;</code></pre>
<pre><code class="language-javascript">const todoList = document.querySelector(&quot;#todo-list&quot;);

todoList.addEventListener(&quot;click&quot;, (event) =&gt; {
  const button = event.target.closest('button[data-action=&quot;remove&quot;]');

  if (!button || !todoList.contains(button)) return;

  button.closest(&quot;li&quot;).remove();
});</code></pre>
<p>장점:</p>
<ul>
<li>자식마다 listener를 만들지 않아도 된다.</li>
<li>나중에 추가한 자식에도 동작한다.</li>
<li>동적인 목록·table 처리에 유용하다.</li>
</ul>
<hr />
<h2 id="3-비동기-javascript">3. 비동기 JavaScript</h2>
<h3 id="동기와-비동기">동기와 비동기</h3>
<pre><code class="language-text">동기:  작업 A 완료 → 작업 B → 작업 C
비동기: 오래 걸리는 A를 요청 → B와 C 실행 → A 완료 결과 처리</code></pre>
<p>JavaScript 코드는 한 번에 하나씩 실행하지만, timer·network 같은 작업은 브라우저 환경에 맡길 수 있다. 완료 후 callback이 queue에 들어가고 event loop가 call stack이 비었을 때 실행 기회를 준다.</p>
<h2 id="31-callback에서-promise까지">3.1 Callback에서 Promise까지</h2>
<table>
<thead>
<tr>
<th>방식</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>Callback</td>
<td>완료 후 실행할 함수를 전달. 중첩되면 흐름·오류 처리가 복잡</td>
</tr>
<tr>
<td>Promise</td>
<td>비동기 결과를 상태가 있는 객체로 표현하고 <code>.then()</code>, <code>.catch()</code> 사용</td>
</tr>
<tr>
<td>async/await</td>
<td>Promise 기반 코드를 동기 코드처럼 읽기 쉽게 표현</td>
</tr>
</tbody></table>
<h2 id="32-promise-상태">3.2 Promise 상태</h2>
<table>
<thead>
<tr>
<th>상태</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td>pending</td>
<td>아직 완료·실패하지 않음</td>
</tr>
<tr>
<td>fulfilled</td>
<td>성공하여 결과값이 생김</td>
</tr>
<tr>
<td>rejected</td>
<td>실패하여 이유가 생김</td>
</tr>
</tbody></table>
<p>settled는 fulfilled 또는 rejected가 되어 더 이상 상태가 바뀌지 않는 상태를 뜻한다.</p>
<h2 id="33-promise-생성">3.3 Promise 생성</h2>
<pre><code class="language-javascript">function wait(milliseconds) {
  return new Promise((resolve, reject) =&gt; {
    if (milliseconds &lt; 0) {
      reject(new Error(&quot;시간은 0 이상이어야 합니다.&quot;));
      return;
    }

    setTimeout(() =&gt; resolve(milliseconds), milliseconds);
  });
}</code></pre>
<ul>
<li>executor <code>(resolve, reject) =&gt; {}</code>는 Promise 생성 시 즉시 실행된다.</li>
<li><code>resolve(value)</code>는 성공 결과를, <code>reject(error)</code>는 실패 이유를 정한다.</li>
<li>한 번 settled되면 뒤의 resolve/reject 호출은 상태를 바꾸지 못한다.</li>
<li><code>resolve()</code>나 <code>reject()</code> 자체가 현재 함수 실행을 중단하지는 않는다. 실행을 끝내려면 <code>return</code>을 사용한다.</li>
</ul>
<h2 id="34-promise-소비">3.4 Promise 소비</h2>
<pre><code class="language-javascript">wait(1000)
  .then((milliseconds) =&gt; {
    console.log(`${milliseconds}ms 완료`);
  })
  .catch((error) =&gt; {
    console.error(error);
  })
  .finally(() =&gt; {
    console.log(&quot;성공·실패와 관계없이 실행&quot;);
  });</code></pre>
<table>
<thead>
<tr>
<th>메서드</th>
<th>실행 조건</th>
</tr>
</thead>
<tbody><tr>
<td><code>.then(onFulfilled)</code></td>
<td>성공</td>
</tr>
<tr>
<td><code>.catch(onRejected)</code></td>
<td>실패</td>
</tr>
<tr>
<td><code>.finally(onFinally)</code></td>
<td>성공·실패 모두</td>
</tr>
</tbody></table>
<h2 id="35-promise-chaining">3.5 Promise chaining</h2>
<p><code>.then()</code>이 반환한 값은 다음 <code>.then()</code>으로 전달된다. Promise를 반환하면 다음 단계가 그 Promise를 기다린다.</p>
<pre><code class="language-javascript">step1()
  .then((value) =&gt; step2(value))
  .then((value) =&gt; step3(value))
  .then((result) =&gt; console.log(result))
  .catch((error) =&gt; console.error(error));</code></pre>
<p>중괄호를 썼다면 <code>return</code>을 잊지 않는다.</p>
<pre><code class="language-javascript">step1().then((value) =&gt; {
  return step2(value);
});</code></pre>
<h2 id="36-async와-await">3.6 <code>async</code>와 <code>await</code></h2>
<pre><code class="language-javascript">async function runSteps() {
  try {
    const first = await step1();
    const second = await step2(first);
    const result = await step3(second);
    return result;
  } catch (error) {
    console.error(error);
    throw error;
  }
}</code></pre>
<ul>
<li><code>async</code> 함수는 항상 Promise를 반환한다.</li>
<li>일반 값을 return하면 fulfilled Promise로 감싸진다.</li>
<li>throw하면 rejected Promise가 된다.</li>
<li><code>await</code>는 Promise가 settled될 때까지 <strong>해당 async 함수의 나머지 실행만</strong> 일시 중단한다.</li>
<li>다른 JavaScript 전체가 멈추는 것은 아니다.</li>
<li><code>await</code>는 async 함수 내부 또는 module의 top level에서 사용할 수 있다.</li>
</ul>
<h3 id="순차와-병렬">순차와 병렬</h3>
<p>서로 의존하는 작업은 순차 실행한다.</p>
<pre><code class="language-javascript">const user = await getUser();
const posts = await getPosts(user.id);</code></pre>
<p>서로 독립적인 작업은 동시에 시작할 수 있다.</p>
<pre><code class="language-javascript">const [weather, airQuality] = await Promise.all([
  getWeather(),
  getAirQuality(),
]);</code></pre>
<p>독립 작업을 불필요하게 하나씩 await하면 전체 시간이 길어질 수 있다.</p>
<h2 id="37-event-loop-핵심-흐름">3.7 Event loop 핵심 흐름</h2>
<pre><code class="language-javascript">console.log(&quot;A&quot;);

setTimeout(() =&gt; console.log(&quot;B&quot;), 0);

Promise.resolve().then(() =&gt; console.log(&quot;C&quot;));

console.log(&quot;D&quot;);</code></pre>
<p>예상 출력:</p>
<pre><code class="language-text">A
D
C
B</code></pre>
<ol>
<li>동기 코드 <code>A</code>, <code>D</code>가 call stack에서 먼저 실행된다.</li>
<li>Promise callback은 microtask queue에 들어간다.</li>
<li>timer callback은 task queue에 들어간다.</li>
<li>현재 stack이 비면 microtask를 먼저 비운 뒤 다음 task를 실행한다.</li>
</ol>
<p><code>setTimeout(..., 0)</code>은 즉시 실행이 아니라 최소 지연 후 queue에서 순서를 기다린다는 뜻이다.</p>
<hr />
<h2 id="4-browser-api">4. Browser API</h2>
<p>JavaScript 언어 자체와 브라우저가 제공하는 API는 구분해야 한다.</p>
<table>
<thead>
<tr>
<th>Browser API</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>DOM API</td>
<td>HTML 구조·내용·event 조작</td>
</tr>
<tr>
<td>Timer API</td>
<td>지연·반복 실행 예약</td>
</tr>
<tr>
<td>Web Storage API</td>
<td>브라우저에 key-value 저장</td>
</tr>
<tr>
<td>Fetch API</td>
<td>HTTP 요청</td>
</tr>
</tbody></table>
<h2 id="41-timer-api">4.1 Timer API</h2>
<pre><code class="language-javascript">const timeoutId = setTimeout(() =&gt; {
  console.log(&quot;1초 후 한 번 실행&quot;);
}, 1000);

clearTimeout(timeoutId);</code></pre>
<pre><code class="language-javascript">const intervalId = setInterval(() =&gt; {
  console.log(new Date().toLocaleTimeString());
}, 1000);

clearInterval(intervalId);</code></pre>
<ul>
<li>delay 단위는 millisecond이다.</li>
<li>실제 실행 시점은 지정 시간보다 늦을 수 있다.</li>
<li>component 제거·페이지 전환 등 더 이상 필요 없을 때 timer를 해제한다.</li>
<li>일정 작업이 실행 시간보다 오래 걸릴 수 있다면 <code>setInterval</code> 대신 재귀적 <code>setTimeout</code>도 고려한다.</li>
</ul>
<h2 id="42-web-storage">4.2 Web Storage</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>localStorage</th>
<th>sessionStorage</th>
</tr>
</thead>
<tbody><tr>
<td>유지</td>
<td>명시적으로 삭제할 때까지</td>
<td>해당 page session·tab이 끝날 때까지</td>
</tr>
<tr>
<td>공유</td>
<td>같은 origin의 tab·window에서 접근 가능</td>
<td>생성된 tab의 page session에 한정</td>
</tr>
<tr>
<td>저장 형식</td>
<td>문자열</td>
<td>문자열</td>
</tr>
</tbody></table>
<pre><code class="language-javascript">localStorage.setItem(&quot;theme&quot;, &quot;dark&quot;);
const theme = localStorage.getItem(&quot;theme&quot;);
localStorage.removeItem(&quot;theme&quot;);
// localStorage.clear();</code></pre>
<p>객체는 JSON으로 변환한다.</p>
<pre><code class="language-javascript">const settings = { theme: &quot;dark&quot;, fontSize: 18 };

localStorage.setItem(&quot;settings&quot;, JSON.stringify(settings));

const savedText = localStorage.getItem(&quot;settings&quot;);
const savedSettings = savedText ? JSON.parse(savedText) : null;</code></pre>
<p>주의:</p>
<ul>
<li>Storage 값은 문자열이다.</li>
<li>같은 origin에서 동작한다.</li>
<li>동기 API이므로 매우 큰 데이터를 반복 저장하지 않는다.</li>
<li>사용자가 내용을 볼 수 있으며 XSS가 발생하면 접근될 수 있다.</li>
<li>비밀번호와 민감한 개인정보를 저장하지 않는다.</li>
<li>인증 token 저장 위치는 보안 설계에 따라 결정해야 하며, 단순히 localStorage가 적합하다고 가정하면 안 된다.</li>
</ul>
<h2 id="43-fetch-api">4.3 Fetch API</h2>
<pre><code class="language-javascript">async function loadData() {
  const response = await fetch(&quot;data.json&quot;);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  const data = await response.json();
  return data;
}</code></pre>
<p>중요한 두 단계:</p>
<ol>
<li><code>await fetch()</code> → HTTP response header를 받은 <code>Response</code></li>
<li><code>await response.json()</code> → response body를 읽고 JSON parsing</li>
</ol>
<blockquote>
<p><code>fetch</code>는 404·500 같은 HTTP 오류에서 자동으로 reject되지 않는다. network 자체가 실패하거나 요청이 중단되는 경우 등에 reject되므로 <code>response.ok</code>를 직접 확인한다.</p>
</blockquote>
<h3 id="오류와-loading-처리">오류와 loading 처리</h3>
<pre><code class="language-javascript">const resultBox = document.querySelector(&quot;#result&quot;);

async function renderData() {
  resultBox.textContent = &quot;로딩 중...&quot;;
  resultBox.className = &quot;loading&quot;;

  try {
    const data = await loadData();
    resultBox.textContent = data.message;
    resultBox.className = &quot;success&quot;;
  } catch (error) {
    console.error(error);
    resultBox.textContent = &quot;데이터를 불러오지 못했습니다.&quot;;
    resultBox.className = &quot;error&quot;;
  }
}</code></pre>
<h3 id="post-예시">POST 예시</h3>
<pre><code class="language-javascript">async function createUser(user) {
  const response = await fetch(&quot;/api/users&quot;, {
    method: &quot;POST&quot;,
    headers: {
      &quot;Content-Type&quot;: &quot;application/json&quot;,
    },
    body: JSON.stringify(user),
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}</code></pre>
<hr />
<h2 id="5-javascript-module">5. JavaScript Module</h2>
<p>Module은 코드 파일마다 독립된 scope를 제공하고 필요한 값만 import/export하도록 한다.</p>
<h3 id="named-exportimport">Named export/import</h3>
<pre><code class="language-javascript">// math.js
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}</code></pre>
<pre><code class="language-javascript">// app.js
import { PI, add } from &quot;./math.js&quot;;

console.log(add(PI, 2));</code></pre>
<p>이름을 바꾸어 가져올 수 있다.</p>
<pre><code class="language-javascript">import { add as sum } from &quot;./math.js&quot;;</code></pre>
<h3 id="default-exportimport">Default export/import</h3>
<pre><code class="language-javascript">// logger.js
export default function log(message) {
  console.log(message);
}</code></pre>
<pre><code class="language-javascript">// app.js
import writeLog from &quot;./logger.js&quot;;</code></pre>
<table>
<thead>
<tr>
<th>방식</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>named export</td>
<td>파일에서 여러 개 가능, <code>{ 정확한이름 }</code>으로 import</td>
</tr>
<tr>
<td>default export</td>
<td>파일에서 하나, import하는 쪽이 이름 지정</td>
</tr>
</tbody></table>
<p>프로젝트에서는 일관된 기준을 정한다. 자동 완성과 안전한 이름 변경 때문에 named export를 선호하는 팀도 많다.</p>
<h3 id="html에서-module-실행">HTML에서 module 실행</h3>
<pre><code class="language-html">&lt;script type=&quot;module&quot; src=&quot;script/app.js&quot;&gt;&lt;/script&gt;</code></pre>
<p>Module 특징:</p>
<ul>
<li>파일별 scope가 분리된다.</li>
<li>자동으로 strict mode가 적용된다.</li>
<li>기본적으로 defer처럼 HTML parsing 후 실행된다.</li>
<li>같은 module은 URL 기준으로 한 번 평가되고 재사용된다.</li>
<li>import 경로는 브라우저에서 일반적으로 <code>./math.js</code>처럼 정확한 상대 경로와 확장자를 쓴다.</li>
<li><code>file://</code>로 직접 열면 CORS·module loading 문제를 만날 수 있으므로 Live Server 같은 HTTP server로 실행한다.</li>
</ul>
<hr />
<h2 id="6-종합-실습-실시간-날씨">6. 종합 실습: 실시간 날씨</h2>
<p>원본 과제의 핵심은 다음 세 책임을 나누는 것이다.</p>
<pre><code class="language-text">HTML: 사용자 선택과 결과 영역
realtimeInfo.js: DOM과 event, 화면 상태
weatherAPI.js: 서버 요청과 데이터 반환</code></pre>
<h2 id="61-html">6.1 HTML</h2>
<pre><code class="language-html">&lt;label for=&quot;city-select&quot;&gt;도시&lt;/label&gt;
&lt;select id=&quot;city-select&quot;&gt;
  &lt;option value=&quot;&quot;&gt;도시를 선택하세요&lt;/option&gt;
  &lt;option value=&quot;37.5665,126.9780&quot;&gt;서울&lt;/option&gt;
  &lt;option value=&quot;35.1796,129.0756&quot;&gt;부산&lt;/option&gt;
&lt;/select&gt;

&lt;section id=&quot;weather-box&quot; aria-live=&quot;polite&quot;&gt;
  도시를 선택하세요.
&lt;/section&gt;

&lt;script type=&quot;module&quot; src=&quot;script/realtimeInfo.js&quot;&gt;&lt;/script&gt;</code></pre>
<p><code>aria-live=&quot;polite&quot;</code>는 비동기로 바뀐 결과를 보조 기술이 적절한 시점에 안내하도록 돕는다.</p>
<h2 id="62-api-모듈">6.2 API 모듈</h2>
<pre><code class="language-javascript">// script/weatherAPI.js
export async function fetchCurrentWeather(latitude, longitude) {
  const parameters = new URLSearchParams({
    latitude,
    longitude,
    current: &quot;temperature_2m,relative_humidity_2m&quot;,
  });

  const response = await fetch(
    `https://api.open-meteo.com/v1/forecast?${parameters}`,
  );

  if (!response.ok) {
    throw new Error(`날씨 요청 실패: HTTP ${response.status}`);
  }

  const data = await response.json();

  if (!data.current) {
    throw new Error(&quot;현재 날씨 데이터가 없습니다.&quot;);
  }

  return data.current;
}</code></pre>
<h2 id="63-화면-모듈">6.3 화면 모듈</h2>
<pre><code class="language-javascript">// script/realtimeInfo.js
import { fetchCurrentWeather } from &quot;./weatherAPI.js&quot;;

const citySelect = document.querySelector(&quot;#city-select&quot;);
const weatherBox = document.querySelector(&quot;#weather-box&quot;);

citySelect.addEventListener(&quot;change&quot;, handleCityChange);

async function handleCityChange(event) {
  const selectedOption = event.currentTarget.selectedOptions[0];
  const coordinates = event.currentTarget.value;

  if (!coordinates) {
    weatherBox.textContent = &quot;도시를 선택하세요.&quot;;
    return;
  }

  const [latitude, longitude] = coordinates.split(&quot;,&quot;);
  weatherBox.textContent = &quot;로딩 중... ⏳&quot;;

  try {
    const current = await fetchCurrentWeather(latitude, longitude);

    weatherBox.replaceChildren(
      createWeatherResult(selectedOption.textContent, current),
    );
  } catch (error) {
    console.error(error);
    weatherBox.textContent = &quot;날씨 정보를 가져오지 못했습니다.&quot;;
  }
}

function createWeatherResult(cityName, current) {
  const container = document.createElement(&quot;div&quot;);
  const heading = document.createElement(&quot;h2&quot;);
  const temperature = document.createElement(&quot;p&quot;);
  const humidity = document.createElement(&quot;p&quot;);

  heading.textContent = `${cityName} 현재 날씨`;
  temperature.textContent = `온도: ${current.temperature_2m}℃`;
  humidity.textContent = `습도: ${current.relative_humidity_2m}%`;

  container.append(heading, temperature, humidity);
  return container;
}</code></pre>
<h3 id="실행-흐름">실행 흐름</h3>
<pre><code class="language-text">사용자가 도시 변경
  → change event 발생
  → 좌표 분리
  → loading 표시
  → weatherAPI의 fetch 함수 await
  → 성공: DOM에 온도·습도 표시
  → 실패: 사용자용 오류 메시지 + Console에 원인 기록</code></pre>
<h3 id="실습-점검">실습 점검</h3>
<ul>
<li>도시를 선택하지 않은 상태를 처리했는가?</li>
<li>요청 중 loading을 표시하는가?</li>
<li><code>response.ok</code>를 검사하는가?</li>
<li><code>try...catch</code>로 오류를 처리하는가?</li>
<li>API 코드와 DOM 코드를 다른 module로 분리했는가?</li>
<li>서버 데이터를 바로 <code>innerHTML</code>에 넣지 않았는가?</li>
</ul>
<hr />
<h2 id="7-심화편-핵심-정리">7. 심화편 핵심 정리</h2>
<ul>
<li>DOM은 HTML의 객체 트리이며 JavaScript가 화면을 읽고 변경하는 통로이다.</li>
<li>일반 텍스트는 <code>textContent</code>, 구조 생성은 <code>createElement</code>를 우선 고려한다.</li>
<li>Event는 <code>addEventListener()</code>로 등록하며 callback <strong>참조</strong>를 전달한다.</li>
<li>bubbling을 이용한 event delegation은 동적인 목록 처리에 유용하다.</li>
<li>Promise는 비동기 결과의 상태와 값을 나타내며 async/await는 그 위의 읽기 쉬운 문법이다.</li>
<li><code>await</code>는 전체 프로그램이 아니라 현재 async 함수의 진행을 일시 중단한다.</li>
<li>fetch에서는 network 오류뿐 아니라 <code>response.ok</code>를 통한 HTTP 오류 검사도 필요하다.</li>
<li>Storage에는 문자열만 저장되며 민감한 정보를 무심코 저장하면 안 된다.</li>
<li>Module은 scope를 격리하고 import/export로 파일 사이 의존성을 명확하게 만든다.</li>
</ul>