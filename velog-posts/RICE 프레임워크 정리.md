<blockquote>
<p><strong>학습일</strong> : 2026.07.18<br /><strong>주제</strong> : RICE Prompt Framework</p>
</blockquote>
<hr />
<h1 id="rice란">RICE란?</h1>
<p>RICE는 AI에게 원하는 결과를 더욱 정확하게 얻기 위해 사용하는 <strong>프롬프트 설계 프레임워크</strong>이다.</p>
<p>좋은 프롬프트는 단순히 질문을 잘하는 것이 아니라,</p>
<ul>
<li>AI에게 어떤 역할을 줄 것인지</li>
<li>무엇을 수행해야 하는지</li>
<li>어떤 정보를 참고해야 하는지</li>
<li>어떤 형태로 답변을 원하는지</li>
</ul>
<p>를 명확하게 전달해야 한다.</p>
<p>이를 체계적으로 정리한 방법이 <strong>RICE</strong>이다.</p>
<hr />
<h1 id="rice-구성-요소">RICE 구성 요소</h1>
<table>
<thead>
<tr>
<th>요소</th>
<th>의미</th>
<th>목적</th>
</tr>
</thead>
<tbody><tr>
<td>R (Role)</td>
<td>AI의 역할 지정</td>
<td>전문성을 부여</td>
</tr>
<tr>
<td>I (Instruction)</td>
<td>명확한 작업 지시</td>
<td>무엇을 수행할지 지정</td>
</tr>
<tr>
<td>C (Context)</td>
<td>배경 정보 제공</td>
<td>판단에 필요한 정보 제공</td>
</tr>
<tr>
<td>E (Examples)</td>
<td>예시 제공</td>
<td>원하는 답변 스타일 학습</td>
</tr>
<tr>
<td>Format</td>
<td>출력 형식 지정</td>
<td>결과를 일정한 형태로 출력</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-r-role">1. R (Role)</h1>
<p>Role은 AI에게 <strong>역할(Persona)</strong> 을 부여하는 것이다.</p>
<p>AI는 역할에 따라 답변 방식이 크게 달라진다.</p>
<h3 id="예시">예시</h3>
<pre><code>너는 입지 분석 컨설턴트다.</code></pre><pre><code>너는 데이터 분석가다.</code></pre><pre><code>너는 시니어 프론트엔드 개발자다.</code></pre><pre><code>너는 면접관이다.</code></pre><h3 id="왜-필요한가">왜 필요한가?</h3>
<p>같은 질문이라도 역할이 달라지면 답변도 달라진다.</p>
<p>예를 들어</p>
<p>질문</p>
<pre><code>우리 선크림을 어디에 광고하면 좋을까?</code></pre><p>Role이 없다면</p>
<blockquote>
<p>SNS 광고를 추천합니다.</p>
</blockquote>
<p>정도의 일반적인 답변이 나온다.</p>
<p>하지만</p>
<pre><code>너는 브랜드 마케팅 전문가다.</code></pre><p>라고 지정하면</p>
<ul>
<li>타겟 분석</li>
<li>광고 채널</li>
<li>예산</li>
<li>KPI</li>
</ul>
<p>까지 고려한 답변을 받을 수 있다.</p>
<hr />
<h1 id="2-i-instruction">2. I (Instruction)</h1>
<p>Instruction은 AI에게 <strong>무엇을 할지</strong> 명확하게 지시하는 단계이다.</p>
<p>모호한 질문보다</p>
<ul>
<li>개수</li>
<li>조건</li>
<li>기준</li>
</ul>
<p>을 함께 적어주는 것이 좋다.</p>
<h3 id="좋지-않은-예">좋지 않은 예</h3>
<pre><code>추천해줘</code></pre><h3 id="좋은-예">좋은 예</h3>
<pre><code>후보지 A/B/C 중
한 곳을 선택하고

근거 3가지와
리스크 1가지를 작성해줘.</code></pre><p>Instruction에는 가능한 한</p>
<ul>
<li>숫자</li>
<li>조건</li>
<li>우선순위</li>
</ul>
<p>를 포함하는 것이 좋다.</p>
<hr />
<h1 id="3-c-context">3. C (Context)</h1>
<p>Context는 AI가 판단할 수 있도록 <strong>배경 정보</strong>를 제공하는 것이다.</p>
<p>Context가 많을수록 답변의 품질이 좋아진다.</p>
<h3 id="예시-1">예시</h3>
<pre><code>브랜드 팩트시트

- 예산 : 300만원
- 타겟 : 20~30대 여성
- 신제품 출시
- 인스타그램 중심</code></pre><p>또는</p>
<pre><code>후보지 A

유동인구
임대료
경쟁업체
주차장</code></pre><p>등을 제공한다.</p>
<h3 id="context가-중요한-이유">Context가 중요한 이유</h3>
<p>AI는 주어진 정보만으로 판단한다.</p>
<p>정보가 부족하면</p>
<ul>
<li>추측</li>
<li>일반론</li>
<li>모호한 답변</li>
</ul>
<p>이 많아질 수 있다.</p>
<hr />
<h1 id="4-e-examples">4. E (Examples)</h1>
<p>Examples는 원하는 답변의 예시를 보여주는 것이다.</p>
<p>AI는 예시를 매우 잘 따라간다.</p>
<p>예를 들어</p>
<pre><code>아래 형식처럼 작성해줘.

결론

근거1

근거2

근거3

리스크</code></pre><p>처럼 예시를 하나 제공하면</p>
<p>거의 동일한 스타일로 작성한다.</p>
<p>이를 <strong>Few-shot Prompting</strong>이라고도 한다.</p>
<hr />
<h1 id="5-format">5. Format</h1>
<p>Format은 출력 형식을 지정하는 것이다.</p>
<p>예를 들어</p>
<pre><code>Markdown으로 작성</code></pre><pre><code>표 형태로 작성</code></pre><pre><code>JSON으로 출력</code></pre><pre><code>HTML로 작성</code></pre><p>등을 지정할 수 있다.</p>
<h3 id="예시-2">예시</h3>
<pre><code>다음 형식으로 작성해줘.

## 결론

## 근거

## 리스크

## 실행방안</code></pre><p>Format을 지정하지 않으면</p>
<p>AI는 임의의 형식으로 답변할 수 있다.</p>
<hr />
<h1 id="rice-예시">RICE 예시</h1>
<pre><code>Role

너는 입지 분석 컨설턴트다.

Instruction

후보지 A/B/C 중
가장 적합한 곳을 선택하고
근거를 3가지 작성해라.

Context

후보지별
유동인구
임대료
상권 정보
브랜드 특성

Examples

아래 예시처럼 작성

결론

후보 A

근거

①

②

③

리스크

...

Format

Markdown</code></pre><hr />
<h1 id="좋은-프롬프트와-나쁜-프롬프트">좋은 프롬프트와 나쁜 프롬프트</h1>
<h2 id="나쁜-프롬프트">나쁜 프롬프트</h2>
<pre><code>광고 어디에 하면 좋을까?</code></pre><p>문제점</p>
<ul>
<li>역할 없음</li>
<li>정보 없음</li>
<li>조건 없음</li>
<li>출력 형식 없음</li>
</ul>
<hr />
<h2 id="좋은-프롬프트">좋은 프롬프트</h2>
<pre><code>너는 브랜드 마케팅 전문가다.

20~30대 여성을 대상으로 하는 선크림 브랜드이다.

예산은 300만원이다.

가장 효과적인 광고 채널을
우선순위 3개로 추천하고

각 채널의

- 장점
- 단점
- 예상 효과

를 표로 작성해줘.</code></pre><p>차이가 매우 크다.</p>
<hr />
<h1 id="rice를-사용하는-이유">RICE를 사용하는 이유</h1>
<p>RICE를 사용하면</p>
<ul>
<li>답변의 일관성이 높아진다.</li>
<li>원하는 결과를 얻을 가능성이 높아진다.</li>
<li>AI가 추측하는 내용을 줄일 수 있다.</li>
<li>수정 횟수를 줄일 수 있다.</li>
<li>실무에서 재사용하기 쉬운 프롬프트를 만들 수 있다.</li>
</ul>
<p>특히 업무 자동화나 AI 활용 프로젝트에서는 거의 필수적으로 사용되는 구조이다.</p>
<hr />
<h1 id="rice-작성-순서">RICE 작성 순서</h1>
<pre><code class="language-text">Role
    ↓
Instruction
    ↓
Context
    ↓
Examples
    ↓
Format</code></pre>
<p>이 순서대로 작성하면 프롬프트가 훨씬 명확해진다.</p>
<hr />
<h1 id="핵심-정리">핵심 정리</h1>
<table>
<thead>
<tr>
<th>요소</th>
<th>핵심 질문</th>
</tr>
</thead>
<tbody><tr>
<td>Role</td>
<td>AI는 누구인가?</td>
</tr>
<tr>
<td>Instruction</td>
<td>무엇을 해야 하는가?</td>
</tr>
<tr>
<td>Context</td>
<td>어떤 정보를 참고해야 하는가?</td>
</tr>
<tr>
<td>Examples</td>
<td>어떤 스타일을 원하는가?</td>
</tr>
<tr>
<td>Format</td>
<td>어떤 형태로 출력할 것인가?</td>
</tr>
</tbody></table>
<hr />
<h1 id="오늘-배운-내용">오늘 배운 내용</h1>
<ul>
<li>RICE는 프롬프트를 구조적으로 작성하기 위한 프레임워크이다.</li>
<li>Role을 지정하면 AI의 전문성이 달라진다.</li>
<li>Instruction은 최대한 구체적으로 작성한다.</li>
<li>Context가 많을수록 정확도가 높아진다.</li>
<li>Examples는 원하는 답변 스타일을 학습시키는 역할을 한다.</li>
<li>Format을 지정하면 결과를 일정한 형태로 받을 수 있다.</li>
<li>RICE를 활용하면 AI의 응답 품질과 재사용성이 크게 향상된다.</li>
</ul>
<hr />
<h1 id="한-줄-정리">한 줄 정리</h1>
<blockquote>
<p><strong>좋은 AI 답변은 좋은 질문에서 시작하며, 좋은 질문은 RICE 구조를 따른다.</strong></p>
</blockquote>