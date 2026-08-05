<blockquote>
<p>오늘은 Vue 앱을 “컴포넌트 몇 개가 있는 화면”에서 한 단계 발전시켜, <strong>여러 페이지를 이동하고</strong>, <strong>공통 데이터를 공유하며</strong>, <strong>서버에서 실제 데이터를 받아오는 앱</strong>으로 만드는 날입니다.  </p>
</blockquote>
<hr />
<h2 id="오늘-배울-세-기술의-관계">오늘 배울 세 기술의 관계</h2>
<p>세 기술은 따로 외우기보다 역할을 나누어 이해하면 쉽습니다.</p>
<pre><code class="language-text">사용자가 링크 클릭
        ↓
Vue Router: URL 변경 + 화면 전환
        ↓
Pinia: 여러 화면이 함께 쓰는 상태 공유
        ↓
Axios: 서버 API에 요청해서 실제 데이터 받기</code></pre>
<p>날씨 서비스로 예를 들면 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>상황</th>
<th>담당 기술</th>
</tr>
</thead>
<tbody><tr>
<td>서울 날씨 카드 클릭 후 상세 페이지로 이동</td>
<td>Vue Router</td>
</tr>
<tr>
<td>섭씨/화씨 설정을 모든 화면에 적용</td>
<td>Pinia</td>
</tr>
<tr>
<td>OpenWeather API에서 실제 날씨 가져오기</td>
<td>Axios</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-vue-router---url에-따라-화면-바꾸기">1. Vue Router - URL에 따라 화면 바꾸기</h1>
<h2 id="spa에서-페이지-이동은-어떻게-동작할까">SPA에서 페이지 이동은 어떻게 동작할까?</h2>
<p>전통적인 웹사이트는 메뉴를 클릭할 때마다 서버에 새로운 HTML 파일을 요청하고, 브라우저 화면 전체를 새로고침합니다.</p>
<p>하지만 Vue는 보통 SPA(Single Page Application)로 동작합니다.</p>
<ul>
<li>처음에는 <code>index.html</code> 하나를 불러옵니다.</li>
<li>이후 URL이 바뀌면 서버에서 새 HTML을 받지 않습니다.</li>
<li>Vue Router가 URL을 읽고, 해당 위치에 맞는 Vue 컴포넌트만 교체합니다.</li>
</ul>
<p>즉, Vue Router는 다음 규칙을 담당합니다.</p>
<blockquote>
<p>“이 URL에 접속하면, 이 Vue 화면을 보여줘.”</p>
</blockquote>
<hr />
<h2 id="router-route-routerview-routerlink-구분하기">Router, Route, RouterView, RouterLink 구분하기</h2>
<p>처음에는 이름이 비슷해서 헷갈리기 쉽습니다.</p>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 비유</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>router</code></td>
<td>내비게이션 시스템 전체</td>
<td>URL과 화면을 연결하는 관리자</td>
</tr>
<tr>
<td><code>route</code></td>
<td>현재 주소 정보</td>
<td>지금 어느 페이지에 있는지 알려주는 객체</td>
</tr>
<tr>
<td><code>RouterLink</code></td>
<td>SPA 전용 메뉴 링크</td>
<td>새로고침 없이 주소를 변경</td>
</tr>
<tr>
<td><code>RouterView</code></td>
<td>화면이 끼워지는 자리</td>
<td>URL에 맞는 페이지 컴포넌트 출력</td>
</tr>
</tbody></table>
<pre><code class="language-text">RouterLink 클릭
     ↓
URL 변경
     ↓
router가 경로 확인
     ↓
RouterView 위치에 알맞은 View 출력</code></pre>
<hr />
<h2 id="router-설정하기">Router 설정하기</h2>
<p>라우터 규칙은 보통 <code>src/router/index.js</code>에 작성합니다.</p>
<pre><code class="language-js">import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'WeatherHome',
      component: () =&gt; import('@/views/WeatherHomeView.vue'),
    },
    {
      path: '/about',
      name: 'WeatherAbout',
      component: () =&gt; import('@/views/WeatherAboutView.vue'),
    },
  ],
})

export default router</code></pre>
<p>각 <code>route</code> 객체의 핵심 속성은 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>속성</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>path</code></td>
<td>브라우저 URL 경로</td>
<td><code>'/about'</code></td>
</tr>
<tr>
<td><code>name</code></td>
<td>라우트의 고유 이름</td>
<td><code>'WeatherAbout'</code></td>
</tr>
<tr>
<td><code>component</code></td>
<td>해당 경로에서 보여줄 컴포넌트</td>
<td><code>WeatherAboutView</code></td>
</tr>
<tr>
<td><code>redirect</code></td>
<td>다른 주소로 강제 이동</td>
<td><code>'/'</code></td>
</tr>
</tbody></table>
<hr />
<h2 id="정적-import와-lazy-loading">정적 import와 Lazy Loading</h2>
<p>다음 두 코드는 비슷해 보이지만 실행 시점이 다릅니다.</p>
<pre><code class="language-js">import WeatherAboutView from '@/views/WeatherAboutView.vue'</code></pre>
<pre><code class="language-js">component: () =&gt; import('@/views/WeatherAboutView.vue')</code></pre>
<p>첫 번째는 앱이 시작할 때 파일을 바로 불러옵니다.</p>
<p>두 번째는 사용자가 해당 주소에 접근할 때 파일을 불러옵니다. 이것을 <strong>Lazy Loading(지연 로딩)</strong>이라고 합니다.</p>
<pre><code class="language-js">{
  path: '/about',
  component: () =&gt; import('@/views/WeatherAboutView.vue'),
}</code></pre>
<p>페이지가 많아질수록 모든 화면을 한꺼번에 불러오면 첫 로딩이 느려질 수 있습니다. 그래서 페이지 단위 컴포넌트에는 지연 로딩을 자주 사용합니다.</p>
<hr />
<h2 id="mainjs에-router-등록하기"><code>main.js</code>에 Router 등록하기</h2>
<p>라우터 규칙을 만들었다면 Vue 앱에 등록해야 합니다.</p>
<pre><code class="language-js">import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(router)

app.mount('#app')</code></pre>
<p><code>app.use(router)</code>는 다음 뜻입니다.</p>
<blockquote>
<p>“이 Vue 앱에서 Router 기능을 사용할게.”</p>
</blockquote>
<hr />
<h2 id="routerlink와-routerview"><code>RouterLink</code>와 <code>RouterView</code></h2>
<p>보통 <code>App.vue</code>에 메뉴와 화면 출력 영역을 작성합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { RouterLink, RouterView } from 'vue-router'
&lt;/script&gt;

&lt;template&gt;
  &lt;header&gt;
    &lt;nav&gt;
      &lt;RouterLink to=&quot;/&quot;&gt;날씨 홈&lt;/RouterLink&gt;
      &lt;RouterLink to=&quot;/about&quot;&gt;서비스 소개&lt;/RouterLink&gt;
    &lt;/nav&gt;
  &lt;/header&gt;

  &lt;main&gt;
    &lt;RouterView /&gt;
  &lt;/main&gt;
&lt;/template&gt;</code></pre>
<p><code>RouterLink</code>는 일반 <code>&lt;a&gt;</code> 태그와 달리 SPA의 반응형 상태를 유지하면서 이동합니다.</p>
<pre><code class="language-vue">&lt;!-- 권장 --&gt;
&lt;RouterLink to=&quot;/about&quot;&gt;서비스 소개&lt;/RouterLink&gt;

&lt;!-- SPA 내부 이동에는 권장하지 않음 --&gt;
&lt;a href=&quot;/about&quot;&gt;서비스 소개&lt;/a&gt;</code></pre>
<p>일반 <code>&lt;a&gt;</code> 태그는 페이지 전체를 새로고침할 수 있습니다. 그러면 <code>ref</code>, <code>computed</code>, Pinia 상태처럼 메모리에 있던 데이터가 초기화될 수 있습니다.</p>
<hr />
<h2 id="views와-components의-차이"><code>views</code>와 <code>components</code>의 차이</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th><code>views/</code></th>
<th><code>components/</code></th>
</tr>
</thead>
<tbody><tr>
<td>역할</td>
<td>페이지 단위 화면</td>
<td>재사용 가능한 작은 부품</td>
</tr>
<tr>
<td>Router 연결</td>
<td><code>RouterView</code>에 직접 출력</td>
<td>보통 View 내부에서 사용</td>
</tr>
<tr>
<td>예시</td>
<td><code>WeatherHomeView.vue</code></td>
<td><code>SearchBar.vue</code></td>
</tr>
<tr>
<td>재사용성</td>
<td>비교적 낮음</td>
<td>높음</td>
</tr>
</tbody></table>
<p>날씨 프로젝트 구조는 다음처럼 나누면 좋습니다.</p>
<pre><code class="language-text">src/
├── components/
│   └── exercise/
│       ├── BaseDashboardCard.vue
│       ├── SearchBar.vue
│       └── WeatherCard.vue
├── router/
│   └── index.js
└── views/
    ├── WeatherHomeView.vue
    ├── WeatherDetailView.vue
    ├── WeatherAboutView.vue
    └── NotFoundView.vue</code></pre>
<hr />
<h2 id="동적-라우트---도시마다-페이지를-만들지-않는-방법">동적 라우트 - 도시마다 페이지를 만들지 않는 방법</h2>
<p>서울, 부산, 제주마다 별도 페이지 파일을 만들면 비효율적입니다.</p>
<pre><code class="language-text">/weather/seoul
/weather/busan
/weather/jeju</code></pre>
<p>이럴 때 URL 일부를 변수처럼 사용합니다.</p>
<pre><code class="language-js">{
  path: '/weather/:cityId',
  name: 'WeatherDetail',
  component: () =&gt; import('@/views/WeatherDetailView.vue'),
}</code></pre>
<p>여기서 <code>:cityId</code>가 동적 세그먼트입니다.</p>
<pre><code class="language-text">/weather/region_01
/weather/region_17</code></pre>
<p><code>WeatherDetailView.vue</code>에서는 <code>useRoute()</code>로 값을 받을 수 있습니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { useRoute } from 'vue-router'

const route = useRoute()

console.log(route.params.cityId)
&lt;/script&gt;

&lt;template&gt;
  &lt;h1&gt;{{ route.params.cityId }} 지역 상세 날씨&lt;/h1&gt;
&lt;/template&gt;</code></pre>
<hr />
<h2 id="useroute와-userouter는-다르다"><code>useRoute()</code>와 <code>useRouter()</code>는 다르다</h2>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>useRoute()</code></td>
<td>현재 주소 정보를 읽기</td>
</tr>
<tr>
<td><code>useRouter()</code></td>
<td>JavaScript 코드로 페이지 이동하기</td>
</tr>
</tbody></table>
<h3 id="현재-주소-읽기-useroute">현재 주소 읽기: <code>useRoute()</code></h3>
<pre><code class="language-js">import { useRoute } from 'vue-router'

const route = useRoute()

console.log(route.path)
console.log(route.params.cityId)
console.log(route.query.search)</code></pre>
<p>예를 들어 다음 URL이라면,</p>
<pre><code class="language-text">/weather/region_01?search=서울</code></pre>
<pre><code class="language-js">route.params.cityId // 'region_01'
route.query.search // '서울'</code></pre>
<h3 id="코드로-페이지-이동하기-userouter">코드로 페이지 이동하기: <code>useRouter()</code></h3>
<p>날씨 카드의 “상세보기” 버튼을 클릭했을 때 이동시키는 경우입니다.</p>
<pre><code class="language-js">import { useRouter } from 'vue-router'

const router = useRouter()

const showDetail = (item) =&gt; {
  router.push({
    name: 'WeatherDetail',
    params: {
      cityId: item.id,
    },
  })
}</code></pre>
<p>자주 사용하는 이동 방식은 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>특징</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>router.push()</code></td>
<td>이동 기록이 남음</td>
<td>상세 화면 이동</td>
</tr>
<tr>
<td><code>router.replace()</code></td>
<td>이전 기록을 교체</td>
<td>로그인 후 홈으로 이동</td>
</tr>
<tr>
<td><code>router.back()</code></td>
<td>이전 화면으로 이동</td>
<td>뒤로 가기 버튼</td>
</tr>
<tr>
<td><code>router.go(-1)</code></td>
<td>히스토리 1단계 뒤로</td>
<td>뒤로 가기</td>
</tr>
</tbody></table>
<p>로그인 만료처럼 “뒤로 가기”로 다시 접근하면 곤란한 상황에는 <code>replace()</code>가 적합합니다.</p>
<hr />
<h2 id="존재하지-않는-주소-처리하기---404-페이지">존재하지 않는 주소 처리하기 - 404 페이지</h2>
<p>사용자가 없는 주소로 접근해도 빈 화면을 보여주면 안 됩니다.</p>
<pre><code class="language-js">{
  path: '/:pathMatch(.*)*',
  name: 'NotFound',
  component: () =&gt; import('@/views/NotFoundView.vue'),
}</code></pre>
<p>이 경로는 어떤 라우트에도 맞지 않는 주소를 처리합니다.</p>
<p>중요한 점은 <strong>항상 routes 배열의 마지막에 작성해야 한다는 것</strong>입니다.</p>
<hr />
<h1 id="2-pinia---여러-화면이-함께-쓰는-데이터-저장소">2. Pinia - 여러 화면이 함께 쓰는 데이터 저장소</h1>
<h2 id="props만으로-데이터를-전달하면-생기는-문제">Props만으로 데이터를 전달하면 생기는 문제</h2>
<p>부모에서 자식으로 데이터 하나를 전달하는 것은 <code>props</code>로 충분합니다.</p>
<pre><code class="language-text">WeatherHomeView
  └── WeatherCard</code></pre>
<p>하지만 컴포넌트 구조가 깊어지면 문제가 생깁니다.</p>
<pre><code class="language-text">App
  └── WeatherHomeView
       └── DashboardLayout
            └── WeatherCard</code></pre>
<p>최하위 <code>WeatherCard</code>만 필요한 데이터를 중간 컴포넌트들이 계속 전달해야 할 수 있습니다. 이것을 <strong>Props Drilling</strong>이라고 합니다.</p>
<hr />
<h2 id="pinia란">Pinia란?</h2>
<p>Pinia는 컴포넌트 구조 밖에 <strong>공용 반응형 데이터 저장소(Store)</strong>를 만드는 라이브러리입니다.</p>
<pre><code class="language-text">WeatherHomeView ─┐
WeatherDetailView ├── Pinia Store
NavigationBar ────┘</code></pre>
<p>어느 컴포넌트든 필요한 Store를 가져와 데이터를 읽거나 변경할 수 있습니다.</p>
<hr />
<h2 id="pinia의-핵심-state-getters-actions">Pinia의 핵심: State, Getters, Actions</h2>
<table>
<thead>
<tr>
<th>Pinia 개념</th>
<th>Vue 문법과 연결</th>
<th>쉬운 설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>state</code></td>
<td><code>ref</code>, <code>reactive</code></td>
<td>원본 데이터</td>
</tr>
<tr>
<td><code>getters</code></td>
<td><code>computed</code></td>
<td>원본 데이터를 가공한 읽기 전용 값</td>
</tr>
<tr>
<td><code>actions</code></td>
<td>함수</td>
<td>상태 변경, API 호출 등의 행동</td>
</tr>
</tbody></table>
<p>예를 들어 온도 단위 설정을 Store로 만들 수 있습니다.</p>
<pre><code class="language-text">state   : 현재 온도 단위
getter  : 화면에 보여줄 기호
action  : 섭씨/화씨 전환</code></pre>
<hr />
<h2 id="pinia-등록하기">Pinia 등록하기</h2>
<p><code>main.js</code>에 Pinia를 등록합니다.</p>
<pre><code class="language-js">import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

app.use(createPinia())

app.mount('#app')</code></pre>
<hr />
<h2 id="store-만들기---온도-단위-예제">Store 만들기 - 온도 단위 예제</h2>
<p><code>src/stores/configStore.js</code> 파일을 만듭니다.</p>
<pre><code class="language-js">import { computed, ref } from 'vue'
import { defineStore } from 'pinia'

export const useConfigStore = defineStore('config', () =&gt; {
  // State: 원본 상태
  const unit = ref('celsius')

  // Getter: 상태를 기반으로 계산한 값
  const unitSymbol = computed(() =&gt; {
    return unit.value === 'celsius' ? '°C' : '°F'
  })

  // Action: 상태를 바꾸는 함수
  const toggleUnit = () =&gt; {
    unit.value = unit.value === 'celsius' ? 'fahrenheit' : 'celsius'
  }

  return {
    unit,
    unitSymbol,
    toggleUnit,
  }
})</code></pre>
<p>이 Store는 전역에서 사용할 수 있는 작은 설정 상자입니다.</p>
<pre><code class="language-text">configStore
├── unit: 'celsius'
├── unitSymbol: '°C'
└── toggleUnit()</code></pre>
<hr />
<h2 id="컴포넌트에서-store-사용하기">컴포넌트에서 Store 사용하기</h2>
<pre><code class="language-vue">&lt;script setup&gt;
import { useConfigStore } from '@/stores/configStore'

const configStore = useConfigStore()
&lt;/script&gt;

&lt;template&gt;
  &lt;button @click=&quot;configStore.toggleUnit&quot;&gt;
    현재 단위: {{ configStore.unitSymbol }}
  &lt;/button&gt;
&lt;/template&gt;</code></pre>
<p>버튼을 클릭하면 <code>unit</code>이 바뀌고, 이 Store를 사용하는 모든 컴포넌트가 자동으로 다시 렌더링됩니다.</p>
<hr />
<h2 id="섭씨를-화씨로-변환하기">섭씨를 화씨로 변환하기</h2>
<pre><code class="language-vue">&lt;script setup&gt;
import { computed } from 'vue'
import { useConfigStore } from '@/stores/configStore'

const props = defineProps({
  temp: {
    type: Number,
    required: true,
  },
})

const configStore = useConfigStore()

const displayTemp = computed(() =&gt; {
  if (configStore.unit === 'fahrenheit') {
    return Math.round((props.temp * 9) / 5 + 32)
  }

  return props.temp
})
&lt;/script&gt;

&lt;template&gt;
  &lt;p&gt;{{ displayTemp }}{{ configStore.unitSymbol }}&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p>예를 들어 원본이 <code>25°C</code>라면 다음처럼 표시됩니다.</p>
<pre><code class="language-text">섭씨 모드: 25°C
화씨 모드: 77°F</code></pre>
<p>중요한 원칙은 다음입니다.</p>
<blockquote>
<p>원본 날씨 데이터는 섭씨로 유지하고, 화면에 보여줄 때만 계산해서 변환한다.</p>
</blockquote>
<p>원본 데이터를 화씨로 직접 바꾸면 다시 섭씨로 되돌릴 때 관리가 복잡해집니다.</p>
<hr />
<h2 id="pinia를-사용하면-좋은-데이터">Pinia를 사용하면 좋은 데이터</h2>
<table>
<thead>
<tr>
<th>Pinia에 어울리는 상태</th>
<th>컴포넌트 내부 <code>ref</code>에 어울리는 상태</th>
</tr>
</thead>
<tbody><tr>
<td>로그인 사용자 정보</td>
<td>검색창에 입력 중인 글자</td>
</tr>
<tr>
<td>로그인 토큰</td>
<td>버튼 hover 상태</td>
</tr>
<tr>
<td>다크 모드</td>
<td>모달 열림 여부</td>
</tr>
<tr>
<td>언어 설정</td>
<td>한 컴포넌트의 임시 선택값</td>
</tr>
<tr>
<td>섭씨/화씨 단위</td>
<td>카드 하나의 접힘 상태</td>
</tr>
</tbody></table>
<p>기준은 간단합니다.</p>
<blockquote>
<p>여러 화면 또는 여러 컴포넌트가 함께 써야 하면 Pinia를 고려한다.</p>
</blockquote>
<hr />
<h1 id="3-axios---서버에서-실제-데이터-가져오기">3. Axios - 서버에서 실제 데이터 가져오기</h1>
<h2 id="http란">HTTP란?</h2>
<p>HTTP는 브라우저와 서버가 데이터를 주고받기 위한 약속입니다.</p>
<pre><code class="language-text">Vue 화면(클라이언트)
        ↓ 요청(Request)
API 서버
        ↓ 응답(Response)
Vue 화면에 데이터 출력</code></pre>
<p>예를 들어 날씨 앱은 서버에 다음처럼 요청합니다.</p>
<pre><code class="language-text">“서울의 현재 날씨 데이터를 보내줘.”</code></pre>
<p>서버는 보통 JSON 형식으로 응답합니다.</p>
<pre><code class="language-json">{
  &quot;name&quot;: &quot;Seoul&quot;,
  &quot;main&quot;: {
    &quot;temp&quot;: 28,
    &quot;humidity&quot;: 65
  }
}</code></pre>
<hr />
<h2 id="http-메서드와-crud">HTTP 메서드와 CRUD</h2>
<table>
<thead>
<tr>
<th>HTTP 메서드</th>
<th>CRUD</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>GET</code></td>
<td>Read</td>
<td>데이터 조회</td>
</tr>
<tr>
<td><code>POST</code></td>
<td>Create</td>
<td>새 데이터 생성</td>
</tr>
<tr>
<td><code>PUT</code> / <code>PATCH</code></td>
<td>Update</td>
<td>데이터 수정</td>
</tr>
<tr>
<td><code>DELETE</code></td>
<td>Delete</td>
<td>데이터 삭제</td>
</tr>
</tbody></table>
<p>예를 들어 게시글 API라면 다음처럼 생각할 수 있습니다.</p>
<pre><code class="language-text">GET    /posts      → 게시글 목록 조회
POST   /posts      → 게시글 작성
PATCH  /posts/1    → 1번 게시글 일부 수정
DELETE /posts/1    → 1번 게시글 삭제</code></pre>
<p>REST API 주소는 자원을 나타내는 명사 형태가 좋습니다.</p>
<pre><code class="language-text">좋지 않은 예: /getWeather
좋은 예:     /weather</code></pre>
<p>무엇을 할지는 URL이 아니라 HTTP 메서드가 표현합니다.</p>
<hr />
<h2 id="axios-설치하기">Axios 설치하기</h2>
<pre><code class="language-bash">npm install axios</code></pre>
<p>설치 후 <code>package.json</code>의 dependencies에 Axios가 추가됩니다.</p>
<hr />
<h2 id="axios와-fetch-비교">Axios와 Fetch 비교</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>Fetch</th>
<th>Axios</th>
</tr>
</thead>
<tbody><tr>
<td>설치</td>
<td>필요 없음</td>
<td><code>npm install axios</code> 필요</td>
</tr>
<tr>
<td>JSON 변환</td>
<td><code>response.json()</code> 필요</td>
<td><code>response.data</code>에 자동 변환</td>
</tr>
<tr>
<td>공통 URL 설정</td>
<td>직접 구현</td>
<td><code>axios.create()</code> 지원</td>
</tr>
<tr>
<td>인터셉터</td>
<td>직접 구현</td>
<td>요청/응답 인터셉터 지원</td>
</tr>
<tr>
<td>실무 활용</td>
<td>가능</td>
<td>많이 사용됨</td>
</tr>
</tbody></table>
<hr />
<h2 id="가장-기본적인-axios-요청">가장 기본적인 Axios 요청</h2>
<p>외부 API 호출에는 시간이 걸립니다. 따라서 <code>async</code>와 <code>await</code>를 사용합니다.</p>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'
import axios from 'axios'

const weatherData = ref(null)
const isLoading = ref(false)
const errorMessage = ref('')

const fetchWeather = async () =&gt; {
  isLoading.value = true
  errorMessage.value = ''

  try {
    const response = await axios.get(
      'https://jsonplaceholder.typicode.com/posts/1',
    )

    weatherData.value = response.data
  } catch (error) {
    console.error('통신 실패:', error)
    errorMessage.value = '데이터를 불러오지 못했습니다.'
  } finally {
    isLoading.value = false
  }
}
&lt;/script&gt;

&lt;template&gt;
  &lt;button :disabled=&quot;isLoading&quot; @click=&quot;fetchWeather&quot;&gt;
    {{ isLoading ? '불러오는 중...' : '데이터 불러오기' }}
  &lt;/button&gt;

  &lt;p v-if=&quot;errorMessage&quot;&gt;{{ errorMessage }}&lt;/p&gt;

  &lt;pre v-else-if=&quot;weatherData&quot;&gt;{{ weatherData }}&lt;/pre&gt;

  &lt;p v-else&gt;아직 가져온 데이터가 없습니다.&lt;/p&gt;
&lt;/template&gt;</code></pre>
<p>이 코드에서 상태별 역할은 명확합니다.</p>
<table>
<thead>
<tr>
<th>상태</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>weatherData</code></td>
<td>서버에서 받은 결과</td>
</tr>
<tr>
<td><code>isLoading</code></td>
<td>요청 진행 여부</td>
</tr>
<tr>
<td><code>errorMessage</code></td>
<td>오류 메시지</td>
</tr>
<tr>
<td><code>try</code></td>
<td>통신 성공 시 실행</td>
</tr>
<tr>
<td><code>catch</code></td>
<td>통신 실패 시 실행</td>
</tr>
<tr>
<td><code>finally</code></td>
<td>성공·실패와 관계없이 마지막에 실행</td>
</tr>
</tbody></table>
<hr />
<h2 id="왜-finally가-필요할까">왜 <code>finally</code>가 필요할까?</h2>
<p>로딩 상태는 성공했을 때도, 실패했을 때도 종료되어야 합니다.</p>
<pre><code class="language-js">isLoading.value = true

try {
  // API 요청
} catch (error) {
  // 오류 처리
} finally {
  isLoading.value = false
}</code></pre>
<p><code>finally</code>가 없다면 오류가 났을 때 “로딩 중…” 버튼이 계속 남을 수 있습니다.</p>
<hr />
<h2 id="axios-요청-메서드">Axios 요청 메서드</h2>
<pre><code class="language-js">axios.get('/weather')

axios.post('/weather', {
  city: 'Seoul',
})

axios.patch('/weather/1', {
  temp: 28,
})

axios.delete('/weather/1')</code></pre>
<p>쿼리 스트링은 <code>params</code>로 전달할 수 있습니다.</p>
<pre><code class="language-js">axios.get('/weather', {
  params: {
    city: 'Seoul',
    lang: 'kr',
  },
})</code></pre>
<p>위 요청은 서버에 따라 다음과 비슷한 URL로 전달됩니다.</p>
<pre><code class="language-text">/weather?city=Seoul&amp;lang=kr</code></pre>
<hr />
<h2 id="axioscreate로-공통-주소-관리하기"><code>axios.create()</code>로 공통 주소 관리하기</h2>
<p>여러 요청에서 같은 API 주소를 반복하면 관리하기 어렵습니다.</p>
<pre><code class="language-js">const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
})</code></pre>
<p>이후에는 짧게 요청할 수 있습니다.</p>
<pre><code class="language-js">const response = await api.get('/weather')</code></pre>
<p>실제 요청 주소는 다음과 같습니다.</p>
<pre><code class="language-text">https://api.example.com/weather</code></pre>
<hr />
<h2 id="api-키-보안-주의사항">API 키 보안 주의사항</h2>
<p>강의 예제처럼 프론트엔드 코드에 API 키를 직접 작성하면, 브라우저 개발자 도구나 빌드 파일을 통해 노출될 수 있습니다.</p>
<pre><code class="language-js">const API_KEY = '비밀키'</code></pre>
<p>학습용 테스트에서는 사용할 수 있지만, 실제 서비스에서는 다음을 기억해야 합니다.</p>
<ul>
<li>프론트엔드의 환경 변수도 완전한 비밀 공간은 아닙니다.</li>
<li>중요한 비밀 키는 백엔드 서버에서 관리합니다.</li>
<li>프론트엔드는 백엔드 API를 거쳐 필요한 데이터만 받는 구조가 안전합니다.</li>
</ul>
<hr />
<h1 id="오늘의-핵심-요약">오늘의 핵심 요약</h1>
<table>
<thead>
<tr>
<th>기술</th>
<th>한 문장 요약</th>
</tr>
</thead>
<tbody><tr>
<td>Vue Router</td>
<td>URL에 맞는 화면을 새로고침 없이 보여준다.</td>
</tr>
<tr>
<td>Pinia</td>
<td>여러 컴포넌트가 함께 써야 하는 상태를 중앙에서 관리한다.</td>
</tr>
<tr>
<td>Axios</td>
<td>Vue 앱이 서버 API와 HTTP 통신하도록 돕는다.</td>
</tr>
</tbody></table>
<p>오늘의 가장 중요한 흐름은 이것입니다.</p>
<blockquote>
<p><strong>Router는 화면의 위치를 관리하고, Pinia는 앱 전체의 상태를 관리하며, Axios는 외부 서버 데이터를 가져온다.</strong></p>
</blockquote>
<p>이 세 가지를 연결하면 단순한 Mockup이 아니라, 실제 서비스 구조에 가까운 Vue 애플리케이션을 만들 수 있습니다.</p>