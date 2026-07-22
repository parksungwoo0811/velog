<h2 id="개념">개념</h2>
<p><strong>NLP(Natural Language Processing, 자연어 처리)</strong>는 사람이 사용하는 <strong>자연어(한국어, 영어 등)를 컴퓨터가 이해하고, 분석하며, 생성할 수 있도록 하는 인공지능 기술</strong>이다.</p>
<p>즉, 사람의 언어를 컴퓨터가 처리할 수 있는 형태로 변환하여 의미를 이해하고 활용하는 기술이다.</p>
<hr />
<h1 id="왜-필요한가">왜 필요한가?</h1>
<p>컴퓨터는 숫자만 이해할 수 있기 때문에 사람이 사용하는 문장을 그대로 이해하지 못한다.</p>
<p>예를 들어,</p>
<blockquote>
<p>&quot;오늘 날씨가 정말 좋다.&quot;</p>
</blockquote>
<p>사람은 자연스럽게 의미를 이해하지만, 컴퓨터는 단순한 문자열로만 인식한다.</p>
<p>NLP는 이러한 문장을 분석하여</p>
<ul>
<li>단어를 분리하고</li>
<li>문장의 구조를 파악하며</li>
<li>의미를 이해하고</li>
<li>필요한 답변이나 결과를 생성한다.</li>
</ul>
<hr />
<h1 id="nlp의-주요-작업">NLP의 주요 작업</h1>
<table>
<thead>
<tr>
<th>작업</th>
<th>설명</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>토큰화(Tokenization)</td>
<td>문장을 단어나 문장 단위로 분리</td>
<td>&quot;안녕하세요 여러분&quot; → [&quot;안녕하세요&quot;, &quot;여러분&quot;]</td>
</tr>
<tr>
<td>형태소 분석</td>
<td>단어의 품사와 의미 분석</td>
<td>먹었습니다 → 먹다(동사) + 었 + 습니다</td>
</tr>
<tr>
<td>개체명 인식(NER)</td>
<td>사람, 장소, 회사 등을 식별</td>
<td>&quot;삼성전자&quot; → 기업</td>
</tr>
<tr>
<td>감성 분석</td>
<td>긍정·부정 감정 분석</td>
<td>&quot;정말 최고다.&quot; → 긍정</td>
</tr>
<tr>
<td>기계 번역</td>
<td>다른 언어로 번역</td>
<td>한국어 → 영어</td>
</tr>
<tr>
<td>문서 요약</td>
<td>긴 문서를 짧게 요약</td>
<td>뉴스 요약</td>
</tr>
<tr>
<td>질의응답(QA)</td>
<td>질문에 대한 답변 생성</td>
<td>ChatGPT</td>
</tr>
<tr>
<td>텍스트 생성</td>
<td>새로운 문장 생성</td>
<td>이메일 작성, 블로그 작성</td>
</tr>
</tbody></table>
<hr />
<h1 id="nlp-처리-과정">NLP 처리 과정</h1>
<pre><code>텍스트 입력
      ↓
토큰화(Tokenization)
      ↓
형태소 분석
      ↓
의미 분석
      ↓
AI 모델(BERT, GPT 등)
      ↓
결과 생성</code></pre><hr />
<h1 id="nlp-기술-예시">NLP 기술 예시</h1>
<ul>
<li>ChatGPT</li>
<li>번역기(파파고, Google Translate)</li>
<li>Siri</li>
<li>Gemini</li>
<li>Claude</li>
<li>음성 비서</li>
<li>챗봇</li>
<li>검색 엔진</li>
</ul>
<hr />
<h1 id="nlp와-llm의-관계">NLP와 LLM의 관계</h1>
<ul>
<li><strong>NLP</strong>는 자연어를 처리하는 <strong>인공지능 분야 전체</strong>를 의미한다.</li>
<li><strong>LLM(Large Language Model)</strong>은 NLP를 구현하는 <strong>최신 기술 중 하나</strong>이다.</li>
</ul>
<p>즉,</p>
<pre><code>인공지능(AI)
    └── NLP(자연어 처리)
          ├── 규칙 기반 NLP
          ├── 머신러닝 기반 NLP
          ├── 딥러닝 기반 NLP
          └── LLM(GPT, Gemini, Claude 등)</code></pre><p>LLM은 NLP 기술을 더욱 발전시킨 대규모 언어 모델이라고 볼 수 있다.</p>
<hr />
<h1 id="한-줄-정리">한 줄 정리</h1>
<blockquote>
<p><strong>NLP(Natural Language Processing)는 사람이 사용하는 자연어를 컴퓨터가 이해·분석·생성할 수 있도록 하는 인공지능 기술이다.</strong></p>
</blockquote>
<h3 id="1-텍스트를-숫자로-표현하는-방법">1. 텍스트를 숫자로 표현하는 방법</h3>
<p>초기 NLP는 문장을 단순히 <strong>단어의 등장 여부나 빈도</strong>로 표현했다.</p>
<p>대표적인 방법은 다음과 같다.</p>
<ul>
<li><strong>Corpus</strong> : 모델이 학습하는 전체 텍스트 데이터 집합</li>
<li><strong>Vocabulary</strong> : Corpus에서 추출한 전체 단어 목록</li>
<li><strong>One-hot Vector</strong> : 각 단어를 0과 1로만 표현하는 방식</li>
<li><strong>BoW(Bag of Words)</strong> : 단어의 등장 횟수로 문서를 표현</li>
<li><strong>TF-IDF</strong> : 자주 등장하지만 모든 문서에 흔한 단어의 중요도를 낮추고, 특정 문서에서만 중요한 단어의 가중치를 높이는 방식</li>
<li><strong>N-gram</strong> : 연속된 N개의 단어를 하나의 단위로 표현하는 방법</li>
</ul>
<p>하지만 이러한 방법들은 <strong>단어의 순서, 의미, 문맥을 제대로 반영하지 못한다</strong>는 한계가 있었다.</p>
<hr />
<h3 id="2-의미를-이해하는-표현-방식">2. 의미를 이해하는 표현 방식</h3>
<p>이러한 한계를 해결하기 위해 <strong>Word Embedding</strong>이 등장했다.</p>
<p>Word Embedding은 단어를 단순한 번호가 아니라 <strong>의미를 담은 벡터(Vector)</strong> 로 표현하는 방법이다.</p>
<p>대표적인 모델인 <strong>Word2Vec</strong>은</p>
<blockquote>
<p><strong>&quot;비슷한 문맥에서 사용되는 단어는 비슷한 의미(=유사한 성격)를 가진다.&quot;</strong></p>
</blockquote>
<p>라는 <strong>분포 가설(Distributional Hypothesis)</strong> 을 기반으로 학습한다.</p>
<p>따라서</p>
<ul>
<li>왕(King)</li>
<li>여왕(Queen)</li>
<li>남자(Man)</li>
<li>여자(Woman)</li>
</ul>
<p>와 같은 단어들의 의미 관계를 벡터 공간에서 학습할 수 있다.</p>
<p>하지만 Word2Vec은 <strong>하나의 단어를 항상 하나의 벡터로만 표현</strong>하기 때문에,</p>
<p>동음이의어나 문맥에 따라 의미가 달라지는 단어는 제대로 표현하지 못한다.</p>
<blockquote>
</blockquote>
<p><strong>연산이 가능한 벡터</strong></p>
<ul>
<li>시제 변화: 먹다 - 현재 + 과거 = 먹었다</li>
<li>수도 관계: 도쿄 - 일본 + 한국 = 서울</li>
</ul>
<hr />
<h3 id="3-언어-모델language-model">3. 언어 모델(Language Model)</h3>
<p>Word Embedding 이후에는 <strong>Language Model(언어 모델)</strong> 이 등장했다.</p>
<p>Language Model은</p>
<blockquote>
<p><strong>앞의 단어들을 보고 다음에 올 단어를 예측하면서 언어를 학습하는 모델</strong>이다.</p>
</blockquote>
<p>대표적인 모델은</p>
<ul>
<li><p><strong>RNN</strong> (앞 선 단어의 기록을 통해 기억하고 다음 예측에 활용)</p>
</li>
<li><blockquote>
<p>문장 길이가 길어지면 잊어버리는 문제가 있음.</p>
</blockquote>
</li>
<li><p><strong>LSTM</strong>(중요한 것만 골라 적고, 불필요한 건 지우개로 지우는 스마트 노트)</p>
</li>
<li><blockquote>
<p>계산해야 할 매개변수가 많아서 학습 속도가 느리다는 단점</p>
</blockquote>
</li>
<li><p><strong>GRU</strong>(LSTM의 다이어트 버전)</p>
</li>
</ul>
<p>가 있다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>RNN</th>
<th>LSTM</th>
<th>GRU</th>
</tr>
</thead>
<tbody><tr>
<td><strong>비유</strong></td>
<td>기억력 나쁜 메모장</td>
<td>정교한 3단계 분류 노트</td>
<td>핵심만 남긴 간소화 노트</td>
</tr>
<tr>
<td><strong>장점</strong></td>
<td>구조가 가장 단순함</td>
<td>긴 문장(장기 의존성)을 매우 잘 기억함</td>
<td>LSTM급 성능 + 더 빠르고 경량화됨</td>
</tr>
<tr>
<td><strong>단점</strong></td>
<td>긴 글을 읽으면 앞 내용을 잊음</td>
<td>구조가 복잡해 학습 속도가 느림</td>
<td>데이터가 아주 적을 땐 LSTM이 약간 더 나을 수도 있음</td>
</tr>
<tr>
<td><strong>구조</strong></td>
<td>Gate 없음</td>
<td>3개 Gate (Forget, Input, Output)</td>
<td>2개 Gate (Reset, Update)</td>
</tr>
</tbody></table>
<p>이 모델들은 문장을 <strong>앞에서부터 순서대로 읽으면서 학습</strong>하지만,</p>
<ul>
<li>긴 문장을 처리할수록 앞의 정보를 잊기 쉽고</li>
<li>병렬 처리가 어려워 학습 속도가 느리다.</li>
</ul>
<p>라는 한계가 있었다.</p>
<hr />
<h3 id="4-attention과-transformer">4. Attention과 Transformer</h3>
<p>이러한 문제를 해결하기 위해 등장한 핵심 아이디어가 <strong>Attention</strong>이다.</p>
<p>Attention은</p>
<blockquote>
<p><strong>현재 단어를 이해하기 위해 문장 속 어떤 단어가 중요한지 계산하여 더 집중하는 메커니즘</strong>이다.</p>
</blockquote>
<p>이를 발전시켜 만든 구조가 <strong>Transformer</strong>이다.</p>
<p>Transformer는 문장을 순서대로만 읽지 않고,</p>
<p><strong>모든 단어가 서로 어떤 관계를 가지는지 동시에 계산(Self-Attention)</strong> 한다.</p>
<p>덕분에</p>
<ul>
<li>긴 문장도 잘 이해할 수 있고</li>
<li>병렬 처리가 가능하며</li>
<li>문맥(Context)을 더욱 정확하게 이해할 수 있게 되었다.</li>
</ul>
<p>Transformer는 현재 대부분의 생성형 AI의 기반이 된다.</p>
<p>대표적인 모델은</p>
<ul>
<li>BERT</li>
<li>GPT</li>
<li>Gemini</li>
<li>Claude</li>
<li>Llama</li>
</ul>
<p>등이 있다.</p>
<hr />
<h3 id="5-contextual-embedding">5. Contextual Embedding</h3>
<p>Transformer의 가장 큰 발전은 <strong>Contextual Embedding(문맥 기반 임베딩)</strong> 이다.</p>
<p>같은 단어라도 <strong>문맥(Context)</strong> 에 따라 서로 다른 의미를 표현할 수 있다.</p>
<p>즉,</p>
<p>단어 하나의 의미가 아니라 <strong>문장 전체의 의미를 함께 고려하여 벡터를 생성</strong>한다.</p>
<p>이 기술 덕분에 현대 LLM은 사람처럼 문맥을 이해하는 것이 가능해졌다.</p>
<hr />
<h2 id="예시">예시</h2>
<h3 id="예시-1-bow">예시 1. BoW</h3>
<p>문장</p>
<pre><code>아주 아주 위험합니다.</code></pre><p>BoW는 단어의 빈도만 계산한다.</p>
<table>
<thead>
<tr>
<th>단어</th>
<th align="right">빈도</th>
</tr>
</thead>
<tbody><tr>
<td>아주</td>
<td align="right">2</td>
</tr>
<tr>
<td>위험합니다</td>
<td align="right">1</td>
</tr>
</tbody></table>
<p>BoW는 <strong>&quot;아주&quot;가 두 번 등장했다는 사실은 알지만</strong>, 문장의 순서나 의미는 이해하지 못한다.</p>
<hr />
<h3 id="예시-2-word-embedding">예시 2. Word Embedding</h3>
<p>다음 두 단어를 생각해 보자.</p>
<ul>
<li>동생</li>
<li>아우</li>
</ul>
<p>표현은 다르지만 의미는 거의 같다.</p>
<p>Word Embedding은 이러한 단어들을 <strong>벡터 공간에서 가까운 위치</strong>에 배치한다.</p>
<p>즉,</p>
<blockquote>
<p>의미가 비슷한 단어는 벡터도 서로 가깝게 표현된다.</p>
</blockquote>
<hr />
<h3 id="예시-3-contextual-embedding">예시 3. Contextual Embedding</h3>
<p>영어 단어 <strong>bank</strong>는 문맥에 따라 의미가 달라진다.</p>
<table>
<thead>
<tr>
<th>문장</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>He deposited money in the <strong>bank</strong>.</td>
<td>은행</td>
</tr>
<tr>
<td>They had a picnic on the river <strong>bank</strong>.</td>
<td>강둑</td>
</tr>
</tbody></table>
<p>기존 Word2Vec은 <strong>bank를 하나의 벡터</strong>로만 표현한다.</p>
<p>하지만 Transformer 기반 모델은 문맥을 함께 고려하여</p>
<ul>
<li>은행</li>
<li>강둑</li>
</ul>
<p>을 서로 다른 의미로 이해한다.</p>
<hr />
<h2 id="실무-활용">실무 활용</h2>
<p>현대 AI 프로젝트에서는 NLP가 거의 모든 언어 기반 서비스의 핵심 기술로 활용된다.</p>
<table>
<thead>
<tr>
<th>활용 분야</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>문서 분류</td>
<td>뉴스, 이메일, 고객 문의 등을 자동으로 분류</td>
</tr>
<tr>
<td>감성 분석</td>
<td>리뷰가 긍정인지 부정인지 분석</td>
</tr>
<tr>
<td>검색 시스템</td>
<td>의미가 비슷한 문서를 벡터 유사도로 검색</td>
</tr>
<tr>
<td>추천 시스템</td>
<td>사용자의 관심사와 비슷한 콘텐츠 추천</td>
</tr>
<tr>
<td>챗봇</td>
<td>자연어 질문을 이해하고 답변 생성</td>
</tr>
<tr>
<td>RAG</td>
<td>PDF, 사내 문서, 매뉴얼 등을 검색하여 LLM 답변에 활용</td>
</tr>
<tr>
<td>기계 번역</td>
<td>한국어 ↔ 영어 등 다양한 언어 번역</td>
</tr>
<tr>
<td>문서 요약</td>
<td>긴 문서나 회의록의 핵심 내용 요약</td>
</tr>
<tr>
<td>키워드 추출</td>
<td>중요한 단어와 개체명을 자동 추출</td>
</tr>
<tr>
<td>정보 추출</td>
<td>이름, 날짜, 장소 등 필요한 정보만 추출</td>
</tr>
</tbody></table>
<p>실무에서는 단순히 단어의 빈도를 계산하는 것이 아니라,</p>
<p><strong>텍스트를 의미 벡터(Embedding)로 변환한 뒤</strong></p>
<ul>
<li>유사도 검색</li>
<li>추천 시스템</li>
<li>문서 검색</li>
<li>질의응답</li>
<li>생성형 AI</li>
<li>LLM</li>
</ul>
<p>등 다양한 서비스에 활용한다.</p>
<hr />
<h2 id="핵심-정리">핵심 정리</h2>
<table>
<thead>
<tr>
<th>개념</th>
<th>쉬운 설명</th>
<th>한계 또는 특징</th>
<th>실무 활용</th>
</tr>
</thead>
<tbody><tr>
<td>Corpus</td>
<td>모델이 학습하는 전체 텍스트 데이터</td>
<td>데이터 품질이 매우 중요</td>
<td>모델 학습 데이터</td>
</tr>
<tr>
<td>Vocabulary</td>
<td>Corpus에서 추출한 단어 목록</td>
<td>단어 수가 많을수록 차원이 커짐</td>
<td>토큰화, 벡터화</td>
</tr>
<tr>
<td>One-hot Vector</td>
<td>단어를 0과 1로 표현</td>
<td>의미 관계를 표현하지 못함</td>
<td>기본 텍스트 표현</td>
</tr>
<tr>
<td>BoW</td>
<td>단어 등장 횟수로 문서 표현</td>
<td>순서와 문맥을 반영하지 못함</td>
<td>간단한 분류, 검색</td>
</tr>
<tr>
<td>TF-IDF</td>
<td>중요한 단어에 높은 가중치 부여</td>
<td>의미 자체는 이해하지 못함</td>
<td>검색, 키워드 추출</td>
</tr>
<tr>
<td>N-gram</td>
<td>연속된 N개의 단어를 하나로 표현</td>
<td>N이 커질수록 데이터가 급격히 증가</td>
<td>문장 패턴 분석</td>
</tr>
<tr>
<td>Word Embedding</td>
<td>단어를 의미 벡터로 표현</td>
<td>문맥에 따라 의미가 바뀌는 단어 처리 한계</td>
<td>유사도 검색, 추천</td>
</tr>
<tr>
<td>Word2Vec</td>
<td>비슷한 문맥의 단어를 비슷한 벡터로 학습</td>
<td>하나의 단어는 하나의 벡터만 가짐</td>
<td>의미 기반 검색</td>
</tr>
<tr>
<td>Language Model</td>
<td>다음 단어를 예측하며 언어를 학습</td>
<td>초기 모델은 긴 문맥 처리에 약함</td>
<td>자동완성, 생성 AI</td>
</tr>
<tr>
<td>RNN / LSTM / GRU</td>
<td>순차적으로 문장을 읽는 모델</td>
<td>병렬 처리와 장기 의존성에 한계</td>
<td>초기 NLP 모델</td>
</tr>
<tr>
<td>Attention</td>
<td>중요한 단어에 더 집중하는 메커니즘</td>
<td>Transformer의 핵심 기술</td>
<td>번역, 요약, 질의응답</td>
</tr>
<tr>
<td>Transformer</td>
<td>Self-Attention 기반의 언어 모델 구조</td>
<td>많은 데이터와 연산 자원 필요</td>
<td>GPT, BERT, LLM</td>
</tr>
<tr>
<td>Contextual Embedding</td>
<td>문맥에 따라 단어 의미를 다르게 표현</td>
<td>현대 NLP의 핵심 기술</td>
<td>챗봇, 검색, RAG</td>
</tr>
<tr>
<td>LLM</td>
<td>방대한 텍스트를 학습한 대규모 언어 모델</td>
<td>Hallucination, Bias 등의 한계 존재</td>
<td>ChatGPT, Claude, Gemini</td>
</tr>
</tbody></table>
<hr />
<h2 id="흐름으로-이해하기">흐름으로 이해하기</h2>
<pre><code>Corpus
    ↓
Vocabulary
    ↓
One-hot Vector
    ↓
BoW / TF-IDF / N-gram
    ↓
Word Embedding (Word2Vec)
    ↓
Language Model (RNN → LSTM → GRU)
    ↓
Attention
    ↓
Transformer
    ↓
Contextual Embedding
    ↓
LLM (GPT, BERT, Gemini, Claude 등)</code></pre><hr />
<h2 id="한-줄-요약">한 줄 요약</h2>
<blockquote>
<p><strong>NLP는 사람의 언어를 숫자(벡터)로 변환하고, 문맥과 의미를 학습하여 검색, 번역, 추천, 요약, 챗봇, RAG, LLM과 같은 다양한 AI 서비스를 가능하게 하는 핵심 기술이다.</strong></p>
</blockquote>