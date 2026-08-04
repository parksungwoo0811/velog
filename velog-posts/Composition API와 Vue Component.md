<p> Vue.js에서 가장 중요한 두 축인 <strong>Composition API</strong>와 <strong>Component</strong>를 다룹니다.</p>
<blockquote>
<p>Composition API는 &quot;컴포넌트 내부 로직을 어떻게 작성할 것인가&quot;에 대한 답이고,
Component는 &quot;화면을 어떻게 조립할 것인가&quot;에 대한 답입니다.</p>
</blockquote>
<p>이 둘을 제대로 이해하면 Vue.js로 실무 수준의 화면을 자유롭게 만들 수 있습니다. 하나씩 차근차근 따라가 봅시다.</p>
<hr />
<h1 id="part-1-composition-api">Part 1. Composition API</h1>
<h2 id="1-1-composition-api가-뭔가요">1-1. Composition API가 뭔가요?</h2>
<p>Vue 2 시절에는 <strong>Options API</strong>라는 방식을 썼습니다. <code>data</code>, <code>computed</code>, <code>methods</code>처럼 <strong>기능의 종류별로</strong> 코드를 나눠 적는 방식이죠.</p>
<p>문제는 하나의 기능(예: &quot;검색&quot; 기능)을 구현하려면 데이터는 <code>data</code>에, 계산 로직은 <code>computed</code>에, 실행 함수는 <code>methods</code>에 나눠 적어야 해서, 코드가 커질수록 <strong>관련된 로직이 파일 여기저기 흩어지는</strong> 문제가 있었습니다.</p>
<p><strong>Composition API</strong>는 이 문제를 해결합니다. &quot;검색&quot;이라는 하나의 기능과 관련된 데이터, 계산, 함수를 <strong>한곳에 모아서(compose)</strong> 작성할 수 있게 해줍니다.</p>
<blockquote>
<p>💡 비유하자면, Options API는 &quot;서랍장 종류별로(양말 서랍, 티셔츠 서랍) 정리&quot;하는 방식이고, Composition API는 &quot;여행 가방 하나에 여행별로(제주도 여행 가방, 출장 가방) 정리&quot;하는 방식입니다.</p>
</blockquote>
<p>작성 방법은 간단합니다. Vue 3.2부터는 <code>&lt;script setup&gt;</code> 태그 안에 그냥 로직을 나열하면 됩니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
// 이 안에 데이터, 함수, 계산된 값을 자유롭게 작성
&lt;/script&gt;</code></pre>
<h2 id="1-2-vue-3가-제공하는-핵심-내장-함수들">1-2. Vue 3가 제공하는 핵심 내장 함수들</h2>
<p>Composition API는 다양한 &quot;내장 함수(Built-in Function)&quot;들의 조합으로 이루어집니다. 전체 그림을 먼저 보고 가면 이해가 훨씬 쉽습니다.</p>
<table>
<thead>
<tr>
<th>카테고리</th>
<th>주요 함수</th>
</tr>
</thead>
<tbody><tr>
<td>애플리케이션</td>
<td><code>createApp</code>, <code>app.config</code> 등</td>
</tr>
<tr>
<td><strong>반응형 상태 관리</strong></td>
<td><code>ref</code>, <code>reactive</code>, <code>toRef</code>, <code>toRefs</code> 등</td>
</tr>
<tr>
<td><strong>계산 및 감시</strong></td>
<td><code>computed</code>, <code>watch</code>, <code>watchEffect</code></td>
</tr>
<tr>
<td><strong>라이프사이클 훅</strong></td>
<td><code>onMounted</code>, <code>onUpdated</code>, <code>onUnmounted</code> 등</td>
</tr>
<tr>
<td>컴포넌트 구성/메타</td>
<td><code>defineProps</code>, <code>defineEmits</code> 등</td>
</tr>
<tr>
<td>렌더링 제어</td>
<td><code>h</code>, <code>nextTick</code> 등</td>
</tr>
<tr>
<td>의존성 주입</td>
<td><code>provide</code>, <code>inject</code></td>
</tr>
</tbody></table>
<p>오늘은 이 중에서 굵게 표시한 <strong>반응형 상태 관리</strong>, <strong>계산 및 감시</strong>, 그리고 뒤에서 배울 <strong>라이프사이클 훅</strong>, <strong>컴포넌트 구성</strong> 함수들을 집중적으로 다룹니다.</p>
<h2 id="1-3-반응형-상태-관리---ref">1-3. 반응형 상태 관리 - <code>ref()</code></h2>
<p>Vue의 가장 핵심적인 개념은 <strong>반응형(Reactive)</strong> 입니다. 변수 값이 바뀌면 화면이 자동으로 따라 바뀌는 것을 말합니다. 일반 JavaScript 변수는 값이 바뀌어도 화면이 알아서 갱신되지 않지만, Vue의 <code>ref()</code>로 감싼 변수는 자동으로 화면이 갱신됩니다.</p>
<p><strong><code>ref()</code>의 특징</strong></p>
<ul>
<li>숫자, 문자열 같은 <strong>원시 타입</strong>은 물론 배열, 객체 같은 <strong>참조 타입</strong>도 모두 반응형으로 만들 수 있습니다.</li>
<li><code>&lt;script setup&gt;</code> 코드 안에서는 <code>.value</code>를 붙여서 접근해야 합니다.</li>
<li>반면 <code>&lt;template&gt;</code> 안에서는 <code>.value</code> 없이 바로 사용합니다. (Vue가 자동으로 처리해줌)</li>
</ul>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const count = ref(0)
const name = ref('홍길동')
const items = ref(['사과', '배'])
const user = ref({ name: '이순신', age: 30 })

const increaseRef = () =&gt; {
  count.value++ // script 안에서는 .value 필수
}
const changeUserName = () =&gt; {
  user.value.name = '장보고'
}
&lt;/script&gt;

&lt;template&gt;
  &lt;p&gt;Ref 카운트: {{ count }}&lt;/p&gt;  &lt;!-- template 안에서는 .value 없이 --&gt;
  &lt;p&gt;이름: &lt;input v-model=&quot;name&quot; /&gt;{{ name }}&lt;/p&gt;
  &lt;button @click=&quot;increaseRef&quot;&gt;Ref 변수 증가&lt;/button&gt;
&lt;/template&gt;</code></pre>
<blockquote>
<p>⚠️ 초보자가 가장 많이 실수하는 부분이 바로 <strong><code>.value</code>를 언제 붙이고 언제 빼는지</strong>입니다.
규칙은 딱 하나: <strong><code>&lt;script&gt;</code> 안에서는 항상 <code>.value</code>, <code>&lt;template&gt;</code> 안에서는 항상 생략</strong>입니다.</p>
</blockquote>
<h2 id="1-4-반응형-상태-관리---reactive">1-4. 반응형 상태 관리 - <code>reactive()</code></h2>
<p><code>reactive()</code>는 <strong>객체나 배열 같은 참조 타입 전용</strong> 반응형 함수입니다. <code>ref()</code>와 달리 <code>.value</code> 없이 바로 속성에 접근할 수 있어 편리해 보이지만, 중요한 함정이 하나 있습니다.</p>
<p><strong>⚠️ reactive()의 반응성 단절 문제</strong></p>
<pre><code class="language-js">let state = reactive({ count: 0 })

// ❌ 통째로 새 객체를 갈아끼우면 반응성 연결이 끊어진다!
state = { count: 5 }

// 🟢 내부 속성만 조심스럽게 변경해야 정상 동작
state.count = 5</code></pre>
<p>객체를 통째로 교체하는 순간, Vue가 추적하던 반응형 연결이 끊어져 버립니다. 이 약점 때문에 실무에서는 <strong>객체나 배열도 그냥 <code>ref()</code>로 통일해서 쓰는 경우가 많습니다.</strong></p>
<blockquote>
<p>💡 <strong>정리하면</strong>: 처음 배우는 단계에서는 <code>reactive()</code>의 미묘한 함정을 신경 쓰기보다, <strong>&quot;모든 반응형 데이터는 일단 <code>ref()</code>로 만든다&quot;</strong>는 원칙을 가져가는 것이 안전합니다.</p>
</blockquote>
<h2 id="1-5-계산된-속성---computed">1-5. 계산된 속성 - <code>computed()</code></h2>
<p><code>computed()</code>는 <strong>다른 반응형 데이터를 가공해서 만드는 파생 데이터</strong>입니다. &quot;장바구니 상품 개수로 총 금액을 계산&quot;하는 것처럼, 원본 데이터에 의존해서 값이 자동으로 계산되는 경우에 사용합니다.</p>
<p><strong>가장 큰 장점은 캐싱(Caching)</strong> 입니다. 의존하는 데이터가 바뀌지 않으면 다시 계산하지 않고 이전 결과를 재사용해서 성능이 좋습니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref, computed } from 'vue'

const count = ref(0)
const dummy = ref(0) // computed와 무관한 변수

// 일반 함수: 화면이 리렌더링될 때마다 무조건 다시 실행됨
const getMethodResult = () =&gt; {
  console.log('❌ 일반 함수 실행됨!')
  return count.value * 2
}

// computed: count가 바뀔 때만 재연산 (dummy가 바뀔 땐 캐시된 값 재사용)
const doubleCount = computed(() =&gt; {
  console.log('✅ Computed 연산 실행됨!')
  return count.value * 2
})
&lt;/script&gt;</code></pre>
<p><code>dummy</code> 버튼을 눌러 화면이 리렌더링되면, 일반 함수 로그는 계속 찍히지만 <code>computed</code> 로그는 찍히지 않습니다. <strong>의존성과 무관한 변경에는 반응하지 않는다</strong>는 뜻이죠. 이 캐싱 효과가 <code>computed()</code>를 쓰는 핵심 이유입니다.</p>
<blockquote>
<p>참고로 <code>&lt;script setup&gt;</code> 안에서 함수를 정의할 때는 <code>function</code>, <code>함수 표현식</code>, <code>화살표 함수</code> 세 가지 방식이 있는데, 실무에서는 <strong>화살표 함수(<code>const 함수명 = () =&gt; {}</code>)</strong> 를 압도적으로 많이 사용합니다.</p>
</blockquote>
<h2 id="1-6-감시자---watch">1-6. 감시자 - <code>watch()</code></h2>
<p><code>watch()</code>는 특정 반응형 데이터가 <strong>변경되는 순간</strong>을 감지해서 후속 작업(API 호출, 로그 기록 등)을 실행하는 함수입니다. <code>computed()</code>가 &quot;값을 계산&quot;하는 것이 목적이라면, <code>watch()</code>는 &quot;변화가 생겼을 때 어떤 행동을 하는 것&quot;이 목적입니다.</p>
<pre><code class="language-js">watch(감시할데이터, (newVal, oldVal) =&gt; {
  // 변경 후 실행할 로직
})</code></pre>
<p>콜백 함수는 <strong>변경된 새 값(newVal)</strong> 과 <strong>변경 전 옛 값(oldVal)</strong> 을 인자로 받습니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref, watch } from 'vue'

const currentCity = ref('서울')
const logMessage = ref('아직 감시 시스템이 작동하지 않았습니다.')

watch(currentCity, (newValue, oldValue) =&gt; {
  logMessage.value = `📍 [${oldValue}]에서 [${newValue}]로 변경됨.`
  console.log(`🤖 ${newValue}의 날씨 API를 다시 조회합니다...`)
})
&lt;/script&gt;</code></pre>
<h3 id="watch의-다양한-활용-패턴"><code>watch()</code>의 다양한 활용 패턴</h3>
<p>실무에서는 단순히 값 하나만 감시하는 것 이상의 여러 패턴이 필요합니다.</p>
<p><strong>① Multi-Source Watch (여러 변수 동시 감시)</strong></p>
<pre><code class="language-js">watch([city, dateType], ([newCity, newDate], [oldCity, oldDate]) =&gt; {
  console.log(`${newCity}의 ${newDate} 날씨를 불러옵니다...`)
})</code></pre>
<p>배열 <code>[ ]</code>로 여러 데이터를 묶으면, 그중 <strong>하나라도 바뀌면</strong> 콜백이 실행됩니다.</p>
<p><strong>② Deep Watch (객체/배열 내부까지 감시)</strong></p>
<p><code>ref()</code>로 만든 객체는 기본적으로 <strong>주소값(참조값)</strong> 만 감시합니다. 즉, <code>user.value.age</code>처럼 내부 속성만 바뀌는 건 기본 설정으로는 감지되지 않습니다. 이럴 땐 <code>{ deep: true }</code> 옵션이 필요합니다.</p>
<pre><code class="language-js">watch(user, (newVal) =&gt; {
  console.log(`나이: ${newVal.age}`)
}, { deep: true })</code></pre>
<p>단, <code>deep: true</code>를 쓰면 <code>newVal</code>과 <code>oldVal</code>이 <strong>똑같은 값</strong>으로 나오는 함정이 있습니다 (주소값이 같기 때문). 만약 변경 전/후 값을 정확히 비교하고 싶다면 아래처럼 <strong>특정 속성만 콕 집어서</strong> 감시하는 방법을 씁니다.</p>
<pre><code class="language-js">// 화살표 함수로 특정 속성(age)만 지정 → 이전 값 추적 가능!
watch(() =&gt; user.value.age, (newAge, oldAge) =&gt; {
  console.log(`나이가 ${oldAge}세 → ${newAge}세로 변경됨!`)
})</code></pre>
<p><strong>③ reactive() 데이터 감시</strong></p>
<p><code>reactive()</code>로 만든 데이터는 애초에 <code>deep</code>이 자동으로 켜져 있는 것처럼 동작합니다. 하지만 마찬가지로 이전 값을 정확히 추적하려면 특정 속성만 지정하는 방식이 필요합니다.</p>
<h3 id="감시자-비교-표">감시자 비교 표</h3>
<table>
<thead>
<tr>
<th>상황</th>
<th>방법</th>
</tr>
</thead>
<tbody><tr>
<td>변수 하나 감시</td>
<td><code>watch(변수, callback)</code></td>
</tr>
<tr>
<td>여러 변수 동시 감시</td>
<td><code>watch([변수1, 변수2], callback)</code></td>
</tr>
<tr>
<td>객체/배열 내부까지 감시</td>
<td><code>watch(변수, callback, { deep: true })</code></td>
</tr>
<tr>
<td>특정 속성만 감시 (이전 값 보존)</td>
<td><code>watch(() =&gt; 변수.value.속성, callback)</code></td>
</tr>
</tbody></table>
<h2 id="1-7-자동-감시자---watcheffect">1-7. 자동 감시자 - <code>watchEffect()</code></h2>
<p><code>watchEffect()</code>는 <code>watch()</code>와 달리 <strong>&quot;무엇을 감시할지&quot; 지정하지 않아도</strong> 됩니다. 콜백 함수 내부에서 사용한 반응형 데이터를 Vue가 <strong>자동으로 인식</strong>해서 감시 목록에 등록합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref, watchEffect } from 'vue'

const username = ref('홍길동')
const age = ref(20)

watchEffect(() =&gt; {
  // 이 안에서 사용한 username, age를 Vue가 자동으로 감시함
  console.log(`이름: ${username.value} / 나이: ${age.value}세`)
})
&lt;/script&gt;</code></pre>
<p><strong><code>watch()</code>와의 결정적인 차이 두 가지</strong></p>
<ol>
<li><code>watchEffect()</code>는 감시 대상을 지정하지 않는다 (자동 추적).</li>
<li><code>watchEffect()</code>는 컴포넌트가 처음 생성될 때 <strong>무조건 1회 즉시 실행</strong>된다. (<code>watch()</code>는 값이 변경되기 전까지는 실행 안 됨)</li>
</ol>
<blockquote>
<p>💡 <strong>언제 무엇을 쓸까?</strong></p>
<ul>
<li>&quot;이전 값과 비교&quot;가 필요하다 → <code>watch()</code></li>
<li>&quot;그냥 관련된 값이 바뀌면 뭔가 실행하고 싶다&quot; → <code>watchEffect()</code></li>
</ul>
</blockquote>
<h2 id="1-8-composition-api-요약">1-8. Composition API 요약</h2>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>ref()</code></td>
<td>모든 타입을 반응형으로 (기본 원칙: 이걸로 통일)</td>
</tr>
<tr>
<td><code>reactive()</code></td>
<td>객체/배열 전용 반응형 (반응성 단절 주의)</td>
</tr>
<tr>
<td><code>computed()</code></td>
<td>캐싱되는 계산된 값 (파생 데이터)</td>
</tr>
<tr>
<td><code>watch()</code></td>
<td>특정 데이터 변경 감지 (이전/새 값 비교)</td>
</tr>
<tr>
<td><code>watchEffect()</code></td>
<td>사용된 데이터 자동 추적 + 즉시 1회 실행</td>
</tr>
</tbody></table>
<hr />
<h1 id="part-2-vue-component-컴포넌트-다루기">Part 2. Vue Component (컴포넌트 다루기)</h1>
<h2 id="2-1-component란-무엇인가">2-1. Component란 무엇인가?</h2>
<p>소프트웨어 공학에서 <strong>컴포넌트(Component)</strong>란 &quot;독립적으로 작동하고, 다른 부품으로 교체 가능하며, 다른 프로그램과 조립할 수 있는 표준화된 모듈&quot;을 뜻합니다. 핵심은 <strong>독립성</strong>과 <strong>교체 가능성</strong>입니다.</p>
<p><strong>Vue Component</strong>는 이 개념을 화면 단위에 그대로 적용한 것입니다. HTML, CSS, JavaScript를 하나의 <code>.vue</code> 파일 안에 뭉쳐놓은 <strong>재사용 가능한 화면 블록</strong>이라고 생각하면 됩니다 (이를 SFC, Single File Component라고 부릅니다).</p>
<blockquote>
<p>💡 레고 블록을 떠올려 보세요. 웹페이지 전체를 통째로 하나의 거대한 코드로 짜는 대신, &quot;검색창&quot;, &quot;날씨 카드&quot;, &quot;헤더&quot;처럼 독립된 블록을 각각 만들고 조립해서 하나의 화면을 완성하는 방식입니다.</p>
</blockquote>
<h2 id="2-2-component-계층-구조-hierarchy">2-2. Component 계층 구조 (Hierarchy)</h2>
<p>컴포넌트들은 서로 <strong>트리(Tree) 구조</strong>로 연결됩니다.</p>
<ul>
<li><strong>부모-자식(Parent-Child) 관계</strong>: 다른 컴포넌트를 품고 있으면 부모, 그 안에서 작동하면 자식. 부모와 자식은 철저히 독립적이라 서로의 내부 변수를 마음대로 들여다볼 수 없습니다.</li>
<li><strong>형제(Sibling) 관계</strong>: 같은 부모 아래 나란히 있는 컴포넌트들. 형제끼리는 직접 대화할 수 없고, <strong>반드시 부모를 거쳐서</strong> 데이터를 주고받아야 합니다.</li>
<li><strong>조상-후손(Ancestors-Descendants) 관계</strong>: 자식의 자식, 그 자식의 자식... 처럼 여러 계층으로 깊어진 구조.</li>
</ul>
<p>이 규칙 때문에 Vue에서는 &quot;데이터를 어떻게 전달할 것인가&quot;가 매우 중요한 주제가 됩니다. (뒤에서 배울 Props, Emits, Provide/Inject가 바로 이 문제를 해결하는 도구들입니다.)</p>
<h2 id="2-3-component-등록하기">2-3. Component 등록하기</h2>
<p>만든 컴포넌트를 사용하려면 먼저 &quot;등록&quot;을 해야 합니다. 방법은 두 가지입니다.</p>
<p><strong>① 지역(Local) 등록</strong> — 특정 컴포넌트에서만 사용하고 싶을 때</p>
<pre><code class="language-vue">&lt;script setup&gt;
import BaseButton from './components/BaseButton.vue'
&lt;/script&gt;

&lt;template&gt;
  &lt;BaseButton /&gt;
  &lt;base-button&gt;&lt;/base-button&gt; &lt;!-- kebab-case도 가능 --&gt;
&lt;/template&gt;</code></pre>
<p><strong>② 전역(Global) 등록</strong> — 앱 전체 어디서나 사용하고 싶을 때, <code>main.js</code>에서 등록</p>
<pre><code class="language-js">import { createApp } from 'vue'
import App from './App.vue'
import BaseButton from './components/BaseButton.vue'

const app = createApp(App)
app.component('BaseButton', BaseButton)
app.mount('#app')</code></pre>
<blockquote>
<p>실무에서는 자주 재사용되는 아주 기본적인 버튼, 아이콘 정도만 전역 등록하고, 대부분은 <strong>지역 등록</strong>을 사용합니다. 필요한 곳에서만 명시적으로 import하는 것이 코드 추적에 유리하기 때문입니다.</p>
</blockquote>
<h2 id="2-4-component-lifecycle-생명주기">2-4. Component Lifecycle (생명주기)</h2>
<p>컴포넌트도 사람처럼 &quot;태어나서 - 자라고 - 죽는&quot; 일생이 있습니다. 이 각 단계를 <strong>Lifecycle</strong>이라고 부릅니다.</p>
<table>
<thead>
<tr>
<th>단계</th>
<th>상태</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>1. 생성 (Creation)</td>
<td>메모리상에 막 태어난 단계</td>
<td>데이터(ref, reactive)와 computed, watch가 초기화됨</td>
</tr>
<tr>
<td>2. 부착 (Mounting)</td>
<td>실제 브라우저 화면(DOM)에 붙는 단계</td>
<td>화면 요소에 접근 가능, API 호출하기 가장 좋은 타이밍</td>
</tr>
<tr>
<td>3. 갱신 (Updating)</td>
<td>데이터가 바뀌어 화면을 다시 그리는 단계</td>
<td>크기/스크롤 재계산 등에 사용</td>
</tr>
<tr>
<td>4. 소멸 (Unmounting)</td>
<td>컴포넌트가 화면에서 완전히 사라지는 단계</td>
<td>타이머 정리, 이벤트 리스너 청소(메모리 누수 방지)</td>
</tr>
</tbody></table>
<p>각 단계에 진입할 때 Vue가 자동으로 실행해주는 함수를 <strong>Lifecycle Hook</strong>이라고 합니다.</p>
<table>
<thead>
<tr>
<th>훅 함수</th>
<th>실행 시점</th>
</tr>
</thead>
<tbody><tr>
<td><code>onMounted()</code></td>
<td>DOM에 마운트된 후 (초기 데이터 요청, DOM 조작에 사용)</td>
</tr>
<tr>
<td><code>onUpdated()</code></td>
<td>DOM 업데이트가 완료된 후</td>
</tr>
<tr>
<td><code>onUnmounted()</code></td>
<td>컴포넌트가 제거된 후 (타이머/이벤트 정리)</td>
</tr>
</tbody></table>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref, onMounted, onUpdated, onUnmounted } from 'vue'

const count = ref(0)
let timerId = null

// 생성 단계 = &lt;script setup&gt; 코드 자체가 실행되는 시점
console.log('1. 컴포넌트가 메모리에 생성됨 (DOM 접근 불가)')

onMounted(() =&gt; {
  console.log('2. 화면에 부착 완료! (API 호출/DOM 조작 적기)')
  timerId = setInterval(() =&gt; { count.value++ }, 3000)
})

onUpdated(() =&gt; {
  console.log(`3. 화면이 다시 그려짐 (현재 count: ${count.value})`)
})

onUnmounted(() =&gt; {
  clearInterval(timerId) // ❌ 안 지우면 메모리 누수!
  console.log('4. 컴포넌트 소멸, 타이머 정리 완료')
})
&lt;/script&gt;</code></pre>
<blockquote>
<p>⚠️ <strong>가장 중요한 실무 포인트</strong>: <code>onMounted()</code>에서 타이머나 이벤트 리스너를 등록했다면, 반드시 <code>onUnmounted()</code>에서 정리(clean-up)해줘야 합니다. 안 그러면 컴포넌트가 사라져도 백그라운드에서 계속 동작하는 <strong>메모리 누수</strong>가 발생합니다.</p>
</blockquote>
<h2 id="2-5-props--emits--부모-자식-간-데이터-통신">2-5. Props &amp; Emits — 부모-자식 간 데이터 통신</h2>
<p>Vue의 컴포넌트 통신 원칙은 딱 한 문장으로 요약됩니다.</p>
<blockquote>
<p><strong>&quot;데이터는 위에서 아래로(Props), 이벤트는 아래에서 위로(Emits)&quot;</strong></p>
</blockquote>
<table>
<thead>
<tr>
<th>구분</th>
<th>🔽 Props (하행선)</th>
<th>🔼 Emits (상행선)</th>
</tr>
</thead>
<tbody><tr>
<td>개념</td>
<td>부모가 자식에게 주는 데이터</td>
<td>자식이 부모에게 보고하는 이벤트</td>
</tr>
<tr>
<td>방향</td>
<td>부모 → 자식</td>
<td>자식 → 부모</td>
</tr>
<tr>
<td>권한</td>
<td>읽기 전용 (자식이 수정 불가)</td>
<td>부모에게 값 전달 가능</td>
</tr>
<tr>
<td>문법</td>
<td><code>defineProps({...})</code></td>
<td><code>defineEmits([...])</code></td>
</tr>
</tbody></table>
<h3 id="defineprops--자식이-데이터-받기"><code>defineProps()</code> — 자식이 데이터 받기</h3>
<p>자식 컴포넌트 내부에서 &quot;부모가 넘겨줄 데이터의 이름과 규격&quot;을 선언합니다. <code>&lt;script setup&gt;</code> 안에서 import 없이 바로 사용 가능한 특수 함수(Compiler Macro)입니다.</p>
<pre><code class="language-vue">&lt;!-- ParentComponent.vue --&gt;
&lt;script setup&gt;
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'
const parentMessage = ref('안녕하세요, 자식 컴포넌트!')
&lt;/script&gt;
&lt;template&gt;
  &lt;ChildComponent :message=&quot;parentMessage&quot; /&gt;
&lt;/template&gt;</code></pre>
<pre><code class="language-vue">&lt;!-- ChildComponent.vue --&gt;
&lt;script setup&gt;
const props = defineProps({ message: String })
&lt;/script&gt;
&lt;template&gt;
  &lt;div&gt;{{ message }}&lt;/div&gt;
&lt;/template&gt;</code></pre>
<p><strong>Props 유효성 검사</strong>를 걸면 더 견고한 컴포넌트를 만들 수 있습니다.</p>
<pre><code class="language-js">defineProps({
  temperature: {
    type: Number,
    required: true // 부모가 안 넘기면 경고 발생
  },
  status: {
    type: String,
    default: '맑음' // 부모가 안 넘기면 기본값 사용
  },
  weeklyForecast: {
    type: Array,
    default: () =&gt; [] // 배열/객체 기본값은 반드시 함수 형태로!
  }
})</code></pre>
<blockquote>
<p>⚠️ <code>props.likes = 999</code>처럼 자식 컴포넌트가 전달받은 Props 값을 <strong>직접 수정하려 하면 에러</strong>가 납니다. Props는 읽기 전용(Read-only)입니다.</p>
</blockquote>
<blockquote>
<p>📌 <strong>네이밍 규칙</strong>: 컴포넌트 내부 데이터는 <code>camelCase</code>(예: <code>cityName</code>)로, HTML 태그의 속성으로 쓸 땐 <code>kebab-case</code>(예: <code>:city-name</code>)로 작성합니다. HTML은 대소문자를 구분하지 못하기 때문에 생긴 관례이며, Vue가 자동으로 서로 변환해줍니다.</p>
</blockquote>
<h3 id="defineemits--자식이-부모에게-신호-보내기"><code>defineEmits()</code> — 자식이 부모에게 신호 보내기</h3>
<pre><code class="language-vue">&lt;!-- ChildComponent.vue --&gt;
&lt;script setup&gt;
const emit = defineEmits(['childEvent'])
const sendToParent = () =&gt; {
  emit('childEvent', '안녕하세요, 부모 컴포넌트!')
}
&lt;/script&gt;
&lt;template&gt;
  &lt;button @click=&quot;sendToParent&quot;&gt;부모에게 메시지 보내기&lt;/button&gt;
&lt;/template&gt;</code></pre>
<pre><code class="language-vue">&lt;!-- ParentComponent.vue --&gt;
&lt;script setup&gt;
const handleChildEvent = (message) =&gt; {
  console.log('자식으로부터 받은 메시지:', message)
}
&lt;/script&gt;
&lt;template&gt;
  &lt;ChildComponent @childEvent=&quot;handleChildEvent&quot; /&gt;
&lt;/template&gt;</code></pre>
<p>동작 순서: ① 자식이 <code>defineEmits</code>로 이벤트 이름을 등록 → ② 원하는 시점에 <code>emit('이벤트이름', 데이터)</code> 호출 → ③ 부모는 <code>@이벤트이름=&quot;핸들러함수&quot;</code>로 수신.</p>
<h2 id="2-6-provide--inject--깊은-계층-건너뛰기">2-6. Provide &amp; Inject — 깊은 계층 건너뛰기</h2>
<p>컴포넌트 계층이 깊어지면(부모 → 자식 → 손자 → ...) 중간 컴포넌트들은 정작 필요하지도 않은 데이터를 오직 전달을 위해 계속 Props로 받아 아래로 넘겨줘야 하는 문제가 생깁니다. 이를 <strong>Props Drilling</strong> 이라고 부릅니다.</p>
<p><code>provide</code>/<code>inject</code>는 중간 계층을 완전히 건너뛰고, 조상 컴포넌트의 데이터를 후손 컴포넌트에서 바로 사용할 수 있게 해줍니다.</p>
<pre><code class="language-js">// GrandParent.vue
import { ref, provide } from 'vue'
const themeColor = ref('dark-mode')
provide('globalTheme', themeColor)</code></pre>
<pre><code class="language-js">// GrandChild.vue
import { inject } from 'vue'
const theme = inject('globalTheme')</code></pre>
<blockquote>
<p>참고로 실무에서는 전역 상태 관리 라이브러리인 <strong>Pinia</strong>(다음 시간에 배울 예정)를 더 많이 사용하기 때문에, <code>provide</code>/<code>inject</code>의 사용 빈도는 상대적으로 낮은 편입니다.</p>
</blockquote>
<h2 id="2-7-component-slot--레이아웃-주입하기">2-7. Component Slot — 레이아웃 주입하기</h2>
<p>Props가 <strong>데이터</strong>를 자식에게 넘긴다면, Slot은 <strong>HTML 마크업(레이아웃) 자체</strong>를 자식에게 넘깁니다. 즉, 자식 컴포넌트의 특정 영역을 비워두고, 부모가 그 자리를 채울 내용을 결정하는 기능입니다.</p>
<p><strong>① Default Slot (이름 없는 슬롯)</strong></p>
<pre><code class="language-vue">&lt;!-- SlotDefaultChild.vue --&gt;
&lt;template&gt;
  &lt;div class=&quot;base-card&quot;&gt;
    &lt;slot&gt;
      &lt;p&gt;기본 콘텐츠 영역입니다.&lt;/p&gt; &lt;!-- 부모가 아무것도 안 넣으면 이게 보임 --&gt;
    &lt;/slot&gt;
  &lt;/div&gt;
&lt;/template&gt;</code></pre>
<pre><code class="language-vue">&lt;!-- SlotDefaultParent.vue --&gt;
&lt;template&gt;
  &lt;SlotDefaultChild&gt;
    &lt;p&gt;단순한 텍스트 문장을 주입합니다.&lt;/p&gt;
  &lt;/SlotDefaultChild&gt;
&lt;/template&gt;</code></pre>
<p><strong>② Named Slot (이름 있는 슬롯)</strong> — 슬롯이 여러 개 필요할 때</p>
<pre><code class="language-vue">&lt;!-- Child: &lt;slot name=&quot;header&quot;&gt;&lt;/slot&gt; --&gt;
&lt;template v-slot:header&gt;
  &lt;h3&gt;Child 주입 제목&lt;/h3&gt;
&lt;/template&gt;</code></pre>
<p><strong>③ Scoped Slot</strong> — 반대로 자식이 부모에게 데이터를 넘길 때</p>
<pre><code class="language-vue">&lt;!-- SlotScopedChild.vue --&gt;
&lt;script setup&gt;
import { ref } from 'vue'
const message = ref('현재 서버 상태 정상')
const userCount = ref(150)
&lt;/script&gt;
&lt;template&gt;
  &lt;slot :text=&quot;message&quot; :count=&quot;userCount&quot;&gt;&lt;/slot&gt;
&lt;/template&gt;</code></pre>
<pre><code class="language-vue">&lt;!-- SlotScopedParent.vue --&gt;
&lt;template&gt;
  &lt;SlotScopedChild v-slot=&quot;slotBag&quot;&gt;
    &lt;p&gt;알림 메시지: {{ slotBag.text }}&lt;/p&gt;
    &lt;p&gt;접속자 수: {{ slotBag.count }}명&lt;/p&gt;
  &lt;/SlotScopedChild&gt;
&lt;/template&gt;</code></pre>
<table>
<thead>
<tr>
<th>Slot 종류</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td>Default Slot</td>
<td>자식의 빈 공간에 부모가 콘텐츠 채우기</td>
</tr>
<tr>
<td>Named Slot</td>
<td>여러 위치를 각각 다른 콘텐츠로 채우기</td>
</tr>
<tr>
<td>Scoped Slot</td>
<td>자식의 데이터를 부모 쪽에서 활용하기</td>
</tr>
</tbody></table>
<hr />
<h1 id="🎯-오늘-배운-내용-한눈에-정리">🎯 오늘 배운 내용 한눈에 정리</h1>
<pre><code>Composition API
 ├─ 반응형 상태: ref(), reactive()
 └─ 계산/감시: computed(), watch(), watchEffect()

Vue Component
 ├─ 개념 &amp; 계층 구조 (부모-자식, 형제, 조상-후손)
 ├─ 등록 방법: 지역 등록 / 전역 등록
 ├─ Lifecycle: 생성 → 부착 → 갱신 → 소멸
 ├─ 통신 방법
 │   ├─ Props (부모→자식, 데이터)
 │   ├─ Emits (자식→부모, 이벤트)
 │   └─ Provide/Inject (조상→후손 직행)
 └─ Slot: 레이아웃 자체를 주입 (Default/Named/Scoped)</code></pre>