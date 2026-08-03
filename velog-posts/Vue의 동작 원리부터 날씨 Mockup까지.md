<h2 id="오늘의-학습-목표">오늘의 학습 목표</h2>
<p>오늘은 Vue 문법을 외우는 날이 아닙니다. 다음 흐름을 이해하는 날입니다.</p>
<pre><code class="language-text">JavaScript 상태가 바뀐다
        ↓
Vue가 변화를 감지한다
        ↓
Template이 새 상태를 표현한다
        ↓
필요한 DOM만 효율적으로 갱신된다</code></pre>
<ul>
<li>Vue가 어떤 문제를 해결하는지 설명할 수 있다.</li>
<li>Vite 기반 Vue 프로젝트를 만들고 실행할 수 있다.</li>
<li><code>.vue</code> 파일의 <code>script</code>, <code>template</code>, <code>style</code> 역할을 구분할 수 있다.</li>
<li><code>ref</code>로 화면과 연결되는 반응형 상태를 만들 수 있다.</li>
<li>주요 디렉티브로 속성, 조건, 반복, 이벤트, 폼을 제어할 수 있다.</li>
<li>오늘 배운 문법만으로 간단한 날씨 Mockup을 완성할 수 있다.</li>
</ul>
<hr />
<h1 id="1-vuejs는-왜-사용할까">1. Vue.js는 왜 사용할까?</h1>
<h2 id="11-vuejs란">1.1 Vue.js란?</h2>
<p>Vue.js는 사용자 인터페이스를 만들기 위한 JavaScript 프레임워크입니다. HTML, CSS, JavaScript만으로도 화면을 만들 수 있지만, 화면의 상태가 많아지면 직접 DOM을 찾고 수정하는 코드가 급격히 복잡해집니다.</p>
<p>순수 JavaScript에서는 보통 다음과 같이 작성합니다.</p>
<pre><code class="language-js">let count = 0

button.addEventListener('click', () =&gt; {
  count++
  document.querySelector('#count').textContent = count
})</code></pre>
<p>개발자는 데이터인 <code>count</code>와 화면인 <code>#count</code>를 함께 관리해야 합니다. Vue에서는 상태만 변경합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const count = ref(0)
&lt;/script&gt;

&lt;template&gt;
  &lt;p&gt;{{ count }}&lt;/p&gt;
  &lt;button @click=&quot;count++&quot;&gt;증가&lt;/button&gt;
&lt;/template&gt;</code></pre>
<p><code>count</code>가 바뀌면 Vue가 화면 갱신을 담당합니다. 이것이 Vue를 배우는 가장 중요한 이유입니다.</p>
<blockquote>
<p>핵심: 개발자는 상태를 선언하고, 화면이 그 상태를 어떻게 보여줄지 작성합니다. DOM 갱신은 Vue가 처리합니다.</p>
</blockquote>
<h2 id="12-mvvm으로-이해하는-vue">1.2 MVVM으로 이해하는 Vue</h2>
<p>Vue를 MVVM 관점으로 설명합니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>Vue에서의 의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>Model</td>
<td>화면이 사용할 데이터와 로직</td>
<td><code>ref</code>, JavaScript 객체, 함수</td>
</tr>
<tr>
<td>View</td>
<td>사용자가 실제로 보는 화면</td>
<td><code>&lt;template&gt;</code>, 브라우저 DOM</td>
</tr>
<tr>
<td>ViewModel</td>
<td>Model과 View를 연결하는 중재자</td>
<td>Vue의 반응성·바인딩 시스템</td>
</tr>
</tbody></table>
<p>예를 들어 입력창에 이름을 입력하면 다음 연결이 일어납니다.</p>
<pre><code class="language-text">사용자 입력(View)
    ↕ v-model
name 상태(Model)</code></pre>
<p>이 구조 덕분에 개발자는 <code>input.value</code>를 직접 읽거나 화면의 텍스트를 매번 직접 교체할 필요가 없습니다.</p>
<h2 id="13-반응성reactivity">1.3 반응성(Reactivity)</h2>
<p>반응성이란 상태가 바뀌었을 때 그 상태를 사용하는 화면도 자동으로 갱신되는 성질입니다.</p>
<pre><code class="language-js">let normalCount = 0       // 값은 변하지만 Vue가 추적하지 않음
const vueCount = ref(0)   // Vue가 변화를 추적함</code></pre>
<p>둘 다 JavaScript 내부 값은 변합니다. 차이는 Vue가 변경을 알고 화면을 다시 그릴 수 있느냐입니다.</p>
<p>반응성은 Vue의 여러 기능을 하나로 묶는 중심 개념입니다.</p>
<ul>
<li><code>{{ count }}</code>는 반응형 값을 화면에 표시합니다.</li>
<li><code>:class</code>는 반응형 값에 따라 클래스를 바꿉니다.</li>
<li><code>v-if</code>는 반응형 조건에 따라 요소를 만들거나 제거합니다.</li>
<li><code>v-for</code>는 반응형 배열의 변화에 맞춰 목록을 갱신합니다.</li>
<li><code>v-model</code>은 입력값과 반응형 상태를 연결합니다.</li>
</ul>
<h2 id="14-virtual-dom">1.4 Virtual DOM</h2>
<p>DOM은 브라우저가 HTML을 객체 트리로 표현한 것입니다. DOM을 반복해서 변경하면 브라우저는 레이아웃 계산과 화면 그리기 작업을 수행해야 합니다.</p>
<p>Vue는 상태가 변할 때 새로운 화면의 모습을 메모리상의 Virtual DOM으로 계산하고, 이전 결과와 비교해 필요한 실제 DOM 변경을 적용합니다. 같은 작업 안에서 연속으로 발생한 상태 변경을 모아 처리하는 것도 불필요한 렌더링을 줄이는 데 도움이 됩니다.</p>
<p>초심자는 다음 한 문장으로 기억하면 충분합니다.</p>
<blockquote>
<p>Virtual DOM은 실제 DOM을 무조건 전부 다시 만드는 기술이 아니라, 상태 변화로 인해 필요한 화면 변경을 계산하기 위한 중간 표현입니다.</p>
</blockquote>
<h2 id="15-컴포넌트-기반-개발">1.5 컴포넌트 기반 개발</h2>
<p>Vue는 화면을 작은 부품인 컴포넌트로 나눕니다.</p>
<pre><code class="language-text">App.vue
├── HeaderBar.vue
├── SearchBar.vue
└── WeatherList.vue
    ├── WeatherCard.vue
    ├── WeatherCard.vue
    └── WeatherCard.vue</code></pre>
<p>컴포넌트의 장점은 다음과 같습니다.</p>
<ul>
<li>응집성: 구조, 동작, 스타일을 한 파일에서 관리합니다.</li>
<li>재사용성: 같은 날씨 카드를 도시별로 반복 사용할 수 있습니다.</li>
<li>독립성: 카드 내부 수정이 다른 화면에 미치는 영향을 줄일 수 있습니다.</li>
<li>유지보수성: 문제가 생긴 부분을 컴포넌트 단위로 찾을 수 있습니다.</li>
</ul>
<h2 id="16-spa-csr-ssr을-구분하자">1.6 SPA, CSR, SSR을 구분하자</h2>
<p>서로 관련된 용어지만 같은 뜻은 아닙니다.</p>
<h3 id="spa와-mpa">SPA와 MPA</h3>
<ul>
<li>MPA: 페이지 이동 때 서버에서 새로운 HTML 문서를 받습니다.</li>
<li>SPA: 최초에 하나의 HTML을 받고, 이후 JavaScript가 필요한 화면 영역을 교체합니다.</li>
</ul>
<h3 id="csr과-ssr">CSR과 SSR</h3>
<ul>
<li>CSR: 브라우저가 JavaScript를 실행해 최종 화면을 완성합니다.</li>
<li>SSR: 서버가 내용(데이터)이 채워진 HTML을 먼저 만들어 전달합니다.</li>
</ul>
<p>Vue + Vite로 만드는 기본 프로젝트는 보통 SPA이자 CSR 방식입니다. Vue Router가 URL과 화면 컴포넌트를 연결합니다. 검색 노출이나 첫 화면 표시가 매우 중요한 서비스라면 Nuxt 같은 SSR 도구를 함께 고려할 수 있습니다.</p>
<table>
<thead>
<tr>
<th>관점</th>
<th>SPA/MPA</th>
<th>CSR/SSR</th>
</tr>
</thead>
<tbody><tr>
<td>핵심 질문</td>
<td>페이지를 몇 개의 HTML 문서로 운영하는가?</td>
<td>최종 HTML을 어디에서 만드는가?</td>
</tr>
<tr>
<td>전형적인 Vue 앱</td>
<td>SPA</td>
<td>CSR</td>
</tr>
<tr>
<td>장점</td>
<td>빠른 화면 전환</td>
<td>프론트엔드 상호작용 구현이 편리함</td>
</tr>
<tr>
<td>고려사항</td>
<td>새로고침 경로 설정, 상태 초기화</td>
<td>초기 JS 크기, SEO</td>
</tr>
</tbody></table>
<h2 id="17-vue-spa를-구성하는-도구">1.7 Vue SPA를 구성하는 도구</h2>
<table>
<thead>
<tr>
<th>도구</th>
<th>역할</th>
<th>쉬운 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Vue</td>
<td>UI와 반응성</td>
<td>화면 엔진</td>
</tr>
<tr>
<td>Vite</td>
<td>개발 서버와 빌드</td>
<td>조립 공장</td>
</tr>
<tr>
<td>Vue Router</td>
<td>URL별 화면 전환</td>
<td>길 안내 시스템</td>
</tr>
<tr>
<td>Pinia</td>
<td>여러 컴포넌트가 공유하는 상태</td>
<td>공용 창고</td>
</tr>
<tr>
<td>Axios</td>
<td>백엔드 API 통신</td>
<td>데이터 배달원</td>
</tr>
</tbody></table>
<p>오늘은 Vue와 Vite를 중심으로 배우며, Router·Pinia·Axios는 이후 파트에서 확장합니다.</p>
<hr />
<h1 id="2-개발환경과-첫-프로젝트">2. 개발환경과 첫 프로젝트</h1>
<h2 id="21-nodejs가-필요한-이유">2.1 Node.js가 필요한 이유</h2>
<p>브라우저가 Vue 프로젝트의 개발 도구를 직접 실행하는 것은 아닙니다. Node.js가 Vite와 npm을 실행합니다.</p>
<ul>
<li>Node.js: JavaScript를 브라우저 밖에서 실행하는 환경</li>
<li>npm: 패키지를 설치하고 프로젝트 명령을 실행하는 도구</li>
<li>Vite: 개발 서버, HMR, 변환, 번들링을 담당하는 빌드 도구</li>
</ul>
<pre><code class="language-text">개발자가 .vue 파일 작성
        ↓
Node.js 위에서 Vite 실행
        ↓
브라우저가 이해할 수 있는 모듈 제공
        ↓
브라우저에 화면 표시</code></pre>
<h2 id="22-프로젝트-생성">2.2 프로젝트 생성</h2>
<pre><code class="language-bash">npm create vue@latest</code></pre>
<p>학습 프로젝트에서는 다음 선택을 사용할 수 있습니다.</p>
<pre><code class="language-text">Project name: skala-vue
TypeScript: No
JSX: No
Vue Router: Yes
Pinia: Yes
Vitest: No
E2E Test: No
ESLint: Yes
Prettier: Yes</code></pre>
<p>이후 프로젝트를 실행합니다.</p>
<pre><code class="language-bash">cd skala-vue
npm install
npm run dev</code></pre>
<p>브라우저에서 <code>http://localhost:5173</code>에 접속합니다.</p>
<h2 id="23-꼭-알아야-하는-npm-명령">2.3 꼭 알아야 하는 npm 명령</h2>
<table>
<thead>
<tr>
<th>명령</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>npm install</code></td>
<td><code>package.json</code>에 기록된 패키지 설치</td>
</tr>
<tr>
<td><code>npm install 패키지명</code></td>
<td>새 패키지 추가</td>
</tr>
<tr>
<td><code>npm uninstall 패키지명</code></td>
<td>패키지 제거</td>
</tr>
<tr>
<td><code>npm run dev</code></td>
<td>Vite 개발 서버 실행</td>
</tr>
<tr>
<td><code>npm run build</code></td>
<td>배포용 <code>dist</code> 생성</td>
</tr>
</tbody></table>
<p><code>node_modules</code>는 설치된 패키지의 실제 파일이 들어 있는 매우 큰 폴더입니다. 직접 수정하지 않으며 Git에도 올리지 않습니다. 필요한 패키지 목록은 <code>package.json</code>, 정확한 설치 버전은 lock 파일이 관리합니다.</p>
<h2 id="24-hmr-체험">2.4 HMR 체험</h2>
<p>HMR(Hot Module Replacement)은 개발 서버를 끄거나 페이지 전체를 새로고침하지 않고 변경된 모듈을 즉시 반영하는 기능입니다.</p>
<ol>
<li><code>npm run dev</code>를 실행합니다.</li>
<li>브라우저에서 앱을 엽니다.</li>
<li><code>AboutView.vue</code>의 문구를 바꿉니다.</li>
<li>저장과 동시에 화면이 바뀌는지 확인합니다.</li>
</ol>
<p>Vue Devtools에서는 컴포넌트 트리, props, 반응형 상태, 라우팅 정보를 확인할 수 있습니다. 화면이 이상할 때 <code>console.log</code>만 보는 것보다 상태가 어느 컴포넌트에 있는지 먼저 확인하는 습관이 좋습니다.</p>
<hr />
<h1 id="3-vue-프로젝트의-실행-흐름">3. Vue 프로젝트의 실행 흐름</h1>
<h2 id="31-주요-폴더와-파일">3.1 주요 폴더와 파일</h2>
<pre><code class="language-text">skala-vue/
├── public/             # 변환 없이 그대로 제공되는 정적 파일
├── src/
│   ├── assets/         # Vite가 처리하는 이미지, CSS 등의 자원
│   ├── components/     # 재사용 가능한 작은 UI 부품
│   ├── views/          # URL과 연결되는 페이지 단위 컴포넌트
│   ├── router/         # 라우팅 규칙
│   ├── stores/         # Pinia 전역 상태
│   ├── App.vue         # 루트 컴포넌트
│   └── main.js         # JavaScript 진입점
├── index.html          # 브라우저가 처음 읽는 HTML
├── package.json        # 명령과 의존성 목록
└── vite.config.js      # Vite 설정</code></pre>
<table>
<thead>
<tr>
<th>디렉터리/파일</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>index.html</code></td>
<td>애플리케이션 진입점(Entry Point). 브라우저가 최초로 읽는 단 하나의 HTML 파일</td>
</tr>
<tr>
<td><code>package.json</code></td>
<td>프로젝트 메타정보, 실행 스크립트 명령어, 의존성 라이브러리 목록 기록</td>
</tr>
<tr>
<td><code>vite.config.js</code></td>
<td>Vite 빌드 엔진 설정 파일</td>
</tr>
<tr>
<td><code>.gitignore</code></td>
<td>Git이 추적하지 않을 파일과 폴더 지정</td>
</tr>
<tr>
<td><code>public/</code></td>
<td>브라우저에 그대로 제공되는 정적 파일 폴더. Vite가 컴파일하지 않음</td>
</tr>
<tr>
<td><code>src/</code></td>
<td>실제 소스 코드를 작성하는 디렉터리</td>
</tr>
<tr>
<td><code>src/main.js</code></td>
<td>애플리케이션의 JavaScript 진입점. <code>index.html</code>에서 호출</td>
</tr>
<tr>
<td><code>src/App.vue</code></td>
<td>애플리케이션의 루트 Vue 컴포넌트. <code>main.js</code>에서 호출</td>
</tr>
<tr>
<td><code>src/style.css</code></td>
<td>스타일(CSS) 파일</td>
</tr>
<tr>
<td><code>src/assets/</code></td>
<td>CSS, 로고, 이미지, 폰트 등 Vite의 컴파일과 최적화가 필요한 자원</td>
</tr>
<tr>
<td><code>src/components/</code></td>
<td>재사용 가능한 작은 UI 컴포넌트를 보관하는 폴더</td>
</tr>
<tr>
<td><code>src/components/HelloWorld.vue</code></td>
<td>기본으로 생성되는 예제 컴포넌트. <code>App.vue</code>에서 호출</td>
</tr>
<tr>
<td><code>src/router/</code></td>
<td>SPA의 페이지 이동 경로를 정의하는 폴더</td>
</tr>
<tr>
<td><code>src/stores/</code></td>
<td>Pinia를 이용한 전역 상태 저장소</td>
</tr>
<tr>
<td><code>src/views/</code></td>
<td>컴포넌트들을 조립해 완성한 페이지 단위 화면을 보관하는 폴더</td>
</tr>
</tbody></table>
<h2 id="32-브라우저에서-appvue까지">3.2 브라우저에서 App.vue까지</h2>
<p>실행 순서를 이해하면 프로젝트가 커져도 길을 잃지 않습니다.</p>
<pre><code class="language-text">index.html의 #app
        ↓
main.js 실행
        ↓
createApp(App)으로 앱 생성
        ↓
router와 pinia 같은 플러그인 등록
        ↓
mount('#app')
        ↓
App.vue와 자식 컴포넌트 렌더링</code></pre>
<p>대표적인 <code>main.js</code>는 다음과 같습니다.</p>
<pre><code class="language-js">import './assets/main.css'
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.mount('#app')</code></pre>
<hr />
<h1 id="4-sfc와-template-기본-문법">4. SFC와 Template 기본 문법</h1>
<h2 id="41-sfc란">4.1 SFC란?</h2>
<p>SFC(Single File Component)는 <code>.vue</code> 확장자를 사용하는 단일 파일 컴포넌트입니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
// 상태와 함수
&lt;/script&gt;

&lt;template&gt;
  &lt;!-- 화면 구조 --&gt;
&lt;/template&gt;

&lt;style scoped&gt;
/* 이 컴포넌트에 적용할 스타일 */
&lt;/style&gt;</code></pre>
<table>
<thead>
<tr>
<th>영역</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;script setup&gt;</code></td>
<td>데이터, 함수, 이벤트 처리 등 동작</td>
</tr>
<tr>
<td><code>&lt;template&gt;</code></td>
<td>사용자가 보게 될 HTML 구조</td>
</tr>
<tr>
<td><code>&lt;style scoped&gt;</code></td>
<td>해당 컴포넌트 중심의 CSS</td>
</tr>
</tbody></table>
<p>컴포넌트 파일명은 <code>WeatherCard.vue</code>처럼 두 단어 이상의 PascalCase를 사용하면 역할이 분명해집니다.</p>
<h2 id="42-options-api와-composition-api">4.2 Options API와 Composition API</h2>
<p>Vue 3에서는 두 방식 모두 사용할 수 있지만, 이 과정은 <code>&lt;script setup&gt;</code>을 사용하는 Composition API 방식으로 진행합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const count = ref(0)
const increase = () =&gt; {
  count.value++
}
&lt;/script&gt;</code></pre>
<p>관련 있는 상태와 함수를 가까이 둘 수 있어 기능 단위로 코드를 읽기 좋습니다.</p>
<h2 id="43-ref와-value">4.3 <code>ref</code>와 <code>.value</code></h2>
<p><code>ref</code>는 값을 Vue가 추적할 수 있는 반응형 상자로 만듭니다.</p>
<pre><code class="language-js">const count = ref(0)

console.log(count.value)
count.value++</code></pre>
<p>사용 위치에 따라 접근법이 다릅니다.</p>
<table>
<thead>
<tr>
<th>위치</th>
<th>사용법</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;script setup&gt;</code></td>
<td><code>.value</code> 필요</td>
<td><code>count.value++</code></td>
</tr>
<tr>
<td><code>&lt;template&gt;</code></td>
<td><code>.value</code> 생략</td>
<td><code>{{ count }}</code></td>
</tr>
</tbody></table>
<p>단, 템플릿 안의 간단한 표현식에서는 Vue가 ref를 자동으로 풀어주므로 <code>@click=&quot;count++&quot;</code>처럼 쓸 수 있습니다.</p>
<h2 id="44-텍스트-보간법">4.4 텍스트 보간법</h2>
<p>이중 중괄호 <code>{{ }}</code>는 JavaScript 값을 텍스트로 출력합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const city = ref('서울')
const temperature = ref(28)
&lt;/script&gt;

&lt;template&gt;
  &lt;p&gt;{{ city }}의 현재 기온은 {{ temperature }}도입니다.&lt;/p&gt;
  &lt;p&gt;내일 예상 기온: {{ temperature + 2 }}도&lt;/p&gt;
  &lt;p&gt;상태: {{ temperature &gt;= 25 ? '더움' : '선선함' }}&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p>보간식에는 값을 반환하는 간단한 표현식을 넣을 수 있습니다. 여러 문장, 반복문, 복잡한 분기는 템플릿이 아니라 script의 함수나 이후에 배울 계산된 속성으로 옮기는 것이 좋습니다.</p>
<hr />
<h1 id="5-directive-html에-vue의-동작을-연결하기">5. Directive: HTML에 Vue의 동작을 연결하기</h1>
<p>디렉티브는 <code>v-</code>로 시작하는 Vue 전용 HTML 속성입니다. 따옴표 안은 일반 문자열이 아니라 JavaScript 표현식으로 평가됩니다.</p>
<pre><code class="language-vue">&lt;button v-if=&quot;isVisible&quot; v-on:click=&quot;handleClick&quot;&gt;확인&lt;/button&gt;</code></pre>
<h2 id="51-자주-쓰는-디렉티브-지도">5.1 자주 쓰는 디렉티브 지도</h2>
<table>
<thead>
<tr>
<th>문법</th>
<th>목적</th>
<th>자주 쓰는 축약형</th>
</tr>
</thead>
<tbody><tr>
<td><code>v-text</code></td>
<td>텍스트 내용 출력</td>
<td>없음</td>
</tr>
<tr>
<td><code>v-html</code></td>
<td>문자열을 HTML로 해석</td>
<td>없음</td>
</tr>
<tr>
<td><code>v-bind</code></td>
<td>HTML 속성과 상태 연결</td>
<td><code>:</code></td>
</tr>
<tr>
<td><code>v-if</code></td>
<td>조건에 따라 요소 생성·제거</td>
<td>없음</td>
</tr>
<tr>
<td><code>v-show</code></td>
<td>조건에 따라 CSS로 표시·숨김</td>
<td>없음</td>
</tr>
<tr>
<td><code>v-for</code></td>
<td>배열이나 객체 반복 렌더링</td>
<td>없음</td>
</tr>
<tr>
<td><code>v-on</code></td>
<td>이벤트 리스너 연결</td>
<td><code>@</code></td>
</tr>
<tr>
<td><code>v-model</code></td>
<td>폼 입력과 상태의 양방향 연결</td>
<td>없음</td>
</tr>
</tbody></table>
<h2 id="52-v-text와-v-html">5.2 <code>v-text</code>와 <code>v-html</code></h2>
<pre><code class="language-vue">&lt;script setup&gt;
const plainText = '&lt;strong&gt;서울&lt;/strong&gt;'
const htmlText = '&lt;strong&gt;서울&lt;/strong&gt;'
&lt;/script&gt;

&lt;template&gt;
  &lt;p v-text=&quot;plainText&quot;&gt;&lt;/p&gt;
  &lt;p v-html=&quot;htmlText&quot;&gt;&lt;/p&gt;
&lt;/template&gt;</code></pre>
<ul>
<li><code>v-text</code>는 태그를 글자로 표시합니다.</li>
<li><code>v-html</code>은 태그를 실제 HTML로 해석합니다.</li>
</ul>
<p><code>v-html</code>에는 매우 중요한 보안 규칙이 있습니다.</p>
<blockquote>
<p>사용자 입력, 게시글 내용, 외부 API 응답처럼 신뢰할 수 없는 문자열을 <code>v-html</code>에 직접 넣지 마세요. 악성 스크립트가 실행되는 XSS 공격으로 이어질 수 있습니다.</p>
</blockquote>
<p>일반 텍스트는 <code>{{ value }}</code> 또는 <code>v-text</code>를 사용하는 것이 안전합니다.</p>
<h2 id="53-v-bind-동적인-속성">5.3 <code>v-bind</code>: 동적인 속성</h2>
<pre><code class="language-vue">&lt;script setup&gt;
const imageUrl = '/images/sunny.png'
const imageAlt = '맑은 날씨 아이콘'
&lt;/script&gt;

&lt;template&gt;
  &lt;img v-bind:src=&quot;imageUrl&quot; v-bind:alt=&quot;imageAlt&quot; /&gt;
  &lt;img :src=&quot;imageUrl&quot; :alt=&quot;imageAlt&quot; /&gt;
&lt;/template&gt;</code></pre>
<p>실무에서는 <code>v-bind:</code> 대신 <code>:</code>를 주로 사용합니다.</p>
<h3 id="class-binding">Class binding</h3>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const isHot = ref(true)
&lt;/script&gt;

&lt;template&gt;
  &lt;p :class=&quot;{ hot: isHot, cool: !isHot }&quot;&gt;오늘의 기온&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p>객체의 key는 클래스 이름이고 value는 적용 조건입니다.</p>
<pre><code class="language-vue">&lt;p :class=&quot;['weather-label', isHot ? 'hot' : 'cool']&quot;&gt;상태&lt;/p&gt;</code></pre>
<h3 id="style-binding">Style binding</h3>
<pre><code class="language-vue">&lt;script setup&gt;
const temperatureStyle = {
  color: 'tomato',
  fontWeight: '700',
}
&lt;/script&gt;

&lt;template&gt;
  &lt;p :style=&quot;temperatureStyle&quot;&gt;28°C&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p>반복되는 디자인은 <code>:style</code>보다 클래스로 분리하는 편이 유지보수에 유리합니다. <code>:style</code>은 사용자 설정값이나 계산된 크기처럼 정말 동적인 값에 적합합니다.</p>
<h3 id="같은-이름-축약">같은 이름 축약</h3>
<p>속성명과 변수명이 같다면 다음과 같이 줄일 수 있습니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
const id = 'city_01'
&lt;/script&gt;

&lt;template&gt;
  &lt;div :id=&quot;id&quot;&gt;&lt;/div&gt;
  &lt;div :id&gt;&lt;/div&gt;
&lt;/template&gt;</code></pre>
<p>처음에는 <code>:id=&quot;id&quot;</code>를 사용해 연결 관계를 명확히 익힌 뒤 축약형을 사용해도 좋습니다.</p>
<h2 id="54-v-if와-v-show">5.4 <code>v-if</code>와 <code>v-show</code></h2>
<pre><code class="language-vue">&lt;p v-if=&quot;temperature &gt;= 30&quot;&gt;폭염 주의&lt;/p&gt;
&lt;p v-else-if=&quot;temperature &gt;= 25&quot;&gt;더운 날씨&lt;/p&gt;
&lt;p v-else&gt;선선한 날씨&lt;/p&gt;</code></pre>
<p><code>v-if</code> 계열은 조건에 따라 실제 DOM을 만들거나 제거합니다.</p>
<pre><code class="language-vue">&lt;p v-show=&quot;isPanelOpen&quot;&gt;상세 패널&lt;/p&gt;</code></pre>
<p><code>v-show</code>는 요소를 유지한 채 <code>display: none</code>을 사용해 숨깁니다.</p>
<table>
<thead>
<tr>
<th>상황</th>
<th>권장</th>
</tr>
</thead>
<tbody><tr>
<td>로그인 여부처럼 조건이 가끔 바뀜</td>
<td><code>v-if</code></td>
</tr>
<tr>
<td>토글 메뉴처럼 매우 자주 열고 닫음</td>
<td><code>v-show</code></td>
</tr>
<tr>
<td>조건이 false일 때 요소 자체가 없어야 함</td>
<td><code>v-if</code></td>
</tr>
<tr>
<td>초기 렌더링 비용보다 전환 속도가 중요함</td>
<td><code>v-show</code></td>
</tr>
</tbody></table>
<h2 id="55-v-for와-key">5.5 <code>v-for</code>와 <code>key</code></h2>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const weatherList = ref([
  { id: 'city_01', name: '서울', temp: 28 },
  { id: 'city_02', name: '수원', temp: 24 },
  { id: 'city_03', name: '부산', temp: 26 },
])
&lt;/script&gt;

&lt;template&gt;
  &lt;article v-for=&quot;(city, index) in weatherList&quot; :key=&quot;city.id&quot;&gt;
    {{ index + 1 }}. {{ city.name }}: {{ city.temp }}°C
  &lt;/article&gt;
&lt;/template&gt;</code></pre>
<p><code>key</code>는 Vue가 각 항목의 정체성을 구분하는 값입니다. 목록 순서가 바뀌거나 항목이 추가·삭제될 때 어떤 DOM을 재사용할지 판단하는 기준이 됩니다.</p>
<p>좋은 key의 조건은 다음과 같습니다.</p>
<ul>
<li>항목마다 고유해야 합니다.</li>
<li>렌더링 사이에서 안정적으로 유지되어야 합니다.</li>
<li>가능하면 데이터의 <code>id</code>를 사용합니다.</li>
</ul>
<p>목록 순서가 달라질 수 있다면 배열의 index를 key로 쓰지 않는 것이 좋습니다. 입력창이나 컴포넌트 내부 상태가 엉뚱한 항목에 남을 수 있기 때문입니다.</p>
<h2 id="56-알아두면-좋은-렌더링-제어">5.6 알아두면 좋은 렌더링 제어</h2>
<table>
<thead>
<tr>
<th>디렉티브</th>
<th>동작</th>
<th>활용</th>
</tr>
</thead>
<tbody><tr>
<td><code>v-pre</code></td>
<td>Vue 문법을 해석하지 않음</td>
<td><code>{{ message }}</code> 자체를 예제로 표시</td>
</tr>
<tr>
<td><code>v-cloak</code></td>
<td>Vue가 준비되기 전 템플릿 노출 방지</td>
<td>CDN 방식의 초기 깜빡임 방지</td>
</tr>
<tr>
<td><code>v-once</code></td>
<td>최초 한 번만 렌더링</td>
<td>이후 바뀌지 않는 정적 콘텐츠</td>
</tr>
<tr>
<td><code>v-memo</code></td>
<td>지정한 의존성이 바뀔 때만 하위 트리 갱신</td>
<td>큰 목록의 고급 최적화</td>
</tr>
</tbody></table>
<p><code>v-once</code>와 <code>v-memo</code>는 성능 문제가 실제로 확인된 뒤 사용하는 것이 좋습니다. 작은 화면에 미리 적용하면 코드만 어려워질 수 있습니다.</p>
<hr />
<h1 id="6-event-handling">6. Event Handling</h1>
<h2 id="61-v-on과-">6.1 <code>v-on</code>과 <code>@</code></h2>
<pre><code class="language-vue">&lt;button v-on:click=&quot;count++&quot;&gt;증가&lt;/button&gt;
&lt;button @click=&quot;count++&quot;&gt;증가&lt;/button&gt;</code></pre>
<h3 id="inline-handler">Inline handler</h3>
<p>짧고 단순한 상태 변경에 적합합니다.</p>
<pre><code class="language-vue">&lt;button @click=&quot;isOpen = !isOpen&quot;&gt;열기/닫기&lt;/button&gt;</code></pre>
<h3 id="method-handler">Method handler</h3>
<p>여러 단계의 로직, 검증, 재사용이 필요하면 함수로 분리합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const message = ref('대기 중')

const selectCity = (cityName) =&gt; {
  message.value = `${cityName}이 선택되었습니다.`
}
&lt;/script&gt;

&lt;template&gt;
  &lt;button @click=&quot;selectCity('서울')&quot;&gt;서울 선택&lt;/button&gt;
  &lt;p&gt;{{ message }}&lt;/p&gt;
&lt;/template&gt;</code></pre>
<h2 id="62-event-객체">6.2 Event 객체</h2>
<p>브라우저는 이벤트가 발생하면 관련 정보를 담은 Event 객체를 만듭니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
const handleClick = (event) =&gt; {
  console.log(event.type)
  console.log(event.target)
  console.log(event.currentTarget)
}
&lt;/script&gt;

&lt;template&gt;
  &lt;button @click=&quot;handleClick&quot;&gt;확인&lt;/button&gt;
&lt;/template&gt;</code></pre>
<ul>
<li><code>target</code>: 실제 이벤트가 시작된 가장 안쪽 요소</li>
<li><code>currentTarget</code>: 현재 이벤트 리스너가 연결된 요소</li>
<li><code>preventDefault()</code>: 링크 이동이나 폼 제출 같은 기본 동작 차단</li>
<li><code>stopPropagation()</code>: 부모 요소로 이벤트가 전파되는 버블링 차단</li>
</ul>
<p>다른 인자와 이벤트 객체를 함께 전달하려면 <code>$event</code>를 사용합니다.</p>
<pre><code class="language-vue">&lt;button @click=&quot;selectCity('서울', $event)&quot;&gt;서울&lt;/button&gt;</code></pre>
<h2 id="63-이벤트-수식어">6.3 이벤트 수식어</h2>
<p>Vue는 자주 쓰는 이벤트 제어를 수식어로 제공합니다.</p>
<pre><code class="language-vue">&lt;form @submit.prevent=&quot;submitForm&quot;&gt;
  &lt;button type=&quot;submit&quot;&gt;저장&lt;/button&gt;
&lt;/form&gt;

&lt;div @click=&quot;selectCard&quot;&gt;
  카드
  &lt;button @click.stop=&quot;showDetail&quot;&gt;상세보기&lt;/button&gt;
&lt;/div&gt;</code></pre>
<table>
<thead>
<tr>
<th>수식어</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>.prevent</code></td>
<td>기본 동작 방지</td>
<td><code>@submit.prevent</code></td>
</tr>
<tr>
<td><code>.stop</code></td>
<td>이벤트 전파 중단</td>
<td>카드 안쪽 버튼</td>
</tr>
<tr>
<td><code>.once</code></td>
<td>한 번만 실행</td>
<td>최초 동의 확인</td>
</tr>
<tr>
<td><code>.self</code></td>
<td>요소 자신이 직접 클릭된 경우만</td>
<td>배경 영역 클릭</td>
</tr>
<tr>
<td><code>.capture</code></td>
<td>캡처 단계에서 처리</td>
<td>고급 이벤트 흐름 제어</td>
</tr>
<tr>
<td><code>.enter</code></td>
<td>Enter 키일 때 실행</td>
<td>검색 입력 확정</td>
</tr>
<tr>
<td><code>.esc</code></td>
<td>Esc 키일 때 실행</td>
<td>모달 닫기</td>
</tr>
<tr>
<td><code>.ctrl</code>, <code>.alt</code>, <code>.shift</code>, <code>.meta</code></td>
<td>조합 키</td>
<td>단축키</td>
</tr>
</tbody></table>
<p>수식어는 왼쪽부터 순서대로 적용되므로 의미가 달라질 수 있습니다.</p>
<pre><code class="language-vue">@click.prevent.self
@click.self.prevent</code></pre>
<p>첫 번째는 클릭의 기본 동작을 먼저 막고 자신인지 검사합니다. 두 번째는 자신이 직접 클릭된 경우에만 기본 동작을 막습니다.</p>
<hr />
<h1 id="7-form과-v-model">7. Form과 <code>v-model</code></h1>
<h2 id="71-양방향-바인딩의-원리">7.1 양방향 바인딩의 원리</h2>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const keyword = ref('')
&lt;/script&gt;

&lt;template&gt;
  &lt;input v-model=&quot;keyword&quot; /&gt;
  &lt;p&gt;검색어: {{ keyword }}&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p><code>v-model</code>은 폼 요소의 값과 반응형 상태를 연결합니다. 텍스트 입력에서 핵심 원리는 다음 조합과 비슷합니다.</p>
<pre><code class="language-vue">&lt;input
  :value=&quot;keyword&quot;
  @input=&quot;keyword = $event.target.value&quot;
/&gt;</code></pre>
<p>즉, 상태를 입력창에 내려주는 <code>:value</code>와 사용자 입력을 상태에 올리는 <code>@input</code>을 묶은 편의 문법입니다.</p>
<h2 id="72-폼-요소에-맞는-초기값">7.2 폼 요소에 맞는 초기값</h2>
<table>
<thead>
<tr>
<th>폼 요소</th>
<th>권장 초기값</th>
<th>저장되는 값</th>
</tr>
</thead>
<tbody><tr>
<td>text, textarea</td>
<td><code>ref('')</code></td>
<td>문자열</td>
</tr>
<tr>
<td>단일 checkbox</td>
<td><code>ref(false)</code></td>
<td>Boolean</td>
</tr>
<tr>
<td>다중 checkbox</td>
<td><code>ref([])</code></td>
<td>선택한 value의 배열</td>
</tr>
<tr>
<td>radio</td>
<td><code>ref('')</code></td>
<td>선택한 하나의 value</td>
</tr>
<tr>
<td>select</td>
<td><code>ref('')</code></td>
<td>선택한 option의 value</td>
</tr>
</tbody></table>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const comment = ref('')
const isAgreed = ref(false)
const favoriteFruits = ref([])
const weatherType = ref('')
const selectedCity = ref('')
&lt;/script&gt;

&lt;template&gt;
  &lt;textarea v-model=&quot;comment&quot;&gt;&lt;/textarea&gt;

  &lt;label&gt;
    &lt;input v-model=&quot;isAgreed&quot; type=&quot;checkbox&quot; /&gt; 약관 동의
  &lt;/label&gt;

  &lt;label&gt;&lt;input v-model=&quot;favoriteFruits&quot; type=&quot;checkbox&quot; value=&quot;사과&quot; /&gt;사과&lt;/label&gt;
  &lt;label&gt;&lt;input v-model=&quot;favoriteFruits&quot; type=&quot;checkbox&quot; value=&quot;딸기&quot; /&gt;딸기&lt;/label&gt;

  &lt;label&gt;&lt;input v-model=&quot;weatherType&quot; type=&quot;radio&quot; value=&quot;맑음&quot; /&gt;맑음&lt;/label&gt;
  &lt;label&gt;&lt;input v-model=&quot;weatherType&quot; type=&quot;radio&quot; value=&quot;비&quot; /&gt;비&lt;/label&gt;

  &lt;select v-model=&quot;selectedCity&quot;&gt;
    &lt;option value=&quot;&quot;&gt;도시 선택&lt;/option&gt;
    &lt;option value=&quot;seoul&quot;&gt;서울&lt;/option&gt;
    &lt;option value=&quot;busan&quot;&gt;부산&lt;/option&gt;
  &lt;/select&gt;
&lt;/template&gt;</code></pre>
<p>초기값 타입이 맞지 않으면 체크 결과를 담을 수 없거나 예상과 다른 형변환이 발생합니다.</p>
<h2 id="73-v-model-수식어">7.3 <code>v-model</code> 수식어</h2>
<table>
<thead>
<tr>
<th>수식어</th>
<th>동작</th>
<th>사용 예</th>
</tr>
</thead>
<tbody><tr>
<td><code>.lazy</code></td>
<td>매 입력이 아니라 change 시점에 반영</td>
<td>입력 완료 후 검증</td>
</tr>
<tr>
<td><code>.number</code></td>
<td>가능한 경우 숫자로 변환</td>
<td>나이, 가격</td>
</tr>
<tr>
<td><code>.trim</code></td>
<td>문자열 양끝 공백 제거</td>
<td>이메일, 검색어</td>
</tr>
</tbody></table>
<pre><code class="language-vue">&lt;input v-model.lazy=&quot;memo&quot; /&gt;
&lt;input v-model.number=&quot;age&quot; type=&quot;number&quot; /&gt;
&lt;input v-model.trim=&quot;email&quot; type=&quot;email&quot; /&gt;</code></pre>
<p>필요하면 연결해서 사용할 수 있습니다.</p>
<pre><code class="language-vue">&lt;input v-model.trim.number=&quot;price&quot; /&gt;</code></pre>
<h2 id="74-한글-입력과-ime">7.4 한글 입력과 IME</h2>
<p>한글은 자음과 모음을 조합하는 IME 입력 과정을 거칩니다. 조합 중인 모든 글자 변화를 즉시 제어해야 하는 특별한 검색 UI라면 <code>v-model</code> 대신 <code>:value</code>와 <code>@input</code>을 직접 사용해 시점을 확인할 수 있습니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const searchKeyword = ref('')

const handleSearchInput = (event) =&gt; {
  searchKeyword.value = event.target.value
}
&lt;/script&gt;

&lt;template&gt;
  &lt;input
    :value=&quot;searchKeyword&quot;
    @input=&quot;handleSearchInput&quot;
    placeholder=&quot;도시 이름을 입력하세요&quot;
  /&gt;
  &lt;p&gt;검색 중인 도시: {{ searchKeyword }}&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p>일반적인 폼은 <code>v-model</code>로 시작하고, IME 조합 시점까지 제어해야 할 때 이벤트를 직접 다루는 방식으로 확장하면 됩니다.</p>
<hr />
<h1 id="8-component-style">8. Component Style</h1>
<h2 id="81-전역-스타일과-scoped-스타일">8.1 전역 스타일과 scoped 스타일</h2>
<pre><code class="language-vue">&lt;style&gt;
.title {
  color: blue;
}
&lt;/style&gt;</code></pre>
<p>일반 <code>&lt;style&gt;</code>은 다른 컴포넌트의 같은 선택자에도 영향을 줄 수 있습니다.</p>
<pre><code class="language-vue">&lt;style scoped&gt;
.title {
  color: tomato;
}
&lt;/style&gt;</code></pre>
<p><code>scoped</code>를 붙이면 Vue가 고유한 속성을 추가해 해당 컴포넌트 범위에 맞는 선택자로 변환합니다.</p>
<h2 id="82-스타일을-어디에-둘까">8.2 스타일을 어디에 둘까?</h2>
<ul>
<li>프로젝트 전체의 색상 초기화와 typography: <code>src/assets/main.css</code></li>
<li>한 컴포넌트에서만 쓰는 스타일: 해당 SFC의 <code>&lt;style scoped&gt;</code></li>
<li>여러 컴포넌트에서 재사용하는 버튼·카드 스타일: 별도 CSS 파일 또는 공통 컴포넌트</li>
</ul>
<pre><code class="language-js">// main.js
import './assets/main.css'</code></pre>
<p>특정 SFC에서 외부 CSS를 불러올 수도 있습니다.</p>
<pre><code class="language-vue">&lt;style&gt;
@import '@/assets/challenge.css';
&lt;/style&gt;</code></pre>
<hr />
<h1 id="9-종합실습-날씨-mockup-만들기">9. 종합실습: 날씨 Mockup 만들기</h1>
<p>이 실습은 오늘 배운 내용을 하나로 연결합니다.</p>
<ul>
<li><code>ref</code>: 날씨 목록, 검색어, 선택 메시지</li>
<li><code>v-for</code>: 도시별 카드 반복</li>
<li><code>:key</code>: 도시 id로 카드 식별</li>
<li><code>v-if</code>와 <code>v-else</code>: 기온 라벨 분기</li>
<li><code>:class</code>: 더움/선선함 스타일 변경</li>
<li><code>:value</code>, <code>@input</code>: 한글 검색어 입력</li>
<li><code>@click</code>: 카드 선택</li>
<li><code>@click.stop</code>: 상세보기 버튼의 버블링 방지</li>
<li><code>{{ }}</code>: 상태 출력</li>
</ul>
<h2 id="91-완성-예제">9.1 완성 예제</h2>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const weatherList = ref([
  { id: 'city_01', name: '서울', temp: 28, status: '맑음' },
  { id: 'city_02', name: '수원', temp: 24, status: '비' },
  { id: 'city_03', name: '부산', temp: 26, status: '구름' },
])

const searchKeyword = ref('')
const selectedMessage = ref('카드를 클릭하거나 도시를 검색해 보세요.')

const handleSearchInput = (event) =&gt; {
  searchKeyword.value = event.target.value
}

const selectCity = (cityName) =&gt; {
  selectedMessage.value = `${cityName}이 선택되었습니다.`
}

const showDetail = (cityName, status) =&gt; {
  window.alert(`${cityName}의 현재 날씨는 [${status}] 상태입니다.`)
}
&lt;/script&gt;

&lt;template&gt;
  &lt;main class=&quot;weather-dashboard&quot;&gt;
    &lt;h1&gt;🌤️ 과제 1: 날씨 Mockup&lt;/h1&gt;

    &lt;section class=&quot;panel&quot;&gt;
      &lt;label for=&quot;city-search&quot;&gt;🔍 도시 검색&lt;/label&gt;
      &lt;input
        id=&quot;city-search&quot;
        :value=&quot;searchKeyword&quot;
        @input=&quot;handleSearchInput&quot;
        placeholder=&quot;검색할 도시 이름 입력&quot;
      /&gt;
      &lt;p&gt;검색 중인 도시: {{ searchKeyword || '없음' }}&lt;/p&gt;
    &lt;/section&gt;

    &lt;section class=&quot;panel&quot;&gt;
      &lt;h2&gt;🏙️ 지역별 날씨 현황&lt;/h2&gt;

      &lt;article
        v-for=&quot;city in weatherList&quot;
        :key=&quot;city.id&quot;
        class=&quot;weather-card&quot;
        @click=&quot;selectCity(city.name)&quot;
      &gt;
        &lt;div&gt;
          &lt;strong&gt;{{ city.name }} ({{ city.status }})&lt;/strong&gt;
          &lt;p&gt;현재 기온: {{ city.temp }}°C&lt;/p&gt;

          &lt;span v-if=&quot;city.temp &gt;= 25&quot; class=&quot;badge hot&quot;&gt;
            🔥 더움 (25°C 이상)
          &lt;/span&gt;
          &lt;span v-else class=&quot;badge cool&quot;&gt;
            ❄️ 선선함 (25°C 미만)
          &lt;/span&gt;
        &lt;/div&gt;

        &lt;button @click.stop=&quot;showDetail(city.name, city.status)&quot;&gt;
          상세보기
        &lt;/button&gt;
      &lt;/article&gt;
    &lt;/section&gt;

    &lt;p class=&quot;status-bar&quot;&gt;{{ selectedMessage }}&lt;/p&gt;
  &lt;/main&gt;
&lt;/template&gt;

&lt;style scoped&gt;
.weather-dashboard {
  width: min(100%, 720px);
  margin: 40px auto;
  padding: 24px;
  color: #1f2937;
}

.panel {
  margin-bottom: 16px;
  padding: 20px;
  border: 1px solid #dbe2ea;
  border-radius: 12px;
  background: #f8fafc;
}

input {
  box-sizing: border-box;
  width: 100%;
  margin-top: 8px;
  padding: 10px 12px;
}

.weather-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 12px;
  padding: 16px;
  border: 1px solid #dbe2ea;
  border-radius: 10px;
  background: white;
  cursor: pointer;
}

.badge {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 6px;
  color: white;
  font-size: 13px;
}

.hot {
  background: #ff6b6b;
}

.cool {
  background: #4dabf7;
}

.status-bar {
  padding: 12px;
  border-radius: 8px;
  background: #e8f5e9;
  text-align: center;
}
&lt;/style&gt;</code></pre>
<h2 id="92-이벤트-버블링을-눈으로-확인하기">9.2 이벤트 버블링을 눈으로 확인하기</h2>
<p>카드와 카드 안쪽 버튼에는 모두 click 이벤트가 있습니다.</p>
<pre><code class="language-text">상세보기 버튼 클릭
        ↓
버튼 click 실행
        ↓
부모 article로 이벤트 전파
        ↓
카드 click도 실행</code></pre>
<p>버튼의 <code>@click.stop</code>에서 <code>.stop</code>을 제거해 보면 alert와 카드 선택이 함께 실행됩니다. 다시 <code>.stop</code>을 붙이면 상세보기만 실행됩니다. 수식어의 필요성을 가장 잘 체감할 수 있는 실험입니다.</p>
<h2 id="93-실습-확장-과제">9.3 실습 확장 과제</h2>
<h3 id="level-1-빈-검색어-처리">Level 1: 빈 검색어 처리</h3>
<p>검색어가 없으면 <code>없음</code>, 있으면 입력값을 표시해 보세요.</p>
<pre><code class="language-vue">{{ searchKeyword || '없음' }}</code></pre>
<h3 id="level-2-강수-상태-표시">Level 2: 강수 상태 표시</h3>
<p><code>status === '비'</code>인 카드에 우산 아이콘을 추가해 보세요.</p>
<pre><code class="language-vue">&lt;span v-if=&quot;city.status === '비'&quot;&gt;☔ 우산을 챙기세요.&lt;/span&gt;</code></pre>
<h3 id="level-3-카드-강조">Level 3: 카드 강조</h3>
<p>선택된 도시 id를 별도 ref로 저장하고 선택된 카드에 클래스를 적용해 보세요.</p>
<pre><code class="language-js">const selectedCityId = ref('')</code></pre>
<pre><code class="language-vue">&lt;article :class=&quot;{ selected: selectedCityId === city.id }&quot;&gt;</code></pre>
<h3 id="level-4-목록-추가">Level 4: 목록 추가</h3>
<p>폼에서 도시 이름, 기온, 날씨를 입력받아 <code>weatherList.value.push(...)</code>로 새 카드를 추가해 보세요. 이때 각 항목의 <code>id</code>는 반드시 고유해야 합니다.</p>
<hr />
<h1 id="10-초심자가-자주-하는-실수">10. 초심자가 자주 하는 실수</h1>
<h2 id="실수-1-일반-변수를-바꾸면-화면도-바뀔-것이라-생각한다">실수 1: 일반 변수를 바꾸면 화면도 바뀔 것이라 생각한다</h2>
<pre><code class="language-js">let count = 0</code></pre>
<p>화면과 연결할 상태라면 <code>ref</code>를 사용합니다.</p>
<pre><code class="language-js">const count = ref(0)</code></pre>
<h2 id="실수-2-script에서-value를-빠뜨린다">실수 2: script에서 <code>.value</code>를 빠뜨린다</h2>
<pre><code class="language-js">count++        // 잘못된 접근
count.value++  // 올바른 접근</code></pre>
<p>템플릿에서는 <code>{{ count }}</code>처럼 <code>.value</code>를 생략합니다.</p>
<h2 id="실수-3-속성에-보간법을-넣는다">실수 3: 속성에 보간법을 넣는다</h2>
<pre><code class="language-vue">&lt;img src=&quot;{{ imageUrl }}&quot; /&gt;</code></pre>
<p>속성은 <code>v-bind</code>를 사용합니다.</p>
<pre><code class="language-vue">&lt;img :src=&quot;imageUrl&quot; /&gt;</code></pre>
<h2 id="실수-4-v-html에-사용자-입력을-넣는다">실수 4: <code>v-html</code>에 사용자 입력을 넣는다</h2>
<p>신뢰할 수 없는 HTML은 XSS 공격의 통로가 될 수 있습니다. 일반 텍스트 출력에는 보간법을 사용하세요.</p>
<h2 id="실수-5-v-for에서-key를-생략하거나-index만-쓴다">실수 5: <code>v-for</code>에서 key를 생략하거나 index만 쓴다</h2>
<p>가능하면 서버나 데이터가 제공하는 고유 id를 사용합니다.</p>
<pre><code class="language-vue">&lt;li v-for=&quot;city in weatherList&quot; :key=&quot;city.id&quot;&gt;</code></pre>
<h2 id="실수-6-v-if와-v-show를-무조건-같은-것으로-본다">실수 6: <code>v-if</code>와 <code>v-show</code>를 무조건 같은 것으로 본다</h2>
<ul>
<li><code>v-if</code>: DOM 생성·제거</li>
<li><code>v-show</code>: CSS 표시·숨김</li>
</ul>
<p>전환 빈도와 요소 존재 여부를 기준으로 선택합니다.</p>
<h2 id="실수-7-중첩된-클릭-이벤트의-버블링을-놓친다">실수 7: 중첩된 클릭 이벤트의 버블링을 놓친다</h2>
<p>카드 안쪽 버튼이 카드 클릭까지 실행한다면 <code>@click.stop</code>을 검토합니다.</p>
<h2 id="실수-8-폼-상태의-초기-타입을-잘못-정한다">실수 8: 폼 상태의 초기 타입을 잘못 정한다</h2>
<p>다중 checkbox는 <code>ref([])</code>, 단일 checkbox는 <code>ref(false)</code>로 시작합니다.</p>
<hr />
<h1 id="11최종-점검">11.최종 점검</h1>
<p>다음 질문에 스스로 답해 보세요.</p>
<ol>
<li>일반 변수와 <code>ref</code>의 가장 큰 차이는 무엇인가?</li>
<li><code>&lt;script setup&gt;</code>과 <code>&lt;template&gt;</code>에서 ref에 접근하는 방식은 어떻게 다른가?</li>
<li><code>{{ }}</code>, <code>:</code>, <code>@</code>는 각각 무엇을 연결하는가?</li>
<li><code>v-if</code>와 <code>v-show</code>는 DOM을 어떻게 다르게 처리하는가?</li>
<li><code>v-for</code>의 <code>key</code>에 배열 index보다 id가 좋은 이유는 무엇인가?</li>
<li><code>v-html</code>에 외부 데이터를 넣을 때 어떤 위험이 있는가?</li>
<li>카드 안의 버튼 클릭이 카드 클릭으로 전파되지 않게 하려면 어떻게 하는가?</li>
<li>다중 checkbox의 상태를 왜 배열로 선언해야 하는가?</li>
</ol>
<h2 id="한-문장으로-정리">한 문장으로 정리</h2>
<blockquote>
<p>Vue 개발은 DOM을 직접 조작하는 일이 아니라, 반응형 상태를 설계하고 그 상태가 어떤 화면이 되어야 하는지 선언하는 일입니다.</p>
</blockquote>
<p>오늘 배운 <code>ref</code>, 보간법, 디렉티브, 이벤트, <code>v-model</code>은 이후 Composition API, 컴포넌트 통신, Router, Pinia, Axios를 배우기 위한 기반입니다. 각 문법을 따로 외우기보다 모두가 “상태와 화면을 연결하는 방법”이라는 점을 기억하세요.</p>