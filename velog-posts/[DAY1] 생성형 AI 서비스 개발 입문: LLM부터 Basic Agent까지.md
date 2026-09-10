<blockquote>
<p><strong>말을 생성하는 LLM에 프롬프트, 데이터, 도구, 기억을 연결하여 실제로 쓸 수 있는 AI 서비스를 만든다.</strong></p>
</blockquote>
<hr />
<h1 id="1-introduction">1. Introduction</h1>
<h2 id="11-ai-application-개발이란">1.1 AI Application 개발이란?</h2>
<p>전통적인 프로그램은 개발자가 작성한 규칙을 그대로 따른다.</p>
<pre><code class="language-text">입력 → 개발자가 만든 규칙 → 출력</code></pre>
<p>예를 들어 쇼핑몰에서 주문 금액이 5만 원 이상이면 무료 배송을 적용한다고 하자.</p>
<pre><code class="language-python">if order_price &gt;= 50000:
    shipping_fee = 0</code></pre>
<p>입력과 규칙이 같으면 결과도 항상 같다. 이를 <strong>결정론적 방식</strong>이라고 한다.</p>
<p>AI Application은 여기에 ML, DL, LLM 같은 학습 모델을 핵심 엔진으로 넣는다. 사용자의 문장과 상황을 해석한 뒤 확률적으로 예측하거나, 적절한 답변과 행동을 만든다.</p>
<pre><code class="language-text">사용자 요청 → 맥락 해석 → AI 모델의 예측·추론 → 답변 또는 행동</code></pre>
<p>예를 들어 “부모님과 조용히 식사할 서울 식당을 추천해 줘”라는 요청에는 가격, 지역, 분위기, 동행자라는 여러 맥락이 들어 있다. AI Application은 이 맥락을 이해해 후보를 정리할 수 있다.</p>
<h2 id="12-네-가지-개발-방식-비교">1.2 네 가지 개발 방식 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>전통 소프트웨어</th>
<th>AI 모델 개발</th>
<th>AI Application 개발</th>
<th>AI Agent 개발</th>
</tr>
</thead>
<tbody><tr>
<td>중심 원리</td>
<td>규칙 기반</td>
<td>데이터 기반</td>
<td>문맥 기반</td>
<td>목표 기반</td>
</tr>
<tr>
<td>작동 방식</td>
<td>정해 둔 알고리즘 실행</td>
<td>데이터에서 패턴 학습</td>
<td>LLM, 프롬프트, RAG, API를 연결</td>
<td>목표를 보고 계획하고 도구를 선택해 반복 실행</td>
</tr>
<tr>
<td>개발자의 역할</td>
<td>비즈니스 로직, DB, UI 구현</td>
<td>데이터 정제, 모델 학습·평가</td>
<td>프롬프트, 검색, 모델, 백엔드 연결</td>
<td>역할·도구·권한·실패 대응·협업 구조 설계</td>
</tr>
<tr>
<td>자율성</td>
<td>없음</td>
<td>낮음</td>
<td>중간</td>
<td>높음</td>
</tr>
<tr>
<td>외부 도구</td>
<td>코드에 지정된 기능만 실행</td>
<td>보통 모델 내부 추론에 집중</td>
<td>설계된 파이프라인 안에서 사용</td>
<td>상황에 따라 에이전트가 도구 선택</td>
</tr>
<tr>
<td>운영 관점</td>
<td>DevOps</td>
<td>MLOps</td>
<td>LLMOps</td>
<td>AgentOps</td>
</tr>
</tbody></table>
<p>각 운영 관점의 차이도 알아 두자.</p>
<ul>
<li><strong>DevOps</strong>: 배포, 서버, 장애, CI/CD를 관리한다.</li>
<li><strong>MLOps</strong>: 학습 데이터와 모델 버전, 성능 저하, 재학습을 관리한다.</li>
<li><strong>LLMOps</strong>: 프롬프트, 토큰 비용, 응답 품질, 환각을 관리한다.</li>
<li><strong>AgentOps</strong>: 도구 호출 성공률, 권한, 무한 반복, 중단·승인 지점을 관리한다.</li>
</ul>
<h2 id="13-langchain-rag-langgraph의-역할">1.3 LangChain, RAG, LangGraph의 역할</h2>
<p>세 기술은 경쟁 관계라기보다 서로 다른 문제를 해결한다.</p>
<table>
<thead>
<tr>
<th>기술</th>
<th>쉬운 비유</th>
<th>담당 영역</th>
</tr>
</thead>
<tbody><tr>
<td>LangChain</td>
<td>조립식 레고 연결판</td>
<td>프롬프트, 모델, 파서, 도구, 메모리를 연결해 선형 파이프라인 구성</td>
</tr>
<tr>
<td>RAG</td>
<td>답하기 전에 참고 자료를 찾아오는 사서</td>
<td>외부 문서 검색, 관련 문맥 주입, 환각 완화</td>
</tr>
<tr>
<td>LangGraph</td>
<td>분기와 반복이 가능한 업무 흐름도</td>
<td>상태 관리, 순환 실행, 중단·재개, 멀티 에이전트 구성</td>
</tr>
</tbody></table>
<p>예를 들어 사내 규정 챗봇을 만든다면 다음처럼 나눌 수 있다.</p>
<ol>
<li>RAG가 규정 문서에서 질문과 관련된 부분을 찾는다.</li>
<li>LangChain이 <code>질문 + 검색 결과 + 프롬프트 + 모델</code>을 연결한다.</li>
<li>복잡한 승인·재검색·재검토 흐름이 필요하면 LangGraph로 상태와 반복을 관리한다.</li>
</ol>
<h2 id="14-ai-제품의-6단계-진화">1.4 AI 제품의 6단계 진화</h2>
<h3 id="1단계-백그라운드-ml">1단계: 백그라운드 ML</h3>
<p>AI가 화면 전면에 드러나지 않고 기존 서비스 뒤에서 추천, 번역, 스팸 분류 등을 수행한다.</p>
<blockquote>
<p>예: 사용자는 동영상 추천 기능을 쓰지만 내부 모델의 존재를 크게 의식하지 않는다. Google 번역, Youtube 추천</p>
</blockquote>
<h3 id="2단계-채팅">2단계: 채팅</h3>
<p>사용자가 자연어로 AI와 직접 대화한다. AI가 보조 기능을 넘어 제품의 중심 인터페이스가 된다.</p>
<blockquote>
<p>예: “이 문장을 초등학생도 이해하도록 바꿔 줘.” GPT</p>
</blockquote>
<h3 id="3단계-rag">3단계: RAG</h3>
<p>LLM이 학습 당시의 지식만 사용하는 대신, 외부 문서나 최신 데이터를 검색한 뒤 그 내용을 근거로 답한다. 최신성 부족과 환각을 줄이는 데 도움이 된다.</p>
<blockquote>
<p>주의: RAG는 환각을 <strong>줄이는 기술</strong>이지 완전히 없애는 마법은 아니다. 검색 결과가 틀리거나 질문과 관련이 없으면 답변도 나빠질 수 있다. 웹 검색을 하는 GPT</p>
</blockquote>
<h3 id="4단계-foreground-agent">4단계: Foreground Agent</h3>
<p>사용자가 보는 화면에서 에이전트가 계획을 세우고 도구를 사용한다. 진행 중 사용자가 확인하거나 방향을 바꿀 수 있다.</p>
<blockquote>
<p>예: 여행 에이전트가 항공편과 숙소를 조회하고 일정안을 만든 뒤, 실제 예약 전 사용자 승인을 요청한다. 맞춤형 여행 플래너</p>
</blockquote>
<h3 id="5단계-background-agent">5단계: Background Agent</h3>
<p>사용자가 계속 지켜보지 않아도 백그라운드에서 일을 처리하고 결과를 보고한다. 사람은 작업자보다 검토자에 가까워진다.</p>
<blockquote>
<p>예: 밤사이 이슈를 분석하고 코드를 수정한 뒤 테스트 결과를 아침에 보고하는 개발 에이전트. 자율형 소프트웨어 엔지니어</p>
</blockquote>
<h3 id="6단계-agent-to-agent-생태계">6단계: Agent-to-Agent 생태계</h3>
<p>서로 다른 서비스의 에이전트가 표준화된 방식으로 정보를 주고받고 협상하거나 작업을 위임한다.</p>
<blockquote>
<p>예: 개인 일정 에이전트가 병원 예약 에이전트와 가능한 시간을 조율하고, 결제 에이전트에 승인 요청을 넘긴다. Google A2A </p>
</blockquote>
<h2 id="15-rag의-작동-원리">1.5 RAG의 작동 원리</h2>
<p>RAG는 크게 <strong>사전 준비 단계</strong>와 <strong>질문 처리 단계</strong>로 나뉜다.</p>
<h3 id="사전-준비">사전 준비</h3>
<pre><code class="language-text">문서 불러오기
→ 문서를 작은 조각으로 나누기
→ 각 조각을 숫자 벡터로 변환하기
→ 벡터 데이터베이스에 저장하기</code></pre>
<ul>
<li><strong>Document Load</strong>: PDF, Word, CSV, 웹 페이지 등을 읽는다.</li>
<li><strong>Text Split</strong>: 긴 문서를 검색 가능한 작은 Chunk로 나눈다.</li>
<li><strong>Embedding</strong>: 텍스트의 의미를 비교할 수 있도록 숫자 벡터로 바꾼다.</li>
<li><strong>Vector DB</strong>: 벡터와 원문을 저장한다.</li>
</ul>
<h3 id="질문-처리">질문 처리</h3>
<pre><code class="language-text">사용자 질문
→ 질문과 의미가 가까운 문서 조각 검색
→ 검색 결과를 Context로 프롬프트에 삽입
→ LLM이 질문과 Context를 함께 보고 답변</code></pre>
<p>쉽게 말하면 시험 문제를 기억만으로 풀게 하지 않고, 관련 교과서 페이지를 먼저 찾아 책상 위에 펼쳐 주는 방식이다.</p>
<h2 id="16-agent란">1.6 Agent란?</h2>
<p>Agent는 사용자의 목표를 달성하기 위해 환경을 관찰하고, 계획을 세우며, 필요한 도구를 선택해 행동하는 시스템이다.</p>
<p>핵심 특성은 다섯 가지다.</p>
<ol>
<li><strong>자율성</strong>: 모든 단계를 사람이 일일이 지시하지 않아도 된다.</li>
<li><strong>목표 지향성</strong>: 단순 답변보다 과업 완료를 중심으로 판단한다.</li>
<li><strong>추론·계획</strong>: 필요한 단계를 나누고 순서를 정한다.</li>
<li><strong>환경 인식</strong>: 도구 결과나 시스템 상태를 관찰한다.</li>
<li><strong>도구 활용</strong>: 검색, 계산, DB, 이메일, 캘린더 등을 사용할 수 있다.</li>
</ol>
<h3 id="llm과-agent의-차이">LLM과 Agent의 차이</h3>
<table>
<thead>
<tr>
<th>LLM</th>
<th>Agent</th>
</tr>
</thead>
<tbody><tr>
<td>주로 텍스트를 입력받아 텍스트를 생성</td>
<td>LLM을 두뇌로 쓰면서 도구와 실행 흐름까지 결합</td>
</tr>
<tr>
<td>최신 날씨를 모르면 그럴듯하게 추측할 수 있음</td>
<td>날씨 API나 검색 도구로 현재 정보를 확인 가능</td>
</tr>
<tr>
<td>“어떻게 하면 되는지” 설명하는 데 강함</td>
<td>허용된 범위에서 실제 작업까지 수행 가능</td>
</tr>
</tbody></table>
<p>LLM은 똑똑한 상담가에 가깝고, Agent는 상담가에게 전화기, 계산기, 캘린더, 업무 매뉴얼을 함께 준 것에 가깝다.</p>
<h3 id="커피를-가져오는-agent-예시">커피를 가져오는 Agent 예시</h3>
<p>사용자가 “커피를 가져다줘”라고 요청했다고 하자.</p>
<ol>
<li>요청의 목표를 파악한다.</li>
<li>사용할 수 있는 도구 목록을 확인한다.</li>
<li>커피 머신을 선택한다.</li>
<li>도구를 실행해 커피를 만든다.</li>
<li>결과를 확인하고 사용자에게 전달한다.</li>
</ol>
<p>중요한 점은 도구가 많다고 좋은 Agent가 되는 것이 아니라는 사실이다. Agent가 도구의 목적과 입력 형식을 정확히 이해하고, 필요한 도구만 선택해야 한다.</p>
<h2 id="17-react-생각과-행동의-반복">1.7 ReAct: 생각과 행동의 반복</h2>
<p>ReAct는 <strong>Reasoning + Acting</strong>의 합성어다.</p>
<pre><code class="language-text">질문
→ 판단(Thought)
→ 행동(Act)
→ 관찰(Observation)
→ 다시 판단
→ 충분하면 종료</code></pre>
<p>예를 들어 “내일 부산 출장에 우산이 필요한지 알려 줘”라는 요청은 다음과 같이 처리할 수 있다.</p>
<ol>
<li>내일 부산 날씨가 필요하다고 판단한다.</li>
<li>날씨 도구를 호출한다.</li>
<li>강수 확률이라는 관찰 결과를 얻는다.</li>
<li>강수 확률이 높으면 우산을 권한다.</li>
</ol>
<p>실무에서는 모델의 비공개 내부 사고 과정을 사용자에게 모두 노출하기보다, <strong>어떤 도구를 왜 호출했고 어떤 근거로 결론을 냈는지</strong>를 추적 가능한 실행 로그로 관리하는 편이 안전하다.</p>
<h2 id="18-자율성이-커질수록-커지는-책임">1.8 자율성이 커질수록 커지는 책임</h2>
<p>Agent가 서로 소통하거나 사람에게 현실의 업무를 요청할 수 있게 되면 새로운 문제가 생긴다.</p>
<ul>
<li>잘못된 정보가 에이전트 사이에서 빠르게 확산될 수 있다.</li>
<li>위험하거나 불법적인 현실 작업이 사람에게 전달될 수 있다.</li>
<li>결제와 신원 검증이 약하면 사기 문제가 생길 수 있다.</li>
<li>사고가 났을 때 사용자, 개발사, 도구 제공자 중 누가 책임지는지 모호할 수 있다.</li>
<li>사람을 단순 실행 수단으로 취급하는 윤리 문제가 생길 수 있다.</li>
</ul>
<p>따라서 성능뿐 아니라 감시, 통제, 보안, 승인, 책임 소재를 함께 설계해야 한다.</p>
<h2 id="19-ai-거버넌스와-개발-체크리스트">1.9 AI 거버넌스와 개발 체크리스트</h2>
<p>교재는 안전한 AI 개발을 위해 다음 기준을 소개한다.</p>
<ul>
<li><strong>OWASP LLM 보안 지침</strong>: 프롬프트 인젝션, 민감정보 유출 등 LLM 애플리케이션의 대표 보안 위험을 다룬다.</li>
<li><strong>NIST AI Risk Management Framework</strong>: AI 위험을 조직적으로 식별하고 관리하기 위한 프레임워크다.</li>
<li><strong>EU AI Act</strong>: 위험 수준에 따라 AI 시스템에 의무를 부과하는 규제 체계다.</li>
<li><strong>LangChain/LangGraph의 제어 기능</strong>: 가드레일, 미들웨어, 권한, 상태 관리 등 추상적인 원칙을 코드로 구현하는 데 도움을 준다.</li>
</ul>
<blockquote>
<p>법과 프레임워크는 바뀔 수 있으므로 실제 서비스 출시 전에는 최신 공식 문서와 조직의 법무·보안 정책을 다시 확인해야 한다.</p>
</blockquote>
<h3 id="실무-체크리스트-8가지">실무 체크리스트 8가지</h3>
<table>
<thead>
<tr>
<th>영역</th>
<th>확인할 것</th>
<th>구현 아이디어</th>
</tr>
</thead>
<tbody><tr>
<td>기술적 안전성</td>
<td>환각을 줄였는가?</td>
<td>RAG, 신뢰도 평가, 중요한 답변의 사람 승인</td>
</tr>
<tr>
<td>보안</td>
<td>프롬프트 인젝션을 막는가?</td>
<td>입력 검사, 시스템 지시와 사용자 데이터 분리</td>
</tr>
<tr>
<td>권한</td>
<td>도구가 과도한 권한을 갖지 않는가?</td>
<td>역할 기반 권한, 최소 권한 토큰, 도구별 격리</td>
</tr>
<tr>
<td>공정성</td>
<td>집단별 편향이 있는가?</td>
<td>보호 속성별 성능 분리 측정, 정기 평가</td>
</tr>
<tr>
<td>투명성</td>
<td>결정 과정을 추적할 수 있는가?</td>
<td>모델·프롬프트 버전, 도구 호출, 근거, 결과 로그</td>
</tr>
<tr>
<td>개인정보</td>
<td>필요한 정보만 처리하는가?</td>
<td>데이터 최소화, 마스킹, 암호화, 보존 기간 제한</td>
</tr>
<tr>
<td>책임성</td>
<td>사고를 조사할 기록이 있는가?</td>
<td>변경 방지 감사 로그, 모델 카드, 담당자 정의</td>
</tr>
<tr>
<td>콘텐츠 안전</td>
<td>유해한 입력·출력을 다루는가?</td>
<td>필터, 가드레일, 배포 전 Red Teaming</td>
</tr>
</tbody></table>
<hr />
<h1 id="2-llm과-친해지기">2. LLM과 친해지기</h1>
<h2 id="21-llm은-무엇을-하는가">2.1 LLM은 무엇을 하는가?</h2>
<p>LLM은 Large Language Model, 즉 대규모 언어 모델이다. 방대한 텍스트에서 단어와 문장 사이의 패턴을 학습하고, 현재까지 주어진 내용 다음에 올 토큰을 확률적으로 예측한다.</p>
<pre><code class="language-text">“더운 여름에 가장 필요한 것은”
→ 에어컨일 확률이 높음
→ 다음 토큰 생성
→ 지금까지 생성한 문장을 다시 입력으로 사용
→ 종료 토큰이 나올 때까지 반복</code></pre>
<p>여기서 <strong>토큰</strong>은 모델이 글을 처리하는 작은 단위다. 한 글자나 한 단어와 정확히 같지는 않다.</p>
<h3 id="다음-글자-맞히기가-어떻게-번역과-추론을-할까">“다음 글자 맞히기”가 어떻게 번역과 추론을 할까?</h3>
<p>LLM은 학습 과정에서 번역문, 질의응답, 설명, 코드, 대화 같은 수많은 형식을 접한다. 그리고 프롬프트 안의 역할 구조를 보고 어떤 형식의 다음 문장이 필요한지 판단한다.</p>
<pre><code class="language-text">system: 너는 한영 번역가다.
user: “I love you”를 자연스럽게 번역해 줘.
assistant:</code></pre>
<p>모델은 <code>assistant:</code> 뒤에 번역 결과가 이어지는 패턴을 학습했기 때문에 번역을 생성한다. 즉, 별도의 번역 버튼을 누르는 것이 아니라 <strong>문맥에 맞는 출력을 이어 쓰는 능력</strong>을 범용 작업처럼 활용하는 것이다.</p>
<h2 id="22-llm을-사람처럼-오해하면-안-되는-이유">2.2 LLM을 사람처럼 오해하면 안 되는 이유</h2>
<p>LLM은 언어, 상식, 전문 분야의 패턴을 많이 학습했기 때문에 사람처럼 자연스럽게 말한다. 그러나 다음 사실을 기억해야 한다.</p>
<ul>
<li>실제 경험이나 감정이 있는 것은 아니다.</li>
<li>사람과 같은 방식으로 세상을 이해한다고 단정할 수 없다.</li>
<li>가장 그럴듯한 답을 생성하므로 틀린 답도 자신 있게 말할 수 있다.</li>
<li>학습 이후의 최신 정보나 비공개 정보는 기본적으로 알 수 없다.</li>
<li>말만으로는 외부 시스템에서 행동할 수 없다.</li>
</ul>
<p>따라서 LLM의 답은 “정답”이 아니라 <strong>검증해야 할 생성 결과</strong>다.</p>
<h2 id="23-llm의-한계를-보완하는-6가지-기술">2.3 LLM의 한계를 보완하는 6가지 기술</h2>
<table>
<thead>
<tr>
<th>기술</th>
<th>보완하는 한계</th>
<th>쉬운 설명</th>
</tr>
</thead>
<tbody><tr>
<td>Prompt Engineering</td>
<td>모호한 지시</td>
<td>역할, 목표, 조건, 출력 형식을 명확히 알려 준다.</td>
</tr>
<tr>
<td>Fine-tuning</td>
<td>범용적이고 들쭉날쭉한 스타일</td>
<td>특정 분야의 예시로 모델 행동을 추가 학습한다.</td>
</tr>
<tr>
<td>RAG</td>
<td>최신 정보 부족, 근거 부족</td>
<td>외부 자료를 찾아 답변에 참고시킨다.</td>
</tr>
<tr>
<td>Function/Tool Calling</td>
<td>행동할 수 없음</td>
<td>함수와 API를 호출하게 한다.</td>
</tr>
<tr>
<td>LangChain</td>
<td>연결 코드의 복잡성</td>
<td>프롬프트·모델·도구를 표준 부품처럼 연결한다.</td>
</tr>
<tr>
<td>LangGraph</td>
<td>일방향 흐름의 한계</td>
<td>상태, 분기, 반복, 중단·재개를 관리한다.</td>
</tr>
</tbody></table>
<p>Fine-tuning과 RAG의 차이도 자주 출제된다.</p>
<ul>
<li><strong>Fine-tuning</strong>은 모델의 행동 방식이나 전문성을 조정하는 데 알맞다.</li>
<li><strong>RAG</strong>는 자주 바뀌는 사실이나 조직의 문서를 그때그때 참고시키는 데 알맞다.</li>
</ul>
<h2 id="24-대표적인-모델-생태계">2.4 대표적인 모델 생태계</h2>
<p>교재는 OpenAI, Anthropic, Google, Meta, Mistral AI, Microsoft, xAI 등의 모델군을 비교한다. 초보자는 세 가지만 먼저 구분하면 된다.</p>
<table>
<thead>
<tr>
<th>제공 방식</th>
<th>특징</th>
<th>적합한 상황</th>
</tr>
</thead>
<tbody><tr>
<td>상용 API형</td>
<td>서버 운영 부담이 적고 기능 연동이 편함</td>
<td>빠른 개발, 관리형 서비스</td>
</tr>
<tr>
<td>공개 가중치형</td>
<td>자체 환경에 배포하고 세밀하게 제어 가능</td>
<td>내부망, 데이터 통제, 특수 최적화</td>
</tr>
<tr>
<td>혼합형</td>
<td>공개 모델과 상용 서비스를 함께 제공</td>
<td>상황에 따른 선택이 필요할 때</td>
</tr>
</tbody></table>
<p>모델을 선택할 때는 단순한 성능 순위보다 다음을 함께 본다.</p>
<ul>
<li>정확도와 추론 능력</li>
<li>입력 가능한 문맥 길이</li>
<li>멀티모달 지원 여부</li>
<li>Tool Calling과 Structured Output 안정성</li>
<li>응답 속도와 비용</li>
<li>데이터 보안과 배포 위치</li>
<li>라이선스와 상업적 이용 조건</li>
</ul>
<blockquote>
<p>모델명, 가격, 문맥 길이, SDK 사용법은 빠르게 바뀐다. 교재의 구체적인 버전명은 학습 당시의 예시로 이해하고, 개발 시점의 공식 문서를 확인하자.</p>
</blockquote>
<h2 id="25-hugging-face">2.5 Hugging Face</h2>
<p>Hugging Face는 모델, 데이터셋, 데모 애플리케이션을 공유하는 AI 생태계다. 흔히 “머신러닝 모델의 GitHub”에 비유한다.</p>
<ul>
<li><strong>Model Hub</strong>: 학습된 모델을 검색하고 내려받는다.</li>
<li><strong>Datasets</strong>: 학습·평가용 데이터셋을 찾는다.</li>
<li><strong>Spaces</strong>: 모델 데모나 간단한 앱을 공개한다.</li>
<li><strong>Transformers</strong>: 다양한 Transformer 모델을 공통 인터페이스로 사용한다.</li>
</ul>
<h3 id="pipeline으로-빠르게-체험하기"><code>pipeline()</code>으로 빠르게 체험하기</h3>
<p><code>pipeline()</code>은 복잡한 전처리와 후처리를 감싸 주는 고수준 인터페이스다.</p>
<pre><code class="language-python">from transformers import pipeline

classifier = pipeline(
    task=&quot;sentiment-analysis&quot;,
    model=&quot;공개된-감정분석-모델&quot;
)

result = classifier([&quot;오늘 기분이 좋아요.&quot;, &quot;서비스가 아쉬웠어요.&quot;])</code></pre>
<p>DAY1에서 다루는 대표 작업은 다음과 같다.</p>
<ul>
<li><strong>text-classification</strong>: 문장을 긍정·부정·중립이나 주제별로 분류한다.</li>
<li><strong>text-generation</strong>: 주어진 문장 뒤를 생성한다.</li>
<li><strong>question-answering</strong>: 제공된 Context에서 질문의 답을 찾는다.</li>
<li><strong>sentiment-analysis</strong>: 문장의 감정과 신뢰 점수를 반환한다.</li>
</ul>
<h3 id="max_length와-max_new_tokens"><code>max_length</code>와 <code>max_new_tokens</code></h3>
<table>
<thead>
<tr>
<th>옵션</th>
<th>제한 대상</th>
</tr>
</thead>
<tbody><tr>
<td><code>max_length</code></td>
<td>입력 토큰과 출력 토큰의 합</td>
</tr>
<tr>
<td><code>max_new_tokens</code></td>
<td>새로 생성할 출력 토큰만</td>
</tr>
</tbody></table>
<p>출력 길이를 직접 통제하려면 대체로 <code>max_new_tokens</code>가 이해하기 쉽다.</p>
<h3 id="알아-둘-pip-install-옵션">알아 둘 <code>pip install</code> 옵션</h3>
<table>
<thead>
<tr>
<th>옵션</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>-U</code>, <code>--upgrade</code></td>
<td>기존 패키지를 업그레이드</td>
</tr>
<tr>
<td><code>-q</code>, <code>--quiet</code></td>
<td>설치 로그를 줄임</td>
</tr>
<tr>
<td><code>-r</code>, <code>--requirement</code></td>
<td>파일에 적힌 패키지를 한꺼번에 설치</td>
</tr>
<tr>
<td><code>-e</code>, <code>--editable</code></td>
<td>로컬 프로젝트를 개발 모드로 설치</td>
</tr>
<tr>
<td><code>-i</code>, <code>--index-url</code></td>
<td>다른 패키지 저장소 사용</td>
</tr>
<tr>
<td><code>-t</code>, <code>--target</code></td>
<td>지정 폴더에 설치</td>
</tr>
<tr>
<td><code>--no-cache-dir</code></td>
<td>다운로드 캐시를 사용하지 않음</td>
</tr>
<tr>
<td><code>--force-reinstall</code></td>
<td>강제로 다시 설치</td>
</tr>
<tr>
<td><code>--no-deps</code></td>
<td>의존 패키지를 제외하고 설치</td>
</tr>
<tr>
<td><code>--pre</code></td>
<td>정식 출시 전 버전도 허용</td>
</tr>
</tbody></table>
<p><code>--no-deps</code>, <code>--force-reinstall</code>, <code>--pre</code>는 환경 충돌을 만들 수 있으므로 이유를 알고 있을 때만 사용한다.</p>
<h2 id="26-google-genai-sdk의-기본-흐름">2.6 Google GenAI SDK의 기본 흐름</h2>
<p>기본 개발 흐름은 다음과 같다.</p>
<ol>
<li>제공자 콘솔에서 API 키를 발급한다.</li>
<li>키를 소스 코드에 직접 적지 않고 환경 변수나 비밀 관리 도구에 저장한다.</li>
<li>SDK를 설치하고 클라이언트를 만든다.</li>
<li>모델과 입력을 지정해 호출한다.</li>
<li>필요하면 시스템 지시, temperature, 대화 세션, 스트리밍을 사용한다.</li>
</ol>
<pre><code class="language-python"># 예시 구조: 실제 키는 코드에 작성하지 않는다.
from google import genai

client = genai.Client()
response = client.models.generate_content(
    model=&quot;사용할-모델&quot;,
    contents=&quot;AI가 학습하는 원리를 짧게 설명해 줘.&quot;
)
print(response.text)</code></pre>
<h3 id="system-instruction">System instruction</h3>
<p>System instruction은 모델의 역할과 기본 규칙을 설정한다.</p>
<blockquote>
<p>예: “너는 초등학생에게 과학을 설명하는 친절한 선생님이다.”</p>
</blockquote>
<p>같은 질문도 역할 지시에 따라 말투와 난이도가 달라진다.</p>
<h3 id="temperature">Temperature</h3>
<p>temperature가 낮으면 일관되고 보수적인 답변이, 높으면 다양하고 창의적인 답변이 나올 가능성이 커진다.</p>
<h3 id="chat-session과-대화-기억">Chat session과 대화 기억</h3>
<p>Google GenAI의 Chat session 객체는 한 세션에서 주고받은 메시지를 누적 관리할 수 있다.</p>
<pre><code class="language-text">사용자: 강아지를 두 마리 키워.
사용자: 내가 몇 마리 키운다고 했지?
AI: 두 마리라고 했어요.</code></pre>
<p>모델 자체가 영구적으로 기억한 것이 아니라 SDK가 이전 메시지를 다시 함께 보내 준 결과다.</p>
<h3 id="streaming과-history">Streaming과 history</h3>
<ul>
<li><strong>Streaming</strong>: 완성된 답을 한꺼번에 기다리지 않고 생성되는 조각부터 화면에 표시한다.</li>
<li><strong>History 확인</strong>: 세션에 누적된 사용자·AI 메시지를 조회한다.</li>
</ul>
<h2 id="27-openai-sdk의-기본-흐름">2.7 OpenAI SDK의 기본 흐름</h2>
<p>OpenAI SDK도 큰 구조는 비슷하다.</p>
<pre><code class="language-python">from openai import OpenAI

client = OpenAI()
messages = [
    {&quot;role&quot;: &quot;system&quot;, &quot;content&quot;: &quot;너는 친절한 영어 교사다.&quot;},
    {&quot;role&quot;: &quot;user&quot;, &quot;content&quot;: &quot;오늘의 회화 표현을 알려 줘.&quot;},
]

response = client.chat.completions.create(
    model=&quot;사용할-모델&quot;,
    messages=messages,
)</code></pre>
<blockquote>
<p>위 코드는 교재의 호출 구조를 설명하기 위한 예시다. 실제 개발에서는 현재 SDK의 권장 API와 모델명을 공식 문서에서 확인해야 한다.</p>
</blockquote>
<h3 id="역할별-메시지">역할별 메시지</h3>
<ul>
<li><code>system</code>: AI의 역할과 상위 규칙</li>
<li><code>user</code>: 사용자의 질문이나 요청</li>
<li><code>assistant</code>: 이전 AI 답변</li>
</ul>
<h3 id="기억이-없는-호출과-있는-호출">기억이 없는 호출과 있는 호출</h3>
<p>단발 호출에서는 새 요청마다 이전 대화가 자동으로 전달되지 않을 수 있다.</p>
<pre><code class="language-text">첫 호출: “내 별명은 파랑이야.”
두 번째 호출: “내 별명이 뭐였지?”</code></pre>
<p>두 번째 호출에 첫 대화를 포함하지 않으면 모델은 알 수 없다. 기억을 이어 가려면 이전 <code>user</code>와 <code>assistant</code> 메시지를 목록에 누적해 다시 보내야 한다.</p>
<h3 id="google-chat-session과-openai식-메시지-관리-비교">Google Chat session과 OpenAI식 메시지 관리 비교</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>세션 객체 방식</th>
<th>메시지 목록 방식</th>
</tr>
</thead>
<tbody><tr>
<td>대화 기록</td>
<td>SDK 객체가 내부 누적</td>
<td>개발자가 목록에 직접 누적</td>
</tr>
<tr>
<td>장점</td>
<td>간단하게 대화 지속</td>
<td>저장·삭제·요약·감사 정책을 세밀하게 제어</td>
</tr>
<tr>
<td>주의점</td>
<td>세션 수명과 저장 범위 확인</td>
<td>메시지 누락과 토큰 증가 관리</td>
</tr>
</tbody></table>
<h3 id="tts">TTS</h3>
<p>Text-to-Speech는 텍스트를 음성으로 변환한다. 모델, 음성, 입력 텍스트를 지정한 뒤 오디오 파일로 저장하거나 스트리밍할 수 있다. 공개 서비스에서는 음성 생성 사실을 알리고, 타인의 목소리를 사칭하지 않도록 주의한다.</p>
<h3 id="멀티턴-대화">멀티턴 대화</h3>
<p>멀티턴 대화는 반복문에서 다음 순서로 구성한다.</p>
<ol>
<li>사용자 입력을 메시지 목록에 추가한다.</li>
<li>전체 메시지를 모델에 보낸다.</li>
<li>AI 답변을 다시 메시지 목록에 추가한다.</li>
<li>종료 명령이 들어올 때까지 반복한다.</li>
</ol>
<p>대화가 길어질수록 토큰 비용과 문맥 길이가 증가하므로 오래된 메시지를 요약하거나 필요한 기억만 남기는 전략이 필요하다.</p>
<h3 id="문서-요약-파이프라인">문서 요약 파이프라인</h3>
<p>문서 요약 예제의 핵심 흐름은 다음과 같다.</p>
<pre><code class="language-text">텍스트 파일 읽기
→ 요약 목적과 출력 목차를 System Prompt로 정의
→ 원문을 모델에 전달
→ Markdown 형식 결과 생성
→ 결과 파일 저장</code></pre>
<p>“짧게 요약해 줘”보다 제목, 문제 인식, 핵심 주장, 결론처럼 원하는 항목을 명시하면 결과가 더 일정해진다. 공개 글에는 원문 속 개인 이름, 연락처, 고객 정보가 섞여 있지 않은지 저장 전에 확인해야 한다.</p>
<hr />
<h1 id="3-langchain">3. LangChain</h1>
<h2 id="31-langchain이란">3.1 LangChain이란?</h2>
<p>LangChain은 LLM 기반 애플리케이션을 만드는 데 필요한 모델, 프롬프트, 도구, 검색, 메모리 등을 표준화된 방식으로 연결하는 오픈소스 프레임워크다.</p>
<h3 id="langchain을-사용하는-이유">LangChain을 사용하는 이유</h3>
<ol>
<li><strong>모델 독립성</strong>: 제공자가 달라도 비슷한 인터페이스로 다룰 수 있다.</li>
<li><strong>코드 재사용성</strong>: 모델을 바꿔도 전체 비즈니스 로직을 다시 만들 필요가 줄어든다.</li>
<li><strong>비용·성능 라우팅</strong>: 쉬운 작업은 저렴한 모델, 어려운 작업은 강한 모델로 보낼 수 있다.</li>
<li><strong>LCEL 가독성</strong>: 프롬프트, 모델, 파서를 파이프처럼 연결한다.</li>
<li><strong>풍부한 모듈</strong>: 문서 로더, 분할기, 검색기, 벡터 DB 연동 등을 재사용한다.</li>
<li><strong>Context 관리</strong>: 대화 기억과 RAG를 일관된 구조로 연결한다.</li>
</ol>
<p>LangChain은 LLM 그 자체가 아니다. 여러 부품을 연결하는 <strong>애플리케이션 프레임워크</strong>다.</p>
<h2 id="32-langchain의-여섯-가지-기본-컴포넌트">3.2 LangChain의 여섯 가지 기본 컴포넌트</h2>
<table>
<thead>
<tr>
<th>컴포넌트</th>
<th>역할</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>Chat Model</td>
<td>추론과 생성</td>
<td>두뇌</td>
</tr>
<tr>
<td>Message</td>
<td>역할이 붙은 대화 단위</td>
<td>대화 쪽지</td>
</tr>
<tr>
<td>Prompt Template</td>
<td>동적으로 지시문 생성</td>
<td>빈칸이 있는 업무 양식</td>
</tr>
<tr>
<td>Structured Output</td>
<td>응답을 정해진 데이터 구조로 변환</td>
<td>표준 신청서</td>
</tr>
<tr>
<td>LCEL</td>
<td>컴포넌트 연결</td>
<td>조립 라인</td>
</tr>
<tr>
<td>Tool Calling</td>
<td>외부 함수·API 사용</td>
<td>손과 발</td>
</tr>
</tbody></table>
<h2 id="33-chat-model">3.3 Chat Model</h2>
<p>Chat Model은 메시지 목록을 입력으로 받고 AI 메시지를 출력하는 표준 인터페이스다. 여러 상용·공개 모델을 비슷한 방식으로 사용할 수 있으며 동기, 비동기, 스트리밍, 배치 호출을 지원한다.</p>
<h3 id="모델-초기화">모델 초기화</h3>
<p>공급자 전용 클래스를 직접 사용할 수도 있고, 모델 이름을 보고 적절한 공급자 클래스를 선택해 주는 <code>init_chat_model()</code>을 사용할 수도 있다.</p>
<pre><code class="language-python">from langchain.chat_models import init_chat_model

model = init_chat_model(
    &quot;사용할-모델&quot;,
    temperature=0.1,
    timeout=30,
)</code></pre>
<p>단, 실제 공급자 통합 패키지는 별도로 설치해야 한다.</p>
<h3 id="핵심-파라미터">핵심 파라미터</h3>
<h4 id="temperature-1"><code>temperature</code></h4>
<ul>
<li><code>0.0 ~ 0.1</code>: 문서 기반 답변, 데이터 추출, 코드처럼 일관성이 중요한 작업</li>
<li><code>0.7 ~ 1.0</code>: 카피라이팅, 아이디어 발상처럼 다양성이 중요한 작업</li>
</ul>
<p>temperature가 0이어도 항상 완전히 같은 답이 보장되는 것은 아니다.</p>
<h4 id="timeout"><code>timeout</code></h4>
<p>모델 응답을 무한정 기다리지 않도록 제한한다. 운영 환경에서는 timeout, 재시도, 실패 시 대체 모델을 함께 설계한다.</p>
<h4 id="max_tokens"><code>max_tokens</code></h4>
<p>응답의 최대 길이를 제한해 비용 폭증이나 지나치게 긴 출력을 막는다. 요약·번역처럼 긴 결과가 필요한 작업에서는 너무 작게 설정하지 않는다.</p>
<h3 id="호출-방식">호출 방식</h3>
<table>
<thead>
<tr>
<th>메서드</th>
<th>의미</th>
<th>사용 예</th>
</tr>
</thead>
<tbody><tr>
<td><code>invoke()</code></td>
<td>하나의 입력을 처리하고 전체 결과 반환</td>
<td>일반 질의응답</td>
</tr>
<tr>
<td><code>stream()</code></td>
<td>생성되는 조각을 순서대로 반환</td>
<td>챗봇 화면</td>
</tr>
<tr>
<td><code>batch()</code></td>
<td>여러 입력을 병렬 처리</td>
<td>문서 여러 개 요약</td>
</tr>
</tbody></table>
<p><code>batch()</code>에서는 API의 요청 제한을 넘지 않도록 <code>max_concurrency</code>로 동시 실행 수를 제한한다.</p>
<h2 id="34-message">3.4 Message</h2>
<p>Message는 역할과 내용을 함께 담는 대화의 기본 단위다.</p>
<table>
<thead>
<tr>
<th>유형</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>SystemMessage</code></td>
<td>모델의 역할, 정책, 기본 지시</td>
</tr>
<tr>
<td><code>HumanMessage</code></td>
<td>사용자의 질문과 요청</td>
</tr>
<tr>
<td><code>AIMessage</code></td>
<td>모델의 답변 또는 Tool 호출 요청</td>
</tr>
<tr>
<td><code>ToolMessage</code></td>
<td>Tool 실행 결과</td>
</tr>
</tbody></table>
<p>Agent의 흐름에서는 다음처럼 이어진다.</p>
<pre><code class="language-text">HumanMessage
→ AIMessage(tool call)
→ ToolMessage(result)
→ AIMessage(final answer)</code></pre>
<h3 id="객체와-딕셔너리">객체와 딕셔너리</h3>
<p>메시지를 딕셔너리로 작성할 수도 있지만, LangChain에서는 전용 Message 객체가 더 안전하다.</p>
<ul>
<li>IDE의 자동완성과 타입 검사를 받을 수 있다.</li>
<li><code>role</code>, <code>content</code> 오타를 줄일 수 있다.</li>
<li>이미지, 음성, Tool call, 메타데이터처럼 복잡한 구조로 확장하기 쉽다.</li>
<li>공급자마다 다른 원본 메시지 형식을 표준화할 수 있다.</li>
</ul>
<h2 id="35-prompt와-prompt-template">3.5 Prompt와 Prompt Template</h2>
<p>Prompt는 모델에 전달하는 입력이고, Prompt Template은 변하는 값이 들어갈 자리를 가진 재사용 가능한 프롬프트다.</p>
<pre><code class="language-python">from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template(
    &quot;{country}의 수도를 도시 이름만으로 답해 주세요.&quot;
)
formatted = prompt.format(country=&quot;프랑스&quot;)</code></pre>
<p>프롬프트는 단순 질문을 넘어 다음 역할을 한다.</p>
<ul>
<li>모델이 일할 문맥과 역할을 설정한다.</li>
<li>검색된 여러 정보를 하나의 입력 형식으로 통합한다.</li>
<li>답변 범위, 말투, 길이, 형식을 통제한다.</li>
</ul>
<h3 id="네-가지-활용-방식">네 가지 활용 방식</h3>
<ol>
<li><code>PromptTemplate.from_template()</code>로 문자열에서 템플릿 생성</li>
<li><code>PromptTemplate()</code>에 템플릿과 입력 변수 직접 지정</li>
<li><code>load_prompt()</code>로 YAML 같은 외부 파일에서 불러오기</li>
<li><code>ChatPromptTemplate</code>로 System, Human, AI 메시지 목록 구성</li>
</ol>
<p>외부 파일로 프롬프트를 관리하면 코드와 프롬프트를 분리해 버전 관리하거나 비개발자가 수정하기 쉬워진다. 다만 프롬프트 파일도 실행 로직에 영향을 주므로 코드처럼 검토해야 한다.</p>
<h3 id="messagesplaceholder"><code>MessagesPlaceholder</code></h3>
<p><code>MessagesPlaceholder</code>는 이전 대화 메시지 여러 개가 들어갈 예약 자리다.</p>
<pre><code class="language-python">prompt = ChatPromptTemplate.from_messages([
    (&quot;system&quot;, &quot;너는 친절한 게임 안내원이다.&quot;),
    MessagesPlaceholder(variable_name=&quot;chat_history&quot;),
    (&quot;human&quot;, &quot;{input}&quot;),
])</code></pre>
<p>일반 <code>{input}</code>에는 문자열 하나가 들어가지만, <code>MessagesPlaceholder</code>에는 Message 객체 목록이 들어간다.</p>
<h2 id="36-structured-output">3.6 Structured Output</h2>
<p>LLM의 자유로운 문장은 사람이 읽기에는 좋지만 프로그램이 처리하기에는 불안정하다.</p>
<pre><code class="language-text">“이 문의는 불만 같고 꽤 화가 난 상태입니다.”</code></pre>
<p>이 문장을 자동화에 쓰려면 다음처럼 일정한 구조가 필요하다.</p>
<pre><code class="language-json">{
  &quot;intent&quot;: &quot;complaint&quot;,
  &quot;sentiment&quot;: &quot;negative&quot;,
  &quot;priority&quot;: 3
}</code></pre>
<p>Structured Output은 모델의 답변을 미리 정한 스키마에 맞추는 방법이다. DB 저장, API 호출, 알림 분기 같은 후속 작업이 쉬워진다.</p>
<h3 id="pydantic-방식">Pydantic 방식</h3>
<p>Python 애플리케이션에서는 Pydantic 모델이 편리하다.</p>
<pre><code class="language-python">from pydantic import BaseModel, Field
from typing import Literal, Optional

class Movie(BaseModel):
    title: str = Field(description=&quot;영화 제목&quot;)
    year: Optional[int] = Field(default=None, description=&quot;개봉 연도&quot;)
    genre: Literal[&quot;액션&quot;, &quot;로맨스&quot;, &quot;SF&quot;, &quot;코미디&quot;, &quot;기타&quot;]
    rating: float = Field(description=&quot;10점 만점 평점&quot;)

structured_model = model.with_structured_output(Movie)
result = structured_model.invoke(&quot;영화 한 편을 소개해 줘.&quot;)</code></pre>
<p>장점은 다음과 같다.</p>
<ul>
<li><code>result.title</code>처럼 필드에 바로 접근할 수 있다.</li>
<li>숫자와 문자열 타입이 구분된다.</li>
<li>정규표현식으로 응답을 억지로 파싱할 필요가 줄어든다.</li>
<li>IDE와 타입 검사의 도움을 받을 수 있다.</li>
</ul>
<p><code>Field(description=...)</code>은 단순 주석이 아니라 모델이 필드의 의미를 이해하는 프롬프트다. 허용 범위, 단위, 예시, 값이 없을 때의 처리 방법을 구체적으로 적어야 한다.</p>
<h3 id="json-schema-방식">JSON Schema 방식</h3>
<p>JSON Schema는 언어 중립적인 딕셔너리 형태로 구조를 정의한다. Python이 아닌 시스템과 공유하기 좋다.</p>
<p>키 이름은 호환성을 위해 영문, 숫자, 밑줄처럼 안전한 문자로 작성하고 <code>type</code>, <code>properties</code>, <code>required</code>, <code>description</code>을 명확히 정의한다.</p>
<h3 id="structured-output도-검증이-필요하다">Structured Output도 검증이 필요하다</h3>
<p>형식이 맞는다고 내용까지 참인 것은 아니다. 다음을 별도로 검사해야 한다.</p>
<ul>
<li>필수값이 빠지지 않았는가?</li>
<li>날짜·금액·평점의 범위가 올바른가?</li>
<li>원문에 없는 사실을 생성하지 않았는가?</li>
<li>허용된 분류값에 속하는가?</li>
</ul>
<h2 id="37-lcel">3.7 LCEL</h2>
<p>LCEL은 LangChain Expression Language의 약자다. 여러 컴포넌트를 선언적으로 연결해 실행 가능한 파이프라인을 만든다.</p>
<pre><code class="language-python">chain = prompt | model | output_parser</code></pre>
<p><code>|</code>의 왼쪽 출력이 오른쪽 입력으로 전달된다.</p>
<pre><code class="language-text">입력 딕셔너리
→ Prompt Template
→ Chat Model
→ Output Parser
→ 최종 문자열 또는 구조화 데이터</code></pre>
<h3 id="주요-특징">주요 특징</h3>
<ul>
<li><strong>선언적 구문</strong>: 데이터 흐름을 읽기 쉽다.</li>
<li><strong>모듈성</strong>: 작은 체인을 조합하고 재사용한다.</li>
<li><strong>유연성</strong>: 순차, 병렬, 조건 분기를 구성한다.</li>
<li><strong>확장성</strong>: 일반 함수를 Runnable로 바꿔 넣을 수 있다.</li>
<li><strong>표준 인터페이스</strong>: 호출 방식이 일관된다.</li>
</ul>
<h3 id="기본-구성-요소">기본 구성 요소</h3>
<ul>
<li><code>Runnable</code>: 실행 가능한 모든 LCEL 컴포넌트의 공통 개념</li>
<li><code>RunnableSequence</code>: 여러 단계를 순서대로 실행</li>
<li><code>RunnableParallel</code> 또는 Runnable Map: 여러 작업을 병렬 실행</li>
<li><code>RunnableLambda</code>: 일반 함수를 Runnable로 감쌈</li>
<li>Chain: 여러 Runnable을 조합한 실행 흐름</li>
</ul>
<h3 id="output-parser를-붙이는-이유">Output Parser를 붙이는 이유</h3>
<p><code>prompt | model</code>의 결과는 보통 AI Message 객체다. 여기에 <code>StrOutputParser()</code>를 붙이면 다음 단계가 사용하기 쉬운 문자열을 얻을 수 있다.</p>
<pre><code class="language-python">chain = prompt | model | StrOutputParser()</code></pre>
<p>영어 회화 생성 예제라면 상황을 입력받는 Prompt Template, Chat Model, 문자열 Parser를 연결한 뒤 “식당 주문”, “피자 주문” 같은 상황만 바꿔 재사용할 수 있다.</p>
<h3 id="lcel-표준-인터페이스">LCEL 표준 인터페이스</h3>
<table>
<thead>
<tr>
<th>동기</th>
<th>비동기</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>invoke()</code></td>
<td><code>ainvoke()</code></td>
<td>입력 하나 처리</td>
</tr>
<tr>
<td><code>stream()</code></td>
<td><code>astream()</code></td>
<td>결과를 조각 단위로 처리</td>
</tr>
<tr>
<td><code>batch()</code></td>
<td><code>abatch()</code></td>
<td>여러 입력 처리</td>
</tr>
</tbody></table>
<p>비동기는 한 작업을 더 빨리 계산하는 마법이 아니다. 네트워크 응답을 기다리는 동안 다른 일을 처리해 전체 서버 자원을 효율적으로 쓰는 방식이다.</p>
<h3 id="병렬-실행">병렬 실행</h3>
<p>서로 의존하지 않는 작업은 <code>RunnableParallel</code>로 동시에 실행할 수 있다.</p>
<pre><code class="language-text">입력: “대한민국”
├─ 수도 조사
└─ 면적 조사
결과: {&quot;capital&quot;: ..., &quot;area&quot;: ...}</code></pre>
<p>하나의 질문을 긍정 관점과 부정 관점에서 각각 분석한 뒤 두 결과를 합쳐 요약하는 구조도 만들 수 있다.</p>
<h3 id="주요-runnable">주요 Runnable</h3>
<h4 id="runnablepassthrough"><code>RunnablePassthrough</code></h4>
<p>입력을 그대로 다음 단계로 보낸다. 원본 질문을 보존하면서 다른 계산 결과를 함께 만들 때 유용하다.</p>
<pre><code class="language-text">입력 {num: 1}
├─ 원본 유지
├─ num × 3 필드 추가
└─ num + 1 결과 생성</code></pre>
<h4 id="runnableparallel"><code>RunnableParallel</code></h4>
<p>독립적인 여러 체인을 동시에 실행하고 결과를 딕셔너리로 묶는다.</p>
<h4 id="runnablelambda"><code>RunnableLambda</code></h4>
<p>전처리, 계산, 여러 답변 결합 같은 일반 Python 함수를 파이프라인 안에 넣는다.</p>
<h4 id="runnablebranch"><code>RunnableBranch</code></h4>
<p>입력 조건에 따라 다른 체인으로 라우팅한다.</p>
<pre><code class="language-text">질문 분류
├─ 수학 → 수학 전문가 체인
├─ 과학 → 과학 전문가 체인
└─ 기타 → 일반 답변 체인</code></pre>
<p>분류 결과가 틀리면 잘못된 체인으로 이동하므로 라벨을 짧고 명확하게 제한하고 예외 경로를 마련해야 한다.</p>
<h2 id="38-tool-calling">3.8 Tool Calling</h2>
<p>Tool Calling은 LLM이 외부 함수, API, 계산기, DB, 검색기 등을 호출하도록 만드는 기능이다.</p>
<p>LLM의 대표 한계와 Tool의 연결은 다음과 같다.</p>
<table>
<thead>
<tr>
<th>LLM의 한계</th>
<th>연결할 Tool</th>
</tr>
</thead>
<tbody><tr>
<td>현재 날씨를 모름</td>
<td>날씨 API</td>
</tr>
<tr>
<td>정확한 계산에 약할 수 있음</td>
<td>계산기</td>
</tr>
<tr>
<td>사내 문서를 모름</td>
<td>RAG 검색 Tool</td>
</tr>
<tr>
<td>일정 변경을 직접 못 함</td>
<td>캘린더 API</td>
</tr>
<tr>
<td>업무 데이터를 모름</td>
<td>권한이 제한된 DB 조회 Tool</td>
</tr>
</tbody></table>
<h3 id="built-in-tool과-custom-tool">Built-in Tool과 Custom Tool</h3>
<ul>
<li><strong>Built-in Tool</strong>: 프레임워크나 통합 패키지가 제공하는 검색기·도구를 가져다 쓴다.</li>
<li><strong>Toolkit</strong>: 관련 Tool 여러 개를 묶은 모음이다.</li>
<li><strong>Custom Tool</strong>: 개발자가 일반 Python 함수를 직접 도구로 만든다.</li>
</ul>
<p>검색 Tool은 결과 수, 검색 깊이, 허용·제외 도메인, 요약 답변 포함 여부, 원문 포함 여부, 이미지 포함 여부 등을 설정할 수 있다. 원문을 너무 많이 포함하면 비용과 프롬프트 인젝션 위험이 커질 수 있으므로 필요한 만큼만 가져온다.</p>
<h3 id="사용자-정의-tool-만들기">사용자 정의 Tool 만들기</h3>
<pre><code class="language-python">from langchain.tools import tool

@tool
def multiply(a: int, b: int) -&gt; int:
    &quot;&quot;&quot;두 정수 a와 b를 곱한다.&quot;&quot;&quot;
    return a * b</code></pre>
<p>좋은 Tool을 만들기 위한 네 가지 요소는 다음과 같다.</p>
<ol>
<li>한 가지 책임을 가진 함수</li>
<li>정확한 매개변수 타입</li>
<li>모델이 이해할 수 있는 명확한 docstring</li>
<li>Tool 변환 데코레이터</li>
</ol>
<p>Tool의 설명은 모델이 “언제 이 도구를 써야 하는가?”를 결정하는 근거가 된다. 모호한 설명은 잘못된 Tool 선택으로 이어진다.</p>
<h3 id="tool-calling의-장점과-주의점">Tool Calling의 장점과 주의점</h3>
<p>장점:</p>
<ul>
<li>LLM과 실제 기능을 연결한다.</li>
<li>모델이 상황에 따라 도구를 선택할 수 있다.</li>
<li>인자를 구조화해 전달할 수 있다.</li>
<li>호출 기록을 남기기 쉽다.</li>
<li>허용된 함수만 노출해 행동 범위를 제한할 수 있다.</li>
</ul>
<p>주의점:</p>
<ul>
<li>API 키는 코드나 블로그에 공개하지 않는다.</li>
<li>쓰기·삭제·결제 Tool은 읽기 Tool보다 엄격한 승인을 둔다.</li>
<li>Tool 입력을 검증하고 timeout과 오류 처리를 추가한다.</li>
<li>검색 결과와 웹 콘텐츠를 신뢰할 수 없는 입력으로 취급한다.</li>
</ul>
<hr />
<h1 id="4-basic-agent">4. Basic Agent</h1>
<h2 id="41-basic-agent의-구조">4.1 Basic Agent의 구조</h2>
<p>Basic Agent는 모델과 도구를 연결하고, 모델이 필요한 도구를 선택해 실행하도록 만든다.</p>
<pre><code class="language-text">사용자 요청
→ Model
↔ Tools
→ 최종 답변</code></pre>
<p>Advanced Agent는 여기에 실행 전후를 제어하는 미들웨어, 상태, 가드레일 등이 더해진다. 이 내용은 다음 학습일의 범위다.</p>
<h2 id="42-agent의-7단계-실행-흐름">4.2 Agent의 7단계 실행 흐름</h2>
<ol>
<li><strong>Request</strong>: 사용자가 질문이나 명령을 보낸다.</li>
<li><strong>Model 판단</strong>: 모델이 요청을 해석하고 Tool이 필요한지 판단한다.</li>
<li><strong>Action</strong>: AI Message에 Tool 이름과 인자를 담아 호출을 요청한다.</li>
<li><strong>Tools 실행</strong>: 실제 함수나 API가 실행된다.</li>
<li><strong>Observation</strong>: Tool 결과가 Tool Message로 모델에 돌아온다.</li>
<li><strong>Model 종합</strong>: 결과를 해석하고 추가 Tool이 필요한지 판단한다.</li>
<li><strong>Result</strong>: 사용자에게 읽기 쉬운 최종 AI Message를 보낸다.</li>
</ol>
<p>모델은 두뇌, Tool은 손과 발이라고 생각하면 쉽다.</p>
<h2 id="43-agent-생성">4.3 Agent 생성</h2>
<pre><code class="language-python">from langchain.agents import create_agent

agent = create_agent(
    model=model,
    tools=[get_weather],
)</code></pre>
<p>Tool 없이 Agent나 모델에 현재 날씨를 물으면 학습된 일반 지식으로 답하거나 모른다고 해야 한다. 실시간 정보가 필요한 요청에는 반드시 실제 데이터 Tool이 필요하다.</p>
<h2 id="44-tool-사용-예시">4.4 Tool 사용 예시</h2>
<h3 id="예시-1-날씨와-옷차림-추천">예시 1: 날씨와 옷차림 추천</h3>
<p>사용자 요청:</p>
<blockquote>
<p>“오늘 서울 날씨에 맞는 옷을 추천해 줘.”</p>
</blockquote>
<p>실행 흐름:</p>
<pre><code class="language-text">HumanMessage
→ AIMessage: get_weather(location=&quot;서울&quot;) 호출
→ ToolMessage: 비, 기온 등 실제 결과
→ AIMessage: 우산과 옷차림을 함께 추천</code></pre>
<p>날씨 Tool은 사실을 가져오고, 모델은 그 사실을 사용자의 목적에 맞게 설명한다.</p>
<h3 id="예시-2-여러-단계-계산">예시 2: 여러 단계 계산</h3>
<p>덧셈, 뺄셈, 곱셈, 나눗셈 Tool을 각각 등록할 수 있다.</p>
<p><code>42 + 3 × 23</code>을 계산한다면 Agent는 다음 순서를 선택할 수 있다.</p>
<ol>
<li>곱셈 Tool로 <code>3 × 23 = 69</code> 계산</li>
<li>덧셈 Tool로 <code>42 + 69 = 111</code> 계산</li>
<li>결과를 자연어로 설명</li>
</ol>
<p>모델이 Tool 없이 자체 계산할 수도 있기 때문에 반드시 Tool을 사용해야 하는 교육·감사 상황이라면 System Prompt나 정책으로 사용을 강제한다.</p>
<h3 id="예시-3-외부-서점-api-조회">예시 3: 외부 서점 API 조회</h3>
<p>베스트셀러 목록을 조회하는 Custom Tool은 다음 요소를 가진다.</p>
<ul>
<li>입력: 가져올 상위 개수</li>
<li>검증: 최대 개수 제한</li>
<li>동작: 외부 API 요청</li>
<li>예외 처리: HTTP 오류 확인</li>
<li>출력: 책 정보 목록</li>
</ul>
<p>인증 키는 환경 변수나 비밀 관리 시스템에서 읽어야 하며, 코드·노트북·블로그에 직접 적지 않는다.</p>
<h3 id="예시-4-여러-tool을-가진-챗봇">예시 4: 여러 Tool을 가진 챗봇</h3>
<p>하나의 Agent에 현재 시각, 계산, 날씨 Tool을 등록하면 질문에 따라 서로 다른 Tool을 선택한다.</p>
<pre><code class="language-text">“지금 몇 시야?” → 시간 Tool
“제주 날씨는?” → 날씨 Tool
“17 × 24는?” → 계산 Tool
“Agent가 뭐야?” → Tool 없이 모델이 설명</code></pre>
<p>교재의 간단한 계산 예제처럼 문자열을 바로 <code>eval()</code>에 넣으면 임의 코드 실행 위험이 있다. 실제 서비스에서는 허용한 연산자만 처리하는 안전한 수식 파서나 검증된 계산 도구를 사용해야 한다.</p>
<h2 id="45-short-term-memory">4.5 Short-term Memory</h2>
<p>LLM 호출은 기본적으로 매번 독립적이다. 다중 턴 대화에서 이전 맥락을 이어 가려면 대화 상태를 저장하고 다시 불러와야 한다.</p>
<h3 id="단기-메모리와-장기-메모리-비교">단기 메모리와 장기 메모리 비교</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>단기 메모리</th>
<th>장기 메모리</th>
</tr>
</thead>
<tbody><tr>
<td>범위</td>
<td>하나의 대화 Thread</td>
<td>여러 대화와 세션</td>
</tr>
<tr>
<td>예</td>
<td>방금 말한 이름, 직전 질문</td>
<td>장기 선호, 직업, 알레르기</td>
</tr>
<tr>
<td>대표 저장 방식</td>
<td>Checkpointer</td>
<td>Store 또는 영속 DB</td>
</tr>
<tr>
<td>수명</td>
<td>세션 중심</td>
<td>정책에 따라 장기간</td>
</tr>
</tbody></table>
<p>DAY1에서는 단기 메모리를 중심으로 다룬다.</p>
<h3 id="checkpointer와-thread_id">Checkpointer와 <code>thread_id</code></h3>
<ul>
<li><strong>Checkpointer</strong>: 대화 메시지와 Agent 상태를 저장하는 공간</li>
<li><strong><code>thread_id</code></strong>: 어떤 대화 기록을 가져올지 알려 주는 열쇠</li>
</ul>
<pre><code class="language-python">from langgraph.checkpoint.memory import InMemorySaver

agent = create_agent(
    model=model,
    tools=tools,
    checkpointer=InMemorySaver(),
)</code></pre>
<p>같은 <code>thread_id</code>를 쓰면 이전 대화를 이어 가고, 다른 <code>thread_id</code>를 쓰면 별개의 새 대화가 된다.</p>
<pre><code class="language-text">thread_1: “내 별명은 파랑이야.”
thread_1: “내 별명이 뭐지?” → 파랑

thread_2: “내 별명이 뭐지?” → 알 수 없음</code></pre>
<p><code>InMemorySaver</code>는 학습과 테스트에 편하지만 프로세스가 종료되면 데이터가 사라질 수 있다. 운영 환경에서는 영속 저장소, 암호화, 접근 권한, 보존 기간, 삭제 요청 처리까지 설계해야 한다.</p>
<h2 id="46-agent의-구조화된-답변">4.6 Agent의 구조화된 답변</h2>
<p>Agent도 최종 결과를 Pydantic 같은 스키마로 강제할 수 있다. 교재의 예시는 고객 이메일을 읽어 다음 정보를 만든다.</p>
<ul>
<li>의도: 불만, 문의, 확인, 기타</li>
<li>감정: 긍정, 부정, 중립</li>
<li>요약</li>
<li>다음 행동</li>
</ul>
<h3 id="전체-구현-순서">전체 구현 순서</h3>
<ol>
<li>이메일 읽기·보내기 Tool 정의</li>
<li><code>EmailAnalysis</code> 스키마 정의</li>
<li><code>response_format</code>을 지정해 Agent 생성</li>
<li>Agent 실행</li>
<li>구조화 결과를 DB 저장이나 업무 분기에 사용</li>
</ol>
<pre><code class="language-python">class EmailAnalysis(BaseModel):
    intent: Literal[&quot;complaint&quot;, &quot;inquiry&quot;, &quot;confirmation&quot;, &quot;other&quot;]
    sentiment: Literal[&quot;positive&quot;, &quot;negative&quot;, &quot;neutral&quot;]
    summary: str
    next_action: str</code></pre>
<h3 id="messages와-structured_response"><code>messages</code>와 <code>structured_response</code></h3>
<p>Agent 실행 결과에는 서로 다른 두 종류의 정보가 들어갈 수 있다.</p>
<ul>
<li><code>response[&quot;messages&quot;]</code>: 사용자 요청, Tool 호출, Tool 결과, 최종 답변으로 이어지는 실행 기록</li>
<li><code>response[&quot;structured_response&quot;]</code>: 스키마에 맞춰 정리된 최종 업무 데이터</li>
</ul>
<p>쉽게 말하면 <code>messages</code>는 <strong>일을 처리한 과정</strong>, <code>structured_response</code>는 <strong>제출용 결과표</strong>다.</p>
<h3 id="tool-emulator">Tool Emulator</h3>
<p>실제 이메일 서버나 유료 API가 준비되지 않았을 때 LLM이 Tool 결과를 흉내 내는 Emulator를 사용할 수 있다. 프로토타입과 화면 시연에는 편하지만 결과는 실제 데이터가 아니다.</p>
<blockquote>
<p>에뮬레이션 결과를 실제 고객 메일, 결제 결과, 재고 정보처럼 취급하면 안 된다. 테스트 데이터임을 명확히 표시해야 한다.</p>
</blockquote>
<h3 id="후처리-자동화">후처리 자동화</h3>
<p>구조화 결과를 딕셔너리로 바꾸면 일반 코드로 분기할 수 있다.</p>
<pre><code class="language-python">data = analysis.model_dump()

if data[&quot;intent&quot;] == &quot;complaint&quot; and data[&quot;sentiment&quot;] == &quot;negative&quot;:
    # 담당자에게 검토 요청
    # 이슈 티켓 생성
    pass
elif data[&quot;intent&quot;] == &quot;inquiry&quot;:
    # FAQ 검색 후 답변 초안 생성
    pass</code></pre>
<p>실제 이메일 전송, 환불, 티켓 생성은 외부에 영향을 주는 작업이다. 모델이 분류했다고 바로 실행하기보다 중요한 행동 앞에는 사람의 승인, 권한 검사, 중복 실행 방지 장치를 둔다.</p>
<hr />
<h1 id="5-day1-전체-연결하기">5. DAY1 전체 연결하기</h1>
<p>지금까지 배운 개념은 하나의 서비스 안에서 다음처럼 연결된다.</p>
<pre><code class="language-text">사용자 질문
→ Message로 표현
→ Prompt Template에 삽입
→ Chat Model이 판단
→ 필요하면 Tool 호출
→ Tool 결과를 다시 모델에 전달
→ Structured Output으로 정리
→ Checkpointer가 같은 대화의 상태 저장</code></pre>
<p>LangChain의 관점에서는 다음과 같이 볼 수 있다.</p>
<pre><code class="language-text">Prompt | Model | Parser
           ↕
          Tools
           ↕
        Memory</code></pre>
<h2 id="핵심-용어-한눈에-보기">핵심 용어 한눈에 보기</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>한 줄 정의</th>
</tr>
</thead>
<tbody><tr>
<td>LLM</td>
<td>문맥을 바탕으로 다음 토큰을 생성하는 대규모 언어 모델</td>
</tr>
<tr>
<td>Prompt</td>
<td>모델에 전달하는 질문, 지시, 문맥</td>
</tr>
<tr>
<td>Message</td>
<td>역할과 내용을 가진 대화 단위</td>
</tr>
<tr>
<td>Chat Model</td>
<td>메시지를 입력받고 AI 메시지를 만드는 표준 인터페이스</td>
</tr>
<tr>
<td>RAG</td>
<td>관련 외부 자료를 검색해 답변 문맥에 추가하는 방식</td>
</tr>
<tr>
<td>Structured Output</td>
<td>모델 결과를 정해진 스키마로 받는 방식</td>
</tr>
<tr>
<td>LCEL</td>
<td>LangChain 컴포넌트를 선언적으로 연결하는 표현식</td>
</tr>
<tr>
<td>Tool</td>
<td>검색, 계산, API 호출 등 외부 작업 인터페이스</td>
</tr>
<tr>
<td>Agent</td>
<td>목표를 보고 모델과 Tool을 사용해 과업을 수행하는 시스템</td>
</tr>
<tr>
<td>Checkpointer</td>
<td>대화 Thread의 상태를 저장하는 장치</td>
</tr>
<tr>
<td><code>thread_id</code></td>
<td>어떤 대화 기억을 사용할지 구분하는 식별자</td>
</tr>
</tbody></table>
<h2 id="초보자가-자주-헷갈리는-질문">초보자가 자주 헷갈리는 질문</h2>
<h3 id="q1-langchain이-llm인가">Q1. LangChain이 LLM인가?</h3>
<p>아니다. LLM은 답을 생성하는 모델이고, LangChain은 모델과 다른 부품을 연결하는 프레임워크다.</p>
<h3 id="q2-prompt-template이-있으면-memory도-생기나">Q2. Prompt Template이 있으면 Memory도 생기나?</h3>
<p>아니다. Prompt Template은 입력 양식이다. 이전 대화를 기억하려면 메시지 기록이나 Checkpointer를 별도로 연결해야 한다.</p>
<h3 id="q3-structured-output이면-내용도-항상-정확한가">Q3. Structured Output이면 내용도 항상 정확한가?</h3>
<p>아니다. 형식을 안정시키는 기술이지 사실성을 보장하는 기술은 아니다. 원문 근거와 값의 범위를 검증해야 한다.</p>
<h3 id="q4-tool을-등록하면-모델이-반드시-쓰나">Q4. Tool을 등록하면 모델이 반드시 쓰나?</h3>
<p>아니다. 모델은 필요 없다고 판단하면 Tool을 사용하지 않을 수 있다. 반드시 써야 한다면 정책이나 프롬프트로 강제하고 호출 기록을 검증한다.</p>
<h3 id="q5-같은-thread_id면-영원히-기억하나">Q5. 같은 <code>thread_id</code>면 영원히 기억하나?</h3>
<p>아니다. 저장소의 종류와 수명에 따라 달라진다. 메모리 기반 Checkpointer는 프로그램 종료 시 사라질 수 있다.</p>
<h3 id="q6-rag와-agent는-같은가">Q6. RAG와 Agent는 같은가?</h3>
<p>아니다. RAG는 관련 자료를 검색해 답변에 넣는 패턴이고, Agent는 목표에 따라 검색을 포함한 여러 Tool을 선택하고 반복 실행하는 시스템이다.</p>
<h2 id="셀프-체크">셀프 체크</h2>
<p>다음 질문에 말로 답할 수 있다면 DAY1의 큰 흐름을 이해한 것이다.</p>
<ol>
<li>전통 소프트웨어, AI Application, AI Agent의 차이는 무엇인가?</li>
<li>RAG의 사전 준비 단계와 질문 처리 단계를 설명할 수 있는가?</li>
<li>LLM이 그럴듯하지만 틀린 답을 만들 수 있는 이유는 무엇인가?</li>
<li>System, Human, AI, Tool Message의 역할은 무엇인가?</li>
<li><code>invoke</code>, <code>stream</code>, <code>batch</code>는 언제 사용하는가?</li>
<li>Pydantic Structured Output의 장점은 무엇인가?</li>
<li><code>RunnableParallel</code>, <code>RunnableLambda</code>, <code>RunnableBranch</code>는 각각 무엇을 하는가?</li>
<li>Agent의 7단계 흐름을 설명할 수 있는가?</li>
<li>Checkpointer와 <code>thread_id</code>는 어떤 관계인가?</li>
<li>Tool을 실제 서비스에 연결할 때 어떤 보안 장치가 필요한가?</li>
</ol>
<h2 id="마무리">마무리</h2>
<p>DAY1의 핵심은 “LLM에게 질문을 잘하는 법”에서 끝나지 않는다. 실제 서비스는 LLM을 중심으로 프롬프트, 메시지, 출력 구조, 외부 도구, 대화 상태를 안전하게 연결해야 한다.</p>
<p>처음에는 다음 순서로 작은 예제를 만드는 것이 좋다.</p>
<ol>
<li>Chat Model을 한 번 호출한다.</li>
<li>Prompt Template과 Output Parser를 연결한다.</li>
<li>Structured Output을 추가한다.</li>
<li>읽기 전용 Tool 하나를 연결한다.</li>
<li>Checkpointer로 두 번의 대화를 이어 본다.</li>
<li>Tool 호출 기록과 실패 상황을 확인한다.</li>
</ol>
<p>이 과정을 이해하면 DAY2의 Runtime, State, Middleware, Guardrails, Long-term Memory를 배울 준비가 된 것이다.</p>