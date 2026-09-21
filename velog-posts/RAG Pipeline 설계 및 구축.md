<h2 id="0--학습-지도">0.  학습 지도</h2>
<p>1일차의 흐름은 다음과 같다.</p>
<pre><code class="language-text">RAG가 필요한 이유 이해
        ↓
전체 RAG 파이프라인 파악
        ↓
문서 불러오기(Document Loader)
        ↓
문서 나누기(Text Splitter / Chunking)
        ↓
의미를 숫자로 바꾸기(Embedding)
        ↓
벡터와 메타데이터 저장하기(Vector DB / Index / Store)</code></pre>
<p>오늘은 지식 창고를 만드는 데 집중한다. 질문이 들어온 뒤 검색하고 답을 생성하는 Retriever, Prompt, LLM, Chain은 다음 학습 단계에 해당한다.</p>
<hr />
<h1 id="part-1-rag-개요">Part 1. RAG 개요</h1>
<h2 id="1-rag란-무엇인가">1. RAG란 무엇인가?</h2>
<p>RAG는 <strong>Retrieval-Augmented Generation</strong>, 우리말로는 보통 <strong>검색 증강 생성</strong>이라고 한다.</p>
<table>
<thead>
<tr>
<th>단어</th>
<th>쉬운 뜻</th>
<th>RAG에서 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td>Retrieval</td>
<td>찾기</td>
<td>질문과 관련된 문서 조각을 찾는다.</td>
</tr>
<tr>
<td>Augmented</td>
<td>보강하기</td>
<td>찾은 내용을 LLM 입력에 추가한다.</td>
</tr>
<tr>
<td>Generation</td>
<td>생성하기</td>
<td>질문과 근거 문서를 바탕으로 답변을 만든다.</td>
</tr>
</tbody></table>
<p>쉽게 말하면 RAG는 <strong>오픈북 시험을 보는 AI</strong>다.</p>
<ul>
<li>일반 LLM: 기억에 의존해 답한다.</li>
<li>RAG: 먼저 자료를 찾고, 찾은 근거를 읽은 뒤 답한다.</li>
</ul>
<p>예를 들어 사용자가 “이번 달 교통비 정산 기준이 뭐야?”라고 질문했다고 하자. 일반 LLM은 회사의 최신 규정을 알지 못한다. RAG는 사내 규정에서 관련 부분을 검색해 LLM에 전달하고, LLM은 그 근거 안에서 답을 구성한다.</p>
<pre><code class="language-text">사용자 질문
  → 관련 자료 검색
  → 질문 + 검색 자료를 LLM에 전달
  → 근거에 기반한 답변 생성</code></pre>
<p>RAG의 핵심은 LLM을 더 똑똑하게 재학습시키는 것이 아니다. <strong>LLM이 답하기 직전에 필요한 지식을 제공하는 것</strong>이다.</p>
<hr />
<h2 id="2-왜-rag가-필요한가">2. 왜 RAG가 필요한가?</h2>
<h3 id="21-비즈니스-관점-기업의-지식은-모델-밖에-있다">2.1 비즈니스 관점: 기업의 지식은 모델 밖에 있다</h3>
<p>기업에는 업무 매뉴얼, 계약서, 회의록, 기술 문서, FAQ, 규정처럼 중요한 자료가 많다. 하지만 일반 LLM은 다음 이유로 이 내용을 그대로 알 수 없다.</p>
<ul>
<li>내부 문서는 모델의 사전 학습 데이터에 포함되지 않는다.</li>
<li>문서는 계속 개정되므로 모델이 알고 있는 정보가 금방 낡을 수 있다.</li>
<li>보안과 권한 때문에 모든 자료를 외부 학습에 사용할 수도 없다.</li>
<li>업무 답변은 “그럴듯함”보다 정확한 근거와 최신성이 중요하다.</li>
</ul>
<p>따라서 실제 서비스에서는 세 가지 고민이 생긴다.</p>
<ol>
<li><strong>보안과 적합성</strong>: 사용자가 볼 수 있는 자료만 보여 주는가?</li>
<li><strong>비용</strong>: 매번 모델을 다시 학습시키지 않고 지식을 갱신할 수 있는가?</li>
<li><strong>부정확한 결과</strong>: 근거 없는 답이나 환각을 어떻게 줄일 것인가?</li>
</ol>
<p>RAG는 내부 자료를 모델 자체에 외우게 하는 대신, 필요할 때 연결한다. 문서가 바뀌면 모델 전체를 재학습하기보다 지식 창고의 문서와 인덱스를 갱신하면 된다.</p>
<h3 id="실생활-비유-신입-직원과-사내-규정집">실생활 비유: 신입 직원과 사내 규정집</h3>
<p>신입 직원이 모든 규정을 외우기는 어렵다. 대신 잘 정리된 규정집과 검색 시스템이 있으면 질문이 생겼을 때 해당 조항을 찾아 정확히 답할 수 있다. RAG에서 LLM은 신입 직원, 문서 저장소는 규정집, Retriever는 검색 담당자에 가깝다.</p>
<hr />
<h3 id="22-기술-관점-llm의-기억만으로는-부족하다">2.2 기술 관점: LLM의 기억만으로는 부족하다</h3>
<p>LLM의 지식은 모델 파라미터 안에 압축되어 있다. 이 방식에는 근본적인 한계가 있다.</p>
<ul>
<li>지식을 고치거나 추가하려면 재학습이 필요할 수 있다.</li>
<li>어떤 출처에서 나온 답인지 확인하기 어렵다.</li>
<li>최신 정보와 전문 분야 정보에 약할 수 있다.</li>
<li>존재하지 않는 내용을 자연스럽게 만들어 내는 환각이 생길 수 있다.</li>
</ul>
<p>RAG는 두 종류의 기억을 결합한다고 이해할 수 있다.</p>
<table>
<thead>
<tr>
<th>기억</th>
<th>의미</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>Parametric memory</td>
<td>LLM 파라미터에 학습된 지식</td>
<td>머릿속에 외운 내용</td>
</tr>
<tr>
<td>Non-parametric memory</td>
<td>외부 문서·DB에 저장된 지식</td>
<td>필요할 때 펼쳐 보는 자료</td>
</tr>
</tbody></table>
<p>외부 기억을 사용하면 모델을 다시 학습하지 않고도 문서를 추가하거나 교체할 수 있고, 답변에 사용한 근거를 추적하기도 쉬워진다.</p>
<hr />
<h2 id="3-rag는-만능-해결책인가">3. RAG는 만능 해결책인가?</h2>
<p>아니다. RAG는 유용하지만, 검색할 문서의 품질이 나쁘면 답변도 좋아지기 어렵다.</p>
<h3 id="31-rag와-다른-개선-방법은-어떻게-다를까">3.1 RAG와 다른 개선 방법은 어떻게 다를까?</h3>
<p>LLM의 결과를 개선하는 방법에는 프롬프트 작성, 미세조정, PEFT, RAG 등이 있다. 서로 완전히 대체하는 기술이라기보다 해결하려는 문제가 다르다.</p>
<table>
<thead>
<tr>
<th>방법</th>
<th>바꾸는 것</th>
<th>잘 맞는 문제</th>
<th>대표적인 한계</th>
</tr>
</thead>
<tbody><tr>
<td>Prompt Engineering</td>
<td>모델에게 주는 지시문</td>
<td>역할, 답변 형식, 작업 절차 조정</td>
<td>모델이 모르는 사실을 새로 제공하지는 못함</td>
</tr>
<tr>
<td>Full Fine-tuning</td>
<td>모델 파라미터 전반</td>
<td>말투·행동·과업 자체를 크게 변경</td>
<td>학습·유지 비용이 크고 근거 추적이 어려움</td>
</tr>
<tr>
<td>PEFT</td>
<td>일부 파라미터 또는 추가 어댑터</td>
<td>비교적 적은 비용으로 특정 행동 학습</td>
<td>최신 문서를 계속 반영하는 지식 저장소와는 역할이 다름</td>
</tr>
<tr>
<td>RAG</td>
<td>검색할 외부 지식과 검색 과정</td>
<td>최신·내부·전문 자료에 근거한 답변</td>
<td>문서 전처리와 검색 품질 관리가 필요</td>
</tr>
</tbody></table>
<p>예를 들어 “답을 항상 JSON으로 작성하라”는 요구는 프롬프트가 잘 맞고, “우리 회사의 최신 휴가 규정을 근거와 함께 알려 달라”는 요구는 RAG가 잘 맞는다. 실제 서비스에서는 RAG로 근거를 제공하고 프롬프트로 답변 방식과 제한을 지시하는 식으로 함께 사용한다.</p>
<h3 id="기대-효과">기대 효과</h3>
<ul>
<li>내부·전문 지식을 활용한 정확한 답변</li>
<li>실제 문서에 근거한 응답과 출처 추적</li>
<li>사용자의 질문 맥락을 반영한 답변</li>
<li>최신 문서 반영을 위한 비교적 쉬운 업데이트</li>
<li>의료·금융·제조·법무 등 전문 영역으로의 확장</li>
</ul>
<h3 id="현실적인-난점">현실적인 난점</h3>
<table>
<thead>
<tr>
<th>문제</th>
<th>왜 생기는가?</th>
<th>기본 대응</th>
</tr>
</thead>
<tbody><tr>
<td>필요한 정보 자체가 없음</td>
<td>지식 저장소가 불완전함</td>
<td>“정보 없음”을 말하게 하고 자료를 보충한다.</td>
</tr>
<tr>
<td>정답 문서를 찾지 못함</td>
<td>청킹·임베딩·검색 전략이 부적절함</td>
<td>전처리와 검색 성능을 따로 평가한다.</td>
</tr>
<tr>
<td>문서는 찾았지만 답을 잘못 뽑음</td>
<td>문맥이 부족하거나 노이즈가 많음</td>
<td>청크 품질과 프롬프트를 개선한다.</td>
</tr>
<tr>
<td>출력 형식이 깨짐</td>
<td>LLM 출력은 항상 일정하지 않음</td>
<td>스키마, 파서, 검증기를 둔다.</td>
</tr>
<tr>
<td>대규모 처리 병목</td>
<td>수집·임베딩·검색량이 커짐</td>
<td>배치, 분산 처리, 사전 필터링을 사용한다.</td>
</tr>
<tr>
<td>위험한 코드나 명령 생성</td>
<td>생성 결과를 무검증 실행함</td>
<td>실행 권한 제한, 격리, 사람의 승인을 둔다.</td>
</tr>
<tr>
<td>PDF 표·이미지 추출 실패</td>
<td>눈에 보이는 배치와 텍스트 구조가 다름</td>
<td>전용 파서, OCR, VLM을 문서별로 시험한다.</td>
</tr>
</tbody></table>
<p>핵심은 “RAG를 붙였다”가 아니라 <strong>수집부터 생성 후 검증까지 전체 품질을 관리했는가</strong>다.</p>
<h3 id="32-실패-지점을-세-층으로-나누어-보기">3.2 실패 지점을 세 층으로 나누어 보기</h3>
<p>RAG의 품질은 최종 답 한 줄만 보고 판단하면 원인을 찾기 어렵다.</p>
<ol>
<li><strong>Retrieval Accuracy</strong>: 검색 단계에서 맞는 문서를 가져왔는가?</li>
<li><strong>Response Correctness</strong>: 생성된 답변의 내용이 실제로 맞는가?</li>
<li><strong>Answer Relevance</strong>: 그 답변이 사용자의 질문에 제대로 대응하는가?</li>
</ol>
<p>정답 문서를 못 찾았다면 청킹·임베딩·검색을 고쳐야 한다. 정답 문서를 찾았는데 답이 틀렸다면 프롬프트, 컨텍스트 구성, LLM 또는 출력 검증을 살펴야 한다. 이렇게 층을 나누면 “모델이 나쁘다”라는 막연한 결론을 피할 수 있다.</p>
<hr />
<h2 id="4-기업이-rag를-도입하는-과정">4. 기업이 RAG를 도입하는 과정</h2>
<p>현업에서는 보통 한 번에 완성된 시스템을 만들지 않는다.</p>
<pre><code class="language-text">기술에 대한 거부감
  → 작은 실험과 PoC
  → 위험·역량 검토
  → 내부 챗봇 적용
  → 제한된 고객 업무 적용
  → 사람과 협업하는 에이전트로 확장</code></pre>
<p>이 과정에서 정확성뿐 아니라 데이터 유출, 규제, 지식재산권, 감사 로그, 격리 환경, 사람의 승인(Human-in-the-loop)까지 함께 검토해야 한다.</p>
<h3 id="실무-체크">실무 체크</h3>
<ul>
<li>이 답변의 근거 문서를 보여 줄 수 있는가?</li>
<li>사용자가 접근 권한이 없는 문서가 검색되지 않는가?</li>
<li>문서 개정 시 이전 버전과 최신 버전을 구분하는가?</li>
<li>검색 실패와 생성 실패를 별도로 측정하는가?</li>
<li>결과를 외부 시스템이 바로 실행한다면 승인 절차가 있는가?</li>
</ul>
<hr />
<h2 id="5-rag는-끝났다는-주장-어떻게-이해해야-할까">5. “RAG는 끝났다”는 주장, 어떻게 이해해야 할까?</h2>
<p>긴 컨텍스트를 지원하는 LLM, 도구를 직접 호출하는 에이전트, 지식을 지속해서 요약·연결하는 위키형 시스템이 발전하면서 전통적인 RAG가 필요 없다는 주장도 나온다.</p>
<p>여기서 구분해야 할 것은 <strong>초기형 RAG의 한계</strong>와 <strong>검색 자체의 필요성</strong>이다.</p>
<h3 id="전통적인-rag가-비판받는-이유">전통적인 RAG가 비판받는 이유</h3>
<ul>
<li>질문할 때마다 처음부터 검색한다.</li>
<li>이전 질문에서 얻은 지식을 자동으로 누적하지 않는다.</li>
<li>임베딩은 의미를 압축하므로 미세한 차이를 놓칠 수 있다.</li>
<li>단순 벡터 유사도만으로는 복잡한 의도를 충분히 이해하지 못한다.</li>
</ul>
<h3 id="그래도-검색이-계속-필요한-이유">그래도 검색이 계속 필요한 이유</h3>
<ul>
<li>기업 데이터 대부분은 범용 모델에 학습되어 있지 않다.</li>
<li>모든 문서를 매번 LLM 컨텍스트에 넣으면 비용과 지연이 커진다.</li>
<li>대규모 자료에서는 필요한 범위를 먼저 줄이는 과정이 필요하다.</li>
<li>권한·버전·출처 필터링은 단순히 컨텍스트를 길게 만드는 것으로 해결되지 않는다.</li>
</ul>
<p>따라서 “RAG가 사라진다”기보다 <strong>단순 Top-K 벡터 검색에서 하이브리드 검색, 재정렬, 그래프, 에이전트, 지속적 지식 관리로 진화한다</strong>고 보는 편이 정확하다.</p>
<h3 id="지속적으로-자라는-지식-시스템의-예">지속적으로 자라는 지식 시스템의 예</h3>
<p>위키형 지식 시스템은 다음과 같은 순환 구조를 사용할 수 있다.</p>
<ul>
<li><strong>Ingest</strong>: 새 자료를 분석해 관련 지식 페이지와 인덱스를 갱신한다.</li>
<li><strong>Query</strong>: 복잡한 질문에 답하고, 재사용 가치가 있는 분석 결과를 다시 지식으로 축적한다.</li>
<li><strong>Lint</strong>: 깨진 연결, 상충하는 주장, 빠진 개념을 찾아 정리한다.</li>
</ul>
<p>사람은 좋은 원자료를 제공하고 질문하며, 에이전트는 요약·분류·연결·정비를 담당한다. 이런 방식은 소규모 팀의 깊은 맥락 관리에 유용하지만, 대규모 문서 검색·권한 필터·실시간 원문 조회를 모두 대체한다고 단정하기는 어렵다. 오히려 기존 검색 계층 위에 지속적 지식 관리 계층을 더하는 형태로 이해할 수 있다.</p>
<hr />
<h2 id="6-rag-기법의-진화-지도">6. RAG 기법의 진화 지도</h2>
<p>각 기법의 세부 구현은 뒤에서 배우더라도, 1일차에는 어떤 문제를 해결하려고 등장했는지 알아두자.</p>
<table>
<thead>
<tr>
<th>기법</th>
<th>핵심 아이디어</th>
<th>해결하려는 문제</th>
</tr>
</thead>
<tbody><tr>
<td>Naive RAG</td>
<td>질문과 비슷한 청크를 찾아 답변에 넣음</td>
<td>외부 지식을 간단히 연결</td>
</tr>
<tr>
<td>Advanced RAG</td>
<td>질의 변환, 하이브리드 검색, 재정렬 등 적용</td>
<td>검색 품질 개선</td>
</tr>
<tr>
<td>Self-RAG</td>
<td>검색 필요성과 답변 근거를 모델이 스스로 점검</td>
<td>불필요한 검색과 환각 감소</td>
</tr>
<tr>
<td>Corrective RAG</td>
<td>검색 결과 품질이 낮으면 다시 검색</td>
<td>잘못된 검색 결과 보정</td>
</tr>
<tr>
<td>Adaptive RAG</td>
<td>질문 난이도에 따라 처리 전략 변경</td>
<td>모든 질문에 같은 비용을 쓰는 문제</td>
</tr>
<tr>
<td>Modular RAG</td>
<td>기능을 독립 모듈로 만들고 조합</td>
<td>변경·확장하기 어려운 고정 파이프라인</td>
</tr>
<tr>
<td>Agentic RAG</td>
<td>에이전트가 도구와 검색 순서를 능동적으로 선택</td>
<td>복잡하고 동적인 문제 처리</td>
</tr>
<tr>
<td>Graph RAG</td>
<td>개체와 관계를 그래프로 색인</td>
<td>여러 문서를 잇는 관계형·다단계 질문</td>
</tr>
</tbody></table>
<p>이 표는 발전 연도를 외우기 위한 것이 아니다. <strong>검색, 질문 이해, 구조, 자기 검토 중 어디를 개선한 방식인지</strong> 구분하는 것이 중요하다.</p>
<hr />
<h1 id="part-2-rag-a-to-z---전처리-파이프라인">Part 2. RAG A to Z - 전처리 파이프라인</h1>
<h2 id="7-전체-파이프라인을-먼저-보자">7. 전체 파이프라인을 먼저 보자</h2>
<p>RAG는 크게 두 구간으로 나뉜다.</p>
<h3 id="71-preprocessing-미리-지식-창고-만들기">7.1 Preprocessing: 미리 지식 창고 만들기</h3>
<pre><code class="language-text">원본 문서
  → Document Loader
  → Text Splitter
  → Embedding
  → Vector Store</code></pre>
<h3 id="72-runtime-질문이-왔을-때-답하기">7.2 Runtime: 질문이 왔을 때 답하기</h3>
<pre><code class="language-text">사용자 질문
  → Retriever
  → Prompt
  → LLM
  → Chain
  → 최종 답변</code></pre>
<p>각 부품을 도서관에 비유하면 다음과 같다.</p>
<table>
<thead>
<tr>
<th>단계</th>
<th>역할</th>
<th>도서관 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Document Loader</td>
<td>문서를 읽어 들임</td>
<td>책을 입고한다.</td>
</tr>
<tr>
<td>Text Splitter</td>
<td>검색 가능한 크기로 나눔</td>
<td>책에 장·절·카드를 만든다.</td>
</tr>
<tr>
<td>Embedding</td>
<td>의미를 숫자 벡터로 바꿈</td>
<td>비슷한 주제의 좌표를 부여한다.</td>
</tr>
<tr>
<td>Vector Store</td>
<td>벡터와 출처를 저장함</td>
<td>책과 색인을 보관한다.</td>
</tr>
<tr>
<td>Retriever</td>
<td>질문과 관련된 문서를 찾음</td>
<td>사서가 알맞은 자료를 고른다.</td>
</tr>
<tr>
<td>Prompt</td>
<td>질문과 자료를 묶어 지시함</td>
<td>답변 형식과 참고 범위를 알려 준다.</td>
</tr>
<tr>
<td>LLM</td>
<td>자연어 답변을 만듦</td>
<td>자료를 읽고 설명한다.</td>
</tr>
<tr>
<td>Chain</td>
<td>전체 흐름을 연결함</td>
<td>업무 절차 전체를 자동화한다.</td>
</tr>
</tbody></table>
<p>1일차의 목표는 첫 네 단계를 이해하는 것이다.</p>
<hr />
<h2 id="8-document-loader-문서를-제대로-읽어-들이기">8. Document Loader: 문서를 제대로 읽어 들이기</h2>
<p>Document Loader는 PDF, 워드 파일, 웹페이지, DB, API 등 여러 데이터 원천에서 문서를 가져와 RAG가 다룰 수 있는 형태로 바꾸는 단계다.</p>
<h3 id="81-기본-절차">8.1 기본 절차</h3>
<ol>
<li><strong>데이터 소스 선택</strong>: 어떤 문서가 답변에 필요한지 정한다.</li>
<li><strong>데이터 수집</strong>: 파일 읽기, DB 조회, API 호출 등을 수행한다.</li>
<li><strong>필터링과 정제</strong>: 불필요한 포맷, 중복, 민감 정보 등을 처리한다.</li>
<li><strong>데이터 로드</strong>: 본문과 메타데이터를 공통 형식으로 만든다.</li>
</ol>
<p>개념적인 결과는 다음과 비슷하다.</p>
<pre><code class="language-python">document = {
    &quot;page_content&quot;: &quot;문서에서 추출한 본문&quot;,
    &quot;metadata&quot;: {
        &quot;source&quot;: &quot;원본 식별자&quot;,
        &quot;page&quot;: 7,
        &quot;title&quot;: &quot;문서 제목&quot;
    }
}</code></pre>
<h3 id="82-로더를-고를-때-확인할-것">8.2 로더를 고를 때 확인할 것</h3>
<ul>
<li>한국어가 깨지지 않는가?</li>
<li>특수문자, 수식, 표를 어느 정도 보존하는가?</li>
<li>페이지 번호와 제목 같은 메타데이터를 가져오는가?</li>
<li>문서의 읽기 순서를 올바르게 복원하는가?</li>
<li>대량 문서를 처리하는 속도와 메모리 사용량은 적절한가?</li>
</ul>
<h3 id="실패-예시">실패 예시</h3>
<p>눈에는 두 개의 열로 보이는 PDF를 단순 추출했더니 왼쪽 첫 줄, 오른쪽 첫 줄, 왼쪽 둘째 줄 순으로 섞일 수 있다. 이 상태로 임베딩하면 검색 시스템은 문장이 깨진 자료를 학습하게 된다. 따라서 <strong>파싱 결과를 사람이 직접 표본 검사하는 과정</strong>이 필요하다.</p>
<hr />
<h2 id="9-text-splitter-긴-문서를-검색-가능한-조각으로-나누기">9. Text Splitter: 긴 문서를 검색 가능한 조각으로 나누기</h2>
<p>문서 전체를 하나의 덩어리로 저장하면 세부 내용을 찾기 어렵고, 너무 잘게 나누면 문맥이 끊긴다. Text Splitter는 이 균형을 잡는 단계다.</p>
<h3 id="91-token과-chunk의-차이">9.1 Token과 Chunk의 차이</h3>
<ul>
<li><strong>Token</strong>: 모델이 텍스트를 처리하는 최소 단위. 단어 하나일 수도 있고 단어의 일부일 수도 있다.</li>
<li><strong>Chunk</strong>: 검색과 입력을 위해 여러 토큰을 묶은 문서 조각이다.</li>
</ul>
<p>예를 들어 “사내 카페는 2층에 있고 운영 시간은 9시부터 18시까지다”라는 문장이 여러 토큰으로 나뉘더라도, 검색할 때는 문장이나 문단 전체를 하나의 청크로 유지하는 편이 의미 보존에 유리하다.</p>
<h3 id="92-chunk-size와-chunk-overlap">9.2 Chunk Size와 Chunk Overlap</h3>
<ul>
<li><strong>Chunk size</strong>: 청크 하나의 최대 길이</li>
<li><strong>Chunk overlap</strong>: 인접 청크가 겹치는 길이</li>
</ul>
<pre><code class="language-text">원문: A B C D E F G H I J

청크 1: A B C D E F
청크 2:       E F G H I J
              └─ overlap ─┘</code></pre>
<p>Overlap은 청크 경계에서 문장이 끊겨 의미가 사라지는 것을 줄인다. 하지만 너무 크면 저장량과 중복 검색이 늘어난다. 시작점으로 청크 크기의 약 10~20%를 겹치게 시험할 수 있지만, 정답은 문서 구조와 평가 결과로 정해야 한다.</p>
<h3 id="93-tokenizer-먼저-chunking-먼저">9.3 Tokenizer 먼저? Chunking 먼저?</h3>
<p>두 방식 모두 가능하다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>장점</th>
<th>주의점</th>
<th>어울리는 경우</th>
</tr>
</thead>
<tbody><tr>
<td>전체 토큰화 → 토큰 수로 분할</td>
<td>모델 입력 한도를 정확히 맞추기 쉬움</td>
<td>의미 경계를 자를 수 있음</td>
<td>입력 제한 관리가 중요한 처리</td>
</tr>
<tr>
<td>문장·문단 분할 → 각 청크 토큰화</td>
<td>문서의 의미 구조를 보존하기 쉬움</td>
<td>청크 길이가 들쭉날쭉할 수 있음</td>
<td>검색, 요약, 일반적인 RAG</td>
</tr>
</tbody></table>
<hr />
<h2 id="10-청킹-전략-전체-정리">10. 청킹 전략 전체 정리</h2>
<h3 id="101-fixed-size-chunking">10.1 Fixed-size Chunking</h3>
<p>정해진 문자 수나 토큰 수만큼 기계적으로 자른다.</p>
<ul>
<li>장점: 빠르고 구현이 쉽다.</li>
<li>단점: 문장과 문단의 의미 경계를 끊을 수 있다.</li>
<li>추천: 빠른 기준선(Baseline)을 만들 때</li>
</ul>
<h3 id="102-recursive-chunking">10.2 Recursive Chunking</h3>
<p>먼저 문단 같은 큰 단위로 나누고, 너무 길면 문장, 단어처럼 더 작은 단위로 반복 분할한다.</p>
<ul>
<li>장점: 크기 제한을 지키면서 문서 구조를 어느 정도 보존한다.</li>
<li>단점: 단순 고정 분할보다 처리와 저장 부담이 커질 수 있다.</li>
<li>추천: 구조가 비교적 일정한 일반 문서</li>
</ul>
<h3 id="103-document-based-chunking">10.3 Document-based Chunking</h3>
<p>짧은 문서 하나를 통째로 하나의 청크로 사용한다.</p>
<ul>
<li>장점: 문맥이 온전히 유지된다.</li>
<li>단점: 긴 문서에서는 세부 구절 검색이 어렵다.</li>
<li>추천: 짧은 공지, 메모, FAQ, 요약 보고서</li>
</ul>
<h3 id="104-semantic-chunking">10.4 Semantic Chunking</h3>
<p>길이가 아니라 주제 전환이나 의미 유사성을 기준으로 나눈다.</p>
<ul>
<li>장점: 문맥과 주제가 잘 보존되어 검색 품질을 높일 수 있다.</li>
<li>단점: 계산 비용이 늘고, 의미 경계를 잘못 판단할 수 있다.</li>
<li>추천: 주제가 여러 번 바뀌는 장문</li>
</ul>
<h3 id="105-llm-based-chunking">10.5 LLM-based Chunking</h3>
<p>LLM에게 문서의 의미 구조를 분석시켜 분할한다.</p>
<ul>
<li>장점: 다양한 문서에 유연하게 대응한다.</li>
<li>단점: 비용, 속도, 결과 일관성 문제가 있다.</li>
<li>추천: 규칙만으로 구조를 잡기 어려운 문서</li>
</ul>
<h3 id="106-late-chunking">10.6 Late Chunking</h3>
<p>초기에 문맥을 최대한 유지한 표현을 만든 뒤, 검색 시점에 질문과 관련된 부분을 골라 사용한다.</p>
<ul>
<li>장점: 앞뒤 문맥 손실을 줄일 수 있다.</li>
<li>단점: 실시간 처리 비용과 구현 난도가 높다.</li>
<li>추천: 속도보다 문맥 정확도가 더 중요한 경우</li>
</ul>
<blockquote>
<p>구현 방식은 사용하는 임베딩 모델과 시스템 설계에 따라 다를 수 있다. “원문 전체를 단순히 하나의 벡터로 만든 뒤 나중에 임의로 자른다” 정도로만 이해하면 실제 구현을 지나치게 단순화할 수 있다.</p>
</blockquote>
<h3 id="107-hierarchical-chunking">10.7 Hierarchical Chunking</h3>
<p>상위 섹션과 하위 문단을 함께 저장하고 관계를 유지한다.</p>
<ul>
<li>장점: 작은 단위의 정밀 검색과 큰 단위의 문맥을 함께 얻는다.</li>
<li>단점: 저장량이 늘고, 검색 단계에서 부모·자식 청크를 다루는 로직이 필요하다.</li>
<li>추천: 장·절 구조가 분명한 매뉴얼, 법령, 기술 문서</li>
</ul>
<h3 id="선택-원칙">선택 원칙</h3>
<p>청킹에는 만능 정답이 없다. <strong>페이지 단위나 Recursive Chunking으로 기준선을 만든 뒤, 실제 질문 세트로 검색 성공률을 비교</strong>하는 방식이 안전하다.</p>
<hr />
<h2 id="11-메타데이터-청크의-신분증">11. 메타데이터: 청크의 신분증</h2>
<p>청크 본문만 저장하면 출처, 권한, 최신 버전을 판단하기 어렵다. 그래서 각 청크에는 메타데이터가 필요하다.</p>
<pre><code class="language-json">{
  &quot;content&quot;: &quot;검색 가능한 문서 조각&quot;,
  &quot;metadata&quot;: {
    &quot;chunk_id&quot;: &quot;doc123_0007&quot;,
    &quot;document_id&quot;: &quot;doc123&quot;,
    &quot;title&quot;: &quot;문서 제목&quot;,
    &quot;page&quot;: 7,
    &quot;section&quot;: &quot;10.1&quot;,
    &quot;section_title&quot;: &quot;섹션 제목&quot;,
    &quot;updated_at&quot;: &quot;YYYY-MM-DD&quot;,
    &quot;access_tag&quot;: [&quot;authorized_group&quot;],
    &quot;document_type&quot;: &quot;manual&quot;
  }
}</code></pre>
<p>메타데이터는 다음에 사용된다.</p>
<ul>
<li>답변의 출처와 페이지 표시</li>
<li>특정 문서 유형이나 날짜로 검색 범위 제한</li>
<li>최신 버전만 검색</li>
<li>사용자 권한에 따른 접근 제어</li>
<li>잘못된 답변이 나온 경로 추적</li>
</ul>
<h3 id="보안-주의">보안 주의</h3>
<p>권한 태그를 저장하는 것만으로 보안이 완성되지는 않는다. <strong>검색 전에 서버 측에서 권한 필터를 강제</strong>해야 하며, 사용자가 프롬프트로 우회할 수 없도록 설계해야 한다.</p>
<hr />
<h2 id="12-전처리-실무-pdf와-복잡한-문서는-왜-어려운가">12. 전처리 실무: PDF와 복잡한 문서는 왜 어려운가?</h2>
<h3 id="121-기본-정제-항목">12.1 기본 정제 항목</h3>
<ul>
<li>줄바꿈 때문에 생긴 하이픈 복원</li>
<li>유니코드와 공백 정규화</li>
<li>깨진 글자 비율 점검</li>
<li>반복되는 머리말·꼬리말·페이지 번호 제거</li>
<li>너무 짧고 의미 없는 잔여 텍스트 제거</li>
<li>표의 제목·캡션은 표와 함께 보존</li>
<li>각주는 본문과 분리할지 결정</li>
<li>빈 페이지나 추출 실패 페이지 탐지</li>
</ul>
<p>머리말이 매 페이지 반복되면 검색 결과가 특정 문구로 오염될 수 있다. 반대로 페이지 번호와 제목을 모두 지우면 출처 추적이 어려워진다. 본문에서는 제거하되 메타데이터에는 남기는 식으로 목적을 나눌 수 있다.</p>
<h3 id="122-표-파싱이-어려운-이유">12.2 표 파싱이 어려운 이유</h3>
<p>PDF에서 표는 실제 표가 아니라 화면 위 특정 좌표에 놓인 글자 묶음일 수 있다.</p>
<ul>
<li>행과 열의 관계가 텍스트 추출 과정에서 무너진다.</li>
<li>병합 셀이 빈 값으로 처리될 수 있다.</li>
<li>스캔 PDF에는 선택 가능한 텍스트 레이어가 없다.</li>
<li>중첩 표와 복잡한 헤더는 단순 좌표 규칙으로 복원하기 어렵다.</li>
</ul>
<h3 id="표-유형별-접근">표 유형별 접근</h3>
<table>
<thead>
<tr>
<th>문서 상태</th>
<th>접근</th>
<th>장점</th>
<th>한계</th>
</tr>
</thead>
<tbody><tr>
<td>텍스트 레이어가 있는 단순 표</td>
<td>좌표 기반 PDF 파서</td>
<td>빠르고 비용이 낮음</td>
<td>병합 셀 후처리 필요</td>
</tr>
<tr>
<td>스캔 이미지 표</td>
<td>OCR + 좌표 군집화</td>
<td>이미지에서도 글자 추출 가능</td>
<td>인식·좌표 오차 발생</td>
</tr>
<tr>
<td>복잡한 중첩 표</td>
<td>VLM + JSON 스키마</td>
<td>시각적 구조 이해에 유리</td>
<td>비용과 지연, 검증 필요</td>
</tr>
</tbody></table>
<p>어떤 도구든 일부 페이지만 보고 선택하면 위험하다. 실제 문서 유형별 표본을 만들고, 열 순서와 병합 셀이 제대로 복원되는지 검증해야 한다.</p>
<hr />
<h2 id="13-법령·규정·계약처럼-구조가-복잡한-문서">13. 법령·규정·계약처럼 구조가 복잡한 문서</h2>
<p>이런 문서는 단순히 일정 길이로 자르면 의미가 쉽게 깨진다.</p>
<h3 id="반드시-보존할-관계">반드시 보존할 관계</h3>
<ul>
<li>상위 조·항·호 경로</li>
<li>본문에 딸린 단서와 예외</li>
<li>다른 조항을 인용하거나 준용하는 관계</li>
<li>정의 조항과 본문 용어의 연결</li>
<li>개정일과 적용 시점</li>
</ul>
<p>예를 들어 “제10조에 따른다”라는 문장만 검색되면 실제 제10조의 내용을 알 수 없다. 또 예외 조항을 다른 청크로 떼어 놓으면 일반 규칙만 보고 잘못 답할 수 있다.</p>
<h3 id="세-가지-구조화-전략">세 가지 구조화 전략</h3>
<ol>
<li><p><strong>규칙 기반 계층 파싱</strong></p>
<ul>
<li>조·항·호 번호를 정규식으로 인식하고 트리로 만든다.</li>
<li>번호 체계가 일정하면 정확하고 저렴하다.</li>
<li>불규칙한 문서와 암묵적 참조에 약하다.</li>
</ul>
</li>
<li><p><strong>LLM 기반 구조화</strong></p>
<ul>
<li>원하는 JSON 스키마를 주고 문서 구조와 관계를 추출한다.</li>
<li>“앞 항”, “이전 조항” 같은 표현을 해석하기 쉽다.</li>
<li>비용, 속도, 환각을 검증해야 한다.</li>
</ul>
</li>
<li><p><strong>참조 그래프</strong></p>
<ul>
<li>조항을 노드로, 참조·정의·예외·개정 관계를 엣지로 저장한다.</li>
<li>여러 조항을 거치는 질문에 강하다.</li>
<li>구축과 유지 비용이 크다.</li>
</ul>
</li>
</ol>
<p>실무에서는 규칙 기반 파싱으로 기본 구조를 잡고, LLM이나 그래프로 어려운 관계만 보강하는 혼합 방식도 유용하다.</p>
<hr />
<h2 id="14-embedding-의미를-숫자-좌표로-바꾸기">14. Embedding: 의미를 숫자 좌표로 바꾸기</h2>
<p>Embedding은 텍스트를 의미를 담은 숫자 배열, 즉 <strong>벡터</strong>로 변환하는 과정이다.</p>
<p>예를 들어 다음 문장들은 단어가 완전히 같지 않아도 의미가 비슷하다.</p>
<ul>
<li>“노트북 배터리가 빨리 닳아요.”</li>
<li>“랩톱 전원이 오래가지 않아요.”</li>
</ul>
<p>좋은 임베딩 모델은 두 문장을 벡터 공간에서 가깝게 배치한다. 그래서 사용자의 표현과 문서의 표현이 달라도 의미 기반 검색이 가능해진다.</p>
<pre><code class="language-text">문서 청크 → [0.12, -0.44, 0.07, ...]
사용자 질문 → [0.10, -0.41, 0.09, ...]
                 ↓
             유사도 계산</code></pre>
<h3 id="one-hot-encoding과-learned-embedding">One-hot Encoding과 Learned Embedding</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>특징</th>
<th>한계/장점</th>
</tr>
</thead>
<tbody><tr>
<td>One-hot</td>
<td>단어마다 한 칸만 1이고 나머지는 0</td>
<td>단어 수만큼 차원이 커지고 단어 간 의미 관계를 표현하기 어렵다.</td>
</tr>
<tr>
<td>Learned Embedding</td>
<td>신경망이 의미 관계를 학습한 조밀한 벡터</td>
<td>비슷한 의미는 가깝게, 다른 의미는 멀게 표현할 수 있다.</td>
</tr>
</tbody></table>
<h3 id="token-embedding-encoding-다시-구분하기">Token, Embedding, Encoding 다시 구분하기</h3>
<ul>
<li><strong>Token</strong>: 문자 또는 단어 조각</li>
<li><strong>Embedding</strong>: 토큰이나 문장에 대응하는 의미 벡터</li>
<li><strong>Encoding</strong>: 위치와 주변 문맥까지 반영한 내부 표현</li>
</ul>
<p>같은 단어도 문장에 따라 뜻이 달라질 수 있다. “은행에 간다”의 은행과 “은행나무 아래에 있다”의 은행은 문맥 인코딩이 달라져야 한다.</p>
<hr />
<h2 id="15-임베딩-모델을-선택하는-기준">15. 임베딩 모델을 선택하는 기준</h2>
<p>임베딩 모델을 바꾸면 기존 문서 벡터를 대부분 다시 만들어야 한다. 벡터 차원과 분포가 달라질 수 있기 때문이다. 따라서 단순 API 가격만 보고 고르면 안 된다.</p>
<h3 id="주요-기준">주요 기준</h3>
<ul>
<li><strong>언어</strong>: 한국어와 다국어 성능이 충분한가?</li>
<li><strong>도메인</strong>: 제조, 의료, 법률처럼 전문 용어를 구분하는가?</li>
<li><strong>최대 입력 길이</strong>: 긴 청크를 처리할 수 있는가?</li>
<li><strong>차원</strong>: 저장 공간과 검색 비용을 감당할 수 있는가?</li>
<li><strong>처리 비용</strong>: 최초 문서뿐 아니라 모든 사용자 질문도 임베딩해야 한다.</li>
<li><strong>운영 방식</strong>: 외부 API, 사내 배포, 지연시간, 개인정보 요구를 만족하는가?</li>
</ul>
<p>공개 리더보드는 후보를 줄이는 출발점일 뿐이다. 최종 선택은 <strong>우리 문서와 우리 질문으로 만든 평가 세트</strong>에서 내려야 한다.</p>
<h3 id="간단한-평가-예시">간단한 평가 예시</h3>
<p>고객센터 문서에서 실제 질문 100개를 준비하고, 각 질문의 정답 문서 ID를 표시한다. 후보 모델마다 Top-3 검색 결과 안에 정답 문서가 들어오는 비율을 비교한다. 한국어 오탈자, 약어, 제품명, 전문 용어가 포함된 질문도 반드시 넣는다.</p>
<hr />
<h2 id="16-dimension-벡터의-차원은-무엇인가">16. Dimension: 벡터의 차원은 무엇인가?</h2>
<p>차원은 대상을 설명하는 숫자 축의 개수다.</p>
<ul>
<li>2차원 벡터: <code>[길이, 너비]</code></li>
<li>고차원 텍스트 벡터: 사람이 이름 붙이기 어려운 수백~수천 개의 의미 특징</li>
</ul>
<p>차원이 높으면 복잡한 의미를 표현할 가능성이 커지지만, 저장 공간과 계산량도 증가한다. 너무 높은 차원에서는 데이터가 희박해지고 거리 차이가 뚜렷하지 않아지는 <strong>차원의 저주</strong>도 나타날 수 있다.</p>
<pre><code class="language-text">낮은 차원: 빠르고 작지만 정보 손실 가능
높은 차원: 표현력은 크지만 메모리·검색 비용 증가</code></pre>
<p>중요한 점은 “차원이 높을수록 무조건 좋다”가 아니라 <strong>모델이 정한 차원 안에서 실제 검색 품질과 운영 비용을 함께 비교하는 것</strong>이다.</p>
<hr />
<h2 id="17-vector-database-벡터와-출처를-함께-저장하기">17. Vector Database: 벡터와 출처를 함께 저장하기</h2>
<p>Vector Database는 임베딩 벡터를 저장하고, 질문 벡터와 가까운 자료를 빠르게 찾도록 만든 저장 시스템이다.</p>
<p>일반 관계형 DB가 행과 열을 조건으로 찾는 데 강하다면, Vector DB는 <strong>의미적으로 가까운 벡터 검색</strong>에 특화되어 있다.</p>
<h3 id="핵심-구성-요소">핵심 구성 요소</h3>
<table>
<thead>
<tr>
<th>Vector DB 개념</th>
<th>의미</th>
<th>관계형 DB에 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Collection</td>
<td>같은 목적의 벡터 묶음</td>
<td>Table</td>
</tr>
<tr>
<td>Point/Object</td>
<td>벡터 하나와 부가 정보</td>
<td>Row</td>
</tr>
<tr>
<td>ID</td>
<td>벡터의 고유 식별자</td>
<td>Primary Key</td>
</tr>
<tr>
<td>Vector</td>
<td>임베딩 숫자 배열</td>
<td>일반 필드와 다른 특수 데이터</td>
</tr>
<tr>
<td>Metadata/Payload</td>
<td>출처·날짜·권한 등의 정보</td>
<td>Column 데이터</td>
</tr>
</tbody></table>
<p>제품마다 Collection, Point, Object, Payload, Namespace처럼 용어가 다를 수 있지만 본질은 비슷하다.</p>
<h3 id="vector-db가-필요한-이유">Vector DB가 필요한 이유</h3>
<ul>
<li>대량 벡터에서 빠른 유사도 검색</li>
<li>메타데이터 조건과 의미 검색의 결합</li>
<li>문서 추가·삭제·갱신 관리</li>
<li>데이터 증가에 따른 확장</li>
<li>검색용 인덱스 운영</li>
</ul>
<hr />
<h2 id="18-vector-index-전부-비교하지-않고-빨리-찾는-방법">18. Vector Index: 전부 비교하지 않고 빨리 찾는 방법</h2>
<p>벡터가 100개면 모두 비교해도 괜찮다. 하지만 수백만 개라면 질문 하나마다 모든 벡터와 거리를 계산하는 방식은 느리다. Vector Index는 처음부터 비교 후보를 줄이는 장치다.</p>
<h3 id="181-flat">18.1 Flat</h3>
<p>모든 벡터를 직접 비교한다.</p>
<ul>
<li>장점: 가장 정확한 최근접 이웃을 찾을 수 있다.</li>
<li>단점: 데이터가 커지면 매우 느리다.</li>
<li>추천: 소규모 데이터, 정확도 기준선</li>
</ul>
<h3 id="182-ivf">18.2 IVF</h3>
<p>벡터를 여러 구역으로 나누고, 질문과 가까운 구역만 검색한다.</p>
<ul>
<li>비유: 전국의 모든 집을 보지 않고 먼저 가까운 도시를 고른다.</li>
<li>장점: 검색 범위를 줄여 빠르다.</li>
<li>단점: 잘못된 구역을 고르면 정답을 놓칠 수 있다.</li>
<li>조절: 더 많은 구역을 살피면 정확도는 오르고 속도는 느려진다.</li>
</ul>
<h3 id="183-hnsw">18.3 HNSW</h3>
<p>비슷한 벡터끼리 연결한 다층 그래프를 따라 탐색한다.</p>
<ul>
<li>비유: 고속도로로 먼 거리를 이동한 뒤 지방도로, 골목길 순으로 목적지에 접근한다.</li>
<li>장점: 실시간 검색에서 빠르면서 높은 재현율을 기대할 수 있다.</li>
<li>단점: 인덱스 메모리 사용이 크고 삽입·구축 비용이 있다.</li>
</ul>
<h3 id="184-pq">18.4 PQ</h3>
<p>벡터를 압축해 저장하고 근사 거리로 검색한다.</p>
<ul>
<li>장점: 저장 공간과 계산량을 크게 줄인다.</li>
<li>단점: 압축 과정에서 정보가 손실되어 정확도가 낮아질 수 있다.</li>
<li>추천: 메모리가 제한된 환경이나 초대규모 데이터</li>
</ul>
<hr />
<h2 id="19-recall-latency-memory의-삼각관계">19. Recall, Latency, Memory의 삼각관계</h2>
<p>인덱스 선택에서는 세 가지를 동시에 최고로 만들기 어렵다.</p>
<ul>
<li><strong>Recall</strong>: 실제 정답이 검색 후보 안에 포함되는 비율</li>
<li><strong>Latency</strong>: 검색 요청 하나에 걸리는 시간</li>
<li><strong>Memory</strong>: 벡터와 인덱스를 저장하는 메모리</li>
</ul>
<table>
<thead>
<tr>
<th>방식</th>
<th>Recall</th>
<th>속도</th>
<th>메모리</th>
<th>한 줄 요약</th>
</tr>
</thead>
<tbody><tr>
<td>Flat</td>
<td>매우 높음</td>
<td>느림</td>
<td>큼</td>
<td>정확도를 위해 전부 비교</td>
</tr>
<tr>
<td>IVF</td>
<td>조절 가능</td>
<td>빠름</td>
<td>중간</td>
<td>공간을 나눠 후보 축소</td>
</tr>
<tr>
<td>HNSW</td>
<td>높은 편</td>
<td>매우 빠름</td>
<td>큼</td>
<td>메모리를 써서 실시간 성능 확보</td>
</tr>
<tr>
<td>PQ</td>
<td>낮아질 수 있음</td>
<td>빠름</td>
<td>매우 작음</td>
<td>정확도를 일부 양보하고 압축</td>
</tr>
</tbody></table>
<p>따라서 “최고의 인덱스”가 아니라 서비스 요구에 맞는 인덱스를 고른다.</p>
<ul>
<li>실시간 고객 응대: 낮은 지연시간이 중요</li>
<li>감사·법률 검색: 정답 누락을 줄이는 것이 중요</li>
<li>모바일·엣지: 메모리 절약이 중요</li>
<li>오프라인 분석: 지연시간을 어느 정도 허용 가능</li>
</ul>
<hr />
<h2 id="20-vector-store-선택하기">20. Vector Store 선택하기</h2>
<p>벡터 관련 도구는 크게 로컬 검색 라이브러리, 자체 운영 DB, 관리형 클라우드 서비스로 볼 수 있다.</p>
<table>
<thead>
<tr>
<th>유형</th>
<th>예시</th>
<th>적합한 상황</th>
<th>주의점</th>
</tr>
</thead>
<tbody><tr>
<td>로컬 인덱스/라이브러리</td>
<td>FAISS</td>
<td>실험, 소규모 PoC, 알고리즘 제어</td>
<td>DB의 인증·백업·분산 운영 기능은 직접 구성해야 함</td>
</tr>
<tr>
<td>개발 친화적 로컬 Store</td>
<td>Chroma</td>
<td>빠른 프로토타입</td>
<td>대규모 운영 요구를 별도 검토</td>
</tr>
<tr>
<td>서버형 Vector DB</td>
<td>Milvus</td>
<td>대규모 분산 데이터</td>
<td>인프라와 운영 복잡도</td>
</tr>
<tr>
<td>필터·실시간 중심 DB</td>
<td>Qdrant</td>
<td>벡터 + 메타데이터 조건 검색</td>
<td>규모와 운영 환경별 성능 시험 필요</td>
</tr>
<tr>
<td>관리형 클라우드</td>
<td>Pinecone</td>
<td>인프라 부담을 줄이고 빠르게 상용화</td>
<td>비용, 데이터 위치, 공급자 종속 검토</td>
</tr>
</tbody></table>
<p>제품 이름보다 먼저 요구사항을 적어야 한다.</p>
<h3 id="선택-질문">선택 질문</h3>
<ol>
<li>벡터가 몇 개까지 늘어날 것인가?</li>
<li>초당 검색 요청 수와 목표 응답 시간은 얼마인가?</li>
<li>날짜·조직·권한 같은 메타데이터 필터가 복잡한가?</li>
<li>데이터는 외부 클라우드에 저장해도 되는가?</li>
<li>실시간 추가·수정·삭제가 필요한가?</li>
<li>백업, 복제, 장애 복구, 모니터링이 필요한가?</li>
<li>운영 인력이 DB를 직접 관리할 수 있는가?</li>
</ol>
<p>FAISS는 빠른 벡터 검색을 제공하는 강력한 라이브러리지만, 일반적인 의미의 완전한 데이터베이스 운영 기능 전체를 제공하는 것은 아니다. PoC와 운영 시스템을 같은 기준으로 고르면 안 된다.</p>
<hr />
<h2 id="21-왜-조합이-이렇게-많을까">21. 왜 조합이 이렇게 많을까?</h2>
<p>Document Loader, Text Splitter, Embedding Model, Vector Store마다 선택지가 많다. 각 단계의 후보 수를 곱하면 조합은 금방 수백만 개가 된다.</p>
<p>하지만 모든 조합을 시험할 필요는 없다.</p>
<h3 id="현실적인-실험-순서">현실적인 실험 순서</h3>
<ol>
<li>대표 문서 유형과 대표 질문을 작은 평가 세트로 만든다.</li>
<li>단순하고 재현 가능한 기준선을 만든다.</li>
<li>한 번에 변수 하나만 바꾼다.</li>
<li>검색 품질, 속도, 비용을 함께 기록한다.</li>
<li>운영 요구를 만족하는 후보만 남긴다.</li>
</ol>
<p>예를 들어 다음 순서가 가능하다.</p>
<pre><code class="language-text">기준선
PDF 파서 A + Recursive Chunking + 임베딩 A + Flat 검색

실험 1: 파서만 B로 변경
실험 2: 청크 크기만 변경
실험 3: 임베딩 모델만 변경
실험 4: 인덱스를 HNSW로 변경</code></pre>
<p>여러 요소를 한꺼번에 바꾸면 무엇이 품질을 개선했는지 알 수 없다.</p>
<hr />
<h1 id="part-3-1일차-핵심-연결하기">Part 3. 1일차 핵심 연결하기</h1>
<h2 id="22-하나의-예제로-전체-과정-보기">22. 하나의 예제로 전체 과정 보기</h2>
<p>상황: 전자제품 고객센터가 제품 매뉴얼로 답하는 RAG를 만든다.</p>
<h3 id="1단계-document-loader">1단계: Document Loader</h3>
<p>PDF 매뉴얼을 읽어 본문, 페이지, 제품명, 버전을 추출한다. 표와 다단 문서가 깨지지 않는지 검사한다.</p>
<h3 id="2단계-text-splitter">2단계: Text Splitter</h3>
<p>“충전”, “초기화”, “오류 코드” 같은 섹션을 기준으로 나눈다. 하위 절은 작은 청크로 만들고 상위 섹션 제목을 메타데이터에 남긴다.</p>
<h3 id="3단계-embedding">3단계: Embedding</h3>
<p>각 청크를 의미 벡터로 변환한다. “배터리가 빨리 닳아요”라는 질문과 “사용 시간이 짧아지는 경우”라는 매뉴얼 문장이 가깝게 검색되는지 시험한다.</p>
<h3 id="4단계-vector-store">4단계: Vector Store</h3>
<p>벡터와 함께 제품 모델, 문서 버전, 페이지, 공개 범위를 저장한다. 구형 제품 문서가 최신 제품 질문에 섞이지 않도록 메타데이터 필터를 설계한다.</p>
<h3 id="아직-끝나지-않은-이유">아직 끝나지 않은 이유</h3>
<p>지식 창고만 만들었을 뿐이다. 다음 단계에서는 질문을 벡터로 바꾸고, 후보 문서를 검색하고, 필요하면 재정렬한 뒤, 근거와 질문을 프롬프트에 넣어 답변을 생성해야 한다.</p>
<hr />
<h2 id="23-초보자가-자주-하는-오해">23. 초보자가 자주 하는 오해</h2>
<h3 id="오해-1-rag를-쓰면-환각이-사라진다">오해 1. RAG를 쓰면 환각이 사라진다</h3>
<p>검색이 틀리거나 LLM이 근거를 무시하면 환각은 여전히 생긴다. RAG는 환각을 줄이는 수단이지 완전 제거 장치가 아니다.</p>
<h3 id="오해-2-문서를-vector-db에-넣기만-하면-된다">오해 2. 문서를 Vector DB에 넣기만 하면 된다</h3>
<p>파싱, 청킹, 메타데이터, 권한, 버전 관리가 검색 품질을 좌우한다. 벡터 저장은 전체 파이프라인의 한 단계일 뿐이다.</p>
<h3 id="오해-3-청크는-작을수록-정확하다">오해 3. 청크는 작을수록 정확하다</h3>
<p>너무 작으면 필요한 문맥이 사라지고, 너무 크면 관련 없는 정보가 섞인다. 실제 질문으로 평가해야 한다.</p>
<h3 id="오해-4-차원이-높을수록-좋다">오해 4. 차원이 높을수록 좋다</h3>
<p>표현력과 함께 비용도 증가한다. 모델·인덱스·데이터에 따라 최적점이 다르다.</p>
<h3 id="오해-5-리더보드-1위-모델이-우리-데이터에서도-1위다">오해 5. 리더보드 1위 모델이 우리 데이터에서도 1위다</h3>
<p>언어, 도메인, 문서 길이, 질문 형태가 다르면 결과도 달라진다. 자체 평가가 필수다.</p>
<h3 id="오해-6-vector-store와-vector-db는-항상-같은-뜻이다">오해 6. Vector Store와 Vector DB는 항상 같은 뜻이다</h3>
<p>일상적으로 섞어 쓰지만, 라이브러리 수준의 벡터 검색 도구와 인증·백업·분산 운영까지 갖춘 DB는 운영 범위가 다르다.</p>
<hr />
<h2 id="24-1일차-최종-체크리스트">24. 1일차 최종 체크리스트</h2>
<p>아래 질문에 답할 수 있다면 1일차 핵심을 이해한 것이다.</p>
<ul>
<li><input disabled="" type="checkbox" /> RAG의 Retrieval, Augmented, Generation을 각각 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> RAG와 모델 재학습의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> RAG의 기대 효과와 실무 한계를 함께 말할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Preprocessing과 Runtime을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Document Loader에서 파싱 품질과 메타데이터가 중요한 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> Token과 Chunk의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Chunk size와 overlap의 trade-off를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 고정, 재귀, 문서, 의미, LLM, Late, 계층형 청킹을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 표·스캔 PDF·법령 문서가 어려운 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> Embedding과 Dimension의 의미를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 임베딩 모델을 자체 데이터로 평가해야 하는 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> Vector DB가 벡터와 메타데이터를 함께 저장하는 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> Flat, IVF, HNSW, PQ의 기본 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Recall, Latency, Memory 사이의 trade-off를 이해한다.</li>
<li><input disabled="" type="checkbox" /> PoC용 도구와 운영용 데이터베이스의 요구사항이 다름을 안다.</li>
</ul>
<hr />
<h2 id="25-한-장-요약">25. 한 장 요약</h2>
<pre><code class="language-text">[문제]
LLM은 내부·최신 문서를 모르고, 근거 없는 답을 만들 수 있다.

[RAG의 해법]
질문과 관련된 외부 문서를 찾아 LLM의 답변 근거로 제공한다.

[1일차의 핵심]
좋은 답변은 좋은 지식 창고에서 시작한다.

원본 문서
 → 정확히 읽고
 → 의미가 보존되게 나누고
 → 의미 벡터로 바꾸고
 → 출처·권한·버전과 함께 저장한다.

[설계 원칙]
최고의 단일 도구를 찾기보다
우리 문서·질문·보안·속도·비용에 맞는 조합을
작은 평가 세트로 검증한다.</code></pre>
<p>다음 학습에서는 이렇게 만든 지식 창고에서 실제로 관련 문서를 찾는 Retriever, 유사도, 하이브리드 검색, Reranker, Prompt, LLM, Chain과 평가 방법을 연결하게 된다.</p>
<hr />
<h1 id="part-4-runtime-질문을-받아-답변하는-과정">Part 4. Runtime: 질문을 받아 답변하는 과정</h1>
<h2 id="26-retriever-질문과-관련된-문서를-찾는-검색기">26. Retriever: 질문과 관련된 문서를 찾는 검색기</h2>
<h3 id="261-retriever란">26.1 Retriever란?</h3>
<p>Retriever는 사용자의 질문과 관련된 문서 조각을 Vector DB에서 찾아오는 구성 요소다.</p>
<p>도서관에 비유하면 다음과 같다.</p>
<ul>
<li>Vector DB: 책과 자료가 정리된 서가</li>
<li>사용자 질문: 찾고 싶은 정보</li>
<li>Retriever: 질문을 듣고 관련 책의 위치를 찾아주는 사서</li>
<li>검색 결과: 사서가 골라 준 몇 개의 책 또는 페이지</li>
</ul>
<h3 id="262-기본-검색-과정">26.2 기본 검색 과정</h3>
<pre><code class="language-text">사용자 질문
  ↓
질문을 벡터로 변환
  ↓
저장된 문서 벡터와 유사도 비교
  ↓
가장 가까운 상위 K개 문서 선택
  ↓
문서 내용·위치·메타데이터 반환</code></pre>
<p>여기서 <code>K</code>는 검색 결과로 가져올 문서 수다. <code>top_k=3</code>이면 가장 관련 있다고 판단된 청크 3개를 반환한다.</p>
<h3 id="263-질문과-문서에-같은-embedding-모델을-써야-하는-이유">26.3 질문과 문서에 같은 Embedding 모델을 써야 하는 이유</h3>
<p>벡터의 좌표는 Embedding 모델이 만든 고유한 의미 공간 안에서만 해석할 수 있다. 문서는 A 모델로, 질문은 B 모델로 변환하면 서로 다른 지도에서 위치를 비교하는 것과 비슷해진다.</p>
<p>따라서 특별한 호환 설계가 없다면 문서와 질문에는 같은 Embedding 모델을 사용해야 한다.</p>
<h3 id="264-top-k가-크면-항상-좋을까">26.4 top-k가 크면 항상 좋을까?</h3>
<p>그렇지 않다.</p>
<table>
<thead>
<tr>
<th>설정</th>
<th>장점</th>
<th>위험</th>
</tr>
</thead>
<tbody><tr>
<td>작은 K</td>
<td>핵심 문맥만 전달, 비용 감소</td>
<td>필요한 근거를 놓칠 수 있음</td>
</tr>
<tr>
<td>큰 K</td>
<td>정답 문서가 포함될 가능성 증가</td>
<td>무관한 문서가 섞이고 프롬프트가 길어짐</td>
</tr>
</tbody></table>
<p>예를 들어 휴가 규정 한 문장만 필요한데 관련성이 낮은 문서 20개까지 넣으면, LLM이 오래된 규정이나 다른 부서 규정을 섞어 답할 수 있다.</p>
<hr />
<h2 id="27-cosine-similarity-벡터의-방향을-비교하는-방법">27. Cosine Similarity: 벡터의 방향을 비교하는 방법</h2>
<h3 id="271-핵심-개념">27.1 핵심 개념</h3>
<p>Cosine Similarity는 두 벡터의 <strong>길이보다 방향이 얼마나 비슷한지</strong> 측정한다.</p>
<pre><code class="language-text">cosine_similarity(A, B) = (A · B) / (||A|| × ||B||)</code></pre>
<ul>
<li><code>A · B</code>: 두 벡터의 내적</li>
<li><code>||A||</code>, <code>||B||</code>: 각 벡터의 크기</li>
<li>값의 범위: 일반적으로 <code>-1</code>부터 <code>1</code></li>
</ul>
<table>
<thead>
<tr>
<th align="right">값</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td align="right">1에 가까움</td>
<td>방향이 매우 비슷함</td>
</tr>
<tr>
<td align="right">0에 가까움</td>
<td>서로 관련이 적음</td>
</tr>
<tr>
<td align="right">-1에 가까움</td>
<td>반대 방향</td>
</tr>
</tbody></table>
<h3 id="272-손전등-비유">27.2 손전등 비유</h3>
<p>두 사람이 손전등을 같은 방향으로 비추면 유사도가 높다. 한 사람의 손전등이 훨씬 밝더라도 방향이 같다면 의미는 비슷하다고 판단한다. 이처럼 Cosine Similarity는 벡터의 크기보다 방향에 집중한다.</p>
<h3 id="273-주의할-점">27.3 주의할 점</h3>
<p>Cosine Similarity가 높다고 해서 반드시 업무상 정답이라는 뜻은 아니다. 의미는 비슷하지만 적용 대상, 날짜, 권한이 다른 문서일 수도 있다. 그래서 메타데이터 필터와 재정렬이 함께 필요할 수 있다.</p>
<hr />
<h2 id="28-mmr-관련성과-다양성을-함께-고려하기">28. MMR: 관련성과 다양성을 함께 고려하기</h2>
<h3 id="281-단순-유사도-검색의-문제">28.1 단순 유사도 검색의 문제</h3>
<p>유사도만 기준으로 상위 문서를 가져오면 거의 같은 내용의 청크가 반복될 수 있다.</p>
<pre><code class="language-text">질문: 재택근무의 장단점은?

단순 검색 결과
1. 재택근무는 출퇴근 시간을 줄인다.
2. 재택근무는 통근 시간을 절약한다.
3. 재택근무는 이동 시간을 줄여 준다.</code></pre>
<p>세 문서가 모두 관련은 있지만 사실상 같은 관점이다. 협업 저하나 보안 문제 같은 다른 관점은 놓쳤다.</p>
<h3 id="282-mmr의-아이디어">28.2 MMR의 아이디어</h3>
<p>MMR(Maximal Marginal Relevance)은 다음 두 조건을 함께 본다.</p>
<ol>
<li>질문과 관련이 높은가?</li>
<li>이미 선택한 문서와 너무 비슷하지 않은가?</li>
</ol>
<p>개념적으로는 다음 점수가 가장 높은 문서를 반복해서 선택한다.</p>
<pre><code class="language-text">MMR 점수
= λ × 질문과 후보 문서의 관련성
− (1−λ) × 이미 선택한 문서와의 중복성</code></pre>
<p><code>λ</code>는 0과 1 사이의 값이다.</p>
<ul>
<li><code>λ</code>가 1에 가까움: 질문과의 관련성을 더 중시</li>
<li><code>λ</code>가 0에 가까움: 결과의 다양성을 더 중시</li>
</ul>
<h3 id="283-언제-유용할까">28.3 언제 유용할까?</h3>
<ul>
<li>하나의 사실을 정확히 찾을 때: 일반 유사도 검색이 단순하고 효과적일 수 있음</li>
<li>여러 관점이 필요한 질문: MMR이 유용할 수 있음</li>
<li>보고서·비교·장단점 질문: 중복을 줄이는 효과가 큼</li>
</ul>
<p>MMR도 무조건 우월한 방법은 아니다. 다양성을 지나치게 강조하면 정답과 거리가 있는 문서가 포함될 수 있으므로 실제 질문으로 평가해야 한다.</p>
<hr />
<h2 id="29-hybrid-retriever-키워드-검색과-의미-검색의-결합">29. Hybrid Retriever: 키워드 검색과 의미 검색의 결합</h2>
<h3 id="291-sparse와-dense-검색">29.1 Sparse와 Dense 검색</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>Sparse Retrieval</th>
<th>Dense Retrieval</th>
</tr>
</thead>
<tbody><tr>
<td>중심</td>
<td>단어의 일치와 중요도</td>
<td>문장의 의미</td>
</tr>
<tr>
<td>대표 방법</td>
<td>TF-IDF, BM25, SPLADE</td>
<td>Embedding, Vector Search</td>
</tr>
<tr>
<td>강점</td>
<td>제품명, 코드, 고유명사, 정확한 표현</td>
<td>동의어, 표현이 다른 유사 의미</td>
</tr>
<tr>
<td>약점</td>
<td>단어가 다르면 놓치기 쉬움</td>
<td>정확한 코드·고유명사를 놓칠 수 있음</td>
</tr>
</tbody></table>
<p>예를 들어 질문이 “자동차 보험”이고 문서에는 “차량 보장 상품”이라고 적혀 있다면 의미 검색이 유리할 수 있다. 반대로 <code>ERR-1042</code> 같은 오류 코드는 정확한 키워드 검색이 유리하다.</p>
<h3 id="292-hybrid-검색-흐름">29.2 Hybrid 검색 흐름</h3>
<pre><code class="language-text">사용자 질문
  ├─ Sparse 검색 → BM25 등의 순위
  └─ Dense 검색  → Vector Similarity 순위
                ↓
            결과 순위 결합
                ↓
             최종 후보</code></pre>
<h3 id="293-rrf로-순위를-합치는-이유">29.3 RRF로 순위를 합치는 이유</h3>
<p>BM25 점수와 벡터 유사도 점수는 척도가 다르다. 원점수를 그대로 더하면 한쪽 점수가 부당하게 큰 영향을 줄 수 있다.</p>
<p>RRF(Reciprocal Rank Fusion)는 원점수 대신 <strong>각 검색 결과에서 몇 위인지</strong>를 이용해 순위를 결합한다. 서로 다른 검색기의 점수 범위를 억지로 맞추지 않아도 된다는 장점이 있다.</p>
<p>쉽게 말해 두 명의 심사위원이 서로 다른 채점표를 쓸 때, 점수 자체보다 각 심사위원의 순위를 합쳐 최종 순위를 정하는 방식이다.</p>
<hr />
<h2 id="30-bm25와-splade">30. BM25와 SPLADE</h2>
<h3 id="301-bm25">30.1 BM25</h3>
<p>BM25는 검색어가 문서에 얼마나 중요하게 등장하는지를 계산하는 대표적인 키워드 검색 방법이다.</p>
<p>BM25가 잘 맞는 상황은 다음과 같다.</p>
<ul>
<li>상품 코드, 법 조항, 사람 이름처럼 정확한 문자열이 중요할 때</li>
<li>빠르고 설명 가능한 기준선이 필요할 때</li>
<li>GPU 등 계산 자원이 제한적일 때</li>
</ul>
<p>하지만 질문에는 “자동차”, 문서에는 “차량”만 있을 경우처럼 단어가 다르면 관련 문서를 놓칠 수 있다.</p>
<h3 id="302-splade">30.2 SPLADE</h3>
<p>SPLADE는 학습된 언어 모델을 이용해 문서와 질문을 희소 벡터로 표현한다. 원문에 없는 관련 단어까지 확장할 수 있어 단순 키워드 불일치 문제를 줄인다.</p>
<p>예를 들어 “자동차”라는 질문에서 “차량”, “승용차” 같은 관련 표현에 가중치를 줄 수 있다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>BM25</th>
<th>SPLADE</th>
</tr>
</thead>
<tbody><tr>
<td>학습 모델</td>
<td>불필요</td>
<td>필요</td>
</tr>
<tr>
<td>의미 확장</td>
<td>제한적</td>
<td>가능</td>
</tr>
<tr>
<td>운영 비용</td>
<td>비교적 낮음</td>
<td>비교적 높음</td>
</tr>
<tr>
<td>설명 용이성</td>
<td>높음</td>
<td>상대적으로 낮음</td>
</tr>
<tr>
<td>권장 시작점</td>
<td>기본 기준선</td>
<td>품질 개선 실험</td>
</tr>
</tbody></table>
<p>실무에서는 먼저 BM25로 기준 성능을 만들고, 자체 데이터에서 SPLADE의 품질 향상이 추가 비용을 정당화하는지 확인하는 편이 안전하다.</p>
<h3 id="303-운영-시-함께-볼-것">30.3 운영 시 함께 볼 것</h3>
<ul>
<li><strong>데이터 계보(Data Lineage)</strong>: 어떤 원문과 전처리·모델·인덱스 버전에서 검색 결과가 만들어졌는지 추적한다.</li>
<li><strong>카나리 배포(Canary Deployment)</strong>: 새 검색기를 일부 트래픽에 먼저 적용하고 품질과 장애 여부를 확인한 뒤 확대한다.</li>
</ul>
<hr />
<h2 id="31-lost-in-the-middle과-reranker">31. Lost in the Middle과 Reranker</h2>
<h3 id="311-lost-in-the-middle">31.1 Lost in the Middle</h3>
<p>LLM에 긴 문맥을 주면 모든 위치의 정보를 똑같이 잘 활용할 것 같지만, 실제로는 문맥의 중간에 있는 정보를 놓치는 현상이 생길 수 있다. 이를 <strong>Lost in the Middle</strong>이라고 한다.</p>
<p>따라서 관련 문서를 많이 넣는 것만으로는 충분하지 않다. 정말 중요한 근거를 골라 앞쪽에 배치하고, 중복과 잡음을 줄여야 한다.</p>
<h3 id="312-2단계-검색">31.2 2단계 검색</h3>
<pre><code class="language-text">1단계 Retriever: 많은 문서에서 후보를 빠르게 추출
2단계 Reranker: 후보를 더 정교하게 다시 채점하고 정렬</code></pre>
<p>공항 검색에 비유하면 Retriever는 수많은 여행지 중 후보 도시 20개를 빠르게 고르는 단계이고, Reranker는 예산·시간·취향을 함께 검토해 최종 5개를 정렬하는 단계다.</p>
<h3 id="313-bi-encoder와-cross-encoder">31.3 Bi-Encoder와 Cross-Encoder</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>Bi-Encoder 기반 Retriever</th>
<th>Cross-Encoder 기반 Reranker</th>
</tr>
</thead>
<tbody><tr>
<td>처리 방식</td>
<td>질문과 문서를 각각 벡터화</td>
<td>질문과 문서를 한 쌍으로 함께 입력</td>
</tr>
<tr>
<td>계산 시점</td>
<td>문서 벡터를 미리 계산 가능</td>
<td>요청마다 후보 문서와 함께 계산</td>
</tr>
<tr>
<td>속도</td>
<td>빠름</td>
<td>느림</td>
</tr>
<tr>
<td>정밀도</td>
<td>후보 검색에 적합</td>
<td>세밀한 관련성 판단에 적합</td>
</tr>
<tr>
<td>역할</td>
<td>넓게 찾기</td>
<td>정확히 줄 세우기</td>
</tr>
</tbody></table>
<p>Reranker를 전체 문서에 바로 적용하면 너무 느리다. 그래서 빠른 Retriever가 후보를 줄인 다음, 소수 후보에만 Reranker를 적용한다.</p>
<h3 id="314-모델-조합은-직접-평가해야-한다">31.4 모델 조합은 직접 평가해야 한다</h3>
<p>특정 Reranker가 모든 Embedding 모델과 데이터에서 항상 최고인 것은 아니다. 언어, 문서 길이, 도메인, 후보 수에 따라 결과가 달라진다. 공개 벤치마크는 후보 선정에 활용하고 최종 선택은 자체 평가셋으로 결정한다.</p>
<hr />
<h2 id="32-prompt-검색-결과를-답변-지시문으로-바꾸기">32. Prompt: 검색 결과를 답변 지시문으로 바꾸기</h2>
<h3 id="321-prompt의-역할">32.1 Prompt의 역할</h3>
<p>Retriever가 가져온 문서를 그대로 LLM에 던지는 것이 아니라, 질문·근거·답변 규칙을 하나의 입력으로 구성해야 한다.</p>
<pre><code class="language-text">[Instruction]
아래 문맥만을 근거로 질문에 답하세요.
문맥에 답이 없으면 모른다고 답하세요.

[Context]
검색된 문서 조각들

[Question]
사용자 질문

[Answer]</code></pre>
<h3 id="322-좋은-rag-prompt의-기본-원칙">32.2 좋은 RAG Prompt의 기본 원칙</h3>
<ul>
<li>문맥과 질문을 명확히 구분한다.</li>
<li>근거가 없을 때 추측하지 말도록 지시한다.</li>
<li>필요한 답변 언어와 형식을 지정한다.</li>
<li>출처가 필요하면 문서 ID나 제목을 함께 반환하도록 한다.</li>
<li>서로 충돌하는 문서가 있으면 충돌 사실을 표시하도록 한다.</li>
</ul>
<h3 id="323-모르면-모른다의-한계">32.3 “모르면 모른다”의 한계</h3>
<p>이 한 문장만으로 환각을 완전히 막을 수는 없다. 검색 문서가 틀렸거나, 오래됐거나, 서로 충돌할 수 있기 때문이다. Prompt 규칙과 함께 검색 평가, 메타데이터, 출처 표시가 필요하다.</p>
<hr />
<h2 id="33-llm-문맥을-읽고-자연어-답변-생성하기">33. LLM: 문맥을 읽고 자연어 답변 생성하기</h2>
<p>LLM 단계는 Prompt에 포함된 질문과 검색 문맥을 종합해 최종 답변을 만든다.</p>
<p>핵심 역할은 다음 두 가지다.</p>
<ol>
<li><strong>사용자 의도 이해</strong>: 질문이 무엇을 요구하는지 해석한다.</li>
<li><strong>문맥 기반 생성</strong>: 사전 학습 지식만 사용하지 않고 제공된 근거를 반영해 답한다.</li>
</ol>
<p>중요한 점은 LLM이 검색기의 실수를 자동으로 해결해 주는 만능 장치가 아니라는 것이다.</p>
<pre><code class="language-text">좋은 LLM + 잘못된 검색 결과 → 그럴듯하지만 잘못된 답변 가능
적절한 LLM + 정확한 검색 결과 → 근거 있는 답변 가능성 증가</code></pre>
<p>라이브러리의 모델 초기화 코드는 버전에 따라 바뀔 수 있다. 따라서 특정 예제 코드를 외우기보다 다음 구조를 이해하는 것이 중요하다.</p>
<pre><code class="language-python"># 개념 예시
model = initialize_chat_model(
    model=&quot;사용할 모델&quot;,
    temperature=0,
)</code></pre>
<p><code>temperature=0</code>은 대체로 결과의 무작위성을 줄이는 설정이지만, 완전히 동일하거나 항상 정확한 답을 보장하지는 않는다.</p>
<hr />
<h2 id="34-chain-모든-단계를-하나의-파이프라인으로-연결하기">34. Chain: 모든 단계를 하나의 파이프라인으로 연결하기</h2>
<p>Chain은 지금까지 만든 구성 요소를 순서대로 연결한다.</p>
<pre><code class="language-text">질문
  ↓
Retriever가 관련 문서 검색
  ↓
검색 문서 + 원래 질문을 Prompt에 삽입
  ↓
LLM이 답변 생성
  ↓
Output Parser가 사용하기 좋은 형태로 정리</code></pre>
<p>LangChain의 LCEL에서는 <code>|</code> 기호로 데이터의 흐름을 표현할 수 있다.</p>
<pre><code class="language-python"># 구조를 이해하기 위한 개념 예시
chain = (
    {&quot;context&quot;: retriever, &quot;question&quot;: passthrough}
    | prompt
    | llm
    | output_parser
)

answer = chain.invoke(&quot;질문&quot;)</code></pre>
<p>여기서 핵심은 문법 암기가 아니라 <strong>동일한 질문이 검색용 입력과 Prompt의 question 항목으로 각각 전달되고, 검색 결과는 context 항목으로 들어간다</strong>는 점이다.</p>
<hr />
<h1 id="part-5-rag를-어떻게-평가할까">Part 5. RAG를 어떻게 평가할까?</h1>
<h2 id="35-검색과-답변-생성을-분리해서-평가하기">35. 검색과 답변 생성을 분리해서 평가하기</h2>
<p>RAG 평가는 크게 두 부분으로 나눈다. <strong>평가 기준은 굉장히 중요함</strong></p>
<table>
<thead>
<tr>
<th>평가 대상</th>
<th>확인할 질문</th>
<th>대표 방법</th>
</tr>
</thead>
<tbody><tr>
<td>Retrieval</td>
<td>관련 문서를 잘 찾았는가?</td>
<td>Hit Rate@K, MRR</td>
</tr>
<tr>
<td>Generation</td>
<td>근거에 맞는 좋은 답을 만들었는가?</td>
<td>LLM-as-a-Judge, 사람 평가</td>
</tr>
</tbody></table>
<p>분리해서 보는 이유는 장애 원인을 찾기 위해서다.</p>
<pre><code class="language-text">정답 문서를 못 찾음 → Retrieval 문제
정답 문서를 찾았는데 답이 틀림 → Prompt/LLM/Generation 문제</code></pre>
<p>전체 답변 점수 하나만 보면 어느 단계를 개선해야 하는지 알기 어렵다.</p>
<hr />
<h2 id="36-hit-ratek와-mrr">36. Hit Rate@K와 MRR</h2>
<h3 id="361-hit-ratek">36.1 Hit Rate@K</h3>
<p>상위 K개 검색 결과 안에 정답 문서가 하나라도 들어 있는지를 측정한다.</p>
<pre><code class="language-text">Hit Rate@K
= 상위 K개 안에 정답이 포함된 질문 수 / 전체 질문 수</code></pre>
<p>예를 들어 질문 4개 중 3개의 정답 문서가 상위 3개 안에 있다면 <code>Hit Rate@3 = 3/4 = 0.75</code>다.</p>
<p>장점은 단순하고 직관적이라는 것이다. 단점은 정답이 1위인지 3위인지 구분하지 못한다는 것이다.</p>
<h3 id="362-mrr">36.2 MRR</h3>
<p>MRR(Mean Reciprocal Rank)은 정답 문서가 얼마나 앞에 나타나는지 평가한다.</p>
<pre><code class="language-text">Reciprocal Rank = 1 / 정답 문서의 순위
MRR = 모든 질문의 Reciprocal Rank 평균</code></pre>
<table>
<thead>
<tr>
<th align="right">정답 위치</th>
<th align="right">Reciprocal Rank</th>
</tr>
</thead>
<tbody><tr>
<td align="right">1위</td>
<td align="right">1.0</td>
</tr>
<tr>
<td align="right">2위</td>
<td align="right">0.5</td>
</tr>
<tr>
<td align="right">3위</td>
<td align="right">약 0.333</td>
</tr>
<tr>
<td align="right">검색 실패</td>
<td align="right">0</td>
</tr>
</tbody></table>
<p>주의할 점은 정답이 없을 때 실제로 <code>1/0</code>을 계산하는 것이 아니라, 규칙상 점수를 <code>0</code>으로 처리한다는 것이다.</p>
<h3 id="363-두-지표를-함께-보는-이유">36.3 두 지표를 함께 보는 이유</h3>
<ul>
<li>Hit Rate@K: 정답을 후보 안에 넣었는가?</li>
<li>MRR: 그 정답을 얼마나 앞에 놓았는가?</li>
</ul>
<p>Hit Rate가 같더라도 MRR이 높은 검색기는 정답을 더 앞에 배치한다. 이는 제한된 문맥 창을 사용하고 중요한 근거를 앞에 두어야 하는 RAG에서 의미가 크다.</p>
<p>또한 이 지표들은 모델을 학습시키는 방법이 아니라, 준비된 정답 데이터로 검색 성능을 <strong>측정하는 방법</strong>이다.</p>
<hr />
<h2 id="37-llm-as-a-judge로-답변-평가하기">37. LLM-as-a-Judge로 답변 평가하기</h2>
<p>LLM-as-a-Judge는 별도의 LLM에게 평가자 역할을 맡겨 생성 답변을 채점하는 방법이다.</p>
<h3 id="371-평가-항목-예시">37.1 평가 항목 예시</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>확인 내용</th>
</tr>
</thead>
<tbody><tr>
<td>정확성</td>
<td>사실적 오류가 없는가?</td>
</tr>
<tr>
<td>충실성</td>
<td>답변이 제공된 문맥에 근거하는가?</td>
</tr>
<tr>
<td>관련성</td>
<td>질문에 직접 답하는가?</td>
</tr>
<tr>
<td>일관성</td>
<td>앞뒤 논리가 맞는가?</td>
</tr>
<tr>
<td>명확성</td>
<td>쉽게 이해할 수 있는가?</td>
</tr>
<tr>
<td>유창성</td>
<td>문장이 자연스러운가?</td>
</tr>
<tr>
<td>간결성</td>
<td>불필요한 반복이 없는가?</td>
</tr>
</tbody></table>
<h3 id="372-평가-prompt의-원칙">37.2 평가 Prompt의 원칙</h3>
<ul>
<li>“좋은 답인지 평가하라”처럼 모호하게 쓰지 않는다.</li>
<li>각 항목의 의미와 점수 기준을 구체적으로 제시한다.</li>
<li>점수와 판단 근거를 정해진 형식으로 출력하게 한다.</li>
<li>필요하면 반복 평가나 여러 모델의 교차 평가로 편차를 확인한다.</li>
<li>의료·법률·사내 규정 등 도메인별 중요 기준을 별도로 반영한다.</li>
</ul>
<h3 id="373-한계">37.3 한계</h3>
<p>LLM 평가자도 편향되거나 지나치게 관대할 수 있다. 그러므로 중요한 서비스에서는 사람 평가 표본과 비교해 평가자가 실제 품질을 잘 구분하는지 확인해야 한다.</p>
<hr />
<h2 id="38-retrieve-평가-데이터셋-만들기">38. Retrieve 평가 데이터셋 만들기</h2>
<p>검색기를 평가하려면 최소한 다음 쌍이 필요하다.</p>
<pre><code class="language-text">(질문, 정답 문서 또는 정답 청크 ID)</code></pre>
<p>기존 사용자 질문과 정답 문서가 충분하지 않다면 LLM을 이용해 합성 평가 데이터를 만들 수 있다.</p>
<pre><code class="language-text">정답 청크를 LLM에 제공
  ↓
그 청크만으로 답할 수 있는 질문 생성
  ↓
(생성 질문, 정답 청크 ID) 저장</code></pre>
<p>예를 들어 “환불은 결제 후 7일 이내 가능하다”는 청크가 있다면, “환불 신청은 언제까지 가능한가요?”라는 질문을 생성할 수 있다.</p>
<h3 id="합성-데이터의-주의점">합성 데이터의 주의점</h3>
<ul>
<li>실제 사용자가 하지 않을 어색한 질문이 만들어질 수 있다.</li>
<li>질문에 정답이 너무 직접적으로 드러날 수 있다.</li>
<li>하나의 청크만 보고 만든 질문은 여러 문서를 종합해야 하는 현실 질문을 반영하지 못할 수 있다.</li>
<li>생성된 질문과 정답 연결이 올바른지 사람이 일부 검수해야 한다.</li>
</ul>
<p>따라서 합성 데이터는 빠른 출발점으로 사용하고, 운영 중 수집한 실제 질문과 전문가 검수 데이터로 보완한다.</p>
<hr />
<h2 id="39-내-데이터에-맞는-retriever-비교하기">39. 내 데이터에 맞는 Retriever 비교하기</h2>
<p>비교 후보는 다음처럼 다양하다.</p>
<ul>
<li>Vector Similarity: 의미가 가까운 문서를 검색</li>
<li>MMR: 관련성과 다양성을 함께 고려</li>
<li>BM25: 키워드 일치 중심</li>
<li>한국어 형태소 분석 기반 BM25: 한국어의 조사·어미를 고려</li>
<li>Ensemble: 키워드 검색과 벡터 검색 결합</li>
<li>MultiQuery Retriever: LLM이 질문을 여러 표현으로 바꾸어 각각 검색</li>
<li>Parent Document Retriever: 작은 청크로 정확히 찾고 더 큰 상위 문서를 반환</li>
</ul>
<p>교재의 비교 수치는 특정 평가 데이터에서 나온 예시다. 다른 데이터에도 같은 순위가 재현된다는 보장은 없다. 중요한 결론은 <strong>우리 데이터와 실제 질문으로 Hit@1·Hit@3·Hit@5·MRR을 비교해야 한다</strong>는 것이다.</p>
<h3 id="실무-평가-순서-예시">실무 평가 순서 예시</h3>
<ol>
<li>대표 질문과 정답 문서를 준비한다.</li>
<li>동일한 데이터와 필터 조건으로 검색기들을 실행한다.</li>
<li>Hit Rate@K와 MRR을 계산한다.</li>
<li>품질뿐 아니라 지연 시간과 비용도 기록한다.</li>
<li>실패 질문을 유형별로 분석한다.</li>
<li>가장 단순한 방법 대비 개선 효과가 충분한 조합을 선택한다.</li>
</ol>
<hr />
<h1 id="part-6-요약-기반-문서-검색">Part 6. 요약 기반 문서 검색</h1>
<h2 id="40-왜-원문-대신-요약을-검색할까">40. 왜 원문 대신 요약을 검색할까?</h2>
<p>긴 문서를 그대로 잘게 나누면 청크가 많아지고 핵심 주제가 여러 곳에 흩어질 수 있다. 문서를 먼저 요약해 검색용 표현을 만들면 검색 공간을 줄이고 핵심 주제 중심으로 비교할 수 있다.</p>
<pre><code class="language-text">문서 수집·로딩
  ↓
문서 요약 생성
  ↓
원문과 요약을 연결해 저장
  ↓
질문과 유사한 요약 검색
  ↓
선택된 요약 또는 연결된 원문으로 답변 생성</code></pre>
<h3 id="401-저장-방법">40.1 저장 방법</h3>
<ul>
<li>원문 메타데이터의 <code>summary</code> 필드에 요약을 함께 저장</li>
<li>검색용 요약 문서를 별도로 저장하고 원문 ID를 연결</li>
</ul>
<p>두 방식 모두 검색 결과에서 원문으로 돌아갈 수 있어야 한다. 요약만 남기면 세부 근거를 확인하기 어렵다.</p>
<h3 id="402-긴-문서의-요약-방식">40.2 긴 문서의 요약 방식</h3>
<ul>
<li><strong>Map-Reduce</strong>: 여러 조각을 각각 요약한 뒤 마지막에 합친다.</li>
<li><strong>Map-Refine</strong>: 첫 요약을 만들고 다음 조각을 읽을 때마다 기존 요약을 보완한다.</li>
<li><strong>Clustering Map-Reduce</strong>: 비슷한 조각을 묶어 대표 내용을 요약한 뒤 통합한다.</li>
</ul>
<h3 id="403-장점과-위험">40.3 장점과 위험</h3>
<table>
<thead>
<tr>
<th>장점</th>
<th>위험</th>
</tr>
</thead>
<tbody><tr>
<td>검색 대상이 짧아져 속도·비용을 줄일 수 있음</td>
<td>요약에서 빠진 정보는 검색하기 어려움</td>
</tr>
<tr>
<td>문서의 전체 주제를 잘 표현할 수 있음</td>
<td>요약 오류가 검색과 답변에 전파될 수 있음</td>
</tr>
<tr>
<td>불필요한 세부 표현의 영향을 줄일 수 있음</td>
<td>고유명사·숫자·예외 조건이 손실될 수 있음</td>
</tr>
</tbody></table>
<p>따라서 요약 기반 검색은 원문을 버리는 기술이 아니다. <strong>요약으로 후보를 찾고, 원문에서 근거를 확인하는 구조</strong>가 안전하다.</p>
<hr />
<h2 id="41-chain-of-density-같은-길이에-더-많은-핵심-담기">41. Chain of Density: 같은 길이에 더 많은 핵심 담기</h2>
<p>Chain of Density는 요약을 한 번에 완성하지 않고 여러 번 개선하면서 정보 밀도를 높이는 전략이다.</p>
<pre><code class="language-text">1차: 매우 간결한 요약 생성
  ↓
누락된 핵심 개체·키워드 탐색
  ↓
기존 길이를 크게 늘리지 않고 핵심 정보 추가
  ↓
여러 차례 반복해 밀도 향상</code></pre>
<p>예를 들어 첫 요약이 “회사는 신규 검색 시스템을 도입했다”라면, 다음 단계에서는 빠진 정보인 도입 시기, 적용 부서, 목적, 주요 성과 등을 선별해 비슷한 길이 안에 넣는다.</p>
<h3 id="요약-기반-검색과의-차이">요약 기반 검색과의 차이</h3>
<table>
<thead>
<tr>
<th>개념</th>
<th>목적</th>
</tr>
</thead>
<tbody><tr>
<td>요약 기반 검색</td>
<td>짧은 요약을 검색 대상으로 사용해 검색 효율과 관련성을 높임</td>
</tr>
<tr>
<td>Chain of Density</td>
<td>요약을 반복 개선해 핵심 정보의 밀도와 요약 품질을 높임</td>
</tr>
</tbody></table>
<p>Chain of Density를 적용하면 정보가 풍부한 검색용 요약을 만들 수 있지만, 반복 호출만큼 비용과 시간이 증가한다. 반복 횟수가 늘수록 항상 좋아지는 것도 아니므로 품질이 포화되는 지점을 평가해야 한다.</p>
<hr />
<h2 id="42-96페이지까지-전체-흐름">42. 96페이지까지 전체 흐름</h2>
<pre><code class="language-text">[Preprocessing / Indexing]
원본 문서
 → Loader
 → Text Splitter
 → Embedding
 → Vector DB

[Runtime]
사용자 질문
 → Retriever
 → 필요 시 Hybrid Search
 → 필요 시 Reranker
 → Prompt에 질문과 근거 결합
 → LLM
 → 최종 답변

[Evaluation]
Retrieval: Hit Rate@K + MRR
Generation: 정확성·충실성·관련성 등을 평가

[Advanced Tip]
긴 문서는 검색용 요약을 만들고 원문과 연결
 → 필요하면 Chain of Density로 요약 밀도 개선</code></pre>
<h3 id="최종-체크리스트">최종 체크리스트</h3>
<ul>
<li><input disabled="" type="checkbox" /> Retriever가 질문 벡터와 문서 벡터를 비교하는 과정을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Cosine Similarity와 MMR의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Sparse, Dense, Hybrid 검색의 장단점을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> BM25와 SPLADE의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Retriever와 Reranker의 역할 및 속도 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> Lost in the Middle이 왜 문서 선택과 정렬에 영향을 주는지 안다.</li>
<li><input disabled="" type="checkbox" /> Prompt, LLM, Chain이 어떻게 연결되는지 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Hit Rate@K와 MRR을 함께 보는 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> LLM-as-a-Judge의 평가 기준과 한계를 안다.</li>
<li><input disabled="" type="checkbox" /> 합성 검색 평가 데이터의 장점과 위험을 안다.</li>
<li><input disabled="" type="checkbox" /> 공개 벤치마크보다 자체 데이터 평가가 중요한 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> 요약 기반 검색과 Chain of Density의 목적을 구분할 수 있다.</li>
</ul>
<p>오늘 범위의 핵심은 다음 한 문장으로 정리할 수 있다.</p>
<blockquote>
<p>RAG의 품질은 좋은 모델 하나로 결정되는 것이 아니라, 관련 근거를 잘 찾고, 정확히 재정렬하고, 명확한 지시와 함께 전달한 뒤, 검색과 답변을 각각 평가하는 전체 과정에서 결정된다.</p>
</blockquote>