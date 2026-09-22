<h2 id="rag-pipeline-설계-및-구축---2">RAG Pipeline 설계 및 구축 - 2</h2>
<blockquote>
<p>오늘의 한 문장: <strong>좋은 RAG는 문서를 많이 넣는 시스템이 아니라, 질문에 맞는 근거를 고르고 검증하며 필요할 때 경로를 바꾸는 시스템이다.</strong></p>
</blockquote>
<p>이 글은 1일차에서 만든 지식 창고를 더 똑똑하게 활용하는 방법을 다룬다. 단순한 검색 RAG의 한계를 살펴보고, 검색 전·중·후에 품질을 높이는 Advanced RAG, 스스로 점검하는 Self-RAG, 조립 가능한 Modular RAG, 상황에 맞게 도구를 선택하는 Agentic RAG를 순서대로 이해한다. 마지막에는 LangGraph의 기본 구조와 RAG 평가 방법을 연결한다.</p>
<p>특정 조직·개인·내부 문서의 식별 정보와 민감할 수 있는 예시는 제외하고, 공개 가능한 일반 개념과 재구성한 예시만 사용했다.</p>
<hr />
<h2 id="0-2일차-학습-지도">0. 2일차 학습 지도</h2>
<pre><code class="language-text">긴 문서를 요약하는 전략
        ↓
Naive RAG가 실패하는 이유
        ↓
Advanced RAG: 검색 전·중·후 품질 개선
        ↓
Self-RAG: 검색과 답변을 스스로 점검
        ↓
Modular RAG: 필요한 기능을 조립하고 분기
        ↓
Agentic RAG: 도구와 경로를 능동적으로 선택
        ↓
LangGraph: 상태(State)로 흐름을 구현
        ↓
RAG 평가: 검색과 생성 품질을 측정</code></pre>
<p>1일차가 &quot;검색 가능한 지식 창고&quot;를 만드는 날이었다면, 2일차는 <strong>그 창고를 어떤 전략으로 활용해야 정확한 답에 가까워지는가</strong>를 배우는 날이다.</p>
<hr />
<h1 id="part-1-긴-문서를-다루는-요약-전략">Part 1. 긴 문서를 다루는 요약 전략</h1>
<h2 id="1-왜-요약-전략이-필요한가">1. 왜 요약 전략이 필요한가?</h2>
<p>긴 문서를 한 번에 LLM에 넣으면 입력 길이 제한, 비용, 중요한 정보가 묻히는 문제가 생긴다. 그래서 문서 길이와 목적에 맞게 요약 방식을 선택한다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>흐름</th>
<th>잘 맞는 상황</th>
<th>주의점</th>
</tr>
</thead>
<tbody><tr>
<td>Stuff (Vanilla)</td>
<td>전체 문서 → LLM 한 번</td>
<td>문서가 짧을 때</td>
<td>긴 문서는 한도를 넘거나 핵심이 묻힐 수 있음</td>
</tr>
<tr>
<td>Map-Reduce</td>
<td>청크별 요약 → 요약들을 통합</td>
<td>긴 문서를 빠르게 요약할 때</td>
<td>청크 간 연결 정보가 빠질 수 있음</td>
</tr>
<tr>
<td>Refine</td>
<td>첫 요약 → 다음 청크마다 점진 보완</td>
<td>문서 순서와 흐름이 중요할 때</td>
<td>순차 처리라 느리고 앞선 요약 오류가 누적될 수 있음</td>
</tr>
</tbody></table>
<h3 id="11-stuff-가장-단순한-방식">1.1 Stuff: 가장 단순한 방식</h3>
<pre><code class="language-text">작은 문서 전체
      ↓
     LLM
      ↓
    요약문</code></pre>
<p>회의록 한 장처럼 짧은 문서에는 간단하고 효과적이다. 하지만 수십 페이지 문서를 모두 넣는 방식은 컨텍스트가 길어져 비용이 커지고, 중간의 중요한 내용을 놓칠 수 있다.</p>
<h3 id="12-map-reduce-나누어-요약하고-합치기">1.2 Map-Reduce: 나누어 요약하고 합치기</h3>
<pre><code class="language-text">긴 문서
  ├─ 청크 1 → 요약 1
  ├─ 청크 2 → 요약 2
  ├─ 청크 3 → 요약 3
  └─ 청크 N → 요약 N
                ↓
           요약들을 통합
                ↓
             최종 요약</code></pre>
<p>청크별 처리 일부를 병렬화할 수 있어 긴 문서에 유리하다. 반면 서로 다른 청크에 흩어진 원인과 결과를 최종 통합 과정에서 놓칠 수 있다.</p>
<h3 id="13-refine-기존-요약을-계속-다듬기">1.3 Refine: 기존 요약을 계속 다듬기</h3>
<pre><code class="language-text">청크 1 → 첫 요약
첫 요약 + 청크 2 → 갱신 요약
갱신 요약 + 청크 3 → 다시 갱신
                    ↓
                 최종 요약</code></pre>
<p>시간 순서가 중요한 사건 기록이나 보고서에는 유용하다. 다만 처음 요약이 잘못된 방향으로 시작하면 이후 단계도 그 영향을 받을 수 있다.</p>
<h3 id="초보자-판단-기준">초보자 판단 기준</h3>
<pre><code class="language-text">짧은 문서인가?                 → Stuff부터
긴 문서를 빠르게 훑어야 하나?   → Map-Reduce
문서의 순서와 누적 흐름이 중요한가? → Refine</code></pre>
<p>요약은 원문을 대체하는 장치가 아니다. 검색용 요약을 만들더라도 <strong>원문 ID와 출처를 연결해 두고, 최종 답변 전에는 원문 근거를 확인</strong>하는 편이 안전하다.</p>
<hr />
<h1 id="part-2-naive-rag의-한계와-advanced-rag">Part 2. Naive RAG의 한계와 Advanced RAG</h1>
<h2 id="2-naive-rag란">2. Naive RAG란?</h2>
<p>Naive RAG는 가장 기본적인 RAG 흐름이다.</p>
<pre><code class="language-text">질문
  → 질문 임베딩
  → Vector DB에서 Top-K 청크 검색
  → 검색 청크 + 질문을 LLM에 전달
  → 답변</code></pre>
<p>구현은 빠르고 이해하기 쉽다. 하지만 &quot;벡터 유사도 상위 문서&quot;가 항상 &quot;질문에 가장 잘 답하는 문서&quot;는 아니다.</p>
<h2 id="3-naive-rag가-실패하는-대표-이유">3. Naive RAG가 실패하는 대표 이유</h2>
<h3 id="31-의미-유사도와-질문-의도는-다르다">3.1 의미 유사도와 질문 의도는 다르다</h3>
<p>사용자가 다음처럼 비교를 요구한다고 하자.</p>
<blockquote>
<p>&quot;A와 B의 차이점은 무엇인가요?&quot;</p>
</blockquote>
<p>벡터 검색은 A 또는 B 설명 하나만 매우 비슷하다고 판단해 가져올 수 있다. 하지만 비교 질문에는 <strong>A의 근거와 B의 근거, 그리고 비교 기준</strong>이 함께 필요하다.</p>
<pre><code class="language-text">벡터 유사도 높음 ≠ 질문 의도에 완전히 적합함</code></pre>
<h3 id="32-검색-결과가-많을수록-좋은-것은-아니다">3.2 검색 결과가 많을수록 좋은 것은 아니다</h3>
<p>관련 문서를 많이 넣으면 정답 근거를 포함할 가능성은 높아진다. 하지만 중복·오래된 규정·무관한 설명까지 함께 들어가면 LLM이 핵심을 고르기 어려워진다.</p>
<pre><code class="language-text">좋은 Context = 많은 Context가 아니라
질문에 필요한 근거가 선명하게 정리된 Context</code></pre>
<h3 id="33-고정-청크는-문맥을-끊을-수-있다">3.3 고정 청크는 문맥을 끊을 수 있다</h3>
<p>정답이 두 청크의 경계에 걸리면 하나만 검색됐을 때 조건이나 예외가 빠질 수 있다. 1일차에서 배운 Chunk size, overlap, 구조 기반 청킹이 중요한 이유다.</p>
<h3 id="34-단일-쿼리는-복잡한-질문에-약하다">3.4 단일 쿼리는 복잡한 질문에 약하다</h3>
<p>&quot;두 회사의 최근 3년 실적과 차이점을 비교해 줘&quot;는 사실상 여러 질문을 포함한다. 질문 하나를 벡터 하나로 바꿔 검색하면 필요한 근거 일부를 놓치기 쉽다.</p>
<h3 id="35-피드백-루프가-없다">3.5 피드백 루프가 없다</h3>
<p>기본 RAG는 답을 생성하고 끝난다. 검색 결과가 틀렸는지, 답이 근거에 충실한지, 다시 검색해야 하는지 판단하는 단계가 없다.</p>
<h2 id="4-advanced-rag-네-구간에서-검색-품질-높이기">4. Advanced RAG: 네 구간에서 검색 품질 높이기</h2>
<p>Advanced RAG는 기본 흐름을 버리는 기술이 아니다. <strong>검색 전·검색·검색 후를 더 정교하게 만드는 접근</strong>이다.</p>
<table>
<thead>
<tr>
<th>구간</th>
<th>핵심 질문</th>
<th>대표 전략</th>
</tr>
</thead>
<tbody><tr>
<td>Indexing</td>
<td>문서를 어떻게 저장할까?</td>
<td>계층 구조, 의미 기반 청킹</td>
</tr>
<tr>
<td>Pre-Retrieval</td>
<td>무엇을 검색할까?</td>
<td>Query Rewrite, Routing, Expansion</td>
</tr>
<tr>
<td>Retrieval</td>
<td>어떻게 찾을까?</td>
<td>Hybrid Search, Multi-hop Search</td>
</tr>
<tr>
<td>Post-Retrieval</td>
<td>무엇을 LLM에 보낼까?</td>
<td>Reranker, Reorder, Compression</td>
</tr>
</tbody></table>
<pre><code class="language-text">문서 준비 → 질문 개선 → 더 좋은 검색 → 결과 정제 → LLM 답변</code></pre>
<hr />
<h2 id="5-advanced-rag---indexing">5. Advanced RAG - Indexing</h2>
<h3 id="51-hierarchical-structure-문서의-계층을-보존하기">5.1 Hierarchical Structure: 문서의 계층을 보존하기</h3>
<p>일반 고정 청킹은 문서를 일정 크기로만 나눈다. 계층형 구조는 문서의 논리적 구조를 함께 저장한다.</p>
<pre><code class="language-text">문서
 ├─ 1장: 휴가 규정
 │   ├─ 1.1 신청 조건
 │   ├─ 1.2 승인 절차
 │   └─ 1.3 예외 사항
 └─ 2장: 재택근무 규정</code></pre>
<p>질문이 &quot;재택근무 승인 절차&quot;라면, 먼저 관련 장을 좁히고 그 아래 절과 문단을 찾을 수 있다.</p>
<table>
<thead>
<tr>
<th>장점</th>
<th>trade-off</th>
</tr>
</thead>
<tbody><tr>
<td>제목·소제목·문단의 관계를 유지</td>
<td>인덱싱 설계가 복잡해짐</td>
</tr>
<tr>
<td>작은 청크로 정확히 찾고 큰 문맥을 반환 가능</td>
<td>검색 단계가 여러 번 필요할 수 있음</td>
</tr>
<tr>
<td>Multi-hop 질문에 활용 가능</td>
<td>메타데이터와 ID 연결 관리가 필요</td>
</tr>
</tbody></table>
<p>ParentDocumentRetriever는 이런 계층 구조를 활용하는 대표적인 구현 예다.</p>
<h3 id="52-semantic-chunking-의미가-바뀌는-곳에서-나누기">5.2 Semantic Chunking: 의미가 바뀌는 곳에서 나누기</h3>
<p>Semantic Chunking은 글자 수가 아니라 문장·문단 간 의미 변화로 청크 경계를 찾는다.</p>
<pre><code class="language-text">[휴가 신청 조건과 승인 절차]  ← 같은 주제라 함께 유지

[보안 위반 시 처리 절차]      ← 주제가 바뀌므로 새 청크</code></pre>
<p>고정 청킹보다 답에 필요한 정보가 한 청크에 모일 수 있지만, 임베딩 계산 비용과 처리 시간이 늘 수 있다. 문서 구조가 이미 명확한 규정집이라면 구분자 기반 청킹이 오히려 안정적일 수도 있다.</p>
<h3 id="53-graph-rag-관계를-함께-저장하기">5.3 Graph RAG: 관계를 함께 저장하기</h3>
<p>Graph RAG는 문서를 평평한 청크 모음으로만 보지 않고, 사람·제품·정책·조직처럼 중요한 개체(Entity)와 그 관계(Relation)를 그래프로 표현하는 접근이다.</p>
<pre><code class="language-text">정책 A ── 적용 대상 ── 부서 B
정책 A ── 개정됨 ── 정책 C
부서 B ── 담당함 ── 업무 D</code></pre>
<p>&quot;정책 A가 바뀐 뒤 부서 B의 업무 D는 어떻게 달라졌는가&quot;처럼 여러 관계를 따라가야 하는 Multi-hop 질문에서 특히 도움이 될 수 있다. 실무에서는 모든 문서를 그래프로 바꾸기보다, 벡터 검색으로 후보를 찾고 관계 검색이 필요한 질문에만 Graph RAG를 함께 쓰는 하이브리드 구조를 자주 검토한다.</p>
<hr />
<h2 id="6-advanced-rag---pre-retrieval">6. Advanced RAG - Pre-Retrieval</h2>
<p>Pre-Retrieval은 검색하기 전에 질문을 더 검색하기 좋은 형태로 바꾸는 단계다.</p>
<table>
<thead>
<tr>
<th>전략</th>
<th>하는 일</th>
<th>쉬운 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Query Rewriting</td>
<td>모호한 질문을 명확한 검색어로 재작성</td>
<td>질문을 검색 엔진이 이해하게 고쳐 쓰기</td>
</tr>
<tr>
<td>Query Routing</td>
<td>알맞은 데이터 소스·검색기로 보냄</td>
<td>어느 서가로 갈지 결정</td>
</tr>
<tr>
<td>Query Expansion</td>
<td>관련 키워드를 추가</td>
<td>연관된 단어도 함께 찾기</td>
</tr>
<tr>
<td>Query Transformation</td>
<td>복합 질문을 하위 질문으로 바꿈</td>
<td>큰 문제를 작은 문제로 분해</td>
</tr>
</tbody></table>
<h3 id="61-query-rewriting">6.1 Query Rewriting</h3>
<pre><code class="language-text">원래 질문: &quot;A와 B 중 무엇이 더 좋아요?&quot;
재작성: &quot;A와 B를 구조, 학습 방식, 사용 사례 기준으로 비교해 주세요.&quot;</code></pre>
<p>재작성의 목적은 말투를 예쁘게 만드는 것이 아니라 <strong>검색에 필요한 조건과 비교 기준을 드러내는 것</strong>이다.</p>
<h3 id="62-query-routing">6.2 Query Routing</h3>
<p>같은 질문이라도 필요한 데이터가 다르다.</p>
<pre><code class="language-text">&quot;올해 ESG 수치와 정책은?&quot;
  ├─ 정책 설명 → 문서 Vector DB
  └─ 최신 수치 → 관계형 DB 또는 API</code></pre>
<p>Routing은 질문을 분류해 적절한 검색 소스, Retriever, Prompt, 모델을 선택하는 전략이다. 잘못 분기하면 이후 검색이 아무리 좋아도 답이 틀릴 수 있다.</p>
<h3 id="63-query-expansion">6.3 Query Expansion</h3>
<pre><code class="language-text">원래 질문: &quot;전기차 충전 인프라 문제는?&quot;
확장 단어: 충전소, 전력망, 급속 충전, 주행 거리 불안 등</code></pre>
<p>관련 표현을 넓혀 Recall을 높일 수 있다. 단, 너무 많은 키워드를 추가하면 무관한 문서까지 늘어나므로 Expansion 결과도 평가해야 한다.</p>
<h3 id="64-query-transformation--decomposition">6.4 Query Transformation / Decomposition</h3>
<pre><code class="language-text">복합 질문: &quot;두 회사의 최근 3년 실적을 비교해 주세요.&quot;

하위 질문 1: 회사 A의 최근 3년 실적은?
하위 질문 2: 회사 B의 최근 3년 실적은?
하위 질문 3: 두 결과의 차이는?</code></pre>
<p>하위 질문마다 근거를 찾고 마지막에 통합하면 복합 질문에 강해진다. 대신 호출 수와 통합 오류 가능성이 늘어난다.</p>
<hr />
<h2 id="7-advanced-rag---retrieval">7. Advanced RAG - Retrieval</h2>
<h3 id="71-hybrid-search">7.1 Hybrid Search</h3>
<p>Hybrid Search는 키워드 검색과 의미 검색을 결합한다.</p>
<pre><code class="language-text">질문
 ├─ BM25: 정확한 코드·이름·조항 찾기
 └─ Vector Search: 동의어·의미가 비슷한 문서 찾기
                 ↓
             순위 결합</code></pre>
<p>오류 코드, 제품명, 법 조항은 키워드 검색이 강하고, 자연어 표현 변화는 의미 검색이 강하다. 두 검색 결과를 함께 쓰면 한 방식의 사각지대를 줄일 수 있다.</p>
<h3 id="72-multi-hop-search">7.2 Multi-hop Search</h3>
<p>Multi-hop Search는 하나의 문서만으로 답하기 어려운 질문에 필요한 정보를 단계적으로 연결한다.</p>
<pre><code class="language-text">질문
 → 첫 번째 근거 문서 검색
 → 그 결과를 단서로 두 번째 문서 검색
 → 여러 근거를 종합해 답변</code></pre>
<p>예를 들어 &quot;특정 정책이 바뀐 뒤 어떤 부서의 절차가 달라졌는가&quot;는 정책 변경 문서와 부서별 절차 문서를 모두 찾아야 할 수 있다.</p>
<hr />
<h2 id="8-advanced-rag---post-retrieval">8. Advanced RAG - Post-Retrieval</h2>
<p>검색된 문서를 바로 LLM에 넣지 않고, 답변에 가장 유용한 Context로 다듬는 단계다.</p>
<table>
<thead>
<tr>
<th>전략</th>
<th>핵심 질문</th>
<th>하는 일</th>
</tr>
</thead>
<tbody><tr>
<td>Reranker</td>
<td>무엇이 가장 중요한가?</td>
<td>질문과 문서를 함께 보고 후보 순위 재정렬</td>
</tr>
<tr>
<td>Context Reorder</td>
<td>어떤 순서로 보여 줄까?</td>
<td>LLM이 중요한 문서를 잘 활용하도록 배치</td>
</tr>
<tr>
<td>Compressor</td>
<td>무엇만 남길까?</td>
<td>중복·노이즈를 줄이고 핵심 문장 추출·요약</td>
</tr>
</tbody></table>
<h3 id="81-reranker">8.1 Reranker</h3>
<p>빠른 Retriever가 후보 20개를 가져오고, 느리지만 정확한 Reranker가 상위 5개를 고르는 <strong>Two-Stage Retrieval</strong>이 대표 구조다.</p>
<h3 id="82-context-reorder">8.2 Context Reorder</h3>
<p>LLM은 긴 문맥의 앞과 뒤 정보를 상대적으로 잘 활용하고 중간은 놓칠 수 있다. 따라서 중요한 근거가 앞 또는 뒤에 배치되도록 순서를 조정한다.</p>
<h3 id="83-compression">8.3 Compression</h3>
<p>압축은 전체 문서를 무조건 짧게 만드는 작업이 아니다. 질문에 직접 필요한 문장만 남겨 Context Window와 노이즈를 줄이는 작업이다. 원문 출처를 유지하지 않으면 압축 과정에서 근거 추적이 어려워진다.</p>
<h2 id="9-advanced-rag도-만능은-아니다">9. Advanced RAG도 만능은 아니다</h2>
<p>Advanced RAG는 검색 품질과 문맥 정렬을 개선하지만 다음 문제는 남는다.</p>
<ul>
<li>검색 단계가 많아질수록 지연 시간과 비용이 커진다.</li>
<li>문서 형식이 복잡하면 인덱싱·압축 품질이 기대보다 낮을 수 있다.</li>
<li>검색 근거가 있어도 LLM이 내용을 왜곡할 수 있다.</li>
<li>답변이 실제로 어떤 근거 문장을 사용했는지 완전히 추적하기 어렵다.</li>
</ul>
<p>그래서 다음 단계인 Self-RAG처럼 &quot;답을 만든 뒤 다시 확인하는 구조&quot;가 등장한다.</p>
<hr />
<h1 id="part-3-self-rag-스스로-점검하는-rag">Part 3. Self-RAG: 스스로 점검하는 RAG</h1>
<h2 id="10-self-rag란">10. Self-RAG란?</h2>
<p>Self-RAG는 RAG 흐름 중간에 자기 점검 질문을 넣어, 검색 필요성·문서 관련성·답변 근거성·최종 유용성을 평가하는 접근이다.</p>
<pre><code class="language-text">사용자 질문
  ↓
[검색이 필요한가?]
  ├─ 아니오 → 바로 답변
  └─ 예 → 문서 검색
             ↓
        [문서가 관련 있는가?]
             ↓
            답변 생성
             ↓
        [문서가 답을 뒷받침하는가?]
             ↓
        [답이 질문에 유용한가?]</code></pre>
<h2 id="11-reflection-token-네-가지">11. Reflection Token 네 가지</h2>
<p>교재의 핵심 판단을 초보자 언어로 바꾸면 다음과 같다.</p>
<table>
<thead>
<tr>
<th>판단</th>
<th>확인하는 질문</th>
<th>처리 예</th>
</tr>
</thead>
<tbody><tr>
<td>Retrieve?</td>
<td>지금 검색이 필요한가?</td>
<td>단순 인사는 검색 없이 답변</td>
</tr>
<tr>
<td>IsREL?</td>
<td>검색 문서가 질문과 관련 있는가?</td>
<td>무관 문서는 버리고 재검색</td>
</tr>
<tr>
<td>IsSUP?</td>
<td>답변의 주장이 문서에 뒷받침되는가?</td>
<td>근거 없는 부분 재생성 또는 제거</td>
</tr>
<tr>
<td>IsUSE?</td>
<td>최종 답변이 질문에 유용한가?</td>
<td>품질 점수로 최종 채택 여부 판단</td>
</tr>
</tbody></table>
<p>Self-RAG 연구에서는 문서·답변 후보를 구간별로 평가하고 여러 후보 중 더 나은 경로를 고르는 방식도 사용한다. 초보자 단계에서는 이를 &quot;한 번 생성한 답을 무조건 믿지 않고, 근거와 유용성을 확인해 재검색·재생성 여부를 결정한다&quot;고 이해하면 충분하다.</p>
<h3 id="예시">예시</h3>
<pre><code class="language-text">질문: &quot;현재 규정에서 환불 기한은?&quot;

1. Retrieve? → 예, 최신 규정 문서가 필요함
2. IsREL?    → 검색 문서가 환불 규정인지 확인
3. Generate  → 문서 기반으로 답변 생성
4. IsSUP?    → 날짜·조건이 문서에 실제 존재하는지 확인
5. IsUSE?    → 질문에 직접 답했고 읽기 쉬운지 확인</code></pre>
<h2 id="12-self-rag의-장점과-한계">12. Self-RAG의 장점과 한계</h2>
<table>
<thead>
<tr>
<th>개선점</th>
<th>남는 한계</th>
</tr>
</thead>
<tbody><tr>
<td>모든 질문에 무조건 검색하지 않을 수 있음</td>
<td>검색·평가·재생성 호출로 비용 증가</td>
</tr>
<tr>
<td>무관 문서를 폐기하고 재검색 가능</td>
<td>평가 모델도 틀리거나 편향될 수 있음</td>
</tr>
<tr>
<td>답변의 근거 충실성을 추가 확인</td>
<td>복잡한 다단계 추론과 외부 도구 사용은 제한적</td>
</tr>
<tr>
<td>재검색·재생성 루프 구성 가능</td>
<td>종료 조건을 잘못 설계하면 반복이 길어질 수 있음</td>
</tr>
</tbody></table>
<p>Self-RAG는 &quot;AI가 자기 검사를 하니 항상 맞다&quot;는 의미가 아니다. <strong>추가 검증 단계를 시스템에 넣어 오류를 발견할 기회를 늘리는 구조</strong>다.</p>
<hr />
<h1 id="part-4-modular-rag-레고처럼-조립하는-rag">Part 4. Modular RAG: 레고처럼 조립하는 RAG</h1>
<h2 id="13-modular-rag란">13. Modular RAG란?</h2>
<p>Modular RAG는 RAG를 하나의 고정 파이프라인이 아니라, 교체·추가·분기가 가능한 기능 블록으로 나눈다.</p>
<pre><code class="language-text">[질문 변환] → [검색] → [재정렬] → [압축] → [생성] → [검증]
        필요하면 바꾸거나, 빼거나, 병렬로 연결</code></pre>
<table>
<thead>
<tr>
<th>특징</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Independent</td>
<td>모듈을 비교적 독립적으로 설계</td>
</tr>
<tr>
<td>Flexible &amp; Scalable</td>
<td>새 검색기·평가기를 추가하거나 교체하기 쉬움</td>
</tr>
<tr>
<td>Dynamic Graph</td>
<td>질문 조건에 따라 흐름을 분기·반복 가능</td>
</tr>
</tbody></table>
<h2 id="14-모듈-지도">14. 모듈 지도</h2>
<table>
<thead>
<tr>
<th>큰 모듈</th>
<th>하위 모듈 예시</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Indexing</td>
<td>Chunk Optimization, Hierarchy, Knowledge Graph</td>
<td>지식을 검색 가능한 구조로 저장</td>
</tr>
<tr>
<td>Pre-Retrieval</td>
<td>Transformation, Expansion, Query Construction</td>
<td>질문을 검색에 적합하게 만듦</td>
</tr>
<tr>
<td>Retrieval</td>
<td>Retriever Selection, Source Selection, Fine-tuning</td>
<td>알맞은 근거를 찾음</td>
</tr>
<tr>
<td>Post-Retrieval</td>
<td>Rerank, Compression, Selection</td>
<td>LLM 입력을 정제</td>
</tr>
<tr>
<td>Generation</td>
<td>Generator Tuning, Verification</td>
<td>답변 생성과 안전 검증</td>
</tr>
<tr>
<td>Orchestration</td>
<td>Routing, Scheduling, Reasoning Path</td>
<td>모듈 실행 순서와 경로 제어</td>
</tr>
</tbody></table>
<p>모듈화의 목적은 모든 기능을 넣는 것이 아니다. <strong>실패 원인에 맞는 기능만 선택해 교체할 수 있게 만드는 것</strong>이다.</p>
<hr />
<h2 id="15-modular-rag의-네-가지-패턴">15. Modular RAG의 네 가지 패턴</h2>
<h3 id="151-linear-pattern-순서대로-실행">15.1 Linear Pattern: 순서대로 실행</h3>
<pre><code class="language-text">Query → Query Transformation → Retrieve → Rerank → Generate</code></pre>
<p>가장 이해하기 쉽고 디버깅이 편하다. 다만 쉬운 질문에도 모든 단계를 거치면 비용과 시간이 낭비될 수 있다.</p>
<h3 id="152-routing-pattern-질문에-따라-경로-선택">15.2 Routing Pattern: 질문에 따라 경로 선택</h3>
<pre><code class="language-text">질문 분석
  ├─ 사내 규정 질문 → Vector DB
  ├─ 정확한 코드 질문 → BM25 중심 검색
  ├─ 최신 외부 정보 → 웹 검색 도구
  └─ 단순 인사 → 검색 없이 답변</code></pre>
<p>Routing은 Retriever뿐 아니라 Prompt, 모델, 문서 필터, Post-Retrieval 전략까지 바꿀 수 있다. 유연하지만 라우터가 잘못 판단하면 전체 흐름이 틀어진다.</p>
<h3 id="153-branching-pattern-여러-경로를-병렬로-탐색">15.3 Branching Pattern: 여러 경로를 병렬로 탐색</h3>
<pre><code class="language-text">복합 질문
  ├─ 하위 질문 A → 검색 A → 부분 답변 A
  ├─ 하위 질문 B → 검색 B → 부분 답변 B
  └─ 하위 질문 C → 검색 C → 부분 답변 C
                          ↓
                       Fusion
                          ↓
                       최종 답변</code></pre>
<p>질문 누락을 줄이고 다양한 관점을 얻을 수 있지만, 처리량과 중복이 늘고 Fusion 품질이 중요해진다.</p>
<h3 id="154-loop-pattern-평가-결과에-따라-반복">15.4 Loop Pattern: 평가 결과에 따라 반복</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>흐름</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>Interactive Loop</td>
<td>검색 → 생성 → Judge → 정해진 횟수 반복</td>
<td>구현 단순, 불필요 반복 위험</td>
</tr>
<tr>
<td>Recursive Loop</td>
<td>Judge → Query Rewrite → 재검색·재생성</td>
<td>복합 질문에 강함, 종료 조건 필수</td>
</tr>
<tr>
<td>Adaptive Loop</td>
<td>먼저 루프 필요 여부를 판단</td>
<td>비용 제어 가능, Judge 설계가 핵심</td>
</tr>
</tbody></table>
<p>반복 구조에는 항상 다음과 같은 안전장치가 필요하다.</p>
<pre><code class="language-text">최대 반복 횟수
최대 비용 또는 최대 토큰
동일 질문 재시도 감지
근거가 없을 때 안전한 &quot;모름&quot; 응답</code></pre>
<h2 id="16-modular-rag의-현실적인-한계">16. Modular RAG의 현실적인 한계</h2>
<ul>
<li>패턴이 복잡할수록 지연 시간과 운영 난이도가 올라간다.</li>
<li>Judge나 Verification 기준은 설계자 주관과 LLM 품질에 영향을 받는다.</li>
<li>복수 경로의 답변을 합치는 Fusion이 부정확하면 오히려 답이 나빠질 수 있다.</li>
<li>근거 추적과 전체 시스템 평가 프레임워크는 여전히 어려운 과제다.</li>
</ul>
<hr />
<h1 id="part-5-agentic-rag-도구와-전략을-능동적으로-선택하기">Part 5. Agentic RAG: 도구와 전략을 능동적으로 선택하기</h1>
<h2 id="17-agentic-rag란">17. Agentic RAG란?</h2>
<p>Agentic RAG는 정해진 검색 순서만 실행하는 대신, 에이전트가 질문을 보고 다음 행동을 선택하는 구조다.</p>
<pre><code class="language-text">질문
  ↓
어떤 정보가 필요한가?
  ↓
어떤 도구·데이터 소스를 쓸까?
  ↓
검색 → 평가 → 필요 시 재검색·질문 수정
  ↓
근거를 통합해 답변</code></pre>
<p>핵심은 &quot;LLM이 스스로 모든 것을 안다&quot;가 아니라 <strong>정해진 권한 안에서 Retriever, DB, API, 웹 검색 같은 도구를 선택하고 흐름을 제어한다</strong>는 점이다.</p>
<h2 id="18-agentic-pattern">18. Agentic Pattern</h2>
<table>
<thead>
<tr>
<th>패턴</th>
<th>쉬운 뜻</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Reflection</td>
<td>자기 점검</td>
<td>앞 행동과 결과를 평가해 개선점 찾기</td>
</tr>
<tr>
<td>Planning</td>
<td>계획 수립</td>
<td>큰 질문을 작업 순서로 나누기</td>
</tr>
<tr>
<td>Tool Use</td>
<td>도구 사용</td>
<td>검색기, DB, API 등을 필요한 때 호출</td>
</tr>
<tr>
<td>Multi-Agent Collaboration</td>
<td>역할 분담</td>
<td>여러 전문 에이전트의 결과를 결합</td>
</tr>
</tbody></table>
<h3 id="181-reflection">18.1 Reflection</h3>
<p>Reflection은 결과를 보고 &quot;어떤 검색이 효과적이었고 무엇이 부족했는가&quot;를 기록·평가하는 과정이다. 보통 한 세션 안에서 임시로 유지되며, 장기적으로 쓰려면 별도의 메모리와 검증된 저장 정책이 필요하다.</p>
<h3 id="182-planning">18.2 Planning</h3>
<p>계획을 세우는 에이전트는 현장 소장에 비유할 수 있다.</p>
<pre><code class="language-text">목표: 배송 지연 원인과 대안을 알려 줘
  ↓
작업 분해: 주문 상태 확인 / 배송 현황 확인 / 대안 조회
  ↓
도구 배정: 주문 DB / 실시간 API / 정책 문서 검색
  ↓
결과 통합</code></pre>
<p>계획은 작업 순서를 정할 뿐, 실제 데이터를 검증하지 않는다. 중요한 사실은 반드시 신뢰할 수 있는 도구·문서 근거로 확인해야 한다.</p>
<hr />
<h2 id="19-agentic-rag-아키텍처">19. Agentic RAG 아키텍처</h2>
<h3 id="191-single-agent-rag">19.1 Single-Agent RAG</h3>
<p>하나의 Agent가 질문 분석, 도구 선택, 검색, 답변 생성을 모두 수행하는 가장 단순한 Agentic 구조다. 빠르게 시작하기 좋지만 역할이 많아질수록 Prompt가 복잡해지고, 왜 특정 도구를 골랐는지 추적하기 어려워질 수 있다.</p>
<h3 id="192-multi-agent-rag">19.2 Multi-Agent RAG</h3>
<pre><code class="language-text">Router Agent
  ├─ 내부 문서 검색 Agent
  ├─ 최신 웹 정보 검색 Agent
  ├─ 정형 데이터 조회 Agent
  └─ 결과 통합 Agent</code></pre>
<p>각 에이전트는 명확한 역할과 입력·출력 형식을 가져야 한다. 역할이 겹치면 비용은 늘고 결과 책임은 모호해진다.</p>
<h3 id="193-hierarchical-agentic-rag">19.3 Hierarchical Agentic RAG</h3>
<p>상위 Master Agent 또는 Supervisor가 작업을 나누고, 하위 Agent가 전문 작업을 수행한다.</p>
<table>
<thead>
<tr>
<th>장점</th>
<th>위험</th>
</tr>
</thead>
<tbody><tr>
<td>복잡한 작업을 전문 역할로 나눌 수 있음</td>
<td>상위 Agent의 오판이 전체에 영향</td>
</tr>
<tr>
<td>하위 Agent를 추가해 확장하기 쉬움</td>
<td>통신·통합 과정이 병목이 될 수 있음</td>
</tr>
<tr>
<td>관리와 관찰 구조가 명확함</td>
<td>설계 복잡도와 예측 불가능성 증가</td>
</tr>
</tbody></table>
<p>Supervisor는 명확한 흐름에서 유용하지만, 모든 문제에 계층형 구조가 필요한 것은 아니다. 단순 FAQ에는 과한 설계가 될 수 있다.</p>
<h3 id="194-corrective-rag">19.4 Corrective RAG</h3>
<p>검색 결과가 질문과 관련 있는지 평가하고, 문제가 있으면 질문을 고치거나 다른 정보원으로 전환한다.</p>
<pre><code class="language-text">초기 검색
  ↓
관련성 평가
  ├─ 관련 있음 → 답변 생성
  └─ 관련 없음/모호함
          ↓
      Query Rewrite 또는 외부 검색
          ↓
        다시 평가</code></pre>
<h3 id="195-adaptive-rag">19.5 Adaptive RAG</h3>
<p>Adaptive RAG는 질문 난이도와 상황에 따라 경로 자체를 바꾼다.</p>
<pre><code class="language-text">간단한 FAQ → 내부 문서 한 번 검색
비교 질문   → 하위 질문 분해 + 여러 문서 검색
최신 현황   → 내부 문서 + 실시간 API 또는 웹 정보</code></pre>
<h2 id="20-agentic-rag-workflow">20. Agentic RAG Workflow</h2>
<pre><code class="language-text">User Query : → 사용자가 해결하고 싶은 질문이나 요청을 입력하는 단계
  ↓
Router : → 질문의 성격을 판단해 어떤 검색 방법·데이터 소스·도구를 사용할지 결정하는 단계
  ↓
Retrieve : → Router가 선택한 문서 DB, 웹, API 등에서 질문의 답에 필요한 근거 정보를 찾아오는 단계
  ↓
문서 관련성 평가
  ├─ 낮음 → Query Transformation → 재검색
  └─ 높음 → Generate
                 ↓
             답변 근거성 평가
                 ├─ 부족 → 재생성 또는 재검색
                 └─ 충분 → 최종 답변</code></pre>
<h2 id="21-naive-rag와-agentic-rag-비교">21. Naive RAG와 Agentic RAG 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>Naive RAG</th>
<th>Agentic RAG</th>
</tr>
</thead>
<tbody><tr>
<td>검색 흐름</td>
<td>고정된 한 번의 검색</td>
<td>조건에 따라 반복·분기</td>
</tr>
<tr>
<td>데이터 소스</td>
<td>보통 하나의 Vector DB</td>
<td>문서·DB·API·웹 등 선택 가능</td>
</tr>
<tr>
<td>복합 질문</td>
<td>제한적</td>
<td>계획·분해·도구 조합 가능</td>
</tr>
<tr>
<td>최신성</td>
<td>저장된 문서에 의존</td>
<td>적절한 도구가 있으면 실시간 정보 활용 가능</td>
</tr>
<tr>
<td>비용·지연</td>
<td>비교적 예측 가능</td>
<td>호출·루프에 따라 증가 가능</td>
</tr>
<tr>
<td>적합 사례</td>
<td>FAQ, 단순 문서 Q&amp;A</td>
<td>복합 조사, 보고서, 다단계 업무</td>
</tr>
</tbody></table>
<h2 id="22-agentic-rag의-한계">22. Agentic RAG의 한계</h2>
<ul>
<li>반복 검색과 재생성은 비용·응답 시간을 크게 늘릴 수 있다.</li>
<li>종료 조건이 없으면 무한 루프나 불필요한 재시도가 생길 수 있다.</li>
<li>여러 Agent가 있다고 자동으로 협업 품질이 높아지는 것은 아니다.</li>
<li>자기 평가는 사실 검증과 다르며, 모델 편향을 그대로 가질 수 있다.</li>
<li>동적 경로는 왜 그런 답이 나왔는지 추적하기 어려울 수 있다.</li>
<li>장기 메모리는 개인정보·오래된 정보·권한 문제를 함께 관리해야 한다.</li>
</ul>
<p>따라서 Agentic RAG는 기본 RAG와 평가 체계가 안정된 뒤, 실제로 복잡한 판단·분기 요구가 있을 때 도입하는 편이 좋다.</p>
<h2 id="graph-rag-쉽게-이해하기">Graph RAG 쉽게 이해하기</h2>
<p>일반 RAG는 문서를 작은 조각(Flat Chunk)으로 나누고, 질문과 가장 비슷한 조각을 찾습니다.</p>
<pre><code class="language-text">질문: &quot;A 정책이 바뀌면 B 부서 업무는 어떻게 달라져?&quot;
        ↓
A 정책 청크, B 부서 청크를 각각 검색</code></pre>
<p>문제는 A와 B가 서로 다른 청크에 흩어져 있으면, <strong>“A가 B에 영향을 준다”는 관계 자체를 놓칠 수 있다</strong>는 점입니다.</p>
<p>Graph RAG는 문서를 단순 청크 모음으로만 저장하지 않고, 중요한 대상과 관계를 연결한 <strong>지식 지도</strong>로 저장합니다.</p>
<pre><code class="language-text">[정책 A] ── 적용 대상 ──&gt; [부서 B]
[부서 B] ── 담당 업무 ──&gt; [업무 C]</code></pre>
<p>즉, 핵심은 검색기를 바꾸는 것만이 아니라 <strong>지식을 저장하는 표현 방식 자체를 바꾸는 것</strong>입니다.</p>
<hr />
<h2 id="flat-chunk와-graph-rag-비교">Flat Chunk와 Graph RAG 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>일반 RAG</th>
<th>Graph RAG</th>
</tr>
</thead>
<tbody><tr>
<td>저장 단위</td>
<td>독립된 텍스트 청크</td>
<td>엔티티와 관계가 연결된 그래프</td>
</tr>
<tr>
<td>잘하는 질문</td>
<td>특정 사실 찾기</td>
<td>관계, 영향, 원인, 전체 구조 파악</td>
</tr>
<tr>
<td>예시 질문</td>
<td>“환불 기한은 며칠인가?”</td>
<td>“정책 변경이 어떤 부서와 업무에 영향을 주는가?”</td>
</tr>
<tr>
<td>검색 방식</td>
<td>의미가 가까운 청크 찾기</td>
<td>특정 엔티티 주변 또는 연결 관계 탐색</td>
</tr>
<tr>
<td>주의점</td>
<td>관계가 청크 사이에서 끊길 수 있음</td>
<td>그래프 생성 비용과 품질 관리가 필요함</td>
</tr>
</tbody></table>
<hr />
<h2 id="핵심-용어">핵심 용어</h2>
<pre><code class="language-text">Entity(엔티티)
= 문서에서 중요한 대상

예: 사람, 회사, 제품, 정책, 부서, 기술</code></pre>
<pre><code class="language-text">Relationship(관계)
= 엔티티끼리 어떤 관계인지 나타내는 연결선

예: &quot;A 회사가 B 제품을 개발했다&quot;
    [A 회사] ── 개발 ──&gt; [B 제품]</code></pre>
<pre><code class="language-text">Knowledge Graph(지식 그래프)
= 엔티티와 관계를 연결해 만든 지식 지도</code></pre>
<hr />
<h2 id="graph-rag-파이프라인">Graph RAG 파이프라인</h2>
<h3 id="1-문서를-chunk로-나눈다">1. 문서를 Chunk로 나눈다</h3>
<pre><code class="language-text">원본 문서
  ↓
Text Chunk</code></pre>
<p>이 단계까지는 일반 RAG와 같습니다.</p>
<h3 id="2-llm이-엔티티와-관계를-추출한다">2. LLM이 엔티티와 관계를 추출한다</h3>
<p>예를 들어 문장에 다음 내용이 있다고 가정해 봅시다.</p>
<pre><code class="language-text">A사는 2025년에 B 제품을 출시했고,
B 제품은 C 부서의 업무 효율을 높였다.</code></pre>
<p>LLM은 이를 다음처럼 구조화합니다.</p>
<pre><code class="language-text">[A사] ── 출시 ──&gt; [B 제품]
[B 제품] ── 업무 효율 향상 ──&gt; [C 부서]</code></pre>
<h3 id="3-지식-그래프를-만든다">3. 지식 그래프를 만든다</h3>
<p>문서 전체에서 추출한 엔티티와 관계를 연결합니다.</p>
<pre><code class="language-text">[A사] → [B 제품] → [C 부서]
                  ↓
              [업무 효율]</code></pre>
<p>이제 문서 조각이 아니라, “누가 무엇과 어떤 관계인지”를 탐색할 수 있습니다.</p>
<h3 id="4-연결이-많은-그룹을-찾는다">4. 연결이 많은 그룹을 찾는다</h3>
<p>그래프에는 서로 자주 연결되는 엔티티 집단이 생깁니다. 이를 <strong>커뮤니티(Community)</strong>라고 합니다.</p>
<pre><code class="language-text">커뮤니티 1: 제품·기술·개발팀
커뮤니티 2: 정책·법무·규정
커뮤니티 3: 고객·서비스·지원</code></pre>
<p>Leiden 알고리즘은 이런 연결이 촘촘한 그룹을 찾아주는 방법이라고 이해하면 됩니다. 알고리즘의 수학적 원리를 외울 필요는 없습니다.</p>
<h3 id="5-커뮤니티별로-요약을-미리-만든다">5. 커뮤니티별로 요약을 미리 만든다</h3>
<pre><code class="language-text">제품·기술 커뮤니티
→ “주요 제품, 담당 부서, 기술 관계” 요약

정책·규정 커뮤니티
→ “주요 정책, 적용 대상, 개정 이력” 요약</code></pre>
<p>이 요약은 나중에 전체 문서의 큰 흐름을 빠르게 파악하는 데 사용됩니다.</p>
<hr />
<h2 id="이미지-1은-무엇을-보여-주나">이미지 1은 무엇을 보여 주나?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/xxtimrecht/post/a4992456-e63d-4375-8ccd-a78d6afca883/image.png" /></p>
<p>이미지의 의미는 대략 다음과 같습니다.</p>
<pre><code class="language-text">점(Node)      = 엔티티
선(Edge)      = 엔티티 사이 관계
같은 색 점들  = 서로 연결이 많은 커뮤니티
큰 점         = 다른 엔티티와 많이 연결된 핵심 엔티티</code></pre>
<p>왼쪽은 큰 주제 단위의 커뮤니티를 보여 주고, 오른쪽은 그 커뮤니티 내부를 더 세분화한 모습입니다.</p>
<p>도시 지도로 비유하면 다음과 같습니다.</p>
<pre><code class="language-text">왼쪽: 서울, 부산, 대구처럼 큰 도시 구분
오른쪽: 서울 안의 강남, 홍대, 여의도처럼 세부 지역 구분</code></pre>
<hr />
<h2 id="local-search와-global-search">Local Search와 Global Search</h2>
<p>Graph RAG는 질문 종류에 따라 두 방식으로 검색할 수 있습니다.</p>
<table>
<thead>
<tr>
<th>검색 방식</th>
<th>하는 일</th>
<th>잘 맞는 질문</th>
</tr>
</thead>
<tbody><tr>
<td>Local Search</td>
<td>특정 엔티티 주변의 관계를 탐색</td>
<td>“A 제품과 관련된 부서는 어디인가?”</td>
</tr>
<tr>
<td>Global Search</td>
<td>여러 커뮤니티 요약을 종합</td>
<td>“전체 문서에서 가장 중요한 기술 흐름은 무엇인가?”</td>
</tr>
</tbody></table>
<h3 id="local-search">Local Search</h3>
<pre><code class="language-text">질문: &quot;B 제품은 어떤 부서와 관련 있나요?&quot;
        ↓
[B 제품] 노드 탐색
        ↓
연결된 [A사], [C 부서], [업무 효율] 확인
        ↓
구체적인 관계 중심 답변</code></pre>
<h3 id="global-search">Global Search</h3>
<pre><code class="language-text">질문: &quot;전체 조직의 주요 변화는 무엇인가요?&quot;
        ↓
커뮤니티별 요약 확인
        ↓
관련 커뮤니티 요약들을 종합
        ↓
큰 흐름 중심 답변</code></pre>
<hr />
<h2 id="이미지-2의-흐름">이미지 2의 흐름</h2>
<p><img alt="" src="https://velog.velcdn.com/images/xxtimrecht/post/9d3d0228-806a-4fc1-976a-b0930af38bea/image.png" /></p>
<p>왼쪽 보라색 영역은 <strong>미리 준비하는 Indexing 단계</strong>입니다.</p>
<pre><code class="language-text">원본 문서
  ↓
Text Chunks
  ↓
엔티티와 관계 추출
  ↓
Knowledge Graph 생성
  ↓
커뮤니티 탐지
  ↓
커뮤니티별 요약 생성</code></pre>
<p>오른쪽 초록색 영역은 <strong>질문이 들어온 뒤의 Query 단계</strong>입니다.</p>
<pre><code class="language-text">사용자 질문
  ↓
관련 커뮤니티 요약 선택
  ↓
커뮤니티별 부분 답변 생성
  ↓
부분 답변을 다시 종합
  ↓
전체 답변 생성</code></pre>
<hr />
<h2 id="한-문장-정리">한 문장 정리</h2>
<blockquote>
<p>Graph RAG는 문서를 독립적인 청크로만 저장하지 않고, 중요한 대상과 그 관계를 그래프로 연결해 특정 관계와 전체 구조를 더 잘 찾도록 만드는 RAG 방식이다.</p>
</blockquote>
<p>실무에서는 일반 Vector RAG를 완전히 버리기보다 다음처럼 함께 쓰는 경우가 많습니다.</p>
<pre><code class="language-text">단순 사실 질문
→ Vector Search 중심

관계·원인·영향·다단계 질문
→ Graph Search 또는 Graph + Vector Hybrid</code></pre>
<hr />
<h1 id="part-6-langgraph-기초-상태로-rag-흐름-만들기">Part 6. LangGraph 기초: 상태로 RAG 흐름 만들기</h1>
<h2 id="23-langgraph를-왜-배우나">23. LangGraph를 왜 배우나?</h2>
<p>기본 Chain은 대체로 위에서 아래로 한 번 흐른다.</p>
<pre><code class="language-text">질문 → 검색 → 답변</code></pre>
<p>하지만 재검색, 품질 평가, 질문 재작성, 사람 승인처럼 분기와 반복이 필요하면 그래프 구조가 편하다. LangGraph는 이런 흐름을 <strong>State, Node, Edge</strong>로 표현한다.</p>
<h2 id="24-네-가지-핵심-구성-요소">24. 네 가지 핵심 구성 요소</h2>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>쉬운 뜻</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>State</td>
<td>공동 작업 노트</td>
<td>현재 질문, 검색 결과, 답변, 평가를 저장·공유</td>
</tr>
<tr>
<td>Node</td>
<td>작업자</td>
<td>검색, 답변 생성, 평가처럼 하나의 작업 수행</td>
</tr>
<tr>
<td>Edge</td>
<td>연결선</td>
<td>다음 실행 순서 정의</td>
</tr>
<tr>
<td>Conditional Edge</td>
<td>갈림길</td>
<td>State를 보고 다음 Node 선택</td>
</tr>
</tbody></table>
<pre><code class="language-text">START
  ↓
retrieve Node
  ↓
generate Node
  ↓
judge Node
  ├─ 충분함 → END
  └─ 부족함 → retrieve 또는 rewrite Node</code></pre>
<h2 id="25-state-그래프의-기억-저장소">25. State: 그래프의 기억 저장소</h2>
<p>State는 노드 사이에서 공유하는 데이터 구조다.</p>
<pre><code class="language-python">from typing import Annotated, TypedDict
import operator
from langgraph.graph.message import add_messages

class GraphState(TypedDict):
    question: str
    context: str
    answer: str
    relevance: str
    steps: Annotated[list, operator.add]
    messages: Annotated[list, add_messages]</code></pre>
<p>State를 설계할 때는 &quot;나중에 필요할 정보만&quot; 넣어야 한다. 모든 Prompt, 모든 문서, 모든 로그를 State에 쌓으면 디버깅과 비용 관리가 어려워진다.</p>
<h3 id="251-overwrite-새-값으로-덮어쓰기">25.1 Overwrite: 새 값으로 덮어쓰기</h3>
<p>Reducer가 없는 일반 필드는 마지막 업데이트가 이전 값을 덮어쓴다.</p>
<pre><code class="language-text">answer = &quot;초기 답변&quot;
      ↓
answer = &quot;검증 후 수정 답변&quot;</code></pre>
<p>질문, 현재 Context, 최종 답변처럼 &quot;현재 상태 하나&quot;만 필요할 때 적합하다.</p>
<h3 id="252-reducer-값을-누적·병합하기">25.2 Reducer: 값을 누적·병합하기</h3>
<p>반복 과정의 단계 기록이나 대화 메시지는 기존 값을 보존해야 한다.</p>
<pre><code class="language-text">steps: [&quot;question&quot;]
  → [&quot;question&quot;, &quot;retrieve&quot;]
  → [&quot;question&quot;, &quot;retrieve&quot;, &quot;generate&quot;]</code></pre>
<table>
<thead>
<tr>
<th>Reducer</th>
<th>사용 예</th>
</tr>
</thead>
<tbody><tr>
<td><code>operator.add</code></td>
<td>일반 리스트 누적</td>
</tr>
<tr>
<td><code>add_messages</code></td>
<td>대화 메시지를 안전하게 병합</td>
</tr>
</tbody></table>
<p>주의: Reducer를 쓴 리스트 필드는 노드가 리스트 형태로 반환해야 한다. State 키 이름도 노드 반환값과 정확히 같아야 한다.</p>
<h2 id="26-node-하나의-책임만-가진-작업-함수">26. Node: 하나의 책임만 가진 작업 함수</h2>
<p>Node는 State를 입력받아 바뀐 부분만 dictionary로 반환한다.</p>
<pre><code class="language-python">def retrieve_document(state: GraphState) -&gt; dict:
    docs = retriever.invoke(state[&quot;question&quot;])
    return {&quot;context&quot;: format_docs(docs)}</code></pre>
<p>좋은 Node 설계 원칙:</p>
<ul>
<li>검색, 생성, 평가처럼 책임을 하나로 제한한다.</li>
<li>State 전체를 새로 만들지 말고 변경한 키만 반환한다.</li>
<li>반환 키가 State 스키마와 일치하는지 확인한다.</li>
<li>외부 API·DB 호출 실패에 대비한 오류 처리와 관찰 로그를 둔다.</li>
</ul>
<h2 id="27-edge와-conditional-edge">27. Edge와 Conditional Edge</h2>
<p>일반 Edge는 순서를 고정한다.</p>
<pre><code class="language-python">workflow.add_edge(&quot;retrieve&quot;, &quot;generate&quot;)</code></pre>
<p>Conditional Edge는 State를 판단해 다음 경로를 고른다.</p>
<pre><code class="language-python">workflow.add_conditional_edges(
    &quot;judge&quot;,
    choose_next,
    {
        &quot;grounded&quot;: &quot;END&quot;,
        &quot;retry&quot;: &quot;retrieve&quot;,
    },
)</code></pre>
<p>조건 분기에는 다음 안전장치가 필요하다.</p>
<ul>
<li>조건 함수의 반환값과 경로 이름을 정확히 맞춘다.</li>
<li>재시도 횟수와 최대 비용을 제한한다.</li>
<li>근거를 찾지 못했을 때 종료할 수 있게 한다.</li>
<li>실패·보류 상태를 별도로 기록한다.</li>
</ul>
<hr />
<h1 id="part-7-rag-평가-잘-찾고-근거-있게-답했는지-측정하기">Part 7. RAG 평가: 잘 찾고, 근거 있게 답했는지 측정하기</h1>
<h2 id="28-rag-평가는-왜-여러-층으로-나눌까">28. RAG 평가는 왜 여러 층으로 나눌까?</h2>
<p>RAG의 실패는 한 종류가 아니다.</p>
<pre><code class="language-text">정답 문서를 못 찾음       → Retrieval 문제
정답 문서는 찾았지만 왜곡함 → Generation 문제
답은 맞지만 질문과 무관함   → Relevance 문제</code></pre>
<p>따라서 검색과 생성, 자동 지표와 사람·LLM 평가를 함께 본다.</p>
<table>
<thead>
<tr>
<th>평가 계열</th>
<th>무엇을 보는가?</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>Heuristic</td>
<td>참조 답안과 단어·구문이 얼마나 겹치는가</td>
<td>ROUGE, BLEU, METEOR</td>
</tr>
<tr>
<td>의미 유사도</td>
<td>문장의 뜻이 얼마나 가까운가</td>
<td>SemScore</td>
</tr>
<tr>
<td>LLM-as-a-Judge</td>
<td>정확성·충실성·관련성 등을 종합 평가</td>
<td>구조화된 평가 Prompt</td>
</tr>
<tr>
<td>RAG 전용 평가</td>
<td>검색 문맥과 답변을 함께 평가</td>
<td>RAGAS</td>
</tr>
<tr>
<td>운영 관찰</td>
<td>Trace·데이터셋·사용자 피드백 관리</td>
<td>LangSmith</td>
</tr>
</tbody></table>
<hr />
<h2 id="29-heuristic-지표">29. Heuristic 지표</h2>
<h3 id="291-rouge-참조-답안의-핵심을-얼마나-놓치지-않았나">29.1 ROUGE: 참조 답안의 핵심을 얼마나 놓치지 않았나</h3>
<p>ROUGE는 생성 답안과 참조 답안의 n-gram 겹침을 본다.</p>
<table>
<thead>
<tr>
<th>종류</th>
<th>보는 것</th>
</tr>
</thead>
<tbody><tr>
<td>ROUGE-1</td>
<td>한 단어 단위 겹침</td>
</tr>
<tr>
<td>ROUGE-2</td>
<td>연속된 두 단어 겹침</td>
</tr>
<tr>
<td>ROUGE-L</td>
<td>순서를 유지한 가장 긴 공통 부분 수열</td>
</tr>
</tbody></table>
<p>요약에서 참조 답안의 핵심 단어가 빠지지 않았는지 보는 데 유용하다. 하지만 의미가 같은 다른 표현을 낮게 평가할 수 있다.</p>
<h3 id="292-bleu-생성-답안의-표현이-참조와-얼마나-정확히-맞나">29.2 BLEU: 생성 답안의 표현이 참조와 얼마나 정확히 맞나</h3>
<p>BLEU는 주로 기계 번역 평가에서 쓰이며, 생성 문장 안의 n-gram 중 참조와 일치하는 비율인 Precision을 중심으로 본다.</p>
<pre><code class="language-text">BLEU ≈ n-gram Precision의 기하평균 × Brevity Penalty</code></pre>
<p>너무 짧은 답만 만들어 높은 점수를 얻지 못하도록 길이 패널티를 둔다. 하지만 긴 구문이 하나라도 맞지 않으면 점수가 크게 떨어질 수 있고, 동의어나 자연스러운 의역을 잘 반영하지 못한다.</p>
<h3 id="293-meteor-단어의-형태와-일부-의미까지-고려하기">29.3 METEOR: 단어의 형태와 일부 의미까지 고려하기</h3>
<p>METEOR는 정확한 단어 일치뿐 아니라 어간, 동의어 같은 매칭을 고려하고 Precision·Recall·단어 순서 패널티를 함께 본다.</p>
<table>
<thead>
<tr>
<th>장점</th>
<th>한계</th>
</tr>
</thead>
<tbody><tr>
<td>단순 문자 일치보다 유연함</td>
<td>언어별 형태소·동의어 자원에 의존</td>
</tr>
<tr>
<td>Recall과 Precision을 함께 고려</td>
<td>복잡한 사실 오류를 완전히 판별하지 못함</td>
</tr>
<tr>
<td>단어 순서가 심하게 어긋나면 감점</td>
<td>도메인 특화 용어에는 별도 설계 필요</td>
</tr>
</tbody></table>
<h3 id="294-세-지표를-어떻게-구분할까">29.4 세 지표를 어떻게 구분할까?</h3>
<table>
<thead>
<tr>
<th>지표</th>
<th>중심 관점</th>
<th>잘 맞는 사용처</th>
</tr>
</thead>
<tbody><tr>
<td>ROUGE</td>
<td>참조의 핵심을 얼마나 포함했는가</td>
<td>요약, 키워드 커버리지</td>
</tr>
<tr>
<td>BLEU</td>
<td>생성 문구가 참조와 얼마나 정확히 일치하는가</td>
<td>번역, 정형 문장</td>
</tr>
<tr>
<td>METEOR</td>
<td>형태·동의어·순서까지 포함한 유연한 일치</td>
<td>단어 표현 변화가 있는 문장</td>
</tr>
</tbody></table>
<p>이 지표들은 빠르고 재현 가능하지만, &quot;사실이 맞는가&quot;나 &quot;검색 문서에 근거했는가&quot;를 보장하지는 않는다.</p>
<hr />
<h2 id="30-semscore-의미-유사도로-평가하기">30. SemScore: 의미 유사도로 평가하기</h2>
<p>SemScore는 참조 답안과 생성 답안을 각각 임베딩한 뒤 Cosine Similarity로 의미적 유사성을 측정한다.</p>
<pre><code class="language-text">참조 답안 → 벡터 A
생성 답안 → 벡터 B
           ↓
Cosine Similarity(A, B)</code></pre>
<p>단어가 다르더라도 의미가 비슷하면 어느 정도 높은 점수를 줄 수 있다는 장점이 있다. 하지만 서로 비슷한 주제를 말하면서도 날짜·숫자·대상 같은 중요한 사실이 틀린 경우에는 높은 점수를 받을 수 있으므로, 사실성 평가는 별도로 필요하다.</p>
<hr />
<h2 id="31-llm-as-a-judge">31. LLM-as-a-Judge</h2>
<p>LLM-as-a-Judge는 다른 LLM 또는 별도 평가 모델에게 평가자 역할을 맡기는 방식이다.</p>
<h3 id="311-평가-입력">31.1 평가 입력</h3>
<pre><code class="language-text">Question: 사용자 질문
Context: 검색된 문서
Answer: RAG가 생성한 답변
Criteria: 정확성, 충실성, 관련성 등</code></pre>
<h3 id="312-대표-평가-항목">31.2 대표 평가 항목</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>확인 내용</th>
</tr>
</thead>
<tbody><tr>
<td>Correctness</td>
<td>사실이 맞는가?</td>
</tr>
<tr>
<td>Faithfulness</td>
<td>답이 Context에 실제로 근거하는가?</td>
</tr>
<tr>
<td>Relevance</td>
<td>질문에 직접 답하는가?</td>
</tr>
<tr>
<td>Consistency</td>
<td>답변 내부의 논리가 맞는가?</td>
</tr>
<tr>
<td>Clarity</td>
<td>이해하기 쉽게 표현했는가?</td>
</tr>
<tr>
<td>Fluency</td>
<td>문장이 자연스러운가?</td>
</tr>
<tr>
<td>Conciseness</td>
<td>불필요하게 반복하지 않았는가?</td>
</tr>
</tbody></table>
<h3 id="313-평가-prompt는-표준화해야-한다">31.3 평가 Prompt는 표준화해야 한다</h3>
<p>모호하게 &quot;답이 좋은가요?&quot;라고 물으면 매번 다른 기준으로 평가할 수 있다. 점수 범위, 판단 절차, 출력 형식을 고정하는 것이 좋다.</p>
<pre><code class="language-text">역할: 당신은 엄격한 문서 기반 답변 평가자입니다.

평가 기준:
- Correctness: Context와 사실적으로 일치하는가? 1~5점
- Faithfulness: Context에 없는 주장을 하지 않는가? 1~5점
- Relevance: 질문에 직접 답하는가? 1~5점

출력:
- 각 점수
- 근거 설명
- 발견한 근거 없는 주장</code></pre>
<p>LLM 평가자도 편향·관대함·일관성 부족 문제가 있을 수 있다. 중요 서비스에서는 사람 검수 표본, 반복 평가, 다른 모델 간 교차 평가를 함께 사용한다.</p>
<p>G-Eval처럼 평가 절차와 정형 출력(Form-filling)을 강조하는 방식, 사실성과 답변-근거 정렬을 중점적으로 보는 방식 등은 모두 이 문제를 줄이기 위한 변형이다. 이름보다 중요한 것은 <strong>우리 업무의 평가 기준을 먼저 명확히 정하고 같은 기준으로 반복 측정하는 것</strong>이다.</p>
<hr />
<h2 id="32-ragas-검색과-생성을-함께-보는-평가">32. RAGAS: 검색과 생성을 함께 보는 평가</h2>
<p>RAGAS는 RAG 시스템에 맞춰 Retrieval과 Generation을 분리하면서도 함께 평가하는 프레임워크다.</p>
<pre><code class="language-text">Question + Contexts + Ground Truth + Answer
                    ↓
      Retrieval 점수 + Generation 점수</code></pre>
<h3 id="321-평가-데이터셋의-기본-구성">32.1 평가 데이터셋의 기본 구성</h3>
<table>
<thead>
<tr>
<th>필드</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td>Question</td>
<td>사용자가 물을 질문</td>
</tr>
<tr>
<td>Contexts</td>
<td>시스템이 검색한 문서들</td>
</tr>
<tr>
<td>Ground Truth</td>
<td>사람이 검증한 참조 답변 또는 정답 근거</td>
</tr>
<tr>
<td>Answer</td>
<td>RAG 시스템이 실제로 생성한 답변</td>
</tr>
<tr>
<td>Metadata</td>
<td>출처, 문서 버전, 난이도 등</td>
</tr>
</tbody></table>
<p>합성 테스트 데이터는 빠르게 시작하는 데 유용하다. 그러나 실제 사용자 질문과 다른 형태로 치우칠 수 있으므로, 운영 질문과 전문가 검수 데이터로 보완해야 한다.</p>
<h3 id="322-retrieval-지표">32.2 Retrieval 지표</h3>
<table>
<thead>
<tr>
<th>지표</th>
<th>쉬운 질문</th>
</tr>
</thead>
<tbody><tr>
<td>Context Precision</td>
<td>검색한 문서 중 정말 관련 있는 문서 비율은?</td>
</tr>
<tr>
<td>Context Recall</td>
<td>답에 필요한 관련 근거를 빠뜨리지 않았나?</td>
</tr>
</tbody></table>
<pre><code class="language-text">Precision이 낮음 → 노이즈 문서가 많이 섞임
Recall이 낮음    → 필요한 근거를 놓침</code></pre>
<h3 id="323-generation-지표">32.3 Generation 지표</h3>
<table>
<thead>
<tr>
<th>지표</th>
<th>쉬운 질문</th>
</tr>
</thead>
<tbody><tr>
<td>Faithfulness</td>
<td>답의 주장이 검색 문서에 뒷받침되는가?</td>
</tr>
<tr>
<td>Answer Relevancy</td>
<td>답이 질문에 직접 답하는가?</td>
</tr>
<tr>
<td>Context Relevancy</td>
<td>답과 검색 문맥이 실제로 연결되는가?</td>
</tr>
<tr>
<td>Conciseness</td>
<td>불필요한 반복 없이 핵심을 말하는가?</td>
</tr>
<tr>
<td>Fluency</td>
<td>문장이 자연스럽고 읽기 쉬운가?</td>
</tr>
</tbody></table>
<h3 id="324-점수-해석-예시">32.4 점수 해석 예시</h3>
<pre><code class="language-text">Context Precision 0.60
→ 검색한 5개 중 약 3개가 실제로 관련 있음

Context Recall 1.00
→ 답에 필요한 근거는 모두 찾음

Faithfulness 1.00
→ 답의 주장 모두가 Context에 근거함</code></pre>
<p>점수 하나만 보고 시스템을 판단하면 안 된다. Recall이 높아도 Precision이 낮으면 노이즈가 많고, 검색이 좋아도 Faithfulness가 낮으면 생성 단계에서 환각이 생긴 것이다.</p>
<hr />
<h2 id="33-langsmith-평가와-운영-관찰을-연결하기">33. LangSmith: 평가와 운영 관찰을 연결하기</h2>
<p>LangSmith는 RAG의 실행 Trace, 데이터셋, 자동 평가, 사용자 피드백을 함께 관리하는 도구다.</p>
<pre><code class="language-text">실행 Trace 확인
  → 어느 문서를 검색했는가?
  → 어떤 Prompt가 만들어졌는가?
  → 어떤 답을 생성했는가?

평가 결과 확인
  → 관련성·사실성·환각 점수는?
  → 실패 질문은 무엇인가?</code></pre>
<p>LangSmith 자체가 답변 품질을 자동으로 보장하지는 않는다. 대신 실패한 실행을 추적하고, 데이터셋과 평가 결과를 비교하며, 사용자 피드백을 다음 개선에 연결하기 쉽게 해 준다.</p>
<hr />
<h1 id="part-8-2일차-전체-요약">Part 8. 2일차 전체 요약</h1>
<h2 id="34-rag-성숙도-지도">34. RAG 성숙도 지도</h2>
<pre><code class="language-text">Naive RAG
질문 → 검색 → 답변

Advanced RAG
문서 구조화 → 질문 개선 → Hybrid/Multi-hop 검색 → 재정렬·압축 → 답변

Self-RAG
검색 필요성·문서 관련성·답변 근거성·유용성을 스스로 점검

Modular RAG
기능 블록을 교체·분기·반복 가능한 그래프로 조립

Agentic RAG
질문에 맞는 도구와 경로를 선택하고 평가 결과에 따라 재시도</code></pre>
<h2 id="35-문제별-개선-방향">35. 문제별 개선 방향</h2>
<table>
<thead>
<tr>
<th>증상</th>
<th>먼저 점검할 곳</th>
<th>대표 개선</th>
</tr>
</thead>
<tbody><tr>
<td>정답 문서를 못 찾음</td>
<td>Indexing, Query, Retriever</td>
<td>계층 청킹, Rewrite, Hybrid, Multi-hop</td>
</tr>
<tr>
<td>비슷한 문서만 반복됨</td>
<td>Retriever 결과</td>
<td>MMR, Reranker, 다양성 제어</td>
</tr>
<tr>
<td>문서는 찾았지만 답이 왜곡됨</td>
<td>Context, Prompt, Generation</td>
<td>Compression, 출처 구조화, Faithfulness 평가</td>
</tr>
<tr>
<td>질문이 복잡함</td>
<td>Pre-Retrieval, Orchestration</td>
<td>Decomposition, Branching, Planning</td>
</tr>
<tr>
<td>최신 정보가 필요함</td>
<td>Routing, Tool Use</td>
<td>API·웹·DB를 포함한 적절한 라우팅</td>
</tr>
<tr>
<td>재시도가 끝나지 않음</td>
<td>Graph Control</td>
<td>최대 반복·비용·종료 조건</td>
</tr>
<tr>
<td>점수는 높은데 실제 사용성이 낮음</td>
<td>평가셋과 사용자 피드백</td>
<td>실제 질문 추가, 사람 평가, 온라인 피드백</td>
</tr>
</tbody></table>
<h2 id="36-2일차-최종-체크리스트">36. 2일차 최종 체크리스트</h2>
<ul>
<li><input disabled="" type="checkbox" /> Naive RAG에서 유사도와 질문 의도가 다를 수 있음을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Advanced RAG의 Indexing, Pre-Retrieval, Retrieval, Post-Retrieval을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 계층형 구조와 Semantic Chunking의 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> Rewrite, Routing, Expansion, Decomposition의 목적을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Hybrid Search와 Multi-hop Search가 필요한 상황을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Reranker, Reorder, Compression의 역할을 안다.</li>
<li><input disabled="" type="checkbox" /> Self-RAG의 Retrieve?, IsREL?, IsSUP?, IsUSE? 판단을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Modular RAG의 Linear, Routing, Branching, Loop 패턴을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Agentic RAG가 모든 문제에 필요한 것은 아니라는 점을 안다.</li>
<li><input disabled="" type="checkbox" /> LangGraph의 State, Node, Edge, Conditional Edge를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Overwrite와 Reducer의 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> RAG 평가에서 Retrieval과 Generation을 따로 봐야 하는 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> ROUGE, BLEU, METEOR, SemScore, LLM-as-a-Judge의 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> RAGAS의 Context Precision, Context Recall, Faithfulness, Answer Relevancy를 설명할 수 있다.</li>
</ul>
<hr />
<h2 id="37-한-장-요약">37. 한 장 요약</h2>
<pre><code class="language-text">[기본 RAG의 한계]
Top-K 벡터 검색만으로는 질문 의도·문맥·노이즈·복합 질문을 충분히 다루기 어렵다.

[2일차의 해법]
질문 전에는 더 잘 찾게 만들고,
검색 중에는 여러 방식과 근거 연결을 활용하고,
검색 후에는 중요한 문맥만 골라 정렬하고,
답변 뒤에는 근거성과 유용성을 검증한다.

[설계 원칙]
가장 복잡한 RAG를 만드는 것이 목표가 아니다.
실제 실패 원인을 측정한 뒤,
필요한 모듈만 추가하고,
비용·속도·보안·근거 추적을 함께 관리한다.</code></pre>
<blockquote>
<p>RAG가 발전할수록 중요한 것은 &quot;AI에게 더 많은 자율성을 주는 것&quot;만이 아니다. <strong>어떤 근거로, 어떤 경로를 거쳐, 어느 시점에 멈추고 사람에게 넘길지를 설계하는 것</strong>이 핵심이다.</p>
</blockquote>