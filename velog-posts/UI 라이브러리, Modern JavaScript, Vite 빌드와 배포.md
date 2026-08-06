<blockquote>
<p>오늘의 목표는 Vue로 만든 앱을 더 빠르고 안전하게 완성한 뒤, 실제 사용자가 접속할 수 있도록 배포하는 것입니다.</p>
</blockquote>
<h2 id="오늘의-큰-흐름">오늘의 큰 흐름</h2>
<pre><code class="language-text">UI 라이브러리
  → 화면을 빠르고 일관성 있게 만든다.

Modern JavaScript
  → 데이터를 안전하고 간결하게 처리한다.

Vite 빌드와 배포
  → 개발 코드를 브라우저용 결과물로 만들고 서비스한다.</code></pre>
<hr />
<h1 id="1-ui-라이브러리---검증된-ui-부품-상자">1. UI 라이브러리 - 검증된 UI 부품 상자</h1>
<h2 id="ui-라이브러리란">UI 라이브러리란?</h2>
<p>UI 라이브러리는 버튼, 입력창, 모달, 테이블, 알림처럼 웹 화면에서 반복해서 필요한 부품을 미리 만들어 둔 패키지입니다.</p>
<p>직접 버튼 하나를 만들려면 HTML, CSS, hover 효과, disabled 상태, 키보드 접근성까지 신경 써야 합니다.</p>
<pre><code class="language-html">&lt;button&gt;저장&lt;/button&gt;</code></pre>
<p>하지만 UI 라이브러리를 사용하면 디자인과 기본 동작이 준비된 컴포넌트를 바로 사용할 수 있습니다.</p>
<pre><code class="language-vue">&lt;el-button type=&quot;primary&quot;&gt;저장&lt;/el-button&gt;</code></pre>
<blockquote>
<p>UI 라이브러리는 “디자인을 포기하는 도구”가 아니라, 반복 작업을 줄이고 기능 구현에 집중하게 해주는 도구입니다.</p>
</blockquote>
<hr />
<h2 id="ui-라이브러리를-쓰는-이유">UI 라이브러리를 쓰는 이유</h2>
<table>
<thead>
<tr>
<th>장점</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>개발 속도 향상</td>
<td>자주 쓰는 UI를 처음부터 만들 필요가 없음</td>
</tr>
<tr>
<td>일관된 디자인</td>
<td>버튼, 색상, 여백, 폼이 같은 규칙을 따름</td>
</tr>
<tr>
<td>반응형 지원</td>
<td>모바일과 데스크톱 대응이 비교적 쉬움</td>
</tr>
<tr>
<td>접근성 지원</td>
<td>키보드 포커스, ARIA 속성 등을 기본 제공</td>
</tr>
<tr>
<td>검증된 동작</td>
<td>모달, 테이블, 날짜 선택 등 복잡한 기능을 안정적으로 사용</td>
</tr>
</tbody></table>
<hr />
<h2 id="vue-3에서-자주-쓰는-ui-라이브러리">Vue 3에서 자주 쓰는 UI 라이브러리</h2>
<table>
<thead>
<tr>
<th>라이브러리</th>
<th>특징</th>
<th>어울리는 프로젝트</th>
</tr>
</thead>
<tbody><tr>
<td>Vuetify</td>
<td>Material Design 기반</td>
<td>모바일 친화적 서비스</td>
</tr>
<tr>
<td>Element Plus</td>
<td>관리 화면, 테이블, 폼에 강함</td>
<td>사내 시스템, 관리자 페이지</td>
</tr>
<tr>
<td>PrimeVue</td>
<td>다양한 테마와 컴포넌트</td>
<td>복잡한 데이터 중심 서비스</td>
</tr>
</tbody></table>
<p>이번 강의에서는 <strong>Element Plus</strong>를 사용합니다.</p>
<hr />
<h2 id="element-plus-설치와-등록">Element Plus 설치와 등록</h2>
<h3 id="1-설치">1. 설치</h3>
<pre><code class="language-bash">npm install element-plus</code></pre>
<h3 id="2-mainjs에-등록">2. <code>main.js</code>에 등록</h3>
<pre><code class="language-js">import { createApp } from 'vue'
import { createPinia } from 'pinia'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.use(ElementPlus)

app.mount('#app')</code></pre>
<p>여기서 CSS import를 빼먹으면 컴포넌트는 보이더라도 Element Plus 스타일이 적용되지 않습니다.</p>
<hr />
<h2 id="자주-쓰는-컴포넌트">자주 쓰는 컴포넌트</h2>
<h3 id="버튼">버튼</h3>
<pre><code class="language-vue">&lt;el-button&gt;기본 버튼&lt;/el-button&gt;
&lt;el-button type=&quot;primary&quot;&gt;저장&lt;/el-button&gt;
&lt;el-button type=&quot;danger&quot;&gt;삭제&lt;/el-button&gt;
&lt;el-button :loading=&quot;isLoading&quot;&gt;불러오는 중&lt;/el-button&gt;</code></pre>
<h3 id="입력창">입력창</h3>
<pre><code class="language-vue">&lt;script setup&gt;
import { ref } from 'vue'

const searchQuery = ref('')
&lt;/script&gt;

&lt;template&gt;
  &lt;el-input
    v-model=&quot;searchQuery&quot;
    placeholder=&quot;도시를 검색하세요&quot;
    clearable
  /&gt;
&lt;/template&gt;</code></pre>
<p>기존의 <code>:value</code>와 <code>@input</code> 조합을 <code>v-model</code>로 더 간단하게 작성한 형태입니다.</p>
<h3 id="상태-태그">상태 태그</h3>
<p>날씨 상태처럼 짧은 정보를 강조할 때 좋습니다.</p>
<pre><code class="language-vue">&lt;el-tag type=&quot;success&quot;&gt;맑음&lt;/el-tag&gt;
&lt;el-tag type=&quot;info&quot;&gt;흐림&lt;/el-tag&gt;
&lt;el-tag type=&quot;primary&quot;&gt;구름&lt;/el-tag&gt;</code></pre>
<h3 id="빈-결과-화면">빈 결과 화면</h3>
<pre><code class="language-vue">&lt;el-empty description=&quot;검색 결과가 없습니다.&quot; /&gt;</code></pre>
<p>직접 아이콘과 문구, 여백을 구성하지 않아도 일관된 빈 결과 화면을 만들 수 있습니다.</p>
<hr />
<h2 id="alert-대신-메시지-컴포넌트-사용하기"><code>alert()</code> 대신 메시지 컴포넌트 사용하기</h2>
<p>브라우저 기본 <code>alert()</code>은 학습용으로는 충분하지만, 실제 서비스에서는 디자인 통일이 어렵습니다.</p>
<pre><code class="language-js">import { ElMessage } from 'element-plus'

const saveWeather = () =&gt; {
  ElMessage.success('날씨 정보가 저장되었습니다.')
}</code></pre>
<p>삭제처럼 중요한 작업은 확인 창을 사용할 수 있습니다.</p>
<pre><code class="language-js">import { ElMessageBox } from 'element-plus'

const confirmDelete = async () =&gt; {
  try {
    await ElMessageBox.confirm(
      '정말 삭제하시겠습니까?',
      '삭제 확인',
      {
        confirmButtonText: '삭제',
        cancelButtonText: '취소',
        type: 'warning',
      },
    )

    console.log('삭제 진행')
  } catch {
    console.log('삭제 취소')
  }
}</code></pre>
<hr />
<h2 id="ui-라이브러리를-사용할-때-주의할-점">UI 라이브러리를 사용할 때 주의할 점</h2>
<p>UI 라이브러리를 설치했다고 모든 화면을 라이브러리 컴포넌트만으로 만들 필요는 없습니다.</p>
<p>좋은 기준은 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>직접 CSS가 더 좋은 경우</th>
<th>UI 라이브러리가 더 좋은 경우</th>
</tr>
</thead>
<tbody><tr>
<td>브랜드만의 독특한 카드 디자인</td>
<td>복잡한 테이블</td>
</tr>
<tr>
<td>간단한 페이지 레이아웃</td>
<td>날짜 선택</td>
</tr>
<tr>
<td>프로젝트 전용 작은 컴포넌트</td>
<td>모달, 알림, 파일 업로드</td>
</tr>
<tr>
<td>애니메이션이 중요한 화면</td>
<td>폼 검증</td>
</tr>
</tbody></table>
<blockquote>
<p>“반복되고 복잡한 UI”에는 라이브러리를, “서비스만의 개성이 필요한 UI”에는 직접 CSS를 사용하는 편이 좋습니다.</p>
</blockquote>
<hr />
<h1 id="2-modern-javascript---vue를-더-잘-쓰기-위한-최신-문법">2. Modern JavaScript - Vue를 더 잘 쓰기 위한 최신 문법</h1>
<p>Vue의 <code>ref</code>, <code>computed</code>, <code>props</code>, <code>axios</code> 코드는 결국 JavaScript로 작성합니다. 따라서 Modern JavaScript 문법을 알면 Vue 코드가 훨씬 짧고 안전해집니다.</p>
<hr />
<h2 id="var-대신-const와-let"><code>var</code> 대신 <code>const</code>와 <code>let</code></h2>
<table>
<thead>
<tr>
<th>키워드</th>
<th>값 변경</th>
<th>권장 용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>const</code></td>
<td>불가능</td>
<td>기본값, 함수, 바뀌지 않는 참조</td>
</tr>
<tr>
<td><code>let</code></td>
<td>가능</td>
<td>반복문 변수, 변경되는 일반 변수</td>
</tr>
<tr>
<td><code>var</code></td>
<td>가능</td>
<td>과거 문법, 새 코드에서는 사용하지 않음</td>
</tr>
</tbody></table>
<pre><code class="language-js">const cityName = '서울'
let temperature = 25

temperature = 27</code></pre>
<p>기본 원칙은 간단합니다.</p>
<blockquote>
<p>일단 <code>const</code>로 선언하고, 나중에 값 변경이 필요할 때만 <code>let</code>을 사용합니다.</p>
</blockquote>
<p>주의할 점도 있습니다.</p>
<pre><code class="language-js">const weather = {
  name: '서울',
  temp: 25,
}

weather.temp = 28 // 가능</code></pre>
<p><code>const</code>는 객체 내부 값을 절대 바꾸지 못하게 하는 것이 아니라, 변수 자체가 다른 객체를 가리키도록 재할당하는 것을 막습니다.</p>
<hr />
<h2 id="화살표-함수">화살표 함수</h2>
<p>Vue에서는 이벤트 핸들러, 배열 메서드, 비동기 함수에서 화살표 함수를 자주 사용합니다.</p>
<pre><code class="language-js">const getMessage = () =&gt; {
  return '안녕하세요'
}</code></pre>
<p>코드가 한 줄이면 더 줄일 수 있습니다.</p>
<pre><code class="language-js">const getMessage = () =&gt; '안녕하세요'</code></pre>
<p>배열에서 조건에 맞는 도시만 찾는 예시입니다.</p>
<pre><code class="language-js">const hotCities = weatherList.filter((city) =&gt; city.temp &gt;= 25)</code></pre>
<hr />
<h2 id="템플릿-리터럴">템플릿 리터럴</h2>
<p>문자열과 변수를 합칠 때 백틱(<code>)과</code>${}`를 사용합니다.</p>
<pre><code class="language-js">const city = '서울'
const temp = 28

const message = `${city}의 현재 기온은 ${temp}°C입니다.`</code></pre>
<p>기존 방식보다 읽기 쉽습니다.</p>
<pre><code class="language-js">const message = city + '의 현재 기온은 ' + temp + '°C입니다.'</code></pre>
<p>Vue 상태바 문구에도 자연스럽게 사용할 수 있습니다.</p>
<pre><code class="language-js">selectedCityInfo.value = `${item.name}이 선택되었습니다.`</code></pre>
<hr />
<h2 id="구조-분해-할당">구조 분해 할당</h2>
<p>객체나 배열에서 필요한 값만 꺼낼 때 사용합니다.</p>
<pre><code class="language-js">const weather = {
  name: '서울특별시',
  temp: 28,
  status: '맑음',
}

const { name, temp } = weather

console.log(name) // 서울특별시
console.log(temp) // 28</code></pre>
<p>Axios 응답을 다룰 때도 자주 사용합니다.</p>
<pre><code class="language-js">const response = await axios.get('/weather')
const { data } = response</code></pre>
<p>배열은 순서대로 꺼냅니다.</p>
<pre><code class="language-js">const coordinates = [37.5665, 126.978]

const [latitude, longitude] = coordinates</code></pre>
<hr />
<h2 id="spread-연산자----복사하고-펼치기">Spread 연산자 <code>...</code> - 복사하고 펼치기</h2>
<p>Spread 연산자는 배열이나 객체를 펼쳐 새 배열 또는 새 객체를 만들 때 사용합니다.</p>
<pre><code class="language-js">const oldCities = ['서울', '부산']
const newCities = [...oldCities, '제주']

console.log(newCities)
// ['서울', '부산', '제주']</code></pre>
<p>객체를 수정할 때 특히 중요합니다.</p>
<pre><code class="language-js">const weather = {
  name: '서울',
  temp: 25,
}

const updatedWeather = {
  ...weather,
  temp: 28,
}</code></pre>
<p>원본 <code>weather</code>는 유지되고, 수정된 새 객체가 만들어집니다.</p>
<pre><code class="language-text">weather        → 서울, 25도
updatedWeather → 서울, 28도</code></pre>
<p>이러한 방식은 Vue와 Pinia에서 상태를 안전하게 다룰 때 자주 사용됩니다.</p>
<hr />
<h2 id="rest-문법----나머지-모으기">Rest 문법 <code>...</code> - 나머지 모으기</h2>
<p>기호는 Spread와 같지만 역할이 반대입니다.</p>
<pre><code class="language-js">const weather = {
  name: '서울',
  temp: 28,
  status: '맑음',
  humidity: 60,
}

const { name, temp, ...details } = weather

console.log(details)
// { status: '맑음', humidity: 60 }</code></pre>
<ul>
<li>Spread: 값을 펼친다.</li>
<li>Rest: 남은 값을 모은다.</li>
</ul>
<hr />
<h2 id="optional-chaining-">Optional Chaining <code>?.</code></h2>
<p>API 데이터는 항상 모든 값이 있다고 보장할 수 없습니다.</p>
<pre><code class="language-js">const user = {
  name: '홍길동',
}</code></pre>
<p>다음 코드는 오류가 발생합니다.</p>
<pre><code class="language-js">user.profile.address.city</code></pre>
<p>왜냐하면 <code>profile</code>이 없기 때문입니다.</p>
<p>Optional Chaining을 사용하면 안전하게 접근할 수 있습니다.</p>
<pre><code class="language-js">const city = user?.profile?.address?.city</code></pre>
<p>값이 없으면 오류 대신 <code>undefined</code>를 반환합니다.</p>
<hr />
<h2 id="nullish-coalescing-">Nullish Coalescing <code>??</code></h2>
<p><code>??</code>는 값이 <code>null</code> 또는 <code>undefined</code>일 때만 기본값을 제공합니다.</p>
<pre><code class="language-js">const cityName = city ?? '도시 정보 없음'</code></pre>
<p><code>||</code>와 <code>??</code>의 차이를 꼭 기억해야 합니다.</p>
<pre><code class="language-js">const temp = 0

const wrong = temp || 20
const correct = temp ?? 20

console.log(wrong) // 20
console.log(correct) // 0</code></pre>
<p>온도 <code>0</code>은 실제 데이터입니다. 따라서 기본값으로 바뀌면 안 됩니다.</p>
<blockquote>
<p>숫자 <code>0</code>, 빈 문자열 <code>''</code>, <code>false</code>도 유효한 데이터라면 <code>||</code>보다 <code>??</code>가 더 안전합니다.</p>
</blockquote>
<hr />
<h2 id="promise와-async--await">Promise와 <code>async / await</code></h2>
<p>Axios처럼 서버 통신에는 시간이 걸립니다. JavaScript는 기다리는 동안 화면이 멈추지 않도록 비동기로 동작합니다.</p>
<pre><code class="language-js">const response = await axios.get('/weather')</code></pre>
<p><code>await</code>는 “응답이 도착할 때까지 이 함수 안에서 기다려”라는 의미입니다.</p>
<pre><code class="language-js">const fetchWeather = async () =&gt; {
  try {
    const response = await axios.get('/weather')

    console.log(response.data)
  } catch (error) {
    console.error('날씨 조회 실패:', error)
  }
}</code></pre>
<table>
<thead>
<tr>
<th>문법</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>async</code></td>
<td>함수 안에서 비동기 처리를 사용하겠다고 선언</td>
</tr>
<tr>
<td><code>await</code></td>
<td>Promise가 끝날 때까지 기다림</td>
</tr>
<tr>
<td><code>try</code></td>
<td>정상 처리 영역</td>
</tr>
<tr>
<td><code>catch</code></td>
<td>오류 처리 영역</td>
</tr>
<tr>
<td><code>finally</code></td>
<td>성공·실패와 관계없이 마지막에 실행</td>
</tr>
</tbody></table>
<hr />
<h1 id="3-vite-빌드와-실무-배포">3. Vite 빌드와 실무 배포</h1>
<h2 id="개발-서버와-배포-파일은-다르다">개발 서버와 배포 파일은 다르다</h2>
<p>개발 중에는 다음 명령어를 사용합니다.</p>
<pre><code class="language-bash">npm run dev</code></pre>
<p>이 명령은 로컬 개발 서버를 실행합니다.</p>
<pre><code class="language-text">http://localhost:5173</code></pre>
<p>하지만 이 상태는 내 컴퓨터에서 개발하기 위한 환경입니다. 다른 사람이 접속할 수 있는 서비스가 아닙니다.</p>
<p>사용자에게 배포하려면 다음 명령어를 실행해야 합니다.</p>
<pre><code class="language-bash">npm run build</code></pre>
<hr />
<h2 id="빌드란">빌드란?</h2>
<p>빌드는 여러 개의 Vue, JavaScript, CSS, 이미지 파일을 브라우저가 빠르게 읽을 수 있는 정적 파일 묶음으로 바꾸는 과정입니다.</p>
<pre><code class="language-text">src/
├── App.vue
├── views/
├── components/
└── assets/

        ↓ npm run build

dist/
├── index.html
└── assets/
    ├── index-해시값.js
    └── index-해시값.css</code></pre>
<p>빌드가 끝나면 <code>dist</code> 폴더가 생성됩니다.</p>
<blockquote>
<p><code>dist</code>는 Distribution의 줄임말이며, 실제 배포 대상입니다.</p>
</blockquote>
<hr />
<h2 id="파일명-뒤의-해시값은-왜-붙을까">파일명 뒤의 해시값은 왜 붙을까?</h2>
<pre><code class="language-text">index-BACWybNg.js</code></pre>
<p>파일명 뒤의 문자열은 해시값입니다.</p>
<p>코드 내용이 바뀌면 해시값도 바뀝니다. 브라우저가 이전 파일을 캐시하고 있어도, 새 파일명으로 다시 받아 오게 만드는 장치입니다.</p>
<hr />
<h2 id="배포-전-필수-명령어">배포 전 필수 명령어</h2>
<pre><code class="language-bash">npm run lint
npm run format
npm run build</code></pre>
<table>
<thead>
<tr>
<th>명령어</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>npm run lint</code></td>
<td>문법 오류, 사용하지 않는 변수 등 점검</td>
</tr>
<tr>
<td><code>npm run format</code></td>
<td>들여쓰기, 따옴표, 줄바꿈 형식 통일</td>
</tr>
<tr>
<td><code>npm run build</code></td>
<td>배포용 <code>dist</code> 폴더 생성</td>
</tr>
</tbody></table>
<hr />
<h2 id="eslint와-prettier-차이">ESLint와 Prettier 차이</h2>
<p>둘은 역할이 다릅니다.</p>
<table>
<thead>
<tr>
<th>도구</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>ESLint</td>
<td>코드의 오류 가능성, 좋지 않은 패턴 검사</td>
</tr>
<tr>
<td>Prettier</td>
<td>들여쓰기, 줄바꿈, 따옴표 등 코드 모양 정리</td>
</tr>
</tbody></table>
<p>예를 들어 ESLint는 이런 실수를 잡아낼 수 있습니다.</p>
<pre><code class="language-js">const city = 'Seoul'

console.log(citty)</code></pre>
<p><code>city</code>와 <code>citty</code>가 다르기 때문에 오류 가능성을 알려줍니다.</p>
<p>Prettier는 이런 코드를,</p>
<pre><code class="language-js">const city=&quot;Seoul&quot;</code></pre>
<p>다음처럼 프로젝트 규칙에 맞춰 정리합니다.</p>
<pre><code class="language-js">const city = 'Seoul'</code></pre>
<hr />
<h2 id="vite-환경-변수">Vite 환경 변수</h2>
<p>API 서버 주소처럼 환경마다 달라지는 값은 코드에 직접 작성하지 않는 편이 좋습니다.</p>
<pre><code class="language-js">const apiUrl = 'https://api.example.com'</code></pre>
<p>개발 서버와 실제 서버의 주소가 다르면 매번 코드를 수정해야 합니다.</p>
<p>Vite에서는 <code>.env</code> 파일을 사용합니다.</p>
<pre><code class="language-env"># .env.development
VITE_API_BASE_URL=http://localhost:8080</code></pre>
<pre><code class="language-env"># .env.production
VITE_API_BASE_URL=https://api.example.com</code></pre>
<p>Vue 코드에서는 다음처럼 사용합니다.</p>
<pre><code class="language-js">const apiBaseUrl = import.meta.env.VITE_API_BASE_URL</code></pre>
<p>Vite에서 클라이언트 코드에 노출할 변수는 반드시 <code>VITE_</code>로 시작해야 합니다.</p>
<pre><code class="language-env">VITE_API_BASE_URL=https://api.example.com</code></pre>
<p>중요한 보안 원칙도 있습니다.</p>
<blockquote>
<p><code>VITE_</code>로 시작하는 값은 브라우저에 전달됩니다.<br />따라서 비밀번호, JWT 비밀키, 데이터베이스 비밀번호 같은 진짜 비밀값을 넣으면 안 됩니다.</p>
</blockquote>
<hr />
<h2 id="환경별-빌드하기">환경별 빌드하기</h2>
<p><code>package.json</code>에 다음과 같이 설정할 수 있습니다.</p>
<pre><code class="language-json">{
  &quot;scripts&quot;: {
    &quot;build:staging&quot;: &quot;vite build --mode staging&quot;,
    &quot;build:production&quot;: &quot;vite build --mode production&quot;
  }
}</code></pre>
<p>그리고 명령어로 구분합니다.</p>
<pre><code class="language-bash">npm run build:staging
npm run build:production</code></pre>
<hr />
<h2 id="github-pages-배포-시-알아둘-점">GitHub Pages 배포 시 알아둘 점</h2>
<p>GitHub 저장소 이름이 <code>test</code>라면 GitHub Pages에서 앱이 다음 주소처럼 열릴 수 있습니다.</p>
<pre><code class="language-text">https://사용자이름.github.io/test/</code></pre>
<p>이 경우 Vite의 <code>base</code> 설정이 필요할 수 있습니다.</p>
<pre><code class="language-js">import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  base: '/test/',
  plugins: [vue()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
})</code></pre>
<p><code>base</code> 설정이 맞지 않으면 배포 후 CSS, JavaScript, 이미지 경로를 찾지 못해 화면이 깨질 수 있습니다.</p>
<hr />
<h2 id="vue-router-배포-시-주의할-점">Vue Router 배포 시 주의할 점</h2>
<p>SPA에서 <code>/weather/region_01</code>처럼 상세 주소로 직접 접근할 때 서버가 이를 Vue 앱으로 연결해 주어야 합니다.</p>
<p>정적 호스팅 환경에 따라 새로고침 시 404가 발생할 수 있습니다.</p>
<p>이 문제를 줄이는 방법은 다음과 같습니다.</p>
<ul>
<li>호스팅 서비스에서 SPA fallback 설정을 확인한다.</li>
<li>GitHub Pages에서는 Hash History 방식도 고려한다.</li>
<li>배포 전 상세 페이지 URL에 직접 접속해 본다.</li>
</ul>
<hr />
<h1 id="4일차-최종-점검표">4일차 최종 점검표</h1>
<h2 id="ui-라이브러리">UI 라이브러리</h2>
<ul>
<li><input disabled="" type="checkbox" /> Element Plus를 설치했다.</li>
<li><input disabled="" type="checkbox" /> <code>main.js</code>에 Element Plus와 CSS를 등록했다.</li>
<li><input disabled="" type="checkbox" /> 버튼, 입력창, 태그, 메시지 중 하나 이상을 적용했다.</li>
<li><input disabled="" type="checkbox" /> 기본 <code>alert()</code> 대신 <code>ElMessage</code> 또는 <code>ElMessageBox</code>를 사용해 봤다.</li>
</ul>
<h2 id="modern-javascript">Modern JavaScript</h2>
<ul>
<li><input disabled="" type="checkbox" /> <code>var</code> 대신 <code>const</code>, <code>let</code>을 사용한다.</li>
<li><input disabled="" type="checkbox" /> 화살표 함수와 템플릿 리터럴을 사용할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 객체/배열 구조 분해 할당을 사용할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Spread 연산자로 새 배열·객체를 만들 수 있다.</li>
<li><input disabled="" type="checkbox" /> <code>?.</code>와 <code>??</code>로 API 데이터 누락을 안전하게 처리할 수 있다.</li>
<li><input disabled="" type="checkbox" /> <code>async / await</code>, <code>try / catch</code>를 사용할 수 있다.</li>
</ul>
<h2 id="vite-빌드와-배포">Vite 빌드와 배포</h2>
<ul>
<li><input disabled="" type="checkbox" /> <code>npm run lint</code>를 실행했다.</li>
<li><input disabled="" type="checkbox" /> <code>npm run format</code>을 실행했다.</li>
<li><input disabled="" type="checkbox" /> <code>npm run build</code>가 성공했다.</li>
<li><input disabled="" type="checkbox" /> <code>dist</code> 폴더가 생성되었는지 확인했다.</li>
<li><input disabled="" type="checkbox" /> API 관련 설정을 환경 변수로 분리했다.</li>
<li><input disabled="" type="checkbox" /> GitHub 저장소가 Public인지 확인했다.</li>
<li><input disabled="" type="checkbox" /> 배포 후 시크릿 창에서 정상 접속되는지 확인했다.</li>
</ul>
<hr />
<h2 id="오늘의-핵심-요약">오늘의 핵심 요약</h2>
<table>
<thead>
<tr>
<th>주제</th>
<th>한 문장 정리</th>
</tr>
</thead>
<tbody><tr>
<td>UI 라이브러리</td>
<td>반복되는 UI를 빠르고 일관성 있게 만든다.</td>
</tr>
<tr>
<td>Modern JavaScript</td>
<td>데이터를 짧고 안전하게 처리하는 최신 문법이다.</td>
</tr>
<tr>
<td>Vite 빌드</td>
<td>개발 코드를 브라우저용 정적 파일로 변환한다.</td>
</tr>
<tr>
<td>배포</td>
<td><code>dist</code> 폴더를 정적 호스팅 서비스에 올리는 과정이다.</td>
</tr>
</tbody></table>
<blockquote>
<p>좋은 Vue 프로젝트는 화면만 예쁜 프로젝트가 아닙니다.<br /><strong>일관된 UI</strong>, <strong>안전한 데이터 처리</strong>, <strong>검증된 빌드와 배포 과정</strong>까지 갖춰야 실제 서비스에 가까워집니다.</p>
</blockquote>