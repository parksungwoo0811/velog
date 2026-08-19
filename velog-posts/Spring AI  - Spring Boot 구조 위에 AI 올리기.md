<blockquote>
<p>용어 정리 → Spring Boot 기초 → Spring AI 개요와 아키텍처 → 개발환경 구성 → 의존성·설정과 <code>ChatClient</code> </p>
<p>오늘의 핵심: <strong>AI부터 호출하지 않는다. 먼저 Spring Boot의 계층을 세우고, 그 위에 AI를 하나의 부품처럼 올린다.</strong></p>
</blockquote>
<hr />
<h2 id="오늘-무엇을-배우는가">오늘 무엇을 배우는가</h2>
<p>오늘의 흐름은 다음 한 줄로 요약할 수 있다.</p>
<pre><code class="language-text">웹 요청 → Controller → Service → Repository / ChatClient → DTO 응답</code></pre>
<hr />
<h1 id="part-0-오리엔테이션과-용어-정리">Part 0. 오리엔테이션과 용어 정리</h1>
<p>처음부터 모든 용어를 외울 필요는 없다. 오늘은 각 용어가 <strong>어느 문제를 해결하는 말인지</strong>를 구분하는 것이 목표다.</p>
<h2 id="0-1-llm-기본-용어">0-1. LLM 기본 용어</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 뜻</th>
<th>왜 중요한가</th>
</tr>
</thead>
<tbody><tr>
<td>토큰(Token)</td>
<td>모델이 글을 읽고 쓰는 조각</td>
<td>입력·출력 길이, 비용, 속도의 기준이다</td>
</tr>
<tr>
<td>컨텍스트 윈도우</td>
<td>한 요청에 담을 수 있는 전체 토큰의 한도</td>
<td>넘치면 앞의 대화나 정보가 잘릴 수 있다</td>
</tr>
<tr>
<td>프롬프트</td>
<td>모델에게 전달하는 업무 지시서 전체</td>
<td>역할, 지시, 맥락, 예시, 출력 형식을 담는다</td>
</tr>
<tr>
<td>시스템 메시지</td>
<td>모든 요청에 적용할 역할과 규칙</td>
<td>사용자 질문과 분리해야 규칙을 관리하기 쉽다</td>
</tr>
<tr>
<td><code>temperature</code></td>
<td>답변의 무작위성을 조절하는 값</td>
<td>분류·추출은 낮게, 창작은 비교적 높게 둔다</td>
</tr>
<tr>
<td><code>top-p</code></td>
<td>다음 토큰 후보의 확률 범위를 제한하는 값</td>
<td>보통 <code>temperature</code>와 동시에 무리하게 조절하지 않는다</td>
</tr>
<tr>
<td>할루시네이션</td>
<td>모르는 내용을 그럴듯하게 지어내는 현상</td>
<td>근거 제공, 출처 표시, 모르면 모른다는 규칙이 필요하다</td>
</tr>
<tr>
<td>파인튜닝</td>
<td>모델의 가중치를 추가 학습하는 방법</td>
<td>최신 지식 주입보다는 말투·형식·행동 패턴에 효과적이다</td>
</tr>
<tr>
<td>멀티모달</td>
<td>텍스트 외 이미지·음성 등을 함께 처리하는 능력</td>
<td>사용할 모델이 해당 입력 형식을 지원해야 한다</td>
</tr>
</tbody></table>
<h3 id="llm은-어떻게-답하는가">LLM은 어떻게 답하는가</h3>
<p>LLM은 정답 데이터베이스에서 문장을 꺼내는 프로그램이 아니다. 지금까지 받은 문맥을 보고 <strong>다음에 올 토큰의 확률을 계산하여 하나씩 이어 붙이는 프로그램</strong>에 가깝다.</p>
<p>그래서 다음 특성이 자연스럽게 따라온다.</p>
<ul>
<li>같은 질문에도 답이 조금씩 달라질 수 있다.</li>
<li>문장은 자연스러워도 사실은 틀릴 수 있다.</li>
<li>모델이 이전 대화를 스스로 기억하는 것이 아니다. 필요한 대화 이력을 요청마다 다시 보내야 한다.</li>
<li>대화가 길수록 입력 토큰, 비용, 지연 시간이 함께 늘어난다.</li>
</ul>
<h2 id="0-2-임베딩과-벡터-검색">0-2. 임베딩과 벡터 검색</h2>
<p>임베딩의 핵심은 단 한 문장이다.</p>
<blockquote>
<p>뜻이 가까운 문장은 숫자 공간에서도 가까운 위치에 놓인다.</p>
</blockquote>
<table>
<thead>
<tr>
<th>용어</th>
<th>뜻</th>
<th>실무 포인트</th>
</tr>
</thead>
<tbody><tr>
<td>임베딩(Embedding)</td>
<td>문장을 의미를 담은 숫자 배열로 변환한 것</td>
<td>의미 검색과 RAG의 재료다</td>
</tr>
<tr>
<td>차원(Dimension)</td>
<td>임베딩 벡터의 길이</td>
<td>모델별로 다르며 모델 변경 시 기존 데이터를 다시 색인해야 한다</td>
</tr>
<tr>
<td>코사인 유사도</td>
<td>두 벡터의 방향이 얼마나 비슷한지 나타내는 값</td>
<td>일반적으로 1에 가까울수록 비슷하다</td>
</tr>
<tr>
<td>벡터 DB</td>
<td>벡터, 원문, 메타데이터를 함께 저장·검색하는 저장소</td>
<td>pgvector, Chroma, Qdrant 등이 있다</td>
</tr>
<tr>
<td>청킹(Chunking)</td>
<td>긴 문서를 검색 단위로 나누는 작업</td>
<td>너무 작으면 맥락이 끊기고 너무 크면 잡음과 비용이 늘어난다</td>
</tr>
<tr>
<td>겹침(Overlap)</td>
<td>이웃 청크가 일부 문장을 공유하게 하는 것</td>
<td>문장 경계에서 근거가 잘리는 문제를 줄인다</td>
</tr>
<tr>
<td><code>top-k</code></td>
<td>검색 결과로 가져올 청크 수</td>
<td>너무 크면 관계없는 근거와 토큰 비용이 늘어난다</td>
</tr>
<tr>
<td>MMR</td>
<td>유사도뿐 아니라 결과의 다양성도 보는 검색 방식</td>
<td>비슷한 문서만 몰려 나오는 것을 줄인다</td>
</tr>
<tr>
<td>HNSW·IVF</td>
<td>대량의 벡터를 빠르게 찾기 위한 인덱스</td>
<td>데이터가 커지면 검색 성능을 위해 필요하다</td>
</tr>
</tbody></table>
<p>예를 들어 사용자가 <code>휴가 내는 법</code>이라고 물었는데 회사 문서에는 <code>연차 신청 절차</code>라고 적혀 있어도, 두 문장의 의미 벡터가 가까우면 관련 문서를 찾을 수 있다. 이것이 단순 키워드 검색과 의미 검색의 큰 차이다.</p>
<h2 id="0-3-rag-용어">0-3. RAG 용어</h2>
<p><strong>RAG(Retrieval-Augmented Generation)</strong>는 검색한 근거를 프롬프트에 붙여 모델이 그 근거를 바탕으로 답하게 하는 방식이다.</p>
<pre><code class="language-text">사전 작업: 문서 읽기 → 청킹 → 임베딩 → VectorStore 저장
질문 처리: 질문 임베딩 → 관련 청크 검색 → 근거와 질문 결합 → LLM 답변</code></pre>
<table>
<thead>
<tr>
<th>용어</th>
<th>뜻</th>
<th>기억할 점</th>
</tr>
</thead>
<tbody><tr>
<td>인제스트(Ingest)</td>
<td>문서를 읽고, 나누고, 임베딩해 저장하는 과정</td>
<td>보통 질문 전에 배치로 수행한다</td>
</tr>
<tr>
<td>리트리버(Retriever)</td>
<td>질문과 관련된 문서 조각을 찾는 부품</td>
<td>Spring AI에서는 <code>VectorStore</code>나 Advisor가 담당할 수 있다</td>
</tr>
<tr>
<td>근거·출처</td>
<td>답변의 바탕이 된 문서와 위치</td>
<td>출처 없는 RAG는 검증하기 어렵다</td>
</tr>
<tr>
<td>재순위(Rerank)</td>
<td>검색 후보를 다시 평가해 좋은 결과를 위로 올리는 작업</td>
<td>검색 품질 개선 효과가 큰 편이다</td>
</tr>
<tr>
<td>하이브리드 검색</td>
<td>키워드 검색과 의미 검색을 결합하는 방식</td>
<td>제품 코드·고유명사와 자연어 의미를 함께 찾을 때 유리하다</td>
</tr>
<tr>
<td>필터(Filter)</td>
<td>메타데이터 조건으로 검색 범위를 제한하는 것</td>
<td>사용자 권한은 프롬프트가 아니라 필터·코드로 강제한다</td>
</tr>
<tr>
<td>재현율·정밀도</td>
<td>필요한 근거를 찾았는가 / 좋은 근거가 상위에 왔는가</td>
<td>답이 틀리면 먼저 검색 결과부터 직접 확인한다</td>
</tr>
<tr>
<td><code>QuestionAnswerAdvisor</code></td>
<td>검색과 프롬프트 결합을 도와주는 Spring AI 부품</td>
<td>RAG의 반복 조립 코드를 줄여 준다</td>
</tr>
</tbody></table>
<p>RAG 답변이 틀렸다면 프롬프트부터 고치지 말고 <strong>검색된 청크 안에 답이 실제로 있는지</strong> 먼저 본다. 검색 결과에 근거가 없다면 모델은 올바른 답을 만들 재료가 없다.</p>
<h2 id="0-4-도구와-에이전트">0-4. 도구와 에이전트</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>뜻</th>
<th>안전 기준</th>
</tr>
</thead>
<tbody><tr>
<td>Tool Calling</td>
<td>모델이 어떤 함수를 어떤 인자로 호출할지 선택하는 기능</td>
<td>모델은 선택하고, 실제 실행은 우리 코드가 한다</td>
</tr>
<tr>
<td>도구 스키마</td>
<td>모델에게 보이는 함수 이름·설명·파라미터</td>
<td>설명이 모호하면 엉뚱한 도구를 호출한다</td>
</tr>
<tr>
<td>ReAct</td>
<td>판단과 행동을 번갈아 수행하는 패턴</td>
<td>도구 결과를 본 뒤 다음 행동을 결정한다</td>
</tr>
<tr>
<td>에이전트 루프</td>
<td>도구 호출과 판단을 반복하는 흐름</td>
<td>횟수·시간·비용 상한이 반드시 필요하다</td>
</tr>
<tr>
<td>MCP</td>
<td>AI와 외부 도구를 연결하는 표준 규약</td>
<td>도구를 소비하거나 외부에 제공할 수 있다</td>
</tr>
<tr>
<td>승인 게이트·HITL</td>
<td>위험한 행동 전에 사람이 승인하는 절차</td>
<td>환불·발송·삭제처럼 되돌리기 어려운 작업에 필요하다</td>
</tr>
<tr>
<td>가드레일</td>
<td>입력과 출력에 적용하는 안전 규칙</td>
<td>차단 처리는 메모리 저장보다 먼저 수행한다</td>
</tr>
<tr>
<td>프롬프트 인젝션</td>
<td>기존 지시를 무시하게 유도하는 공격</td>
<td>문서 속에 숨은 간접 인젝션도 고려해야 한다</td>
</tr>
<tr>
<td>멀티 에이전트</td>
<td>역할을 나눈 여러 에이전트의 협업</td>
<td>먼저 단일 도구와 권한 경계를 명확히 해야 한다</td>
</tr>
</tbody></table>
<p>가장 중요한 원칙은 다음과 같다.</p>
<blockquote>
<p>모델이 행동을 제안하더라도 권한을 검증하고 실제 행동을 수행하는 주체는 우리 코드다.</p>
</blockquote>
<p>사용자 ID를 프롬프트에 적어 놓고 모델이 지켜 주기를 기대하면 안 된다. 모델은 값을 착각하거나 만들어 낼 수 있으므로 도구 내부에서 인증 정보와 권한을 다시 검증해야 한다.</p>
<h2 id="0-5-spring-ai-구현-용어">0-5. Spring AI 구현 용어</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>역할</th>
<th>실무 포인트</th>
</tr>
</thead>
<tbody><tr>
<td><code>ChatClient</code></td>
<td>모델 호출의 표준 진입점</td>
<td>빌더에 공통 기본값을 미리 넣는다</td>
</tr>
<tr>
<td><code>Advisor</code></td>
<td>요청·응답에 공통 처리를 끼우는 장치</td>
<td>순서가 정책이다. 차단은 저장보다 앞에 둔다</td>
</tr>
<tr>
<td><code>VectorStore</code></td>
<td>벡터 저장·검색 추상화</td>
<td>인메모리에서 pgvector로 구현을 교체하기 쉽다</td>
</tr>
<tr>
<td><code>DocumentReader</code>·<code>Splitter</code></td>
<td>문서 읽기와 분할</td>
<td>인제스트 파이프라인의 앞부분이다</td>
</tr>
<tr>
<td><code>ChatMemory</code></td>
<td>대화 이력을 저장하고 다시 주입</td>
<td>대화 ID 규칙을 한곳에서 관리한다</td>
</tr>
<tr>
<td>구조화 출력</td>
<td>문자열이 아닌 Java 객체로 응답 받기</td>
<td>직접 문자열을 파싱하지 않고 <code>.entity(Class)</code>를 쓴다</td>
</tr>
<tr>
<td>Starter·BOM</td>
<td>자동 구성 의존성·버전 정렬</td>
<td>Spring AI 버전은 BOM 한곳에서 맞춘다</td>
</tr>
<tr>
<td><code>ToolContext</code></td>
<td>도구에 안전하게 전달할 실행 문맥</td>
<td>사용자 ID 같은 신뢰 정보는 프롬프트가 아니라 이 경로로 넘긴다</td>
</tr>
<tr>
<td>Profile</td>
<td>개발·운영 환경별 설정 분리</td>
<td>코드가 아니라 설정을 교체한다</td>
</tr>
</tbody></table>
<p>교재의 용어 정리 표지는 <code>운영·품질·보안</code>도 범위로 안내한다. 별도 상세 표는 없지만, 관련 개념인 토큰·지연 측정, RAG 검색 품질, 권한 필터, 가드레일, 인젝션, 프로파일은 각 파트에서 계속 다룬다.</p>
<hr />
<h1 id="part-1-spring-boot-기초---어노테이션과-계층-구조">Part 1. Spring Boot 기초 - 어노테이션과 계층 구조</h1>
<h2 id="1-1-왜-계층을-나누는가">1-1. 왜 계층을 나누는가</h2>
<p>계층 분리는 코드를 멋있게 보이게 하려는 규칙이 아니다. <strong>변경의 파급을 한 역할 안에 가두기 위한 구조</strong>다.</p>
<p>택배 회사로 비유해 보자.</p>
<table>
<thead>
<tr>
<th>비유</th>
<th>Spring 계층</th>
<th>책임</th>
</tr>
</thead>
<tbody><tr>
<td>주문 접수 창구</td>
<td>Controller(화면)</td>
<td>HTTP 요청을 받고 검증한 뒤 응답한다</td>
</tr>
<tr>
<td>업무 처리 담당자</td>
<td>Service(정책)</td>
<td>업무 순서와 규칙을 결정한다</td>
</tr>
<tr>
<td>창고 관리자</td>
<td>Repository·Mapper(저장소)</td>
<td>DB나 외부 시스템에서 데이터를 가져온다</td>
</tr>
<tr>
<td>창구의 서류 양식</td>
<td>DTO</td>
<td>계층 밖으로 주고받을 데이터 모양을 정한다</td>
</tr>
</tbody></table>
<p>핵심 규칙은 두 가지다.</p>
<ol>
<li>각 계층은 자기 책임에 집중한다.
__2. 호출은 위에서 아래로 흐른다. <code>Controller → Service → Repository</code>
__</li>
</ol>
<p>Controller가 Repository를 직접 호출하거나 Repository가 Service를 다시 호출하면 경계가 무너진다.</p>
<h2 id="1-2-요청-한-번의-여정">1-2. 요청 한 번의 여정</h2>
<p><code>GET /orders/12345?userId=user1</code> 요청을 예로 들면 다음 순서로 흐른다.</p>
<pre><code class="language-text">1. Controller: URL·쿼리 파라미터를 Java 값으로 변환하고 입력 검증
2. Service: 주문 조회라는 업무 흐름과 권한 조건 결정
3. Repository/Mapper: DB에서 조건에 맞는 데이터 조회
4. DTO 변환: Entity의 내부 필드를 제외하고 응답 모양 생성
5. Controller: DTO를 JSON으로 직렬화하여 200 OK 반환</code></pre>
<p>오류도 계층마다 다른 얼굴을 가진다.</p>
<ul>
<li><code>400 Bad Request</code>: 입력 형식·검증 문제</li>
<li><code>404 Not Found</code>: 업무 규칙상 자원을 찾지 못함</li>
<li><code>500 Internal Server Error</code>: DB 연결·SQL·예상하지 못한 내부 오류</li>
<li>값이 <code>null</code>: DTO 변환에서 필드를 누락했을 가능성</li>
</ul>
<h2 id="1-3-어노테이션-지도">1-3. 어노테이션 지도</h2>
<p>어노테이션은 스프링에게 의도를 알리는 표지다. 실제로 객체를 만들고 요청을 연결하는 일은 스프링이 한다.</p>
<table>
<thead>
<tr>
<th>목적</th>
<th>대표 어노테이션</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Bean 등록</td>
<td><code>@Component</code>, <code>@Service</code>, <code>@Repository</code>, <code>@Controller</code></td>
<td>클래스를 스프링 관리 객체로 등록한다</td>
</tr>
<tr>
<td>HTTP 매핑</td>
<td><code>@RestController</code>, <code>@GetMapping</code>, <code>@PostMapping</code></td>
<td>HTTP 요청을 메서드에 연결한다</td>
</tr>
<tr>
<td>의존성 주입</td>
<td>생성자 주입, <code>@Autowired</code>, <code>@Qualifier</code></td>
<td>필요한 Bean을 찾아 넣는다</td>
</tr>
<tr>
<td>설정</td>
<td><code>@Configuration</code>, <code>@Bean</code>, <code>@ConfigurationProperties</code></td>
<td>직접 만드는 Bean과 외부 설정을 정의한다</td>
</tr>
<tr>
<td>검증·예외</td>
<td><code>@Valid</code>, <code>@RestControllerAdvice</code>, <code>@ExceptionHandler</code></td>
<td>입력을 검증하고 예외를 HTTP 응답으로 바꾼다</td>
</tr>
<tr>
<td>부가 기능</td>
<td><code>@Transactional</code>, <code>@Async</code>, <code>@Retryable</code>, <code>@Aspect</code></td>
<td>본래 로직 밖에서 트랜잭션·비동기·재시도·로깅을 처리한다</td>
</tr>
</tbody></table>
<h3 id="springbootapplication의-정체"><code>@SpringBootApplication</code>의 정체</h3>
<p><code>@SpringBootApplication</code>은 크게 세 기능을 묶은 어노테이션이다.</p>
<ul>
<li><code>@Configuration</code>: 이 클래스가 설정 클래스임을 알린다.</li>
<li><code>@ComponentScan</code>: 현재 패키지와 그 하위에서 Component 계열을 찾는다.</li>
<li><code>@EnableAutoConfiguration</code>: 클래스패스와 설정을 보고 필요한 Bean을 자동 구성한다.</li>
</ul>
<p>따라서 메인 클래스는 보통 최상위 패키지에 둔다. 메인 클래스보다 바깥 패키지에 있는 <code>@Service</code>는 스캔되지 않을 수 있다. Spring AI의 자동 구성도 이 구조 위에서 동작한다.</p>
<h3 id="스테레오타입-어노테이션">스테레오타입 어노테이션</h3>
<p><code>@Service</code>, <code>@Repository</code>, <code>@Controller</code>는 모두 넓게 보면 <code>@Component</code>의 특수형이다. Bean 등록이라는 기본 기능은 비슷하지만, 이름을 통해 클래스의 의도를 바로 전달한다.</p>
<ul>
<li><code>@Controller</code>: View를 반환하는 웹 컨트롤러</li>
<li><code>@RestController</code>: <code>@Controller + @ResponseBody</code>, 주로 JSON 반환</li>
<li><code>@Service</code>: 업무 흐름과 규칙</li>
<li><code>@Repository</code>: 데이터 접근이며, DB 예외를 스프링 표준 예외로 변환하는 기능도 있다</li>
<li><code>@Component</code>: 위 역할에 딱 맞지 않는 범용 부품</li>
</ul>
<h2 id="1-4-bean과-의존성-주입">1-4. Bean과 의존성 주입</h2>
<p>Bean은 스프링이 생성하고 관리하는 객체다. 회사의 공용 노트북처럼, 필요한 객체를 각 클래스가 직접 <code>new</code>로 만들지 않고 스프링에게 받아 쓴다고 생각하면 쉽다.</p>
<p>기본 Bean은 애플리케이션 전체에 하나인 <strong>싱글턴</strong>이다. 그러므로 요청마다 달라지는 사용자 ID를 Bean의 필드에 저장하면 동시 요청끼리 값이 섞일 수 있다.</p>
<pre><code class="language-java">@Service
class SafeOrderService {
    private final OrderRepository repository; // 공유해도 되는 불변 협력자

    SafeOrderService(OrderRepository repository) {
        this.repository = repository;
    }

    void handle(String userId) {               // 요청별 값은 파라미터
        repository.findByOwnerId(userId);
    }
}</code></pre>
<p>대표 스코프는 다음과 같다.</p>
<ul>
<li><code>singleton</code>: 애플리케이션당 한 개, 기본값</li>
<li><code>prototype</code>: 주입을 요청할 때마다 새 객체</li>
<li><code>request</code>: HTTP 요청마다 새 객체</li>
</ul>
<h3 id="왜-생성자-주입인가">왜 생성자 주입인가</h3>
<p>생성자 주입을 쓰면 필요한 의존성이 클래스의 생성자에 드러나며, <code>final</code>로 불변성을 지킬 수 있다. 테스트에서도 스프링 전체를 띄우지 않고 가짜 객체를 넘겨 <code>new</code>로 만들 수 있다. 생성자가 하나면 <code>@Autowired</code>는 생략할 수 있다.</p>
<p>같은 타입의 Bean이 여러 개라면 Bean 이름이나 <code>@Qualifier</code>로 선택한다. 생성자 인자가 지나치게 많다면 그 클래스가 너무 많은 책임을 가지고 있다는 신호다.</p>
<h2 id="1-5-controller-service-repository-mapper">1-5. Controller, Service, Repository, Mapper</h2>
<h3 id="controller-받고-검증하고-돌려준다">Controller: 받고, 검증하고, 돌려준다</h3>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/api/orders&quot;) //공통 경로는 클래스에
class OrderController {
    private final OrderService service; //서비스만 아는

    OrderController(OrderService service) {
        this.service = service;
    }

    @GetMapping(&quot;/{orderId}&quot;)
    OrderResponse find(@PathVariable String orderId, Principal principal) {
        return service.find(orderId, principal.getName());
    }
}</code></pre>
<p>Controller에 업무 판단을 위한 <code>if</code>가 계속 늘어난다면 Service로 옮겨야 할 신호다(업무 규칙이 새어 들어옴). Controller는 HTTP 언어를 Java의 값으로 번역하는 경계에 집중한다.</p>
<h3 id="service-업무-흐름과-트랜잭션">Service: 업무 흐름과 트랜잭션</h3>
<p>Service는 여러 Repository와 외부 API를 조합하여 하나의 업무를 끝낸다. 조회가 대부분인 클래스에는 <code>@Transactional(readOnly = true)</code>를 기본으로 두고, 실제 쓰기 메서드만 <code>@Transactional</code>로 재정의할 수 있다.</p>
<h3 id="repository-데이터에-닿는-곳">Repository: 데이터에 닿는 곳</h3>
<p>Repository는 DB나 외부 API 접근을 캡슐화한다. 사용자 권한 조건은 조회 후 Java 코드로 거르는 것보다 쿼리에 처음부터 포함하는 편이 안전하다.</p>
<pre><code class="language-java">Optional&lt;Order&gt; findByIdAndOwnerId(String id, String ownerId);</code></pre>
<p>이렇게 하면 다른 사용자의 주문은 애초에 조회되지 않는다. <code>findById()</code>로 가져온 후 소유자를 비교하는 방식은 비교 코드를 한 번만 빠뜨려도 정보가 노출될 수 있다.</p>
<h3 id="mapper-sql을-직접-쓰는-데이터-계층">Mapper: SQL을 직접 쓰는 데이터 계층</h3>
<p>MyBatis Mapper는 인터페이스와 SQL을 연결하고 구현체는 MyBatis가 만든다. Repository와 같은 데이터 접근 층에 있지만 SQL을 직접 제어한다는 점이 다르다.</p>
<table>
<thead>
<tr>
<th>기준</th>
<th>JPA Repository</th>
<th>MyBatis Mapper</th>
</tr>
</thead>
<tbody><tr>
<td>SQL 작성</td>
<td>메서드 이름·JPQL·프레임워크가 생성</td>
<td>개발자가 직접 작성</td>
</tr>
<tr>
<td>강점</td>
<td>CRUD, 도메인 중심 변경</td>
<td>동적 조건, 복잡한 조인·집계, 조회 최적화</td>
</tr>
<tr>
<td>주의점</td>
<td>N+1, 예상치 못한 쿼리 확인</td>
<td>SQL과 객체 매핑을 직접 관리</td>
</tr>
</tbody></table>
<p>둘 중 하나만 고집할 필요는 없다. 일반 CRUD는 JPA, 복잡한 조회는 MyBatis처럼 한 프로젝트에서 함께 쓰는 방식도 흔하다.</p>
<h2 id="1-6-dto와-entity">1-6. DTO와 Entity</h2>
<p>Entity를 API 응답으로 그대로 내보내면 DB 구조가 외부 API 계약이 된다. 컬럼 변경이 클라이언트 장애로 이어지고, 원가나 소유자 같은 내부 필드가 노출될 수도 있다.</p>
<pre><code class="language-java">public record CreateOrderRequest(
    @NotBlank String item,
    @Positive int quantity
) {}

public record OrderResponse(
    String id,
    String item,
    String status
) {
    static OrderResponse from(Order order) {
        return new OrderResponse(order.getId(), order.getItem(), order.getStatus().name());
    }
}</code></pre>
<p>DTO 변환 방식은 상황에 따라 선택한다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>적합한 경우</th>
<th>대가</th>
</tr>
</thead>
<tbody><tr>
<td>DTO 정적 팩터리</td>
<td>변환이 단순하고 다른 Bean이 필요 없을 때</td>
<td>DTO가 Entity를 알게 된다</td>
</tr>
<tr>
<td>Mapper Component</td>
<td>변환 규칙이 복잡하거나 다른 Bean이 필요할 때</td>
<td>클래스가 하나 더 생긴다</td>
</tr>
<tr>
<td>MapStruct</td>
<td>필드가 많고 같은 변환이 반복될 때</td>
<td>빌드 설정과 생성 코드를 이해해야 한다</td>
</tr>
<tr>
<td>SQL 조회 DTO</td>
<td>조회 전용 화면을 빠르게 만들 때</td>
<td>재사용성이 낮아질 수 있다</td>
</tr>
</tbody></table>
<ul>
<li>DTO를 쉽게 말하면, 밖으로 나갈 때 보여 줄 것만 골라 담는 상자이다
(DB표를 그대로 보여주지 않는다 -&gt; 사고를 막는 장치)</li>
</ul>
<p>어떤 방법이든 변환 코드를 한곳에 모으는 것이 중요하다.</p>
<h2 id="1-7-설정-bean과-aop">1-7. 설정 Bean과 AOP</h2>
<p>라이브러리 클래스에는 직접 <code>@Component</code>를 붙일 수 없다. 이런 객체는 <code>@Configuration</code> 클래스의 <code>@Bean</code> 메서드에서 만든다.</p>
<pre><code class="language-java">@Configuration
class AiConfig {
    @Bean
    ChatClient supportClient(ChatClient.Builder builder) {
        return builder
            .defaultSystem(&quot;친절한 고객 상담원으로 답한다.&quot;)
            .build();
    }
}</code></pre>
<p><code>@Bean</code> 메서드의 파라미터도 스프링이 주입한다. <code>ChatClient</code>, <code>VectorStore</code> 같은 Spring AI 객체를 용도별로 조립할 때 이 패턴을 사용한다. <code>@ConditionalOnMissingBean</code>을 붙이면 사용자가 같은 타입의 Bean을 정의하지 않았을 때만 기본 Bean을 등록할 수 있다. <code>@ConfigurationProperties</code>는 <code>helpdesk.rag.top-k</code> 같은 외부 설정을 타입이 있는 객체에 한 번에 묶어 준다.</p>
<p>AOP는 로깅, 감사, 실행 시간 측정처럼 여러 클래스에 반복될 코드를 한곳에 모은다. <code>@Transactional</code>도 프록시 기반 AOP의 한 예다. 같은 클래스 안에서 <code>this.otherMethod()</code>로 호출하면 프록시를 거치지 않아 AOP가 적용되지 않는 자기 호출 문제를 주의한다. Spring AI의 Advisor도 “공통 처리를 호출 앞뒤에 끼운다”는 점에서 비슷한 발상이다.</p>
<h2 id="1-8-검증-예외-api-문서">1-8. 검증, 예외, API 문서</h2>
<p>입력 검증 규칙은 DTO에 선언하고 Controller에서 <code>@Valid</code>로 실행한다. 업무 예외는 Service에서 던지고, <code>@RestControllerAdvice</code>가 한곳에서 HTTP 응답으로 바꾼다.</p>
<p>예상하지 못한 오류의 스택 트레이스를 사용자에게 그대로 보여 주면 내부 구조가 노출된다. 사용자에게는 안전한 메시지와 <code>traceId</code>만 주고, 상세 오류는 서버 로그에 남긴다.</p>
<p>Swagger/OpenAPI는 Controller의 코드와 검증 규칙으로 실행 가능한 API 문서를 만든다.</p>
<ul>
<li>Swagger는 API 코드를 바꾸는 도구가 아니라, API의 사용법을 문서화하고 실제 요청으로 검증하는 도구다.</li>
</ul>
<pre><code class="language-gradle">implementation &quot;org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.6&quot;</code></pre>
<ul>
<li>Swagger UI: <code>http://localhost:8080/swagger-ui.html</code></li>
<li>OpenAPI JSON: <code>/v3/api-docs</code></li>
</ul>
<p>Swagger의 <code>Try it out</code>으로 정상 요청뿐 아니라 검증 실패, 타인 자원 접근, 동적 검색 조건, 권한 오류도 확인한다. 운영에서는 문서 노출 정책을 별도로 정해야 한다.</p>
<h2 id="1-9-profile-테스트-로깅">1-9. Profile, 테스트, 로깅</h2>
<h3 id="profile">Profile</h3>
<p>개발·운영 차이는 코드보다 설정으로 표현한다. <code>application.yml</code>의 공통값 위에 <code>application-local.yml</code>, <code>application-prod.yml</code> 같은 환경별 설정이 덮어쓴다. <code>@Profile</code>로 환경별 Bean 자체를 교체할 수도 있다.</p>
<h3 id="테스트-범위">테스트 범위</h3>
<table>
<thead>
<tr>
<th>테스트</th>
<th>띄우는 범위</th>
<th>알맞은 대상</th>
</tr>
</thead>
<tbody><tr>
<td>단위 테스트</td>
<td>스프링을 띄우지 않음</td>
<td>순수 로직, Service, Tool</td>
</tr>
<tr>
<td><code>@DataJpaTest</code></td>
<td>JPA와 DB 관련 영역</td>
<td>Repository와 데이터 쿼리</td>
</tr>
<tr>
<td><code>@WebMvcTest</code></td>
<td>웹 계층</td>
<td>Controller, 검증, HTTP 매핑</td>
</tr>
<tr>
<td><code>@SpringBootTest</code></td>
<td>전체 컨텍스트</td>
<td>Bean 조립과 통합 흐름</td>
</tr>
</tbody></table>
<p>테스트는 모델의 문장 전체를 그대로 비교하지 않는다. LLM 응답은 확률적이고 모델 버전도 바뀔 수 있다. 비어 있지 않은지, 객체 형식인지, 특정 근거를 포함하는지처럼 계약을 검증한다.</p>
<h3 id="로그">로그</h3>
<ul>
<li><code>ERROR</code>: 사람이 조치해야 하는 최종 실패</li>
<li><code>WARN</code>: 재시도·폴백 등 이상이 있었지만 처리는 된 상태</li>
<li><code>INFO</code>: 주요 업무 흐름과 도구 호출</li>
<li><code>DEBUG</code>: 검색 결과나 프롬프트 같은 개발용 상세 정보</li>
</ul>
<p>프롬프트 원문에는 전화번호·주문번호 등 개인정보가 포함될 수 있으므로 운영 <code>INFO</code> 로그에 그대로 남기지 않는다. 한 요청의 로그를 연결할 <code>traceId</code>를 함께 기록한다.</p>
<h2 id="1-10-ai는-어느-계층에-두는가">1-10. AI는 어느 계층에 두는가</h2>
<p>가장 흔한 실수는 Controller가 <code>ChatClient</code>를 직접 호출하는 것이다. 그렇게 하면 웹 요청, 프롬프트, 모델 옵션, 도구, 예외 처리가 한 파일에 섞인다.</p>
<p>권장 책임 분리는 다음과 같다.</p>
<table>
<thead>
<tr>
<th>패키지</th>
<th>책임</th>
</tr>
</thead>
<tbody><tr>
<td><code>web</code></td>
<td>REST·SSE, 입력과 출력</td>
</tr>
<tr>
<td><code>service</code></td>
<td>업무 흐름과 프롬프트 조립</td>
</tr>
<tr>
<td><code>config</code></td>
<td><code>ChatClient</code>, Advisor, 모델 옵션 Bean 구성</td>
</tr>
<tr>
<td><code>rag</code></td>
<td>문서 인제스트, 검색, 근거 조립</td>
</tr>
<tr>
<td><code>tools</code></td>
<td>모델이 요청할 수 있는 실제 행동</td>
</tr>
<tr>
<td><code>advisor</code></td>
<td>로깅, 안전, 메모리 같은 공통 정책</td>
</tr>
<tr>
<td><code>eval</code></td>
<td>골든 세트와 품질 기준선</td>
</tr>
</tbody></table>
<p>AI 요청은 대략 다음 순서로 흐를 수 있다.</p>
<pre><code class="language-text">ChatController
  → 감사 Advisor
  → 안전 Advisor
  → 메모리 Advisor
  → RAG 검색
  → HelpDeskService와 ChatClient
  → 필요하면 Tool → Repository
  → 토큰·지연 측정
  → Answer DTO 반환</code></pre>
<p>이때 순서는 정책이다. 악성 입력을 메모리에 먼저 저장한 뒤 차단하면 이미 늦다. 안전 검사는 메모리 저장보다 앞에 있어야 한다.</p>
<h3 id="part-1-체크">Part 1 체크</h3>
<ul>
<li>계층 분리의 목적은 변경의 파급을 가두는 것이다.</li>
<li>Controller는 요청과 응답, Service는 업무, Repository·Mapper는 데이터를 담당한다.</li>
<li>Entity 대신 DTO를 외부에 노출한다.</li>
<li>Bean은 기본 싱글턴이므로 요청별 상태를 필드에 보관하지 않는다.</li>
<li>의존성은 생성자로 주입한다.</li>
<li><code>ChatClient</code>는 Controller가 아니라 Service에 주입한다.</li>
</ul>
<table>
<thead>
<tr>
<th>항목</th>
<th>한 줄 정리</th>
<th>실무 포인트</th>
</tr>
</thead>
<tbody><tr>
<td>계층 분리</td>
<td>변경의 파급을 가두기 위해 나눈다</td>
<td>위 → 아래로만 호출하며 역방향 호출은 금지한다</td>
</tr>
<tr>
<td><code>@RestController</code></td>
<td>요청을 받고, 검증하고, 응답을 돌려준다</td>
<td><code>if</code>가 쌓이면 업무 규칙을 Service로 옮길 신호다</td>
</tr>
<tr>
<td><code>@Service</code></td>
<td>업무 흐름과 트랜잭션의 경계다</td>
<td>클래스는 <code>readOnly</code>, 쓰기 메서드만 재정의한다</td>
</tr>
<tr>
<td><code>@Repository</code></td>
<td>데이터에 접근하는 유일한 곳이다</td>
<td>권한 조건은 조회 후가 아니라 쿼리 안에 포함한다</td>
</tr>
<tr>
<td>Mapper(MyBatis)</td>
<td>SQL이 주인공인 조회를 담당한다</td>
<td>Repository와 같은 계층이지만 구현 방식이 다르다</td>
</tr>
<tr>
<td>DTO</td>
<td>계층 사이의 방화벽이다</td>
<td>Entity를 API 응답으로 직접 내보내지 않는다</td>
</tr>
<tr>
<td>생성자 주입</td>
<td>필요한 의존성이 생성자 시그니처에 드러난다</td>
<td>의존성이 5개를 넘으면 책임이 과도한지 점검한다</td>
</tr>
<tr>
<td><code>@Bean</code></td>
<td>라이브러리 타입을 Spring Bean으로 등록한다</td>
<td>메서드 파라미터에도 Bean이 주입되며 AI 설정의 기본 형태다</td>
</tr>
<tr>
<td>AI 배치</td>
<td>Config·Advisor·Service에 책임을 분산한다</td>
<td>Controller는 AI 구현을 모르게 한다</td>
</tr>
<tr>
<td>변환</td>
<td>Entity ↔ DTO 변환은 한곳에서 처리한다</td>
<td>입력 경로가 여러 개여도 출력 변환 지점은 하나로 모은다</td>
</tr>
<tr>
<td>API 문서</td>
<td>코드에서 생성하고 Swagger UI로 시험한다</td>
<td>운영 프로파일에서는 외부 접근을 제한하거나 비활성화한다</td>
</tr>
</tbody></table>
<hr />
<h1 id="part-2-spring-ai-개요와-아키텍처">Part 2. Spring AI 개요와 아키텍처</h1>
<h2 id="2-1-spring-ai란">2-1. Spring AI란</h2>
<p>Spring AI는 다양한 AI 모델을 Spring Boot 방식으로 사용하게 해 주는 프레임워크다. 개발자가 공급자의 HTTP API, 인증 헤더, 요청·응답 JSON을 매번 직접 처리하는 대신 Spring의 Bean, DI, 자동 구성, 추상화를 사용한다.</p>
<pre><code class="language-text">애플리케이션 코드
  → ChatClient
  → ChatModel 같은 공통 추상화
  → OpenAI·Anthropic·Azure OpenAI 등의 공급자 구현</code></pre>
<p>핵심 설계 원칙은 다음과 같다.</p>
<ul>
<li><strong>이식성(Portability)</strong>: 공급자가 달라도 비슷한 인터페이스를 사용한다.</li>
<li><strong>추상화 우선</strong>: 구체 공급자 구현보다 <code>ChatModel</code> 같은 인터페이스에 의존한다.</li>
<li><strong>Spring Boot 통합</strong>: Bean, DI, Starter, AutoConfiguration을 그대로 사용한다.</li>
<li><strong>확장성</strong>: Advisor와 Tool 같은 확장 지점을 표준화한다.</li>
</ul>
<h2 id="2-2-3대-핵심-추상화">2-2. 3대 핵심 추상화</h2>
<h3 id="①-chatmodel">① <code>ChatModel</code></h3>
<p>프롬프트를 받아 모델의 응답을 반환하는 저수준 대화 추상화다(텍스트 응답 및 대화/생성 요약). 직접 사용할 수 있지만, 실무에서는 옵션·Advisor·객체 변환을 편하게 제공하는 <code>ChatClient</code>로 감싸 쓰는 경우가 많다.</p>
<h3 id="②-embeddingmodel">② <code>EmbeddingModel</code></h3>
<p>텍스트를 의미 벡터로(숫자) 바꾼다. 검색, RAG, 분류, 군집화의 준비물이다.</p>
<pre><code class="language-java">float[] vector = embeddingModel.embed(&quot;조심해서 나쁠 건 없지&quot;);</code></pre>
<h3 id="③-vectorstore">③ <code>VectorStore</code></h3>
<p>임베딩 벡터와 원문·메타데이터를 저장하고, 질문 벡터와 가까운 문서를 찾는다. 개발 중에는 인메모리 구현을 쓰다가 운영에서는 pgvector·Redis·Chroma 등으로 바꿀 수 있다.</p>
<p>이 세 가지를 조합하면 챗봇, 의미 검색, RAG, 에이전트의 토대를 만들 수 있다.</p>
<h2 id="2-3-공급자-독립성과-옵션의-경계">2-3. 공급자 독립성과 옵션의 경계</h2>
<p>비즈니스 코드가 <code>ChatClient</code>, <code>ChatModel</code> 같은 공통 추상화에만 의존하면 모델 공급자를 바꿀 때 주로 Starter 의존성과 <code>application.yml</code>만 변경한다.</p>
<p>다만 완전한 무료 교체는 아니다. <code>model</code>, <code>temperature</code>, <code>maxTokens</code> 같은 공통 옵션만 사용하면 이식성이 높지만, JSON Schema 출력이나 추론 강도 같은 공급자 전용 옵션을 사용하면 해당 코드가 그 공급자에 묶인다. 고유 기능을 쓰지 말라는 뜻이 아니라, <strong>종속되는 경계를 알고 한곳에 모으라</strong>는 뜻이다.</p>
<h3 id="part-2-체크">Part 2 체크</h3>
<ul>
<li>LLM은 다음 토큰을 확률적으로 생성하므로 자연스럽지만 틀릴 수 있다.</li>
<li><code>ChatModel</code>은 대화, <code>EmbeddingModel</code>은 의미 벡터, <code>VectorStore</code>는 저장·검색을 담당한다.</li>
<li>개발 코드는 추상화에 의존하고 공급자 차이는 설정과 구현 경계로 가둔다.</li>
<li>임베딩 테스트는 정확한 소수점보다 순위와 의미적 경향을 검증한다.</li>
</ul>
<hr />
<h1 id="part-3-개발환경-구성---키와-첫-실행">Part 3. 개발환경 구성 - 키와 첫 실행</h1>
<h2 id="3-0-로컬-도구-점검">3-0. 로컬 도구 점검</h2>
<ul>
<li>JDK 21을 설치하고 Gradle toolchain도 21로 고정한다.</li>
<li>VS Code에서는 Java Extension Pack을 설치하고 파일 인코딩을 UTF-8로 맞춘다.</li>
<li>IDE에서 키를 전달한다면 소스 관리에서 제외된 <code>launch.json</code> 또는 안전한 환경 변수 방식을 사용한다.</li>
<li>프로젝트는 시스템 Gradle이 아니라 <code>./gradlew</code>로 실행해 팀 전체가 같은 Gradle 버전을 사용한다.</li>
<li><code>gradle/wrapper/gradle-wrapper.jar</code>를 포함한 Wrapper 파일은 프로젝트에 커밋한다.</li>
</ul>
<h2 id="3-1-프로젝트-의존성">3-1. 프로젝트 의존성</h2>
<p>교재 기준 시작 구성은 JDK 21, Spring Boot, Web, Swagger, Test, Spring AI BOM, 모델 Starter다.</p>
<pre><code class="language-gradle">plugins {
    id &quot;java&quot;
    id &quot;org.springframework.boot&quot; version &quot;4.1.0&quot;
    id &quot;io.spring.dependency-management&quot; version &quot;1.1.7&quot;
}

dependencies {
    implementation &quot;org.springframework.boot:spring-boot-starter-web&quot;
    implementation &quot;org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.6&quot;
    testImplementation &quot;org.springframework.boot:spring-boot-starter-test&quot;

    implementation platform(&quot;org.springframework.ai:spring-ai-bom:2.0.0&quot;)
    implementation &quot;org.springframework.ai:spring-ai-starter-model-openai&quot;
}</code></pre>
<p>BOM은 관련 모듈 버전을 한곳에서 정렬하고, Starter는 공급자 연동과 자동 구성까지 묶어 제공한다. 모델 공급자 Starter는 필요한 것만 넣는다.</p>
<h2 id="3-2-api-키를-안전하게-보관하기">3-2. API 키를 안전하게 보관하기</h2>
<p>API 키는 비밀번호이면서 곧 결제 수단이다.</p>
<table>
<thead>
<tr>
<th>환경</th>
<th>권장 방식</th>
</tr>
</thead>
<tbody><tr>
<td>개인 개발</td>
<td>셸 환경 변수, 소스 관리에서 제외한 IDE 실행 설정</td>
</tr>
<tr>
<td>팀</td>
<td>키를 공유하지 않고 구성원별 발급</td>
</tr>
<tr>
<td>CI</td>
<td>저장소의 Secret 기능</td>
</tr>
<tr>
<td>운영</td>
<td>Kubernetes Secret, Vault 같은 비밀 관리 시스템</td>
</tr>
</tbody></table>
<p><code>application.yml</code>에는 실제 키 대신 환경 변수 자리만 둔다.</p>
<pre><code class="language-yaml">spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}</code></pre>
<pre><code class="language-bash">export OPENAI_API_KEY=&quot;발급받은-키&quot;</code></pre>
<p>이미 Git에 키를 커밋했다면 파일에서 지우는 것보다 <strong>키를 즉시 폐기하고 재발급하는 일이 먼저</strong>다. Git 이력이나 캐시에 남아 있을 수 있기 때문이다. 키 원문과 프롬프트 개인정보도 로그에 남기지 않는다.</p>
<p>키를 발급할 때는 사용량·결제 한도도 함께 설정한다. 키 보관은 보안 문제이고, 호출 상한은 비용 사고를 막는 문제다. 둘 중 하나만 해서는 충분하지 않다.</p>
<h2 id="3-3-벡터-db는-언제-필요한가">3-3. 벡터 DB는 언제 필요한가</h2>
<p>RAG 전까지는 인메모리 저장소로도 충분하다. 서버 재시작 후 데이터를 유지해야 하거나 운영에 가까운 검색을 시험할 때 pgvector 같은 영속 저장소를 고려한다.</p>
<pre><code class="language-yaml">services:
  pgvector:
    image: pgvector/pgvector:pg17
    environment:
      POSTGRES_DB: springai
      POSTGRES_USER: springai
      POSTGRES_PASSWORD: springai
    ports:
      - &quot;5432:5432&quot;
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:</code></pre>
<pre><code class="language-bash">docker compose up -d
docker compose ps
docker compose down</code></pre>
<p><code>docker compose down -v</code>는 볼륨 데이터까지 삭제하므로 의미를 알고 사용해야 한다.</p>
<h2 id="3-4-첫-실행">3-4. 첫 실행</h2>
<p>자동 구성된 <code>ChatClient.Builder</code>를 주입받아 가장 작은 API를 만든다.</p>
<pre><code class="language-java">@RestController
class HelloAiController {
    private final ChatClient chat;

    HelloAiController(ChatClient.Builder builder) {
        this.chat = builder.build();
    }

    @GetMapping(&quot;/hello&quot;)
    String hello(@RequestParam(defaultValue = &quot;안녕하세요&quot;) String q) {
        return chat.prompt().user(q).call().content();
    }
}</code></pre>
<pre><code class="language-bash">./gradlew bootRun
curl 'http://localhost:8080/hello?q=Spring%20AI를%20한%20문장으로'</code></pre>
<p>응답 한 줄이 돌아오면 의존성, 네트워크, 키, 자동 구성, 모델 호출 경로가 연결된 것이다. 이 단계에서 막힌 문제를 해결한 뒤 다음 장으로 넘어가야 환경 문제와 코드 문제를 구분할 수 있다.</p>
<h2 id="3-5-트러블슈팅">3-5. 트러블슈팅</h2>
<table>
<thead>
<tr>
<th>증상</th>
<th>가능성 높은 원인</th>
<th>먼저 할 일</th>
</tr>
</thead>
<tbody><tr>
<td>기동은 되지만 호출 시 502</td>
<td>환경 변수 누락</td>
<td>키 설정 후 애플리케이션 재시작</td>
</tr>
<tr>
<td>401 Unauthorized</td>
<td>키 오류·만료·결제 문제</td>
<td>공급자 콘솔에서 키와 크레딧 확인</td>
</tr>
<tr>
<td>429 Too Many Requests</td>
<td>요청 한도 초과</td>
<td>동시 호출을 줄이고 잠시 후 재시도</td>
</tr>
<tr>
<td>한글 깨짐</td>
<td>인코딩 불일치</td>
<td>에디터·JVM·터미널을 UTF-8로 통일</td>
</tr>
<tr>
<td>Bean 주입 실패</td>
<td>메인 클래스의 패키지 위치·스캔 문제</td>
<td>메인 클래스를 최상위 패키지에 둔다</td>
</tr>
<tr>
<td>임베딩만 실패</td>
<td>임베딩 모델 설정 누락</td>
<td>채팅 모델과 별도로 임베딩 설정 확인</td>
</tr>
<tr>
<td>빌드는 되는데 IDE에 빨간 줄</td>
<td>Java 언어 서버 캐시</td>
<td>언어 서버 Workspace 정리·Gradle 재동기화</td>
</tr>
<tr>
<td>응답이 지나치게 느림</td>
<td>프록시·방화벽·외부망 제한</td>
<td>특히 회사망의 외부 API 접근 확인</td>
</tr>
</tbody></table>
<p>스택 트레이스는 맨 위만 보지 말고 마지막 <code>Caused by</code>부터 실제 원인을 찾는다.</p>
<h3 id="part-3-체크">Part 3 체크</h3>
<ul>
<li>JDK는 toolchain으로 21에 고정해 환경 차이를 줄인다.</li>
<li>VS Code, UTF-8, Gradle Wrapper를 프로젝트 공통 실행 기준으로 맞춘다.</li>
<li>Spring AI BOM과 모델 Starter로 시작한다.</li>
<li>키는 환경 변수나 Secret으로 주입하고 소스·로그에 남기지 않으며 사용 한도를 설정한다.</li>
<li>채팅 모델과 임베딩 모델은 별도 설정이 필요할 수 있다.</li>
<li><code>/hello</code>의 실제 응답 한 줄을 확인해야 환경 구성이 끝난 것이다.</li>
</ul>
<hr />
<h1 id="part-4-의존성·설정과-chatclient">Part 4. 의존성·설정과 ChatClient</h1>
<h2 id="4-1-bom-starter-모듈-지도">4-1. BOM, Starter, 모듈 지도</h2>
<ul>
<li><code>spring-ai-bom</code>: Spring AI 모듈 버전을 일괄 관리한다.</li>
<li><code>spring-ai-starter-model-*</code>: 모델 공급자와 자동 구성을 추가한다.</li>
<li><code>spring-ai-starter-vector-store-*</code>: 벡터 저장소와 자동 구성을 추가한다.</li>
<li><code>spring-ai-starter-mcp-*</code>: MCP 클라이언트·서버 기능을 추가한다.</li>
<li><code>spring-ai-advisors-vector-store</code>: RAG용 Advisor를 제공한다.</li>
<li><code>spring-ai-rag</code>: 질문 변환·재순위 같은 모듈형 RAG 파이프라인을 제공한다.</li>
<li><code>spring-ai-*-document-reader</code>: PDF, Markdown 등 문서를 읽는 부품이다.</li>
</ul>
<p>Starter는 하나만 추가해도 많은 Bean을 자동 등록할 수 있으므로, 필요한 모듈만 넣는다.</p>
<h2 id="4-2-applicationyml-profile-설정-우선순위">4-2. <code>application.yml</code>, Profile, 설정 우선순위</h2>
<p>공급자, 모델, 공통 옵션은 코드가 아니라 설정 파일에 둔다.</p>
<pre><code class="language-yaml">spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          temperature: 0.7</code></pre>
<p>개발은 저렴하고 빠른 모델, 운영은 품질과 안정성을 우선한 모델을 선택할 수 있다.</p>
<pre><code class="language-yaml"># application-dev.yml
spring.ai.openai.chat.options:
  model: gpt-4o-mini
  temperature: 0.0
  max-tokens: 300</code></pre>
<p>설정이 겹칠 때 교재가 설명하는 우선순위는 다음과 같다.</p>
<pre><code class="language-text">커맨드라인 인자
  &gt; OS 환경 변수
  &gt; application-{profile}.yml
  &gt; application.yml
  &gt; 코드의 기본값</code></pre>
<p>환경 변수는 점과 하이픈을 밑줄로 바꾸고 대문자로 표현한다. 예를 들어 <code>spring.ai.openai.api-key</code>는 <code>SPRING_AI_OPENAI_API_KEY</code>가 된다.</p>
<h2 id="4-3-autoconfiguration">4-3. AutoConfiguration</h2>
<p>Spring Boot는 Starter와 <code>application.yml</code>을 보고 <code>ChatModel</code>, <code>ChatClient.Builder</code>, <code>EmbeddingModel</code> 같은 Bean을 자동으로 구성한다. 개발자는 공급자 SDK를 직접 배선하기보다 필요한 추상화 Bean을 생성자로 주입받는다.</p>
<p>권장 패키지 예시는 다음과 같다.</p>
<pre><code class="language-text">com.example.ai
├── config/    # ChatClient·Advisor·VectorStore Bean
├── web/       # REST·SSE
├── service/   # 업무 흐름과 프롬프트 조립
├── rag/       # 문서 인제스트와 검색
├── tools/     # AI가 호출할 행동
├── advisor/   # 로깅·안전·메모리
└── dto/       # 요청·응답 객체</code></pre>
<h2 id="4-4-builder-메서드-체이닝-fluent-api">4-4. Builder, 메서드 체이닝, Fluent API</h2>
<p>햄버거 주문서에 옵션을 하나씩 체크한다고 생각하면 쉽다.</p>
<ul>
<li>Builder: 만들 객체의 옵션을 모은다.</li>
<li>메서드 체이닝: 메서드가 자신을 반환해 <code>.</code>으로 계속 이어 쓴다.(다음 impute)</li>
<li>Fluent API: 코드가 문장처럼 읽히도록 설계된 API다.</li>
<li><code>build()</code>·<code>call()</code>: 준비한 내용을 실제로 확정하거나 실행하는 종료 메서드다.</li>
</ul>
<pre><code class="language-java">String answer = chat
    .prompt()
    .system(&quot;너는 상담원이다&quot;)
    .user(&quot;반품 규정을 알려줘&quot;)
    .call()
    .content();</code></pre>
<p><code>build()</code>나 <code>call()</code>을 빼면 주문서만 작성하고 제출하지 않은 것처럼 실제 객체 생성이나 모델 호출이 일어나지 않는다.</p>
<h2 id="4-5-chatclient란">4-5. <code>ChatClient</code>란</h2>
<p><code>ChatModel</code>이 저수준 모델 호출 추상화라면, <code>ChatClient</code>는 그 위에서 프롬프트 작성, 옵션, Advisor, 동기·스트리밍 호출, 문자열·객체 변환을 묶어 제공하는 Fluent API다.</p>
<p>용도별 전화 창구를 만든다고 생각하면 된다.</p>
<ul>
<li>추출용: 추측 금지, 없으면 <code>null</code>, <code>temperature=0</code></li>
<li>상담용: 친절한 말투, 약간의 자연스러움, 로깅 Advisor</li>
<li>아이디어용: 다양한 답을 위해 상대적으로 높은 온도</li>
</ul>
<pre><code class="language-java">@Configuration
class AiConfig {
    @Bean
    ChatClient extractClient(ChatClient.Builder builder) {
        return builder
            .defaultSystem(&quot;정확히 추출한다. 추측하지 말고 없으면 null.&quot;)
            .defaultOptions(ChatOptions.builder().temperature(0.0).build())
            .build();
    }

    @Bean
    ChatClient supportClient(ChatClient.Builder builder) {
        return builder
            .defaultSystem(&quot;친절한 고객 상담원으로 답한다.&quot;)
            .defaultOptions(ChatOptions.builder().temperature(0.7).build())
            .build();
    }
}</code></pre>
<p>하나의 <code>ChatClient</code>에 모든 일을 몰아넣으면 분류의 안정성과 상담의 자연스러움처럼 서로 다른 기본값이 충돌한다.</p>
<h2 id="4-6-메시지-역할과-프롬프트">4-6. 메시지 역할과 프롬프트</h2>
<ul>
<li>System: 역할, 규칙, 말투처럼 요청마다 공통인 내용</li>
<li>User: 이번 사용자의 질문</li>
<li>대화 이력: 이후에는 ChatMemory Advisor가 자동 주입할 수 있음</li>
</ul>
<p>좋은 프롬프트는 신입 직원에게 업무를 맡기는 문서와 비슷하다.</p>
<ol>
<li>역할: 누구로서 답하는가</li>
<li>지시: 무엇을 해야 하는가</li>
<li>맥락: 어떤 자료를 참고하는가</li>
<li>예시: 원하는 결과의 견본은 무엇인가</li>
<li>출력 형식: 어떤 모양으로 제출하는가</li>
</ol>
<p>동적 값은 문자열 연결보다 자리 표시자와 파라미터로 바인딩한다.</p>
<pre><code class="language-java">String reply = chat.prompt()
    .user(u -&gt; u
        .text(&quot;{topic}을 초보자에게 3문장으로 설명해 줘&quot;)
        .param(&quot;topic&quot;, topic))
    .call()
    .content();</code></pre>
<p>이 방식은 템플릿을 재사용하기 쉽고, 사용자 입력과 지시를 구분하는 데도 유리하다. 다만 파라미터 바인딩만으로 모든 프롬프트 인젝션이 완전히 차단되는 것은 아니므로 별도 검증과 권한 제어도 필요하다.</p>
<h2 id="4-7-호출-방식과-결과-받기">4-7. 호출 방식과 결과 받기</h2>
<table>
<thead>
<tr>
<th>호출</th>
<th>반환</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>.call().content()</code></td>
<td><code>String</code></td>
<td>짧은 답, 분류, 추출</td>
</tr>
<tr>
<td><code>.call().entity(Type.class)</code></td>
<td>Java 객체</td>
<td>구조화된 API 응답</td>
</tr>
<tr>
<td><code>.call().chatClientResponse()</code></td>
<td>응답과 실행 컨텍스트</td>
<td>출처·Advisor 컨텍스트가 필요할 때</td>
</tr>
<tr>
<td><code>.call().chatResponse()</code></td>
<td>모델 메타데이터 포함 응답</td>
<td>토큰, 모델, 종료 이유 확인</td>
</tr>
<tr>
<td><code>.stream().content()</code></td>
<td><code>Flux&lt;String&gt;</code></td>
<td>긴 채팅 답변을 실시간 표시</td>
</tr>
</tbody></table>
<p>동기 호출은 완성된 결과를 다루기 쉽지만 긴 답변에서는 사용자가 빈 화면을 오래 본다. 스트리밍은 첫 글자를 빨리 보여 체감 속도를 높이지만 취소, 타임아웃, 중간 오류 처리가 필요하다.</p>
<h2 id="4-8-응답-메타데이터">4-8. 응답 메타데이터</h2>
<p>본문만 보지 말고 다음 정보도 운영에 활용한다.</p>
<ul>
<li>prompt tokens: 입력에 사용한 토큰</li>
<li>completion tokens: 출력에 사용한 토큰</li>
<li>total tokens: 총 사용량과 비용 계산의 근거</li>
<li>finish reason: 정상 종료, 길이 제한, 도구 호출 등 종료 이유</li>
<li>model: 실제 답한 모델, 폴백 여부 확인</li>
</ul>
<p><code>finishReason=length</code>라면 문장이나 JSON이 중간에서 잘렸을 수 있다. 이를 정상 응답으로 처리하면 사용자에게 끊긴 문장이 나가거나 객체 파싱이 실패하므로 확인해야 한다.</p>
<h3 id="part-4-체크">Part 4 체크</h3>
<ul>
<li>BOM은 버전, Starter는 공급자 연동과 자동 구성을 맡는다.</li>
<li>공급자와 모델 설정은 <code>application.yml</code>, 비밀값은 환경 변수에 둔다.</li>
<li><code>ChatClient</code>는 용도별 Bean으로 나눈다.</li>
<li>System과 User 메시지의 책임을 분리한다.</li>
<li><code>.call()</code>에서 실제 모델 호출이 일어나고 <code>.content()</code> 등으로 결과를 꺼낸다.</li>
<li>토큰 수와 <code>finishReason</code>도 결과의 일부로 다룬다.</li>
</ul>
<hr />
<h1 id="1일차-최종-정리">1일차 최종 정리</h1>
<p>오늘 배운 내용은 다음 구조로 연결된다.</p>
<pre><code class="language-text">Spring Boot 계층
  Controller: HTTP 경계
  Service: 업무 흐름
  Repository/Mapper: 데이터 경계
  DTO: 외부 계약

Spring AI 추상화
  ChatModel: 대화
  EmbeddingModel: 의미 벡터
  VectorStore: 벡터 저장·검색
  ChatClient: 모델 호출의 편리한 표준 창구

실제 서비스
  설정과 비밀은 코드 밖으로
  AI 호출은 Service 안으로
  공통 정책은 Config·Advisor로
  권한 검증은 쿼리·도구 코드로
  장애는 추적하고 핵심 기능은 폴백으로 보호</code></pre>
<h2 id="반드시-기억할-10문장">반드시 기억할 10문장</h2>
<ol>
<li>계층을 나누는 이유는 변경의 파급을 가두기 위해서다.</li>
<li>Controller는 받고 돌려주며, 업무 판단은 Service가 한다.</li>
<li>Repository와 Mapper는 데이터에 닿는 같은 층의 서로 다른 방식이다.</li>
<li>Entity를 API에 그대로 노출하지 않고 DTO로 변환한다.</li>
<li>Bean은 기본 싱글턴이므로 요청별 값을 필드에 저장하지 않는다.</li>
<li>LLM은 기억하거나 진실을 조회하는 존재가 아니라 다음 토큰을 확률적으로 생성한다.</li>
<li><code>ChatModel</code>, <code>EmbeddingModel</code>, <code>VectorStore</code>가 Spring AI의 기본 뼈대다.</li>
<li>공급자·모델·옵션은 설정으로, API 키는 환경 변수나 Secret으로 관리한다.</li>
<li><code>ChatClient</code>는 용도별 Bean으로 만들고 Service에 주입한다.</li>
<li>AI가 실패해도 핵심 서비스가 함께 죽지 않도록 오류 처리와 폴백을 설계한다.</li>
</ol>
<h2 id="스스로-확인할-질문">스스로 확인할 질문</h2>
<h3 id="1-왜-controller가-repository나-chatclient를-직접-호출하면-안-될까">1. 왜 Controller가 Repository나 ChatClient를 직접 호출하면 안 될까?</h3>
<p>계층 간 책임이 뒤섞여 비즈니스 로직의 재사용과 테스트가 어려워지고 변경 영향이 Controller까지 퍼지기 때문이다.</p>
<h3 id="2-싱글턴-service의-필드에-currentuserid를-저장하면-어떤-문제가-생길까">2. 싱글턴 Service의 필드에 <code>currentUserId</code>를 저장하면 어떤 문제가 생길까?</h3>
<p>여러 사용자의 요청이 하나의 Service 인스턴스를 공유하므로 값이 덮어써져 다른 사용자의 정보가 노출될 수 있다.</p>
<h3 id="3-jpa-repository와-mybatis-mapper는-언제-각각-유리할까">3. JPA Repository와 MyBatis Mapper는 언제 각각 유리할까?</h3>
<p>단순 CRUD와 객체 중심 개발에는 JPA가, 복잡한 SQL과 세밀한 쿼리 제어에는 MyBatis가 유리하다.</p>
<h3 id="4-chatmodel-embeddingmodel-vectorstore의-입력과-출력은-무엇일까">4. <code>ChatModel</code>, <code>EmbeddingModel</code>, <code>VectorStore</code>의 입력과 출력은 무엇일까?</h3>
<p><code>ChatModel</code>은 프롬프트를 받아 답변을, <code>EmbeddingModel</code>은 텍스트를 받아 숫자 벡터를, <code>VectorStore</code>는 문서·벡터를 저장하고 검색 결과를 반환한다.</p>
<h3 id="5-temperature0이-모든-답을-절대적으로-동일하게-보장한다고-말할-수-있을까">5. <code>temperature=0</code>이 모든 답을 절대적으로 동일하게 보장한다고 말할 수 있을까?</h3>
<p>서버 구현, 모델 업데이트 및 연산 차이 등이 있으므로 <code>temperature=0</code>도 결과의 완전한 동일성을 보장하지 않는다.</p>
<h3 id="6-모델-공급자를-바꿀-때-어떤-파일과-의존성을-먼저-확인해야-할까">6. 모델 공급자를 바꿀 때 어떤 파일과 의존성을 먼저 확인해야 할까?</h3>
<p><code>build.gradle</code>의 모델 스타터 의존성과 <code>application.yml</code>의 공급자·모델·API 키 설정을 먼저 확인해야 한다.</p>
<h3 id="7-finishreasonlength를-무시하면-어떤-장애가-생길까">7. <code>finishReason=length</code>를 무시하면 어떤 장애가 생길까?</h3>
<p>토큰 제한으로 중간에 잘린 답변을 완성된 결과로 처리해 불완전한 문장이나 JSON 파싱 오류가 발생할 수 있다.</p>
<h3 id="8-주문-요약-ai가-장애일-때-주문-조회까지-실패시키는-것이-옳을까">8. 주문 요약 AI가 장애일 때 주문 조회까지 실패시키는 것이 옳을까?</h3>
<p>주문 조회가 핵심 기능이라면 AI 요약 장애를 분리하고 원본 주문 정보는 정상적으로 제공하는 것이 바람직하다.</p>
<p>이 질문에 자기 말로 답할 수 있다면 1일차의 목표를 달성한 것이다. 다음 단계에서는 이 뼈대 위에 프롬프트 설계, 구조화 출력, 임베딩, RAG를 얹어 “그럴듯한 답”을 “근거 있는 답”으로 발전시킨다.</p>