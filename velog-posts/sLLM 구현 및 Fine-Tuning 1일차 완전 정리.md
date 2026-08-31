<blockquote>
<p>이 글은 <strong>처음 sLLM과 Fine-Tuning을 배우는 사람</strong>을 위한 1일차 학습 노트다.<br />메인 교재의 1일차 범위(교재 3~48쪽)를 순서대로 빠짐없이 다루고, 입문 가이드·실습 가이드·LoRA 파라미터 구조 가이드의 설명을 보조로 덧붙였다.<br />강의 시간표는 제외하고 개념의 연결과 실습 흐름에 집중한다.</p>
</blockquote>
<hr />
<h2 id="0-오늘-무엇을-배우는가">0. 오늘 무엇을 배우는가?</h2>
<p>1일차의 큰 질문은 다음 한 문장으로 요약할 수 있다.</p>
<blockquote>
<p><strong>왜 기업 내부에서 sLLM을 사용하며, 작은 비용으로 어떻게 우리 업무에 맞게 바꿀 수 있을까?</strong></p>
</blockquote>
<p>이를 위해 다음 순서로 학습한다.</p>
<ol>
<li>LLM 서비스가 요청을 받아 답을 만드는 전체 파이프라인</li>
<li>MLM과 CLM의 차이, 생성형 모델이 CLM 중심인 이유</li>
<li>LLM과 sLLM의 차이와 기업이 sLLM을 선택하는 이유</li>
<li>HR·제조·상담 업무에서의 sLLM 활용 사례</li>
<li>Full Fine-Tuning과 PEFT, LoRA·Adapter·Prompt 계열 기법</li>
<li>실습 프로젝트와 데이터셋의 구조</li>
<li>Qwen2.5 기반 LoRA/QLoRA 학습과 Base 모델 대비 평가</li>
</ol>
<p>오늘의 핵심 산출물은 <strong>LLM/sLLM 구조 차이 이해</strong>, <strong>Use Case 도출</strong>, <strong>PEFT(특히 LoRA) 실습 및 코드 리뷰</strong>다.</p>
<hr />
<h2 id="1-먼저-잡는-전체-그림-천재-신입사원-비유fine-tunnung이-중요한-이유">1. 먼저 잡는 전체 그림: 천재 신입사원 비유(Fine-Tunnung이 중요한 이유)</h2>
<p>사전학습된 LLM을 인터넷의 수많은 글을 읽고 입사한 <strong>천재 신입사원</strong>이라고 생각해 보자.</p>
<ul>
<li>언어를 잘 이해하고 일반 상식도 많다.</li>
<li>그러나 우리 회사의 연차 규정, 결재 절차, 시스템 코드명은 모른다.</li>
<li>내용을 알더라도 회사가 원하는 말투와 답변 형식을 따르지 않을 수 있다.</li>
<li>매 질문을 외부 전문가에게 보내면 보안·비용·지연 문제가 생길 수 있다.</li>
</ul>
<p>처음부터 새 직원을 태어나게 해 교육하는 것이 <strong>사전학습</strong>이라면, 이미 유능한 직원을 짧은 사내 교육으로 업무에 맞추는 것이 <strong>Fine-Tuning</strong>이다. 이때 직원의 뇌 전체를 다시 만드는 대신 작은 업무용 보정 장치만 붙이는 방법이 <strong>PEFT</strong>이고, 대표적인 방법이 <strong>LoRA</strong>다.</p>
<pre><code class="language-text">사전학습된 Base Model
        ↓
사내 질문-답변 데이터로 SFT
        ↓
LoRA Adapter에 업무 행동 패턴 저장
        ↓
Base Model + LoRA Adapter
        ↓
도메인에 적응한 sLLM</code></pre>
<p>중요한 구분이 하나 있다.</p>
<ul>
<li><strong>Fine-Tuning</strong>: 모델의 행동, 답변 형식, 도메인 표현 방식 등을 가중치에 학습시킨다.</li>
<li><strong>RAG</strong>: 질문할 때 외부 문서를 찾아 컨텍스트로 제공한다.</li>
<li><strong>Quantization</strong>: 모델을 더 적은 메모리로 실행하도록 숫자의 표현 정밀도를 줄인다.</li>
</ul>
<p>셋은 경쟁 관계가 아니라 서로 다른 문제를 해결한다. 실제 시스템에서는 함께 사용할 수 있다.</p>
<hr />
<h1 id="part-1-llm-파이프라인-개요">Part 1. LLM 파이프라인 개요</h1>
<h2 id="1-1-llm-서비스는-어떻게-답을-만드는가">1-1. LLM 서비스는 어떻게 답을 만드는가?</h2>
<p>사용자에게는 채팅창 하나만 보이지만, 내부에서는 여러 단계가 차례로 실행된다.</p>
<pre><code class="language-text">요청 수신 → 전처리 → 모델 추론 → 후처리 → 응답 반환</code></pre>
<h3 id="①-요청-수신">① 요청 수신</h3>
<p>웹·앱·사내 시스템에서 질문을 받는다. API Gateway는 인증, 요청 제한, 라우팅 같은 입구 역할을 맡을 수 있다.</p>
<h3 id="②-전처리">② 전처리</h3>
<p>질문을 모델이 처리할 형태로 준비한다.</p>
<ul>
<li>시스템 지시문과 사용자 질문 결합</li>
<li>대화 템플릿 적용</li>
<li>긴 입력 자르기</li>
<li>토크나이저로 문장을 토큰 ID로 변환</li>
<li>필요하면 문서 검색 결과를 컨텍스트로 추가</li>
</ul>
<h3 id="③-모델-추론">③ 모델 추론</h3>
<p>모델이 앞의 토큰을 바탕으로 다음 토큰을 하나씩 예측한다.</p>
<pre><code class="language-text">입력: 연차는 며칠 전에 신청해야 하나요?
출력 생성: 연 → 차 → 는 → ...</code></pre>
<h3 id="④-후처리">④ 후처리</h3>
<p>생성 결과를 서비스 정책에 맞게 다듬는다.</p>
<ul>
<li>특수 토큰 제거</li>
<li>금지 표현·개인정보 검사</li>
<li>JSON 등 요구 형식 검증</li>
<li>출처나 관련 문서 링크 결합</li>
</ul>
<h3 id="⑤-응답-반환">⑤ 응답 반환</h3>
<p>완성된 답을 사용자에게 스트리밍하거나 한 번에 반환한다. 운영 환경에서는 지연시간, 오류, 토큰 사용량과 같은 로그도 함께 남긴다.</p>
<h2 id="1-2-외부-파이프라인과-내부-파이프라인">1-2. 외부 파이프라인과 내부 파이프라인</h2>
<h3 id="외부-파이프라인-상용-llm-사용">외부 파이프라인: 상용 LLM 사용</h3>
<pre><code class="language-text">Client → API Gateway → 상용 LLM(GPT·Claude 등) → 응답</code></pre>
<p>장점은 범용 성능이 강하고 도입이 빠르며 자체 모델 인프라가 거의 필요 없다는 점이다. 반면 요청 데이터가 벤더 인프라로 전달되고, 사내 고유 지식 반영에는 별도의 프롬프트·검색·튜닝 전략이 필요하다.</p>
<h3 id="내부-파이프라인-사내-sllm-사용">내부 파이프라인: 사내 sLLM 사용</h3>
<pre><code class="language-text">Client → 내부 Gateway → 사내 sLLM ↔ 내부 DB·문서 → 응답</code></pre>
<p>요청과 응답이 사내망 안에 머물고 내부 데이터 저장소와 직접 연결할 수 있다. 대신 GPU·서버·모델 버전·보안·모니터링을 직접 운영해야 한다.</p>
<table>
<thead>
<tr>
<th>비교 항목</th>
<th>상용 LLM 중심</th>
<th>완전 내부 sLLM 중심</th>
</tr>
</thead>
<tbody><tr>
<td>도입 속도</td>
<td>빠름</td>
<td>인프라 구축 필요</td>
</tr>
<tr>
<td>범용 성능</td>
<td>대체로 강함</td>
<td>모델 크기에 따라 제한</td>
</tr>
<tr>
<td>데이터 통제</td>
<td>계약·벤더 정책에 의존</td>
<td>조직이 인프라와 로그 통제</td>
</tr>
<tr>
<td>기술 감사</td>
<td>서비스 내부가 블랙박스일 수 있음</td>
<td>전체 실행 경로 감사 가능</td>
</tr>
<tr>
<td>도메인 적응</td>
<td>프롬프트·RAG·제공 기능에 의존</td>
<td>RAG·PEFT 등 직접 설계 가능</td>
</tr>
<tr>
<td>운영 부담</td>
<td>상대적으로 낮음</td>
<td>상대적으로 높음</td>
</tr>
</tbody></table>
<h2 id="1-3-상용-llm을-사내망에-배치해도-남는-문제">1-3. 상용 LLM을 사내망에 배치해도 남는 문제</h2>
<p>VPC나 전용 인스턴스를 사용한다고 해서 모든 문제가 자동으로 사라지는 것은 아니다.</p>
<ul>
<li>데이터 처리와 학습 활용 여부는 계약과 벤더 정책의 영향을 받는다.</li>
<li>실제 내부 처리 방식을 조직이 완전히 감사하기 어려울 수 있다.</li>
<li>금융·의료·국방·공공처럼 규제가 강한 곳은 이 불확실성 자체를 위험으로 볼 수 있다.</li>
<li>커스터마이징 범위가 제한되거나 특정 벤더에 종속될 수 있다.</li>
</ul>
<p>따라서 “외부 서비스가 나쁘다”가 아니라 <strong>데이터 민감도, 감사 요구, 비용, 성능, 운영 역량을 함께 비교해 선택해야 한다</strong>는 것이 핵심이다.</p>
<h2 id="1-4-내부-파이프라인의-두-목적">1-4. 내부 파이프라인의 두 목적</h2>
<h3 id="목적-1-데이터-주권-확보">목적 1: 데이터 주권 확보</h3>
<p>민감 데이터가 외부로 전송되지 않는 구조를 만든다. 계약 문구에만 의존하지 않고 아키텍처 자체로 통제 범위를 명확히 하는 것이다.</p>
<h3 id="목적-2-버티컬-도메인-최적화">목적 2: 버티컬 도메인 최적화</h3>
<p>법률, 의료, 제조, 사내 HR처럼 범위는 좁지만 깊은 지식이 필요한 업무에 모델을 맞춘다.</p>
<blockquote>
<p>범용 LLM은 넓은 지식을 제공하고, 버티컬 sLLM은 좁은 업무에서 깊이·통제·낮은 비용을 노린다.</p>
</blockquote>
<hr />
<h1 id="part-2-mlm과-clm-이해">Part 2. MLM과 CLM 이해</h1>
<h2 id="2-1-두-학습-방식의-직관">2-1. 두 학습 방식의 직관</h2>
<h3 id="mlm-빈칸-맞히기">MLM: 빈칸 맞히기</h3>
<p>Masked Language Model은 문장의 일부를 가리고 앞뒤 문맥을 모두 보며 빈칸을 예측한다.</p>
<pre><code class="language-text">나는 오늘 [MASK]에 간다.</code></pre>
<p>앞과 뒤를 동시에 볼 수 있어 문장의 의미 이해, 분류, 개체명 인식에 강하다. 대표 모델은 BERT와 RoBERTa다.</p>
<h3 id="clm-다음-단어-이어-쓰기">CLM: 다음 단어 이어 쓰기</h3>
<p>Causal Language Model은 현재까지 나온 토큰만 보고 다음 토큰을 예측한다.</p>
<pre><code class="language-text">나는 → 오늘 → 회사에 → 간다</code></pre>
<p>이 방식을 반복하면 긴 문장, 대화, 요약, 코드를 생성할 수 있다. 대표 모델은 GPT, Llama, Mistral, Qwen 계열이다.</p>
<h2 id="2-2-구조-비교">2-2. 구조 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>MLM</th>
<th>CLM</th>
</tr>
</thead>
<tbody><tr>
<td>학습 목표</td>
<td>가린 토큰 예측</td>
<td>다음 토큰 예측</td>
</tr>
<tr>
<td>문맥 방향</td>
<td>양방향</td>
<td>이전 토큰에서 다음 토큰으로 진행</td>
</tr>
<tr>
<td>대표 구조</td>
<td>Encoder 또는 Encoder-only</td>
<td>Decoder-only</td>
</tr>
<tr>
<td>대표 모델</td>
<td>BERT, RoBERTa</td>
<td>GPT, Llama, Mistral, Qwen</td>
</tr>
<tr>
<td>강점</td>
<td>문장 이해, 분류, 추출, 개체명 인식</td>
<td>자유 생성, 대화, 요약, 코드 생성</td>
</tr>
</tbody></table>
<p>통역팀에 비유하면 Encoder는 상대의 말을 끝까지 듣고 의미를 정리하는 <strong>청취 담당</strong>, Decoder는 앞에서부터 말을 한 단어씩 만들어 내는 <strong>발화 담당</strong>이다.</p>
<h2 id="2-3-생성형-fine-tuning이-clm-중심인-이유">2-3. 생성형 Fine-Tuning이 CLM 중심인 이유</h2>
<ol>
<li><strong>한 구조로 다양한 생성 과제를 처리한다.</strong> 요약, 상담, 코드 생성 모두 “입력 뒤에 적절한 출력을 이어 쓰기”로 표현할 수 있다.</li>
<li><strong>Instruction 형식과 잘 맞는다.</strong> <code>지시 → 응답</code> 데이터는 CLM의 다음 토큰 예측과 자연스럽게 연결된다.</li>
<li><strong>MLM은 자유 생성에 불리하다.</strong> MLM은 이해·분류·추출에는 강하지만 긴 답을 순차 생성하도록 설계된 방식은 아니다.</li>
<li><strong>현대 오픈소스 sLLM 대부분이 Decoder-only다.</strong> 실습 모델인 Qwen도 이 계열이다.</li>
</ol>
<h2 id="2-4-산업-흐름">2-4. 산업 흐름</h2>
<pre><code class="language-text">2018 BERT
  → MLM 기반 이해 과제 확산
2019~2020 GPT-2·GPT-3
  → 대규모 CLM의 생성 능력 부각
2022 InstructGPT·ChatGPT
  → Instruction Tuning과 인간 선호 정렬 확산
2023 이후 Llama·Qwen 등
  → 오픈소스·경량 CLM이 내부 파이프라인 후보로 부상</code></pre>
<h2 id="2-5-clm을-업무에-맞추는-대표-방식">2-5. CLM을 업무에 맞추는 대표 방식</h2>
<table>
<thead>
<tr>
<th>방식</th>
<th>무엇을 배우는가?</th>
<th>핵심 데이터/대상</th>
</tr>
</thead>
<tbody><tr>
<td>Instruction Tuning</td>
<td>지시를 따르고 원하는 형식으로 답하는 법</td>
<td>지시-응답 쌍</td>
</tr>
<tr>
<td>RLHF</td>
<td>인간이 선호하는 답변 방향</td>
<td>보상 모델과 정책 최적화</td>
</tr>
<tr>
<td>DPO</td>
<td>선택 답변이 거절 답변보다 낫다는 선호</td>
<td>선호·비선호 답변 쌍</td>
</tr>
<tr>
<td>PEFT</td>
<td>적은 추가 파라미터로 업무에 적응</td>
<td>LoRA 등 경량 어댑터</td>
</tr>
</tbody></table>
<p>사내 sLLM에서는 비용과 구현 난도를 고려해 <strong>Instruction Tuning + LoRA</strong> 조합을 자주 사용한다.</p>
<hr />
<h1 id="part-3-llm과-sllm-비교">Part 3. LLM과 sLLM 비교</h1>
<h2 id="3-1-둘의-공통-기반">3-1. 둘의 공통 기반</h2>
<p>LLM과 sLLM은 완전히 다른 종류의 기술이 아니다. 둘 다 보통 다음 요소를 갖는다.</p>
<ul>
<li><strong>Tokenizer</strong>: 문자열을 모델이 다룰 토큰으로 분해한다.</li>
<li><strong>Embedding</strong>: 토큰을 의미를 담은 숫자 벡터로 바꾼다.</li>
<li><strong>Transformer</strong>: Attention을 이용해 문맥 관계를 계산한다.</li>
<li><strong>전처리와 후처리</strong>: 입력 템플릿 적용과 출력 정리를 담당한다.</li>
</ul>
<p>sLLM은 일반적으로 더 작은 파라미터 규모, 경량화 구조, 양자화, On-Device 실행 가능성을 강조한다. 다만 “몇 개 이하가 sLLM”이라는 절대적 경계보다 <strong>목표 장비와 업무에서 충분히 작고 효율적인가</strong>가 더 중요하다.</p>
<h2 id="3-2-버티컬-sllm이-갖춰야-할-다섯-특성">3-2. 버티컬 sLLM이 갖춰야 할 다섯 특성</h2>
<ol>
<li><strong>도메인 지식 반영도</strong>: 전문 용어와 사내 프로세스를 정확히 이해해야 한다.</li>
<li><strong>추론 효율성</strong>: 저비용 서버나 On-Device에서도 허용 가능한 속도가 나와야 한다.</li>
<li><strong>데이터 프라이버시</strong>: 민감 정보가 외부로 나가지 않는 구조를 갖춰야 한다.</li>
<li><strong>최신성 유지 용이성</strong>: 규정이 바뀔 때 낮은 비용으로 갱신할 수 있어야 한다.</li>
<li><strong>안전한 출력 제어</strong>: 규정 밖의 내용은 추측하지 않는 등 업무 정책을 따라야 한다.</li>
</ol>
<h2 id="3-3-다차원-비교">3-3. 다차원 비교</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>대형 범용 LLM</th>
<th>sLLM</th>
</tr>
</thead>
<tbody><tr>
<td>파라미터 규모</td>
<td>수백억~수조 규모가 가능</td>
<td>수억~수십억 규모가 흔함</td>
</tr>
<tr>
<td>추론 비용·지연</td>
<td>높은 GPU 비용 또는 외부 API 왕복</td>
<td>로컬·On-Device 가능성이 큼</td>
</tr>
<tr>
<td>커스터마이징</td>
<td>전체 튜닝 비용이 큼</td>
<td>PEFT로 상대적으로 저렴</td>
</tr>
<tr>
<td>지식 범위</td>
<td>넓은 범용 지식</td>
<td>좁은 도메인에 집중 가능</td>
</tr>
<tr>
<td>프라이버시</td>
<td>외부 인프라 사용 시 제약</td>
<td>완전 내부 운용 가능</td>
</tr>
<tr>
<td>운영 난도</td>
<td>벤더가 상당 부분 담당 가능</td>
<td>조직이 직접 운영할 수 있음</td>
</tr>
</tbody></table>
<p>작다는 사실만으로 좋은 모델이 되는 것은 아니다. 작은 모델은 자원이 적은 대신 범용 능력이 약할 수 있으므로, <strong>업무 데이터·검색·평가·안전 정책</strong>으로 부족한 부분을 채워야 한다.</p>
<h2 id="3-4-fine-tuning은-sllm을-업무-모델로-만드는-다리">3-4. Fine-Tuning은 sLLM을 업무 모델로 만드는 다리</h2>
<pre><code class="language-text">기본 sLLM
(가볍지만 회사 업무는 모름)
        +
Fine-Tuning
(업무형 질문-답변과 행동 방식 학습)
        =
도메인 특화 sLLM</code></pre>
<p>Fine-Tuning으로 기대하는 변화는 다음과 같다.</p>
<ul>
<li>전문 용어와 업무 표현 반영</li>
<li>답변 형식과 말투 정렬</li>
<li>지시 수행 정확도 향상</li>
<li>규정에 없으면 추측하지 않는 안전 행동 학습</li>
</ul>
<p>단, 자주 바뀌는 사실을 모두 가중치에 외우게 하는 것은 관리가 어렵다. <strong>최신 문서의 사실 검색은 RAG</strong>, <strong>반복되는 응답 행동과 형식은 Fine-Tuning</strong>으로 나누는 것이 좋은 출발점이다.</p>
<hr />
<h1 id="part-4-sllm-use-case">Part 4. sLLM Use Case</h1>
<h2 id="4-1-상용-llm-사용이-어려운-상황">4-1. 상용 LLM 사용이 어려운 상황</h2>
<table>
<thead>
<tr>
<th>제약</th>
<th>설명</th>
<th>대표 상황</th>
</tr>
</thead>
<tbody><tr>
<td>망분리·보안 규제</td>
<td>외부 네트워크 연결 자체가 금지</td>
<td>금융, 국방, 공공</td>
</tr>
<tr>
<td>오프라인 처리</td>
<td>네트워크가 없어도 동작해야 함</td>
<td>공장, 선박, 플랜트</td>
</tr>
<tr>
<td>초저지연</td>
<td>외부 API 왕복 시간을 감당하기 어려움</td>
<td>실시간 상담·제어</td>
</tr>
<tr>
<td>반복 호출 비용</td>
<td>대량 API 호출 비용이 누적</td>
<td>고빈도 문서 처리</td>
</tr>
<tr>
<td>특수 용어·PII</td>
<td>외부 반출이 금지된 데이터</td>
<td>의료, 법률, 상담 로그</td>
</tr>
</tbody></table>
<h2 id="4-2-세-가지-대표-사례">4-2. 세 가지 대표 사례</h2>
<h3 id="사례-a-hr-내부-문서-요약·질의응답">사례 A: HR 내부 문서 요약·질의응답</h3>
<p>외부 전송이 금지된 인사 규정과 업무 문서를 사내 sLLM이 요약하고 질문에 답한다.</p>
<h3 id="사례-b-제조-설비-매뉴얼-qa">사례 B: 제조 설비 매뉴얼 Q&amp;A</h3>
<p>네트워크가 불안정하거나 단절된 공장 안에서 On-Device sLLM이 매뉴얼 기반 답변을 제공한다.</p>
<h3 id="사례-c-고객-상담-로그-분석">사례 C: 고객 상담 로그 분석</h3>
<p>PII가 포함된 상담 기록을 외부로 내보내지 않고 내부에서 분류·요약한다.</p>
<h2 id="4-3-사내-문서-기반-sllm-qa-챗봇">4-3. 사내 문서 기반 sLLM Q&amp;A 챗봇</h2>
<p>교재의 실행 시나리오는 RAG 흐름을 다음과 같이 설명한다.</p>
<pre><code class="language-text">사내 문서 저장소
  → 문서 수집·청크 분할
  → 임베딩 생성
  → 벡터 DB 저장
  → 질문과 유사한 청크 검색
  → 검색 문서를 sLLM 컨텍스트로 제공
  → 최종 답변 반환</code></pre>
<h3 id="초보자를-위한-도서관-비유">초보자를 위한 도서관 비유</h3>
<ul>
<li>문서 저장소: 도서관</li>
<li>청크: 책을 주제별로 나눈 페이지 묶음</li>
<li>임베딩: 각 묶음의 의미를 숫자 좌표로 바꾼 것</li>
<li>벡터 DB: 의미가 비슷한 자료를 빠르게 찾는 색인</li>
<li>RAG: 질문과 관련된 자료를 찾아 모델 책상 위에 올려주는 사서</li>
<li>sLLM: 자료를 읽고 자연어 답변을 쓰는 직원</li>
</ul>
<p>RAG가 문서를 찾아준다고 해서 답이 자동으로 정확해지는 것은 아니다. 청크 크기, 검색 품질, 프롬프트, 문서 버전, 모델의 근거 준수 여부를 함께 평가해야 한다.</p>
<hr />
<h1 id="part-5-fine-tuning-전략">Part 5. Fine-Tuning 전략</h1>
<h2 id="5-1-full-fine-tuning과-peft">5-1. Full Fine-Tuning과 PEFT</h2>
<h3 id="full-fine-tuning">Full Fine-Tuning</h3>
<p>모델의 전체 또는 매우 많은 원본 파라미터를 업데이트한다.</p>
<ul>
<li>장점: 변화의 자유도가 크다.</li>
<li>단점: GPU 메모리, 저장 공간, 학습 시간이 많이 든다.</li>
<li>위험: 데이터와 설정에 따라 기존 능력을 잃는 Catastrophic Forgetting 가능성이 있다.</li>
</ul>
<h3 id="peft">PEFT</h3>
<p>Parameter-Efficient Fine-Tuning은 원본 파라미터 대부분을 동결하고 아주 작은 부분만 학습한다.</p>
<ul>
<li>학습 메모리와 저장 비용이 크게 줄어든다.</li>
<li>업무별 Adapter만 교체할 수 있다.</li>
<li>원본 모델을 보존하기 쉽다.</li>
<li>적은 데이터와 자원으로 실험하기 좋다.</li>
</ul>
<h2 id="5-2-peft-기법-지도">5-2. PEFT 기법 지도</h2>
<table>
<thead>
<tr>
<th>기법</th>
<th>쉬운 비유</th>
<th>핵심 방식</th>
<th>적합한 상황</th>
</tr>
</thead>
<tbody><tr>
<td>LoRA</td>
<td>원래 도로 옆의 작은 우회로</td>
<td>Linear Layer 옆에 저랭크 A·B 행렬 추가</td>
<td>일반적인 도메인 SFT</td>
</tr>
<tr>
<td>Adapter</td>
<td>생산 라인 사이의 보정 공정</td>
<td>레이어 사이에 작은 모듈 삽입</td>
<td>태스크별 모듈 분리</td>
</tr>
<tr>
<td>Prefix-Tuning</td>
<td>모든 층의 작업반장에게 메모 제공</td>
<td>여러 레이어에 학습 가능한 prefix 제공</td>
<td>원본 가중치 동결, 경량 적응</td>
</tr>
<tr>
<td>Prompt-Tuning</td>
<td>입구에 붙이는 학습형 지시문</td>
<td>입력 앞에 virtual token embedding 추가</td>
<td>극소 자원 환경</td>
</tr>
</tbody></table>
<p>교재의 시나리오 매핑은 다음과 같다.</p>
<ul>
<li>HR 문서 요약처럼 도메인 지식·표현을 반영: <strong>LoRA / Adapter</strong></li>
<li>극소 자원 On-Device 적응: <strong>Prompt Tuning / Prefix Tuning</strong></li>
<li>복잡한 지시 이해와 응답 품질 개선: <strong>Instruction Tuning + LoRA</strong></li>
</ul>
<h2 id="5-3-lora의-핵심-원리">5-3. LoRA의 핵심 원리</h2>
<p>원래 Linear Layer의 가중치가 <code>W</code>라면 Full Fine-Tuning은 <code>W</code> 자체를 고친다. LoRA는 <code>W</code>를 동결하고 변화량 <code>ΔW</code>만 작은 두 행렬로 표현한다.</p>
<pre><code class="language-text">기존 출력: y = Wx

LoRA 적용 출력:
y = Wx + (α/r) · B(Ax)

ΔW ≈ BA</code></pre>
<ul>
<li><code>W</code>: 사전학습된 원본 가중치, 동결</li>
<li><code>A</code>, <code>B</code>: 새로 추가되어 학습되는 작은 행렬</li>
<li><code>r</code>: 변화량을 표현할 중간 차원의 크기</li>
<li><code>alpha</code>: LoRA 변화량의 스케일</li>
</ul>
<p>책 전체를 다시 쓰는 대신 필요한 페이지에 <strong>포스트잇</strong>을 붙이는 것과 같다. 중요한 점은 “기존 파라미터 15억 개 중 일부를 골라 학습”하는 것이 아니라, <strong>기존 파라미터는 동결하고 새 파라미터를 추가해 학습</strong>한다는 것이다.</p>
<h3 id="왜-작은-행렬로-충분할까">왜 작은 행렬로 충분할까?</h3>
<p>Fine-Tuning에 필요한 변화 <code>ΔW</code>는 원래 모델 전체보다 훨씬 단순한 몇 가지 방향으로 표현될 수 있다는 <strong>저랭크 가설</strong>을 이용한다.</p>
<p>예를 들어 원래 행렬이 <code>1536 × 1536</code>이라면:</p>
<pre><code class="language-text">원본 W: 1536 × 1536 = 2,359,296개

r = 8인 LoRA:
A: 8 × 1536
B: 1536 × 8
합계: 24,576개</code></pre>
<p>한 레이어에서 약 236만 개를 직접 조정하는 대신 약 2.5만 개로 변화량을 표현한다.</p>
<h2 id="5-4-lora-설정-읽기">5-4. LoRA 설정 읽기</h2>
<pre><code class="language-python">from peft import LoraConfig, TaskType, get_peft_model

lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=8,
    lora_alpha=16,
    lora_dropout=0.05,
    target_modules=[&quot;q_proj&quot;, &quot;v_proj&quot;],
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()</code></pre>
<table>
<thead>
<tr>
<th>설정</th>
<th>뜻</th>
<th>값을 키우거나 범위를 늘리면</th>
</tr>
</thead>
<tbody><tr>
<td><code>r</code></td>
<td>LoRA가 사용할 중간 차원, 즉 학습 용량</td>
<td>표현력·파라미터·메모리 증가</td>
</tr>
<tr>
<td><code>lora_alpha</code></td>
<td>LoRA 변화량의 크기를 조절하는 스케일</td>
<td>보정 영향이 강해질 수 있음</td>
</tr>
<tr>
<td><code>lora_dropout</code></td>
<td>학습 중 일부 LoRA 경로를 무작위로 끔</td>
<td>과적합 완화 가능</td>
</tr>
<tr>
<td><code>target_modules</code></td>
<td>LoRA를 붙일 Linear Layer 이름</td>
<td>대상이 많을수록 학습 범위 증가</td>
</tr>
<tr>
<td><code>task_type</code></td>
<td>모델 과제 유형</td>
<td>생성형 모델은 <code>CAUSAL_LM</code></td>
</tr>
<tr>
<td><code>bias=&quot;none&quot;</code></td>
<td>bias는 학습하지 않음</td>
<td>학습 파라미터 절약</td>
</tr>
</tbody></table>
<p><code>r</code>은 무조건 클수록 좋은 값이 아니다. 너무 작으면 표현력이 부족하고, 너무 크면 비용과 과적합 위험이 늘 수 있다. 데이터 규모와 평가 결과로 선택해야 한다.</p>
<h2 id="5-5-q-k-v와-target_modules">5-5. Q, K, V와 target_modules</h2>
<p>Self-Attention을 도서관 검색으로 생각해 보자.</p>
<ul>
<li><strong>Q(Query)</strong>: 지금 찾고 싶은 키워드</li>
<li><strong>K(Key)</strong>: 각 책에 붙은 색인 태그</li>
<li><strong>V(Value)</strong>: 선택된 책에서 실제로 가져올 내용</li>
<li><strong>O(Output)</strong>: 모은 정보를 다음 단계로 보내는 출력 투영</li>
</ul>
<p><code>q_proj</code>, <code>k_proj</code>, <code>v_proj</code>, <code>o_proj</code>는 모델 설계에 이미 존재한다. 사용자는 이 가운데 어디에 LoRA를 붙일지 <code>target_modules</code>로 고른다.</p>
<pre><code class="language-python">target_modules=[
    &quot;q_proj&quot;, &quot;k_proj&quot;, &quot;v_proj&quot;, &quot;o_proj&quot;,
    &quot;gate_proj&quot;, &quot;up_proj&quot;, &quot;down_proj&quot;,
]</code></pre>
<p>교육용 최소 예제는 Q와 V만 사용해 원리를 쉽게 보여준다. 실제 교재의 HR SFT 설정은 Attention 투영층뿐 아니라 MLP의 <code>gate/up/down</code> 투영층까지 넓혀 학습 용량을 확보한다.</p>
<p>또한 <code>target_modules=[&quot;q_proj&quot;]</code>는 q_proj 하나만 뜻하지 않는다. Transformer의 <strong>모든 반복 레이어에 있는 같은 이름의 모듈</strong>에 각각 독립적인 LoRA A·B가 붙는다.</p>
<h2 id="5-6-prompt-tuning">5-6. Prompt Tuning</h2>
<p>Prompt Tuning은 모델 내부의 많은 층에 LoRA를 붙이는 대신, 입력 앞에 학습 가능한 가상 토큰을 붙인다.</p>
<pre><code class="language-python">from peft import PromptTuningConfig, PromptTuningInit, get_peft_model

prompt_config = PromptTuningConfig(
    task_type=&quot;CAUSAL_LM&quot;,
    prompt_tuning_init=PromptTuningInit.TEXT,
    num_virtual_tokens=20,
    prompt_tuning_init_text=&quot;다음 사내 문서를 바탕으로 정확하게 답변하라:&quot;,
    tokenizer_name_or_path=&quot;internal-sllm-base&quot;,
)

model = get_peft_model(base_model, prompt_config)</code></pre>
<p>학습 대상이 매우 작아 저장·연산 자원이 극도로 제한된 환경에 유리하지만, 복잡한 도메인 변화 표현력은 LoRA보다 제한될 수 있다.</p>
<h2 id="5-7-sft-cpt-rlhfdpo-지식-증류-구분">5-7. SFT, CPT, RLHF/DPO, 지식 증류 구분</h2>
<h3 id="sft-모범답안-따라-쓰기">SFT: 모범답안 따라 쓰기</h3>
<p>질문과 정답 쌍을 이용해 “이렇게 답하라”는 행동과 형식을 가르친다.</p>
<h3 id="cpt-도메인-원문을-더-읽히기">CPT: 도메인 원문을 더 읽히기</h3>
<p>Continued Pre-Training은 라벨 없는 대량의 도메인 원문으로 사전학습을 이어간다. 방대한 전문 지식 자체를 모델 내부 표현에 익히게 하려는 목적이다.</p>
<h3 id="rlhfdpo-더-선호되는-답-고르기">RLHF/DPO: 더 선호되는 답 고르기</h3>
<ul>
<li>RLHF: 별도의 보상 모델을 만들고 그 신호로 정책을 최적화한다.</li>
<li>DPO: 선호 답과 비선호 답의 비교 쌍을 직접 학습한다.</li>
</ul>
<h3 id="지식-증류-큰-교사가-작은-학생의-학습-자료를-만든다">지식 증류: 큰 교사가 작은 학생의 학습 자료를 만든다</h3>
<p>더 강한 교사 모델이 합성 질문-답변을 만들고, 품질을 검수한 뒤 학생 sLLM이 SFT로 학습한다. 교사는 상용 모델일 수도 있고 사내 대형 모델일 수도 있다.</p>
<p>주의할 점은 세 가지다.</p>
<ol>
<li>교사도 모르는 사내 고유 지식은 만들어낼 수 없다.</li>
<li>합성 데이터는 사실성·다양성·형식을 반드시 검수해야 한다.</li>
<li>상용 API 출력을 모델 학습에 사용하는 것이 약관상 허용되는지 확인해야 한다.</li>
</ol>
<h2 id="5-8-교재의-성능-예시-읽는-법">5-8. 교재의 성능 예시 읽는 법</h2>
<p>교재에는 Fine-Tuning 전후 예시가 제시된다.</p>
<table>
<thead>
<tr>
<th>시나리오</th>
<th align="right">전</th>
<th align="right">후</th>
<th align="right">예시 개선폭</th>
</tr>
</thead>
<tbody><tr>
<td>HR 규정 요약</td>
<td align="right">58%</td>
<td align="right">84%</td>
<td align="right">+26%p</td>
</tr>
<tr>
<td>설비 매뉴얼 Q&amp;A</td>
<td align="right">47%</td>
<td align="right">71%</td>
<td align="right">+24%p</td>
</tr>
<tr>
<td>상담 로그 분류</td>
<td align="right">62%</td>
<td align="right">89%</td>
<td align="right">+27%p</td>
</tr>
</tbody></table>
<p>이 수치는 <strong>개념 설명을 위한 예시값</strong>이다. 실제 성능 주장이 아니며, 프로젝트에서는 독립된 Evaluation Dataset으로 직접 측정해야 한다.</p>
<hr />
<h1 id="part-6-프로젝트-구조-이해">Part 6. 프로젝트 구조 이해</h1>
<h2 id="6-1-전체-데모-프로젝트">6-1. 전체 데모 프로젝트</h2>
<pre><code class="language-text">sllm-main/
├── requirements.txt
├── readme.md
├── dataset/
│   ├── source/                 # 원천 문서
│   ├── training/               # 학습·검증 데이터
│   └── evaluation/             # 최종 평가 데이터
├── scripts/
│   ├── build_sft_dataset.py    # 형식 검증 및 데이터 분리
│   ├── build_training_dataset.py # 원천 문서에서 데이터 생성
│   ├── train_lora.py           # LoRA/QLoRA 학습
│   └── evaluate_model.py       # Base와 튜닝 모델 비교
├── models/hr-qwen-lora/        # 학습된 Adapter
├── outputs/                    # 로그와 평가 결과
├── vector_db/
├── run.ipynb
├── sllm_service/
├── frontend/
└── app.py</code></pre>
<p>전체 데모에는 데이터 생성, RAG, 서비스 코드까지 있지만 1일차 학생 실습은 <strong>준비된 학습·검증·평가 데이터로 SFT를 수행하고 전후 성능을 비교하는 것</strong>에 집중한다.</p>
<h2 id="6-2-학생-배포본">6-2. 학생 배포본</h2>
<pre><code class="language-text">sllm-student/
├── requirements.txt
├── dataset/
│   ├── training/
│   │   ├── hr_sft_train.jsonl
│   │   └── hr_sft_validation.jsonl
│   └── evaluation/
│       └── hr_eval.jsonl
├── scripts/
│   ├── train_lora.py
│   └── evaluate_model.py
├── models/hr-qwen-lora/        # 처음에는 빈 폴더
└── outputs/                    # 처음에는 빈 폴더</code></pre>
<p>학생 배포본에는 원천 데이터 수집·전처리, RAG, 프런트엔드, 서비스 배포 코드가 포함되지 않는다.</p>
<h2 id="6-3-코드를-읽는-순서">6-3. 코드를 읽는 순서</h2>
<pre><code class="language-text">① 데이터 준비
   build_training_dataset.py / build_sft_dataset.py
        ↓
② 학습
   train_lora.py
        ↓
③ 평가
   evaluate_model.py
        ↓
④ 서비스
   sllm_service (데모·후속 범위)</code></pre>
<p>실습 배포본에 데이터 생성 코드가 없더라도 전체 파이프라인에서 데이터가 어디서 왔는지는 이해해야 한다.</p>
<h2 id="6-4-세-데이터셋은-왜-나누는가">6-4. 세 데이터셋은 왜 나누는가?</h2>
<h3 id="train">Train</h3>
<p>LoRA Adapter가 실제로 답을 배우는 데이터다.</p>
<h3 id="validation">Validation</h3>
<p>학습 중 보지 않은 데이터로 <code>eval_loss</code>를 확인하고, 과적합 여부와 최적 체크포인트를 판단한다. Validation을 반복해서 보고 설정을 바꾸면 간접적으로 설정 선택에 사용되는 셈이다.</p>
<h3 id="evaluationtest">Evaluation(Test)</h3>
<p>학습이 끝난 뒤 최종 Base 모델과 Fine-tuned 모델을 공정하게 비교한다. <strong>모델 학습에는 사용하지 않는다.</strong></p>
<p>시험 공부에 비유하면 Train은 교과서 문제, Validation은 중간 모의고사, Evaluation은 마지막까지 봉인한 기말고사다.</p>
<h2 id="6-5-instruction-sft와-context-based-qa-sft">6-5. Instruction SFT와 Context-based QA SFT</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>Instruction SFT</th>
<th>Context-based QA SFT</th>
</tr>
</thead>
<tbody><tr>
<td>입력</td>
<td>사람이 만든 <code>messages</code> Q&amp;A</td>
<td>HR 매뉴얼 원천 문서</td>
</tr>
<tr>
<td>생성 방식</td>
<td>형식 검증 후 Train/Validation/Evaluation 분리</td>
<td>문서를 Section으로 나누고 질문·답변 자동 생성</td>
</tr>
<tr>
<td>정답</td>
<td>사람이 미리 작성한 assistant 답변</td>
<td>결론 + 적용 규정 원문 + 안전 문구</td>
</tr>
<tr>
<td>목적</td>
<td>지시-응답 스타일 학습</td>
<td>문서 근거 응답과 환각 억제 행동 학습</td>
</tr>
<tr>
<td>한계</td>
<td>원본 Q&amp;A 품질에 크게 좌우</td>
<td>문서 구조와 품질이 질문 다양성을 제한</td>
</tr>
<tr>
<td>관계</td>
<td>동일한 학습 스크립트 사용</td>
<td>하나의 HR LoRA Adapter로 통합 가능</td>
</tr>
</tbody></table>
<hr />
<h1 id="part-7-peft-기초-실습">Part 7. PEFT 기초 실습</h1>
<h2 id="7-1-실습-목표와-시나리오">7-1. 실습 목표와 시나리오</h2>
<p>초기 시나리오는 약 200개의 사내 인사·총무 FAQ 질문-답변으로 정확도를 개선하는 것이다. 구체화된 HR 실습은 <strong>Human AI Corporation의 인사 규정 안내 챗봇</strong>을 만든다.</p>
<ul>
<li>Base Model: Qwen2.5 계열 Instruct 모델</li>
<li>데이터: 연차, 재택근무, 보안교육 등 HR 규정</li>
<li>학습: LoRA 또는 CUDA 환경의 QLoRA</li>
<li>기대: 규정 키워드 반영률 상승, 존재하지 않는 규정을 만드는 환각 감소</li>
</ul>
<p>실습의 목적은 단지 학습 명령을 실행하는 것이 아니다.</p>
<ol>
<li>LoRA 적용 전후 Trainable Parameter가 어떻게 변하는지 확인한다.</li>
<li><code>rank</code>, <code>alpha</code>, <code>target_modules</code>의 의미를 코드에서 찾는다.</li>
<li><code>loss</code>, <code>eval_loss</code>가 어떻게 변하는지 본다.</li>
<li>동일 질문으로 Base와 Fine-tuned 모델을 비교한다.</li>
</ol>
<h2 id="7-2-모델-개선의-세-축">7-2. 모델 개선의 세 축</h2>
<p>교재는 관련 기술을 세 축으로 나눈다.</p>
<table>
<thead>
<tr>
<th>축</th>
<th>대표 기술</th>
<th align="right">모델 학습 여부</th>
<th>좋아지는 것</th>
</tr>
</thead>
<tbody><tr>
<td>모델 성능 향상</td>
<td>LoRA/QLoRA Fine-Tuning</td>
<td align="right">O</td>
<td>HR 규정 도메인 적응</td>
</tr>
<tr>
<td>Context &amp; Reasoning</td>
<td>FAISS/BGE-M3 기반 RAG, 맥락 압축, Self-refine</td>
<td align="right">보통 X</td>
<td>외부 문서 활용과 답변 근거</td>
</tr>
<tr>
<td>추론 최적화</td>
<td>4bit Quantization</td>
<td align="right">X</td>
<td>메모리와 실행 비용</td>
</tr>
</tbody></table>
<p>이 분류를 기억하면 “RAG도 Fine-Tuning인가?”, “4bit로 바꾸면 지식을 학습하는가?” 같은 혼동이 줄어든다.</p>
<h2 id="7-3-instruction-sft-데이터-검증과-분리">7-3. Instruction SFT 데이터 검증과 분리</h2>
<p>대화 데이터는 대략 다음 형태다.</p>
<pre><code class="language-json">{
  &quot;messages&quot;: [
    {&quot;role&quot;: &quot;user&quot;, &quot;content&quot;: &quot;연차 신청 기한을 알려주세요.&quot;},
    {&quot;role&quot;: &quot;assistant&quot;, &quot;content&quot;: &quot;사내 규정에 따르면 ...&quot;}
  ]
}</code></pre>
<p><code>build_sft_dataset.py</code>의 핵심 흐름은 다음과 같다.</p>
<ol>
<li>각 레코드에 <code>messages</code> 배열이 있는지 검사한다.</li>
<li><code>user</code>와 <code>assistant</code> 역할이 모두 있는지 검사한다.</li>
<li>난수 시드를 고정하고 레코드를 섞는다.</li>
<li>약 10%를 Evaluation, 약 10%를 Validation, 나머지를 Train으로 저장한다.</li>
</ol>
<pre><code class="language-python">random.seed(RANDOM_SEED)
random.shuffle(records)

eval_n = max(1, round(total * 0.1))
val_n = max(1, round(total * 0.1))

evaluation = records[:eval_n]
validation = records[eval_n:eval_n + val_n]
train = records[eval_n + val_n:]</code></pre>
<p>같거나 거의 같은 질문이 세 세트에 중복되면 데이터 누출이 생길 수 있으므로, 실무에서는 의미 중복도 검사해야 한다.</p>
<h2 id="7-4-context-based-qa-데이터-생성">7-4. Context-based QA 데이터 생성</h2>
<h3 id="단계-1-문서를-section으로-분리">단계 1: 문서를 Section으로 분리</h3>
<p><code>Chapter 3</code>, <code>제3장</code>, <code>3.1 연차</code> 같은 제목 패턴을 찾아 HR 매뉴얼을 의미 단위로 나눈다.</p>
<pre><code class="language-python">@dataclass(frozen=True)
class ManualSection:
    section_id: str
    title: str
    content: str</code></pre>
<p>각 Section은 ID, 제목, 원문 내용을 갖는다. 너무 짧거나 유효하지 않은 Section은 제거한다.</p>
<h3 id="단계-2-여러-질문-표현-만들기">단계 2: 여러 질문 표현 만들기</h3>
<p>같은 규정을 한 문장만으로 묻지 않고 표현을 바꾼다.</p>
<pre><code class="language-text">연차 규정을 설명해 주세요.
직원이 연차와 관련해 반드시 알아야 할 사항은?
연차를 실제 업무에 적용할 때 조건과 예외는?
연차 규정의 핵심 기준과 예외를 정리해 주세요.</code></pre>
<p>이렇게 하면 모델이 특정 문장 하나를 암기하기보다 다양한 표현에 대응할 가능성이 높아진다.</p>
<h3 id="단계-3-근거가-포함된-구조화-답변-만들기">단계 3: 근거가 포함된 구조화 답변 만들기</h3>
<pre><code class="language-text">결론: 핵심 규정을 먼저 한 문장으로 답한다.

적용 규정: 해당 Section 원문을 제시한다.

위 규정만으로 판단하기 어려운 상황은 추측하지 않고 담당 부서에 확인한다.</code></pre>
<h3 id="단계-4-평가-키워드와-안전-답변-만들기">단계 4: 평가 키워드와 안전 답변 만들기</h3>
<p><code>must_include</code>에는 다음 우선순위로 핵심어를 뽑는다.</p>
<ol>
<li>AURORA, ORBIT 같은 회사 코드</li>
<li><code>3영업일</code> 같은 숫자·기한</li>
<li>사내 정책 용어</li>
</ol>
<p>규정에 없는 질문도 데이터에 넣고 다음과 같은 안전 행동을 학습한다.</p>
<blockquote>
<p>현재 제공된 규정에서는 해당 내용을 확인할 수 없습니다. 임의로 추측하거나 단정하지 말고 담당 부서에 확인해야 합니다.</p>
</blockquote>
<p>이는 환각을 완전히 제거하는 장치는 아니지만, “근거가 없으면 모른다고 말하기”를 명시적으로 학습시키는 중요한 데이터 설계다.</p>
<h2 id="7-5-실행-환경-자동-분기">7-5. 실행 환경 자동 분기</h2>
<p><code>train_lora.py</code>는 장치를 자동 감지한다.</p>
<pre><code class="language-python">def detect_device_type():
    if torch.cuda.is_available():
        return &quot;cuda&quot;
    if torch.backends.mps.is_available():
        return &quot;mps&quot;
    return &quot;cpu&quot;</code></pre>
<table>
<thead>
<tr>
<th>환경</th>
<th>학습 모드</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>NVIDIA CUDA</td>
<td>기본적으로 4bit NF4 QLoRA 또는 FP16 LoRA</td>
<td>메모리 효율과 속도</td>
</tr>
<tr>
<td>Apple Silicon MPS</td>
<td>FP32 LoRA</td>
<td>속도보다 안정성 우선</td>
</tr>
<tr>
<td>CPU</td>
<td>FP32 LoRA, 짧은 최대 길이</td>
<td>가장 느리므로 설정 축소</td>
</tr>
</tbody></table>
<h2 id="7-6-qlora는-무엇인가">7-6. QLoRA는 무엇인가?</h2>
<p>일반 LoRA는 원본 모델을 보통 16bit 등의 정밀도로 메모리에 올리고 A·B만 학습한다. QLoRA는 <strong>동결된 Base Model을 4bit로 양자화해 보관</strong>하고 LoRA Adapter는 학습 가능한 정밀도로 유지한다.</p>
<pre><code class="language-python">bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type=&quot;nf4&quot;,
    bnb_4bit_compute_dtype=&quot;bfloat16&quot;,
)</code></pre>
<ul>
<li>장점: GPU 메모리가 크게 줄어 더 큰 모델을 학습할 수 있다.</li>
<li>대가: 양자화·복원 연산 부담과 작은 정보 손실 가능성이 있다.</li>
<li>핵심: QLoRA도 학습되는 주체는 LoRA Adapter다.</li>
</ul>
<h2 id="7-7-실제-lora-config">7-7. 실제 LoRA Config</h2>
<p>교재의 HR 실습 설정은 기본적으로 다음과 같은 방향이다.</p>
<pre><code class="language-python">def create_lora_config(profile):
    return LoraConfig(
        r=profile.lora_rank,                 # 기본 16
        lora_alpha=profile.lora_alpha,       # 기본 32
        lora_dropout=profile.lora_dropout,
        bias=&quot;none&quot;,
        task_type=&quot;CAUSAL_LM&quot;,
        target_modules=[
            &quot;q_proj&quot;, &quot;k_proj&quot;, &quot;v_proj&quot;, &quot;o_proj&quot;,
            &quot;gate_proj&quot;, &quot;up_proj&quot;, &quot;down_proj&quot;,
        ],
    )</code></pre>
<p><code>r=16</code>, <code>alpha=32</code>라는 값 자체를 정답처럼 외우지 말자. 이는 출발점이며 데이터 규모, 모델 구조, 메모리, Validation 결과에 맞춰 조정한다.</p>
<h2 id="7-8-sfttrainer-설정">7-8. SFTTrainer 설정</h2>
<pre><code class="language-python">def create_sft_config(profile, output_dir):
    return SFTConfig(
        output_dir=str(output_dir),
        num_train_epochs=profile.epochs,
        learning_rate=profile.learning_rate,
        eval_strategy=&quot;epoch&quot;,
        save_strategy=&quot;epoch&quot;,
        load_best_model_at_end=True,
        metric_for_best_model=&quot;eval_loss&quot;,
        assistant_only_loss=True,
        packing=False,
    )</code></pre>
<h3 id="중요한-옵션">중요한 옵션</h3>
<ul>
<li><code>num_train_epochs</code>: 전체 Train 데이터를 몇 번 반복할지 결정한다.</li>
<li><code>learning_rate</code>: 한 번의 업데이트에서 LoRA 값을 얼마나 바꿀지 결정한다.</li>
<li><code>eval_strategy=&quot;epoch&quot;</code>: 매 Epoch마다 Validation 평가를 한다.</li>
<li><code>load_best_model_at_end=True</code>: 마지막 모델이 아니라 Validation 기준 최적 모델을 선택한다.</li>
<li><code>metric_for_best_model=&quot;eval_loss&quot;</code>: 낮은 <code>eval_loss</code>를 기준으로 고른다.</li>
<li><code>assistant_only_loss=True</code>: 대화 전체가 아니라 assistant 답변 토큰을 중심으로 Loss를 계산한다.</li>
<li><code>packing=False</code>: 여러 짧은 샘플을 한 시퀀스에 압축하지 않는다.</li>
</ul>
<h2 id="7-9-학습-내부에서는-무슨-일이-일어나는가">7-9. 학습 내부에서는 무슨 일이 일어나는가?</h2>
<pre><code class="language-text">① Forward
입력 데이터가 모델을 지나 예측 생성
        ↓
② Loss 계산
예측과 정답의 차이를 숫자로 계산
        ↓
③ Backward
오차의 원인을 뒤에서 앞으로 추적해 기울기 계산
        ↓
④ Optimizer Step
동결된 Base는 그대로 두고 LoRA A·B만 업데이트</code></pre>
<pre><code class="language-python">outputs = model(**batch)
loss = outputs.loss
loss.backward()
optimizer.step()
optimizer.zero_grad()</code></pre>
<p>이 흐름은 신경망 학습의 일반 원리다. Transformer에서는 RNN의 시간축 역전파인 BPTT가 아니라 일반적인 계산 그래프 역전파를 사용한다.</p>
<ul>
<li>완성된 제품(모델 출력)을 검사해서 오차를 확인한 뒤, 그 오차가 왜 생겼는지 거꾸로 추적하며 보정장치의 손잡이를 얼마나 돌려야 할지 계산하는 과정, 이것이 <strong>역전파(Backpropagation)</strong>입니다. </li>
</ul>
<h3 id="lora를-붙이는-순간과-학습되는-순간">LoRA를 붙이는 순간과 학습되는 순간</h3>
<ul>
<li>LoRA를 붙이는 순간: <code>target_modules</code>와 <code>r</code>에 따라 A·B의 모양과 파라미터 수가 즉시 정해진다.</li>
<li>학습 중: Forward와 Backward를 반복하며 A·B의 실제 값이 데이터에 맞게 채워진다.</li>
</ul>
<p>일반적인 초기 상태에서는 한 행렬은 작은 무작위 값, 다른 행렬은 0으로 두어 LoRA 변화량이 0에서 시작하도록 한다. 따라서 시작 시점의 동작은 Base Model과 같고 학습하면서 보정이 생긴다.</p>
<h2 id="7-10-학습-실행과-저장">7-10. 학습 실행과 저장</h2>
<pre><code class="language-python">dataset = load_training_dataset(train_path, validation_path)
tokenizer = load_tokenizer()
model = load_model(profile)
lora_config = create_lora_config(profile)

trainer = SFTTrainer(
    model=model,
    args=create_sft_config(profile, output_dir),
    train_dataset=dataset[&quot;train&quot;],
    eval_dataset=dataset[&quot;validation&quot;],
    processing_class=tokenizer,
    peft_config=lora_config,
)

trainer.train()
trainer.save_model(str(output_dir))
tokenizer.save_pretrained(str(output_dir))</code></pre>
<p>저장 결과는 전체 Qwen 모델의 복사본이 아니라 Base에 덧붙일 Adapter다.</p>
<pre><code class="language-text">models/hr-qwen-lora/
├── adapter_config.json
└── adapter_model.safetensors</code></pre>
<p>개념적으로는 다음과 같다.</p>
<pre><code class="language-text">Base Qwen + HR LoRA Adapter = HR 업무에 적응한 Qwen</code></pre>
<h2 id="7-11-실습-실행-순서">7-11. 실습 실행 순서</h2>
<p>교재의 전체 데모 흐름은 다음과 같다.</p>
<pre><code class="language-bash"># 1) 원천 문서에서 Context-based QA SFT 데이터 생성
python scripts/build_training_dataset.py

# 2) Instruction SFT 데이터 검증·분리
python scripts/build_sft_dataset.py

# 3) LoRA/QLoRA 학습
SFT_MAX_LENGTH=512 \
SFT_EPOCHS=3 \
SFT_LEARNING_RATE=1e-4 \
SFT_GRAD_ACCUM=8 \
python scripts/train_lora.py

# 4) Base와 Fine-tuned 모델 평가
python scripts/evaluate_model.py</code></pre>
<p>학생 실습 가이드의 수업 권장 예시는 더 가볍다.</p>
<pre><code class="language-bash">SFT_MAX_LENGTH=384 \
SFT_EPOCHS=2 \
SFT_LEARNING_RATE=1e-4 \
python scripts/train_lora.py</code></pre>
<table>
<thead>
<tr>
<th>환경 변수</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>SFT_MAX_LENGTH</code></td>
<td>한 학습 샘플에 사용할 최대 토큰 길이</td>
</tr>
<tr>
<td><code>SFT_EPOCHS</code></td>
<td>전체 Train 데이터를 반복할 횟수</td>
</tr>
<tr>
<td><code>SFT_LEARNING_RATE</code></td>
<td>LoRA Adapter의 학습률</td>
</tr>
<tr>
<td><code>SFT_GRAD_ACCUM</code></td>
<td>여러 미니배치의 기울기를 모아 한 번 업데이트하는 횟수</td>
</tr>
</tbody></table>
<p><code>max_length</code>가 작으면 메모리는 줄지만 긴 답이 잘릴 수 있다. Epoch가 너무 많으면 과적합할 수 있고, 학습률이 너무 크면 학습이 불안정해질 수 있다.</p>
<p>교재에는 재학습 전 <code>rm -rf models/hr-qwen-lora</code>로 기존 Adapter를 지우는 명령도 제시된다. 이 명령은 복구가 어려우므로 <strong>경로를 확인하고 기존 결과가 필요 없는 경우에만</strong> 사용해야 한다. 가능하면 먼저 다른 이름으로 보관한다.</p>
<h2 id="7-12-학습-로그-읽기">7-12. 학습 로그 읽기</h2>
<p>학습 중에는 다음 값이 중요하다.</p>
<ul>
<li><code>loss</code>: Train 데이터에서의 오차</li>
<li><code>eval_loss</code>: Validation 데이터에서의 오차</li>
<li><code>learning_rate</code>: 현재 학습률</li>
<li><code>epoch</code>: 데이터 전체 반복 횟수</li>
<li><code>step</code>: 파라미터 업데이트 진행 단계</li>
</ul>
<pre><code class="language-text">Train loss ↓, Eval loss ↓
→ 대체로 학습과 일반화가 함께 좋아지는 신호

Train loss ↓, Eval loss ↑
→ Train 데이터에만 맞는 과적합 가능성

둘 다 거의 변하지 않음
→ 학습률, 데이터 형식, 학습 대상, 샘플 품질 점검</code></pre>
<p>대표 로그 파일은 다음과 같다.</p>
<pre><code class="language-text">outputs/
├── training_log.json
└── training_summary.json</code></pre>
<h2 id="7-13-base-vs-lora-평가-구조">7-13. Base vs LoRA 평가 구조</h2>
<p>공정한 비교를 위해 동일한 모델 로딩 경로와 생성 조건을 사용하고 Adapter만 켰다 껐다 한다.</p>
<pre><code class="language-python">tuned_model = PeftModel.from_pretrained(
    base_model,
    str(ADAPTER_PATH),
    is_trainable=False,
)

with tuned_model.disable_adapter():
    base_answer = generate_answer(tuned_model, tokenizer, question)

tuned_answer = generate_answer(tuned_model, tokenizer, question)</code></pre>
<pre><code class="language-text">같은 Evaluation 질문
      ├── Adapter OFF → Base 답변
      └── Adapter ON  → Fine-tuned 답변
                         ↓
                    같은 지표로 비교</code></pre>
<p>Temperature, max tokens, seed 등 생성 조건도 같아야 비교가 공정하다.</p>
<h2 id="7-14-평가-지표">7-14. 평가 지표</h2>
<h3 id="keyword-score">Keyword Score</h3>
<p><code>must_include</code> 핵심어 중 답변에 실제 포함된 비율이다. 금지어가 포함되면 감점할 수 있다.</p>
<pre><code class="language-python">score = 포함된_필수_키워드_수 / 전체_필수_키워드_수</code></pre>
<p>회사 코드, 숫자, 기한처럼 정확히 포함돼야 하는 정보를 측정하기 좋다. 그러나 키워드만 억지로 나열한 틀린 답도 높은 점수를 받을 수 있어 단독 사용하면 안 된다.</p>
<h3 id="token-f1">Token F1</h3>
<p>생성 답변과 Reference Answer에서 겹치는 토큰의 Precision과 Recall을 조화평균한다.</p>
<pre><code class="language-text">Precision: 모델이 말한 내용 중 정답과 겹치는 비율
Recall: 정답의 내용 중 모델이 포함한 비율
F1: 두 값을 함께 고려한 점수</code></pre>
<p>표현이 다른 올바른 답에는 낮게 나올 수 있다는 한계가 있다.</p>
<h3 id="hallucination-safety">Hallucination Safety</h3>
<p><code>unknown_policy</code> 질문에서 “규정에 없음”, “담당 부서 확인” 같은 안전 표현이 있는지, 근거 없는 확답 패턴은 없는지 확인한다.</p>
<h3 id="종합-점수-예시">종합 점수 예시</h3>
<pre><code class="language-text">일반 질문:
keyword × 0.7 + F1 × 0.3

규정에 없는 질문:
keyword × 0.3 + F1 × 0.2 + safety × 0.5</code></pre>
<p>가중치는 업무 위험에 맞춰 정해야 한다. HR 규정처럼 잘못된 확답이 위험한 서비스라면 Safety 비중을 높이는 것이 합리적이다.</p>
<h2 id="7-15-평가-결과물">7-15. 평가 결과물</h2>
<pre><code class="language-text">outputs/
├── evaluation_results.jsonl   # 질문별 Base·Fine-tuned 상세 결과
└── evaluation_summary.json    # 전체 평균과 승패 요약</code></pre>
<p>확인해야 할 것은 단순한 승리 횟수만이 아니다.</p>
<ul>
<li>Fine-tuned가 Base보다 Keyword Score가 실제로 높은가?</li>
<li>Token F1과 전체 평균이 함께 좋아졌는가?</li>
<li>규정에 없는 질문의 안전 응답률이 좋아졌는가?</li>
<li>특정 질문 유형만 좋아지고 다른 유형은 나빠지지 않았는가?</li>
<li>답이 길어지기만 해서 점수가 오른 것은 아닌가?</li>
</ul>
<hr />
<h1 id="part-8-lora-파라미터-구조-초보자가-자주-묻는-16가지">Part 8. LoRA 파라미터 구조: 초보자가 자주 묻는 16가지</h1>
<h2 id="q1-기존-파라미터-일부를-골라-학습하나">Q1. 기존 파라미터 일부를 골라 학습하나?</h2>
<p>아니다. Base 파라미터는 동결하고, 선택한 Linear Layer 옆에 새로운 A·B 행렬을 추가해 그것만 학습한다.</p>
<h2 id="q2-어댑터가-병렬로-붙는다는-뜻인가">Q2. 어댑터가 병렬로 붙는다는 뜻인가?</h2>
<p>그렇다. 기존 <code>Wx</code> 경로는 그대로 두고 작은 <code>BAx</code> 경로를 병렬로 계산해 최종 출력에 더한다. 별도 모델 두 개가 도는 것이 아니라 한 레이어 안에 작은 보정 경로가 생긴다.</p>
<h2 id="q3-왜-작은-보정만으로-가능한가">Q3. 왜 작은 보정만으로 가능한가?</h2>
<p>새 과제에 필요한 변화량은 원본 모델 전체보다 낮은 차원의 패턴으로 표현할 수 있다는 저랭크 가설을 이용한다.</p>
<h2 id="q4-a와-b의-값은-누가-정하나">Q4. A와 B의 값은 누가 정하나?</h2>
<p>사람이 직접 입력하지 않는다. 데이터에 대한 Loss를 줄이는 방향으로 역전파와 Optimizer가 자동 학습한다. Base 모델과 데이터가 달라지면 결과 값도 달라진다.</p>
<h2 id="q5-lora를-붙이자마자-trainable-parameter가-보이는-이유는">Q5. LoRA를 붙이자마자 Trainable Parameter가 보이는 이유는?</h2>
<p>붙이는 순간 <code>target_modules</code>와 <code>r</code>에 따라 A·B의 <strong>자리와 개수</strong>가 생긴다. 그러나 쓸모 있는 <strong>값</strong>은 이후 학습으로 채워진다.</p>
<h2 id="q6-모든-파라미터에-lora를-붙이나">Q6. 모든 파라미터에 LoRA를 붙이나?</h2>
<p>아니다. <code>target_modules</code>에 지정한 Linear Layer에만 붙인다.</p>
<h2 id="q7-q_proj-같은-이름은-누가-정하나">Q7. q_proj 같은 이름은 누가 정하나?</h2>
<p>Qwen 모델 구조를 만든 설계자가 정했다. Fine-Tuning 담당자는 이미 존재하는 모듈 중 적용 대상을 고른다.</p>
<h2 id="q8-q-k-v의-역할은">Q8. Q, K, V의 역할은?</h2>
<p>Q는 찾고 싶은 것, K는 비교할 색인, V는 실제로 가져올 정보다.</p>
<h2 id="q9-왜-간단한-예제는-q와-v만-사용하나">Q9. 왜 간단한 예제는 Q와 V만 사용하나?</h2>
<p>학습 파라미터를 작게 유지해 LoRA 원리를 관찰하기 쉽기 때문이다. 필수 규칙은 아니며 실제 SFT에서는 Attention 전체와 MLP까지 확장할 수 있다.</p>
<h2 id="q10-q_proj-하나를-지정하면-한-층에만-붙나">Q10. q_proj 하나를 지정하면 한 층에만 붙나?</h2>
<p>아니다. 반복된 모든 Transformer Layer의 <code>q_proj</code>에 각각 별도의 A·B가 붙고 독립적으로 학습된다.</p>
<h2 id="q11-파라미터가-줄어드는-수학적-이유는">Q11. 파라미터가 줄어드는 수학적 이유는?</h2>
<p>큰 <code>d_out × d_in</code> 변화 행렬을 직접 학습하지 않고 <code>d_out × r</code>과 <code>r × d_in</code> 두 작은 행렬로 분해하기 때문이다.</p>
<pre><code class="language-text">Full 변화량 파라미터: d_out × d_in
LoRA 파라미터:       r × d_in + d_out × r</code></pre>
<h2 id="q12-실무에서-무엇을-결정해야-하나">Q12. 실무에서 무엇을 결정해야 하나?</h2>
<p>핵심은 <strong>어디까지 바꿀지</strong>(<code>target_modules</code>)와 <strong>얼마나 큰 보정 용량을 줄지</strong>(<code>r</code>)다.</p>
<ul>
<li>적은 Target + 낮은 r: 싸고 가볍지만 표현력 작음</li>
<li>많은 Target + 높은 r: 표현력 크지만 메모리·시간·과적합 위험 증가</li>
</ul>
<h2 id="q13-lora를-한-문장으로-말하면">Q13. LoRA를 한 문장으로 말하면?</h2>
<blockquote>
<p>Base Model을 동결한 채 특정 Linear Layer 옆에 작은 저랭크 Adapter를 추가하고 그것만 학습해, 적은 비용으로 모델 행동을 조정하는 PEFT 기법이다.</p>
</blockquote>
<h2 id="q14-여러-lora-adapter를-동시에-쓸-수-있나">Q14. 여러 LoRA Adapter를 동시에 쓸 수 있나?</h2>
<p>기술적으로 선택·전환·조합할 수 있다. 다만 언어모델에서는 서로 다른 문체·규칙·도메인이 간섭할 수 있어 무작정 합치기보다 라우팅하거나 하나의 통합 데이터셋으로 다시 학습하는 편이 안전하다.</p>
<h2 id="q15-같은-모델이면-lora-결과도-항상-같은가">Q15. 같은 모델이면 LoRA 결과도 항상 같은가?</h2>
<p>아니다. Base 가중치, 데이터 순서, seed, <code>r</code>, <code>alpha</code>, target, 학습률, 배치 크기, Epoch, 정밀도와 하드웨어 연산까지 영향을 준다. 구조와 파라미터 수는 같아도 최종 A·B 값은 달라질 수 있다.</p>
<h2 id="q16-결국-데이터가-가장-중요한가">Q16. 결국 데이터가 가장 중요한가?</h2>
<p>좋은 범위에서 하이퍼파라미터를 선택했다면, 실제 품질은 질문-답변의 정확성, 다양성, 균형, 안전 예시, 도메인 전문가의 검수에 크게 좌우된다.</p>
<blockquote>
<p>LoRA는 학습할 그릇을 작게 만드는 기술이고, 그 안에 무엇을 담는지는 데이터 설계가 결정한다.</p>
</blockquote>
<hr />
<h1 id="part-9-실습-결과를-해석할-때-흔한-실수">Part 9. 실습 결과를 해석할 때 흔한 실수</h1>
<h2 id="9-1-loss가-낮으면-무조건-좋은-모델이다">9-1. Loss가 낮으면 무조건 좋은 모델이다?</h2>
<p>아니다. Train Loss만 낮고 Eval Loss가 높으면 과적합일 수 있다. 또한 Loss가 낮아도 실제 업무의 안전성과 사실성이 나쁠 수 있다.</p>
<h2 id="9-2-fine-tuning이-모든-사내-문서를-정확히-기억한다">9-2. Fine-Tuning이 모든 사내 문서를 정확히 기억한다?</h2>
<p>아니다. 작은 데이터로 학습하면 행동과 일부 표현은 개선돼도 모든 사실을 완전하고 정확하게 저장한다고 보장할 수 없다. 최신 규정은 RAG와 원문 출처로 보강해야 한다.</p>
<h2 id="9-3-rag는-모델을-학습한다">9-3. RAG는 모델을 학습한다?</h2>
<p>기본적인 RAG는 모델 파라미터를 바꾸지 않는다. 질문 시점에 관련 문서를 검색해 입력에 추가한다.</p>
<h2 id="9-4-4bit-quantization이-모델을-더-똑똑하게-만든다">9-4. 4bit Quantization이 모델을 더 똑똑하게 만든다?</h2>
<p>아니다. 목적은 메모리와 추론 비용 절감이다. 정밀도가 줄어 작은 성능 저하가 생길 수 있다.</p>
<h2 id="9-5-keyword-score-하나만-높으면-성공이다">9-5. Keyword Score 하나만 높으면 성공이다?</h2>
<p>아니다. 문장의 의미, 금지 내용, 안전 응답, 사람 평가를 함께 봐야 한다.</p>
<h2 id="9-6-학습-데이터와-평가-데이터를-섞어도-된다">9-6. 학습 데이터와 평가 데이터를 섞어도 된다?</h2>
<p>안 된다. 시험 문제를 미리 공부한 것과 같아 성능이 부풀려진다. 유사 질문 중복까지 검사해야 한다.</p>
<hr />
<h1 id="part-10-1일차-핵심-체크리스트">Part 10. 1일차 핵심 체크리스트</h1>
<h2 id="개념">개념</h2>
<ul>
<li><input disabled="" type="checkbox" /> LLM 서빙의 요청 → 전처리 → 추론 → 후처리 → 응답 흐름을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 외부 상용 LLM과 내부 sLLM 파이프라인의 장단점을 비교할 수 있다.</li>
<li><input disabled="" type="checkbox" /> MLM과 CLM의 학습 목표와 대표 용도를 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> sLLM이 단순히 작은 모델만을 뜻하지 않는 이유를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> RAG, Fine-Tuning, Quantization의 역할을 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Full Fine-Tuning과 PEFT의 차이를 설명할 수 있다.</li>
</ul>
<h2 id="lora">LoRA</h2>
<ul>
<li><input disabled="" type="checkbox" /> Base 가중치는 동결되고 A·B 행렬이 새로 학습된다는 것을 안다.</li>
<li><input disabled="" type="checkbox" /> <code>r</code>, <code>lora_alpha</code>, <code>lora_dropout</code>, <code>target_modules</code>의 뜻을 안다.</li>
<li><input disabled="" type="checkbox" /> Q, K, V, O 및 MLP Projection의 역할을 대략 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> LoRA와 QLoRA의 차이가 Base Model 양자화 여부임을 안다.</li>
</ul>
<h2 id="데이터와-평가">데이터와 평가</h2>
<ul>
<li><input disabled="" type="checkbox" /> Train, Validation, Evaluation을 분리하는 이유를 안다.</li>
<li><input disabled="" type="checkbox" /> Instruction SFT와 Context-based QA SFT의 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> 규정에 없는 질문을 위한 안전 답변 데이터가 왜 필요한지 안다.</li>
<li><input disabled="" type="checkbox" /> Keyword Score, Token F1, Hallucination Safety를 함께 봐야 함을 안다.</li>
<li><input disabled="" type="checkbox" /> Base와 Fine-tuned 모델의 생성 조건을 동일하게 비교할 수 있다.</li>
</ul>
<hr />
<h1 id="part-11-스스로-확인하는-미니-퀴즈">Part 11. 스스로 확인하는 미니 퀴즈</h1>
<details>
<strong>1. BERT와 Qwen 중 사내 답변 생성 SFT에 더 직접적으로 맞는 구조는?</strong>

<p>Qwen 같은 Decoder-only CLM이다. 지시 뒤에 응답 토큰을 순서대로 생성하는 학습과 잘 맞기 때문이다.</p>
</details>

<details>
<strong>2. LoRA는 기존 15억 파라미터 가운데 일부를 골라 수정하는가?</strong>

<p>아니다. 기존 파라미터는 동결하고 선택한 Linear Layer 옆에 새로운 저랭크 A·B 파라미터를 추가해 학습한다.</p>
</details>

<details>
<strong>3. 회사 규정이 매주 바뀐다면 Fine-Tuning만 하면 될까?</strong>

<p>권장하기 어렵다. 최신 사실은 RAG로 현재 문서를 검색하고, Fine-Tuning은 답변 방식과 근거 준수 행동을 학습하는 식으로 역할을 나누는 편이 관리하기 쉽다.</p>
</details>

<details>
<strong>4. Train Loss는 계속 내려가는데 Eval Loss가 올라간다면?</strong>

<p>과적합 가능성이 있다. Epoch, 학습률, 데이터 중복, 데이터 다양성, Adapter 용량을 점검한다.</p>
</details>

<details>
<strong>5. 규정에 없는 질문에 그럴듯한 답을 만든다면 무엇을 개선해야 하나?</strong>

<p>Unknown-policy 안전 예시를 보강하고, RAG 근거 없이는 답하지 않는 프롬프트·검증 로직을 추가하며, Hallucination Safety 평가를 강화한다.</p>
</details>

<hr />
<h1 id="마무리">마무리</h1>
<p>1일차의 출발은 <strong>내부 데이터 주권과 도메인 최적화를 위해 sLLM 파이프라인이 필요할 수 있다</strong>는 문제의식이다. 생성형 업무에는 Decoder-only CLM이 자연스럽고, 작은 모델을 실제 업무에 맞추기 위해 Instruction SFT와 PEFT를 사용할 수 있다. LoRA는 Base Model을 동결한 채 작은 저랭크 보정 행렬만 학습하므로 비용을 크게 줄인다. 그러나 최종 품질을 결정하는 것은 단지 <code>r</code>이나 <code>alpha</code>가 아니라 <strong>정확하고 다양한 도메인 데이터, 공정한 평가, 근거 없는 확답을 막는 안전 설계</strong>다.</p>
<p>다음 단계에서는 오늘 만든 Adapter와 데이터·평가 이해를 바탕으로 RAG, 서비스 파이프라인, 추론 최적화와 배포 구조를 연결하게 된다.</p>