<h2 id="학습-목표">학습 목표</h2>
<p>오늘은 한 단계 더 나아가 <strong>AI가 원하는 형식으로 답하게 만들고, 우리 문서를 근거로 답하게 만드는 방법</strong>을 배운다.</p>
<p>오늘의 핵심 목표는 다음과 같다.</p>
<ul>
<li>역할·맥락·지시·출력 형식을 갖춘 프롬프트를 작성한다.</li>
<li>호출별 모델 옵션을 조절하고 스트리밍 응답을 처리한다.</li>
<li>AI의 문자열 응답을 Java 객체로 안전하게 변환한다.</li>
<li>이미지 입력, 임베딩, 긴 문서 처리의 기본 원리를 이해한다.</li>
<li>캐시·라우팅·병렬 처리·평가-교정 같은 LLM 워크플로를 구분한다.</li>
<li>문서를 잘라 벡터로 저장하고 관련 근거를 검색하는 RAG 파이프라인을 이해한다.</li>
<li>검색 실패와 답변 실패를 구분해 RAG를 개선한다.</li>
<li>사내 문서 Q&amp;A API를 만들고 골든 세트로 품질을 측정한다.</li>
</ul>
<p>한 문장으로 정리하면 다음과 같다.</p>
<blockquote>
<p>Day 2의 목적은 모델이 아는 것에만 의존하지 않고, 우리 문서를 검색해 근거와 출처가 있는 답을 안정적인 형식으로 반환하게 만드는 것이다.</p>
</blockquote>
<hr />
<p>전체 흐름은 다음과 같다.</p>
<pre><code class="language-text">좋은 프롬프트
    ↓
안정적인 객체 응답
    ↓
테스트 가능한 AI 워크플로
    ↓
문서 인제스트와 검색
    ↓
검색 품질 개선
    ↓
근거 있는 사내 문서 Q&amp;A</code></pre>
<p>Day 1의 AI가 주문 데이터를 짧게 요약하는 <strong>쓰는 AI</strong>에 가까웠다면, Day 2의 AI는 문서를 찾아 읽은 뒤 답하는 <strong>읽는 AI</strong>에 가깝다.</p>
<hr />
<h2 id="5-5장---프롬프트·옵션·스트리밍">5. 5장 - 프롬프트·옵션·스트리밍</h2>
<h3 id="51-프롬프트란-무엇인가">5.1 프롬프트란 무엇인가</h3>
<p>프롬프트는 단순한 질문 문장이 아니라, 모델에게 전달하는 <strong>작업 명세서</strong>다.</p>
<p>사람에게 “정리해 줘”라고만 말하면 길이, 독자, 형식이 제각각인 답이 나온다. AI도 마찬가지다. 원하는 결과가 있다면 최소한 다음 네 요소를 알려 주는 편이 좋다.</p>
<ol>
<li>역할: 누구처럼 행동해야 하는가?</li>
<li>맥락: 어떤 상황과 자료를 사용해야 하는가?</li>
<li>지시: 정확히 무엇을 해야 하는가?</li>
<li>형식: 결과를 어떤 모양으로 반환해야 하는가?</li>
</ol>
<p>나쁜 예시는 다음과 같다.</p>
<pre><code class="language-text">이 문의를 분류해 줘.</code></pre>
<p>개선한 예시는 다음과 같다.</p>
<pre><code class="language-text">역할: 너는 쇼핑몰 고객 문의 분류 담당자다.
맥락: 분류값은 BILLING, DELIVERY, PRODUCT 중 하나다.
지시: 고객 문의에서 가장 가까운 분류를 선택한다.
형식: 설명 없이 분류값 하나만 출력한다.

고객 문의: 결제가 두 번 됐어요.</code></pre>
<p>좋은 프롬프트는 모델을 더 똑똑하게 바꾸는 것이 아니라, <strong>모델이 풀어야 할 문제의 모호함을 줄인다.</strong></p>
<h3 id="52-system-메시지와-user-메시지">5.2 System 메시지와 User 메시지</h3>
<p>두 메시지의 역할을 식당에 비유하면 이해하기 쉽다.</p>
<ul>
<li>System 메시지: 식당의 운영 원칙과 직원 매뉴얼</li>
<li>User 메시지: 손님이 지금 한 주문</li>
</ul>
<p>System 메시지에는 반복해서 지켜야 할 원칙을 둔다.</p>
<pre><code class="language-text">너는 고객지원 상담 도우미다.
제공된 정보만 사용한다.
확인되지 않은 사실을 추측하지 않는다.
답을 모르면 &quot;확인되지 않습니다&quot;라고 말한다.
답변은 한국어 두 문장 이내로 작성한다.</code></pre>
<p>User 메시지에는 호출할 때마다 달라지는 값을 둔다.</p>
<pre><code class="language-text">문의: 배송이 아직 도착하지 않았어요.</code></pre>
<p>System 프롬프트를 작성할 때는 역할, 범위, 근거 규칙, 불확실성 처리, 출력 형식, 금지 사항을 확인한다.</p>
<h3 id="53-prompttemplate">5.3 PromptTemplate</h3>
<p>매번 문자열을 이어 붙이면 프롬프트가 길어질수록 읽기 어렵고 누락이 생긴다. <code>PromptTemplate</code>은 <code>{변수}</code> 자리에 값을 넣어 같은 프롬프트 구조를 재사용하게 한다.</p>
<pre><code class="language-java">String template = &quot;&quot;&quot;
        다음 고객 문의를 {categories} 중 하나로 분류하라.
        설명하지 말고 분류값만 출력하라.

        문의: {question}
        &quot;&quot;&quot;;

PromptTemplate promptTemplate = new PromptTemplate(template);

Prompt prompt = promptTemplate.create(Map.of(
        &quot;categories&quot;, &quot;BILLING, DELIVERY, PRODUCT&quot;,
        &quot;question&quot;, question
));</code></pre>
<p>변수 바인딩을 사용하면 다음 장점이 있다.</p>
<ul>
<li>프롬프트 본문과 데이터를 구분할 수 있다.</li>
<li>같은 규칙으로 여러 질문을 처리할 수 있다.</li>
<li>프롬프트 변경 내역을 코드 리뷰하기 쉽다.</li>
<li>문자열 연결 과정에서 따옴표나 줄바꿈이 깨지는 일을 줄인다.</li>
</ul>
<p>JSON 예시처럼 중괄호가 많은 템플릿은 <code>{name}</code> 변수 문법과 충돌할 수 있다. 이때는 중괄호 escaping 규칙을 확인하거나 구조화 출력 기능을 사용하는 편이 안전하다.</p>
<h3 id="54-긴-프롬프트를-리소스-파일로-분리하기">5.4 긴 프롬프트를 리소스 파일로 분리하기</h3>
<p>긴 프롬프트를 Java 코드 안에 모두 넣으면 업무 코드와 문장이 섞인다. 샘플은 다음 파일에 코드 리뷰 프롬프트를 분리한다.</p>
<pre><code class="language-text">src/main/resources/prompts/code-review.st</code></pre>
<p>개념적인 사용 방법은 다음과 같다.</p>
<pre><code class="language-java">@Value(&quot;classpath:prompts/code-review.st&quot;)
private Resource codeReviewPrompt;

PromptTemplate template = new PromptTemplate(codeReviewPrompt);</code></pre>
<p>리소스 파일로 분리하면 프롬프트만 따로 리뷰하고 버전 차이를 확인하기 쉽다. 프롬프트도 프로그램의 동작을 바꾸므로 소스 코드처럼 버전 관리해야 한다.</p>
<h3 id="55-few-shot">5.5 Few-shot</h3>
<p>Few-shot은 설명만 하는 대신 <strong>입력과 정답 예시를 몇 개 보여 주는 방법</strong>이다.</p>
<pre><code class="language-text">문의: 카드가 이중으로 결제됐어요.
정답: BILLING

문의: 택배가 아직 안 왔어요.
정답: DELIVERY

문의: 이 노트북의 무게가 궁금해요.
정답: PRODUCT

문의: {question}
정답:</code></pre>
<p>새 직원에게 말로만 분류 규칙을 설명하는 것보다 처리 완료된 사례 세 건을 보여 주는 편이 빠른 것과 같다.</p>
<p>Few-shot은 다음 상황에서 특히 유용하다.</p>
<ul>
<li>분류 기준을 글로 정확히 설명하기 어렵다.</li>
<li>출력 형식을 짧고 일정하게 고정하고 싶다.</li>
<li>경계 사례에서 원하는 판단 방식을 보여 주고 싶다.</li>
</ul>
<p>예시는 품질에 큰 영향을 주므로 서로 모순되지 않아야 하며 실제 입력과 비슷해야 한다.</p>
<h3 id="56-단계적-추론과-출력-범위">5.6 단계적 추론과 출력 범위</h3>
<p>복잡한 문제는 한 번에 답을 요구하기보다 작은 단계로 나누면 도움이 된다.</p>
<pre><code class="language-text">주어진 정보에서 사실을 정리하고, 가능한 원인을 비교한 뒤 최종 결론을 작성하라.
최종 응답에는 결론과 근거만 표시하라.</code></pre>
<p>다만 모든 요청에 긴 추론을 요구하면 지연 시간과 토큰 비용이 커진다. 단순 분류나 값 추출에는 필요하지 않다. 또한 내부 추론 전체를 사용자에게 그대로 노출하기보다 최종 판단과 필요한 근거만 반환한다.</p>
<h3 id="57-chatoptions">5.7 ChatOptions</h3>
<p><code>ChatOptions</code>는 모델의 답변 성향과 출력 한도를 조절한다.</p>
<pre><code class="language-java">ChatOptions options = ChatOptions.builder()
        .temperature(0.2)
        .maxTokens(300)
        .build();

String answer = chatClient.prompt()
        .user(question)
        .options(options)
        .call()
        .content();</code></pre>
<h4 id="temperature">temperature</h4>
<ul>
<li>낮은 값: 분류, 정보 추출, 규칙 기반 요약처럼 일관성이 중요한 작업</li>
<li>높은 값: 아이디어 제안, 문구 작성처럼 다양한 답이 유용한 작업</li>
</ul>
<p><code>temperature=0</code>은 무작위성을 크게 낮추지만 모든 답을 글자 단위로 절대 동일하게 보장하지는 않는다. 모델 버전, 공급자 구현, 서버 상태, 입력 컨텍스트가 달라질 수 있기 때문이다.</p>
<h4 id="maxtokens">maxTokens</h4>
<p>출력 가능한 토큰 수를 제한한다. 너무 작으면 답변이 중간에 잘리고 <code>finishReason=length</code>가 나타날 수 있다.</p>
<p>이를 무시하면 다음과 같은 문제가 생긴다.</p>
<ul>
<li>JSON 닫는 괄호가 없어 객체 변환이 실패한다.</li>
<li>문장의 결론이나 출처가 잘린다.</li>
<li>사용자는 불완전한 답을 정상 응답으로 오해한다.</li>
</ul>
<p>따라서 응답의 종료 이유와 구조화 변환 실패를 관찰해야 한다.</p>
<h3 id="58-출력-형식을-고정하는-네-단계">5.8 출력 형식을 고정하는 네 단계</h3>
<p>후속 Java 코드가 응답을 사용한다면 출력 형식이 흔들리지 않아야 한다.</p>
<table>
<thead>
<tr>
<th>방법</th>
<th>강제력</th>
<th>적합한 상황</th>
</tr>
</thead>
<tbody><tr>
<td>프롬프트로 형식 지시</td>
<td>약함</td>
<td>단순한 한 줄·불릿 응답</td>
</tr>
<tr>
<td>Few-shot 예시</td>
<td>중간</td>
<td>말로 설명하기 어려운 분류·형식</td>
</tr>
<tr>
<td><code>.entity()</code> 구조화 출력</td>
<td>강함</td>
<td>대부분의 Java 객체 응답</td>
</tr>
<tr>
<td>공급자 JSON mode·Schema</td>
<td>가장 강함</td>
<td>형식 위반이 치명적인 경우</td>
</tr>
</tbody></table>
<p>공급자 전용 JSON mode는 강력하지만 모델 공급자를 바꿀 때 수정 범위가 커질 수 있다. 일반적인 업무 API에서는 먼저 <code>.entity()</code>를 고려한다.</p>
<h3 id="59-프롬프트-안티패턴">5.9 프롬프트 안티패턴</h3>
<table>
<thead>
<tr>
<th>안티패턴</th>
<th>문제</th>
<th>개선 방향</th>
</tr>
</thead>
<tbody><tr>
<td>“최대한 잘 요약해 줘”</td>
<td>성공 기준이 모호함</td>
<td>문장 수·길이·독자를 지정</td>
</tr>
<tr>
<td>한 프롬프트를 계속 늘림</td>
<td>지시가 충돌하고 중요한 규칙이 묻힘</td>
<td>작업을 작은 호출로 분리</td>
</tr>
<tr>
<td>사용자 입력을 문자열로 직접 연결</td>
<td>프롬프트 인젝션과 구분 오류 위험</td>
<td><code>{변수}</code> 파라미터 바인딩</td>
</tr>
<tr>
<td>형식을 말로만 길게 설명</td>
<td>해석이 달라짐</td>
<td>Few-shot 또는 구조화 출력</td>
</tr>
<tr>
<td>금지 문장만 나열</td>
<td>모델이 해야 할 대안이 불명확</td>
<td>금지와 올바른 행동을 함께 제시</td>
</tr>
<tr>
<td>출력 검증 없이 바로 사용</td>
<td>형식·범위 오류가 업무로 전파</td>
<td>Schema와 Java 검증 추가</td>
</tr>
</tbody></table>
<h3 id="510-동기-호출과-스트리밍">5.10 동기 호출과 스트리밍</h3>
<p>일반 호출은 전체 답이 완성될 때까지 기다렸다가 한 번에 반환한다.</p>
<pre><code class="language-java">String answer = chatClient.prompt()
        .user(question)
        .call()
        .content();</code></pre>
<p>스트리밍 호출은 생성되는 내용을 조각별로 반환한다.</p>
<pre><code class="language-java">Flux&lt;String&gt; stream = chatClient.prompt()
        .user(question)
        .stream()
        .content();</code></pre>
<p>비유하면 일반 호출은 요리가 모두 끝난 뒤 한 상을 받는 방식이고, 스트리밍은 완성된 접시부터 받는 방식이다. 총 조리 시간이 크게 줄지 않아도 첫 결과를 빨리 보므로 체감 속도가 좋아진다.</p>
<h3 id="511-sse-controller">5.11 SSE Controller</h3>
<p>브라우저로 텍스트 스트림을 보낼 때는 <code>text/event-stream</code>을 사용할 수 있다.</p>
<pre><code class="language-java">@GetMapping(value = &quot;/ch04/stream&quot;, produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux&lt;String&gt; stream(@RequestParam String q) {
    return streamingService.stream(q);
}</code></pre>
<p>터미널에서는 <code>curl -N</code>으로 버퍼링을 끈다.</p>
<pre><code class="language-bash">curl -N 'http://localhost:8080/ch04/stream?q=Spring%20AI를%20소개해줘'</code></pre>
<p>스트리밍에서 반드시 생각할 항목은 다음과 같다.</p>
<ul>
<li>사용자가 연결을 끊었을 때 모델 호출도 취소되는가?</li>
<li>전체 요청에 시간 제한이 있는가?</li>
<li>일부 조각을 보낸 뒤 오류가 나면 어떻게 표시하는가?</li>
<li>첫 토큰 시간과 전체 완료 시간을 따로 측정하는가?</li>
<li>취소·오류·완료 시 리소스가 정리되는가?</li>
</ul>
<pre><code class="language-java">return stream
        .timeout(Duration.ofSeconds(60))
        .doOnCancel(() -&gt; log.info(&quot;클라이언트 취소&quot;))
        .onErrorResume(ex -&gt; Flux.just(&quot;응답 생성 중 오류가 발생했습니다.&quot;));</code></pre>
<p>스트리밍 취소를 처리하지 않으면 사용자가 화면을 닫은 뒤에도 모델 호출과 비용이 계속될 수 있다.</p>
<h3 id="512-재시도와-폴백">5.12 재시도와 폴백</h3>
<p>외부 모델은 일시적으로 <code>429</code>, <code>5xx</code>, 타임아웃을 반환할 수 있다.</p>
<p>재시도는 모든 오류에 무조건 적용하지 않는다.</p>
<ul>
<li><code>429</code>, 일시적 네트워크 오류: 짧은 지수 백오프 후 제한된 횟수만 재시도</li>
<li>잘못된 API 키 <code>401</code>: 재시도해도 해결되지 않음</li>
<li>잘못된 요청 <code>400</code>: 입력이나 설정을 수정해야 함</li>
<li>계속 실패: 기본 응답, 이전 결과, 사람이 처리할 경로로 폴백</li>
</ul>
<p>재시도는 호출 수와 지연 시간을 늘리므로 최대 횟수와 전체 시간 제한을 함께 둔다.</p>
<h3 id="513-프롬프트-평가-방법">5.13 프롬프트 평가 방법</h3>
<p>프롬프트를 바꾼 뒤 예시 한두 개만 보고 좋아졌다고 판단하면 안 된다.</p>
<ol>
<li>실제 업무를 대표하는 질문 20~30개를 준비한다.</li>
<li>이전 프롬프트와 새 프롬프트에 같은 질문을 넣는다.</li>
<li>정확성, 형식 준수, 거절, 길이, 지연, 토큰을 비교한다.</li>
<li>개선된 항목과 나빠진 항목을 함께 기록한다.</li>
</ol>
<p>프롬프트 변경도 코드 변경과 마찬가지로 회귀 테스트가 필요하다.</p>
<hr />
<h2 id="6-6장---구조화-출력·멀티모달·임베딩">6. 6장 - 구조화 출력·멀티모달·임베딩</h2>
<h3 id="61-문자열-응답의-문제">6.1 문자열 응답의 문제</h3>
<p>AI가 다음 문자열을 반환했다고 가정한다.</p>
<pre><code class="language-text">분류는 BILLING이고 우선순위는 HIGH입니다.</code></pre>
<p>프로그램이 이 문장에서 값을 꺼내려면 단어 위치, 구분 기호, 조사 변화를 모두 처리해야 한다. 모델이 표현을 조금만 바꾸어도 파싱 코드가 깨진다.</p>
<p>프로그램이 필요로 하는 것은 자연스러운 문장보다 다음처럼 형태가 정해진 데이터다.</p>
<pre><code class="language-json">{
  &quot;category&quot;: &quot;BILLING&quot;,
  &quot;priority&quot;: &quot;HIGH&quot;,
  &quot;summary&quot;: &quot;카드가 중복 결제됨&quot;,
  &quot;tags&quot;: [&quot;카드&quot;, &quot;중복결제&quot;]
}</code></pre>
<h3 id="62-record와-entity">6.2 record와 entity()</h3>
<p>응답 규격을 Java <code>record</code>로 정의한다.</p>
<pre><code class="language-java">public record TicketResult(
        Category category,
        Priority priority,
        String summary,
        List&lt;String&gt; tags
) {
}</code></pre>
<p>그다음 문자열을 직접 자르지 않고 객체로 받는다.</p>
<pre><code class="language-java">TicketResult result = chatClient.prompt()
        .user(question)
        .call()
        .entity(TicketResult.class);</code></pre>
<p>이 방식의 장점은 다음과 같다.</p>
<ul>
<li>필드 이름과 타입이 코드에 명확히 드러난다.</li>
<li>Service와 Controller가 문자열 규칙에 의존하지 않는다.</li>
<li>컴파일러와 Jackson의 도움을 받을 수 있다.</li>
<li>Swagger 문서에도 객체 구조가 표시된다.</li>
</ul>
<p>구조화 출력은 모델이 JSON을 반환하도록 유도하고 이를 Java 객체로 변환하는 기능이다. 모델이 데이터베이스처럼 완벽하게 규격을 보장하는 것은 아니므로 검증과 복구가 여전히 필요하다.</p>
<h3 id="63-beanoutputconverter와-json-schema">6.3 BeanOutputConverter와 JSON Schema</h3>
<p><code>BeanOutputConverter</code>는 Java 타입을 보고 모델이 따라야 할 출력 형식을 만든다.</p>
<pre><code class="language-java">BeanOutputConverter&lt;TicketResult&gt; converter =
        new BeanOutputConverter&lt;&gt;(TicketResult.class);

String format = converter.getFormat();</code></pre>
<p>프롬프트에 <code>format</code>을 포함해 형식을 지시하고, 반환 문자열을 객체로 변환할 수 있다.</p>
<p>필드 의미가 모호할 때는 설명을 추가한다.</p>
<pre><code class="language-java">public record MovieCard(
        @JsonPropertyDescription(&quot;1에서 5 사이의 평점&quot;) int rating,
        @JsonPropertyDescription(&quot;POSITIVE, NEUTRAL, NEGATIVE 중 하나&quot;) Sentiment sentiment,
        String summary
) {
}</code></pre>
<p>Schema는 형식을 강하게 유도하지만 내용의 진실성까지 보장하지 않는다. <code>rating</code>이 정수로 왔어도 <code>99</code>라면 업무 규칙상 잘못된 값이다.</p>
<h3 id="64-목록과-중첩-객체">6.4 목록과 중첩 객체</h3>
<p>단일 객체가 아닌 목록은 제네릭 타입 정보가 필요하다.</p>
<pre><code class="language-java">List&lt;Keyword&gt; keywords = chatClient.prompt()
        .user(text)
        .call()
        .entity(new ParameterizedTypeReference&lt;List&lt;Keyword&gt;&gt;() {});</code></pre>
<p>중첩 구조도 record로 표현할 수 있다.</p>
<pre><code class="language-java">record Address(String city, String detail) {}
record Customer(String name, Address address) {}</code></pre>
<p><code>Map&lt;String, Object&gt;</code>도 받을 수 있지만 타입 안정성이 약하고 필드가 바뀌어도 컴파일 단계에서 알기 어렵다. 구조가 알려져 있다면 명시적인 record나 DTO가 우선이다.</p>
<h3 id="65-enum·날짜·숫자의-함정">6.5 enum·날짜·숫자의 함정</h3>
<h4 id="enum">enum</h4>
<p>모델이 약속하지 않은 값을 만들 수 있다.</p>
<pre><code class="language-java">enum Sentiment {
    POSITIVE,
    NEUTRAL,
    NEGATIVE,
    UNKNOWN
}</code></pre>
<p>가능한 값은 프롬프트와 Schema에 명확히 적고, 운영에서는 <code>UNKNOWN</code> 같은 안전한 값을 고려한다.</p>
<h4 id="날짜">날짜</h4>
<p><code>내일</code>, <code>다음 주</code>는 기준 시각과 시간대에 따라 달라진다. 입력에 기준 날짜와 시간대를 주고 출력은 ISO 형식으로 고정한다.</p>
<pre><code class="language-text">기준 날짜: 2026-08-18, Asia/Seoul
출력 날짜 형식: YYYY-MM-DD</code></pre>
<h4 id="숫자">숫자</h4>
<p>타입이 숫자라는 사실과 값이 유효하다는 사실은 다르다.</p>
<pre><code class="language-java">if (result.rating() &lt; 1 || result.rating() &gt; 5) {
    throw new IllegalArgumentException(&quot;평점 범위 오류&quot;);
}</code></pre>
<h3 id="66-구조화-출력-실패-복구">6.6 구조화 출력 실패 복구</h3>
<p>구조화 출력은 실패할 수 있다.</p>
<ul>
<li>코드 펜스가 JSON을 감싼다.</li>
<li>필수 필드가 빠진다.</li>
<li>enum에 없는 값이 온다.</li>
<li><code>maxTokens</code> 때문에 JSON이 잘린다.</li>
<li>설명 문장이 JSON 앞뒤에 붙는다.</li>
</ul>
<p>안전한 복구 순서는 다음과 같다.</p>
<ol>
<li>코드 펜스 등 명백한 포장 문자를 제거해 한 번 변환한다.</li>
<li>원래 입력과 오류 내용을 사용해 낮은 temperature로 한 번만 재요청한다.</li>
<li>계속 실패하면 <code>UNKNOWN</code> 같은 안전한 기본 객체를 반환하거나 사람이 처리하도록 보낸다.</li>
</ol>
<p>무한 재시도는 장애와 비용을 키운다. 복구 단계와 횟수를 코드로 고정한다.</p>
<h3 id="67-멀티모달">6.7 멀티모달</h3>
<p>멀티모달 모델은 텍스트 외에 이미지 같은 입력을 함께 이해한다.</p>
<pre><code class="language-java">Media image = new Media(
        MediaType.IMAGE_PNG,
        imageResource
);

ReceiptResult result = chatClient.prompt()
        .user(user -&gt; user
                .text(&quot;영수증에서 상호, 날짜, 총액을 추출하라.&quot;)
                .media(image))
        .call()
        .entity(ReceiptResult.class);</code></pre>
<p>입력과 출력의 흐름은 다음과 같다.</p>
<pre><code class="language-text">이미지 + 텍스트 지시
        ↓
멀티모달 모델
        ↓
영수증 객체, 이미지 설명, 분류 결과</code></pre>
<p>계약서 금액, 의료 정보, 결제 승인처럼 위험이 큰 업무에서는 모델 결과를 바로 확정하지 말고 검증 규칙이나 사람의 승인을 둔다.</p>
<h3 id="68-임베딩">6.8 임베딩</h3>
<p>임베딩은 문장이나 문서를 의미를 나타내는 숫자 배열인 벡터로 바꾸는 과정이다.</p>
<pre><code class="language-text">&quot;반품은 7일 이내 가능합니다&quot;
        ↓ EmbeddingModel
[0.018, -0.442, 0.107, ...]</code></pre>
<p>비슷한 의미의 문장은 벡터 공간에서 가까운 위치에 놓이는 경향이 있다.</p>
<pre><code class="language-text">입력: 텍스트
EmbeddingModel
출력: 실수 벡터</code></pre>
<p><code>ChatModel</code>과 비교하면 다음과 같다.</p>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>입력</th>
<th>출력</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>ChatModel</code></td>
<td>메시지·프롬프트</td>
<td>생성된 텍스트·메타데이터</td>
<td>답변 생성</td>
</tr>
<tr>
<td><code>EmbeddingModel</code></td>
<td>텍스트</td>
<td>숫자 벡터</td>
<td>의미 비교·검색</td>
</tr>
<tr>
<td><code>VectorStore</code></td>
<td>벡터와 문서 / 검색 벡터</td>
<td>유사 문서와 점수</td>
<td>근거 저장·검색</td>
</tr>
</tbody></table>
<p>임베딩 자체가 답을 작성하지는 않는다. RAG에서는 질문과 문서를 같은 임베딩 공간에 놓고 가까운 문서를 찾는 데 사용한다.</p>
<h3 id="69-긴-문서-처리">6.9 긴 문서 처리</h3>
<p>긴 문서를 한 번에 모델에 넣으면 컨텍스트 한도와 비용 문제가 생긴다.</p>
<h4 id="map-reduce">Map-Reduce</h4>
<pre><code class="language-text">문서 조각 A → 부분 요약 A ┐
문서 조각 B → 부분 요약 B ├→ 최종 요약
문서 조각 C → 부분 요약 C ┘</code></pre>
<p>각 조각을 독립적으로 처리할 수 있어 병렬화하기 쉽지만, 조각 사이의 연결 관계가 약해질 수 있다.</p>
<h4 id="refine">Refine</h4>
<pre><code class="language-text">조각 A → 초기 요약
초기 요약 + 조각 B → 개선 요약
개선 요약 + 조각 C → 최종 요약</code></pre>
<p>이전 결과를 다음 단계가 이어받으므로 흐름을 보존하기 쉽지만 순차 호출이라 느릴 수 있다.</p>
<h3 id="610-토큰-비용-줄이기">6.10 토큰 비용 줄이기</h3>
<ul>
<li>필요한 문서 조각만 모델에 넣는다.</li>
<li>출력 형식을 짧게 제한한다.</li>
<li>반복되는 안정적인 결과는 캐시한다.</li>
<li>단순 작업은 작은 모델에 보낸다.</li>
<li>긴 대화에서는 오래된 내용을 요약한다.</li>
<li>구조화 출력에 필요하지 않은 설명을 요구하지 않는다.</li>
</ul>
<p>비용을 줄이는 가장 확실한 호출은 <strong>하지 않은 호출</strong>이다.</p>
<h3 id="611-그-밖의-모달리티">6.11 그 밖의 모달리티</h3>
<p>Spring AI는 모델 종류가 달라도 인터페이스를 주입받아 호출하는 공통 형태를 제공한다.</p>
<table>
<thead>
<tr>
<th>기능</th>
<th>대표 인터페이스</th>
<th>일반적인 비용 단위</th>
<th>예시 용도</th>
</tr>
</thead>
<tbody><tr>
<td>텍스트 생성</td>
<td><code>ChatModel</code></td>
<td>입·출력 토큰</td>
<td>대화·분류·요약</td>
</tr>
<tr>
<td>임베딩</td>
<td><code>EmbeddingModel</code></td>
<td>입력 토큰</td>
<td>검색·RAG</td>
</tr>
<tr>
<td>이미지 생성</td>
<td><code>ImageModel</code></td>
<td>이미지 장수·크기</td>
<td>썸네일·시안</td>
</tr>
<tr>
<td>음성 합성</td>
<td><code>TextToSpeechModel</code></td>
<td>문자 또는 음성 길이</td>
<td>안내 음성·접근성</td>
</tr>
<tr>
<td>음성 인식</td>
<td><code>SpeechModel</code> 계열</td>
<td>음성 길이</td>
<td>회의·상담 녹취</td>
</tr>
</tbody></table>
<p>텍스트 토큰만 보던 관찰 방식으로 이미지와 음성 비용을 계산하면 안 된다. 공급자별 과금 단위, 지원 형식, 파일 크기 제한을 따로 확인한다.</p>
<hr />
<h2 id="7-7장---llm-활용-심화">7. 7장 - LLM 활용 심화</h2>
<h3 id="71-ai-기능을-테스트해야-하는-이유">7.1 AI 기능을 테스트해야 하는 이유</h3>
<p>AI 응답은 외부 네트워크, 모델 버전, 비용, 확률적 출력의 영향을 받는다. 모든 단위 테스트에서 실제 모델을 부르면 테스트가 느리고 불안정해진다.</p>
<p>우리가 자주 검증해야 하는 것은 모델의 지식 자체보다 다음과 같은 <strong>우리 코드의 동작</strong>이다.</p>
<ul>
<li>AI 응답을 DTO로 바르게 변환하는가?</li>
<li>분류 결과에 따라 올바른 Service로 라우팅하는가?</li>
<li>실패하면 폴백하는가?</li>
<li>허용하지 않은 값은 거부하는가?</li>
<li>최대 반복 횟수에서 멈추는가?</li>
</ul>
<h3 id="72-mock-chatmodel">7.2 Mock ChatModel</h3>
<p>단위 테스트에서는 <code>ChatModel</code>이 고정 응답을 반환하도록 모의 처리한다.</p>
<pre><code class="language-java">ChatModel chatModel = mock(ChatModel.class);

when(chatModel.call(any(Prompt.class)))
        .thenReturn(fixedChatResponse(&quot;BILLING&quot;));</code></pre>
<p>이렇게 하면 API 키와 네트워크 없이 라우팅 코드의 결과를 반복해서 검증할 수 있다.</p>
<p>테스트 층을 나누면 좋다.</p>
<table>
<thead>
<tr>
<th>테스트</th>
<th>실행 시점</th>
<th>검증 대상</th>
</tr>
</thead>
<tbody><tr>
<td>단위 테스트</td>
<td>매 커밋</td>
<td>파싱·라우팅·폴백·업무 규칙</td>
</tr>
<tr>
<td>계약 테스트</td>
<td>일별 또는 배포 전</td>
<td>실제 공급자 연결과 응답 형태</td>
</tr>
<tr>
<td>골든 세트</td>
<td>모델·프롬프트 변경 시</td>
<td>대표 질문의 품질 회귀</td>
</tr>
</tbody></table>
<p>생성 문장 전체를 정확히 일치시키기보다 필수 키워드, JSON 구조, 출처, 거절 여부 같은 계약을 검증한다.</p>
<h3 id="73-작업에-맞는-모델-선택">7.3 작업에 맞는 모델 선택</h3>
<p>항상 가장 큰 모델을 사용할 필요는 없다.</p>
<table>
<thead>
<tr>
<th>작업</th>
<th>일반적인 선택 방향</th>
</tr>
</thead>
<tbody><tr>
<td>분류·값 추출</td>
<td>작은 모델, 낮은 temperature</td>
</tr>
<tr>
<td>짧은 요약</td>
<td>중간급 모델, 낮거나 중간 temperature</td>
</tr>
<tr>
<td>상담 문장</td>
<td>자연스러운 표현이 가능한 중간·대형 모델</td>
</tr>
<tr>
<td>복잡한 추론·코드</td>
<td>성능이 높은 대형 모델</td>
</tr>
</tbody></table>
<p>모델 공급자를 바꿀 때는 먼저 다음을 확인한다.</p>
<ol>
<li><code>build.gradle</code> 또는 <code>pom.xml</code>의 Spring AI 모델 스타터 의존성</li>
<li><code>application.yml</code>의 공급자별 모델명·base URL·API 키 환경변수</li>
<li><code>ChatClientConfig</code>와 모델 옵션에서 공급자 전용 타입을 사용했는지</li>
<li>임베딩 모델도 함께 바뀌는지와 벡터 차원이 달라지는지</li>
<li>구조화 출력, 도구 호출, 멀티모달 등 필요한 기능을 새 모델이 지원하는지</li>
</ol>
<h3 id="74-호출-최적화-순서">7.4 호출 최적화 순서</h3>
<p>모델을 더 싸게 쓰기 위한 기본 순서는 다음과 같다.</p>
<pre><code class="language-text">캐시에 있는가?
    ↓ 없음
질문 난이도와 유형을 분류한다
    ↓
필요한 컨텍스트만 남긴다
    ↓
작업에 맞는 모델을 호출한다</code></pre>
<h3 id="75-캐시">7.5 캐시</h3>
<p>같은 질문에 같은 답을 줄 수 있다면 결과를 캐시한다.</p>
<pre><code class="language-java">@Cacheable(cacheNames = &quot;faq&quot;, key = &quot;#question&quot;)
public String answer(String question) {
    return chatClient.prompt()
            .user(question)
            .call()
            .content();
}</code></pre>
<p>캐시 키는 공백, 대소문자, 불필요한 구두점을 정규화할 수 있다. 다만 다음 응답은 함부로 캐시하면 안 된다.</p>
<ul>
<li>사용자별 개인정보가 포함된 답</li>
<li>현재 주문 상태처럼 실시간으로 바뀌는 정보</li>
<li>권한별로 결과가 다른 문서 검색</li>
</ul>
<p>프롬프트 캐싱을 지원하는 공급자에서는 변하지 않는 긴 System 프롬프트를 앞에, 호출마다 달라지는 값을 뒤에 두면 캐시 적중에 유리하다.</p>
<h3 id="76-routing">7.6 Routing</h3>
<p>라우팅은 질문을 분류해 서로 다른 처리 경로로 보내는 패턴이다.</p>
<pre><code class="language-text">질문
  ↓ 분류기
  ├─ SIMPLE_FAQ → 작은 모델 또는 캐시
  ├─ DOCUMENT   → RAG
  └─ COMPLEX    → 큰 모델 또는 복합 워크플로</code></pre>
<p>분류기가 틀리면 뒤의 처리도 틀리므로 라우팅 결과와 이유를 로그로 남기고 라우팅 자체를 테스트한다.</p>
<h3 id="77-chaining">7.7 Chaining</h3>
<p>앞 단계의 출력을 다음 단계의 입력으로 연결한다.</p>
<pre><code class="language-text">원문 → 핵심 사실 추출 → 요약 작성 → 형식 검사</code></pre>
<p>한 호출에 모든 지시를 넣는 것보다 각 단계의 책임과 실패 지점을 확인하기 쉽다. 반면 호출 수와 지연이 늘어나므로 정말 필요한 단계만 둔다.</p>
<h3 id="78-parallel">7.8 Parallel</h3>
<p>독립적인 작업을 동시에 실행한다.</p>
<pre><code class="language-text">고객 후기 ─┬→ 감정 분석 ─┐
           ├→ 핵심어 추출 ├→ 결과 합치기
           └→ 위험 표현 ─┘</code></pre>
<p>병렬 처리는 전체 대기 시간을 줄일 수 있지만 호출 수와 비용을 줄이지 않는다. 무제한으로 동시에 호출하면 공급자의 rate limit을 만나기 쉽다.</p>
<p>AI 전용 스레드 풀 크기를 동시 호출 상한으로 사용한다.</p>
<pre><code class="language-java">@Bean(&quot;aiExecutor&quot;)
public Executor aiExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(4);
    executor.setMaxPoolSize(4);
    executor.setQueueCapacity(20);
    executor.initialize();
    return executor;
}</code></pre>
<h3 id="79-orchestrator-workers">7.9 Orchestrator-Workers</h3>
<p>오케스트레이터가 큰 목표를 작은 작업으로 나누고, 워커들이 각 작업을 수행한 뒤 결과를 합친다.</p>
<pre><code class="language-text">&quot;신입사원 온보딩 문서를 만들어라&quot;
              ↓
오케스트레이터: 회사 소개·개발 환경·보안 규정으로 분해
              ↓
세 워커가 각각 초안 작성
              ↓
결과 통합</code></pre>
<p>고정된 체인으로 표현하기 어려운 복합 작업에 유용하지만, 모델이 만든 작업 계획을 그대로 신뢰해서는 안 된다. 단계 수, 시간, 비용, 사용할 수 있는 기능을 제한한다.</p>
<h3 id="710-evaluator-optimizer">7.10 Evaluator-Optimizer</h3>
<p>생성자가 초안을 만들고 평가자가 기준에 따라 검사한 뒤 필요한 경우 한 번 더 고친다.</p>
<pre><code class="language-text">초안 생성 → 평가
              ├─ 통과 → 반환
              └─ 실패 → 피드백으로 수정 → 재평가</code></pre>
<pre><code class="language-java">for (int round = 0; round &lt; maxRounds; round++) {
    Evaluation evaluation = evaluate(draft);
    if (evaluation.passed()) {
        return draft;
    }
    draft = revise(draft, evaluation.feedback());
}
return draft;</code></pre>
<p>반드시 최대 반복 횟수를 둔다. 품질을 높일 수 있지만 생성과 평가 때문에 호출 비용이 두세 배가 될 수 있다.</p>
<h3 id="711-구조화-출력은-내용-검증이-아니다">7.11 구조화 출력은 내용 검증이 아니다</h3>
<p>객체 변환에 성공했다는 것은 데이터 모양이 맞다는 뜻일 뿐, 사실이 맞다는 뜻은 아니다.</p>
<pre><code class="language-text">형식 검증: rating이 int인가?
업무 검증: rating이 1~5인가?
근거 검증: 입력 후기 내용과 평가가 일치하는가?</code></pre>
<p>세 검증을 분리해야 한다.</p>
<h3 id="712-배치-처리">7.12 배치 처리</h3>
<p>대량 문서를 처리할 때는 한 요청 안에서 수백 건을 무제한 호출하지 않는다.</p>
<ul>
<li>입력을 적당한 배치로 나눈다.</li>
<li>동시 호출 수를 제한한다.</li>
<li>실패한 항목만 재실행할 수 있게 ID와 상태를 저장한다.</li>
<li>중간 결과를 저장해 전체를 처음부터 다시 하지 않는다.</li>
<li>비용과 처리량을 기록한다.</li>
</ul>
<hr />
<h2 id="8-8장---rag-기본">8. 8장 - RAG 기본</h2>
<h3 id="81-rag란-무엇인가">8.1 RAG란 무엇인가</h3>
<p>RAG는 Retrieval-Augmented Generation의 약자로, <strong>관련 자료를 검색한 뒤 그 자료를 넣어 답을 생성하는 방식</strong>이다.</p>
<p>닫힌 책 시험과 열린 책 시험으로 비교할 수 있다.</p>
<ul>
<li>일반 ChatModel: 학습 때 기억한 내용으로 답하는 닫힌 책 시험</li>
<li>RAG: 질문과 관련된 사내 문서를 찾아 펼쳐 놓고 답하는 열린 책 시험</li>
</ul>
<p>RAG가 필요한 이유는 다음과 같다.</p>
<ul>
<li>모델이 학습하지 않은 사내 문서를 사용할 수 있다.</li>
<li>최근에 바뀐 규정을 재학습 없이 반영할 수 있다.</li>
<li>답변에 근거와 출처를 함께 제시할 수 있다.</li>
<li>문서에서 확인되지 않은 질문을 거절하도록 만들 수 있다.</li>
</ul>
<p>RAG는 모델을 재학습시키는 기술이 아니다. 질문할 때 필요한 문서를 찾아 프롬프트에 추가하는 기술이다.</p>
<h3 id="82-전체-파이프라인">8.2 전체 파이프라인</h3>
<p>RAG는 크게 인덱싱과 질의 처리로 나뉜다.</p>
<h4 id="인덱싱-문서를-미리-준비하는-과정">인덱싱: 문서를 미리 준비하는 과정</h4>
<pre><code class="language-text">문서 읽기
  ↓
작은 조각으로 분할
  ↓
각 조각을 임베딩 벡터로 변환
  ↓
벡터와 원문·메타데이터 저장</code></pre>
<h4 id="질의-처리-질문할-때마다-실행하는-과정">질의 처리: 질문할 때마다 실행하는 과정</h4>
<pre><code class="language-text">사용자 질문
  ↓ 임베딩
질문 벡터
  ↓ 유사도 검색
관련 문서 조각
  ↓ 질문과 근거를 프롬프트에 결합
ChatModel
  ↓
근거가 있는 답변</code></pre>
<p>문서 인덱싱은 문서가 추가·변경될 때 수행하고, 검색과 생성은 사용자 질문마다 수행한다.</p>
<h3 id="83-documentreader">8.3 DocumentReader</h3>
<p><code>DocumentReader</code>는 PDF, Markdown, 텍스트 같은 원본을 Spring AI의 <code>Document</code> 목록으로 읽는다.</p>
<pre><code class="language-java">TextReader reader = new TextReader(resource);
List&lt;Document&gt; documents = reader.get();</code></pre>
<p>파일 형식에 따라 Tika 기반 Reader나 PDF Reader를 사용할 수 있다. 읽은 직후 텍스트가 비어 있거나 글자가 깨지지 않았는지 확인한다. 검색 결과가 전혀 없을 때 모델보다 먼저 Reader와 인제스트를 확인해야 한다.</p>
<h3 id="84-chunking">8.4 Chunking</h3>
<p>책 한 권 전체를 서점 직원에게 건네며 “반품 기한을 찾아 달라”고 하는 것보다 목차와 적당한 페이지 단위로 나누어 찾는 편이 빠르다. RAG의 chunk도 같은 역할을 한다.</p>
<p>청크가 너무 작으면 문장의 앞뒤 맥락이 끊긴다. 너무 크면 관련 없는 내용이 함께 들어가 검색 정확도와 토큰 효율이 나빠진다.</p>
<p>시작값의 예시는 다음과 같다.</p>
<table>
<thead>
<tr>
<th>문서 성격</th>
<th>시작 청크 크기</th>
</tr>
</thead>
<tbody><tr>
<td>FAQ</td>
<td>300~500 토큰</td>
</tr>
<tr>
<td>정책 문서</td>
<td>600~900 토큰</td>
</tr>
<tr>
<td>기술 문서</td>
<td>800~1,200 토큰</td>
</tr>
<tr>
<td>회의록</td>
<td>400~700 토큰</td>
</tr>
<tr>
<td>법률·계약</td>
<td>제목·조항 구조 기준</td>
</tr>
</tbody></table>
<p>겹침은 보통 10~20%부터 실험한다. 겹침은 문장이 경계에서 잘리는 문제를 줄이지만 중복 저장과 중복 검색을 늘린다.</p>
<p>중요한 것은 숫자를 외우는 것이 아니라 골든 세트로 조합을 비교하는 것이다.</p>
<h3 id="85-메타데이터">8.5 메타데이터</h3>
<p>메타데이터는 문서 조각에 붙이는 이름표다.</p>
<pre><code class="language-java">reader.getCustomMetadata().put(&quot;source&quot;, &quot;return-policy&quot;);
reader.getCustomMetadata().put(&quot;department&quot;, &quot;commerce&quot;);
reader.getCustomMetadata().put(&quot;version&quot;, &quot;2026-08&quot;);
reader.getCustomMetadata().put(&quot;validUntil&quot;, &quot;2026-12-31&quot;);</code></pre>
<p>메타데이터는 다음 용도로 사용한다.</p>
<ul>
<li>답변에 출처 표시</li>
<li>부서·등급·사용자 권한 필터</li>
<li>최신 버전만 검색</li>
<li>문서 삭제와 재색인</li>
<li>검색 실패 원인 추적</li>
</ul>
<p>메타데이터는 인제스트할 때 넣어야 한다. 답변을 만들 때 뒤늦게 원래 출처를 복원하려 하면 정확하지 않다.</p>
<h3 id="86-임베딩-모델-선택">8.6 임베딩 모델 선택</h3>
<p>임베딩 모델을 고를 때는 다음을 본다.</p>
<ul>
<li>벡터 차원</li>
<li>한국어와 업무 도메인의 검색 성능</li>
<li>한 번에 입력할 수 있는 길이</li>
<li>비용과 속도</li>
<li>로컬 실행 또는 외부 API 여부</li>
<li>장기간 같은 모델 버전을 유지할 수 있는가</li>
</ul>
<p>임베딩 모델을 바꾸면 기존 문서와 새 질문이 서로 다른 좌표 체계를 사용하게 된다. 같은 차원 수라고 해도 의미 공간은 다를 수 있으므로 일반적으로 모든 문서를 다시 임베딩하고 재색인해야 한다.</p>
<h3 id="87-vectorstore">8.7 VectorStore</h3>
<p><code>VectorStore</code>는 문서 벡터와 원문, 메타데이터를 저장하고 유사 문서를 찾는 추상화다.</p>
<pre><code class="language-java">vectorStore.add(chunks);

List&lt;Document&gt; results = vectorStore.similaritySearch(
        SearchRequest.builder()
                .query(question)
                .topK(4)
                .similarityThreshold(0.5)
                .build()
);</code></pre>
<p>저장소 선택지는 pgvector, Redis, Elasticsearch, Chroma, 관리형 벡터 DB 등이 있다.</p>
<ul>
<li>학습·단위 테스트: 단순 메모리 저장소</li>
<li>이미 PostgreSQL을 운영: pgvector 고려</li>
<li>키워드와 벡터 검색을 함께 사용: Elasticsearch 계열 고려</li>
<li>대규모 벡터 전용 운영: 관리형 서비스 고려</li>
</ul>
<p>HNSW와 IVF 같은 인덱스는 대량 벡터의 근사 최근접 검색을 빠르게 한다. 저장소의 벡터 차원 설정과 임베딩 모델 출력 차원이 일치해야 한다.</p>
<h3 id="88-etl과-documenttransformer">8.8 ETL과 DocumentTransformer</h3>
<p>인제스트는 데이터 처리 관점에서 ETL과 같다.</p>
<pre><code class="language-text">Extract   : Reader가 파일을 읽는다
Transform : 문서를 분할하고 요약·키워드·메타데이터를 보강한다
Load      : VectorStore에 저장한다</code></pre>
<p>변환 단계를 별도로 두면 청크 전략이나 메타데이터 규칙을 독립적으로 테스트할 수 있다. 문서가 갱신되면 같은 임베딩 모델과 변환 규칙으로 재색인하고, 모델을 바꾸면 전체 문서를 다시 임베딩한다.</p>
<h3 id="89-top-k·threshold·mmr">8.9 top-k·threshold·MMR</h3>
<h4 id="top-k">top-k</h4>
<p>가장 비슷한 문서 조각을 몇 개 가져올지 정한다.</p>
<ul>
<li>너무 작음: 필요한 근거를 놓침</li>
<li>너무 큼: 잡음, 토큰, 지연이 증가</li>
</ul>
<p>처음에는 3~5 정도에서 시작해 측정한다.</p>
<h4 id="similarity-threshold">similarity threshold</h4>
<p>점수가 너무 낮은 결과는 근거로 사용하지 않는다. 임계값이 너무 높으면 정답 문서도 거절하고, 너무 낮으면 관련 없는 문서가 들어간다. 실제 점수 분포를 먼저 관찰해야 한다.</p>
<h4 id="mmr">MMR</h4>
<p>Maximum Marginal Relevance는 비슷한 조각만 반복해서 가져오지 않고 관련성과 다양성을 함께 고려한다.</p>
<h3 id="810-메타데이터-필터는-보안-경계다">8.10 메타데이터 필터는 보안 경계다</h3>
<p>프롬프트에 “다른 부서 문서는 보여 주지 마”라고 적는 것만으로 권한을 보장할 수 없다. 검색 단계에서 애초에 권한 없는 문서가 결과에 들어오지 않게 해야 한다.</p>
<pre><code class="language-text">사용자 질문
  ↓
권한 필터: department=finance AND tier&lt;=userTier
  ↓
허용된 문서 안에서만 유사도 검색</code></pre>
<p>권한 필터는 프롬프트 기능이 아니라 데이터 접근 통제다.</p>
<h3 id="811-questionansweradvisor">8.11 QuestionAnswerAdvisor</h3>
<p>Spring AI의 Advisor는 ChatClient 호출 앞뒤에 공통 동작을 끼워 넣는다. RAG Advisor는 질문으로 관련 문서를 검색하고 프롬프트 컨텍스트에 넣는 반복 작업을 줄인다.</p>
<p>*<em>평범한 chatclient도 붙이면 RAG질문이 됨
*</em></p>
<p>개념적인 흐름은 다음과 같다.</p>
<pre><code class="language-java">ChatClient ragClient = builder
        .defaultAdvisors(QuestionAnswerAdvisor.builder(vectorStore).build())
        .build();</code></pre>
<p>Advisor가 편리해도 검색 결과, 점수, 메타데이터를 관찰할 수 있어야 한다. 처음 구현할 때는 <code>/retrieve</code> API로 검색을 따로 눈으로 확인하는 것이 좋다.</p>
<h3 id="812-대화-메모리와-rag">8.12 대화 메모리와 RAG</h3>
<p>대화 메모리는 이전 대화를 기억하고, RAG는 외부 문서를 검색한다.</p>
<pre><code class="language-text">대화 메모리: &quot;그 주문&quot;이 앞에서 말한 12345라는 사실
RAG: 회사의 현재 반품 규정이 7일이라는 사실</code></pre>
<p>둘은 목적이 다르며 함께 사용할 수 있다. 대화 ID를 사용자·세션별로 분리하지 않으면 다른 사용자의 대화가 섞일 수 있다.</p>
<h3 id="813-rag-실패-진단">8.13 RAG 실패 진단</h3>
<table>
<thead>
<tr>
<th>증상</th>
<th>먼저 볼 곳</th>
<th>가능한 원인</th>
</tr>
</thead>
<tbody><tr>
<td>검색 결과가 없음</td>
<td>인제스트·Reader</td>
<td>문서 미적재, 임베딩 설정 오류</td>
</tr>
<tr>
<td>엉뚱한 문서가 나옴</td>
<td>점수·청크·필터</td>
<td>청크 부적절, threshold 낮음</td>
</tr>
<tr>
<td>필요한 문서를 못 찾음</td>
<td>질문과 검색</td>
<td>표현 불일치, top-k 부족</td>
</tr>
<tr>
<td>근거는 맞는데 답이 틀림</td>
<td>프롬프트·모델</td>
<td>근거 사용 지시 부족</td>
</tr>
<tr>
<td>출처가 없음</td>
<td>메타데이터</td>
<td>인제스트 때 source 누락</td>
</tr>
<tr>
<td>같은 문서가 반복됨</td>
<td>적재 과정</td>
<td>재인제스트 중복, 청크 중복</td>
</tr>
<tr>
<td>응답이 느림</td>
<td>단계별 시간</td>
<td>top-k 과다, 인덱스 미설정</td>
</tr>
</tbody></table>
<p>RAG 장애의 진단 순서는 <strong>검색 → 프롬프트 → 모델</strong>이다. 모델부터 바꾸면 검색 문제를 숨기면서 비용만 늘 수 있다.</p>
<hr />
<h2 id="9-9장---rag-심화">9. 9장 - RAG 심화</h2>
<h3 id="91-개선-지점을-네-구간으로-나누기">9.1 개선 지점을 네 구간으로 나누기</h3>
<p>RAG는 다음 네 구간으로 나누어 개선한다.</p>
<pre><code class="language-text">Pre-Retrieval  : 질문을 검색하기 좋은 형태로 바꿈
Retrieval      : 후보 문서를 넓게 찾음
Post-Retrieval : 후보를 재정렬·압축함
Generation     : 선택된 근거로 답을 작성함</code></pre>
<p>실패한 질문이 어느 구간의 문제인지 먼저 분류한다.</p>
<p>Spring AI에서는 <code>RetrievalAugmentationAdvisor</code>를 중심으로 질문 변환기, 질의 확장기, 문서 검색기, 후처리기를 구간별로 조합할 수 있다. 특정 구현 하나에 모든 검색 로직을 넣지 않고 문제가 생긴 구간만 교체하는 것이 모듈형 RAG의 장점이다.</p>
<h3 id="92-query-rewrite">9.2 Query Rewrite</h3>
<p>사용자의 구어체나 대명사를 검색하기 쉬운 독립 질문으로 바꾼다.</p>
<pre><code class="language-text">원래 질문: &quot;그거 며칠 안에 보내야 돼?&quot;
대화 맥락: 반품 신청 이야기
변환 질문: &quot;단순 변심 반품 상품은 며칠 이내에 발송해야 하는가?&quot;</code></pre>
<p>변환 과정에서 원래 의도가 바뀔 수 있으므로 원문과 변환문을 함께 로그로 남긴다.</p>
<h3 id="93-translation">9.3 Translation</h3>
<p>문서가 영어이고 질문이 한국어라면 검색용 질문을 문서 언어로 번역할 수 있다. 다만 제품명, 법률 용어, 고유명사가 잘못 번역되지 않는지 평가해야 한다.</p>
<h3 id="94-multiquery">9.4 MultiQuery</h3>
<p>한 질문을 여러 표현으로 확장해 각각 검색하고 결과를 합친다.</p>
<pre><code class="language-text">원문: 물건을 돌려보내려면 언제까지 해야 하나요?

확장 1: 단순 변심 반품 가능 기간
확장 2: 반품 접수 기한
확장 3: 구매 후 교환·반품 일수</code></pre>
<p>표현 차이 때문에 놓친 문서를 찾을 가능성이 커지지만 검색 횟수와 중복 후보가 늘어난다.</p>
<h3 id="95-hyde">9.5 HyDE</h3>
<p>HyDE는 질문에 대한 가상의 답변 문서를 먼저 만든 뒤 그 답변을 임베딩해 검색한다.</p>
<pre><code class="language-text">질문
  ↓ 모델이 가상 답변 작성
가상 답변
  ↓ 임베딩·검색
실제 관련 문서</code></pre>
<p>질문은 짧지만 실제 문서는 설명형 문장일 때 표현 공간을 맞추는 데 도움이 될 수 있다. 반면 모델 호출이 하나 추가되고, 가상 답이 잘못된 방향이면 검색도 그 방향으로 치우친다.</p>
<h3 id="96-contextual-retrieval">9.6 Contextual Retrieval</h3>
<p>청크만 보면 “이 규정은 7일이다”가 무엇에 관한 규정인지 모를 수 있다. 인제스트할 때 문서 제목과 상위 문맥을 청크에 덧붙인다.</p>
<pre><code class="language-text">[문서: 반품 정책 / 항목: 단순 변심]
상품 수령 후 7일 이내 신청할 수 있다.</code></pre>
<p>검색 단서가 풍부해지지만 인제스트 비용과 저장량이 늘어난다.</p>
<h3 id="97-parent-child-retrieval">9.7 Parent-Child Retrieval</h3>
<p>작은 자식 청크로 정확하게 검색한 뒤, 모델에는 더 큰 부모 구간을 제공한다.</p>
<pre><code class="language-text">작은 청크 검색 → 청크가 속한 조항 전체 조회 → 모델 컨텍스트에 조항 제공</code></pre>
<p>검색의 정밀도와 답변의 충분한 맥락을 동시에 얻으려는 방법이다. 부모-자식 ID를 메타데이터로 연결해야 한다.</p>
<h3 id="98-reranking">9.8 Reranking</h3>
<p>벡터 검색으로 후보를 넓게 찾은 뒤 더 정교한 재순위 모델로 다시 정렬한다.</p>
<pre><code class="language-text">벡터 검색 top 20
        ↓ reranker
관련성 높은 top 4
        ↓
ChatModel 컨텍스트</code></pre>
<p>핵심 원칙은 <strong>넓게 찾고, 좁게 넣는 것</strong>이다. 모든 후보를 모델 프롬프트에 넣지 않아 토큰과 잡음을 줄인다.</p>
<h3 id="99-hybrid-search">9.9 Hybrid Search</h3>
<p>벡터 검색은 의미가 비슷한 표현에 강하고, 키워드 검색은 상품 코드, 버전, 사람 이름 같은 정확한 문자열에 강하다.</p>
<pre><code class="language-text">벡터 검색 결과 ─┐
                 ├→ 점수 결합 또는 순위 결합 → 최종 후보
키워드 검색 결과 ┘</code></pre>
<p>두 방법을 결합하면 “의미는 비슷하지만 특정 코드가 반드시 일치해야 하는” 업무 문서에 유리하다.</p>
<h3 id="910-agentic-rag">9.10 Agentic RAG</h3>
<p>일반 RAG는 정해진 순서로 한 번 검색한다. Agentic RAG는 모델이 검색 도구를 사용해 부족한 정보를 판단하고 여러 번 검색한다.</p>
<pre><code class="language-text">질문 → 첫 검색 → 정보 부족 판단 → 검색어 수정 → 두 번째 검색 → 답변</code></pre>
<p>VectorStore 검색을 Tool로 제공할 수 있다. 유연하지만 호출 횟수, 지연, 비용, 예상하지 못한 탐색이 늘 수 있으므로 최대 단계, 시간, 허용 도구, 권한을 제한한다.</p>
<h3 id="911-graphrag">9.11 GraphRAG</h3>
<p>GraphRAG는 사람, 조직, 계약, 제품처럼 관계가 중요한 데이터를 그래프로 연결해 탐색한다.</p>
<p>예를 들어 “A회사와 연결된 계약 중 B제품에 영향을 주는 규정”처럼 여러 관계를 따라가야 하는 질문에 유용하다. 단순 FAQ나 정책 검색에 처음부터 GraphRAG를 도입하면 구축·운영 복잡성이 더 클 수 있다.</p>
<h3 id="912-rag와-fine-tuning-선택">9.12 RAG와 Fine-tuning 선택</h3>
<table>
<thead>
<tr>
<th>요구 사항</th>
<th>우선 고려</th>
</tr>
</thead>
<tbody><tr>
<td>최신 정보·사내 문서·출처</td>
<td>RAG</td>
</tr>
<tr>
<td>말투·행동·반복되는 출력 습관</td>
<td>Fine-tuning</td>
</tr>
<tr>
<td>간단한 지시와 형식</td>
<td>Prompt</td>
</tr>
</tbody></table>
<p>일반적인 검토 순서는 다음과 같다.</p>
<pre><code class="language-text">Prompt로 해결 가능한가?
    ↓ 아니면
RAG로 필요한 지식을 줄 수 있는가?
    ↓ 아니면
Fine-tuning이 필요한 행동 문제인가?</code></pre>
<p>RAG와 Fine-tuning은 경쟁 관계가 아니며 필요하면 함께 사용할 수 있다.</p>
<h3 id="913-rag-평가">9.13 RAG 평가</h3>
<p>검색과 생성을 따로 평가해야 고칠 위치를 알 수 있다.</p>
<h4 id="검색-평가">검색 평가</h4>
<ul>
<li>Recall@k: 필요한 근거가 상위 k개 안에 들어왔는가?</li>
<li>Precision@k: 가져온 k개 중 실제 관련 근거의 비율은 얼마인가?</li>
</ul>
<h4 id="생성-평가">생성 평가</h4>
<ul>
<li>Faithfulness: 답이 제공된 근거에서 벗어나지 않았는가?</li>
<li>Answer Relevance: 사용자의 질문에 직접 답했는가?</li>
<li>Citation Correctness: 표시한 출처가 실제 사용한 근거인가?</li>
<li>Refusal Accuracy: 문서에 없는 질문을 올바르게 거절했는가?</li>
</ul>
<p>예를 들어 정답 문서가 검색 결과에 없으면 생성 모델을 바꾸기보다 검색을 고쳐야 한다. 정답 문서가 있었는데 답을 틀렸다면 프롬프트, 근거 포맷, 모델을 본다.</p>
<h3 id="914-개선-실험-원칙">9.14 개선 실험 원칙</h3>
<ol>
<li>실패 질문 하나를 고른다.</li>
<li>검색 실패인지 생성 실패인지 구분한다.</li>
<li>chunk, top-k, threshold, HyDE, rerank 중 하나만 바꾼다.</li>
<li>같은 골든 세트로 다시 측정한다.</li>
<li>좋아진 수치와 나빠진 수치를 함께 기록한다.</li>
</ol>
<p>여러 기능을 한꺼번에 추가하면 무엇 때문에 좋아지거나 나빠졌는지 알 수 없다.</p>
<hr />
<h2 id="11-실행과-swagger-검증">11. 실행과 Swagger 검증</h2>
<h3 id="111-api-키-등록">11.1 API 키 등록</h3>
<p>macOS 또는 Linux 터미널에서 현재 셸에만 등록한다.</p>
<pre><code class="language-bash">export OPENAI_API_KEY=&quot;발급받은_API_KEY&quot;</code></pre>
<p>확인은 실제 값을 출력하지 않고 등록 여부만 본다.</p>
<pre><code class="language-bash">if [ -n &quot;$OPENAI_API_KEY&quot; ]; then
  echo &quot;OPENAI_API_KEY가 등록되어 있습니다.&quot;
else
  echo &quot;OPENAI_API_KEY가 없습니다.&quot;
fi</code></pre>
<p>키를 소스 코드, <code>application.yml</code>, Git 저장소에 직접 넣지 않는다. 키를 바꾼 뒤에는 실행 중인 애플리케이션을 재시작해야 한다.</p>
<h3 id="112-서버-실행">11.2 서버 실행</h3>
<pre><code class="language-bash">./gradlew bootRun</code></pre>
<p>Gradle 화면이 <code>80% EXECUTING</code>과 <code>IDLE</code>을 표시해도 다음 로그가 있고 프로세스가 끝나지 않았다면 설치 중이 아니라 서버가 요청을 기다리는 정상 상태다.</p>
<pre><code class="language-text">Started ...Application
Tomcat started on port 8080</code></pre>
<h3 id="113-swagger-ui">11.3 Swagger UI</h3>
<pre><code class="language-text">http://localhost:8080/swagger-ui.html</code></pre>
<p>프로젝트 설정에 따라 다음 주소일 수도 있다.</p>
<pre><code class="language-text">http://localhost:8080/swagger-ui/index.html</code></pre>
<p>Swagger에서 다음 순서로 확인한다.</p>
<ol>
<li><code>POST /lab2/ingest</code>로 문서를 적재한다.</li>
<li><code>GET /lab2/retrieve</code>로 검색 결과와 점수를 본다.</li>
<li><code>POST /lab2/ask</code>로 근거 있는 답변을 확인한다.</li>
<li>문서에 없는 질문이 <code>grounded=false</code>인지 확인한다.</li>
<li>인제스트를 다시 실행해 청크 수가 늘지 않는지 확인한다.</li>
</ol>
<h3 id="114-기본-주소가-오류인-이유">11.4 기본 주소가 오류인 이유</h3>
<p><code>http://localhost:8080</code>에 <code>/</code> Controller가 없다면 <code>404</code>가 정상일 수 있다. 서버가 꺼졌다는 뜻은 아니다.</p>
<p>다음 주소로 각각 확인한다.</p>
<pre><code class="language-bash">curl -i 'http://localhost:8080/swagger-ui.html'
curl -i 'http://localhost:8080/v3/api-docs'
curl -i 'http://localhost:8080/lab2/retrieve?q=반품기한'</code></pre>
<hr />
<h2 id="12-자주-발생하는-오류와-해결-방법">12. 자주 발생하는 오류와 해결 방법</h2>
<h3 id="121-401-unauthorized">12.1 401 Unauthorized</h3>
<p>원인:</p>
<ul>
<li>API 키 미등록, 오타, 만료</li>
<li>현재 터미널이 아닌 다른 터미널에 키 등록</li>
<li>키 변경 후 서버를 재시작하지 않음</li>
</ul>
<p>해결:</p>
<pre><code class="language-bash">export OPENAI_API_KEY=&quot;새_API_KEY&quot;
./gradlew bootRun</code></pre>
<h3 id="122-429-too-many-requests">12.2 429 Too Many Requests</h3>
<p>두 종류를 구분한다.</p>
<ul>
<li>Maven Central 429: 의존성 다운로드 서버의 요청 제한</li>
<li>모델 API 429: 호출량, 동시성, 계정 한도 제한</li>
</ul>
<p>Maven Central 문제라면 이미 받은 캐시로 실행해 본다.</p>
<pre><code class="language-bash">./gradlew --offline bootRun</code></pre>
<p>모델 API 문제라면 병렬 호출 수를 낮추고, 제한된 재시도와 지수 백오프를 적용하며, 공급자 사용량과 결제 상태를 확인한다.</p>
<h3 id="123-포트-8080-충돌">12.3 포트 8080 충돌</h3>
<pre><code class="language-text">Web server failed to start. Port 8080 was already in use.</code></pre>
<p>앞서 실행한 Day 2 프로젝트를 <code>Control+C</code>로 종료하거나 다른 포트를 사용한다.</p>
<pre><code class="language-bash">./gradlew bootRun --args='--server.port=8081'</code></pre>
<h3 id="124-검색-결과가-항상-비어-있음">12.4 검색 결과가 항상 비어 있음</h3>
<p>확인 순서:</p>
<ol>
<li>인제스트 API를 실행했는가?</li>
<li>Reader가 실제 텍스트를 읽었는가?</li>
<li>청크 수가 0이 아닌가?</li>
<li>임베딩 모델과 API 키가 설정되었는가?</li>
<li>threshold가 지나치게 높지 않은가?</li>
<li>권한·source 필터가 결과를 모두 제외하지 않는가?</li>
</ol>
<h3 id="125-인제스트할수록-품질이-나빠짐">12.5 인제스트할수록 품질이 나빠짐</h3>
<p>동일 문서가 반복 저장되었을 가능성이 크다. source별 삭제 후 재삽입하거나 고정 ID를 사용해 upsert한다.</p>
<h3 id="126-근거는-맞는데-답이-틀림">12.6 근거는 맞는데 답이 틀림</h3>
<p>검색 문제보다 생성 문제에 가깝다.</p>
<ul>
<li>근거만 사용하라는 System 지시가 있는가?</li>
<li>모르면 거절하라는 규칙이 있는가?</li>
<li>문서 본문과 출처가 구분되어 전달되는가?</li>
<li>구조화 응답의 필드 설명이 명확한가?</li>
</ul>
<h3 id="127-항상-확인되지-않습니다라고-답함">12.7 항상 “확인되지 않습니다”라고 답함</h3>
<p>threshold가 지나치게 높거나 검색 결과를 빈 목록으로 만드는 필터가 있을 수 있다. 먼저 <code>/lab2/retrieve</code>에서 질문별 점수 분포를 확인하고 조절한다.</p>
<h3 id="128-출처가-표시되지-않음">12.8 출처가 표시되지 않음</h3>
<p>답변 단계가 아니라 인제스트 단계에서 <code>source</code> 메타데이터를 넣었는지 확인한다.</p>
<h3 id="129-컨텍스트-길이-초과-또는-토큰-급증">12.9 컨텍스트 길이 초과 또는 토큰 급증</h3>
<ul>
<li>top-k를 줄인다.</li>
<li>청크 크기를 줄인다.</li>
<li>중복 청크를 제거한다.</li>
<li>rerank 후 소수 후보만 넣는다.</li>
<li>긴 근거를 요약하거나 필요한 구간만 추린다.</li>
</ul>
<h3 id="1210-구조화-출력-변환-실패">12.10 구조화 출력 변환 실패</h3>
<ul>
<li><code>finishReason=length</code>인지 확인한다.</li>
<li><code>maxTokens</code>를 충분히 준다.</li>
<li>record 필드와 enum 설명을 명확히 한다.</li>
<li>코드 펜스를 제거한 뒤 한 번 복구한다.</li>
<li>낮은 temperature로 제한된 재시도를 한다.</li>
<li>끝까지 실패하면 안전한 기본 객체를 반환한다.</li>
</ul>
<hr />
<h2 id="14-스스로-답해-볼-핵심-질문">14. 스스로 답해 볼 핵심 질문</h2>
<h3 id="좋은-프롬프트의-네-요소는-무엇인가">좋은 프롬프트의 네 요소는 무엇인가?</h3>
<p>역할, 맥락, 구체적인 지시, 출력 형식이다.</p>
<h3 id="call과-stream의-차이는-무엇인가"><code>call()</code>과 <code>stream()</code>의 차이는 무엇인가?</h3>
<p><code>call()</code>은 완성된 응답을 한 번에 받고 <code>stream()</code>은 생성 중인 응답을 여러 조각으로 받는다.</p>
<h3 id="구조화-출력이면-검증이-필요-없을까">구조화 출력이면 검증이 필요 없을까?</h3>
<p>아니다. 타입 변환에 성공해도 숫자 범위, enum, 사실성, 근거 사용 여부는 별도로 검증해야 한다.</p>
<h3 id="임베딩-모델은-답변을-생성할까">임베딩 모델은 답변을 생성할까?</h3>
<p>아니다. 텍스트를 벡터로 바꾸어 의미가 가까운 문서를 찾는 데 사용한다.</p>
<h3 id="rag와-모델-재학습은-같은가">RAG와 모델 재학습은 같은가?</h3>
<p>아니다. RAG는 질문 시점에 관련 자료를 검색해 프롬프트에 추가한다.</p>
<h3 id="top-k를-크게-하면-항상-좋아질까">top-k를 크게 하면 항상 좋아질까?</h3>
<p>아니다. 정답을 찾을 가능성과 함께 잡음, 토큰, 지연도 늘어난다.</p>
<h3 id="문서-권한을-프롬프트로-통제해도-될까">문서 권한을 프롬프트로 통제해도 될까?</h3>
<p>안 된다. 권한 없는 문서가 검색되지 않도록 메타데이터 필터에서 강제해야 한다.</p>
<h3 id="rag-답이-틀리면-가장-먼저-모델을-바꿔야-할까">RAG 답이 틀리면 가장 먼저 모델을 바꿔야 할까?</h3>
<p>아니다. 먼저 검색 결과에 올바른 근거가 있었는지 확인해 검색 실패와 생성 실패를 구분해야 한다.</p>
<h3 id="골든-세트에-답이-없는-질문이-필요한-이유는-무엇인가">골든 세트에 답이 없는 질문이 필요한 이유는 무엇인가?</h3>
<p>모델이 근거가 없을 때 추측하지 않고 올바르게 거절하는지 검사하기 위해서다.</p>
<h3 id="재인제스트-중복을-막아야-하는-이유는-무엇인가">재인제스트 중복을 막아야 하는 이유는 무엇인가?</h3>
<p>같은 청크가 반복 검색되어 결과 다양성과 품질이 떨어지고 저장량과 비용도 증가하기 때문이다.</p>
<hr />
<h2 id="15-최종-정리">15. 최종 정리</h2>
<p>Day 2의 각 장은 별개의 기능처럼 보이지만 하나의 흐름으로 연결된다.</p>
<ul>
<li>프롬프트는 AI가 해야 할 일을 명확하게 만든다.</li>
<li>옵션은 답변의 다양성과 길이를 제어한다.</li>
<li>스트리밍은 첫 응답 체감 시간을 줄인다.</li>
<li>구조화 출력은 AI 응답을 Java 객체 계약으로 바꾼다.</li>
<li>멀티모달은 텍스트 이외의 입력을 처리한다.</li>
<li>임베딩은 문장 의미를 벡터로 바꾼다.</li>
<li>VectorStore는 질문과 가까운 문서 조각을 찾는다.</li>
<li>RAG는 검색된 문서를 근거로 답하게 한다.</li>
<li>고급 RAG는 질문 변환, 하이브리드 검색, rerank로 검색 품질을 높인다.</li>
<li>골든 세트는 변경 전후의 품질을 수치로 비교하게 한다.</li>
</ul>
<p>가장 중요한 원칙은 다음 세 문장이다.</p>
<blockquote>
<p>답변을 보기 전에 검색 결과를 본다.</p>
</blockquote>
<blockquote>
<p>검색은 넓게 하고 모델에 넣는 근거는 좁게 고른다.</p>
</blockquote>
<blockquote>
<p>검색 실패와 생성 실패를 분리해 한 번에 하나씩 개선한다.</p>
</blockquote>
<p>이 원칙을 지키면 “그럴듯하게 말하는 챗봇”을 넘어, 출처를 보여 주고 모르는 것은 거절하며 품질 변화를 측정할 수 있는 업무용 AI로 발전시킬 수 있다.</p>